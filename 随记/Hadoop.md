# hadoop 进程
## HA
> 1.master: namenode,dfszkfailoverController,resourcemanager,QuorumPeerMain
> 2.slaver: datanode,nodemanager,journalnode

# hdfs 读流程
> 1.首先调用 FileSystem.open()方法,获取到 DistributedFileSystem 实例
> 2.DistributedFileSystem 向 Namenode 发起RPC(远程过程调用)请求获得文件的开始部分或全部block列表,对于每个返回的块,都包含块所在的DataNode地址。这些DataNode会按照Hadoop定义的集群拓扑结构得出客户端的距离,然后再进行排序。如果客户端本身就是一个DataNode,那么他将从本地读取文件。
> 3.DistributedFileSystem 会向客户端client返回一个支持文件定位的输入流对象 FSDataInputStream,用于客户端读取数据。FSDataInputStream 包含一个DFSInputStream对象,这个对象用来管理DataNode和NameNode之间的I/O。
> 4.客户端调用read()方法,DFSInputStream 就会找出离客户端最近的datanode并连接datanode
> 5.DFSInputStream对象中包含文件开始部分的数据块所在的DataNode地址,首先它会连接包含文件第一个块最近DataNode。随后,在数据流中重复调用read()函数,直到这个块全部读完为止。如果第一个block块的数据读完,就会关闭指向第一个block块的datanode连接,接着读取下一个block块
> 6.如果第一批block都读完了,DFSInputStream 就会去 namenode 拿下一批blocks的location,然后继续读,如果所有的block块都读完,这时就会关闭掉所有的流。
> 注意: read 方法是并行的读取 block 信息,不是一块一块的读取。NameNode 只是返回Client请求包含块的DataNode地址,并不是返回请求块的数据。最终读取来所有的 block 会合并成一个完整的最终文件。

# hdfs 写流程
> 1.Client 发起文件上传请求,调用 DistributedFileSystem 对象的 create 方法,创建一个文件输出流 FSDataOutputStream 对象
> 2.通过 DistributedFileSystem 对象与 Hadoop 集群的 NameNode 进行一次RPC远程调用,NameNode 检查目标文件是否已存在,父目录是否存在,返回是否可以上传;在HDFS的 Namespace 中创建一个文件条目 Entry ,该条目没有任何的Block
> 3.通过FSDataOutputStream 对象,向 DataNode 写入数据,数据首先被写入 FSDataOutputStream 对象内部的 Buffer 中,然后数据被分割成一个个 Packet 数据包
> 4.以Packet最小单位(默认64K),基于 Socket 连接发送到按特定算法选择的HDFS集群中一组 DataNode(正常是3个,可能大于等于1)中的一个节点上,在这组DataNode组成的Pipeline上依次传输 Packet: client请求3台 DataNode 中的一台A上传数据(本质上是一个RPC调用,建立pipeline),A收到请求会继续调用B,然后B调用C,将整个pipeline建立完成,后逐级返回 client。
> 5.这组 DataNode 组成的Pipeline反方向上,发送ack,最终由Pipeline中第一个DataNode节点将Pipeline ack发送给Client
> 6.完成向文件写入数据,Client在文件输出流 FSDataOutputStream 对象上调用close方法,关闭流
> 7.调用 DistributedFileSystem 对象的 complete 方法,通知 NameNode 文件写入成功

# hdfs 文件格式
- https://www.jianshu.com/p/43630456a18a
> SequenceFile: 以二进制键值对的形式存储数据
> Avro: 将数据定义和数据一起存储在一条消息中,数据定义以JSON格式存储,数据以二进制格式存储,文件格式更为紧凑,读取大量数据时Avro能够提供更好的序列化和反序列化性能
> RCFile: 以列格式保存每个行组数据它不是存储第一行然后是第二行,而是存储所有行上的第1列,然后是所有行上的第2列,以此类推。RCFile在map阶段从远端拷贝仍然是拷贝整个数据块,并且拷贝到本地目录后RCFile并不是真正直接跳过不需要的列,并跳到需要读取的列, 而是通过扫描每一个row group的头部定义来实现的,但是在整个HDFS Block 级别的头部并没有定义每个列从哪个row group起始到哪个row group结束。所以在读取所有列的情况下,RCFile的性能反而没有SequenceFile高。
> ORCFile: 将数据划分为默认大小为250M的Stripe。每个Stripe包括索引,数据和Footer,索引存储每一列的最大最小值,以及列中每一行的位置。
> Parquet: 是Hadoop的一种列存储格式,提供了高效的编码和压缩方案,特别擅长处理深度嵌套的数据

