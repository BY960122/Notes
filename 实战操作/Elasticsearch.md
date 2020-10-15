# 性能调优 
## 索引 merge 最大线程数
> 先通过 GET \_nodes/{node}/hot_threads 查看线程栈,查看是哪个线程占用 cpu 高
> > 如果是 elasticsearch[{node}][search][T#10] 则是查询导致的
> > 如果是 elasticsearch[{node}][bulk][T#1] 则是数据写入导致的
> >
> > > 在实际调优中,cpu 使用率很高,如果不是 SSD,建议把
```sh
index.merge.scheduler.max_thread_count = 1
```
>>> 该参数可以有效调节写入的性能。因为在存储介质上并发写,由于寻址的原因,写入性能不会提升,只会降低。

## 索引 刷新频率
> 索引的 refresh 会产生一个新的 lucene 段, 这会导致频繁的合并行为,如果业务需求对实时性要求没那么高,可以将此参数调大,实际调优告诉我,该参数确实很给力,cpu 使用率直线下降
```sh
# 默认是 1
index.refresh_interval = 1
```

## 索引 缓存大小
> index buffer 的大小是所有的 shard 公用的,一般建议,对于每个 shard 来说,最多给 512mb,因为再大性能就没什么提升了。
> ES 会将这个设置作为每个 shard 共享的 index buffer,那些特别活跃的 shard 会更多的使用这个 buffer。默认这个参数的值是 10%,也就是 jvm heap 的 10%
```sh
indices.memory.index_buffer_size=1024
```

## translog 到磁盘
> ES 为了保证数据不丢失,每次 index、bulk、delete、update 完成的时候,一定会触发刷新 translog 到磁盘上
> 在提高数据安全性的同时当然也降低了一点性能。如果你不在意这点可能性,还是希望性能优先,可以设置如下参数
```json
"index.translog": {
            "sync_interval": "120s",    // sync间隔调高
            "durability": "async",       // 异步更新
            "flush_threshold_size":"1g"// log文件大小
        }

```

## 其它参数
- discovery.zen.ping_timeout                判断 master 选举过程中，发现其他 node 存活的超时设置
- discovery.zen.fd.ping_interval            节点被 ping 的频率，检测节点是否存活
- discovery.zen.fd.ping_timeout           节点存活响应的时间，默认为 30s，如果网络可能存在隐患，可以适当调大
- discovery.zen.fd.ping_retries ping     失败/超时多少导致节点被视为失败，默认为 3
