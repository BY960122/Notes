## 建表语句
```sql
-- Distribute By: 指定 map 输出结果怎么样划分后分配到各个 Reduce 上去,比如 Distribute By
-- dealer_id: 就可以保证 dealer_id 字段相同的结果被分配到同一个 reduce 上去执行
-- Sort By:是在每个 reduce 中进行排序,是一个局部排序
-- 如果 Distribute By 和 Sort By 的字段是同一个,可以简写为 Cluster By
create table dealer_leads(
companyId INT comment '公司 ID',
userid INT comment '销售 ID',
originalstring STRING comment 'url',
host string comment 'host',
absolutepath string comment '绝对路径') 
partitioned by (part_init_date string) 
-- clustered by (dealer_id) sorted by(leads_id) into 10 buckets
row format delimited fields terminated by '|'
stored as textfile;

-- 创建parquent的表
create table (
...
)
COMMENT '客户基本信息'
PARTITIONED BY ( 
  `part_init_date` string)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='|', 
  'serialization.format'='|') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.parquet.MapredParquetOutputFormat';
```

## 常用操作
```sql
-- 修改表名
alter table dealerinfo rename to dealer_info;
-- 添加列
alter table dealer_info add columns (provinceid int);
-- 修改字段
    -- 只是修改了 Hive 表的元数据信息（元数据信息一般是存储在 MySql 中）,并不对存在于 HDFS 中的表数据做修改
    -- 并不是所有的 Hive 表都可以修改字段,只有使用了 native SerDe (序列化反序列化类型)的表才能修改字段
    -- 可以修改的字段的 SerDe 有：DynamicSerDe, MetadataTypedColumnsetSerDe, LazySimpleSerDe and ColumnarSerDe
alter table dealer_info replace columns (dealerid int,dealername string,cityid int,joindate date,provinceid int);
-- 删除表
drop table if exists dealer_info;
-- 导入
    -- 导入分桶表,这个配置非常关键,为 true 就是设置为启用分桶
    -- set hive.enforce.bucketing = true;
load data local inpath '/home/hadoop/dealerinfodata.txt' overwrite into table dealer_info;
load data local inpath '/home/hadoop/actionlog.txt' overwrite into table dealer_action_log PARTITION (dt='2016-08-19');
insert overwrite table dealer_leads select * from dealer_leads_tmp;
import table dealer_action_log_like from '/user/hive/action_log.export';
-- 导出
    -- 去掉 local 关键字,也可以导出到 HDFS 上
insert overwrite local directory '/home/hadoop/dealer_info.bak2016-08-22' select * from dealer_info;
    -- 覆盖分区
insert overwrite table dealer_leads PARTITION (dt='2016-08-31') select * from dealer_leads_tmp;
    -- 导出分区
export table dealer_action_log partition (dt='2016-08-19') to '/user/hive/action_log.export'

-- 创建索引 
create index t3_index on table t3_new(stu_name) as 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' with deferred rebuild in table t3_index_table; 
-- 显示索引 
show formatted index on t3_new; 
-- 重建索引 
alter index t3_index on t3_new rebuild; 
-- 删除索引 
drop index if exists t3_index on t3_new;
```

## 函数
```sql
-- Hive编写自定义UDF函数上传
add jar /opt/mysoft/hiveexample-1.0.jar;
create temporary function myupper as 'udf.MyUpper';
select myupper('yes') from bingo.test_hive limit 1;

create temporary function myuuid as 'udf.MyUUID';
select myuuid(9) from bingo.test_hive limit 1;

create temporary function uaf as 'udf.UDFArrayFirst';
select uaf(array(1,2,3)) from bingo.test_hive limit 1;
```

## 动态分区
```sh
set hive.exec.dynamic.partition=true;  
set hive.exec.dynamic.partition.mode=nonstrict; 
set hive.exec.max.dynamic.partitions.pernode=1000;
```

## 优化
```sql
-- 排序
select * from (
    select 
        dealer_id,count(leads_id) cnt 
    from dealer_leads where dealer_id!='0' 
    group by dealer_id
) a order by a.cnt;
-- 取前 N 条
select 
    a.leads_id,a.user_name 
from (
    select 
        leads_id,user_name 
    from dealer_leads
    distribute by length(user_name) sort by length(user_name) desc 
    limit 10
) a order by length(a.user_name) desc limit 10;
-- 行转列
select name,concat_ws(',',collect_list(favor)) as favor_list from student_favors group by name;
-- 列转行
select name,favorlist,favor from student_favors_2 view explode(split(favorlist,',') table1 as favor;
```
