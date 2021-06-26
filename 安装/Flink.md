# 下载 Flink
- https://archive.apache.org/dist/flink

# 单集群模式
## 1.配置环境变量
```sh
echo $FLINK_HOME
```
## 2.修改配置文件
### config/flink-conf.yaml
```sh
jobmanager.rpc.address: 192.168.1.201

# 注意
jobmanager.memory.process.size: 1024m
taskmanager.memory.process.size: 1024m

rest.port: 8083
historyserver.web.port: 8084
```
### masters
```sh
192.168.1.201:8083
```
### workers
```sh
192.168.1.202
192.168.1.203
```
### zoo.cfg
```sh
cp /opt/software/apache-zookeeper-3.6.1-bin/conf/zoo.cfg /opt/software/flink-1.13.1/conf/

scp -r /opt/software/flink-1.13.1/conf/ 192.168.1.202:/opt/software/flink-1.13.1/conf/
scp -r /opt/software/flink-1.13.1/conf/ 192.168.1.203:/opt/software/flink-1.13.1/conf/

mv /opt/software/flink-1.13.1/lib/log4j-slf4j-impl-2.12.1.jar /opt/software/flink-1.13.1/
```
## 3.启动
```sh
zkServer.sh start

start-cluster.sh

stop-cluster.sh
```
## 4.web界面
- 192.168.1.201:8083

## 5.测试
```sh
# 注意此方法如果是本地文件,需要每个节点都有才行,如果是hdfs文件,请参考Flink-on-yarn 模式
flink run /opt/software/flink-1.13.1/examples/batch/WordCount.jar
```
## 6.一些报错信息
### Caused by: org.apache.flink.runtime.resourcemanager.exceptions.UnfulfillableSlotRequestException: Could not fulfill slot request..
```sh 
# config/flink-conf.yaml
# 这2个值可能大于了可用内存,导致没有 slot
jobmanager.memory.process.size: 1024m
taskmanager.memory.process.size: 1024m
```

# Flink on Yarn 模式
## 配置环境变量
```sh
export HADOOP_CLASSPATH=`hadoop classpath`
```
## Session-Cluster 模式: 在YARN上启动一个长期运行的Flink集群
```sh
./bin/yarn-session.sh -jm 1024m -tm 4096m
```
## Per-Job-Cluster 模式: 在YARN上运行Flink作业
```sh
flink run -m yarn-cluster -p 4 -yjm 1024m -ytm 4096m /opt/software/flink-1.13.1/examples/batch/WordCount.jar
```