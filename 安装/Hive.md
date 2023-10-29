# 下载 Hive
- http://archive.apache.org/dist/hive/
- https://mirror.tuna.tsinghua.edu.cn/apache/hive/

## 1.配置环境变量
```sh
echo $HIVE_HOME
```
## 2.安装Hadoop
## 3.配置配置文件
### hive-env.sh
```sh
export JAVA_HOME=/opt/software/jdk1.8.0_291
export HIVE_HOME=/opt/software/apache-hive-3.1.1
export HADOOP_HOME=/opt/software/hadoop-3.2.1
```
### hive-site.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>

<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://192.168.1.6:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>By96o122</value>
    </property>
    <property>
        <name>datanucleus.connectionPoolingType</name>
        <value>dbcp</value>
        <description>
          Expects one of [bonecp, dbcp, hikaricp, none].
          Specify connection pool library for datanucleus
        </description>
    </property>
    <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
    </property>
    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>192.168.1.201</value>
    </property>
    <property>
        <name>hive.server2.enable.doAs</name>
        <value>true</value>
    </property>
    <property>
        <name>hive.server2.authentication</name>
        <value>NONE</value>
    </property>
    <!-- Hive整合 Tez -->
    <property>
        <name>hive.tez.container.size</name>
        <value>1024</value>
    </property>
    <!-- 允许 hive 递归读取子目录 -->
    <property>
        <name>mapred.input.dir.recursive</name>
        <value>true</value>
    </property>
    <!-- Hive整合 Spark -->
    <property>
        <name>spark.master</name>
        <value>spark://by202:7077</value>
    </property>
    <property>
        <name>spark.home</name>
        <value>/opt/software/spark-3.0.1-bin-hadoop3.2</value>
    </property>
    <!-- spark.yarn.jars         hdfs://192.168.1.202:9870/spark_jars/* -->
    <!-- # hdfs dfs -put /opt/software/spark-3.0.0-bin-hadoop3.2/jars/* /spark_jars/ -->
    <property>
        <name>spark.yarn.jars</name>
        <value>hdfs://192.168.1.202:9870/spark_jars/*</value>
    </property>
    <property>
        <name>hive.enable.spark.execution.engine</name>
        <value>true</value>
    </property>
    <!-- Hive整合 HBase -->
    <property> 
        <name>hive.zookeeper.quorum</name> 
        <value>192.168.1.201,192.168.1.202,192.168.1.203</value> 
    </property>
</configuration>
```
## 4.初始化 hive
```sh
sh /opt/software/apache-hive-3.1.1/bin/schematool -dbType mysql -initSchema
```
## 5.HIVE 整合 Spark
```http
由于hive的源码对应了spark,scala版本,自定义安装需手动编译hive,参考文档<windows无法编译,涉及bash,尝试用git bash无法成功>:
https://www.cnblogs.com/Darlin356230410/p/14879144.html
https://cwiki.apache.org/confluence/display/Hive/Hive+on+Spark%3A+Getting+Started
```
### hive-site.xml
```xml
<!-- 配置set hive.execution.engine=spark;的关联步骤,连接Spark集群 -->
<property>
        <name>spark.master</name>
        <value>spark://192.168.1.201:7077</value>
</property>
```
## 6.Hive 整合 Hbase
### hive-site.xml
```xml
<property> 
        <name>hive.zookeeper.quorum</name> 
        <value>192.168.1.201,192.168.1.202,192.168.1.203</value> 
</property>
```
```sh
scp /opt/software/apache-hive-3.1.1/conf/hive-site.xml 192.168.1.202:/opt/software/apache-hive-3.1.1/conf/
scp /opt/software/apache-hive-3.1.1/conf/hive-site.xml 192.168.1.203:/opt/software/apache-hive-3.1.1/conf/
scp /opt/software/apache-hive-3.1.1/conf/hive-site.xml 192.168.1.202:/opt/software/spark-3.0.0-bin-hadoop3.2/conf/
scp /opt/software/apache-hive-3.1.1/conf/hive-site.xml 192.168.1.203:/opt/software/spark-3.0.0-bin-hadoop3.2/conf/
```
```hiveql
-- 在hive创建外部表印射hbase
drop table if exists test_hbase;
create external table test_hbase(key string,name string, age int) 
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
with serdeproperties ("hbase.columns.mapping" = ":key,info:name,info:age") 
tblproperties ("hbase.table.name" = "test:test_hbase", "hbase.mapred.output.outputtable" = "test_hbase");
```
## 7.HIVE整合Tez
### (1).下载 protobuf-2.5.0

- https://www.cnblogs.com/bianqi/p/7229437.html
- https://github.com/protocolbuffers/protobuf/tags

### (2).下载 googletest1.5.0

- https://github.com/google/googletest

```sh
# 解压 更名放在 protobuf-2.5.0 目录下
mv googletest-release-1.5.0 gtest
```
### (3).编译,验证
```sh
yum install -y autoconf automake libtool
./autogen.sh
./configure
make install

