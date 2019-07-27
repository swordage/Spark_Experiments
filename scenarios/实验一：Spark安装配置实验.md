# 实验目的
熟悉Spark的安装与配置操作

# 实验原理

Apache Spark是一个开源集群运算框架。Spark在存储器内运行程序的运算速度能做到比Hadoop MapReduce的运算速度快上100倍，即便是运行程序于硬盘时，Spark也能快上10倍速度。使用Spark需要搭配集群管理员和分布式存储系统。Spark支持独立模式（本地Spark集群）、Hadoop YARN或Apache Mesos的集群管理。

#### Spark构成要素

**Spark核心和弹性分布式数据集（RDDs）**
Spark核心是整个项目的基础，提供了分布式任务调度，调度和基本的I／O功能。

**Spark SQL**
Spark SQL在Spark核心上带出一种名为SchemaRDD的数据抽象化概念，提供结构化和半结构化数据相关的支持。

**Spark Streaming**
Spark Streaming充分利用Spark核心的快速调度能力来运行流分析。它截取小批量的数据并对之运行RDD转换。这种设计使流分析可在同一个引擎内使用同一组为批量分析编写而撰写的应用程序代码。

**MLlib**
MLlib是Spark上分布式机器学习框架。Spark分布式存储器式的架构比Hadoop磁盘式的Apache Mahout快上10倍，扩充性甚至比Vowpal Wabbit要好。MLlib可使用许多常见的机器学习和统计算法，简化大规模机器学习时间，其中包括：
- 汇总统计、相关性、分层抽样、假设检定、随机数据生成
- 分类与回归：支持向量机、回归、线性回归、决策树、朴素贝叶斯
- 协同过滤：ALS
- 分群：k-平均算法
- 维度缩减：奇异值分解（SVD），主成分分析（PCA）
- 特征提取和转换：TF-IDF、Word2Vec、StandardScaler
- 最优化：随机梯度下降法（SGD）、L-BFGS

**GraphX**
GraphX是Spark上的分布式图形处理框架。它提供了一组API，可用于表达图表计算并可以模拟Pregel抽象化。GraphX还对这种抽象化提供了优化运行。


# 实验步骤
Spark支持多种运行模式，包括独立模式（本地Spark集群）、Hadoop YARN或Apache Mesos的集群管理，本次及以后的实验均采用Running Spark on YARN模式。
#### 一、安装配置Hadoop环境

1. 解压安装

1）复制 /data/hadoop-2.8.2.tar.gz 到/usr/hadoop目录下，

`# cp /data/hadoop-2.8.2.tar.gz  /usr/local`

然后解压

`# tar -xzvf hadoop-2.8.2.tar.gz `

解压后目录为：/usr/local/hadoop-2.8.2

重命名文件

 `# mv /usr/local/hadoop-2.8.2 /usr/local/hadoop`

2）在/usr/hadoop/目录下，建立tmp、hdfs/name、hdfs/data目录，执行如下命令

```
#mkdir /usr/hadoop
#mkdir /usr/hadoop/tmp
#mkdir /usr/hadoop/hdfs
#mkdir /usr/hadoop/hdfs/data
#mkdir /usr/hadoop/hdfs/name
```

3）设置环境变量，`#vi ~/.bashrc`

```
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native export
PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```

4）使环境变量生效，`# $source ~/.bashrc`

3.Hadoop配置

进入$HADOOP\_HOME/etc/hadoop目录，配置 hadoop-env.sh等。涉及的配置文件如下：

```
$HADOOP_HOME/etc/hadoop/hadoop-env.sh
$HADOOP_HOME/etc/hadoop/yarn-env.sh
$HADOOP_HOME/etc/hadoop/core-site.xml
$HADOOP_HOME/etc/hadoop/hdfs-site.xml
$HADOOP_HOME/etc/hadoop/mapred-site.xml
$HADOOP_HOME/etc/hadoop/yarn-site.xml
```

1）配置hadoop-env.sh：`# vim hadoop-env.sh `

```
export JAVA_HOME=${JAVA_HOME}
```

2）配置yarn-env.sh：

```
export JAVA_HOME=${JAVA_HOME}
```

3）配置core-site.xml，添加如下配置：

```
<configuration>
 <property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
</property>

<property>
    <name>hadoop.tmp.dir</name>
    <value>/usr/hadoop/tmp</value>
</property>
</configuration>
```

4）配置hdfs-site.xml，添加如下配置：

