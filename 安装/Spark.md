# 下载地址
- http://archive.apache.org/dist/spark/ 

# Spark编译
- https://archive.apache.org/dist/spark/spark-3.0.1/spark-3.0.1.tgz
- http://spark.apache.org/docs/latest/building-spark.html#specifying-the-hadoop-version-and-enabling-yarn

## 1.idea打开点进pom文件修改maven源
- https://maven.aliyun.com/repository/central
## 2.gerneral and update...
## 3.build project
```
不能有报错,有报错就执行下第二步,还不成功可能是删了pom文件什么东西,不要删
```
## 4.Linux 打包
### (1).配置mvn,scala,java,修改maven源
### (2).上传前面编译好的整个目录
### (3).执行脚本
```shell script
sh /opt/spark-3.0.0/dev/make-distribution.sh --name hadoop3.2-without-hive --tgz "-Phadoop-3.2,yarn,hadoop-provided,orc-provided,parquet-provided"
```

# Spark安装
## 1.配置环境变量,安装scala
## 2.修改配置文件
### spark-env.sh
```shell script
export JAVA_HOME=/opt/software/jdk1.8.0_241
export LD_LIBRARY_PATH=/opt/software/hadoop-3.2.1/lib/native
export SPARK_LIBRARY_PATH=/opt/software/spark-3.0.1-bin-hadoop3.2/jars
export SPARK_CLASSPATH=/opt/software/spark-3.0.0-bin-hadoop3.2
export SCALA_HOME=/opt/software/scala-2.12.11
export HADOOP_HOME=/opt/software/hadoop-3.2.1
export HADOOP_CONF_DIR=/opt/software/hadoop-3.2.1/etc/hadoop
export SPARK_MASTER_IP=192.168.1.202
export SPARK_WORKER_INSTANCES=1
export SPARK_WORKER_MEMORY=4g
export SPARK_WORKER_CORES=4
export SPARK_EXECUTOR_CORES=4
export SPARK_EXECUTOR_MEMORY=4g
```
### slaves,(写自己会造成既是master,又是worker)
```shell script
192.168.1.201
192.168.1.203
```
## 3.启动Zookeeper,hadoop,spark
```shell script
zkServer.sh start 
start-all.sh 
sh /opt/software/spark-3.0.1-bin-hadoop3.2/sbin/start-all.sh

sh /opt/software/spark-3.0.0-bin-hadoop3.2/sbin/stop-all.sh
```
## 4.web界面
```http
http://192.168.1.201:8081/
```
## 5.测试
```shell script
cd /opt/software/spark-3.0.0-bin-hadoop3.2/
# 本地模式提交测试
./bin/run-example SparkPi 10

# 集群模式提交测试 交互方便调试
./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode client --driver-memory 1g --executor-memory 1g --executor-cores 1 --queue default examples/jars/spark-examples*.jar 3
./bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster --driver-memory 1g --executor-memory 1g --executor-cores 1 --queue default examples/jars/spark-examples*.jar 3

```
## 6.Spark整合进Hive
```http
https://www.bmc.com/blogs/using-spark-with-hive/
```
```shell script
cp /opt/software/apache-hive-3.1.2-bin/conf/hive-site.xml /opt/software/spark-3.0.0-bin-hadoop3.2/conf/
cp mysql-connector-java-8.0.18.jar /opt/software/spark-3.0.0-bin-hadoop3.2/jars/
```
### 开启Hive的MetaStore服务
```shell script
nohup hive --service metastore > metastore.log 2>&1 &
hive --service hiveserver2 --hiveconf hive.server2.thrift.port=10000
```
### 启动spark-shell
```shell script
sh /opt/software/spark-3.0.1-bin-hadoop3.2/bin/spark-shell --master spark://192.168.1.202:7077 --jar mysql-connector-java-8.0.21.jar
```
import org.apache.spark.sql.hive.HiveContext;
val hc = new HiveContext(sc);
hc.sql("show databases").show;
### 启动spark-sql
```shell script
spark-sql --master spark://192.168.1.201:7077 --executor-memory 1024m --total-executor-cores 2
```
## 7.一些报错信息
### Yarn application has already ended! It might have been killed or unable to launch application master.
```xml
<!-- yarn平台查看日志 -->
<!-- web端: http://192.168.1.201:8088 -->
<!-- 命令行: yarn logs -applicationId application_1565102051340_0004 -->
<!-- Current usage: 59.8 MB of 1 GB physical memory used; 2.2 GB of 2.1 GB virtual memory used. Killing container. -->
<!-- 在Hadoop的 yarn-site.xml 增加配置: 1条即可 -->
<property>
     <name>yarn.nodemanager.vmem-check-enabled</name>
     <value>false</value>
     <description>Whether virtual memory limits will be enforced for containers</description>
</property>
<property>
     <name>yarn.nodemanager.vmem-pmem-ratio</name>
     <value>4</value>
     <description>Ratio between virtual memory to physical memory when setting memory limits for containers</description>
</property>
```

