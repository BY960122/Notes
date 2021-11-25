# Spark 和 Flink 的区别
- https://www.cnblogs.com/hdc520/p/13206443.html
> Spark的技术理念是使用微批来模拟流的计算,数据流以时间为单位被切分为一个个批次,通过分布式数据集RDD进行批量处理,是一种伪实时
> Flink是基于事件驱动的,是面向流的处理框架,Flink基于每个事件一行一行地流式处理,是真正的流式计算.另外他也可以基于流来模拟批进行计算实现批处理

# Spark 内存管理模型
- https://bbs.huaweicloud.com/blogs/detail/170627
> Spark是一个基于内存的分布式计算引擎,为了更为高效地利用内存,并减少OOM等内存问题,Spark对JVM内存模型进行了进一步的管理规划,MemoryManager
> 1.spark.executor.memory 指定的JVM堆内内存,将堆内内存逻辑上划分为三个主要内存区域
- Reserved Memory: 这一块是系统预留的内存,由 spark.testing.reservedMemory=300M 决定,(只存spark 内部对象)
- User Memory: 为用户编写的Spark应用逻辑而预留的内存,(Java Heap - Reserved Memory) * (1.0 - spark.memory.fraction)
- Spark Memory: MemoryManager实际管理的是Spark Memory这一部分内存区域,该部分内存区域大小由spark.memory.fraction=0.6控制
- (1).Storage Memory: 该部分内存可用于unroll RDD,缓存RDD,以及缓存broadcast等的传输数据,spark.memory.storageFraction=0.5
- (2).Execution Memory: 该部分内存用作shuffle,joins,sorts以及aggregations等操作的计算内存,由所有task共享.其确保每个task在进行spill之前,至少能获得1/2N的内存,但最多只能获取1/N的内存
> 2.spark.memory.offHeap.size 指定的堆外内存: Spark在JVM内存之外直接开辟的内存空间,用于存储经过序列化的二进制数据
- Execution Memory的空间被对方占用后,可让对方将占用的部分转存到磁盘,然后"归还"借用的空间。
- Storage Memory的空间被对方占用后,无法让对方"归还",因为需要考虑执行过程中的很多因素,实现起来较为复杂。

# Spark 核心组件
## Driver
> Spark驱动器节点,用于执行Spark任务中的main方法,负责实际代码的执行工作。Driver在Spark作业执行时主要负责：
> 1.将用户程序转化为作业(job)
> 2.在Executor之间调度任务(task)
> 3.跟踪Executor的执行情况
> 4.通过UI展示查询运行情况

## Executor
> Spark Executor节点是一个JVM进程,负责在 Spark 作业中运行具体任务,任务彼此之间相互独立
> 1. 负责运行组成Spark应用的任务,并将结果返回给驱动器进程；
> 2. 它们通过自身的块管理器(Block Manager)为用户程序中要求缓存的 RDD 提供内存式存储。RDD是直接缓存在Executor进程内的,因此任务可以在运行时充分利用缓存数据加速运算。

# Spark运行流程
## YarnClient
> Driver在任务提交的本地机器上运行
> 1.client会和 ResourceManager 通讯申请启动 ApplicationMaster
> 2.ResourceManager 分配 container,在合适的 NodeManager 上启动 ApplicationMaster
- ApplicationMaster 会单独启动 Driver 后台线程,Driver 主要初始化 SparkContext 对象
> 3.在 NodeManager 上启动 Executor 进程,然后向 Driver 反向注册
> 4.全部反向注册完成后,Driver开始执行 main 函数,执行到 action 算子的时候,触发一个 job,开始划分 stage,每个 stage 就是一个 taskset ,然后分发到 Executor 执行.

## YarnCluster
> Driver不一定在本地机器上运行

# RDD
> 1.弹性分布式数据集,是一个抽象类,它代表一个不可变,可分区,里面的元素可并行计算的集合。
> 2.一组分区(Partition),即数据集的基本组成单位;
> 3.rdd 之间有依赖关系
> 4.rdd本身不可改变,如果要改变,只能通过转换操作形成新的RDD

## 算子
> transformations,它是用来将RDD进行转化,构建RDD的血缘关系,比如:groupby,map,join,union,distinct,zip,combinerbykey

