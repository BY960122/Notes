# Windows安装
## 1.下载地址
```http
https://github.com/MSOpenTech/redis/releases
```
## 2.进入安装目录.注册成服务
```sh
redis-server.exe --service-install redis.windows.conf 
```
## 3.启动
```sh
redis-server.exe redis.windows.conf

net start redis
```
## 4.命令行进入
```sh
redis-cli.exe -h 127.0.0.1 -p 6379
```

# Linux安装
## 1.安装 Python
## 2.编译
```sh
sudo yum install -y tcl,ruby
cd redis-5.0.4
make && make install
```
## 3.修改配置文件,每个节点2个,不同的节点只需要修改ip
```sh
# redis_backup.conf
# redis.conf
# 修改以下9个地方

bind 192.168.0.212
port 6379
maxmemory 536870912
dbfilename dump.rdb
pidfile /var/run/redis_6379.pid
appendfilename "appendonly.aof"
requirepass xf_server
cluster-enabled yes
cluster-config-file nodes-6379.conf

bind 192.168.0.212
port 6380
maxmemory 536870912
dbfilename dump_back.rdb
pidfile /var/run/redis_6380.pid
appendfilename "appendonly_back.aof"
requirepass xf_server
cluster-enabled yes
cluster-config-file nodes-6380.conf
```
## 4.启动
```sh
./src/redis-server /opt/software/redis-5.0.8/redis.conf > log.txt 2>&1 &
./src/redis-server /opt/software/redis-5.0.8/redis_backup.conf > log_backup.txt 2>&1 &
```
## 5.检验是否成功启动
```sh
ps aux | grep redis | grep -v grep

# 如果要删除redis进程
ps aux | grep -w redis-server | grep -v grep | awk '{print $2}' | xargs kill
```
## 6.连接已启动的redis实例
```sh
# 客户端模式连接
./src/redis-cli -h 192.168.1.201 -p 6379 -a cqdsjb
auth "cqdsjb"

# 集群模式连接
./src/redis-cli -h 192.168.1.201 -p 6379 -a xf_server -c 
```
## 7.创建redis集群
### 须满足条件一: 6个redis实例均正常启动,并且可连接
### 须满足条件二: 所有redis均为空
```sh
./src/redis-cli -a xf_server --cluster create 192.168.1.201:6379 192.168.1.201:6380 192.168.1.202:6379 192.168.1.202:6380 192.168.1.203:6379 192.168.1.203:6380 --cluster-replicas 1

# 成功即显示:
# [OK] All nodes agree about slots configuration.
# >>> Check for open slots...
# >>> Check slots coverage...
# [OK] All 16384 slots covered.

# 如果集群已经创建成功
# /opt/software/redis-5.0.4/ 会有2个文件
# nodes-6379.conf nodes-6380.conf
```
## 8.连接集群验证
```sh
./src/redis-cli -h 192.168.0.201 -p 6379 -a xf_server -c 

# 查看集群管理命令
redis-cli --cluster help

# 查看集群各个节点
cluster nodes
```

# Redis集群监控 
```http
https://github.com/nkrode/RedisLive
```
## 1.安装python第三方依赖包
```sh
#基于python的一个高性能web框架
sudo /usr/local/bin/pip3 install tornado
#时间处理包
sudo /usr/local/bin/pip3 install python-dateutil
#python的redis客户端
sudo /usr/local/bin/pip3 install redis
```
## 2.确保安装了sqlite3
```sh
which sqlite3

# 检查是否报错
$ python3
>>> import sqlite3
```
## 3.修改配置文件
### redis-live.conf
```sh
# 注意修改浏览器地址的访问端口,不要和其它集群冲突
define("port", default=8082, help="run on the given port", type=int)

# 直到运行不报错为止
python3 redis-live.py
```
### redis-monitor.py
```sh
# 直到运行不报错为止
python3 redis-monitor.py
```
## 4.监控命令
```sh
./redis-monitor.py --duration 10
```
## 5.开启web服务
```sh
./redis-live.py

# 添加定时任务
*/5 * * * * cd /opt/software/RedisLive/src; ./redis-monitor.py -duration 30 >/dev/null 2>&1
```
## 6.运行服务
```sh
./redis-live.py > log.txt 2>&1 &

# 查看服务是否启动
ps aux | grep redis-live
```
## 7.浏览器地址
```http
http://192.168.0.201:8082/index.html
```
## 8.压力测试
```sh
cd /opt/software/redis-5.0.4/src/
./redis-benchmark -h 192.168.0.201 -p 6379 -a xf_server -n 1000 -c 20
```

# 一些报错信息
## 编译报错: jemalloc/jemalloc.h: No such file or directory
```sh
# 原因:jemalloc重载了Linux下的ANSI C的malloc和free函数。解决办法：make时添加参数。
make MALLOC=libc
```
## CLUSTERDOWN Hash slot not served
```sh
# 检查
redis-cli --cluster check 192.168.1.101:6379 -a cqdsjb
# 修复
redis-cli --cluster fix 192.168.1.101:6379 -a cqdsjb
```
## [ERR] Node 192.168.0.201:6379 is not empty.Either the node already knows other nodes
```sh
# 原因:创建集群后写入了数据;集群已经创建好了.已经是集群模式就不用再创建
	# 解决办法:
# 如果有数据
./src/redis-cli -h 192.168.0.201 -p 6379 -a xf_server -c 
>keys *
>flushall	
# 如果还有数据,直接删除以下2个文件
dump_backup.rdb dump.rdb
```