## 建表语句
```hiveql
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

-- 或者
create table table_name(
companyId INT comment '公司 ID',
userid INT comment '销售 ID',
originalstring STRING comment 'url'
)
COMMENT '客户基本信息'
partitioned by (part_init_date string)
row format delimited fields terminated by '\001'
stored as parquet
-- location '/user/hive/warehouse/ods/ods_ecif/t_ods_ecif_analysis_detail/'
;
```

## 常用操作
```hiveql
-- 修复表,hdfs有文件,但是meta表没有
msck repair table ecifdb.t_s008_bib_t_cm_clientinfo;
-- 修改表名
alter table dealerinfo rename to dealer_info;
-- 添加列
alter table dealer_info add columns (provinceid int);
-- 修改字段
    -- 只是修改了 Hive 表的元数据信息(元数据信息一般是存储在 MySql 中),并不对存在于 HDFS 中的表数据做修改
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
export table dealer_action_log partition (dt='2016-08-19') to '/user/hive/action_log.export';

-- 创建索引 
create index t3_index on table t3_new(stu_name) as 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' with deferred rebuild in table t3_index_table; 
-- 显示索引 
show formatted index on t3_new; 
-- 重建索引 
alter index t3_index on t3_new rebuild; 
-- 删除索引 
drop index if exists t3_index on t3_new;

-- 行转列
select 
  max(sno)
  ,name
  ,concat_ws(',', collect_set(depart)) as depart 
from students_info
group by name;

-- 列转行
select 
  sno
  , name
  , add_depart
from students_info si 
lateral view explode(split(si.depart,','))  b as add_depart;
```

## 函数
```hiveql
-- 写了库名,调用函数的时候也要加库名.
-- 临时函数
add jar /opt/mysoft/hiveexample-1.0.jar;
create temporary function myupper as 'udf.MyUpper';
select myupper('yes') from bingo.test_hive limit 1;

create temporary function myuuid as 'udf.MyUUID';
select myuuid(9) from bingo.test_hive limit 1;

create temporary function uaf as 'udf.UDFArrayFirst';
select uaf(array(1,2,3)) from bingo.test_hive limit 1;

-- 永久函数
create function myupper as 'udf.MyUpper' using jar /opt/mysoft/hiveexample-1.0.jar;

-- 查看函数
desc function myupper;


-- impala 新建临时函数(写返回值)
create function if not exists hiveudf.md5(string)
returns string 
location 'hdfs:/user/hive/warehouse/hive_md5_udf-1.0.jar' 
symbol='MD5UDF';
-- impala 新建永久函数(不写返回值)
create function if not exists hiveudf.md5
location 'hdfs:/user/hive/warehouse/hive_md5_udf-1.0.jar' 
symbol='MD5UDF';
```

## 优化
```hiveql
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

-- 避免重复group
--开启动态分区 
set hive.exec.dynamic.partition=true; 
set hive.exec.dynamic.partition.mode=nonstrict; 

from stu_ori 

insert into table stu partition(tp) 
select s_age,max(s_birth) stat,'max' tp 
group by s_age

insert into table stu partition(tp) 
select s_age,min(s_birth) stat,'min' tp 
group by s_age;
```