# SparkShuffle
> 主要是有shuffle类算子,比如: reduceByKey,GroupByKey,sortByKey,join,distinct,repartions
> Spark Shuffle分为 map 阶段和 reduce 阶段,或者称之为 ShuffleRead 阶段和 ShuffleWrite 阶段
> 输入时又文件的 split 个数对应 rdd 的 partition 个数,也就是 map 数量
> 经过一系列算子重新分区后,map 端的最后一个分区数对应的就是 reduce 数量,如果设置了spark.default.parallelism,那reduce按这个来
> actions,它是用来触发RDD的计算,得到RDD的相关计算结果或者将RDD保存的文件系统中,比如:take,collect,first,reduce,checkpoint

## Spark 宽窄依赖
> 宽依赖: 父RDD的分区数据划分到子RDD的多个分区,表名有 shuffle 算子,比如: reduceByKey,GroupByKey,sortByKey,join,distinct,repartions
> 窄依赖: 父RDD的每个分区的数据直接到子RDD的对应一个分区,没有 shuffle 过程

# SparkSql
## DataFrame
> 与RDD类似,DataFrame也是一个分布式数据容器,然而DataFrame更像传统数据库的二维表格,除了数据以外,还记录数据的结构信息,即schema
## DataSet
> 是Dataframe API的一个扩展,是Spark最新的数据抽象,Dataframe是Dataset的特列,DataFrame=Dataset[Row] 
> DataFrame只是知道字段,但是不知道字段的类型,所以在执行这些操作的时候是没办法在编译的时候检查是否类型失败的,比如你可以对一个String进行减法操作,在执行的时候才报错,而DataSet不仅仅知道字段,而且知道字段类型,所以有更严格的错误检查。

# RDD,DataFrame,DataSet
## 相同点
> 1.RDD、DataFrame、Dataset全都是spark平台下的分布式弹性数据集,为处理超大型数据提供便利
> 2.三者都有惰性机制,在进行创建、转换,如map方法时,不会立即执行,只有在遇到Action如foreach时,三者才会开始遍历运算。
> 3.三者都会根据spark的内存情况自动缓存运算,这样即使数据量很大,也不用担心会内存溢出。
> 4.三者都有partition的概念
> 5.三者有许多共同的函数,如filter,排序等
> 6.在对DataFrame和Dataset进行操作许多操作都需要这个包进行支持,import spark.implicits._
> 7.DataFrame 和 Dataset 均可使用模式匹配获取各个字段的值和类型

## 不同点
> 1.RDD不支持sparksql操作
> 2.与RDD和Dataset不同,DataFrame每一行的类型固定为Row,每一列的值没法直接访问,只有通过解析才能获取各个字段的值

# SparkStreaming
> 用于流式数据的处理,使用离散化流(discretized stream)作为抽象表示,叫作DStream,在内部,每个时间区间收到的数据都作为 RDD 存在,而DStream是由这些RDD所组成的序列(因此得名“离散化”)


# 优化
## 优化思路
> 1.尽可能多的分配资源,例如一共有20台 worker 节点,每台节点8g内存,10个cpu。那么可以分配 20个 exeuctor,每个 executor 内存8g,使用8个cpu
> 2.设置并行度: spark作业中,各个 stage 的task的数量也就代表了作业在各个阶段stage的并行度,官方推荐,task数量,设置成总core数量的2到3倍
> 3.rdd重用: 多次用到的rdd进行持久化,避免重复计算,方法: rdd.cache,rdd.persist(StorageLevel.MEMORY_ONLY)
> 4.小表直接广播变量
> 5.避免使用shuffle类算子,例如本来join的,改成小表广播变量,然后大表.map(广播变量)
> 6.使用map-side预聚合(类似于combiner),建议使用reduceByKey或者aggregateByKey算子来替代掉groupByKey算子。因为reduceByKey和aggregateByKey算子都会使用用户自定义的函数对每个节点本地
的相同key进行预聚合。而groupByKey算子是不会进行预聚合的,全量的数据会在集群的各个节点之间分发和传输,性能相对来说比较差。
> 7.使用高性能算子,使用mapPartitions替代普通map,mapPartitions类的算子,一次函数调用会处理一个partition所有的数据,而不是一次函数调用处理一条,性能相对来说会高一些。但要注意内存够用.
> 8.使用高性能算子,使用foreachPartition替代foreach
> 9.使用高性能算子,使用filter之后进行coalesce操作,手动减少RDD的partition数量
> 10.使用高性能算子,使用 repartitionAndSortWithinPartitions 替代 repartition 与 sort 类操作
> 11.使用序列化,Spark支持使用Kryo序列化机制。Kryo序列化机制,比默认的Java序列化机制,速度要快,序列化后的数据要更小,大概是Java序列化机制的1/10。所以Kryo序列化优化以后,可以让网络传输的数据变少；在集群中耗费的内存资源大大减少。注册要序列化的自定义类型。

