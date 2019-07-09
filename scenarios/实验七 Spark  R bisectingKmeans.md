# sparkR  
本教程介绍利用Spark 和RStudio实现 bisectingKmeans的功能


### 1. 环境准备

启动hdfs、yarn

```
# service ssh restart
# cd /usr/local/hadoop
# sbin/start-dfs.sh
# sbin/start-yarn.sh

```

### 2. 在RStudio中执行算法
打开桌面上的RStudio
输入以下代码：
```
Sys.setenv(SPARK_HOME = "/usr/local/spark")
library(SparkR, lib.loc = c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib")))
# Initialize SparkSession
sparkR.session(appName = "SparkR-ML-bisectingKmeans-example")

# $example on$
t <- as.data.frame(Titanic)
training <- createDataFrame(t)

# Fit bisecting k-means model with four centers
model <- spark.bisectingKmeans(training, Class ~ Survived, k = 4)

# get fitted result from a bisecting k-means model
fitted.model <- fitted(model, "centers")

# Model summary
head(summary(fitted.model))

# fitted values on training data
fitted <- predict(model, training)
head(select(fitted, "Class", "prediction"))
# $example off$

sparkR.session.stop()


```