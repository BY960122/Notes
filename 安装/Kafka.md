# 下载 Kafka
- https://kafka.apache.org/downloads

## 前置动作
```sh
mkdir -p /opt/logs/kafka-logs
```

## 1.修改/config目录下的配置文件,server.properties
```shell
# broker id唯一,对应修改
broker.id=201
# 对应修改
listeners=PLAINTEXT://192.168.1.201:9092
log.dirs=/opt/logs/kafka-logs
num.partitions=3
zookeeper.connect=192.168.1.201:2181,192.168.1.202:2181,192.168.1.203:2181
```

## 2.修改/config目录下的配置文件,producer.properties
```shell
bootstrap.servers=192.168.1.201:9092,192.168.1.202:9092,192.168.1.203:9092
```
## 3.修改/config目录下的配置文件,zookeeper.properties
```shell
dataDir=/opt/software/zookeeper-3.6.1/zookeeper
```

### 可能出现的报错
## 1./opt/software/kafka_2.12-2.8.1/bin/kafka-run-class.sh: line 330: exec: java: not found
```shell
# 手动设置这一行的java路径
JAVA="/opt/software/jdk1.8.0_291/bin/java"

scp /opt/software/kafka_2.12-2.8.1/bin/kafka-run-class.sh 192.168.1.202:/opt/software/kafka_2.12-2.8.1/bin/
scp /opt/software/kafka_2.12-2.8.1/bin/kafka-run-class.sh 192.168.1.203:/opt/software/kafka_2.12-2.8.1/bin/
```
