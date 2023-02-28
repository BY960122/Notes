# 下载 Zookeeper
- http://archive.apache.org/dist/zookeeper/

## 1.配置配置文件
### zoo.cfg
```sh
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/software/zookeeper-3.6.1/zookeeper
clientPort=2181
server.201=192.168.1.201:2888:3888;2181
server.202=192.168.1.202:2888:3888;2181
server.203=192.168.1.203:2888:3888;2181

mv /opt/software/apache-zookeeper-3.6.1-bin /opt/software/zookeeper-3.6.1

scp -r /opt/software/zookeeper-3.6.1/conf/* 192.168.1.202:/opt/software/zookeeper-3.6.1/conf/
scp -r /opt/software/zookeeper-3.6.1/conf/* 192.168.1.203:/opt/software/zookeeper-3.6.1/conf/

# 每台机器唯一的 id,写在数据目录下
echo 201 > /opt/software/zookeeper-3.6.1/zookeeper/myid
echo 202 > /opt/software/zookeeper-3.6.1/zookeeper/myid
echo 203 > /opt/software/zookeeper-3.6.1/zookeeper/myid
```