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
scan 'hbase:namespace' , {LIMIT => 3}
	# 指定列族
scan 'test:test_hbase',{COLUMNS => ['info:name','info:age'], LIMIT =>2,STARTROW => '1'}
	# 模糊查
scan 'table_name' , {STARTROW => '3513' , LIMIT => 3}
# 查表,指定列族
get 'test:test_hbase','1'
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
# Hive外部表印射
drop table if exists test_hbase;
create external table test_hbase(key string,name string, age int) 
stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' 
with serdeproperties ("hbase.columns.mapping" = ":key,info:name,info:age") 
tblproperties ("hbase.table.name" = "test:test_hbase", "hbase.mapred.output.outputtable" = "test_hbase");
```