```
<configuration>
<!—hdfs-site.xml-->
<property>
    <name>dfs.name.dir</name>
    <value>/usr/hadoop/hdfs/name</value>
</property>

<property>
    <name>dfs.data.dir</name>
    <value>/usr/hadoop/hdfs/data</value>
</property>

<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>
</configuration>
```

5）配置mapred-site.xml，

`# cp mapred-site.xml.template mapred-site.xml`

添加如下配置：
```
<configuration>
<property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
   </property>
</configuration>
```

6）配置yarn-site.xml

添加如下配置：

```
<configuration>
<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
</property>
</configuration>
```

4.Hadoop启动

1）进入/usr/local/hadoop/bin目录，格式化namenode

`# ./hdfs namenode -format`

2）进入/usr/local/hadoop/sbin目录，启动NameNode 和 DataNode 守护进程

`# ./start-dfs.sh`

3）启动ResourceManager 和 NodeManager 守护进程

`# ./start-yarn.sh`

5.启动验证

执行jps命令:`# jps`

有如下进程，说明Hadoop正常启动

![](/pictures/1.png)


#### 二、安装配置Scala环境
Scala压缩包已下载到本机的 /data/spark 目录下，直接解压即可。解压完成后，重命名文件夹为scala
```
# mkdir /usr/spark
# tar zxvf /data/spark/scala-2.11.0.tgz -C /usr/spark/
# cd /usr/spark
# mv scala-2.11.0 scala
```
配置环境变量
```
# gedit /etc/profile
```
在文件末尾加入已下内容
```
export SCALA_HOME=/usr/spark/scala
export PATH=$PATH:$SCALA_HOME/bin
```
执行以下命令使环境变量生效
```
source /etc/profile
```
测试
```
scala -version
```
可以看到版本为 2.11.0

#### 三、下载Spark并解压
Spark压缩包已下载到本机的 /data/spark 目录下，直接解压即可。解压完成后，重命名文件夹为spark

```
# tar -zxvf /data/spark/spark-2.0.2-bin-hadoop2.7.tgz -C /usr/spark
# cd /usr/spark
# mv spark-2.0.2-bin-hadoop2.7 spark
```

#### 四、配置Spark

进入spark配置文件根目录

```
# cd /usr/spark/spark/conf
```

1. 配置 spark-env.sh
```
# cp spark-env.sh.template spark-env.sh
# gedit spark-env.sh
```

在spark-env.sh末尾添加以下内容：

```
export SPARK_HOME=/usr/spark/spark
export SCALA_HOME=/esr/spark/scala
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export HADOOP_HOME=/usr/spark/hadoop
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$SCALA_HOME/bin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$YARN_HOME/etc/hadoop
export SPARK_MASTER_IP=localhost
SPARK_LOCAL_DIRS=/usr/spark/spark
SPARK_DRIVER_MEMORY=1G
export SPARK_LIBARY_PATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$HADOOP_HOME/lib/native
```
2. 添加slaves配置文件

```
cp slaves.template slaves
```

#### 五、启动Hadoop和Yarn
```
# service ssh restart
# cd /usr/local/hadoop
# sbin/start-dfs.sh
# sbin/start-yarn.sh
```
检查启动情况
```
# jps
```

#### 六、测试Spark

1. 进入Spark目录
```
# cd /usr/spark/spark
```

2. 提交测试用例
计算Pi的值，迭代次数为10次
```
# ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
      --master yarn \
      --deploy-mode cluster \
      --driver-memory 4g \
      --executor-memory 2g \
      --executor-cores 1 \
      examples/jars/spark-examples*.jar \
      10
```

![输入命令](/pictures/启动命令_20180410050850.050.png)

3. 执行结果如下图所示，执行成功！

![执行结果](/pictures/执行结果_20180410050853.053.png)

本次执行的任务结果保存在hdfs中，下面查看保存在HDFS中的计算出的Pi值

4. 在浏览器输入 **localhsot:8088**，结果如下：

![](/pictures/webui_20180410051117.017.png)

5. 点击上图箭头所示位置，进入任务详细页面

![](/pictures/logs_20180410050838.038.png)

6. 点击logs，查看执行日志

![](/pictures/logs-1_20180410050842.042.png)

7. 点击stdout，查看输出结果

![](/pictures/result_20180410050844.044.png)

可以看到计算的Pi值，改变执行命令的最后一个参数，能够获得不同精确度的Pi值。