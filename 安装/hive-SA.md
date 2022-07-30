# Hive Windows 单机版

## 1.安装好 windows hadoop单机版,配置环境变量 HIVE_HOME
> 最新版不支持 java 11

## 2.新建hdfs目录
```sh
hdfs dfs -mkdir /user/hive/warehouse
hdfs dfs -mkdir /tmp
hdfs dfs -chmod -R 777 /tmp
hdfs dfs -chmod -R 777 /user/hive/warehouse
```

## 3.将 mysql-connector-java-8.0.x.jar 放入 lib目录下

## 4.修改配置文件
### hive-env.sh
```sh
export HADOOP_HOME=D:\Mysoft\Hadoop-3.3.1
export HIVE_CONF_DIR=D:\Mysoft\apache-hive-3.1.1\conf
export HIVE_AUX_JARS_PATH=D:\Mysoft\apache-hive-3.1.1\lib
```

### hive-site.xml, 如果是hive-default.xml,请改过来
```xml
<configuration>
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    <property>
        <name>hive.execution.engine</name>
        <value>mr</value>
    </property>
    <property>
        <name>hive.exec.scratchdir</name>
        <value>/tmp/hive</value>
    </property>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://192.168.1.6:3306/hive_sa?createDatabaseIfNotExist=true&amp;useSSL=false</value>
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
        <name>hive.exec.local.scratchdir</name>
        <value>D:\Mysoft\apache-hive-3.1.1\data\operationDir</value>
    </property>
    <property>
        <name>hive.downloaded.resources.dir</name>
        <value>D:\Mysoft\apache-hive-3.1.1\data\resourcesDir</value>
    </property>
    <property>
        <name>hive.querylog.location</name>
        <value>D:\Mysoft\apache-hive-3.1.1\data\querylogDir</value>
    </property>
    <property>
        <name>hive.server2.logging.operation.log.location</name>
        <value>D:\Mysoft\apache-hive-3.1.1\data\operationDir</value>
    </property>
</configuration>
```

### hive-exec-log4j.properties

### hive-log4j.properties

## 5.初始化 元数据库
```sh
hive.cmd --service schematool -dbType mysql -initSchema
```

## 6.管理员启动 cmd, 连接 hive
```sh
# 普通连接
hive.cmd
# 启动 hiveserver2 服务
hive.cmd --service hiveserver2
beeline.cmd -u jdbc:hive2://127.0.0.1:10000 -n root
```