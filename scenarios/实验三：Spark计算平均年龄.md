# 实验目的
学习RDD算子相关操作

# 实验原理
- map(func)：数据集中的每个元素经过用户自定义的函数转换形成一个新的RDD，新的RDD叫MappedRDD
![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/3-1_20180410100618.018.png)

- flatMap(func):与map类似，但每个元素输入项都可以被映射到0个或多个的输出项，最终将结果”扁平化“后输出
![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/3-2_20180410100621.021.png)

- reduce
def reduce(f: (T, T) ⇒ T): T
根据映射函数f，对RDD中的元素进行二元计算，返回计算结果。

# 实验步骤
本次实验使用Spark统计一千万个人的年龄总和，然后计算出平均年龄。

#### 1. 生成数据
数据的格式如下所示：
```
id age
1  22
2  23
3  30
```
每行数据分为两列，第一列为id，第二列为年龄，年龄范围为[1, 100]。实验环境中已提前生成了一千万条数据，存放路径为 /data/spark/input/ages.txt。 读者可根据实际需要生成不同数量的数据集，下面给出生成数据的Java代码，更改CAPACITY可改变生成数量，更改PATH可改变文件保存路径。
```
import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStreamWriter;
import java.util.Iterator;
import java.util.LinkedList;
import java.util.List;
import java.util.Random;

public class GenAge {

	public static final int CAPACITY = 10000000;
	public static final String PATH = "/usr/local/spark/input/ages.txt";
	public static List<Integer> ages = new LinkedList<Integer>();

	public static void main(String[] args) throws Exception {
		Random random = new Random();

		for (int i = 0; i < CAPACITY; i++) {
			ages.add(Math.abs(random.nextInt(100) + 1));
		}

		WriteToFile();
	}

	public static void WriteToFile() throws Exception {
		FileOutputStream fos = new FileOutputStream(new File(PATH));
		OutputStreamWriter osw = new OutputStreamWriter(fos);

		Iterator<Integer> it = ages.iterator();

		int index = 0;
		while (it.hasNext()) {
			osw.write(index++ + " " + it.next() +"\n");
		}

		osw.flush();
		osw.close();
	}
}

```

#### 2. 编写代码
1. 打开Eclipse，新建Java项目，项目名为AverageAge，包名为org.apache.spark，新建Java类，类名为AverageAge

2. 导入Spark相关Jar包

在项目名上点击右键，依次点击 Build Path >> Configure BuildPath >> Libraries  >> Add External JARs

在弹出的对话框中进入Spark jars目录 /usr/local/spark/jars，Ctrl+A选择全部的jar文件，点击OK。

3. 输入代码

```

package org.apache.spark;

import java.util.Arrays;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

public class AverageAge {
	private static final String SPACE = " ";

	public static void main(String[] args) {

		SparkConf sparkConf = new SparkConf().setAppName("AverageAge");

		// 读取文件
		JavaSparkContext sc = new JavaSparkContext(sparkConf);
		JavaRDD<String> dataFile = sc.textFile("file:///data/spark/input/ages.txt");

		// 数据分片并取第二个数
		JavaRDD<String> ageData = dataFile.flatMap(s -> Arrays.asList(s.split(SPACE)[1]).iterator());

		// 求出所有年龄个数。
		long count = ageData.count();

		// 转换数据类型
		JavaRDD<Long> ageDataLong = ageData.map(s -> Long.parseLong(String.valueOf(s)));

		// 求出年龄的和
		Long totalAge = ageDataLong.reduce((x, y) -> x + y);

		// 计算平均值
		Double avgAge = totalAge.doubleValue() / count;

		// 输出结果
		System.out.println("Total Age:" + totalAge + ";    Number of People:" + count);
		System.out.println("Average Age is " + avgAge);

		sc.close();
	}
}

```

4. 导出为jar文件

在eclipse中，右键点击项目名，点击Export，选择Java >> JAR file

选择保存路径为 /usr/local/spark，文件名为 AverageAge.jar

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/3-3_20180410095039.039.png)

依次点击OK即可

#### 3. 提交运行
进入spark目录

```
# cd /usr/local/spark
```
执行以下命令提交任务

```
# ./bin/spark-submit --class org.apache.spark.AgerageAge \
    --master yarn \
    --deploy-mode client \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    AgerageAge.jar \
    file:///data/spark/input/input.txt
```

#### 4. 查看结果
本次实验使用deploy-mode client模式，执行结果可直接输出到控制台，查看控制台输出的结果。
![实验结果](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/3-4_20180410095044.044.png)