# MapReduce流程
> 1.maptask 收集我们的 map()方法输出的 kv 对,放到内存缓冲区中
> 2.从内存缓冲区不断溢出本地磁盘文件,可能会溢出多个文件
> 3.多个溢出文件会被合并成大的溢出文件
> 4.在溢出过程中,及合并的过程中,都要调用 partitioner 进行分区和针对 key 进行排序
> 5.reducetask 根据自己的分区号,去各个 maptask 机器上取相应的结果分区数据
> 6.reducetask 会取到同一个分区的来自不同 maptask 的结果文件,reducetask 会将这些文件再进行合并(归并排序)
> 7.合并成大文件后,shuffle 的过程也就结束了,后面进入 reducetask 的逻辑运算过程(从文件中取出一个一个的键值对 group,调用用户自定义的 reduce()方法)
> 备注: 缓冲区的大小可以通过参数调整,参数: io.sort.mb 默认 100M

## reduceTask工作机制
> 1.Copy 阶段: ReduceTask 从各个 MapTask 上远程拷贝一片数据,并针对某一片数据,如果其大小超过一定阈值,则写到磁盘上,否则直接放到内存中。
> 2.Merge 阶段: 在远程拷贝数据的同时,ReduceTask 启动了两个后台线程对内存和磁盘上的文件进行合并,以防止内存使用过多或磁盘上文件过多。
> 3.Sort 阶段: 按照 MapReduce 语义,用户编写 reduce()函数输入数据是按 key 进行聚集的一组数据。为了将 key 相同的数据聚在一起,Hadoop 采用了基于排序的策略。由于各个 MapTask 已经实现对自己的处理结果进行了局部排序,因此,ReduceTask 只需对所有数据进行一次归并排序即可。
> 4.Reduce 阶段: reduce() 函数将计算结果写到 HDFS 上。

# Yarn 作业提交全过程
## 1.作业提交
> 1.client 调用 job.waitForCompletion 方法,向整个集群提交 MapReduce 作业。
> 2.client 向 RM 申请一个作业 id。
> 3.RM 给 client 返回该 job 资源的提交路径和作业 id。
> 4.client 提交 jar 包,切片信息和配置文件到指定的资源提交路径。
> 5.client 提交完资源后,向 RM 申请运行 MrAppMaster。
## 2.作业初始化
> 6.当 RM 收到 client 的请求后,将该 job 添加到容量调度器中。
> 7.某一个空闲的 NM 领取到该 job。
> 8.该 NM 创建 Container,并产生 MRAppmaster。
> 9.下载 client 提交的资源到本地。
## 3.任务分配
> 10.MrAppMaster 向 RM 申请运行多个 maptask 任务资源。
> 11.RM 将运行 maptask 任务分配给另外两个 NodeManager,另两个 NodeManager 分别领取任务并创建容器。
## 4.任务运行
> 12.MR 向两个接收到任务的 NodeManager 发送程序启动脚本,这两个NodeManager 分别启动 maptask,maptask 对数据分区排序。
> 13.MrAppMaster 等待所有 maptask运行完毕后,向 RM 申请容器,运行 reduce task。
> 14.reduce task 向 maptask 获取相应分区的数据。
> 15.程序运行完毕后,MR 会向 RM 申请注销自己。
## 5.进度和状态更新
> YARN 中的任务将其进度和状态(包括 counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval 设置)向应用管理器请求进度更新, 展示给用户。
## 6.作业完成
> 除了向应用管理器请求作业进度外, 客户端每 5 分钟都会通过调用 waitForCompletion() 来检查作业是否完成。
> 时间间隔可以通过 mapreduce.client.completion.pollinterval 来设置。作业完成之后, 应用管理器和 container 会清理工作状态。
> 作业的信息会被作业历史服务器存储以备之后用户核查。

