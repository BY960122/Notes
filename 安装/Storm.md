# 下载 Storm
- http://archive.apache.org/dist/storm/

## 1.配置环境变量,安装 Zookeeper
```shell script
echo $STORM_HOME
```
## 2.配置配置文件
### storm.yaml
```shell script
# 注意: yaml文件格式,属性必须顶格写,缩进只能使用空格,一定不能使用制表符
ui.port: 8082
storm.zookeeper.servers:
  - "192.168.1.201"
  - "192.168.1.202"
  - "192.168.1.203"
		 
nimbus.seeds: ["192.168.1.201","192.168.1.202"]

# Nimbus 和 Supervisor 守护进程需要在本地硬盘的一个目录存储少量的状态(如jars,confs等)
storm.local.dir: /opt/software/apache-storm-2.2.0/data

supervisor.slots.ports:
 - 6700
 - 6701
 - 6702
 - 6703

scp storm.yaml 192.168.1.202:/opt/software/apache-storm-2.2.0/conf/
scp storm.yaml 192.168.1.203:/opt/software/apache-storm-2.2.0/conf/
```
## 3.启动
```shell script
# 如果python报错,把storm和storm.py最上面改成python3,以及 PYTHON="/usr/bin/env python3"
# 所有节点启动: logviewer
mkdir /opt/software/apache-storm-2.2.0/logs
touch /opt/software/apache-storm-2.2.0/logs/logviewer.log

nohup storm logviewer > /opt/software/apache-storm-2.2.0/logs/logviewer.log 2>&1 &
nohup storm supervisor > /opt/software/apache-storm-2.2.0/logs/supervisor.log 2>&1 &

# 所有主节点启动: nimbus
touch /opt/software/apache-storm-2.2.0/logs/nimbus.log
nohup storm nimbus > /opt/software/apache-storm-2.2.0/logs/nimbus.log 2>&1 &

# 选一个主节点启动: ui
touch /opt/software/apache-storm-2.2.0/logs/ui.log
nohup storm ui > /opt/software/apache-storm-2.2.0/logs/ui.log 2>&1 &

```
## 4.web 界面
- http://192.168.1.201:8082/

## 5.测试,运行本地jar包
```shell script
storm jar stormSum.jar by.StormExample.ClusterStormTopology
```
## 6.查看进程
```shell script
storm list
# 也可以在页面找kill按钮
storm kill ClusterStormTopology
```
