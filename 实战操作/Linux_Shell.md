## hdfs_sql_ftp
```shell script
##! /bin/bash
exectime=`date -d "0 day" +%Y%m%d`
if [ ! -n "$1" ]; then
    exectime=2019-01-01
    echo "设定默认日期 ${exectime}"
else 
    exectime=$1
    echo "设定起始日期 ${exectime}"
fi

if [ ! -n "$2" ]; then
    times=1
    echo "设定循环次数 ${times}"
else 
    times=$2
    echo "指定循环次数 ${times}"
fi

for((i=0;i<${times};i++))
do

mysql -u root -phadoop -A << EOF
...
..
.

EOF

hadoop fs -getmerge hdfs://hacluster/tenants/bingo_lhjg/ps.db/phonect_all_jm_yh /home/***.txt
echo "${logtime} 文件生成,准备切分"
split -l 150000000 /home/***.txt -a l ***

##如果文件不存在则新建一个空文件
if [[ ! -f "/home/down_file/***.txt" ]]; then 
touch /home/down_file/***.txt
fi

ftp -n<<!
open 10.245.33.22
user sz_jishi2018 sz_jishi@2018&
binary
cd /PAAS/DOWN_FILE/LGBIGDATE
lcd /home/down_file
prompt
get ***.txt ***.txt
mdelete ***.txt ***.txt
close
bye
!

rm -rf ***.txt

echo "${logtime} 传输完成"

done
```

## s3_delete
```shell script
##! /bin/bash
## 切忌: 请在一个空目录执行,程序会执行 rm -rf cqdsjb_*

## 使用方法: sh s3_delete_lines.sh cqbzk/share/test/ 'A5000000000002016055001|A5000755200002017035002|A5000755200002017015004|A5000755200002017025007'

## 第一个参数为S3路径
if [ ! -n "$1" ]; then
        echo "[ERROR] 请设定s3获取路径" 
        exit
else
        s3_addr=$1
        echo "[INFO] s3获取路径: ${s3_addr}"
fi

## 第二个参数为要匹配删除的关键字
if [ ! -n "$2" ]; then
        echo "[ERROR] 请设定要匹配的关键字"
        exit
else
        keywords=$2
        echo "[INFO] 关键字: ${keywords}"
fi

## 执行前先删除类似于s3文件前缀的文件
rm -rf cqdsjb_*

echo "[INFO] 下载S3文件: s3cmd get s3://${s3_addr}*"
s3cmd get s3://${s3_addr}*

## 把输入的关键字和匹配出的文件名都存入文件中
echo "[INFO] 匹配包含关键字的文件: grep -E '${keywords}' * | awk -F "[:]" '{print $1}' | grep 'cqdsjb_*' | sort | uniq > cqdsjb_files.txt"
grep -E ${keywords} * | awk -F "[:]" '{print $1}' | grep 'cqdsjb_*' | sort | uniq > cqdsjb_files.txt
echo ${keywords} | sed "s/'//g" > cqdsjb_keywords.txt

##cat cqdsjb_keywords.txt | awk -F "[|]" '{for (i=0;i<=NF;i++) print $i}'

echo "[INFO] 文件名去重后开始删除条目..."
for i in `cat cqdsjb_files.txt`
do
        echo "[INFO] ${i}删除前行数: " ` wc -l ${i} | awk '{print $1}'`
        for j in `cat cqdsjb_keywords.txt | sed 's/|/\n/g'`
        do
                ##echo "sed -i '/${j}/'d ${i}"
                ##一次删除多个 sed -i '/***/d;/***/d' ${i}
                sed -i "/${j}/d" ${i}
        done
        echo "[INFO] ${i}删除后行数: " ` wc -l ${i} | awk '{print $1}'`
        echo "[INFO] 文件放回S3: " `s3cmd put ${i} s3://${s3_addr}`
done

echo "[SUCCESS] 程序执行成功^_^"
```

## s3_count
```shell script
##! /bin/bash
## 第一个参数为S3路径
if [ ! -n "$1" ]; then
        echo "[ERROR] 请设定s3获取路径" 
        exit
else
        s3_addr=$1
        echo "[INFO] s3获取路径: ${s3_addr}"
fi

## 执行前先删除类似于s3文件前缀的文件
rm -rf cq*

echo "[INFO] 下载S3文件: s3cmd get s3://${s3_addr}*"
s3cmd get s3://${s3_addr}*

## 建立一个临时文件,把所有文件合并到一起
rm -rf temp_s3.txt
touch temp_s3.txt

for i in `ls cq*`
do
        cat ${i} >> temp_s3.txt
done

count=`wc -l temp_s3.txt | awk '{print $1}'`
fields=`cat temp_s3.txt | awk -F '[\001]' '{print NF}' | uniq`
echo "[INFO] 合并后文件有: ${fields} 个字段,共 ${count} 行,空行率如下: "

