# 基本操作
```sh
# 查询所有节点
curl -XGET localhost:9200/_cat/nodes?v

# 查询所有的索引配置
curl -XGET localhost:9200/_settings/_all?pretty

# 创建索引 ?pretty 输出格式化
curl -XPUT localhost:9200/test1?pretty

# 插入数据 或者在在后面加上 _create
curl -XPUT 'localhost:9200/test1/_doc/1?pretty' -d '
{
  "name": "John Doe"
}'

PUT localhost:9200/test1/_doc/1?pretty
Content-Type: application/json

{
  "name": "John Doe"
}

# 查询指定索引
curl -XGET localhost:9200/test1?pretty
curl -XGET localhost:9200/test1/_doc/1?pretty
curl -XGET localhost:9200/test1/_search?q=*&pretty

# 查询所有索引
curl -XGET localhost:9200/_cat/indices?v

- 查询多条文档: https://wiki.jikexueyuan.com/project/elasticsearch-definitive-guide-cn/030_Data/50_Mget.html

# 检查索引是否存在
curl -i -XHEAD localhost:9200/test1/_doc/123

# 存储的信息都在 _source字段里,查询特定字段,或只查_source的内容
curl -XGET localhost:9200/test1/_doc/1?_source=title,text
curl -XGET localhost:9200/test1/_doc/1?_source

# 删除索引
curl -XDELETE localhost:9200/test2?pretty

# 更新数据
# 第一种: 覆盖,小心操作
curl -XPUT localhost:9200/test1/_doc/1?pretty' -d 
{
  "name": "YH"
}'

# 特别注意: 只在特定版本下生效
curl -XPUT localhost:9200/test1/_doc/1?version=1

# 第二种: 更新  
# 设置重试参数: retry_on_conflict=5 
# 超时时间: timeout=10ms 需要注意的是timeout不会停止执行查询,它仅仅告诉你目前顺利返回结果的节点然后关闭连接。在后台,其他分片可能依旧执行查询,尽管结果已经被发送
POST localhost:9200/test1/_update/1
Content-Type: application/json

{
  "doc": {
    "age": 99
  }
}

# 删除数据
curl -XDELETE localhost:9200/test1/_doc/2?pretty

# 匹配
curl -XPOST localhost:9200/bank/_search?pretty -d '
{
  "query": {
    "match_all": {}
  },
  "size": 1,
  "sort": {
    "balance": {
      "order": "desc"
    }
  }
}'

POST localhost:9200/test1/_search?pretty
Content-Type: application/json

{
  "query": {
    "match_all": {}
  },
  "size": 1,
  "sort": {
    "balance": {
      "order": "desc"
    }
  }
}

## 一些常用匹配
### 等值匹配: 返回 account_number 等于 20 的所有数据
{
  "query": {
    "match": {
      "account_number": 20
    }
  }
}

### 短语匹配: like %**%
{
  "query": {
    "match_phrase": {
      "address": "mill lane"
    }
  }
}

### 返回交集: and ,可选  must_not
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        },
        {
          "match": {
            "address": "lane"
          }
        }
      ]
    }
  }
}

### 返回并集: or
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "address": "mill"
          }
        },
        {
          "match": {
            "address": "lane"
          }
        }
      ]
    }
  }
}

### 交叉使用
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "age": "40"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "state": "ID"
          }
        }
      ]
    }
  }
}

### 范围匹配
{
  "query": {
    "bool": {
      "must": {
        "match_all": {}
      },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}

### 按 state 字段分组
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state"
      }
    }
  }
}

### select state,avg(balance) from ** group by state
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}

```

# 性能调优 
- https://www.elastic.co/guide/en/elasticsearch/reference/current

## 索引 merge 最大线程数
> 先通过 GET \_nodes/{node}/hot_threads 查看线程栈,查看是哪个线程占用 cpu 高
>> 如果是 elasticsearch[{node}][search][T#10] 则是查询导致的
>>
>> 如果是 elasticsearch[{node}][bulk][T#1] 则是数据写入导致的
>>
>>> 在实际调优中,cpu 使用率很高,如果不是 SSD,建议把
```sh
index.merge.scheduler.max_thread_count = 1
```
>>> 该参数可以有效调节写入的性能,因为在存储介质上并发写,由于寻址的原因,写入性能不会提升,只会降低,

## 索引 刷新频率
> 索引的 refresh 会产生一个新的 lucene 段, 这会导致频繁的合并行为,如果业务需求对实时性要求没那么高,可以将此参数调大,实际调优告诉我,该参数确实很给力,cpu 使用率直线下降
```sh
# 默认是 1 , 改为 -1s  这样就是不刷新
curl -XPUT 'http://localhost:9200/twitter/_settings' -d '
{
  "settings": {
    "index": {
      "refresh_interval": "60s"
    }
  }
}'
```

## 索引 缓存大小
> index buffer 的大小是所有的 shard 公用的,一般建议,对于每个 shard 来说,最多给 512mb,因为再大性能就没什么提升了,
>
> ES 会将这个设置作为每个 shard 共享的 index buffer,那些特别活跃的 shard 会更多的使用这个 buffer,默认这个参数的值是 10%,也就是 jvm heap 的 10%
```sh
indices.memory.index_buffer_size=1024
```

## translog 到磁盘
> ES 为了保证数据不丢失,每次 index、bulk、delete、update 完成的时候,一定会触发刷新 translog 到磁盘上
>
> 在提高数据安全性的同时当然也降低了一点性能,如果你不在意这点可能性,还是希望性能优先,可以设置如下参数
```json
{
  "index.translog": {
    "sync_interval": "120s",
    "durability": "async",
    "flush_threshold_size": "1g"
  }
}
```

## 优化es的线程池 
> index：此线程池用于索引和删除操作,它的类型默认为fixed,size默认为可用处理器的数量,队列的size默认为300
>
> search：此线程池用于搜索和计数请求,它的类型默认为fixed,size默认为可用处理器的数量乘以3,队列的size默认为1000
>
> suggest：此线程池用于建议器请求,它的类型默认为fixed,size默认为可用处理器的数量,队列的size默认为1000
>
> get：此线程池用于实时的GET请求,它的类型默认为fixed,size默认为可用处理器的数量,队列的size默认为1000
>
> bulk：你可以猜到,此线程池用于批量操作,它的类型默认为fixed,size默认为可用处理器的数量,队列的size默认为50
>
> percolate：此线程池用于预匹配器操作,它的类型默认为fixed,size默认为可用处理器的数量,队列的size默认为1000
```sh
threadpool.index.type: fixed
threadpool.index.size: 100
threadpool.index.queue_size: 500
```

## 定期优化清理缓存
> 如果你不想重新配置节点并且重启,你可以做一个定时任务来定时清除cache
```sh
# 清除所有索引的cache,如果对查询有实时性要求,慎用！
http://10.22.2.201:9200/*/_cache/clear
# 到了晚上资源空闲的时候我们还能合并优化一下索引
http://10.22.2.201:9200/*/_optimize
```

## 其它参数
- discovery.zen.ping_timeout                判断 master 选举过程中,发现其他 node 存活的超时设置
- discovery.zen.fd.ping_interval            节点被 ping 的频率,检测节点是否存活
- discovery.zen.fd.ping_timeout           节点存活响应的时间,默认为 30s,如果网络可能存在隐患,可以适当调大
- discovery.zen.fd.ping_retries ping     失败/超时多少导致节点被视为失败,默认为 3
