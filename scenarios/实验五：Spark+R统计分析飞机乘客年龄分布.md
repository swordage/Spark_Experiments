# 利用sparkR统计分析飞机乘客年龄分布
本教程介绍利用Spark 和RStudio实现“统计分析飞机乘客年龄分布” 的功能


### 1. 环境准备

启动hdfs、yarn

```
# service ssh restart
# cd /usr/local/hadoop
# sbin/start-dfs.sh
# sbin/start-yarn.sh

```
上传实验数据到hdfs上

```
# 首先上传文件到我的文件夹，进入容器后，在命名行中解压缩并上传
# cd /root/Desktop/myFile/
# unzip /root/Desktop/myFile/ml-100k.zip
# cd /usr/local/hadoop
# cp /root/Desktop/myFile/ml-100k/u.user .
# hadoop fs -mkdir /user/discover
# hadoop fs -put u.user /user/discover/
```

### 2. 在RStudio中进行数据准备
打开桌面上的RStudio
输入以下代码：
```
Sys.setenv(SPARK_HOME = "/usr/local/spark")
library(SparkR, lib.loc = c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib")))
sc <- sparkR.init(master = "local[*]", sparkEnvir = list(spark.driver.memory="2g"))
sqlContext <- sparkRSQL.init(sc)
userdata = read.df(sqlContext,'hdfs://localhost:9000/user/discover/u.user','csv',sep='|')
 names(userdata)<-c("id","age","gender","occupation","ZIPcode")
 head(userdata)
```
### 2. 在RStudio中进行数据分析和绘图
对年龄进行分组统计其数量,看看用户的年龄分布,并使用ggplot2作出条形图。继续回到RStudio会话中。

```
ages<-collect(summarize(groupBy(userdata,userdata$age),count = n(userdata$age)))
head(ages)
library(ggplot2)
ggplot(ages,aes(x=age,y=count))+geom_bar(stat='identity')
```

![](/pictures/20160120161840212_20180928075346.046.png)

可以看到,用户的年龄主要集中在20到30岁之间