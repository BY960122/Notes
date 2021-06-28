# 命令行操作

## 命名空间

```sh
# 创建命名空间
create_namespace 'test'

# 查看所有命名空间
list_namespace
scan 'hbase:namespace'

# 查看命名空间所有表
list_namespace_tables 'hbase'

# 删除命名空间
drop_namespace 'test'
```

## 表

```sh
# 创建表
create 'test:test_hbase','info'
# 插入数据
put 'test:test_hbase','1','info:name','ByDylan'
put 'test:test_hbase','2','info:age','24'
# 添加列族
incr 'test:test_hbase','1','info:sex'
# 查表结构
describe 'hbase:namespace'
# 查表数据
scan 'hbase:namespace'
	# 限制条数
scan 'hbase:namespace',{LIMIT => 3}
	# 指定列族
scan 'test:test_hbase',{COLUMNS => ['info:name','info:age'],LIMIT =>2,STARTROW => '1'}
	# 模糊查
scan 'table_name',{STARTROW => '3513',LIMIT => 3}
# 查表,指定rowkey
get 'test:test_hbase','1'
# 查表,指定字段值
scan 'test:test_hbase',FILTER=>"ValueFilter(=,'binary:field_value')"
scan 'test:test_hbase',{FILTER=>"ColumnPrefixFilter('field_name') AND ValueFilter(=,'binary:field_value')"}
# 查看表是否存在
exists 'test'
# 查表行数
count 'test:test_hbase'
# 查看表是否可用
is_disabled 'test:test_hbase'
is_enabled 'test:test_hbase'
# 删除表数据
delete 'test:test_hbase','1','info:name'
deleteall 'test:test_hbase','1'
# 删除表
disable 'test:test_hbase'
drop 'test:test_hbase'
# 激活表
enable 'test:test_hbase'
# 清空表
truncate 'test:test_hbase'
# 修改表版本
alter 'test:test_hbase',NAME =>'info',VERSIONS =>3
# 删除表列族
alter 'test:test_hbase',{NAME => 'delete',NAME => 'info'}
# 克隆表
snapshot 'sourceTable', 'sourceTable-snapshot'
clone_snapshot 'sourceTable-snapshot', 'newTable'
# Hive外部表印射
drop table if exists test_hbase;
create external table test_hbase(key string,name string, age int) 
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
with serdeproperties ("hbase.columns.mapping" = ":key,info:name,info:age") 
tblproperties ("hbase.table.name" = "test:test_hbase", "hbase.mapred.output.outputtable" = "test_hbase");
# 统计行数
count 'test:test_hbase'
hbase org.apache.hadoop.hbase.mapreduce.RowCounter 'test:test_hbase'
```

# 性能优化

## 默认配置路径

- https://hbase.apache.org/book.html#config.files

## zookeeper.session.timeout=180s

> 这个值越小,当RS故障时Hmaster获知越快,Hlog分裂和region 部署越快,集群恢复时间越短
>
> 但是,设置这个值得原则是留足够的时间进行GC回收,否则会导致频繁的RS宕机一般就做默认即可

## hbase.client.scanner.caching=1

> 这是设计客户端读取数据的配置调优,在hbase-site.xml中进行配置,代表scanner一次缓存多少数据（从服务器一次抓取多少数据来scan）默认的太小,但是对于大文件,值不应太大

## hbase.regionserver.lease.period=60000

> 客户端租用HRegion server 期限,即超时阀值
>
> 这个配合hbase.client.scanner.caching使用,如果内存够大,但是取出较多数据后计算过程较长,可能超过这个阈值,适当可设置较长的响应时间以防被认为宕机

## hbase.regionserver.handler.count=10

> 对于大负载的put（达到了M范围）或是大范围的Scan操作,handler数目不易过大,易造成OOM
>
> 对于小负载的put或是get,delete等操作,handler数要适当调大根据上面的原则,要看我们的业务的情况来设置具体情况具体分析

## hbase.hregion.max.filesize=256M

> hbase自动拆分region的阈值,可以设大或者无限大,无限大需要手动拆分region,懒的人别这样

## hbase.hstore.blockingStoreFiles=7

> 在flush时,当一个region中的Store（CoulmnFamily）内有超过7个storefile时,则block所有的写请求进行compaction,以减少storefile数量

## hbase.hregion.memstore.flush.size

> 单个region内所有的memstore大小总和超过指定值时,flush该 region 的所有 memstore

## hbase.hregion.memstore.block.multiplier=2

> 当一个region里总的memstore占用内存大小超过 hbase.hregion.memstore.flushsize 两倍的大小时,block 该 region 的所有请求,进行flush,释放内存
>
> 虽然我们设置了region所占用的memstores总内存大小,比如64M
>
> 但想象一下,在最后63.9M的时候,我Put了一个200M的数据,此时memstore的大小会瞬间暴涨到超过预期的 hbase.hregion.memstore.flush.size 的几倍
>
> 这个参数的作用是当 memstore 的大小增至超过 hbase.hregion.memstore.flush.size 2倍时,block所有请求,遏制风险进一步扩大

## hbase.regionserver.global.memstore.upperLimit=40%

## hbase.regionserver.global.memstore.lowerLimit=35%

> 在日志中,表现为 "** Flushthread woke up with memory above low water"

## hfile.block.cache.size=0.2

> 由于blockcache是一个LRU,因此blockcache达到上限(heapsize * hfile.block.cache.size)后,会启动淘汰机制,淘汰掉最老的一批数据
>
> 对于注重读响应时间的系统,应该将blockcache设大些,比如设置 block.cache=0.4,memstore=0.39,这会加大缓存命中率

## hbase.regionserver.hlog.blocksize

## hbase.regionserver.maxlogs

> WAL的最大值为这2个的乘积,一旦达到这个值,Memstore flush就会被触发

## dfs.replication.interval=3

> 可以调高,避免hdfs频繁备份,从而提高吞吐率

## dfs.datanode.handler.count=10

## dfs.namenode.handler.count=8

> 可以调高这个处理线程数,使得写数据更快

## dfs.datanode.socket.write.timeout=480s

## dfs.socket.timeout

> 并发写数据量大的时候可以调高一些,否则会出现我另外一篇博客介绍的的错误