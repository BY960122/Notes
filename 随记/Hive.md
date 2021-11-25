# Hive 开窗函数
> 普通的聚合函数聚合的行集是组,开窗函数聚合的行集是窗口。
> 普通的聚合函数每组(Group by)只返回一个值,而开窗函数则可为窗口中的每行都返回一个值。
> 查询的结果多出一列,这一列可以是聚合值,也可以是排序值。
> 开窗函数一般分为两类,聚合开窗函数和排序开窗函数。
- 聚合开窗: count(field) over(),sum,min,max,avg,first_value,last_value,lag(col,n,default) 用于统计窗口内往上第n个值,反之 lead
- 排序开窗: rank(1,1,3),dense_rank(1,1,2),row_number(1,2,3),percent_rank

# Hive 分区分桶
> 分区: 通过表分区能够在特定的区域检索数据,减少扫描成本,在一定程度上提高查询效率
> 分桶: 某一列,让该列数据按照哈希取模的方式随机、均匀地分发到各个桶文件中
> 可以同时分区分桶,每个表分区下都有N个桶,插入时要指定桶字段,或者 SET hive.enforce.bucketing=true;
- INSERT (INTO|OVERWRITE) TABLE <bucketed_table> SELECT <select_statement> DISTRIBUTE BY <bucket_key>
> 如果分桶表创建时定义了排序键,那么数据不仅要分桶,还要排序
- 如果分桶键和排序键不同,且按降序排列,使用Distribute by … Sort by分桶排序
- 如果分桶键和排序键相同,且按升序排列（默认）,使用 Cluster by 分桶- 