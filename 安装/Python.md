# 下载 Python
- https://www.python.org/ftp/python/
- https://tecadmin.net/install-python-3-7-on-ubuntu-linuxmint/

## 1.配置环境变量
```sh
echo $PYTHOME_HOME
```
## 2.依赖准备
```sh
yum install -y zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel 
yum install -y gcc gcc-c++ tcl zlib zlib-devel perl libffi-devel openssl-devel openssl ruby
```
## 3.编译
```sh
tar zvxf Python-3.9.1.tgz -C python-3.9.1
cd python-3.9.1
sudo ./configure --enable-optimizations --prefix=/usr/local/python3
sudo make && make install
```
## 4.检查
```sh
python3.8 --version
python3 --version
```
## 5.离线安装包
- https://pypi.org/

```sh
# Windows
python -m pip install xxx.whl
python -m pip install --user xxx.whl

# Linux
tar -zxvf pip-19.2.3.tar.gz
cd pip-19.2.3
python3.8 setup.py install
```
## 6.普通安装
```sh
pip3 config set global.index-url https://mirrors.cloud.tencent.com/pypi/simple
pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip3 config set global.index-url https://mirrors.tools.huawei.com/pypi/simple/
pip3 config set global.trusted-host mirrors.aliyun.com

pip3 install --upgrade pip
python -m pip install -i https://mirrors.aliyun.com/pypi/simple/ --upgrade pip
pip install -i https://mirrors.aliyun.com/pypi/simple/ you-get

# 查看安装包的依赖情况--离线安装时,如果有依赖需先安装底层依赖
pip show packagename
# -j 以json串格式显示
pip install pipdeptree
pipdeptree -p packagename 
```
## 7.一些报错信息
### Can't connect to HTTPS URL because the SSL module is not available.
```sh
wget https://www.openssl.org/source/openssl-1.1.1e.tar.gz
cd openssl-1.1.1-pre8
#不需要zlib
./config --prefix=/usr/local/openssl no-zlib
make && make install
```
### Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2
```http
https://github.com/lakshayg/tensorflow-build
```
### No module named '_ ctypes'
```http
http://mirror.centos.org/centos/7/os/x86_64/Packages/libffi-devel-3.0.13-18.el7.x86_64.rpm
```
```sh
rpm -ivh --force --nodeps libffi-devel-3.0.13-18.el7.x86_64.rpm
```
### No module named pip || No module named 'pip._internal.cli.main'
- https://bootstrap.pypa.io/get-pip.py
```sh
# windows上这招一般
# python -m ensurepip
python get-pip.py --force-reinstall 
```

# 卸载python3
```sh
# yum list installed | grep python
rpm -qa | grep python3 | xargs rpm -ev --allmatches --nodeps
# 删除所有残余文件
whereis python3
```