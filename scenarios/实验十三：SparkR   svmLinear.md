# sparkR  
本教程介绍利用Spark 和RStudio实现svmLinear 的功能


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
sparkR.session(appName = "SparkR-ML-svmLinear-example")

# $example on$
# load training data
t <- as.data.frame(Titanic)
training <- createDataFrame(t)

# fit Linear SVM model
model <- spark.svmLinear(training,  Survived ~ ., regParam = 0.01, maxIter = 10)

# Model summary
summary(model)

# Prediction
prediction <- predict(model, training)
showDF(prediction)
# $example off$
sparkR.session.stop()

```