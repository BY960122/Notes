## 导入导出
```mysql based
-- 导入文件
-- 用navicat不能使用local关键字
    -- replace 取代,覆盖
load data local infile 'd:\\paasform.txt' [replace] into table bingyi  
fields terminated by '|'
[optionally] enclosed by '"'  -- 如果字段本身有 "a",那么这里就是去掉"",optionally,则只对字符串类型的字段使用encloed-by字符"包裹"
escaped by '/'  -- 转义字符
lines terminated by '/n' -- 行分隔符
ignore 1 lines;  -- 忽略首行
-- 导入sql文件
source d:/mysoft/mysql-8.0.21/sakila-db/sakila-schema.sql
-- 导出
-- 导出_不带表头
select * from dim_lg_station into outfile 'd:\\dim_lg_station.txt'
fields terminated by '|';
-- 导出_带表头
select * from (select 'formid','form','dept','offi','manager','amount','pathfull','begindate','enddate','isimport' union select * from bingyi ) b
into outfile 'd:\\999.txt'
fields terminated by '|';
-- 导出一张表的sql文件
mysqldump -u root -p database table > table.sql
-- 导出整个库
mysqldump -u root -p database > database.sql
-- 导出存储过程
-- d 表示 no-create-db
-- n 表示 no-data
-- t 表示 no-create-info
-- r 表示 导出 function , procedure
mysqldump -u root -p -n -t -d -r --triggers=false database > database.sql
-- 这样导入时,会出现新的问题
-- errorcode:1418,this function has none of deterministic, nosql, or reads sql data inits declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)
-- 解决方法是,在/etc/my.cnf中找到[mysqld],在它下面添加这样一行
log-bin-trust-function-creators=1
```

## 常用命令
```mysql
-- 修改mysql提示符
# prompt \u@\h \d>
-- 查看存储过程
-- mysql 5.x
select `name` from mysql.proc where db = 'your_db_name' and `type` = 'procedure';
-- mysql 8.x
select * from information_schema.routines where routine_name = 'test_proc';
-- 通用
show procedure status;
-- 查看数据库进程
select * from information_schema.processlist where db = 'bingo';
-- 查看数据表信息(行数)
select * from information_schema.tables where table_schema = 'bingo';
-- 查看数据表列信息(类型,注释)
select * from information_schema.columns where table_schema = 'bingo';
-- 修改字符集
alter table t_ods_tgdb_t_dc_glr_info_hv default character set utf8 collate utf8_general_ci;
alter database ecif_etl_test default character set utf8 collate utf8_general_ci;
alter table t_ods_tgdb_t_dc_glr_info_hv convert to character set utf8 collate utf8_general_ci;
-- 解析json
select json_value ('{"firstname":"john"}','$.firstname' default 'no last name found' on error) as "last name";
-- 加密函数
select md5('962464');
select password('by962464');

select to_base64('中');
select from_base64('5Lit');
-- ip存储转换函数
select inet_aton('192.168.0.102');
select inet_ntoa('3232235622');
-- 中文与uniocde互转
select hex(convert('中' using ucs2)) as chinese;
select ord(convert('中' using ucs2)) as chinese;
-- 数字格式化
select format(123456.789,2);
-- 大小写转换
select lower('mysql');
select upper('mysql');
-- 获取左右字符
select left('mysql',2);
select right('mysql',2);
-- 删除前后空格
select ltrim('  mysql   ') as a;
select rtrim('  mysql   ') as a;
-- 删除指定字符
select trim(leading '?' from '??mysql???');
select trim(trailing '?' from '??mysql???');
-- 替换字符
select replace('??my?sql???','?','');
-- 截取字符
select substr('mysql',1,2);
select substr('mysql',-1);
select substr('mysql',3);
-- 行转列
select group_concat(id) from partition_list;
-- 去除换行符回车符
update table_name set field_name = replace(replace(field_name,char(10),''),char(13),'');

-- 清除查询缓存
-- 要开启才行
-- query_cache_size=10m
-- query_cache_type=1
reset query cache;
```

## 日期
```mysql
-- 当前日期和时间
select now();
-- 当前日期,时间
select curdate();
select current_time();
select from_unixtime(unix_timestamp());
-- 添加日期
select date_add('2018-08-19',interval 365 day);
select date_add('2018-08-19',interval -3 month);
select date_add('2018-08-19',interval -1 year);
-- 计算日期差值
select datediff('2018-08-19','2017-02-25');
-- 日期格式化
select date_format('2018-08-19','%y%m%d');
-- 日期转时间戳
select concat(unix_timestamp('2018-10-31 23:45:00'),'000') from test limit 1;
-- 时间戳转日期
select from_unixtime(substring(1541000700000,1,10)) from test limit 1;
-- 周第一天,最后一天
select date_add(now(),interval - weekday(now()) day);
select date_add(now(),interval - weekday(now()) + 6 day);
-- 月第一天,最后一天
select concat_ws('-',substring(now(),1,7),'01');
select last_day(now());
-- 季第一天,最后一天
select date(concat_ws('-',year(now()),elt(quarter(now()),1,4,7,10),1));
select last_day(makedate(year(now()),1) + interval quarter(now())*3-1 month);
```

