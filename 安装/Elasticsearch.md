# 下载 Elasticsearch
- https://github.com/mobz/elasticsearch-head
- https://github.com/medcl/elasticsearch-analysis-ik/
- https://www.elastic.co/cn/downloads/past-releases

## 1.配置环境变量
```sh
echo $ES_JAVA_PATH
echo $ES_PATH
```
## 2.创建用户
```sh
tar -zvxf elasticsearch-8.14.1-linux-x86_64.tar.gz
useradd -d /home/elastic -m elastic
usermod -a -G elastic elastic
chown -R elastic:elastic /opt/software/elasticsearch-8.14.1
```
## 3.修改配置文件
### /etc/sysctl.conf
```sh
vm.max_map_count=655300
# 生效
sysctl -p

scp /etc/sysctl.conf by202:/etc/
scp /etc/sysctl.conf by203:/etc/
```

### config/jvm.options

```sh
# 这俩要设置成一样的
-XX:+UseG1GC
-Xms1g
-Xmx1g
-Dfile.encoding=UTF-8
-Dsun.jnu.encoding=UTF-8
```

### bin/elasticsearch-env

```sh
# 看到java路径都改成
JAVA="/opt/software/jdk-17.0.1/bin/java"

scp /opt/software/elasticsearch-8.14.1/bin/elasticsearch-env by202:/opt/software/elasticsearch-8.14.1/bin/
scp /opt/software/elasticsearch-8.14.1/bin/elasticsearch-env by203:/opt/software/elasticsearch-8.14.1/bin/
```

### config/elasticsearch.yml
```sh
# 集群名称
cluster.name: es-cluster
# 节点名称 - 对应修改
node.name: es-201
# 
path.data: /opt/software/elasticsearch-8.14.1/data
path.logs: /opt/software/elasticsearch-8.14.1/logs
# 启动时锁定内存
bootstrap.memory_lock: false
# 绑定 IP
network.host: 0.0.0.0
# 集群节点: 主机名或主机IP
discovery.seed_hosts: ["by201","by202", "by203"]
# 主节点: 上边配置的节点名
cluster.initial_master_nodes: ["es-201","es-202", "es-203"]
# 设置es允许跨域访问
http.cors.enabled: true
http.cors.allow-origin: "*"

# -----------分界线: 上边看情况配置
# 2.1 设置证书(windows去掉./bin)
生成密钥文件, 此命令名称默认会生成 elastic-stack-ca.p12
./bin/elasticsearch-certutil ca
# 为节点颁发证书(+节点密钥和 CA 证书),单节点版(windows直接用单节点版会报: No subject alternative names present)
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
# (推荐)集群版, 先创建 instance.yml
instances:
  - name: "es-201"
    dns: [ "192.168.1.1" ]
    ip: [ "192.168.1.201" ]
  - name: "es-202"
    dns: [ "192.168.1.1" ]
    ip: [ "192.168.1.202" ]
  - name: "es-203"
    dns: [ "192.168.1.1" ]
    ip: [ "192.168.1.203" ]
# 单节点版
instances:
  - name: "node-1"
    dns: [ "192.168.1.1" ]
    ip: [ "192.168.1.3", "127.0.0.1" ]
# 然后执行
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 --in instance.yml --out certs.zip
yum install -y unzip
unzip certs.zip
# (可选)生成https证书签名文件, CSR: no, CA: yes, 注意路径写法: certs/elastic-stack-ca.p12
elasticsearch-certutil http

# 2.2 将证书放到 config/certs目录
mkdir /opt/software/elasticsearch-8.14.1/config/certs
# 单节点证书(这么试过不行,也会报: No subject alternative names present)
mv /opt/software/elasticsearch-8.14.1/*.p12 config/certs/
scp -r /opt/software/elasticsearch-8.14.1/config/certs by202:/opt/software/elasticsearch-8.14.1/config/
scp -r /opt/software/elasticsearch-8.14.1/config/certs by203:/opt/software/elasticsearch-8.14.1/config/
# 集群证书
mv /opt/software/elasticsearch-8.14.1/es-201 /opt/software/elasticsearch-8.14.1/config/certs/
scp es-202/es-202.p12 by202:/opt/software/elasticsearch-8.14.1/config/certs/
scp es-203/es-203.p12 by203:/opt/software/elasticsearch-8.14.1/config/certs/

# 2.3 生成后补充ssl配置(最终版)
xpack.security.enabled: true

xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.keystore.password: By96o122
xpack.security.transport.ssl.truststore.path: certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.password: By96o122

xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: certs/elastic-certificates.p12
xpack.security.http.ssl.keystore.password: By96o122
xpack.security.http.ssl.truststore.path: certs/elastic-certificates.p12
xpack.security.http.ssl.truststore.password: By96o122

# 检查
tail -n 10 /opt/software/elasticsearch-8.14.1/config/elasticsearch.yml
```

## 4.后续启动
```sh
chown -R elastic:elastic /opt/software/elasticsearch-8.14.1
# 首次启动
rm -rf /opt/software/elasticsearch-8.14.1/data/*
su - elastic
# 前台启动
/opt/software/elasticsearch-8.14.1/bin/elasticsearch
# 后台启动
/opt/software/elasticsearch-8.14.1/bin/elasticsearch -d
# windows 直接双击 elasticsearch.bat

# linux启动成功后再初始化下密码,windows不要./
./elasticsearch-setup-passwords interactive
# By96o122
# 手动指定
./elasticsearch-reset-password -u elastic -i
# By96o122
# 注意: 使用kibana 要使用 kibana_system 用户
./elasticsearch-reset-password -u kibana_system -i
# By96o122
```
## 5.web界面
- http://192.168.1.201:9200

```sh
# 查看运行情况：
curl -k -X GET "http://192.168.1.201:9200/_cluster/health?pretty"
curl -k -u elastic:By96o122 -X GET "https://192.168.1.201:9200/_cluster/health?pretty"

# 如果用 DataGrip 连接, 
# 1. 需要将 License 设置为试用版, 进入 kibana 操作
# 更改 License 类型 - trial
POST _license/start_trial?acknowledge=true

# 更改 License 类型 - basic
POST _license/start_basic?acknowledge=true
# 1.1 idea 自带的也可以
GET https://127.0.0.1:9200
Content-Type: application/json
Authorization: Basic elastic By96o122

# 2. 连接的ssl配置, model: Verify CA, 只填写CA file: **.crt


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
elasticsearch.hosts: "http://by201:9200"

# 其他的不用配置, 进Web页面连
```
## 2.启动
```sh
sh elasticsearch 
./kibana
```
## 3.web界面

- 127.0.0.1:5601

## 4.一些报错信息
### libnss3.so: cannot open shared object file: No such file or directory
```sh
yum install -y nss.x86_64
```
