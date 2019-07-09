# sparkR als
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
sparkR.session(appName = "SparkR-ML-fpm-example")

# $example on$
# Load training data

df <- selectExpr(createDataFrame(data.frame(rawItems = c(
  "1,2,5", "1,2,3,5", "1,2"
))), "split(rawItems, ',') AS items")

fpm <- spark.fpGrowth(df, itemsCol="items", minSupport=0.5, minConfidence=0.6)

# Extracting frequent itemsets

spark.freqItemsets(fpm)

# Extracting association rules

spark.associationRules(fpm)

# Predict uses association rules to and combines possible consequents

predict(fpm, df)

# $example off$

sparkR.session.stop()



```