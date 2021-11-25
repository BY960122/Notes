# Logstash 
> 是一个开源的服务器端数据处理管道,允许您在将数据索引到 Elasticsearch 之前同时从多个来源采集数据,并对数据进行充实和转换。

# Kibana
是一款适用于 Elasticsearch 的数据可视化和管理工具,可以提供实时的直方图、线形图、饼状图和地图

# Elasticsearch
- https://github.com/YVictor13/ElasticSearch-study/blob/master/src/Elasticsearch%20%E6%9E%B6%E6%9E%84%E8%AE%BE%E8%AE%A1%E5%8F%8A%E8%AF%B4%E6%98%8E.md
> 是一个基于 Lucene 的分布式搜索引擎,能够支持准实时的数据检索NRT(near real-time),包括结构化和非结构化数据
> 他能够解决传统数据库解决不了的复杂查询,计算,聚合等操作,还有时序数据的处理,比如日志处理、监控数据的存储、分析和可视化等

# Elasticsearch 架构
> 1.Master: 用于管理和控制Elasticsearch集群,并负责在集群范围内创建删除索引,跟踪哪些节点是集群的一部分,并将分片分配给这些节点.
- node.master=true 表示该节点有资格成为master节点
> 2.数据节点(Data Node): 保存数据并执行与数据相关的操作,如CRUD、搜索和聚合
- node.data=true,表示每个节点都为一个data节点
> 3.摄取节点(或者预处理节点）: 能够在索引操作之前对document进行增强或者丰富等预处理操作,该操作发生在bulk和index之前,通过pipeline和processor的方式实现,非常类似于logstash的filter
- node.ingest=true

# Allocation
> 是指将分片分配给某个节点的过程,包括分配主分片或者副本,如果是副本,还包含从主分片复制数据的过程。
> Allocation操作发生在故障恢复,副本分配,Rebalancing,节点的新增和删除等场景,master节点的主要职责就是决定分片分配到哪一个节点,并且做分片的迁移来平衡集群
> Shard allocation filters: 允许执行分片分配之前,前置一些filter操作
- 例如计划下线一个node,可以设置不允许给该node分配分片
```json
// PUT _cluster/settings
{
  "transient" : {
    "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
  }
}
```
> es在进行shard allocation的时候,会充分考虑每一个node的可用磁盘空间,具体有如下配置:
- cluster.routing.allocation.disk.threshold_enabled:默认是true,如果是false会禁用基于disk的考虑
- cluster.routing.allocation.disk.watermark.low:控制磁盘使用率的低水位,默认是85%,如果一个节点的磁盘空间使用率已经超过了85%,那么就不会分配shard给这个node了
- cluster.routing.allocation.disk.watermark.high:控制磁盘使用率的高水位,默认是90%,如果一个节点的磁盘空间使用率已经超过90%了,那么就会将这个node上的部分shard移动走
- cluster.info.update.interval:es检查集群中每个node的磁盘使用率的时间间隔,默认是30s
- cluster.routing.allocation.disk.include_relocations:默认是true,意味着es在计算一个node的磁盘使用率的时候,会考虑正在分配给这个node的shard。

# Elasticsearch mapping
> 1.ES的mapping非常类似于静态语言中的数据类型: 声明一个变量为int类型的变量,以后这个变量都只能存储int类型的数据
> 2.它还告诉ES如何索引数据以及数据是否能被搜索到。当你的查询没有返回相应的数据,你的mapping很有可能有问题。当你拿不准的时候,直接检查你的mapping
> 3.一个mapping由一个或多个analyzer(分词器)组成, 一个analyzer又由一个或多个filter组成的,逐级传递
> 4.总结来说,mapping的作用就是执行一系列的指令将输入的数据转成可搜索的索引项,一般是些单词

# Elasticsearch 写入过程(借助事物)
> es的准实时在于在写入数据的时候,采用内存buff+文件系统缓存+磁盘三级结构,确保写入的数据能够近实时的被检索到
> 1.新收到的数据存放于内存buff中,同时将数据写入 translog 日志文件,这个时候是检索不到的
> 2.数据从内存buff刷新到文件系统缓存中(OS cache),refresh默认1s,生成一个新的segment,清空内存数据,这个时候可以被检索到了
> 3.默认每隔30分钟或者 translog 大于512MB,执行一个flush操作,首先将commit point写入磁盘文件,标识这个commit point对应的所有的segment file,同时强行将os cache中目前所有的数据都fsync到磁盘文件中去。最后清空现有 translog 日志文件,重启一个 translog ,此时commit操作完成。

# Elasticsearch 节点加入与离开
> 1.节点加入: 当一个新节点加入的时候,它通过discovery.zen.ping.unicast.hosts配置的节点获取集群状态,然后找到master节点,发送一个join request(discovery.zen.join_timeout)。主节点接收到reqest后,同步集群状态到新节点。
> 2.非主节点节点离开: 当一个节点出现3次ping不通的情况时(ping_interval 默认为1sping_timeout 默认为30s),主节点会认为该节点已宕机,将该节点踢出集群。
> 3.主节点离开: 当主节点发生故障时,集群中的其他节点将会 ping 当前的master eligible 节点,并从中选出一个新的主节点。
- 选主过程: 
- (1).节点启动之后,从配置文件获取集群列表,discovery.zen.ping.unicast.hosts: ["host1", "host2"]
- (2).Ping的response会包含该节点的基本信息以及该节点认为的master节点,先从各节点认为的master中选,按照节点id排序,取第一个。
- (3).如果各节点都没有认为的master,则从所有节点中选择,规则同上。
- (4).通过 discovery.zen.minimum_master_nodes 这个参数设置为(N/2)+1,避免因为网络分区而产生脑裂。

# 分片
> 1.由于Elasticsearch中,在一个多分片的索引中写入数据时,需要通过路由来确定具体陷入哪一个分片中,所以在创建索引时需要指定分片数量,
配置副本,并且分片的数量一旦确定就不能修改。
- 分片算法: shard_num = hash(_routing) % num_primary_shards  #_routing默认为id字段或者parent字段
- 分片: index.number_of_shards:5
- 副本: index.number_of_replicas:1
> 2.对文档的新建、索引和删除请求等写操作,必须在主分片上面完成之后才能被复制到副本分片
> 3.Elasticsearch 使用乐观锁来控制加快写入速度的并发写入引起的数据冲突问题,通过为每个文档设置一个version(版本号),当文档被修改时版本号递增来实现。
> 4.当向Elasticsearch写入数据时,Elasticsearch根据文档标识符ID将文档分配到多个分片上,当查询数据时,Elasticsearch会查询所有的分片并汇总结果。
> 5.对于分片,用户并不知道数据存在哪个分片上

# 路由
> 为了避免查询时部分分片查询失败影响结果的准确性,Elasticsearch引入了路由功能。
> 当数据写入时,通过路由将数据写入指定分片
> 当查询数据时,通过相同的路由指明在哪个分片将数据查出来

# Elasticsearch 数据一致性
> ES的数据写入一致性由 translog 保证
>  translog 文件本身也是先写os cache,然后再刷到磁盘的,默认5s,如果宕机,可能会有5s的数据都是,当然不如不考虑性能的影响,可以设置每次写操作都刷到磁盘
> 副本一致性则采用半同步的方式,即主分片写入之后,只要一定数量副分片返回写入成功,则返回客户端成功
> wait_for_active_shards的默认值为int( (primary + number_of_replicas) / 2 ) + 1

# Elasticsearch 索引
> 倒排索引（Inverted Index）也叫反向索引,有反向索引必有正向索引。通俗地来讲,正向索引是通过key找value,反向索引则是通过value找key。
> 倒排索引会列出在所有文档中出现的每个特有词汇,并且可以找到包含每个词汇的全部文档
