# Hadoop Window 单机版

- winutils下载: https://github.com/kontext-tech/winutils

## 1.配置环境变量 HADOOP_HOME,验证 hadoop version
> hadoop 3.3.1 已支持 java 11

# 2.修改配置文件

### core-site.xml
```xml
<configration>
    <!-- 这里配完进 50070端口查看是不是,可能显示为8020,那就用8020 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>localhost:2181</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:8020</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/D:/Mysoft/Hadoop-3.3.1/tmp</value>
        <final>true</final>
    </property>
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
</configration>
```

### hdfs-site.xml
```xml
<configration>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>127.0.0.1:50070</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.name.dir</name>
        <value>file:/D:/Mysoft/Hadoop-3.3.1/name</value>
    </property>
    <property>
        <name>dfs.data.dir</name>
        <value>file:/D:/Mysoft/Hadoop-3.3.1/data/</value>
    </property>
    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>
</configration>
```

### mapred-site.xml
```xml

<configration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configration>
```

### yarn-site.xml
```xml
<configration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>D:\Mysoft\Hadoop-3.3.1\tmp\nm-local-dir</value>
    </property>
</configration>
```

## 3.将 hadoop-winutils 相关文件放入 hadoop-3.3.1/bin目录下,并将 hadoop.dll 放入C盘,windows/system32目录下

## 4.管理员启动 cmd, 输入start-all.cmd , 注意看每个窗口是否有报错信息

## 5.验证,看是否都有一个节点

- localhost:8088
- localhost:50070

## 6.停止,stop-all.cmd