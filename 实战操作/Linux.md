# grep
```shell script
# 在目录或文件中查找内容关键字
grep apache-hive-3.1.2-bin -rl /opt/software/
```
# sed
```shell script
# 替换文件指定字符,如果是 / 记得转义
sed -i "s/\/mysoft/\/software/g" ~/.bash_profile 

# 替换文件夹下所有文件指定字符
sed -i "s/\/mysoft/\/software/g" `grep /mysoft -rl /opt/software/kafka_2.12-2.4.1/config`
```
# 正则
```shell script
# 匹配括号及其内容
\(.+\)
\([^\)]*\)

# 匹配换行符
\s+$
```

# 彩色字体
```shell script
echo -e "\033[30m 黑色字 \033[0m"
echo -e "\033[31m 红色字 \033[0m"
echo -e "\033[32m 绿色字 \033[0m"
echo -e "\033[33m 黄色字 \033[0m"
echo -e "\033[33m INFO \033[0m"
echo -e "\033[34m 蓝色字 \033[0m"
echo -e "\033[35m 紫色字 \033[0m"
echo -e "\033[36m 天蓝字 \033[0m"
echo -e "\033[37m 白色字 \033[0m"
echo -e "\033[40;37m 黑底白字 \033[0m"
echo -e "\033[41;37m 红底白字 \033[0m"
echo -e "\033[42;37m 绿底白字 \033[0m"
echo -e "\033[43;37m 黄底白字 \033[0m"
echo -e "\033[44;37m 蓝底白字 \033[0m"
echo -e "\033[45;37m 紫底白字 \033[0m"
echo -e "\033[46;37m 天蓝底白字 \033[0m"
echo -e "\033[47;30m 白底黑字 \033[0m"
```

# 一些报错信息 
## $'\r': command not found
```shell script
vim  **.sh
: set ff = unix
```
