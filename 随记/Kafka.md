# Kafka 架构
> 1.Producer: 消息生产者
> 2.Consumer: 消息消费者
> 3.Consumer Group （CG）: 消费者组,组内每个消费者消费不同分区的数据,一个分区只能由一个组消费,组之间不影响.
> 4.Broker: 一台 Kafka 服务器就是一个 broker.
> 5.Topic: 一个队列,可以理解成一张表,
> 6.Partition: 物理分区,一个topic 被分为多个 partition,一个partition 被分为.log 和 .index 文件
> 7.Replica: partition 的副本
> 8.leader: 主副本,一个
> 9.follower: 从副本,N个
> 10.AR(Assigned Replicas): 总的分配副本
> 11.OSR(Out-of-Sync Replicas): 数据同步严重滞后的副本组成OSR
> 12.Controller: 负责管理集群broker的上下线，所有topic的分区副本分配和leader选举等工作。

# Kafka producer 分区的原则
> 1.指明 partition 的情况下直接放入对应的分区
> 2.没有指明 partition,但有 key 的情况下,将 key 的 hash 值与 topic 的 partition 数取余得到 partition 值.
> 3.既没有指明 partition,也没有 key,第一次调用时随机生成一个整数,将这个整数与可用的 partition 总数取余得到 partition 值.

# Kafka 分区分配策略
> 默认一个消费者订阅 topic,会分配到对应的所有分区,但有3种情况会涉及,分区的重新分配
> 1.consumer group 内新增 消费者
> 2.topic 新增了分区
> 3.消费者离开了当前组,例如 shutdown
> 设置分区分配策略可以用 partition.assignment.strategy
> 1.range 分区分配策略: 分区按序号排序,消费者线程按字母排序,分区数除以线程数均匀分配,多出来的前面几个消费者多消费一个(缺点)
> 2.RoundRobin 分区分配策略: 轮询算法,将组内所有topic的所有分区分配给每个消费者,有以下2个前提,否则会分配不均匀.
-- 1.同一个消费者组里的每个消费者订阅的主题必须相同
-- 2.同一个消费者组里面的所有消费者的num.streams(消费者个数)必须相等
> 3.Sticky 分区分配策略: 保留了再平衡之前的消费分配结果,避免资源损耗.主要为了实现下面2个目的
-- 1.分区的分配要尽可能的均匀
-- 2.分区的分配尽可能的与上次分配的保持相同

# Kafka 数据可靠性
> 1.leader 维护一个动态的 ISR,即与 leader 保持联系的 follower
> 2.当 follower 同步好数据后, leader 就会给 follower 发送 ack
> 3.如果 follower 长时间没有同步数据,则会被剔除 ISR,时间阀值由 replica.lag.time.max.ms 设定
> 4.leader 的选举也是从 ISR 列表中产生.

# ack 参数配置
> 0: producer 不等待 broker 的ack,如果 broker 发生故障,数据会丢失
> 1: producer 等待 partition 的 leader 副本返回 ack,如果 follower 同步成功之前,leader 故障,会丢数据
> -1: producer 等待所有 partition 的副本返回 ack,但如果在 follower 同步完成后,broker 返回 ack 以前,leader 发生故障,则会造成数据重复.

# Kafka 故障处理细节
> 1.HW: 指消费者能看到的最大 offset,ISR 中最小的 LEO(木桶效应,取最短的那个)
> 2.LEO: 每个副本最大的 offset
> follower 发生故障时,会被踢出ISR,待恢复后,会读取上次记录的HW,并将.log中高于这部分的数据去除,然后从HW位置向 leader 同步,等 follower 的 LEO 追上 HW,则加入 ISR
> leader 发生故障时,会从 ISR 中选一个出来.为保证副本数据的一致性,会将其他 follower 高于 HW 的数据全部去除,然后从新 leader 同步数据.

# Kafka 怎么体现消息的顺序
> 1.生产者: 通过分区的leader副本负责数据顺序写入,来保证消息顺序性
> 2.消费者: 同一个分区内的消息只能被一个 group 里的一个消费者消费,保证分区内消费有序
> 3.消息会发送到不一样的分区,Kafka 只能做到分区内有序,无法做到 全局有序
> 备注: 如果要做全局有序,最好只有一个分区,或者数据按顺序发送到分区,按顺序消费,单线程消费
> 4.max.in.flight.requests.per.connection=1 可以保证消息是按照发送的顺序写入服务器的,enable.idempotence=true,开启生产者的幂等生产,前面的配置可以大于1

# Kafka 拦截器,序列化器,分区器
> 1.拦截器: 
- 生产者拦截器: 用来在消息发送前做一些准备工作，比如按照某个规则过滤不符合要求的消息，修改消息的内容等
- 
> 2.序列化器: 
> 3.分区器: 计算分区号

# Kafka生产者客户端的整体结构是什么样子的?使用了几个线程来处理?分别是什么?
> 2个线程
> 1.main 线程: producer -> 拦截器 -> 序列化器 -> 分区器
> 2.sender 线程: 发送数据