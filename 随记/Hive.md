# Hive 开窗函数
> 普通的聚合函数聚合的行集是组,开窗函数聚合的行集是窗口。
> 普通的聚合函数每组(Group by)只返回一个值,而开窗函数则可为窗口中的每行都返回一个值。
> 查询的结果多出一列,这一列可以是聚合值,也可以是排序值。
> 开窗函数一般分为两类,聚合开窗函数和排序开窗函数。
- 聚合开窗: count(field) over(),sum,min,max,avg,first_value,last_value,lag(col,n,default) 用于统计窗口内往上第n个值,反之 lead
- 排序开窗: rank(1,1,3),dense_rank(1,1,2),row_number(1,2,3),percent_rank

