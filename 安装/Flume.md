# 下载 Flume
- http://archive.apache.org/dist/flume/

# 集群版
- https://zhangweisep.github.io/2018/10/26/Flume%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/

## 1.解压就能用
## 2.范例配置
### 获取端口数据
```sh
vim port.conf

agent.sources = s1
agent.channels = c1
agent.sinks = sk1
#设置Source为netcat端口为5678,使用的channel为c1
agent.sources.s1.type = netcat
agent.sources.s1.bind = localhost
agent.sources.s1.port = 5678
agent.sources.s1.channels = c1
#设置Sink为logger模式,使用的channel为c1
agent.sinks.sk1.type = logger
agent.sinks.sk1.channel = c1
#设置channel为capacity
agent.channels.c1.type = memory

# 运行这个配置
bin/flume-ng agent --name agent -f port.conf -Dflume.root.logger=INFO,console

# 新开一个窗口,随便输入观察刚才那个窗口
telnet localhost 5678
```
### 命令行
```sh
vim exec.conf

....
#设置Source为 exec
agent.sources.s1.type = exec
agent.sources.s1.command = tail -f /home/by/test.txt
agent.sources.s1.channels = c1
....

# 运行这个配置
bin/flume-ng agent --name agent -f exec.conf -Dflume.root.logger=INFO,console
```

### 获取磁盘文件的内容
```sh
vim spooldir.conf

....
#设置Source为 spooldir
agent.sources.s1.type = spooldir
agent.sources.s1.spoolDir = /home/by/test.txt
agent.sources.s1.channels = c1
....

运行这个配置
bin/flume-ng agent --name agent -f spooldir.conf -Dflume.root.logger=INFO,console
```

### 接收http的get和post请求
```sh
vim http.conf

....
#设置Source为 http
agent.sources.s1.type = http
agent.sources.s1.channels = c1
....

# 运行这个配置
bin/flume-ng agent --name agent -f http.conf -Dflume.root.logger=INFO,console

# 在另一个窗口
curl -XPOST localhost:5679 -d'[{"headers":"{"k1":"v1","k2":"v2"}","body":"hello world"}]'
```
### 监听avro端口,并从外部avro客户端接收数据
```sh
vim avro.conf

....
#设置Source为 avro
agent.sources.s1.type = avro
agent.sources.s1.bind = 0.0.0.0
agent.sources.s1.port = 5680
agent.sources.s1.channels = c1
....

# 运行这个配置
bin/flume-ng agent --name agent -f avro.conf -Dflume.root.logger=INFO,console

# 在另一个窗口
bin/flume-ng avro-client -H localhost -p 5680 -F /home/by/test.txt 
```
### 监听avro端口,并从外部端口接收数据
```sh
vim avrosink.conf

....
#设置Source为 netcat,监听5678端口
agent.sources.s1.type = netcat
agent.sources.s1.bind = 0.0.0.0
agent.sources.s1.port = 5678
agent.sources.s1.channels = c1
#设置Sink 
agent.sinks.sk1.type = avro
agent.sinks.sk1.hostname = localhost
agent.sinks.sk1.port = 5680
agent.sinks.sk1.channels = c1
.....

# 运行这个配置
bin/flume-ng agent --name agent -f avrosink.conf -Dflume.root.logger=INFO,console

# 在另一个窗口
telnet 5678
```