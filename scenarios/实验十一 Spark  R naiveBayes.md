# sparkR  
本教程介绍利用Spark 和RStudio实现 naiveBayes的功能


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
sparkR.session(appName = "SparkR-ML-naiveBayes-example")

# $example on$
# Fit a Bernoulli naive Bayes model with spark.naiveBayes
titanic <- as.data.frame(Titanic)
titanicDF <- createDataFrame(titanic[titanic$Freq > 0, -5])
nbDF <- titanicDF
nbTestDF <- titanicDF
nbModel <- spark.naiveBayes(nbDF, Survived ~ Class + Sex + Age)

# Model summary
summary(nbModel)

# Prediction
nbPredictions <- predict(nbModel, nbTestDF)
head(nbPredictions)
# $example off$

sparkR.session.stop()

```