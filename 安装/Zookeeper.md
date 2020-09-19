# 下载 Zookeeper
- http://archive.apache.org/dist/zookeeper/

## 1.配置配置文件
### zoo.cfg
```sh
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/opt/software/apache-zookeeper-3.6.1-bin/zookeeper
clientPort=2181
server.1=192.168.1.201:2888:3888;2181
server.2=192.168.1.202:2888:3888;2181
server.3=192.168.1.203:2888:3888;2181

scp -r /opt/software/apache-zookeeper-3.6.1-bin/ 192.168.1.202:/opt/software/
scp -r /opt/software/apache-zookeeper-3.6.1-bin/ 192.168.1.203:/opt/software/

# 每台机器唯一的 id 
echo 1 > /opt/software/apache-zookeeper-3.6.1-bin/zookeeper/myid
```