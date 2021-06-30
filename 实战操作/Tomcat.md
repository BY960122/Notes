# Tomcat 安装

```sh
# 解压启动即可
sh /opt/software/apache-tomcat-9.0.41/bin/startup.sh
```

## Tomcat windows 启动或 idea 启动控制台乱码

> 修改 conf\logging.properties 所有 encoding 项为 GBK 试试

## Tomcat 修改为 https

### 1.进入 java 8 bin目录下
```sh
# windows 
keytool.exe -genkeypair -alias "tomcat" -keyalg "RSA" -keystore "D:\Mysoft\apache-tomcat-9.0.41\tomcat.keystore" 
# linux
keytool -genkeypair -alias "tomcat" -keyalg "RSA" -keystore /opt/software/apache-tomcat-9.0.41/tomcat.keystore"

# 输入 密钥口令: By9216446o6 , 名字与姓氏: BYDylan 即可,其它直接回车,最后确认输入 y
```

### 2.修改配置文件
#### server.xml
```xml
<!--将原有的注释掉-->
<Connector protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="8080" maxThreads="200"
           scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="D:\Mysoft\apache-tomcat-9.0.41\tomcat.keystore" keystorePass="By9216446o6"
           clientAuth="false" sslProtocol="TLS"/>
```

#### web.xml
```xml
```
