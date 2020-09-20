# 下载 HBase,注意查看Hadoop兼容的版本
- http://hbase.apache.org/book.html#configuration
- http://archive.apache.org/dist/hbase/

## 1.配置环境变量
```sh
echo $HBASE_HOME
```
## 2.配置配置文件：
### hbase-env.sh
```sh
export JAVA_HOME=/opt/software/jdk1.8.0_261
export HBASE_MANAGES_ZK=false
```
### hbase-site.xml
```xml
<configuration>
	<!--配置HBase在HDFS上数据保存的路径-->
	<property>
		<name>hbase.root.dir</name>
		<value>hdfs://192.168.1.201:9000/hbase</value>
	</property>
	<!--配置HBase在HDFS上数据备份的个数-->
	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
	<!--配置HBase集群的模式-->
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>
	<!--解决文件系统不支持 hsync 报错而造成启动失败的问题,原因是二进制版本的 HBase 编译环境是 Hadoop2.x,而 Hadoop2.x 版本不支持 hsync-->
	<property>
		<name>hbase.unsafe.stream.capability.enforce</name>
		<value>false</value>
	</property>
	<!--配置zookeeper地址信息-->
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>192.168.1.201,192.168.1.202,192.168.1.203</value>
	</property>
	<property>
		<name>hbase.zookeeper.property.clientPort</name>
		<value>2181</value>
	</property>
	<!--配置zookeeper数据存放路径-->
	<property>
		<name>hbase.zookeeper.property.dataDir</name>
		<value>/home/myzk</value>
	</property>
	<!--允许HBase集群时间误差的最大值,单位是：毫秒-->
	<property>
		<name>hbase.master.maxclockskew</name>
		<value>180000</value>
	</property>
	<!--HADOOP_ORG.APACHE.HADOOP.HBASE.UTIL.GETJAVAPROPERTY_USER: bad substitution-->
	<property>
		<name>hbase.unsafe.stream.capability.enforce</name>
		<value>false</value>
	</property>
</configuration>
```
### regionservers
```sh
192.168.1.202
192.168.1.203
```
## 3.启动 Zookeeper,Hadoop,HBase
```sh
# Zookeeper
zkServer.sh start
zkServer.sh status
zkServer.sh stop

# Hadoop
start-all.sh

# Hbase
start-hbase.sh
	# regionServer
hbase-daemon.sh start regionserver
hbase-daemon.sh stop regionserver
	# master
hbase-daemon.sh start master
hbase-daemon.sh stop master
```
## 4.进入 HBase
```sh
hbase shell
```
## 5.web界面
```http
<!-- 老版本端口是:60010 -->
192.168.1.201:16010 
```
## 6.搭建HA模式
```sh
# 原理,多启动几个 Master
hbase-daemon.sh start hmaster
hbase-daemon.sh start master
```
## 7.一些报错信息
### org.apache.hadoop.hbase.PleaseHoldException: Master is initializing
```txt
检查hbase-site.xml的配置 hbase.root.dir 别写错
```