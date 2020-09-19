# 下载 Hadoop
- http://archive.apache.org/dist/hadoop/

## 1.关闭防火墙
## 2.配置主机名 Hostname
## 3.配置免密码登录
## 4.配置Java,Zookeeper
## 5.配置环境变量
```sh
echo $HADOOP_HOME
```
## 6.配置配置文件
### hadoop-env.sh 
```sh
export JAVA_HOME=/opt/software/jdk1.8.0_241
export HADOOP_HOME=/opt/software/hadoop-3.2.1
export HIVE_HOME=/opt/software/apache-hive-3.1.1-bin
export TEZ_HOME=/opt/software/tez-0.10.1
```
### hdfs-site.xml
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
### core-site.xml
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
### mapred-site.xml
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
### yarn-site.xml
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
### slaves
```sh
by211
by212
by213
```
## 7.格式化主节点NameNode
```sh
hdfs namenode -format

scp -r /opt/software/hadoop-3.2.1/ 192.168.1.202:/opt/software/hadoop-3.2.1/
scp -r /opt/software/hadoop-3.2.1/ 192.168.1.203:/opt/software/hadoop-3.2.1/
```
## 8.主节点启动
```sh
sh /opt/software/hadoop-3.2.1/sbin/start-all.sh
```
## 9.验证hadoop服务是否正常启动：打印HDFS的报告
```sh
hdfs dfsadmin -report  
```
## 10.测试.随便写一个文本上传
```sh
hdfs dfs -put /home/test.txt /data/
hadoop jar /opt/software/hadoop-3.2.1/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.1.jar wordcount /data/test.txt /out/test1
```
## 11.一些参考信息
### HDFS管理界面 
- 192.168.1.201:50070
### SecondaryNamenode管理界面
- 192.168.1.201:50090
### YARN管理界面
- 192.168.1.201:8088
