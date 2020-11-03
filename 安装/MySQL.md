# 下载 MySQL
- https://downloads.mysql.com/archives/community/
- https://mirrors.cloud.tencent.com/mysql/downloads/MySQL-8.0/

# MySQL集群安裝
- https://www.cnblogs.com/gomysql/p/3664783.html
- https://www.cnblogs.com/gomysql/

# MySQL 8.0 + (windows安装)
## 1.配置环境变量 
```sh
echo $MYSQL_HOME
```
## 2.配置my.ini 文件
## 3.进入安装目录下执行
```txt 
mysqld --initialize --console (这里会产生密码)
```
## 4.再执行
```txt
mysqld --install mysql
net start mysql
```
## 5.进入mysql 修改密码(这是必须是第一步) 
```sql
set global validate_password.length = 6;
set global validate_password.policy = 0;
update mysql.user set host = '%' where user = 'root';
grant all on *.* to 'root'@'%';
alter user root@'%' identified by 'By9216446o6'; 
```
## 6.赋予权限 
```sql
grant all on *.* to 'root'@'%';
```
## 7.刷新权限 
```sql
flush privileges;
```
## 8.查看权限 
```sql
show grants for 'root'@'%';
```

# MySQL 8.0 + (windows升级)
## 1.删除原有的mysql服务
```txt
sc delete mysql
```
## 2.复制原安装目录的data文件夹和my.ini到新目录
## 3.修改环境变量
## 4.安装mysql
```txt
mysqld install
```
## 5.启动mysql,配置文件必须是ANSI格式
```txt
net start mysql
```

# MySQL 8.0 + (Linux安装)
## 1.配置环境变量 
```sh
echo $MYSQL_HOME
```
## 2.查看可以安装哪些安装包
```sh
yum repolist all | grep mysql
rpm -pa | grep mysql
rpm -pa | grep mariadb
```
## 3.删除之前的
```sh
yum remove ***
rpm -ev ***
```
## 4.安装前先添加mysql组,和mysql用户
```sh
groupadd mysql
useradd -r -g mysql mysql
chown -R mysql:mysql /opt/software/mysql-8.0.21
```
## 5.安装
```sh
xz -d mysql-8.0.21-linux-glibc2.12-x86_64.tar.xz
tar -vxf mysql-8.0.21-linux-glibc2.12-x86_64.tar -C mysql-8.0.21

bin/mysqld --initialize --user=mysql --basedir=/opt/software/mysql-8.0.21 --datadir=/opt/software/mysql-8.0.21/data
```
## 6.然后记得检查下配置文件是否存在,没有的话手动添加
```sh
vim /etc/my.cnf

[mysqld]
basedir=/opt/software/mysql-8.0.21
datadir=/opt/software/mysql-8.0.21/data
```
## 7.添加服务并启动,重启记得先停止!!!
```sh
cd support-files/
cp mysql.server /etc/init.d/mysql 
chmod +x /etc/init.d/mysql
chkconfig --add mysql
systemctl start mysql
systemctl status mysql
```
## 8.连接mysql
```sql
set global validate_password.length = 8;
set global validate_password.policy = 0;
update mysql.user set host = '%' where user = 'root';
grant all on *.* to 'root'@'%';
alter user root@'%' identified by 'By9216446o6';
flush privileges;
```
## 9.新建MySQL用户
```sql
create user 'hadoop'@'%' identified by 'hadoop';  
grant all on *.* to 'hadoop'@'%' with grant option;
flush privileges;
```

# 设置主从配置
## 原理 master:每产生磁盘变化就写进binlog日志 slave:转换成repaylog 
## 1.master配置文件
```sh
server-id=102
log-bin=mysql-bin
# statemet(语句变化):影响很多行的情况适合用语句模式
# row(磁盘行数): insert,update,影响单行适合磁盘模式
binlog-format=mixed
```
## 2.slave配置文件
```sh
server-id=201
relay-log=mysql-relay
```
## 3.分别启动主从mysql
## 4.登录master,创建master用户
```sql
create user 'master'@'localhost' identified by 'By9216446o6';
grant all on *.* to 'master'@'localhost';
update mysql.user set host = '%' where user = 'master';
flush privileges;
```
## 5.登录slave
```sql
change master to
master_host='192.168.0.201',
master_user='master',
master_password='By921644606',
master_log_file='mysql-bin.000001',
master_log_pos=***;
```
## 6.查看状态,常用命令
```sql
show slave status;
reset slave;
start slave;
stop slave;
```

