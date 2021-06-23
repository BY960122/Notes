# 权限
```sh
# 所有人可读
chmod ugo+r **.**
```

# 三剑客
```sh
# /home/test.txt

# One day,a little monkey is playing by the well.一天,有只小猴子在井边玩儿.
# He looks in the well and shouts :它往井里一瞧,高喊道：
# “Oh!My god!The moon has fallen into the well!” “噢!我的天!月亮掉到井里头啦!”
# An older monkeys runs over,takes a look,and says,一只大猴子跑来一看,说,
# “Goodness me!The moon is really in the water!” “糟啦!月亮掉在井里头啦!”
# And olderly monkey comes over.老猴子也跑过来.
# He is very surprised as well and cries out:他也非常惊奇,喊道：
# “The moon is in the well.” “糟了,月亮掉在井里头了!”
# A group of monkeys run over to the well .一群猴子跑到井边来,
# They look at the moon in the well and shout:他们看到井里的月亮,喊道：
# “The moon did fall into the well!Come on!Let’get it out!”
# “月亮掉在井里头啦!快来!让我们把它捞起来!”
# Then,the oldest monkey hangs on the tree up side down ,with his feet on the branch .
# 然后,老猴子倒挂在大树上,
# And he pulls the next monkey’s feet with his hands.拉住大猴子的脚,
# All the other monkeys follow his suit,其他的猴子一个个跟着,
# And they join each other one by one down to the moon in the well.
# 它们一只连着一只直到井里.
# Just before they reach the moon,the oldest monkey raises his head and happens to see the moon in the sky,正好他们摸到月亮的时候,老猴子抬头发现月亮挂在天上呢
# He yells excitedly “Don’t be so foolish!The moon is still in the sky!”
# 它兴奋地大叫：“别蠢了!月亮还好好地挂在天上呢!

# Xiaoka       60   80    40    90   77  class-1
# Yizhihua     70    66   50    80   90  class-1
# kerwin       80    90   60    70   60  class-2
# Fengzheng    90    78   62    40   62  class-2
# xman         -     -     -     -   -    -
```
## grep
```sh
# 在目录或文件中查找内容关键字
grep apache-hive-3.1.2-bin -rl /opt/software/
# 查找符合条件的行
grep moon /home/test.txt
# 查找反向符合条件的行
grep -v moon /home/test.txt
# 查找符合条件的行数
grep -c moon /home/test.txt
# 忽略大小写查看
grep -i my /home/test.txt
# 查找符合条件的行并输出行号
grep -n monkey /home/test.txt
# 查找开头是J的行
grep '^J' /home/test.txt
# 查找结尾是呢的行
grep "呢$" /home/test.txt
```

## sed
```sh
# 替换文件指定字符,如果是 / 记得转义 /g:全局替换
sed -i "s/\/mysoft/\/software/g" ~/.bash_profile 

# 替换文件夹下所有文件指定字符
sed -i "s/\/mysoft/\/software/g" `grep /mysoft -rl /opt/software/kafka_2.12-2.4.1/config`

# 删除行
sed '2d' /home/test.txt
# 删除2-4行,并显示行号
sed '=;2,4d' /home/test.txt
# 向前插入
echo "hello" | sed 'i\kitty'
# 替换第二行为hello kitty
sed '2c\hello kitty' /home/test.txt
# 替换第二行到最后一行为hello kitty
sed '2,$c\hello kitty' /home/test.txt
# 把带star的行写入test2文件中,test2提前创建
sed -n '/star/w /home/test2.txt' /home/test.txt
# 打印3行后，退出sed
sed '3q' /home/test.txt
```

## awk
```sh
# 输出整个文本
awk '{print $0}' /home/test.txt
# 输出第一列
awk '{print $1}' /home/test.txt
# 给文本加入标题头
awk 'BEGIN {print "Name  Math\n-----"} {print $0}' /home/test.txt
# 仅打印姓名、数学成绩、班级信息，再加一个文尾(再接再厉):
awk 'BEGIN {print "Name   Math  grade\n-----"} {print $1 2 "\t" $7} END {print "continue to exert oneself"}' /home/test.txt
# 模糊匹配|查询已经分班的学生
awk '$0 ~/class/' /home/test.txt
# 精准匹配|查询1班的学生
awk '$7=="class-1" {print $0}'  /home/test.txt
# 反向匹配|查询不是1班的学生
awk '$7!="class-1" {print $0}'  students_store
# 查询数学大于80的
awk '$2>60 {print $0}'  students_store
# 查询数学大于英语成绩的
awk '$2 > $4  {print $0}'  students_store
# 关系匹配|查询1班和3班的学生
awk '$0 ~/(class-1|class-3)/' students_store
# 查询数学成绩大于60并且语文成绩也大于60
awk '{ if ($2 > 60 && $3 > 60) print $0}' students_store
# 查询数学大于80或者语文大于80
awk '{ if ($2 > 80 || $4 > 80) print $0}' students_store
```

# 正则
```sh
# 匹配括号及其内容
\(.+\)
\([^\)]*\)

# 匹配换行符
\s+$

# 匹配空格及其之后的内容
\ [^\s].*
[\ ].*
```

# 彩色字体
```sh
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

# 常用命令
```sh
# 查看端口占用
lsof -i:8088
# 只下载包不安装
	# 包括依赖
yum install -y --downloadonly --downloaddir=/opt/ lsof
yum reinstall -y --downloadonly --downloaddir=/opt/ lsof
	# 不包括依赖
yum install -y yum-utils
yumdownloader lsof
```

# 一些报错信息 

## $'\r': command not found
```sh
vim  ecif_etl_data_import.sh
: set ff = unix
# 或者 
yum -y install dos2unix
dos2unix ecif_etl_data_import.sh

```