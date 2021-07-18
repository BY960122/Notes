# Docker 常用命令
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

# 启动 镜像
docker run 
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
# mysql 设置挂载盘
docker run -d -p:3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=By9216446o6 --name mysql-docker mysql:5.7

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

# 提交镜像
docker commit -m="描述信息" -a="作者" 容器id 目标镜像名:version

# 检查提交的镜像
docker images
```