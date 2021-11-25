- https://zhuanlan.zhihu.com/p/138101642
# Flink
> Apache Flink 是一个框架和分布式处理引擎,用于对无界和有界数据流进行有状态计算
> 事件驱动型应用是一类具有状态的应用,它从一个或多个事件流提取数据,根据到来的事件触发计算,状态更新或其他外部动作
> 批处理的特点是: 有界、持久、大量,非常适合需要访问全套记录才能完成的计算工作,一般用于离线统计。
> 流处理的特点是: 无界、实时, 无需针对整个数据集执行操作,而是对通过系统传输的每个数据项执行操作,一般用于实时统计。

# Flink checkpoint 与 spark 比较
> spark streaming 的 checkpoint 仅仅是针对 driver 的故障恢复做了数据和元数据的 checkpoint。而 flink 的 checkpoint 机制 要复杂了很多,它采用的是轻量级的分布式快照,实现了每个算子的快照,及流动中的数据的快照

# Flink 搭建方式
> Standalone(单机)
> yarn-session
- 会话模式(Session-Cluster): 通过yarn-session提交作业yarn-session会一直启动,不停地接收客户端提交的作业,有大量的小作业,适合使用这种方式,JobManager 和 Task-managers
- 分离模式(Per-Job-Cluster): 直接提交任务给 YARN,大作业适合使用这种方式

# Flink 运行时组件
> 资源管理器(ResourceManager): 主要负责管理任务管理器(TaskManager)的插槽(slot),负责任务的分配和管理
> 1.作业管理器(JobManager): 每个应用程序都会被一个不同的 JobManager 所控制执行,它包含作业图(JobGraph)、逻辑数据流图(logical dataflow graph),会把 JobGraph 转换成 执行图(ExecutionGraph),然后向资源管理器(ResourceManager)请求执行任务必要的资源,也就是任务管理器(TaskManager)上的插槽(slot)\
- 整个集群的协调者,负责接收Flink Job,协调检查点,Failover 故障恢复等,同时管理Flink集群中从节点TaskManager
> 2.任务管理器(TaskManager): 具体的工作进程,会有多个,每一个都有N个slot
- 每个TaskManager负责管理其所在节点上的资源信息,如内存、磁盘、网络,在启动的时候将资源的状态向 JobManager 汇报
> 分发器(Dispatcher): 可以跨作业运行,它为应用提交提供了 REST 接口。当一个应用被提交执行时,分发器就会启动并将应用移交给一个 JobManager。

# Flink 分层
> ProcessFunction: 可以访问事件的时间戳信息和水位线信息,都继承自 RichFunction 接口,所以都有 open()、close()和 getRuntimeContext()等方法
- ProcessFunction
- KeyedProcessFunction
- CoProcessFunction
- ProcessJoinFunction
- BroadcastProcessFunction
- KeyedBroadcastProcessFunction
- ProcessWindowFunction
- ProcessAllWindowFunction
> DataStream & DataSet: 算子操作
> SQL / TableAPI: 二维表数据结构,sql查询

# Flink 内存管理
> 对象都序列化到一个预分配的内存块上,大量使用堆外内存,如果超过了内存限制,会将部分数据写入硬盘.
> 1.Network Buffers(堆外): 这个是在TaskManager启动的时候分配的,这是一组用于缓存网络数据的内存,MemorySegment(内存抽象)每个块是32K,默认分配2048个,taskmanager.network.numberOfBuffers
> 2.Memory Manage pool: 这部分启动的时候就会分配,用于运行时的算法(shuffle,join)
> 3.User Code: 这部分是除了Memory Manager之外的内存用于User code和TaskManager本身的数据结构

# Flink 序列化
> TypeInformation 是所有类型描述符的基类。它揭示了该类型的一些基本属性,并且可以生成序列化器。TypeInformation 支持以下几种类型
> 1.BasicTypeInfo: 任意Java 基本类型或 String 类型
> 2.BasicArrayTypeInfo: 任意Java基本类型数组或 String 数组
> 3.WritableTypeInfo: 任意 Hadoop Writable 接口的实现类
> 4.TupleTypeInfo: 任意的 Flink Tuple 类型(支持Tuple1 to Tuple25)。Flink tuples 是固定长度固定类型的Java Tuple实现
> 5.CaseClassTypeInfo: 任意的 Scala CaseClass(包括 Scala tuples)
> 6.PojoTypeInfo: 任意的 POJO (Java or Scala),例如,Java对象的所有成员变量,要么是 public 修饰符定义,要么有 getter/setter 方法
> 7.GenericTypeInfo: 任意无法匹配之前几种类型的类
> 针对前六种类型数据集,Flink皆可以自动生成对应的TypeSerializer,能非常高效地对数据集进行序列化和反序列化

