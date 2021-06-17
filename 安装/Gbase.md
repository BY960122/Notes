# GBASE 安装

## 1.所有节点新建用户,安装cgconfig服务
```sh
useradd gbase
passwd gbase
usermod -a -G gbase gbase

yum install -y libcgroup libcgroup-tools
# 检查合法性
cgconfigparser -l /etc/cgconfig.conf
# 启动
systemctl start cgconfig.service
```

## 2.开始安装
```sh
# 主节点执行
tar xjf /opt/software/GBase8a_MPP_Cluster-License-9.5.2.39-redhat7.3-x86_64.tar.bz2 
cd gcinstall 
scp SetSysEnv.py 192.168.1.212:/opt/software/gcinstall 
vim demo.options

installPrefix= /opt/software/gbase
coordinateHost = 192.168.1.211,192.168.1.212
coordinateHostNodeID = 211,212
dataHost = 192.168.1.211,192.168.1.212,192.168.1.213
#existCoordinateHost = 用于扩容
#existDataHost = 用于扩容
dbaUser = gbase
dbaGroup = gbase
dbaPwd = 'gbase'
rootPwd = 'By9216446o6'
#rootPwdFile = rootPwd.json

# 所有节点执行
mkdir /opt/software/gbase 
# cgroup可以不加,用于集群资源管理
python2 SetSysEnv.py --installPrefix=/opt/software/gbase --dbaUser=gbase --cgroup

# 主节点执行
python2 gcinstall.py --silent=demo.options
# 查看日志
tail -f /opt/software/gcinstall/gcinstall.log
```

## 3.获取 License
```
su - root
cd /opt/software/gcinstall 
vim demo.hosts
Hosts=192.168.1.211,192.168.1.212,192.168.1.213

python2 gethostsid --hosts=demo.hosts -u root -p By9216446o6 -f by_hostsfinger.txt
```

## 4.导入 License
```sh
su - gbase
cd /opt/software/gcinstall 
python2 License --hosts=demo.hosts -u gbase -p gbase -f /opt/software/gcinstall/20210617-19.lic

# 查看
python2 chkLicense -n 192.168.1.211,192.168.1.212,192.168.1.213 -u gbase -p gbase
# -f 为检查结果 -f result.txt
python2 chkLicense --hosts=demo.hosts -u gbase -p gbase 

# 没问题后各节点都执行
gcluster_services all restart
```

## 5.设置分片信息
```sh
su - root
cd /opt/software/gcinstall 
vim /opt/software/gcinstall/gcChangeInfo.xml 
<?xml version="1.0" encoding="utf-8"?>
<servers>
<rack>
 <node ip="192.168.1.211"/>
 <node ip="192.168.1.212"/>
 <node ip="192.168.1.213"/>
</rack>
</servers>

su - gbase
# p 每个数据节点存放的分片数量,最小值为 1
# 每个分片的备份数量,取值为 0,1,2。若不输入参数 d,默认值为 1。
gcadmin distribution gcChangeInfo.xml p 2 d 1 
gcadmin showdistribution node
```

## 6.数据库初始化
```sh
# root,没有密码
gccli -uroot 
# gbase用户
gccli -ugbase gbase20110531

# 只需要执行一次
initnodedatamap;

```

## 查看
```sh
su - gbase
gcadmin
```

# GBASE 卸载
```sh
su - root
cd /opt/software/gcinstall 
# 查看进程
gcmonit --status

# 停止服务
gcluster_services all stop 

python2 uninstall.py --silent=demo.options
```