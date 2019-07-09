# sparkR 
本教程介绍利用Spark 和RStudio实现 generalized linear model  的功能


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
# cp /root/Desktop/myFile/sample_multiclass_classification_data.txt .
# hadoop fs -put sample_multiclass_classification_data.txt data/mllib/
```
### 2. 在RStudio中执行算法

打开桌面上的RStudio
输入以下代码：
```
Sys.setenv(SPARK_HOME = "/usr/local/spark")
library(SparkR, lib.loc = c(file.path(Sys.getenv("SPARK_HOME"), "R", "lib")))


# Initialize SparkSession
sparkR.session(appName = "SparkR-ML-glm-example")

# $example on$
training <- read.df("data/mllib/sample_multiclass_classification_data.txt", source = "libsvm")
# Fit a generalized linear model of family "gaussian" with spark.glm
df_list <- randomSplit(training, c(7, 3), 2)
gaussianDF <- df_list[[1]]
gaussianTestDF <- df_list[[2]]
gaussianGLM <- spark.glm(gaussianDF, label ~ features, family = "gaussian")

# Model summary
summary(gaussianGLM)

# Prediction
gaussianPredictions <- predict(gaussianGLM, gaussianTestDF)
head(gaussianPredictions)

# Fit a generalized linear model with glm (R-compliant)
gaussianGLM2 <- glm(label ~ features, gaussianDF, family = "gaussian")
summary(gaussianGLM2)

# Fit a generalized linear model of family "binomial" with spark.glm
training2 <- read.df("data/mllib/sample_multiclass_classification_data.txt", source = "libsvm")
training2 <- transform(training2, label = cast(training2$label > 1, "integer"))
df_list2 <- randomSplit(training2, c(7, 3), 2)
binomialDF <- df_list2[[1]]
binomialTestDF <- df_list2[[2]]
binomialGLM <- spark.glm(binomialDF, label ~ features, family = "binomial")

# Model summary
summary(binomialGLM)

# Prediction
binomialPredictions <- predict(binomialGLM, binomialTestDF)
head(binomialPredictions)

# Fit a generalized linear model of family "tweedie" with spark.glm
training3 <- read.df("data/mllib/sample_multiclass_classification_data.txt", source = "libsvm")
tweedieDF <- transform(training3, label = training3$label * exp(randn(10)))
tweedieGLM <- spark.glm(tweedieDF, label ~ features, family = "tweedie",
                        var.power = 1.2, link.power = 0)

# Model summary
summary(tweedieGLM)
# $example off$

sparkR.session.stop()


```