x=1
while [ ${x} -le ${fields} ]
do
        ##echo ${x}
        ##cat temp_s3.txt | awk -F '[\001]' '{print $'${x}'}' | grep -Ev '^$'| wc -l | awk '{print $1}'
        field_count=`cat temp_s3.txt | awk -F '[\001]' '{print $'${x}'}' | grep -Ev '^$'| wc -l | awk '{print $1}'`
        ##echo "field_count: $field_count"
        ##echo "count: $count"
        ##echo "空行率: `echo "scale=2;(( ${count} - $field_count) /${count})" | bc`"
        echo "scale=2;(( ${count} - $field_count) /${count})" | bc
        let x++
done
```

## zookeeper-start-all.sh
```shell script
##!/bin/bash

echo "Starting ZkServer..."
for i in 1,2,3
do 
    ssh by20$i "source ~/.bash_profile;/opt/mysoft/zookeeper-3.5.1-alpha/bin/zkServer.sh start"
done

elasticsearch-start-all
##!/bin/bash

## 耗时
startTime=`date +%s`
time_consuming(){
    expr `date +%s` - ${startTime}
}

echo "Starting Elasticsearch cluster ........"
for ip in {1..3};
do
    echo "[192.168.1.20${ip}] : Starting"
    ssh 192.168.1.20${ip} "source ~/.bash_profile;su elasticsearch -c 'elasticsearch -d'"
done

for ip in {1..3};
do
    echo "[192.168.1.20${ip}] : jps"
    ssh by20${ip} "source ~/.bash_profile;jps"
done

echo "Time consuming `time_consuming` seconds"
```

## elasticsearch-stop-all
```shell script
##!/bin/bash

## 耗时
startTime=`date +%s`
time_consuming(){
    expr `date +%s` - ${startTime}
}

## 获取IP下所有es节点进程
get_pid(){
    ssh $1 "ps -ef | grep elasticsearch | grep -w 'elasticsearch' | grep -v 'grep'" | grep 'Elasticsearch -d' | awk '{print $2}'
}

echo "Stoping Elasticsearch cluster ........"
for ip in {1..3};
do
    echo "[192.168.1.20${ip}] : Stoping"
    for i in `get_pid 192.168.1.20${ip}`; 
    do
        ssh 192.168.1.20${ip} "kill -9 ${i}"
    done
done

for ip in {1..3};
do
    echo "[192.168.1.20${ip}] : jps"
    ssh by20${ip} "source ~/.bash_profile;jps"
done

echo "Time consuming `time_consuming` seconds"
```

## es-restart
```shell script
##! /bin/bash
startTime=`date +%s`

## es集群IP信息列表
ip_list=(daases62 daases63 daases64 daases65 daases66 daases67 daases133 daases140 daases148 daases149 daases150 daases151 daases242 daases244 daases245 daases246 )
## 节点启动位置列表
addr_list=(/data1/elasticsearch_9200 /data2/elasticsearch_9400 /data3/elasticsearch_9600)
## 节点列表
port_list=(9200 9400 9600)
## 节点进程id
pid=""
## 当前健康节点
health_nodes=""
## 耗时
time_consuming(){
    expr `date +%s` - ${startTime}
}
## 获取IP下所有es节点进程
get_pid(){
    ssh ${1} "ps -ef | grep elasticsearch | grep -w 'elasticsearch' | grep -v 'grep'" | grep ${2} | awk '{print $2}'
}
## 获取es集群健康节点数量
es_colony_health(){
    curl -sX get http://admin:bingo_20181025@77.1.22.65:9200/_cat/health?v | awk '{print $5}' | sed -n '2p'
}

echo "[Currently available nodes] `es_colony_health`"
## 第一层循环遍历集群IP
for ip in ${ip_list[*]}; 
do
    echo "[Enter IP] ${ip}"
    ## 第二层循环遍历节点
    for port in ${port_list[*]}; 
    do
        ## 1.Kill节点
        pid=`get_pid ${ip} ${port}`
        echo "[Prepare kill ${ip}:${port} node] ssh ${ip} "kill -9 ${pid}""
        ssh ${ip} "kill -9 ${pid}"
        ## 2.启动节点
        number=`expr ${port:1:1} / 2 - 1`
        echo "[Prepare start ${ip}:${port} node] ssh ${ip} "su - daases -c sh ${addr_list[${number}]}/bin/elasticsearch –d""
        ssh ${ip} "su - daases -c 'sh ${addr_list[${number}]}/bin/elasticsearch –d'"
        ## 3.该节点注册进集群再重启下一个节点
        health_nodes=`es_colony_health`
        while [[ ! ${health_nodes} == 48 ]]; 
        do
            health_nodes=`es_colony_health`
            echo "[Currently available nodes ${health_nodes} ] Waiting to start ${ip}:${port} sleep 30 seconds"
            sleep 30
        done
        echo "[Start Success ${ip}:${port}] Prepare enter the next node"
    done
done

