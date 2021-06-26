# 下载 Elasticsearch
- https://github.com/mobz/elasticsearch-head
- https://github.com/medcl/elasticsearch-analysis-ik/
- https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-13-1

## 1.配置环境变量
```sh
echo $ELASTICSEARCH
```
## 2.创建用户
```sh
tar -zvxf elasticsearch-7.13.1-linux-x86_64.tar.gz
useradd -d /home/elasticsearch -m elasticsearch
usermod -a -G elasticsearch elasticsearch
chown -R elasticsearch:elasticsearch /opt/software/elasticsearch-7.13.1
```
## 3.修改配置文件
### /etc/sysctl.conf
```sh
vm.max_map_count=655300
# 生效
sysctl -p
```
### config/elasticsearch.yml
```sh
# 集群名称
cluster.name: elasticsearch-cluster
# 节点名称 - 对应修改
node.name: elasticsearch-201
# 启动时锁定内存
bootstrap.memory_lock: false
# 绑定 IP
network.host: 0.0.0.0
# 集群节点
discovery.seed_hosts: ["192.168.1.201","192.168.1.202", "192.168.1.203"]
# 主节点
cluster.initial_master_nodes: ["elasticsearch-203"]
# 设置es允许跨域访问
http.cors.enabled: true
http.cors.allow-origin: "*"
```
### config/jvm.options
```sh
# -XX:+UseConcMarkSweepGC 
-XX:+UseG1GC 

scp /opt/software/elasticsearch-7.13.1/config/* 192.168.1.202:/opt/software/elasticsearch-7.13.1/config/
scp /opt/software/elasticsearch-7.13.1/config/* 192.168.1.203:/opt/software/elasticsearch-7.13.1/config/
```
### bin/elasticsearch-env
```sh
# 看到java路径都改成
JAVA="/opt/software/jdk-11.0.1/bin/java"

scp elasticsearch-env 192.168.1.202:/opt/software/elasticsearch-7.13.1/bin/
scp elasticsearch-env 192.168.1.203:/opt/software/elasticsearch-7.13.1/bin/
```
## 4.启动
```sh
su - elasticsearch
# 前台启动
bin/elasticsearch
# 后台启动
/opt/mysoft/elasticsearch-7.13.1/bin/elasticsearch -d
```
## 5.web界面
- http://192.168.1.201:9200

```sh
# 查看运行情况：
curl 'http://192.168.1.201:9200/_cluster/health?pretty'
```
## 6.一些报错信息
### elasticsearch-env: line 122: syntax error near unexpected token <'
```sh
# 修改 elasticsearch-env 122 行
#   done < <(env) 改为  done <<< 'env'
```

# 下载 Kibana
- https://www.elastic.co/cn/downloads/past-releases

## 1.修改配置文件
### kibana.yml 
```sh
server.host: "0.0.0.0"
elasticsearch.hosts: "http://192.168.1.203:9200"
```
## 2.启动
```sh
sh elasticsearch 
./kibana
```
## 3.web界面
- 192.168.1.201:5601

## 4.一些报错信息
### libnss3.so: cannot open shared object file: No such file or directory
```sh
yum install -y nss.x86_64
```