## 一些调优参数
```sh
# 动态分区
set hive.exec.dynamic.partition=true;  
set hive.exec.dynamic.partition.mode=nonstrict; 
set hive.exec.max.dynamic.partitions.pernode=1000;

# 并行执行优化
# Hive会将一个查询转化成一个或者多个阶段。这样的阶段可以是MapReduce阶段,抽样阶段,合并阶段,limit阶段。或者Hive执行过程中可能需要的其他阶段。
# 打开任务并行执行
set hive.exec.parallel=true;
# 同一个sql允许最大并行度,默认为8。
set hive.exec.parallel.thread.number=16;

# JVM优化
# JVM重用可以使得JVM实例在同一个job中重新使用N次,通常在10-20之间,具体多少需要根据具体业务场景测试得出
# 这个功能的缺点是,开启JVM重用将一直占用使用到的task插槽,以便进行重用,直到任务完成后才能释放
set mapred.job.reuse.jvm.num.tasks=10;
# 或者 mapred-site.xml
# <property>
#     <name>mapreduce.job.jvm.numtasks</name>
#     <value>10</value>
#     <description>How many tasks to run per jvm. If set to -1, there is no limit. 
#     </description>
# </property>

# 推测执行优化
# 在分布式集群环境下,因为程序bug(包括Hadoop本身的bug),负载不均衡或者资源分布不均等原因,会造成同一个作业的多个任务之间运行速度不一致
# Hadoop采用了推测执行(Speculative Execution)机制,它根据一定的法则推测出"拖后腿"的任务,并为这样的任务启动一个备份任务,
# 让该任务与原始任务同时处理同一份数据,并最终选用最先成功运行完成任务的计算结果作为最终结果。
# 如果用户因为输入数据量很大而需要执行长时间的map或者reduce task的话,那么启动推测执行造成的浪费是非常巨大的。
set hive.mapred.reduce.tasks.speculative.execution=true
# 或者 mapred-site.xml
<property>
    <name>mapreduce.map.speculative</name>
    <value>true</value>
    <description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
</property>
<property>
    <name>mapreduce.reduce.speculative</name>
    <value>true</value>
    <description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
</property>

# 合并小文件
# concatenate 命令只支持 RCFILE 和 ORC 文件类型。 
# 使用concatenate命令合并小文件时不能指定合并后的文件数量,但可以多次执行该命令。 
# 当多次使用concatenate后文件数量不在变化,这个跟参数 mapreduce.input.fileinputformat.split.minsize=256mb 的设置有关,可设定每个文件的最小size。
alter table A concatenate;
alter table B partition(day=20201224) concatenate;

# 设置map输入合并小文件的相关参数：
#执行Map前进行小文件合并
#CombineHiveInputFormat底层是 Hadoop的 CombineFileInputFormat 方法
#此方法是在mapper中将多个文件合成一个split作为输入
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; -- 默认

#每个Map最大输入大小(这个值决定了合并后文件的数量)
set mapred.max.split.size=256000000;   -- 256M

#一个节点上split的至少的大小(这个值决定了多个DataNode上的文件是否需要合并)
set mapred.min.split.size.per.node=100000000;  -- 100M

#一个交换机下split的至少的大小(这个值决定了多个交换机上的文件是否需要合并)
set mapred.min.split.size.per.rack=100000000;  -- 100M

设置map输出和reduce输出进行合并的相关参数:
#设置map端输出进行合并,默认为true
set hive.merge.mapfiles = true;
#设置reduce端输出进行合并,默认为false
set hive.merge.mapredfiles = true;
#设置合并文件的大小
set hive.merge.size.per.task = 256*1000*1000;   -- 256M
#当输出文件的平均大小小于该值时,启动一个独立的MapReduce任务进行文件merge
set hive.merge.smallfiles.avgsize=16000000;   -- 16M 

# 启用压缩
# hive的查询结果输出是否进行压缩
set hive.exec.compress.output=true;
# MapReduce Job的结果输出是否使用压缩
set mapreduce.output.fileoutputformat.compress=true;

# 减少Reduce的数量
#reduce 的个数决定了输出的文件的个数,所以可以调整reduce的个数控制hive表的文件数量,
#hive中的分区函数 distribute by 正好是控制MR中partition分区的,
#然后通过设置reduce的数量,结合分区函数让数据均衡的进入每个reduce即可。

#设置reduce的数量有两种方式,第一种是直接设置reduce个数
set mapreduce.job.reduces=10;
#第二种是设置每个reduce的大小,Hive会根据数据总大小猜测确定一个reduce个数
set hive.exec.reducers.bytes.per.reducer=5120000000; -- 默认是1G,设置为5G
#执行以下语句,将数据均衡的分配到reduce中
set mapreduce.job.reduces=10;
insert overwrite table A partition(dt) select * from B distribute by rand();
# 解释：如设置reduce数量为10,则使用 rand(), 随机生成一个数 x % 10 ,这样数据就会随机进入 reduce 中,防止出现有的文件过大或过小

# 使用hadoop的archive将小文件归档
# Hadoop Archive简称HAR,是一个高效地将小文件放入HDFS块中的文件存档工具,它能够将多个小文件打包成一个HAR文件,
# 这样在减少namenode内存使用的同时,仍然允许对文件进行透明的访问
#用来控制归档是否可用
set hive.archive.enabled=true;
#通知hive在创建归档时是否可以设置父目录
set hive.archive.har.parentdir.settable=true;
#控制需要归档文件的大小
set har.partfile.size=1099511627776;
#使用以下命令进行归档
alter table a archive partition(dt='2020-12-24', hr='12');
#对已归档的分区恢复为原文件
alter table a unarchive partition(dt='2020-12-24', hr='12');
```

## 奇异bug
```sh
# 用tez运算后的表,无法用mr查到,例如 union all 之后,tez会多存一级目录 HIVE_UNION_SUBDIR..
# 原因: mr不会递归查询该目录,tez会
# 建议: 将此参数添加进hive-site,允许mr递归读取目录
set mapred.input.dir.recursive=true;

# hive执行结果moveTask操作失败: Unable to move source hdfs ... to destination hdfs ... 
# 解释:hive的查询结果在在进行move操作时,需要进行文件权限的授权,多个文件的授权是并发进行的,hive中该源码是在一个线程池中执行的
# ,该操作在多线程时线程同步有问题的该异常,这是hive的一个bug,目前截止目前的最新版本Apache Hive 2.1.1还没有修复该问题
<property>
    <name>hive.warehouse.subdir.inherit.perms</name>
    <value>true</value>
    <description>
      Set this to false if the table directories should be created
      with the permissions derived from dfs umask instead of
      inheriting the permission of the warehouse or database directory.
    </description>
</property>
```