echo "[Process exectute success^_^] time consuming `time_consuming` seconds"
```

## gp_delete_log
```shell script
## /bin/sh
start_time=`date +%s`
gpip_list=(77.1.33.1 77.1.33.2 77.1.33.3 77.1.33.4 77.1.33.5 77.1.33.6 77.1.33.7 77.1.33.8 77.1.33.9 77.1.33.10 77.1.33.11 77.1.33.1 77.1.33.13 77.1.33.14)
used_space=""


time_commsuming(){
    expr `date +%s` - ${start_time}
}

for gpip in ${gpip_list[*]}; 
do
    echo "[Prepare execute] ssh ${gpip} df -h | sed -n '2p' | awk '{print $3}'"
    used_space=`ssh ${gpip} "df -h | sed -n '2p'" | awk '{print $3}'`
    echo "[ $gpip used_space ]: $used_space"
    if [ ${used_space%?} -ge 100 ]; 


            
    then
        echo "[Prepare execute] ssh ${gpip} find /home/lotus/instance_dw/log/ -mtime +15 -name "*.log" -exec rm -rf {} \;"
        ssh ${gpip} "find /home/lotus/instance_dw/log/ -mtime +15 -name "*.log" -exec rm -rf {} \;"
        used_space=`ssh ${gpip} "df -h | sed -n '2p'" | awk '{print $3}'`
        echo "[Now $gpip used_space ]: $used_space"
    fi
done

echo "[Execute compute]: comsuming time `time_commsuming` seconds^_^"
```

## storm_stop_all
```shell script
##!/bin/bash

startTime=`date +%s`
time_consuming(){
    expr `date +%s` - ${startTime}
}

echo "Stop storm cluster ........"
for ip in {1..3};
do
    ipaddr=`ssh by20${ip} ifconfig | grep inet | grep -v 'grep' | awk -F "[: ]+" '{print $3}' | head -n1`
    nimbus_pid=`ssh by20${ip} ps -ef | grep daemon.nimbus | grep -v 'grep' | awk '{print $2}'`
    logviewer_pid=`ssh by20${ip} ps -ef | grep daemon.logviewer | grep -v 'grep' | awk '{print $2}'`
    supervisor_pid=`ssh by20${ip} ps -ef | grep daemon.supervisor | grep -v 'grep' | awk '{print $2}'`
    ui_pid=`ssh by20${ip} ps -ef | grep daemon.ui.UIServer | grep -v 'grep' | awk '{print $2}'`
    echo "[${ipaddr}] : Stoping"
    ## echo "nimbus_pid : ${nimbus_pid}"
    if [ ! -z ${nimbus_pid} ];
    then
        for i in ${nimbus_pid}; do
            ssh by20${ip} "kill -9 ${i}"
        done
    fi
    ## echo "logviewer_pid : ${logviewer_pid}"
    if [ ! -z ${logviewer_pid} ];
    then
        for i in ${logviewer_pid}; do
            ssh by20${ip} "kill -9 ${i}"
        done
    fi
    ## echo "supervisor_pid : ${supervisor_pid}"
    if [ ! -z ${supervisor_pid} ];
    then
        for i in ${supervisor_pid}; do
            ssh by20${ip} "kill -9 ${i}"
        done
    fi
    ## echo "ui_pid : ${ui_pid}"
    if [ ! "" = "${ui_pid}" ];
    then
        for i in ${ui_pid}; do
            ssh by20${ip} "kill -9 ${i}"
        done
    fi
    ssh by20${ip} "source ~/.bash_profile;jps"
done
echo "Time consuming `time_consuming` seconds"
```

## storm_start_all
```shell script
##!/bin/bash

startTime=`date +%s`
time_consuming(){
    expr `date +%s` - ${startTime}
}

echo "Starting storm cluster ........"
for ip in {1..1};
do
    echo "[192.168.1.20${ip}] : Starting nimbus,ui"
    ssh by20${ip} "source ~/.bash_profile;nohup storm nimbus > /opt/mysoft/apache-storm-2.1.0/logs/nimbus.log 2>&1 &"
    ssh by20${ip} "source ~/.bash_profile;nohup storm ui > /opt/mysoft/apache-storm-2.1.0/logs/ui.log 2>&1 &"
done

for ip in {1..3};
do
    echo "[192.168.1.20${ip}] : Starting logviewer"
    ssh by20${ip} "source ~/.bash_profile;nohup storm logviewer > /opt/mysoft/apache-storm-2.1.0/logs/logviewer.log 2>&1 &"
done

for ip in {2..3};
do
    echo "[192.168.1.20${ip}] : Starting supervisor"
    ssh by20${ip} "source ~/.bash_profile;nohup storm supervisor > /opt/mysoft/apache-storm-2.1.0/logs/supervisor.log 2>&1 &"
done

for ip in {1..3};
do
    echo "[192.168.1.20${ip}] : jps"
    ssh by20${ip} "source ~/.bash_profile;jps"
done

echo "Time consuming `time_consuming` seconds"
```
