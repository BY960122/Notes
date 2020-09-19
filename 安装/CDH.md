# 下载 CDH
- https://blog.csdn.net/wsdc0521/article/details/108366867
- https://docs.cloudera.com/documentation/enterprise/6/release-notes/topics/rg_requirements_supported_versions.html#cm_cdh_compatibility
- https://archive.cloudera.com/cdh6/6.3.2/parcels/CDH-6.3.2-1.cdh6.3.2.p0.1605554-el7.parcel
- https://archive.cloudera.com/cdh6/6.3.2/parcels/manifest.json
- https://archive.cloudera.com/cm6/6.3.1/repo-as-tarball/cm6.3.1-redhat7.tar.gz
- https://archive.cloudera.com/cm6/6.3.1/allkeys.asc

## 1.时间同步
```sh
yum install -y chrony
vim /etc/chrony.conf

# 指定上层NTP服务器为阿里云提供的公网NTP服务器
server 192.168.1.211 iburst minpoll 4 maxpoll 10
# 如果系统时钟的偏移量大于10秒,则允许在前三次更新中步进调整系统时钟
makestep 10 3
# 允许网段
allow 192.168.1.0/16

scp /etc/chrony.conf 192.168.1.211:/etc/
scp /etc/chrony.conf 192.168.1.212:/etc/

systemctl start chronyd.service
systemctl status chronyd.service
systemctl enable chronyd.service

timedatectl set-timezone Asia/Shanghai
```
## 2.http,createrepo
```sh
yum install -y httpd createrepo
systemctl start httpd
systemctl status httpd
systemctl enable httpd
```
### rhel6配置
```sh
vim /etc/sysctl.conf

vm.swappiness=0
```
### rhel7配置
```sh
cd /usr/lib/tuned/
grep "vm.swappiness" * -R
然后将文件中的配置依次配置为0

scp /etc/sysctl.conf 192.168.1.212:/etc/
scp /etc/sysctl.conf 192.168.1.213:/etc/
scp -r /usr/lib/tuned/* 192.168.1.212:/usr/lib/tuned/
scp -r /usr/lib/tuned/* 192.168.1.213:/usr/lib/tuned/
```
## 3.所有节点用透明页
```sh
vim /etc/rc.local 
追加
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled

scp /etc/rc.local 192.168.1.212:/etc/
scp /etc/rc.local 192.168.1.213:/etc/
```
## 4.配置MySQL
```sql
create database cmf;
create database amon;

create user 'cmf'@'%' identified by 'By9216446o6'; 
create user 'amon'@'%' identified by 'By9216446o6'; 
grant all on cmf.* to 'cmf'@'%' ;
grant all on amon.* to 'amon'@'%';
flush privileges;

select host,user from user;
```
## 5.配置 MySQL 驱动
```sh
# 必须是这个路径...
cp /opt/mysql-connector-java.jar /usr/share/java/
scp /opt/mysql-connector-java.jar 192.168.1.212:/usr/share/java/
scp /opt/mysql-connector-java.jar 192.168.1.213:/usr/share/java/
```
## 6.部署离线parcel源
```sh
mkdir -p /var/www/html/cdh6_parcel
mv /opt/CDH* /var/www/html/cdh6_parcel
mv /opt/manifest.json /var/www/html/cdh6_parcel
```
```http
<!-- 查看页面 -->
http://by211/cdh6_parcel/
```
## 7.安装CM
```sh
mkdir /opt/software/cloudera-manager
tar -zvxf /opt/cm6.3.1-redhat7.tar.gz -C /opt/software/cloudera-manager

cd /opt/software/cm6.3.1/RPMS/x86_64/
# 拷贝 allkeys.asc 到当前目录
cp /opt/software/cm6.3.1/RPMS/x86_64/* /var/www/html/cdh6_parcel
cp -r /opt/software/cm6.3.1/repodata/ /var/www/html/cdh6_parcel
# 或者自己在 /opt/software/cm6.3.1/RPMS/x86_64/ 下执行
createrepo .
# 然后把生成的 repodata 目录传过去

# 新建cm源
vim /etc/yum.repos.d/cloudera-manager.repo

[cloudera-manager]
name=Cloudera Manager 6.3.0
baseurl=http://by211/cdh6_parcel/
gpgcheck=0
enabled=1

yum clean all && yum makecache

# 所有节点都执行
rpm -ivh cloudera-manager-daemons-6.3.1-1466458.el7.x86_64.rpm --nodeps --force
# 主节点单独执行 
rpm -ivh cloudera-manager-server-6.3.1-1466458.el7.x86_64.rpm --nodeps --force 
# 从节点单独执行
rpm -ivh cloudera-manager-agent-6.3.1-1466458.el7.x86_64.rpm --nodeps --force

# 从节点上配置cm agent的配置,指向server节点为主节点
sed -i "s/server_host=localhost/server_host=192.168.1.211/g" /etc/cloudera-scm-agent/config.ini
sed -i "s/use_tls=0/use_tls=1/g" /etc/cloudera-scm-agent/config.ini

# 配置主节点的cm server配置:
vim /etc/cloudera-scm-server/db.properties

com.cloudera.cmf.db.type=mysql
com.cloudera.cmf.db.host=192.168.1.6
com.cloudera.cmf.db.name=cmf
com.cloudera.cmf.db.user=cmf
com.cloudera.cmf.db.password=By9216446o6
com.cloudera.cmf.db.setupType=EXTERNAL
```
## 8.启动
```sh
# 主节点启动server
systemctl start cloudera-scm-server
systemctl status cloudera-scm-server

tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log

# 从节点启动agent
systemctl start cloudera-scm-agent
systemctl status cloudera-scm-agent

tail -f /var/log/cloudera-scm-agent/cloudera-scm-agent.log
```
## 9.Web页面
- http://by211:7180