## 参数调优
> 1.Hive 参数对 Spark 同样起作用。
> set hive.exec.dynamic.partition=true; // 是否允许动态生成分区
> set hive.exec.dynamic.partition.mode=nonstrict; // 是否容忍指定分区全部动态生成
> set hive.exec.max.dynamic.partitions = 100; // 动态生成的最多分区数
> 2.运行行为
> set spark.sql.shuffle.partitions; // 需要shuffle是mapper端写出的 partition 个数,也就是 task 数量
> set spark.sql.autoBroadcastJoinThreshold; // 大表 JOIN 小表,小表做广播的阈值
> set spark.dynamicAllocation.enabled; // 开启动态资源分配
> set spark.dynamicAllocation.maxExecutors; //开启动态资源分配后,最多可分配的Executor数
> set spark.dynamicAllocation.minExecutors; //开启动态资源分配后,最少可分配的Executor数
> set spark.sql.adaptive.enabled; // 是否开启调整partition功能,如果开启,spark.sql.shuffle.partitions设置的partition可能会被合并到一个reducer里运行
> set spark.sql.adaptive.shuffle.targetPostShuffleInputSize; //开启 spark.sql.adaptive.enabled 后,两个partition的和低于该阈值会合并到一个 reducer
> set spark.sql.adaptive.minNumPostShufflePartitions; // 开启 spark.sql.adaptive.enabled后的最小的分区数
> set spark.hadoop.mapreduce.input.fileinputformat.split.maxsize; //当几个stripe的大小大于该值时,会合并到一个task中处理
> 3.executor能力
> set spark.executor.cores; //单个executor上可以同时运行的task数
> set spark.executor.memory; // executor用于缓存数据、代码执行的堆内存以及JVM运行时需要的内存
> set spark.yarn.executor.memoryOverhead; //Spark运行还需要一些堆外内存,直接向系统申请,如数据传输时的netty等。
> set spark.sql.windowExec.buffer.spill.threshold; //当用户的SQL中包含窗口函数时,并不会把一个窗口中的所有数据全部读进内存,而是维护一个缓存池,当池中的数据条数大于该参数表示的阈值时,spark将数据写到磁盘
> 4.并行度
> set spark.defalut.parallelism=300
> 5.序列化
> set spark.serializer=org.apache.spark.serializer.KryoSerializer

## 数据倾斜的解决思路
### 1、通过 Spark Web UI
```txt
	通过 Spark Web UI 来查看当前运行的 stage 各个 task 分配的数据量(Shuffle Read Size/Records),
从而进一步确定是不是 task 分配的数据不均匀导致了数据倾斜。
	知道数据倾斜发生在哪一个 stage 之后,接着我们就需要根据 stage 划分原理,推算出来发生倾斜的那个 stage 对应代码中的哪一部分,
这部分代码中肯定会有一个 shuffle 类算子。可以通过 countByKey 查看各个 key 的分布。
```

### 2、通过 key 统计
```scala
df.select("key").sample(false,0.1)           // 数据采样
    .(k => (k,1)).reduceBykey(_ + _)         // 统计 key 出现的次数
    .map(k => (k._2,k._1)).sortByKey(false)  // 根据 key 出现次数进行排序
    .take(10)                                 // 取前 10 个。
```

### 3.过滤异常数据
```txt
1.空值或者异常值之类的,大多是这个原因引起
2.无效数据,大量重复的测试数据或是对结果影响不大的有效数据
```

### 4.提高 shuffle 并行度
```txt
	Spark 在做 Shuffle 时,默认使用 HashPartitioner(非 Hash Shuffle)对数据进行分区。如果并行度设置的不合适,
可能造成大量不相同的 Key 对应的数据被分配到了同一个 Task 上,造成该 Task 所处理的数据远大于其它 Task,从而造成数据倾斜。
	如果调整 Shuffle 时的并行度,使得原本被分配到同一 Task 的不同 Key 发配到不同 Task 上处理,则可降低原 Task 所需处理的数据量,
从而缓解数据倾斜问题造成的短板效应。
	操作流程:
	RDD 操作 可在需要 Shuffle 的操作算子上直接设置并行度或者使用 spark.default.parallelism 设置。
如果是 Spark SQL,还可通过 SET spark.sql.shuffle.partitions=[num_tasks] 设置并行度。默认参数由不同的 Cluster Manager 控制。
dataFrame 和 sparkSql 可以设置 spark.sql.shuffle.partitions=[num_tasks] 参数控制 shuffle 的并发度,默认为200。
```

