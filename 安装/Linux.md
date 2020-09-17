# 下载地址
```http
https://mirrors.aliyun.com/centos/
```

# 修改Hostname
```shell script
vim /etc/hostname
by201

vim /etc/hosts
192.168.1.201 by201
192.168.1.202 by202
192.168.1.203 by203

vim /etc/sysconfig/network
hostname=by201
```

# 修改网卡配置 - 网络地址转换(NAT)模式
```shell script
vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
ONBOOT=yes

systemctl restart network
ip addr
ipconfig
```

# 修改网卡配置 - 桥接网卡模式
```shell script
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.1.201
NETMASK=255.255.255.0
GETEWAY=192.168.1.1
DNS1=192.168.1.1
```

# Centos7修改repo源配置
```
vim /etc/yum.repos.d/CentOS-Base.repo
[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
        http://mirrors.163.com/centos/$releasever/os/$basearch/
        http://mirrors.aliyun.com/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

# Centos8修改repo源配置
```shell script
cd /etc/yum.repos.d/

[Base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/BaseOS/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/BaseOS/$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
#additional packages that may be useful
[Extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/extras/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/os/
gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
[PowerTools]
name=CentOS-$releasever - PowerTools - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/PowerTools/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/PowerTools/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/PowerTools/$basearch/os/
gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official

[AppStream]
name=CentOS-$releasever - AppStream - mirrors.aliyun.com
failovermethod=priority
baseurl=https://mirrors.aliyun.com/centos/$releasever/AppStream/$basearch/os/
        http://mirrors.aliyuncs.com/centos/$releasever/AppStream/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos/$releasever/AppStream/$basearch/os/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
```

# 测试
```shell script
sudo dhclient
yum -y update
yum install -y vim net-tools lsof tree ntp npm nodejs git zip mlocate httpd createrepo iptables iptables-services tar chrony
```

# iptables安装
```http
https://www.cnblogs.com/kreo/p/4368811.html
```
## 1.安装 iptables
```shell script
yum install -y iptables iptables-services
```
## 2.禁用 firewalld
```shell script
systemctl status firewalld
systemctl stop firewalld
systemctl mask firewalld
```
## 3.设置开放端口,必须写在黄色上面
```shell script
vim /etc/sysconfig/iptables 
-A INPUT -p tcp --dport 21 -j ACCEPT
-A INPUT -p tcp --dport 22 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --dport 3306 -j ACCEPT
-A INPUT -p tcp --dport 7182 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```
## 4.启用 iptables
```
service iptables save
systemctl enable iptables.service
systemctl start iptables.service
```

# ssh免密码登录配置
```shell script
vim /etc/hosts

192.168.1.211 by211
192.168.1.212 by212
192.168.1.213 by213

ssh-keygen -t rsa  
cd /root/.ssh  id_rsa(私钥) id_rsa.pub(公钥)

ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.201
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.202
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.203
```

# 创建本地云源repo压缩包
## 1.上传Centos镜像
## 2.挂载到一个目录
```shell script
mkdir /iso
mount Centos** /iso
```
## 3.进入目录
```
cd /iso/AppStream/Packages
createrepo /var/www/html/openstack
```

# 一些报错信息
# 配置Windows ftp linux 报错:200 PORT command successful. Consider using PASV.425 Failed to establish connection.
```shell script
vim /etc/vsftpd/vsftpd.conf
pasv_enable=YES
pasv_min_port=6000
pasv_max_port=7000

vim /etc/sysconfig/iptables 
-A INPUT -p tcp --dport 6000:7000 -j ACCEPT
  
vim /etc/selinux/config
SELINUX=disabled
```
# ping baidu.com 出现 connect: network is unreachable
```shell script
sudo dhclient
sudo systemctl stop dhcpcd
sudo systemctl start dhcpcd
```

# Centos 8 过期的一些包
## unzip 改为 zip
## ntpdate 改为 chrony
# 解除挂载报错:target is busy
```shell script
yum install psmisc
fuser -mv /iso # 把下面的进程kill掉
```

# 所有命令都无法使用
```shell script
export PATH=/bin:/usr/bin:$PATH
source ~/.bash_profile 
```