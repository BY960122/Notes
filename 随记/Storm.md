# Storm
> 是 twitter 开源的一个实时处理框架
> Storm 将流中元素抽象为 Tuple,它就是一个 list
> Storm认为每个stream都有一个stream源,也就是原始元组的源头,所以它将这个源头称为 Spout,将流的状态转换称为 Bolt
> Topology(MapReduce) = Spout(源) + Bolt(处理,转换)
- Spout -> Bolt-1 -> Bolt-2 ...

# Storm 集群结构
> master: Nimbus进程(相当于 Hadoop 的 ResourceManager),负责分发代码,分配任务,故障检测
> worker: Supervisor进程(NodeManager), 执行一个 Topology 的子集
> 一台机器 -> N个 worker进程(jvm进程) -> N个 executor线程 -> 同一个spout/bolt 的N个 task

# 如何提高 storm 组件并行度
> 1.worker
- 默认一个从节点上面最多可以启动4个 worker 进程,可以在 defaults.yaml 配置 supervisor.slots.ports
- 也可以在代码中设置: config.setNumWorkers(workers)
- 默认一个 topology 只使用一个 worker ,可以在代码中使用多个
> 2.executor 
- 默认一个 executor 运行一个 task,可以在代码中设置运行多个
- builder.setSpout(id, spout, parallelism_hint)
- builder.setBolt(id, bolt, parallelism_hint)
> 3.task
- 在代码中设置: boltDeclarer.setNumTasks(num)

# 并行度设置多少合适
> 单spout每秒大概可以发送 500 个 tuple
> 单bolt每秒大概可以接收 2000 个 tuple
> 单acker每秒大概可以接收 6000 个 tuple

# StreamGrouping 分类
> 1.Shuffle Grouping: 随机分组,随机派发stream里面的tuple,保证bolt中的每个任务接收到的tuple数目相同
> 2.Fields Grouping: 按字段分组,比如按userid来分组,具有同样userid的tuple会被分到同一任务,而不同的userid则会被分配到不同的任务
> 3.All Grouping: 广播发送,对于每一个tuple,Bolts中的所有任务都会收到
> 4.Global Grouping: 全局分组,分配给id值最低的那个 task
> 5.Non Grouping: 随机分派,意思是说stream不关心到底谁会收到它的tuple.目前他和 Shuffle Grouping 是一样的效果
> 6.Direct Grouping: 消息的发送者具体由消息接收者的哪个task处理这个消息,必须使用 emitDirect 方法来发射
> 7.localOrShuffleGrouping: 如果目标Bolt 中的一个或者多个Task 和当前产生数据的Task 在同一个Worker 进程里面,那么就走内部的线程间通信,否则 Shuffle Grouping

# 可靠性
> 通过 config.setNumAckers(num) 来设置一个 topology 里面的 acker 的数量,默认值是1

# Storm UI
> deactive: 未激活(暂停)
> emitted: emitted tuple数
> transferred: transferred tuple数
- 与emitted的区别: 如果一个task,emitted一个tuple到2个task中,则 transferred tuple数是emitted tuple数的两倍
> complete latency: spout emitting 一个tuple到spout ack这个tuple的平均时间(可以认为是tuple以及该tuple树的整个处理时间)
> process latency:   bolt收到一个tuple到bolt ack这个tuple的平均时间,如果没有启动acker机制,那么值为0
> execute latency: bolt处理一个tuple的平均时间,不包含acker操作,单位是毫秒(也就是bolt 执行 execute 方法的平均时间)
> capacity: 这个值越接近1,说明bolt或者spout基本一直在调用execute方法,说明并行度不够,需要扩展这个组件的executor数量。(调整组件并行度的依据)
> 总结: execute latency和proces latnecy是处理消息的时效性,而capacity则表示处理能力是否已经饱和,从这3个参数可以知道topology的瓶颈所在。

# Storm 雪崩问题
> spout 发送速度 > bolt 接受速度
> 1.增加 bolt 的并发度
> 2.设置 spout 的发送速度: topology.max.spout.pending,或者通过代码 config.setMaxSpoutPending(num)

# Storm 反压
> 通过监控 Bolt 中的接收队列负载情况,如果超过高水位值就会将反压信息写到 Zookeeper,Zookeeper 上的 watch 会通知该拓扑的所有 Worker 都进入反压状态,最后 Spout 停止发送 tuple。
> 而Storm是直接从源头降速