protoc --version
```
### (4).编译 Tez
- https://blog.csdn.net/Hello_Java2018/article/details/106036782
- https://github.com/apache/tez/tree/master

```sh
chmod a+x /opt/software/apache-maven-3.6.3/bin/mvn
# 顶层.pom文件 配置阿里云源
# tez-ui这个模块,直接注释掉,费事费力容易报错
mvn clean package install -Dhadoop.version=3.2.1 -DskipTests -Dmaven.javadoc.skip=true
```
### (5) 解压 tez-0.10.0-minimal.tar.gz 到集群各个节点,配置环境变量
```sh
mkdir /opt/software/tez-0.10.0
tar -zxvf /opt/software/tez-0.10.0-minimal.tar.gz -C /opt/software/tez-0.10.0
```
### (6) 解压 tez-0.10.0.tar.gz 到集群各个节点,配置环境变量
```sh
echo $TEZ_HOME
hadoop fs -mkdir /tez 
hadoop fs -put /opt/software/tez-0.10.0.tar.gz /tez
```
### (7).新建 tez-site.xml ,放在 Hadoop,Hive配置目录下
```xml
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>tez.lib.uris</name>
        <value>${fs.defaultFS}/tez/tez-0.10.0,${fs.defaultFS}/tez/tez-0.10.0/lib</value>
    </property>
    <property>
        <name>tez.lib.uris.classpath</name>
        <value>${fs.defaultFS}/tez/tez-0.10.0,${fs.defaultFS}/tez/tez-0.10.0/lib</value>
    </property>
    <property>
        <name>tez.use.cluster.hadoop-libs</name>
        <value>true</value>
    </property>
    <property>
        <name>tez.am.resource.memory.mb</name>
        <value>2048</value>
    </property>
    <property>
        <name>tez.am.resource.cpu.vcores</name>
        <value>2</value>
    </property>
    <property>
        <name>tez.task.resource.memory.mb</name>
        <value>2048</value>
    </property>
    <property>
        <name>tez.task.resource.cpu.vcores</name>
        <value>2</value>
    </property>
</configuration>
```
```sh
cp /opt/software/hadoop-3.2.1/etc/hadoop/tez-site.xml /opt/software/apache-hive-3.1.1/conf/
scp /opt/software/hadoop-3.2.1/etc/hadoop/tez-site.xml 192.168.1.202:/opt/software/hadoop-3.2.1/etc/hadoop/
scp /opt/software/hadoop-3.2.1/etc/hadoop/tez-site.xml 192.168.1.203:/opt/software/hadoop-3.2.1/etc/hadoop/
scp /opt/software/hadoop-3.2.1/etc/hadoop/tez-site.xml 192.168.1.202:/opt/software/apache-hive-3.1.1/conf/
scp /opt/software/hadoop-3.2.1/etc/hadoop/tez-site.xml 192.168.1.203:/opt/software/apache-hive-3.1.1/conf/
```
### (8).配置配置
#### hive-site.xml
```xml
<property>
    <name>hive.tez.container.size</name>
    <value>2048</value>
</property>
```
#### hive-env.sh
```sh
export TEZ_HOME=/opt/software/tez-0.10.0
export TEZ_CONF_DIR=/opt/software/tez-0.10.0
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:${TEZ_CONF_DIR}:${TEZ_HOME}/*:${TEZ_HOME}/lib/*
export CLASSPATH=$CLASSPATH:${TEZ_CONF_DIR}:${TEZ_HOME}/*:${TEZ_HOME}/lib/*
```
#### hadoop-env.sh
```sh
export TEZ_HOME=/opt/software/tez-0.10.0
export TEZ_CONF_DIR=/opt/software/tez-0.10.0
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:${TEZ_CONF_DIR}:${TEZ_HOME}/*:${TEZ_HOME}/lib/*
export CLASSPATH=$CLASSPATH:${TEZ_CONF_DIR}:${TEZ_HOME}/*:${TEZ_HOME}/lib/*
```
### (9).测试
```sh
hadoop jar /opt/software/tez-0.10.0/tez-examples-0.10.0.jar orderedwordcount /data/test.txt /tez/output/
hdfs dfs -ls /tez/output/
```
## 8.启动 Hive
```sh
hive

hiveserver2 start
beeline -u jdbc:hive2://192.168.1.201:10000 -n root -p By921644606

set hive.execution.engine=spark;
set spark.driver.extraClassPath=/opt/software/apache-hive-3.1.1/lib/hive-exec-3.1.2.jar;
```
## 9.测试
```hiveql
create table test_hive (id int,name string);
insert into test_hive values (1,'a'),(2,'b'),(3,'c'),(4,'d'),(5,'e'),(6,'f'),(7,'g'),(8,'h'),(9,'i'),(10,'j'),(11,'k'),(12,'l'),(13,'m'),(14,'n'),
(15,'o'),(16,'p'),(17,'q'),(18,'r'),(19,'s'),(20,'t'),(21,'u'),(22,'v'),(23,'w'),(24,'x'),(25,'y'),(26,'z');
select row_number() over (partition by 1 order by id) from test_hive limit 7,7;
```
## 10.一些报错信息
### java.lang.NoSuchMethodError: com.google.common.base.Preconditions.checkArgument(ZLjava/lang/String;Ljava/lang/Object;)V
```sh
# guava 版本过低,用 Hadoop 的
rm -rf /opt/software/apache-hive-3.1.1/lib/guava-19.0.jar
```
### java.lang.NoClassDefFoundError：org.apache.hadoop.hive.conf.HiveVariableSource
```sh
# 注释 hadoop-env.sh 
# export HADOOP_CLASSPATH=${TEZ_JARS}/*:${TEZ_JARS}/lib/*:$HIVE_HOME/lib
```
### Hive Schema version 3.1.0 does not match metastore's schema version 1.2.0 Metastore is not upgraded or corrupt)
```txt
进入 MySQL元数据库 配置 version表对应值为 3.1.0
```
### Caused by: java.lang.ClassNotFoundException: org.apache.spark.SparkConf
```sh
scp /opt/software/spark-3.0.1-bin-hadoop3.2/jars/scala* /opt/software/apache-hive-3.1.1/lib/
scp /opt/software/spark-3.0.1-bin-hadoop3.2/jars/spark* /opt/software/apache-hive-3.1.1/lib/
```
### ERROR:Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
```sh
# 检查元数据库是否启动并可以连接
sh /opt/software/apache-hive-3.1.1/bin/schematool -initSchema -dbType mysql
```