# 优化
## MapReduce 跑得慢的原因
> 1.计算机性能(CPU,内存,磁盘,网络)
> 2.数据倾斜
> 3.小文件过多
> 4.map 和 reduce 数设置的不合理 -> 3.4.2
> 5.map运行的太长
> 6.大量不可分割的超大文件
> 7.spill 溢写次数过多
> 8.merge 次数过多

## MapReduce 优化
### 1.数据输入
> 1.合并小文件
> 2.使用CombinerTextInputFormat,CombineFileInputFormat来作为输入,解决导入大量小文件的问题
> 3.hdfs上的小文件可以采用 sequence file 由一系列的二进制 key/value 组成,如果 key 为文件名,value 为文件内容,则可以将大批小文件合并成一个大文件。
> 4.Hadoop Archive: 将小文件放入 HDFS 块中的文件存档工具,它能够将多个小文件打包成一个 HAR 文件

### 2.Map 阶段
> 1.减少溢写spill次数: 通过调整 io.sort.mb 以及 sort.spill.percent ,增大触发内存的上限
> 2.减少合并merge次数: 通过调整 io.sort.factor ,增大 merge 的文件数目,减少 merge 的次数,从而缩短 mr 处理时间。
> 3.map之后,如果不影响结果,先进行Combiner

### 3.Reduce 阶段
> 1.合理设置 map 和 reduce 的数量,太多会导致资源竞争,太少会高延迟,处理超时.
> 2.设置 map,reduce 共存: 调整 slowstart.completedmaps 参数,使 map 运行到一定程度后,reduce 也开始运行,减少 reduce 的等待时间。
> 3.合理设置reduce端的buffer,调整 mapred.job.reduce.input.buffer.percent ,让内存中的一部分数据直接进入reduce,少了写磁盘,读磁盘的IO

### 其它办法
> 1.开启JVM重用,mapreduce.job.jvm.numtasks=10到20之间,一个jvm运行完一个map后,可以继续运行其它map

## 常用的调优参数
### mapred-default.xml
> mapreduce.task.io.sort.mb=100 ,shuffle 的环形缓冲区大小,默认 100m
> mapreduce.map.sort.spill.percent=0.8 ,环形缓冲区溢出的阈值,默认 80%
> mapreduce.map.memory.mb ,一个 Map Task 可使用的资源上限（单位:MB）,默认为 1024。如果 Map Task 实际使用的资源量超过该值,则会被强制杀死。
> mapreduce.reduce.memory.mb ,一个 Reduce Task 可使用的资源上限（单位:MB）,默认为 1024。如果 Reduce Task实际使用的资源量超过该值,则会被强制杀死。
> mapreduce.map.cpu.vcores ,每个 Map task 可使用的最多 cpu core 数目,默认值: 1
> mapreduce.reduce.cpu.vcores ,每个 Reduce task 可使用的最多 cpu core 数目,默认值: 1
> mapreduce.map.maxattempts 每个 Map Task 最大重试次数,一旦重试参数超过该值,则认为 Map Task 运行失败,默认值：4。
> mapreduce.reduce.maxattempts 每个 Reduce Task 最大重试次数,一旦重试参数超过该值,则认为 Map Task 运行失败,默认值：4。
> mapreduce.reduce.shuffle.parallelcopies ,每个 reduce 去 map 中拿数据的并行数。默认值是 5
> mapreduce.reduce.shuffle.merge.percent ,buffer 中的数据达到多少比例开始写入磁盘。默认值 0.66
> mapreduce.reduce.shuffle.input.buffer.percent ,buffer 大小占 reduce 可用内存的比例。默认值 0.7
> mapreduce.reduce.input.buffer.percent ,指定多少比例的内存用来存放 buffer 中的数据,默认值是 0.0

### yarn-default.xml
> yarn.scheduler.minimum-allocation-mb 1024 给应用程序 container 分配的最小内存
> yarn.scheduler.maximum-allocation-mb 8192 给应用程序 container 分配的最大内存
> yarn.scheduler.minimum-allocation-vcores 1 每个 container 申请的最小 CPU 核数
> yarn.scheduler.maximum-allocation-vcores 32 每个 container 申请的最大 CPU 核数
> yarn.nodemanager.resource.memory-mb 8192 给 containers 分配的最大物理内存