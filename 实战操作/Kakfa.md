# Kafka 官网 demo
- https://cwiki.apache.org/confluence/display/KAFKA/

## 启动kafka服务节点
```sh
nohup /opt/software/kafka_2.12-2.8.1/bin/kafka-server-start.sh /opt/software/kafka_2.12-2.8.1/config/server.properties >> /opt/logs/kafka-logs/server.logs 2>&1 &

## 终止kafka相关进程
jps | grep -e SupportedKafka | awk '{print $1}' | xargs kill -9
```

## 生产者吞吐量测试
```sh
bin/kafka-producer-perf-test.sh --num-records 100000 --topic test --producer-props bootstrap.servers=192.168.1.201:9092,192.168.1.203:9092,192.168.1.202:9092 --throughput 5000 --record-size 102400
```

## 消费者吞吐量测试
```sh
/bin/kafka-consumer-perf-test.sh --topoc test --messages 100000 --num-fetch-threads 10 --threads 10 --broker-list 192.168.1.201:9092,192.168.1.203:9092,192.168.1.202:9092 --group perf-consumer-30108
```

## Topic
```sh
## 创建topic
## 注意: zookeeper写一个也可以
kafka-topics.sh --create --topic bingo --replication-factor 3 --partitions 3 --zookeeper by201:2181

## 查看topic的详情
kafka-topics.sh --describe --bootstrap-server 127.0.0.1:9092 --zookeeper 192.168.1.201:2181 --topic bingo 

kafka-topics.sh --list --bootstrap-server 127.0.0.1:9092 --zookeeper 192.168.1.201:2181 --topic test

## 删除topic
## 注意:此种删除方式仅仅只是逻辑形式上的删除,其topic的数据并没有得到真正的删除,要想真正的删除topic中的数据,只能是到kafka数据存放目录下去一一删除,
## 且上zk上去将topic的元数据进行删除,kafka的元数据存放在zk的/config目录下
kafka-topics.sh --delete --zookeeper by201:2181,by211:2181,by212:2181 --topic bingo 

## 生产者:可选操作 --producer-property security.protocol=SASL_PLAINTEXT --producer-property  sasl.mechanism=PLAIN
bin/kafka-console-producer.sh --broker-list B1:9092 --sync --topic test

## 生产数据
nohup /home/kafka/20170620xdh/kafka_perf_test_jar/start-perf-produce.sh test_10b_3t_mirrormaker 100000000 10 10 >/tmp/test_10b_3t_mirrormaker.out 2>&1 &

## 消费者:可选操作 --producer-property security.protocol=SASL_PLAINTEXT --producer-property  sasl.mechanism=PLAIN
bin/kafka-console-consumer.sh --zookeeper B1:2181,B2:2181,B3:2181 --topic test --from-beginning

## 统计行号,topic数量
/kafka-topics --list --zookeeper B1:2181 | wc -l

## 描述主题
/data01/confluent/bin/kafka-topics --describe --topic 441900_kk_t_tracklog_mirrormaker  --zookeeper localhost:2181 

## 删除主题
/data1/confluent/bin/kafka-topics --delete --topic zntszs_cq_zdryyj --zookeeper B1:2181

## 主题授权
/data1/confluent/bin/kafka-acls --authorizer-properties zookeeper.connect=B1:2181 --add --allow-principal User:mirror_maker_user --consumer --topic 440300_wb_yycs_mirrormaker --group '*'

## 生产授权
/data1/confluent/bin/kafka-acls --authorizer-properties zookeeper.connect=B1:2181 --add --allow-principal User:mirror_maker_user --producer --topic shantou_mirror_maker_1

## 回收权限
/data01/confluent/bin/kafka-acls --authorizer-properties zookeeper.connect=B1:2181 --remove --allow-principal User:stkxc --consumer --topic 440000 _gd_parkinglot_mirrormaker --group '*'

## 查看主题权限
/data1/confluent/bin/kafka-acls --authorizer-properties zookeeper.connect=B1:2181 --list  --topic bingocc_lhw123

## 查看消费者组
/data01/confluent/bin/kafka-consumer-groups --list --bootstrap-server kafka104:9094,kafka105:9094,kafka106:9094,kafka107:9094,kafka108:9094  --command-config /data01/kafka-consumer-group.properties | grep 441700_kk_veh_passrec

## 描述消费者组
/data01/confluent/bin/kafka-consumer-groups --describe --group mirror-maker --bootstrap-server kafka104:9094,kafka105:9094,kafka106:9094,kafka107:9094,kafka108:9094  --command-config /data01/kafka-consumer-group.properties

## 查看主题最新数据
/data01/.version/kafkacat-1.3.1/kafkacat -b kafka104:9094 -X security.protocol=sasl_plaintext -X sasl.mechanisms=PLAIN -X sasl.username=bingo -X sasl.password=bingo -X api.version.request=true -t 441900_ly_t_forec_mirrormaker -f '%s|%o|%t[%p]\n'  -C -o -1 -c 1 -e

## 查看group的offset
./kafka-consumer-groups --describe --group mirror-maker --bootstrap-server kafka104:9094,kafka105:9094,kafka106:9094,kafka107:9094,kafka108:9094 --command-config /data01/kafka-consumer-group.properties | topicname

## 查看指定时间段得指定topic的offset
java -jar /data01/.version/kafkaShell/countOffset/countOffset.jar /data01/.version/kafka_client_jaas.conf kafka104:9094  441900_kk_t_jcbk_mirrormaker lwl_20171027_007225622 20171205140000 20171206140000
```

## 程序操作
```sh
## 安装Kafka包
pip3 search kafka 
    kafka(1.3.5)
sudo /usr/local/bin/pip3 install kafka

>>> import kafka
```
## 生产者程序.py
```py
import json
import time

from kafka import KafkaProducer

producer = KafkaProducer(bootstarp_servers=["192.168.1.201:9092","192.168.1.202:9092","192.168.1.203:9092"],
                    retries=5,acks=1)
                            (几个副本发送确认信息)
for number in range(10):
    time.sleep(0.5)
    message = "Hello {0}".format(number)
    future = producer.send("test",json.dumps(message).encode("utf-8"))
    result = future.get(timeout=100)
    print (result.topic,result.partition,result.offset)
```
## 消费者程序.py
```py
from Kafka import KafkaConsumer
 // group_id指定了所属群组,如果多个消费者进程同属一个群组,他们之间不会出现重复数据
consumer = KafkaConsumer("test",bootstarp_server=["192.168.1.201:9092","192.168.1.202:9092","192.168.1.203:9092"],
            group_id="BY")

for message in consumer:
    print (message.topic,message.partition,message.offset,message.key,message.value)
```

## 一些报错信息
### Kafka重启命令行报错:Port already in use:9999(Bind failed)
```sh
vim bin/kafka-run-class.sh

##Changed JMX port use
if [[ $ISKAFKASERVER = "true"]]; then
    JMX_REMOTE_PORT=$JMX_PORT
else
    JMX_REMOTE_PORT=$CLIENT_JMX_PORT
fi
if [ $JMX_REMOTE_PORT ]; then
    KAFKA_JMX_OPTS="$KAFKA_JMX_OPTS -Dcom.sum.management.jmxremote.port=$JMX_REMOTE_PORT"
fi

## JMX port to use
##if [ $JMX_PORT ];then
##   KAFKA_JMX_OPTS="$KAFKA_JMX_OPTS -Dcom.sum.management.jmxremote.port=$JMX_PORT "
##fi
```