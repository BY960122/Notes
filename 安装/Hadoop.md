### 1.关闭防火墙
```shell script
systemctl stop firewalld.service
systemctl disable firewalld.service
```
### 2.配置主机名  
```shell script
vim /etc/hosts
192.168.1.201 by201
192.168.1.211 by211
192.168.1.212 by212
```
### 3.配置免密码登录
```shell script
vim /etc/hosts

192.168.1.211 by211
192.168.1.212 by212
192.168.1.213 by213

ssh-keygen -t rsa  
cd /root/.ssh  id_rsa(私钥) id_rsa.pub(公钥)

ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.201
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.202
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.203
```
### 4.配置Java,Zookeeper
```shell script
cd /opt/software/apache-zookeeper-3.6.1-bin/conf/
cp zoo.cfg.bak zoo.cfg
vim zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/software/apache-zookeeper-3.6.1-bin/zookeeper
clientPort=2181
server.1=192.168.1.201:2888:3888;2181
server.2=192.168.1.202:2888:3888;2181
server.3=192.168.1.203:2888:3888;2181

scp -r /opt/software/apache-zookeeper-3.6.1-bin/ 192.168.1.201:/opt/software/
scp -r /opt/software/apache-zookeeper-3.6.1-bin/ 192.168.1.202:/opt/software/

# 每台机器唯一的 id 
echo 1 > /opt/software/apache-zookeeper-3.6.1-bin/zookeeper/myid
```
### 5.修改环境变量
```shell script
vim ~/.bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export JAVA_HOME=/opt/software/jdk1.8.0_241
export SCALA_HOME=/opt/software/scala-2.12.11
export PYTHON_HOME=/usr/local/python3
export MYSQL_HOME=/opt/software/mysql-8.0.21
export HADOOP_HOME=/opt/software/hadoop-3.2.1
export ZOOKEEPER_HOME=/opt/software/apache-zookeeper-3.6.1-bin
export HIVE_HOME=/opt/software/apache-hive-3.1.1-bin
export HBASE_HOME=/opt/software/hbase-2.3.1
export SPARK_HOME=/opt/software/spark-3.0.1-bin-hadoop3.2
export KAFKA_HOME=/opt/software/kafka_2.12-2.4.1
export ELASTICSEARCH_HOME=/opt/software/elasticsearch-7.6.1
export STORM_HOME=/opt/software/apache-storm-2.2.0
export FLINK_HOME=/opt/software/flink-1.11.1
export SQOOP_HOME=/opt/software/sqoop-1.4.7.bin__hadoop-2.6.0

# hadoop 3.0才需要配置
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_JOURNALNODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export HDFS_ZKFC_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root

PATH=$PATH:$HOME/bin:$JAVA_HOME/bin:$PYTHON_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$ZOOKEEPER_HOME/bin:$HIVE_HOME/bin:$HIVE_HOME/lib:$HBAS
E_HOME/bin:$HBASE_HOME/lib:$SCALA_HOME/bin:$SPARK_HOME/bin:$KAFKA_HOME/bin:$SQOOP_HOME/bin:$ELASTICSEARCH_HOME/bin:$STORM_HOME/bin:$FLINK_HOME/bin

export PATH

source ~/.bash_profile
```
### 6.修改配置文件
#### hadoop-env.sh 
```shell script
export JAVA_HOME=/opt/software/jdk1.8.0_241
export HADOOP_HOME=/opt/software/hadoop-3.2.1
export HIVE_HOME=/opt/software/apache-hive-3.1.1-bin
export TEZ_HOME=/opt/software/tez-0.10.1
```
#### hdfs-site.xml
```xml
<!--表示数据块的冗余度，默认：3-->
<property>
        <name>dfs.replication</name>
        <value>2</value>
</property>
<!--是否开启HDFS的权限检查，默认是true-->
<property>
        <name>dfs.permissions</name>
        <value>false</value>
</property>
<!--HDFS 监控界面-->
<property>
        <name>dfs.namenode.http-address</name>
        <value>192.168.1.201:50070</value>
</property>
```
#### core-site.xml
```xml
<!--配置NameNode地址,9000是RPC通信端口-->
<property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.1.201:9000</value>
</property>
<!--HDFS数据保存在Linux的哪个目录，默认值是Linux的tmp目录-->
<property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/software/hadoop-3.2.1/data</value>
</property>
<!-- 这样可以远程登录Hive -->
<property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
</property>
<property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
</property>
```
#### mapred-site.xml
```xml
<!--MR程序运行的框架-->
<property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
</property>
<!-- 设置jobhistoryserver 没有配置的话 history入口不可用 -->
<property>
        <name>mapreduce.jobhistory.address</name>
        <value>192.168.1.201:10020</value>
</property>
<!-- 配置web端口 -->
<property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>192.168.1.201:19888</value>
</property>
<!-- 配置正在运行中的日志在hdfs上的存放路径 -->
<property>
        <name>mapreduce.jobhistory.intermediate-done-dir</name>
        <value>/jobhistory/done_intermediate</value>
</property>
<!-- 配置运行过的日志存放在hdfs上的存放路径 -->
<property>
        <name>mapreduce.jobhistory.done-dir</name>
        <value>/jobhistory/done</value>
</property>
<!-- hadoop3.0需要加下面这些,hadoop2不需要 -->
<property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
<property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=$HADOOP_HOME</value>
</property>
```
#### yarn-site.xml
```xml
<!--Yarn的主节点RM的位置-->
<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>192.168.1.201</value>
</property>
<!--MapReduce运行方式：shuffle洗牌-->
<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
</property>
<!--开启日志聚合-->
<property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
</property>
<property>
        <name>yarn.log.server.url</name>
        <value>http://192.168.1.201:19888/jobhistory/logs/</value>
</property>
<!--Spark内存报错-->
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
#### slaves
```shell script
by211
by212
by213
```
### 7.格式化主节点NameNode
```shell script
hdfs namenode -format

scp -r /opt/software/hadoop-3.2.1/ 192.168.1.202:/opt/software/hadoop-3.2.1/
scp -r /opt/software/hadoop-3.2.1/ 192.168.1.203:/opt/software/hadoop-3.2.1/
```
### 8.主节点启动
```shell script
sh /opt/software/hadoop-3.2.1/sbin/start-all.sh
```
### 9.验证hadoop服务是否正常启动：打印HDFS的报告
```shell script
hdfs dfsadmin -report  
```
### 10.测试.随便写一个文本上传
```shell script
hdfs dfs -put /home/test.txt /data/
hadoop jar /opt/software/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /data/test.txt /out/test1
```
### 11.一些参考信息
#### HDFS管理界面 
```http
192.168.1.201:50070
```
#### SecondaryNamenode管理界面
```http
192.168.1.201:50090
```
#### YARN管理界面
```http
192.168.1.201:8088
```
