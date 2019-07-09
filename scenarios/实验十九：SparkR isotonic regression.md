# sparkR 
本教程介绍利用Spark 和RStudio实现isotonic regression的功能


### 1. 环境准备

启动hdfs、yarn

```
# service ssh restart
# cd /usr/local/hadoop
# sbin/start-dfs.sh
# sbin/start-yarn.sh

```
上传数据文件
```
# hadoop fs -mkdir -p data/mllib/
# cp /root/Desktop/myFile/sample_isotonic_regression_libsvm_data.txt .
# hadoop fs -put sample_isotonic_regression_libsvm_data.txt data/mllib/
```
### 2. 在RStudio中执行算法

打开桌面上的RStudio
输入以下代码：
```
Sys.setenv(SPARK_HOME = "/usr/local/spark")
library(SparkR, lib.loc = c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib")))


# Initialize SparkSession
sparkR.session(appName = "SparkR-ML-isoreg-example")

# $example on$
# Load training data
df <- read.df("data/mllib/sample_isotonic_regression_libsvm_data.txt", source = "libsvm")
training <- df
test <- df

# Fit an isotonic regression model with spark.isoreg
model <- spark.isoreg(training, label ~ features, isotonic = FALSE)

# Model summary
summary(model)

# Prediction
predictions <- predict(model, test)
head(predictions)
# $example off$

sparkR.session.stop()

```