# HBase 数据模型
> 1.nameSpace: 命名空间,类似于关系型数据库的 DatabBase 概念,有两个自带的命名空间,分别是 hbase 和 default,hbase 中存放的是 HBase 内置的表,
default 表是用户默认使用的命名空间
> 2.Region: 类似于表概念,只需要声明列族即可,往 HBase 写入数据时,字段可以动态、按需指定
> 3.Row: 表中的每行数据都由一个 RowKey 和多个 Column(列)组成
> 4.Column: 每个列都由 Column Family(列族)和 Column Qualifier(列限定符)进行限定,例如: info: name,info: age
> 5.Time Stamp: 用于标识数据的不同版本(version),如果不指定时间戳,系统会自动为其加上该字段,其值为写入 HBase 的时间
> 6.Cell: 由{rowkey, column Family: column Qualifier, time Stamp} 唯一确定的单元cell 中的数据是没有类型的,全部是字节码形式存贮

# HBase 架构
> 1.Master: Master 是所有 Region Server 的管理者,其实现类为 HMaster
- 对表的操作: create,delete,alter
- 对 Region Server的操作: 分配 regions到每个RegionServer,监控每个 RegionServer的状态,负载均衡和故障转移
> 2.Region Server: Region Server 为 Region 的管理者,其实现类为 HRegionServer
- 对于数据的操作: get, put, delete
- 对于 Region 的操作: splitRegion(切)、compactRegion(压)
- Split时机
- 0.94版本之前: 某个Store下所有 StoreFile 的总大小超过 hbase.hregion.max.filesize,该 Region 就会进行拆分
- 0.94版本之后: Min(R^2*"hbase.hregion.memstore.flush.size",hbase.hregion.max.filesize"),R 为当前 Region Server 中属于该 Table 的个数
- Compaction 分为两种,分别是 Minor Compaction 和 Major Compaction
- Minor Compaction 会将临近的若干个较小的 HFile 合并成一个较大的 HFile,但不会清理过期和删除的数据
- Major Compaction 会将一个 Store 下的所有的 HFile 合并成一个大 HFile,并且会清理掉过期和删除的数据

> 3.Zookeeper: 通过 Zookeeper 来做 Master 的高可用、RegionServer 的监控、元数据的入口以及集群配置的维护等工作
> 4.HDFS: 提供最终的底层数据存储服务,同时为 HBase 提供高可用的支持
> 5.StoreFile: 保存实际数据的物理文件,StoreFile 以 HFile 的形式存储在 HDFS 上,每个 Store 会有一个或多个 StoreFile(HFile),数据在每个 StoreFile 中都是有序的
> 6.MemStore: 写缓存,由于 HFile 中的数据要求是有序的,所以数据是先存储在 MemStore 中,排好序后,等到达刷写时机才会刷写到 HFile,每次刷写都会形成一个新的 HFile
> 7.WAL: 由于数据要经 MemStore 排序后才能刷写到 HFile,但把数据保存在内存中会有很高的概率导致数据丢失,为了解决这个问题,数据会先写在一个叫做 Write-Ahead logfile 的文件中,然后再写入 MemStore 中所以在系统出现故障的时候,数据可以通过这个日志文件重建

# HBase 写流程
> 1.Client 先访问 zookeeper,获取 hbase:meta 表位于哪个 Region Server
> 2.访问对应的 Region Server,获取 hbase:meta 表,根据读请求的 namespace:table/rowkey,查询出目标数据位于哪个 Region Server 中的哪个 Region 中并将该 table 的 region 信息以及 meta 表的位置信息缓存在客户端的 meta cache,方便下次访问
> 3.与目标 Region Server 进行通讯
> 4.将数据顺序写入（追加）到 WAL
> 5.将数据写入对应的 MemStore,数据会在 MemStore 进行排序
> 6.向客户端发送 ack
> 7.等达到 MemStore 的刷写时机后,将数据刷写到 HFile,hbase.hregion.memstore.flush.size=默认值 128M

# HBase 读流程
> 1.Client 先访问 zookeeper,获取 hbase:meta 表位于哪个 Region Server
> 2.访问对应的 Region Server,获取 hbase:meta 表,根据读请求的 namespace:table/rowkey,查询出目标数据位于哪个 Region Server 中的哪个 Region 中并将该 table 的 region 信息以及 meta 表的位置信息缓存在客户端的 meta cache,方便下次访问
> 3.与目标 Region Server 进行通讯
> 4.分别在 Block Cache（读缓存）,MemStore 和 Store File（HFile）中查询目标数据,并将查到的所有数据进行合并此处所有数据是指同一条数据的不同版本（time stamp）或者不同的类型（Put/Delete）
> 5.将从文件中查询到的数据块（Block,HFile 数据存储单元,默认大小为 64KB）缓存到Block Cache
> 6.将合并后的最终结果返回给客户端

# HBase 优化
> 1.内存优化: 一般会分配整个可用内存的 70%给 HBase 的 Java 堆,一般 16-48G 内存就可以了
> 2.允许在 HDFS 的文件中追加内容,可以优秀的配合 HBase 的数据同步和持久化.hdfs-site.xml、hbase-site.xml,属性：dfs.support.append=true
> 3.优化 DataNode 允许的最大文件打开数,hdfs-site.xml,dfs.datanode.max.transfer.threads=4096(默认)
> 4.优化延迟高的数据操作的等待时间,hdfs-site.xml,dfs.image.transfer.timeout=60000毫秒,如果对于某一次数据操作来讲延迟非常高,以确保 socket 不会被 timeout 掉
> 5.优化数据的写入效率,mapred-site.xml,mapreduce.map.output.compress=true,mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.GzipCodec
> 6.设置 RPC 监听数量,hbase-site.xml,Hbase.regionserver.handler.count=30,读写请求较多时,增加此值
> 7.优化 HStore 文件大小,hbase-site.xml,hbase.hregion.max.filesize=10737418240(10GB),走 MR 应适当减小
> 8.优化 HBase 客户端缓存: hbase-site.xml,hbase.client.write.buffer,增大会减小通信次数,但需要内存
> 9.指定 scan.next 扫描 HBase 所获取的行数,hbase-site.xml,hbase.client.scanner.caching,用于指定 scan.next 方法获取的默认行数