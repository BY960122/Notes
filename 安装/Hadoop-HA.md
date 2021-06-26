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
export JAVA_HOME=/opt/software/jdk1.8.0_291
export HADOOP_HOME=/opt/software/hadoop-3.3.1
export HADOOP_CONF_DIR=/opt/software/hadoop-3.3.1/etc/hadoop
export HIVE_HOME=/opt/software/apache-hive-3.1.1-bin
export TEZ_HOME=/opt/software/tez-0.10.0
export TEZ_CONF_DIR=/opt/software/tez-0.10.0
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:${TEZ_CONF_DIR}:${TEZ_HOME}/*:${TEZ_HOME}/lib/*
export CLASSPATH=$CLASSPATH:${TEZ_CONF_DIR}:${TEZ_HOME}/*:${TEZ_HOME}/lib/*
```
### core-site.xml
```xml
<configuration>
    <!-- 指定hdfs的nameservice为ns1 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ns1</value>
    </property>
    <!-- 指定hadoop目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/software/hadoop-3.3.1/data/tmp</value>
    </property>
    <property> 
        <name>dfs.namenode.name.dir</name>
        <value>/opt/software/hadoop-3.3.1/data/name</value>
    </property>
    <property>
        <name>dfs.dataname.data.dir</name> 
        <value>/opt/software/hadoop-3.3.1/data</value>
    </property>
    <!-- 指定zookeeper地址 -->
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>192.168.1.201:2181,192.168.1.202:2181,192.168.1.203:2181</value>
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
    <!-- 当通过fs.defaultFSHadoop中的配置属性将该文件系统用于YARN的资源存储时，可能需要直接使用Hadoop文件系统 -->
    <property>
        <name>fs.alluxio.impl</name>
        <value>alluxio.hadoop.FileSystem</value>
    </property>
</configuration>
```
### hdfs-site.xml
```xml
<configuration> 
    <!--指定hdfs的nameservice为ns1，需要和core-site.xml中的保持一致 -->
    <property>
        <name>dfs.nameservices</name>
        <value>ns1</value>
    </property>
    <!-- ns1下面有两个NameNode，分别是nn1，nn2 -->
    <property>
        <name>dfs.ha.namenodes.ns1</name>
        <value>nn1,nn2</value>
    </property>
    <!-- nn1的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.ns1.nn1</name>
        <value>192.168.1.201:9000</value>
    </property>
    <!-- nn1的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.ns1.nn1</name>
        <value>192.168.1.201:50070</value>
    </property>
    <!-- nn2的RPC通信地址 -->
    <property>
        <name>dfs.namenode.rpc-address.ns1.nn2</name>
        <value>192.168.1.202:9000</value>
    </property>
    <!-- nn2的http通信地址 -->
    <property>
        <name>dfs.namenode.http-address.ns1.nn2</name>
        <value>192.168.1.202:50070</value>
    </property>
    <!-- 指定NameNode的日志在JournalNode上的存放位置 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://192.168.1.201:8485;192.168.1.202:8485/ns1</value>
    </property>
    <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/opt/software/hadoop-3.3.1/journal</value>
    </property>
    <!-- 开启NameNode失败自动切换 -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
    <!-- 配置失败自动切换实现方式 -->
    <property>
        <name>dfs.client.failover.proxy.provider.ns1</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    <!-- 配置隔离机制方法，多个机制用换行分割，即每个机制暂用一行-->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>
                sshfence
                shell(/bin/true)
        </value>
    </property>
    <!-- 使用sshfence隔离机制时需要ssh免登陆 -->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/root/.ssh/id_rsa</value>
    </property>
    <!-- 配置sshfence隔离机制超时时间 -->
    <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
    </property>
</configuration>
```
### mapred-site.xml
```xml
<configuration>
    <!-- 设置名称 -->
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
</configuration>
```
### yarn-site.xml
```xml
<configuration>
    <!-- 开启RM高可靠 -->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    <!-- 指定RM的cluster id -->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>yrc</value>
    </property>
    <!-- 指定RM的名字 -->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    <!-- 分别指定RM的地址 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>192.168.1.201</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>192.168.1.202</value>
    </property>
    <!-- MR报错: org.apache.hadoop.yarn.exceptions.YarnRuntimeException: java.lang.NullPointerException -->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>192.168.1.201:8088</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>192.168.1.202:8088</value>
    </property>
    <!-- 指定zk集群地址 -->
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>192.168.1.201:2181,192.168.1.202:2181,192.168.1.203:2181</value>
    </property>
    <!--MapReduce运行方式：shuffle洗牌-->
    <!-- cp /opt/software/spark-3.0.1-bin-hadoop3.2/yarn/spark-3.0.1-yarn-shuffle.jar /opt/software/hadoop-3.3.1/share/hadoop/yarn/lib/ -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle,spark_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.spark_shuffle.class</name>
        <value>org.apache.spark.network.yarn.YarnShuffleService</value>
    </property>
    <!-- <property>
            <name>yarn.nodemanager.aux-services.tez_shuffle.class</name>
            <value>org.apache.tez.auxservices.ShuffleHandler</value>
    </property> -->
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
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>4096</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>4</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>8192</value>
    </property>
    <property>
        <name>yarn.scheduler.maximum-allocation-vcores</name>
        <value>8</value>
    </property>
</configuration>
```
## 7.启动Zookeeper集群
```sh
zkServer.sh start
zkServer.sh status
```
## 8.在两个主节点上启动journalnode
```sh
# 做这一步之前先删除所有节点/opt/software/hadoop-3.3.1/data目录,然后再创建一个文件夹,确保一致
rm -rf /opt/software/hadoop-3.3.1/journal
rm -rf /opt/software/hadoop-3.3.1/data
    # hadoop-2.*
hadoop-daemon.sh start journalnode
    # hadoop-3.*
hdfs --daemon start journalnode
```
## 9.在一台主节点格式化 HDFS
```sh
hdfs namenode -format
# INFO common.Storage: Storage directory /opt/software/hadoop-3.3.1/data/dfs/name has been successfully formatted.
scp -r /opt/software/hadoop-3.3.1/data 192.168.1.202:/opt/software/hadoop-3.3.1/
scp -r /opt/software/hadoop-3.3.1/data 192.168.1.203:/opt/software/hadoop-3.3.1/
```
## 10.格式化 Zookeeper
```sh
hdfs zkfc -formatZK
```
## 11.启动集群
```sh
start-all.sh
# 注意看2个节点的resourcemanager都启动没
    # hadoop 2
yarn-daemon.sh start resourcemanager
    # hadoop 3
yarn --daemon start resourcemanager
```
## 12.常用命令
```sh
# 启动namenode
hdfs --daemon start namenode

# 启动 resourcemanager 和 nodemanager
start-yarn.sh

# 查看 namenode 状态
hdfs haadmin -getServiceState nn2

# 启动历史日志
yarn --daemon start historyserver
mapred --daemon start historyserver

# 查看作业列表
yarn application -list 

# kill 作业
yarn application -kill application_1600865572447_0003
```
## 13.一些报错信息
### Call From by201/192.168.1.201 to by201:8485 failed on connection exception
```sh
hdfs --daemon start journalnode
```
### hadoop启动报错: java.lang.NoClassDefFoundError:/org/apache/hadoop/yarn/server/timelineCollectorManager
```txt
将hadoop-yarn-server-timelineservice-3.3.1.jar,放到share\hadoop\yarn\lib下
```