### 5.自定义 Partitioner
```scala
.groupByKey(new Partitioner() {
  @Override
  public int numPartitions() {
    return 12;
  }
 
  @Override
  public int getPartition(Object key) {
    int id = Integer.parseInt(key.toString());
    if(id >= 9500000 && id <= 9500084 && ((id - 9500000) % 12) == 0) {
      return (id - 9500000) / 12;
    } else {
      return id % 12;
    }
  }
})
```

### 6.Reduce 端 Join 转化为 Map 端 Join
```scala
from pyspark.sql.functions import broadcast
result = broadcast(A).join(B,["join_col"],"left")
```

### 7.拆分 join 再 union
```txt
	将一个 join 拆分成 倾斜数据集 Join 和 非倾斜数据集 Join,最后进行 union
	1.对包含少数几个数据量过大的 key 的那个 RDD (假设是 leftRDD),通过 sample 算子采样出一份样本来,然后统计一下每个 key 的数量,
计算出来数据量最大的是哪几个 key。具体方法上面已经介绍过了,这里不赘述。
	2.然后将这 k 个 key 对应的数据从 leftRDD 中单独过滤出来,并给每个 key 都打上 1~n 以内的随机数作为前缀,
形成一个单独的 leftSkewRDD；而不会导致倾斜的大部分 key 形成另外一个 leftUnSkewRDD。
	3.接着将需要 join 的另一个 rightRDD,也过滤出来那几个倾斜 key 并通过 flatMap 操作将该数据集中每条数据均转换为 n 条数据(这 n 条数据都按顺序附加一个 0~n 的前缀),
形成单独的 rightSkewRDD；不会导致倾斜的大部分 key 也形成另外一个 rightUnSkewRDD。
	4.现在将 leftSkewRDD 与 膨胀 n 倍的 rightSkewRDD 进行 join,且在 Join 过程中将随机前缀去掉,得到倾斜数据集的 Join 结果 skewedJoinRDD。
注意到此时我们已经成功将原先相同的 key 打散成 n 份,分散到多个 task 中去进行 join 了。
	5.对 leftUnSkewRDD 与 rightUnRDD 进行Join,得到 Join 结果 unskewedJoinRDD。
	6.通过 union 算子将 skewedJoinRDD 与 unskewedJoinRDD 进行合并,从而得到完整的 Join 结果集。
```

### 8.map 端先局部聚合
```txt
	在 map 端加个 combiner 函数进行局部聚合。加上 combiner 相当于提前进行 reduce ,就会把一个 mapper 中的相同 key 进行聚合,
减少 shuffle 过程中数据量 以及 reduce 端的计算量。这种方法可以有效的缓解数据倾斜问题,但是如果导致数据倾斜的 key 大量分布在不同的 mapper 的时候,
这种方法就不是很有效了。
```

### 9.加盐局部聚合 + 去盐全局聚合
```scala
	核心实现思路就是进行两阶段聚合。第一次是局部聚合,先给每个 key 都打上一个 1~n 的随机数,比如 3 以内的随机数,此时原先一样的 key 就变成不一样的了,
比如 (hello,1) (hello,1) (hello,1) (hello,1) (hello,1),就会变成 (1_hello,1) (3_hello,1) (2_hello,1) (1_hello,1) (2_hello,1)。
接着对打上随机数后的数据,执行 reduceByKey 等聚合操作,进行局部聚合,那么局部聚合结果,就会变成了 (1_hello,2) (2_hello,2) (3_hello,1)。
然后将各个 key 的前缀给去掉,就会变成 (hello,2) (hello,2) (hello,1),再次进行全局聚合操作,就可以得到最终结果了,比如 (hello,5)。
def antiSkew(): RDD[(String,Int)] = {
    val SPLIT = "-"
    val prefix = new Random().nextInt(10)
    pairs.map(t => ( prefix + SPLIT + t._1,1))
        .reduceByKey((v1,v2) => v1 + v2)
        .map(t => (t._1.split(SPLIT)(1),t2._2))
        .reduceByKey((v1,v2) => v1 + v2)
}
```
