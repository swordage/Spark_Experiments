# sparkR
本教程介绍利用Spark 和RStudio实现  AFT  survival regression的功能


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
sparkR.session(appName = "SparkR-ML-survreg-example")

# $example on$
# Use the ovarian dataset available in R survival package
library(survival)

# Fit an accelerated failure time (AFT) survival regression model with spark.survreg
ovarianDF <- suppressWarnings(createDataFrame(ovarian))
aftDF <- ovarianDF
aftTestDF <- ovarianDF
aftModel <- spark.survreg(aftDF, Surv(futime, fustat) ~ ecog_ps + rx)

# Model summary
summary(aftModel)

# Prediction
aftPredictions <- predict(aftModel, aftTestDF)
head(aftPredictions)
# $example off$

sparkR.session.stop()


```