# Flink 执行图
> StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图
> StreamGraph: 是根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构。
> JobGraph: StreamGraph 经过优化后生成了 JobGraph,提交给 JobManager 的数据结构。主要的优化为,将多个符合条件的节点 chain 在一起作为一个节点,这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。
> ExecutionGraph :  JobManager 根 据 JobGraph 生 成 ExecutionGraph 。
> ExecutionGraph 是 JobGraph 的并行化版本,是调度层最核心的数据结构。
> 物理执行图: JobManager 根据 ExecutionGraph 对 Job 进行调度后,在各个TaskManager 上部署 Task 后形成的图,并不是一个具体的数据结构。

# Flink 算子链
> Flink会在生成 JobGraph 阶段,将代码中可以优化的算子优化成一个算子链(Operator Chains)以放到一个task(一个线程)中执行
> 算子链机制的好处: 在一起的 sub-task 都会在同一个线程(TaskManager 的 slot)中执行
> 它能减少线程之间的切换,减少消息的序列化/反序列化,减少数据在缓冲区的交换,减少了延迟的同时提高整体的吞吐量
> 两个operator chain在一起的的条件: 
- 上下游的并行度一致
- 下游节点的入度为1(也就是说下游节点没有来自其他节点的输入)
- 上下游节点都在同一个 slot group 中

# Flink 并行度
> 一个数据流的并行度可以认为是其所有算子中最大的并行度
> slot 是 TaskManager 资源的最小单元,最大并行度的那个算子,决定了需要多少个 slot
> slot 是指 TaskManager 的最大并发能力,slot 只对内存隔离,对 cpu 不隔离
> 并行度顺序: 算子设置的并行度 > env 设置的并行度 > WebUI > 配置文件默认的并行度
- 全局环境: env.setParallelism(10)
- source: env.addSource(...).setParallelism(3),kafka topic 有3个分区,设置并行度3,同时消费3个并行度
- 算子: .map(...).setParallelism(5)
- 输出 sink: sum.print().setParallelism(1)

# Flink 分区策略
> GlobalPartitioner: 数据会被分发到下游算子的第一个实例中进行处理。
> ShufflePartitioner: 数据会被随机分发到下游算子的每一个实例中进行处理。
> RebalancePartitioner: 数据会被循环发送到下游的每一个实例中进行处理。
> RescalePartitioner: 根据上下游算子的并行度,循环的方式输出到下游算子的每个实例 A B -> 1,2,3,4 A:1,2 B:3,4
> BroadcastPartitioner: 会将上游数据输出到下游算子的每个实例中,适合于大数据集和小数据集做Jion的场景
> ForwardPartitioner: 用于将记录输出到下游本地的算子实例。它要求上下游算子并行度一样
> KeyGroupStreamPartitioner: Hash分区器,会将数据按 Key 的 Hash 值输出到下游算子实例中
> CustomPartitionerWrapper: 自定义分区器,重写 publicintpartition 方法

# Flink 任务提交流程
> 1.client 向 hdfs 提交jar包和配置,之后向Yarn ResourceManager 提交任务
> 2.ResourceManager分配 Container,并在相应的 NodeManager 启动 AplicationMaster
> 3.AplicationMaster 启动后加载 Jar包和配置构建执行环境,然后启动 JobManager
> 4.之后 ApplicationMaster 向 ResourceManager 申请资源启动 TaskManager
> 5.TaskManager 启动后向 JobManager 发送心跳,等待 JobManager 分配任务

# Flink Window
> Window 是无限数据流处理的核心,Window 将一个无限的 stream 拆分成有限大小的 buckets 桶,我们可以在这些桶上做计算操作。
> CountWindow: 按照指定的数据条数生成一个 Window,与时间无关
> TimeWindow: 
- 滚动窗口 TumblingWindow: 时间对齐,窗口长度固定,没有重叠
- 滑动窗口 Sliding Window: 时间对齐,窗口长度固定,可以有重叠
- 会话窗口 Session Window: 时间无对齐,由一系列事件组合一个指定时间长度的 timeout 间隙组成,类似于 web 应用的 session,也就是一段时间没有接收到新数据就会生成新的窗口

