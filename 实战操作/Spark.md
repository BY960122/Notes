# 数据倾斜的解决思路
## 1、通过 Spark Web UI
```txt
	通过 Spark Web UI 来查看当前运行的 stage 各个 task 分配的数据量(Shuffle Read Size/Records),
从而进一步确定是不是 task 分配的数据不均匀导致了数据倾斜。
	知道数据倾斜发生在哪一个 stage 之后,接着我们就需要根据 stage 划分原理,推算出来发生倾斜的那个 stage 对应代码中的哪一部分,
这部分代码中肯定会有一个 shuffle 类算子。可以通过 countByKey 查看各个 key 的分布。
```

## 2、通过 key 统计
```scala
df.select("key").sample(false,0.1)           // 数据采样
    .(k => (k,1)).reduceBykey(_ + _)         // 统计 key 出现的次数
    .map(k => (k._2,k._1)).sortByKey(false)  // 根据 key 出现次数进行排序
    .take(10)                                 // 取前 10 个。
```

## 3.过滤异常数据
```txt
1.空值或者异常值之类的,大多是这个原因引起
2.无效数据,大量重复的测试数据或是对结果影响不大的有效数据
```

## 4.提高 shuffle 并行度
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

## 5.自定义 Partitioner
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

## 6.Reduce 端 Join 转化为 Map 端 Join
```scala
from pyspark.sql.functions import broadcast
result = broadcast(A).join(B,["join_col"],"left")
```

## 7.拆分 join 再 union
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

## 8.map 端先局部聚合
```txt
	在 map 端加个 combiner 函数进行局部聚合。加上 combiner 相当于提前进行 reduce ,就会把一个 mapper 中的相同 key 进行聚合,
减少 shuffle 过程中数据量 以及 reduce 端的计算量。这种方法可以有效的缓解数据倾斜问题,但是如果导致数据倾斜的 key 大量分布在不同的 mapper 的时候,
这种方法就不是很有效了。
```

## 9.加盐局部聚合 + 去盐全局聚合
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