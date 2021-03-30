# 下载 MongoDB
- https://www.mongodb.com/download-center#community

# Windows安装
## 1.初始化数据目录,记得新建目录
```shell script
mongod --dbpath D:\Mysoft\MongoDB\data
```
## 2.安装服务,记得用管理员
```shell script
mongod.exe --logpath D:\Mysoft\MongoDB\log\mongodb.log --logappend --dbpath D:\Mysoft\MongoDB\data --directoryperdb --serviceName MongoDB --install
```
## 3.启动服务,停止服务
```shell script
net start mongodb
net stop mongodb
```
## 4.设置用户密码,只能单独给一个库创建用户权限
```mongojs
use admin;
db.createUser({user: 'root', pwd: 'By9216446o6', roles: [{ role: "root", db: "admin" }]});
```
## 5.检查是否创建成功
```mongojs
db.auth('root', 'By960122')
```
## 6.查看已有用户
```mongojs
db.system.users.find()
db.system.users.find({user:"root"})
```
## 7.删除用户
```mongojs
db.system.users.remove({user:'root'})
```