# Flink 时间语义
> Event Time：是事件创建的时间。它通常由事件中的时间戳描述,例如采集的日志数据中,每一条日志都会记录自己的生成时间
> Ingestion Time：是数据进入 Flink 的时间。
> Processing Time：是每一个执行基于时间操作的算子的本地系统时间,与机器相关,默认的时间属性就是 Processing Time。

# Flink Watermark
> Watermark 是用于处理乱序事件的,而正确的处理乱序事件,通常用Watermark 机制结合 window 来实现,就是关窗的时间

# Flink 状态一致性
> 内部保证: checkpoint
> source 端: 需要外部源可重设数据的读取位置
> sink 端: 
- 幂等写入: 一个操作,可以重复执行很多次,但只导致一次结果更改,也就是说,后面再重复执行就不起作用了。
- 事务写入: 构建的事务对应着 checkpoint,把结果数据先当成状态保存,等到 checkpoint 真正完成的时候,才把所有对应的结果写入 sink 系统中,实现方式: 预写日志(WAL)和两阶段提交
(2PC),DataStream API 提供了 GenericWriteAheadSink 模板类和TwoPhaseCommitSinkFunction 接口

# Flink 状态后端(存储)
> MemoryStateBackend
> FsStateBackend
> RocksDBStateBackend。

# Flink 是如何保证Exactly-once语义的？
> Flink通过实现两阶段提交和状态保存来实现端到端的一致性语义。分为以下几个步骤:
> 1.开始事务（beginTransaction）创建一个临时文件夹,来写把数据写入到这个文件夹里面
> 2.预提交（preCommit）将内存中缓存的数据写入文件并关闭
> 3.正式提交（commit）将之前写完的临时文件放入目标目录下。这代表着最终的数据会有一些延迟
> 4.丢弃（abort）丢弃临时文件
若失败发生在预提交成功后,正式提交前。可以根据状态来提交预提交的数据,也可删除预提交的数据。

# Flink 重启策略
> 固定延迟重启策略(Fixed Delay Restart Strategy)
> 故障率重启策略(Failure Rate Restart Strategy)
> 没有重启策略(No Restart Strategy)
> Fallback重启策略(Fallback Restart Strategy)

# Flink 分布式缓存
> 目的是在本地读取文件,并把他放在 taskmanager 节点中,防止task重复拉取
```scala
env.registerCachedFile("hdfs:///path/to/your/file", "hdfsFile")
env.registerCachedFile("file:///path/to/exec/file", "localExecFile", true)
```

# Flink SQL
> 1.用户使用对外提供Stream SQL的语法开发业务应用
> 2.用calcite对StreamSQL进行语法检验,语法检验通过后,转换成calcite的逻辑树节点；最终形成calcite的逻辑计划
> 3.采用Flink自定义的优化规则和calcite火山模型、启发式模型共同对逻辑树进行优化,生成最优的Flink物理计划
> 4.对物理计划采用janino codegen生成代码,生成用低阶API DataStream 描述的流应用,提交到Flink平台执行

# Flink 数据倾斜和数据热点问题
> 数据倾斜指的是数据在不同的窗口内堆积的数据量相差过多,本质上产生这种情况的原因是数据源头发送的数据量速度不同导致的
- 在数据进入窗口前做预聚合
- 重新设计窗口聚合的key
> 假设订单场景,北京和上海两个城市订单量增长几十倍,其余城市的数据量不变
> 1.把热key进行拆分,比如把北京和上海按照地区进行拆分聚合

# Flink是如何处理反压的(任务延迟高)
> 资源调优:
- 作业: 算子的parallelism,core(CPU),堆内存
- 作业参数: State,checkpoint
> Flink 内部是基于 producer-consumer 模型来进行消息传递的,Flink的反压设计也是基于这个模型
> Flink 使用了高效有界的分布式阻塞队列,下游消费者消费变慢,上游就会受到阻塞
> Flink是逐级反压,而 Storm 直接从源头降速
- Storm 是通过监控 Bolt 中的接收队列负载情况,如果超过高水位值就会将反压信息写到 Zookeeper
- Zookeeper 上的 watch 会通知该拓扑的所有 Worker 都进入反压状态,最后 Spout 停止发送 tuple