# 设置主主配置
## 1.master配置文件
```sh
server-id=102
log-bin=mysql-bin
relay-log=mysql-relay
binlog-format=mixed
```
## 2.slave配置文件
```sh
server-id=201
log-bin=mysql-bin
relay-log=mysql-relay
binlog-format=mixed
```
## 3.两个服务器同时建立master帐号
```sql
create user 'master'@'localhost' identified by 'By9216446o6';
grant all on *.* to 'master'@'localhost';
update mysql.user set host = '%' where user = 'master';
flush privileges;
change master to 
master_host='192.168.0.201',
master_user='master',
master_password='By921644606',
master_log_file='mysql-bin.000001',
master_log_pos=***;
```
## 4.同时启动
```sql
start slave;
```

# 一些报错
## 主主复制,主键冲突问题
### 分别让2台服务器以奇数和双数自增
```sql
set global auto_increment__increment=2;   #每步增长2
set global auto_increment_offset=1;     #从1开始增长,另外一台只需要修改这里为2
```
## MySQL 8.0.21 x64安装,发生系统错误 193.不是有效的Win32程序
```txt
进入 mysql\bin目录下 查看 mysqld文件,是不是有一个为0kb的,删掉它
```
## 初始化数据库报错:bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
```sh
# 进入mysql-8.0.21/bin 目录执行,查看各个依赖项
ldd mysql 
# 从别的电脑复制一个过来
scp /usr/lib64/libtinfo.so.5 192.168.1.201:/usr/lib64/
# 再查看
ll /usr/lib64/libtinfo.so.5.9
```

# 范例配置文件,不要设置成UTF-8格式
```sh
[mysqld]
# skip-grant-tables
# 设置3306端口
port=3306

# 设置mysql的安装目录
basedir=D:\\Mysoft\\MySQL-8.0.11   # 切记此处一定要用双斜杠\\，单斜杠我这里会出错，不过看别人的教程，有的是单斜杠。自己尝试吧

# 设置mysql数据库的数据的存放目录
datadir=D:\\Mysoft\\MySQL-8.0.11\\data   # 此处同上

# 允许最大连接数
max_connections=200

# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
max_connect_errors=10

# MySQL 8.0.21 必须设置,否则在jdbc连接的时候要制定 &serverTimezone=Asia/Shanghai
default-time_zone = '+8:00'

# 开启导入文件命令 load data
local_infile=ON

# 服务端使用的字符集默认为utf8mb4
# 设置为 False, 在客户端字符集和服务端字符集不同的时候将拒绝连接到服务端执行任何操作
character-set-client-handshake = FALSE  
character-set-server = utf8mb4  
collation-server = utf8mb4_unicode_ci  
init_connect='SET NAMES utf8mb4'

# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

# 默认使用“mysql_native_password”插件认证
default_authentication_plugin=mysql_native_password

# [Err] 1055 Group by警告
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'

# [Err] 1148 导出受限
secure-file-priv="D:\\"

# [Err] 1148 (42000): The used command is not allowed with this MySQL version
#这里需要mysqld 和 mysql 同时设置
local-infile=1

# [Err] 1418 创建函数,存储过程,触发器
log-bin-trust-function-creators=1

# 切分ibdata1
innodb_file_per_table=1

# 插入慢优化,配置前先备份旧配置,再到数据库查看相应默认值,一点点修改.
# =1最安全,=2,系统崩溃时会丢失事物,=0裸奔,最快.有事物慎用
innodb_flush_log_at_trx_commit=1
# 插入缓冲区大小,默认8M
bulk_insert_buffer_size=1024M
# 日志缓存,如果insert很大,建议调大,以节约磁盘I/O
innodb_log_buffer_size=1024M
# 日志文件大小(innodb_log_file_size*innodb_log_files_in_group(default 2))*0.75=日志差值(show engine innodb status)
#innodb_log_file_size=1024M
# 主要缓存innodb表的索引，数据，插入数据时的缓冲
innodb_buffer_pool_size=1024M
# 类似于innodb_buffer_pool_size,它的对象是myisam引擎
key_buffer_size=1024M
# 用于在表空间已满时的增量大小,默认4M,Mysql8.0配置不成功
# innodb_authoextend_increment=256M
# 自动提交事物,关闭效率更高,如果用到事物建议默认为1
# autocommit=1

# 实时更新tables表信息,默认86400为一天
information_schema_stats_expiry=0

# 主从数据库配置
# server-id=102
# log-bin=mysql-bin
# binlog-format=mixed

[mysql]

# 设置mysql客户端默认字符集
default-character-set=utf8mb4

# [Err] 1148 (42000): The used command is not allowed with this MySQL version
local-infile=1

[client]

# 设置mysql客户端连接服务端时默认使用的端口和字符集
port=3306
default-character-set=utf8mb4
```