## 经纬度
```mysql
-- 找到距离纬度:78.3232,经度:65.3234坐标0.4公里里范围内最近的20个位置
select  
  id
  ,(6371 * acos(
  			cos(radians(78.3232)) 
  			* cos(radians(数据库纬度字段)) 
  			* cos(radians(数据库经度字段) - radians(65.3234)) 
  			+ sin(radians(78.3232)) 
  			* sin(radians(数据库纬度字段)))) as distance
from tb_hotel 
having distance < 0.4 
order by distance 
limit 0 , 20;

-- http://www.arubin.org/files/geo_search.pdf
select 
    3956 * 2 * asin(
                sqrt(
                    power(sin((@orig_lat - abs(dest.lat)) * pi() / 180 / 2), 2) 
                    + cos(@orig_lat * pi() / 180) 
                    * cos(abs(dest.lat) * pi() / 180) 
                    * power(sin((@orig_lon - dest.lon) * pi() / 180 / 2), 2))) as distance 
from hotels dest
having distance < @dist order by distance limit 10;

select 
    3956 * 2 * asin(
                sqrt(
                    power(sin((orig.lat - dest.lat) * pi() / 180 / 2), 2) 
                    + cos(orig.lat * pi() / 180) 
                    * cos(dest.lat * pi() / 180) 
                    * power(sin((orig.lon -dest.lon) * pi() / 180 / 2), 2))) as distance 
from hotels dest order by distance limit 10;


```

## 事件
```mysql based
-- 创建事件
    -- 重点人员轨迹信息,每2小时跑一次
create event if not exists event_zdryyj 
on schedule every 2 hour starts '2019-10-16 15:00:00'
on completion preserve enable 
do 
begin 
    call p_analysis_zdryjy();
end

-- 删除Kakfa历史数据
create event if not exists event_delete_kafka 
on schedule every 1 day starts date_add(date_add(curdate(),interval 1 day),interval 1 hour) -- 每天01:00执行
on completion preserve enable 
do 
begin 
    call p_delete_kakfa();
end 

-- 激活事件
show variables like '%event%';
set global event_scheduler = true;
set global event_scheduler = false;

-- 查看事件执行情况
select 
    event_schema,event_name,
    interval_value,interval_field,
    status, 
    created,last_altered,last_executed
from information_schema.events 
where event_schema = 'bingo_zntszs';

-- 删除事件
drop event test;
```

## 分区表
```mysql based
-- 创建分区表
create table partition_list(
id int,
dt int,
primary key (id,dt)) engine = innodb auto_increment 9 default charset = utf8
partition by list columns(dt)(
partition p1 values in (20190101),
partition p2 values in (20190201),
partition p3 values in (20190301),
partition p4 values in (20190401),
partition p5 values in (20190501),
partition p6 values in (20190601),
partition p7 values in (20190701),
partition p8 values in (20190801),
partition p9 values in (20190901),
partition p10 values in (20191001),
partition p11 values in (20191101),
partition p12 values in (20191201)
);

-- 插入数据
insert into partition_list value (1,20190101),(2,20190101),(3,20190101),(4,20190101),(5,20190101),(6,20190101),(7,20191001);
-- 查询分区表
select * from information_schema.`partitions` where table_name = 'partition_list';
-- 查询某一分区值所在分区名
select partition_name from information_schema.`partitions` where table_name = 'partition_list'and partition_description = 20191001;
-- 删除某个分区数据
alter table partition_list truncate partition p2;
-- 动态获取分区值
select @var1 := partition_name from information_schema.`partitions` where table_name = 'partition_list' and partition_description = 20191001;
    -- 测试一下是否赋值成功
select @var1;
    -- 用存储过程操作
create definer=`root`@`%` procedure `truncate_partition_yh`(
in table_name varchar(50),
in dt_name varchar(20)
)
begin
    if dt_name = 20190101 then
        set @sqlcmd = concat('alter table ',table_name,'truncate partition p1');
        prepare temp from @sqlcmd;
        execute temp;
    elseif dt_name = 20190201 then
        set @sqlcmd = concat('alter table ',table_name,'truncate partition p2');
        prepare temp from @sqlcmd;
        execute temp;
    else
        set @sqlcmd = concat('alter table ',table_name,'truncate partition p13');
        prepare temp from @sqlcmd;
        execute temp;
    end if;
end
-- 调用存储过程
call truncate_partition_yh('partition_list',20190101);
-- 检查数据
select partition_description,partition_name,table_rows from information_schema.`partitions` where table_name = 'partition_list';
```