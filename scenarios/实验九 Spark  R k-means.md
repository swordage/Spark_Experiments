# sparkR als
本教程介绍利用Spark 和RStudio实现 k-means的功能


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
sparkR.session(appName = "SparkR-ML-kmeans-example")

# $example on$
# Fit a k-means model with spark.kmeans
t <- as.data.frame(Titanic)
training <- createDataFrame(t)
df_list <- randomSplit(training, c(7,3), 2)
kmeansDF <- df_list[[1]]
kmeansTestDF <- df_list[[2]]
kmeansModel <- spark.kmeans(kmeansDF, ~ Class + Sex + Age + Freq,
                            k = 3)

# Model summary
summary(kmeansModel)

# Get fitted result from the k-means model
head(fitted(kmeansModel))

# Prediction
kmeansPredictions <- predict(kmeansModel, kmeansTestDF)
head(kmeansPredictions)
# $example off$

sparkR.session.stop()


```