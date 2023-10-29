# 下载地址

- https://mirrors.aliyun.com/centos/8.5.2111/isos/x86_64/?spm=a2c6h.25603864.0.0.54f344cb52OwJ1
- [centos7和8的区别](https://www.ycyaw.com/Linux/317.html)

## 修改网卡配置
```sh
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

# 网络地址转换(NAT)模式
ONBOOT=yes
# 桥接网卡模式
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.1.201
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=192.168.1.1

# 关闭防火墙
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld

# centos 7
systemctl restart network

ipconfig
# centos 8
# 确认网卡激活
nmcli connection up enp0s3
# 检查接口是否有正确的 IP 地址分配
ip addr show
# 重启网络管理器
systemctl restart NetworkManager
systemctl enable NetworkManager
# 验证
ping baidu.com
```

## Centos7修改repo源配置

- http://mirrors.aliyun.com/repo/Centos-7.repo
- http://mirrors.163.com/.help/CentOS7-Base-163.repo

## Centos8修改repo源配置

```sh
rm -rf /etc/yum.repos.d/*
vi /etc/yum.repos.d/CentOS-Base.repo

[base]
name=CentOS-8.5.2111 - Base - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos-vault/8.5.2111/BaseOS/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/BaseOS/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/BaseOS/$basearch/os/
gpgcheck=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
#additional packages that may be useful
[extras]
name=CentOS-8.5.2111 - Extras - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos-vault/8.5.2111/extras/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/extras/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/extras/$basearch/os/
gpgcheck=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-8.5.2111 - Plus - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos-vault/8.5.2111/centosplus/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/centosplus/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/centosplus/$basearch/os/
gpgcheck=0
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official
 
[PowerTools]
name=CentOS-8.5.2111 - PowerTools - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos-vault/8.5.2111/PowerTools/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/PowerTools/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/PowerTools/$basearch/os/
gpgcheck=0
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official

[AppStream]
name=CentOS-8.5.2111 - AppStream - mirrors.aliyun.com
baseurl=http://mirrors.aliyun.com/centos-vault/8.5.2111/AppStream/$basearch/os/
        http://mirrors.aliyuncs.com/centos-vault/8.5.2111/AppStream/$basearch/os/
        http://mirrors.cloud.aliyuncs.com/centos-vault/8.5.2111/AppStream/$basearch/os/
gpgcheck=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-Official

# 测试
# centos7: sudo dhclient
yum clean all & yum makecache & yum -y update
yum install -y vim tar
```

## 修改主机名 Hostname
```sh
vim /etc/hostname
by201

vim /etc/hosts
192.168.1.201 by201
192.168.1.202 by202
192.168.1.203 by203

vim /etc/sysconfig/network
hostname=by201
```

## ssh免密码登录配置
```sh
vi /etc/hosts

192.168.1.201 by201
192.168.1.202 by202
192.168.1.203 by203

ssh-keygen -t rsa  
cd /root/.ssh  id_rsa(私钥) id_rsa.pub(公钥)

ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.201
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.202
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.203

# 验证
ssh by201
```

## 防火墙,iptables安装
- https://www.cnblogs.com/kreo/p/4368811.html

### 1.安装 iptables
```sh
yum install -y iptables iptables-services
```
### 2.禁用 firewalld
```sh
systemctl status firewalld
systemctl stop firewalld
systemctl mask firewalld
```

### 选择性禁用
```sh
# 注意看下是不是软连接 -> /etc/sysconfig/selinux
ll /etc/sysconfig/selinux

SELINUX=disabled

# 重启后验证
sestatus
```

### 3.设置开放端口,必须写在黄色上面
```sh
vim /etc/sysconfig/iptables 
-A INPUT -p tcp --dport 21 -j ACCEPT
-A INPUT -p tcp --dport 22 -j ACCEPT
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --dport 3306 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```
### 4.启用 iptables
```
service iptables save
systemctl enable iptables.service
systemctl start iptables.service

systemctl disable iptables.service
systemctl mask iptables.service
```

## 创建本地云源repo压缩包
### 1.上传Centos镜像
### 2.挂载到一个目录
```sh
mkdir /iso
mount Centos** /iso
```
### 3.进入目录
```
cd /iso/AppStream/Packages
createrepo /var/www/html/openstack
```

## 一些报错信息
### 配置Windows ftp linux 报错:200 PORT command successful. Consider using PASV.425 Failed to establish connection.
```sh
vim /etc/vsftpd/vsftpd.conf
pasv_enable=YES
pasv_min_port=6000
pasv_max_port=7000

vim /etc/sysconfig/iptables 
-A INPUT -p tcp --dport 6000:7000 -j ACCEPT
  
vim /etc/selinux/config
SELINUX=disabled
```
### ping baidu.com 出现 connect: network is unreachable
```sh
sudo dhclient
sudo systemctl stop dhcpcd
sudo systemctl start dhcpcd
```
### 所有命令都无法使用
```sh
export PATH=/bin:/usr/bin:$PATH
source ~/.bash_profile 
```

## Centos 8 过期的一些包
### unzip 改为 zip
### ntpdate 改为 chrony

## 一些报错信息
### 解除挂载报错:target is busy
```sh
yum install psmisc
fuser -mv /iso ## 把下面的进程kill掉
```
### Virtualbox未关机就关闭电脑,无法启动,unmount and run xfs_repair
```sh
ls -l /dev/mapper
xfs_repair /dev/mapper/cl_muban-root
# 如果报错 The filesystem has valuable metadata change in a log ...
xfs_repair -L /dev/mapper/cl_muban-root
```

## 一些安装过程
### 安装cgconfig服务
```sh
yum install -y libcgroup libcgroup-tools
# 检查合法性
cgconfigparser -l /etc/cgconfig.conf
# 启动
systemctl start cgconfig.service
```