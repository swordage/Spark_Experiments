# sparkR als
本教程介绍利用Spark 和RStudio实现 kstest的功能


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
sparkR.session(appName = "SparkR-ML-kstest-example")

# $example on$
# Load training data
data <- data.frame(test = c(0.1, 0.15, 0.2, 0.3, 0.25, -1, -0.5))
df <- createDataFrame(data)
training <- df
test <- df

# Conduct the two-sided Kolmogorov-Smirnov (KS) test with spark.kstest
model <- spark.kstest(df, "test", "norm")

# Model summary
summary(model)
# $example off$

sparkR.session.stop()

```