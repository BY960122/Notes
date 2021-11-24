# Flink
> Apache Flink 是一个框架和分布式处理引擎,用于对无界和有界数据流进行有状态计算
> 事件驱动型应用是一类具有状态的应用,它从一个或多个事件流提取数据,根据到来的事件触发计算,状态更新或其他外部动作
> 批处理的特点是: 有界、持久、大量,非常适合需要访问全套记录才能完成的计算工作,一般用于离线统计。
> 流处理的特点是: 无界、实时, 无需针对整个数据集执行操作,而是对通过系统传输的每个数据项执行操作,一般用于实时统计。

# Flink 搭建方式
> Standalone(单机)
> yarn-session
- 会话模式(Session-Cluster): 通过yarn-session提交作业yarn-session会一直启动,不停地接收客户端提交的作业,有大量的小作业,适合使用这种方式,JobManager 和 Task-managers
- 分离模式(Per-Job-Cluster): 直接提交任务给 YARN,大作业适合使用这种方式

# Flink 运行时组件
> 资源管理器(ResourceManager): 主要负责管理任务管理器(TaskManager)的插槽(slot),负责任务的分配和管理
> 作业管理器(JobManager): 每个应用程序都会被一个不同的 JobManager 所控制执行,它包含作业图(JobGraph)、逻辑数据流图(logical dataflow graph),会把 JobGraph 转换成 执行图(ExecutionGraph),然后向资源管理器(ResourceManager)请求执行任务必要的资源,也就是任务管理器(TaskManager)上的插槽(slot)
> 任务管理器(TaskManager): 具体的工作进程,会有多个,每一个都有N个slot
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

# Flink 执行图
> StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图
> StreamGraph: 是根据用户通过 Stream API 编写的代码生成的最初的图。用来表示程序的拓扑结构。
> JobGraph: StreamGraph 经过优化后生成了 JobGraph,提交给 JobManager 的数据结构。主要的优化为,将多个符合条件的节点 chain 在一起作为一个节点,这样可以减少数据在节点之间流动所需要的序列化/反序列化/传输消耗。
> ExecutionGraph :  JobManager 根 据 JobGraph 生 成 ExecutionGraph 。
> ExecutionGraph 是 JobGraph 的并行化版本,是调度层最核心的数据结构。
> 物理执行图: JobManager 根据 ExecutionGraph 对 Job 进行调度后,在各个TaskManager 上部署 Task 后形成的图,并不是一个具体的数据结构。

# Flink 算子链
> Flink会在生成 JobGraph 阶段,将代码中可以优化的算子优化成一个算子链(Operator Chains)以放到一个task（一个线程）中执行
> 算子链机制的好处: 在一起的 sub-task 都会在同一个线程(TaskManager 的 slot)中执行,能够减少不必要的数据交换、序列化和上下文切换,从而提高作业的执行效率

# Flink 并行度
> 一个数据流的并行度可以认为是其所有算子中最大的并行度
> slot 是 TaskManager 资源的最小单元,最大并行度的那个算子,决定了需要多少个 slot
> slot 是指 TaskManager 的最大并发能力,slot 只对内存隔离,对 cpu 不隔离
> 并行度顺序: 算子设置的并行度 > env 设置的并行度 > WebUI > 配置文件默认的并行度
- 全局环境: env.setParallelism(10)
- source: env.addSource(...).setParallelism(3),kafka topic 有3个分区,设置并行度3,同时消费3个并行度
- 算子: .map(...).setParallelism(5)
- 输出 sink: sum.print().setParallelism(1)

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
- 事务写入: 构建的事务对应着 checkpoint,等到 checkpoint 真正完成的时候,才把所有对应的结果写入 sink 系统中,实现方式: 预写日志(WAL)和两阶段提交
(2PC),DataStream API 提供了 GenericWriteAheadSink 模板类和TwoPhaseCommitSinkFunction 接口