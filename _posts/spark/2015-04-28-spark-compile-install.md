---
layout: post
title: Spark 1.3.0 编译安装支持 hadoop2.2.0
category: Spark
tags: [spark, Hadoop 2.2.0]
---

------

### 源码下载编译
下载spark 1.3.0 source code:

```
https://spark.apache.org/downloads.html
```
编译：
修改pom.xml 文件:
指定要修改的hadoop 版本号和hbase 版本号:

```
    <profile>
      <id>hadoop-2.2</id>
      <properties>
        <hadoop.version>2.2.0</hadoop.version>
        <protobuf.version>2.5.0</protobuf.version>
        <hbase.version>0.98.9-hadoop2</hbase.version>
        <avro.mapred.classifier>hadoop2</avro.mapred.classifier>
        <codehaus.jackson.version>1.9.13</codehaus.jackson.version>
      </properties>
    </profile>
```

编译支持yarn, hadoop2.2.0, ganglia, hive并打包:

```
./make-distribution.sh --tgz --name 2.2.0 -Pyarn -Phadoop-2.2 -Pspark-ganglia-lgpl  -Phive
```

成功后会在当前目录下生成打包文件 ：

```
spark-1.3.0-bin-2.2.0.tgz
```

### 安装spark 1.3.0 

简要配置文件：
1、spark-env.sh

```
export JAVA_HOME=/home/hadoop/jvm/jdk1.7.0_51
export SCALA_HOME=/usr/share/scala
export HADOOP_HOME=/home/hadoop/hadoop2/hadoop-2.2.0
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
SPARK_EXECUTOR_INSTANCES=12
export SPARK_MASTER_IP=master1
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_WEBUI_PORT=7080
export SPARK_WORKER_WEBUI_PORT=7081
export SPARK_WORKER_MEMORY=4000m
SPARK_LOCAL_DIR="/data/spark/1.3/tmp"
SPARK_JAVA_OPTS="-Dspark.storage.blockManagerHeartBeatMs=60000 -Dspark.local.dir=$SPARK_LOCAL_DIR -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:$SPARK_HOME/logs/gc.log -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=65"
```

2、spark-defaults.conf

```
spark.master=spark://master1:7077
spark.eventLog.dir=/user/hadoop/spark/applicationHistory
spark.eventLog.enabled=true
spark.yarn.historyServer.address=http://master1:17080
```

3、slaves

```
slave1
slave2
slave3
slave4
slave5
```

4、集群启动

```
sbin/start-all.sh

sbin/start-history-server.sh [在spark 1.3.0版中有点问题]
```

### 示例运行

求Pi:

```
bin/spark-submit --class org.apache.spark.examples.SparkPi  --master spark://oddc2nn1:7077 lib/spark-examples-1.3.0-hadoop2.2.0.jar 1000
```

逻辑回归:

```
bin/spark-submit examples/src/main/python/logistic_regression.py README.md 2
```






