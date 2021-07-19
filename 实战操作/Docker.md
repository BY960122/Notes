# Docker 命令大全

## 常用命令
```sh
# 查看所有镜像
docker images

# 搜索镜像
docker search mysql
# 过滤标星3000以下的
docker search mysql --filter=STARS=3000

# 删除镜像
docker rm -f 容器id
docker rm -f 容器id1 容器id2 容器id3
docker rm -f $(docker images -aq)

# 拉取镜像
docker pull centos

# 启动镜像
# --name='容器名字'
# -d 后台运行
# -it 交互方式运行
# -v 主机目录:容器目录,它俩相互同步,docker 删了数据不会丢 
# -p 端口 主机端口:容器端口 ip:主机端口:容器端口
# -P 随机端口
# --rm 用完就删,一般用于测试
docker run -it centos /bin/bash
docker run -d --name nginx01 -p 3344:80 nginx
# es 启动很占内存,这样启动
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2

# 列出所有运行的容器
docker ps 
# -a 正在运行 + 历史
# -n 最近n个运行的容器
# -q 只显示容器编号

# 退出
exit

# 退出但不通知
ctrl + p + q

# 删除容器(不能删除正在运行的容器)
docker rm 容器id
docker rm -f $(docker ps -aq)
docker -a -q | xargs docker rm 

# 停止容器
docker start 容器id
docker restart 容器id
docker stop 容器id
docker kill 容器id

# 显示日志
docker logs -tf --tail 10 容器id

# 查看进程
docker top
docker stats

# 查看容器信息
docker inspect 容器id

# 进入容器
docker exec -it 容器id bashbashShell
docker attach 容器id

# 把容器内文件拷贝出来
docker cp 容器id:/home/test.txt /home
# 把文件拷贝到容器里面

# docker 图形化管理界面
docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
访问
localhost:8088
```

## 数据同步挂载
```sh
# mysql 设置挂载盘
# :ro readonly,只能通过外部改边,容器内没有权限修改
# :rw 可读可写
docker run -d -p:3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql:ro -e MYSQL_ROOT_PASSWORD=By9216446o6 --name mysql-docker mysql:5.7

# 具名挂载和匿名挂载
# 具名挂载: -v 卷名:容器内路径
# 匿名挂载: -v 容器内路径
# 指定路径挂载: -v /容器外路径:容器内路径
# 查看所有挂载卷
docker volumn ls
# 匿名
# loacl q65w4d6qw46e846q8w4e64q6w84e64684
# 具体名 
# loacl 卷名
# 查看其路径
docker volumn inspect 卷名(或随机那串字符)

docker容器相互同步
# 父容器
docker run -it --name docker01 容器名:版本号
# 子容器,继承父容器
docker run -it --name docker02 --volumns-from docker01

```

## Dockerfile
```sh
# 提交镜像
docker commit -m="描述信息" -a="作者" 容器id 目标镜像名:version

# 所有命令都是大写
# FROM 基础镜像,一切从这里开始
# MAINTAINER 作者,姓名+邮箱
# RUN 镜像构建的时候需要运行的命令
# ADD 步骤: 添加内容,压缩包会自动解压
# WORKDIR 工作目录,刚进去的地方
# VOLUMN 挂载目录
# EXPOSE 保留的端口
# CMD 指定容器启动时要运行的命令,只有最后一个生效,可被替代
# ENTRYPOINT 指定容器启动时要运行的命令,追加模式,例如 ENTRYPOINT ["ls","-a"] , 后面只需要加 docker run 容器id -l 即可,实际执行的是 ls -al
# ONBUILD 当构建一个被集成的 dockerfile , 这个时间就会运行 ONBUILD 命令,触发指令.
# COPY 类似ADD,将文件拷贝进镜像
# ENY 构建的时候设置环境变量

# 示例:创建一个自定义centos,名字一般命令为默认的 Dockerfile ,好处是可以不用加 -f 参数
FROM centos
MAINTAINER BYDylan<921644606@qq.com>

COPY readme.txt /usr/local/readme.txt

ADD jdk-8u291-linux-x64.tar.gz /opt/software/
ADD apache-tomcat-9.0.41.tar.gz /opt/software/

# 只有容器内路径: 匿名挂载
# VOLUMN ["volumns01","volumns02"]
RUN yum install -y vim
RUN yum install -y net-tools

ENV MYPATH /usr/local
ENY JAVA_HOME /opt/software/jdk1.8.0_291
ENY CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENY CATALINA_HOME /opt/software/apache-tomcat-9.0.41
ENY CATALINA_BASE /opt/software/apache-tomcat-9.0.41
ENY PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib

WORKDIR $MYPATH

EXPOSE 80 8080

CMD $MYPATH
CMD /bin/bash
CMD /opt/software/apache-tomcat-9.0.41/bin/startup.sh && tail -f /opt/software/apache-tomcat-9.0.41/logs/catalina.out 

# dockerfile 生成镜像
docker build -f dockerfile文件全路径 -t by-centos:1.0 .
# 启动刚刚的镜像
docker run -it 镜像名对应的image_id /bin/bash

# 查看镜像的构建过程
docker history 镜像id
```