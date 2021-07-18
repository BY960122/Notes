# 安装 Docker
- https://docs.docker.com/engine/install/
- 命令大全 https://docs.docker.com/engine/reference/run/

## Linux 安装
```sh
# 1.卸载旧版本
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
rm -rf /var/lib/docker
rm -rf /var/lib/containerd

# 2.安装依赖
yum install -y yum-utils

# 3.添加yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 顺便更新一下 yum 源
yum makecache fast

# 4.安装, ce 社区版, ee 企业版
yum install docker-ce docker-ce-cli containerd.io 

# 如果需要安装自定版本
# 1.查看有哪些版本
yum list docker-ce --showduplicates | sort -r

# 2.安装指定版本
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io

# 5.启动 docker
systemctl start docker
systemctl stop docker
systemctl restart docker
docker version
docker info

# 6.测试
docker run hello-world

# 7.查看有哪些镜像
docker images
```

## Windows 安装
- https://docs.docker.com/docker-for-windows/install/

```sh
# 安装前准备
# 1.windows 控制面板 -> 程序和 功能 -> 启用或关闭Windows功能 , 打开 适用于 Linux 的 Windows 子系统
# 2.windows 控制面板 -> 程序和 功能 -> 启用或关闭Windows功能 , 打开 Hyper-V
# 如果报错: WSL 2 installation is incomplete, 需要更新一下
https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
netsh winsock reset
```