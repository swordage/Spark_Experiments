# 实验目的
熟悉Spark RDD内存计算模型，了解SparkRDD基于内存的计算流程和MapReduce的区别，了解Spark WordCount运行原理。

# 实验原理
Spark是一个分布式计算框架，而RDD是其对分布式内存数据的抽象，可以认为RDD是就是Spark分布式算法的数据结构，而RDD之上的操作是Spark分布式算法的核心原语，由数据结构和原语设计上层算法。Spark最终会将算法翻译为DAG形式的工作流进行调度，并进行分布式任务的分发。

#### RDD的创建

- 从Hadoop文件系统输入创建
- 从父RDD转换得到新的RDD
- 通过parallelize或makeRDD将单机数据创建为分布式RDD

#### RDD的两种操作算子

- **转换(Transformation)**:Transformation操作会由一个RDD生成一个新的 RDD。Transformation操作是延迟计算的，也就是说从一个RDD转换生成另一个RDD的转换操作不是马上执行，需要等到Actions操作时，才真正开始运算。
- **行动(Action)**:Action算子会触发Spark提交作业（Job），并将数据输出到Spark系统。

#### RDD的内部属性
- 分区列表：通过分区列表可以找到一个RDD中包含的所有分区及其所在地址。
- 计算每个分片的函数：通过函数可以对每个数据块进行RDD需要进行的用户自定义函数运算。
- 对父RDD的依赖列表，为了能够回溯到父RDD，为容错等提供支持。
- 对key-value pair数据类型的RDD的分区器，控制分区策略和分区数。通过分区函数可以确定数据记录在各个分区和节点上的分配，减少分布不平衡。
- 每个数据分区的地址表（如HDFS上的数据块的地址）   

#### Spark RDD和 MapReduce 的区别
- MapReduce只有2个阶段，数据需要大量访问磁盘，数据来源相对单一, Spark RDD可以无数个阶段进行迭代计算，数据来源非常丰富，数据落地介质也非常丰富。
- Spark计算基于内存，MapReduce需要频繁操作磁盘IO

# 实验平台

- 操作系统： Ubuntu14.04
- Hadoop版本：2.7.4
- JDK版本：1.7
- Spark版本 2.0.2
- IDE：Eclipse


# 实验步骤

#### 一、启动HDFS和Yarn

```
# service ssh restart
# /usr/local/hadoop/sbin/start-dfs.sh
# /usr/local/hadoop/sbin/start-yarn.sh
```
在终端输入jps查看启动情况
```
# jps
```

#### 二、创建WordCount项目
1. 创建新的项目Java项目

打开eclipse， 依次点击 File >> New >> Project >> Java Project
项目名为 WordCount

添加新的包，包名为 org.apache.spark

在org.apache.spark包下创建WordCount类，操作完成后文件结构如下：

![项目文件结构](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/1_20180410073830.030.png)

2. 引入Spark相关Jar包

在项目名上点击右键，依次点击 Build Path >> Configure BuildPath >> Libraries  >> Add External JARs

在弹出的对话框中进入Spark jars目录 /usr/local/spark/jars，Ctrl+A选择全部的jar文件，点击OK。

3. 代码编写

```

package org.apache.spark;

import scala.Tuple2;

import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.sql.SparkSession;

import java.util.Arrays;
import java.util.List;
import java.util.regex.Pattern;

public final class WordCount {
  private static final Pattern SPACE = Pattern.compile(" ");

  public static void main(String[] args) throws Exception {

    if (args.length < 1) {
      System.err.println("Usage: WordCount <file>");
      System.exit(1);
    }

    SparkSession spark = SparkSession
      .builder()
      .appName("JavaWordCount")
      .getOrCreate();

    JavaRDD<String> lines = spark.read().textFile(args[0]).javaRDD();

    JavaRDD<String> words = lines.flatMap(s -> Arrays.asList(SPACE.split(s)).iterator());

    JavaPairRDD<String, Integer> ones = words.mapToPair(s -> new Tuple2<>(s, 1));

    JavaPairRDD<String, Integer> counts = ones.reduceByKey((i1, i2) -> i1 + i2);

    List<Tuple2<String, Integer>> output = counts.collect();
    for (Tuple2<?,?> tuple : output) {
      System.out.println(tuple._1() + ": " + tuple._2());
    }
    spark.stop();
  }
}
```


#### 二、提交 WordCount 项目到Spark
项目完成代码编写以后，需导出成jar文件然后提交到Spark。

1. 导出为jar文件

在eclipse中，右键点击项目名，点击Export，选择Java >> JAR file

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/2_20180410073834.034.png)

选择保存路径为 /usr/local/spark，文件名为 WordCount.jar

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/3_20180410073838.038.png)

依次点击OK即可

导出完成后，进入spark根目录
```
# cd /usr/local/spark
```

ls命令可以查看刚导出的WordCount.jar文件

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/4_20180410073841.041.png)

2. 编辑要统计的文件
在spark目录下执行：

```
# mkdir ./input
# gedit ./input/input.text
```
输入一些英文单词，如：
```
abc 123
hello world
hello spark
word count
```
保存

3. 提交到Spark运行

在spark目录下执行以下命令提交到spark运行

```
# ./bin/spark-submit --class org.apache.spark.WordCount \
    --master yarn \
    --deploy-mode cluster \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    WordCount.jar \
    file:///usr/local/spark/input/input.txt
```
最后一个参数为要统计的文件的路径

查看执行结果

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/5_20180410073844.044.png)

#### 三、查看运行结果

在浏览器输入 localhost:8088

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/6_20180410073847.047.png)

点击箭头所指位置

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/7_20180410073851.051.png)

点击logs

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/8_20180410073853.053.png)

查看最终结果

![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/9_20180410073856.056.png)