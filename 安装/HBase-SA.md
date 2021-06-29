# HBase Window 单机版

## 1.安装好 windows hadoop 单机版,配置环境变量 HBASE_HOME
> 2.4.1 只支持到 hadoop 3.2.1
> 2.4.1 已支持 java 11

## 2.修改配置文件

### hbase-env.cmd
```sh
#set JAVA_HOME=C:\Software\Java\jdk-1.8.0_291
set JAVA_HOME=C:\Software\Java\jdk-11.0.11
set HBASE_MANAGES_ZK=true
```

### hbase-site.xml
```xml
<configuration>
    <!--    这个端口进50070页面确认一下-->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://localhost:8020/hbase</value>
    </property>
    <property>
        <name>hbase.tmp.dir</name>
        <value>D:\Mysoft\hbase-2.4.1\tmp</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>localhost</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.clientPort</name>
        <value>2181</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>D:\Mysoft\hbase-2.4.1\zoo</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>false</value>
    </property>
    <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
    </property>
</configuration>
```

## 3.管理员启动 cmd, 启动 hbase
```sh
start-hbase.cmd
stop-hbase.cmd
hbase shell
```