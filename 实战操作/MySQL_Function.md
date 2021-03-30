## 模仿spilt函数
```mysql
-- 用法
select f_split_pos('//1//2//3//4//5//6//7//8//9//','//',3);
-- 返回:3
```
### 第一版,弊端:没有考虑开头也可能有分隔符,不用担心传入的pos参数有问题
```mysql
create definer=`root`@`%` function `f_split_pos`(
    `s` varchar(2000),
    `split` varchar(10),
    `pos` int) returns varchar(200) charset utf8
    begin
            declare splitlen int;
            declare rs varchar(200);
            declare tmppos int;
    ##注意:sqlserver 的所有变量要加@,mysql不用
            set tmppos = 0;
            if ifnull(pos,0) = 0 
                then set pos = 1;
            end if;
        
            if s not like concat('%',split)
            ##sqlserver '%' + 'split' 
                then set s = concat(s,split);
                ##sqlserver s + split
            end if;
        
            set splitlen = length(concat(split,'a')) - 1;
            ##sqlserver len(split + 'a')
        
            while instr(s,split) > 0 do
            ##sqlserver charindex(split,s)
                set tmppos = tmppos + 1;
            
                if tmppos = pos 
                    then set rs = substring(s,1,instr(s,split) - 1);
                end if;
            
                set s = stuff(s,1,instr(s,split) + splitlen - 1,'');
                ##mysql自定义stuff函数: replace(f_old,substring(f_old,f_start,f_length),f_replace);
                ##sqlserver stuff(s,1,instr(s,split) + splitlen - 1,'');
                ##注释: stuff(1要放入字符串,2开始位置,3删除的长度,4用什么来替换)
            end while;
        return ifnull(rs,'-1');
    end
```
### 第二版,弊端:还是要遍历,假如长度有100个单位,取第99个,那就要遍历99次
```mysql
create definer=`root`@`%` function `bingo`.`f_split_pos`(
    `s` blob, -- aa/bb/cc/dd
    `split` varchar(10), -- / 
    `pos` int) returns varchar(200) charset utf8
    begin
            declare rs varchar(200);
            declare tmppos int default 0;
        
            -- 如果开头有分隔符就去掉
            if s like concat(split,'%')
                then set s = substring(s,length(split)+1,length(s));
            end if;
    
            -- 结尾加一个分隔符
            if s not like concat('%',split)
                then set s = concat(s,split);
            end if;
        
            -- 只要分隔符出现的位置大于0,就说明还没有遍历到最后
            while instr(s,split) > 0 do
                set tmppos = tmppos + 1;
            
                -- 如果遍历到了参数的位置,则返回这段,否则接着遍历
                if tmppos = pos 
                    then set rs = substring(s,1,instr(s,split) - 1);
                end if;
            
                -- 没遍历到就把这截删了
                set s = replace(s,substring(s,1,instr(s,split) + length(split) - 1),'');
            end while;
        return ifnull(rs,'-1');
    end
```
### 第三版,直接截取需要的位置,不用循环,性能大大提升
```mysql
CREATE DEFINER=`root`@`%` FUNCTION `bingo`.`f_split_pos`(
    `s` blob, 
    `split` varchar(10), 
    `pos` int) RETURNS varchar(200) CHARSET utf8
    begin
        -- 返回结果
        declare rs varchar(200);
        -- 单位长度
        declare dw_length int default 0;
        
        -- 如果开头有分隔符就去掉,因为开头有分隔符会影响运算单位长度
        if s like concat(split,'%')
            then set s = substring(s,length(split)+1,length(s));
        end if;
        
        -- 直接从需要的位置-1(pos-1)+1,也就是当前pos的起始位置,然后截取单位长度-分隔符长度.
        set dw_length = instr(s,split) + length(split)-1;
        set rs = substring(s,dw_length*(pos-1)+1,dw_length-length(split));
    
        return ifnull(rs,'-1');
    end
```

## 模仿实现排名函数
```mysql
-- 数据准备
drop table if exists players;
create table players (
age int not null
) engine=innodb;
insert into players values (10),(10),(11),(11),(12),(12),(12),(12),(13),(13),(20),(21),(22);
```
### 1.实现 row_number
```mysql
-- 思路:从 1 开始递增即是排名
-- 变量 @currank : 当前排名
select 
    age, 
    @currank := @currank + 1 as my_row_numer -- 从 1 开始递增即是排名
from players p, 
    (select @currank := 0) q
order by age;
```
### 2.实现 dense_rank
```mysql
-- 思路:把上一行 age 存起来,每次都判断,如果相等排名不变,否则排名 + 1
-- 变量 @currank : 当前排名
-- 变量 @preage : 上一行 age 存起来
select 
    age, 
    (case when @preage = age then @currank -- 如果 age 不变,则排名不变,@preage 也不用再赋新值 
    when @preage := age then @currank := @currank + 1 end) as my_dense_rank -- 否则赋值 age , 排名 + 1
from players p, (select @currank :=0, @preage := null) r
order by age;
```
### 3.实现 rank 
```mysql
-- 思路:把上一行 age 存起来,每次都判断,如果相等排名不变,否则排名 + 1,不同的是:因为排名要跳过,所以排名需要第三个变量再存起来
-- 变量 @currank : 当前排名
-- 变量 @preage : 上一行 age 存起来
-- 变量 @incrank : 自增变量
select 
    age,
    my_rank 
from (
    select 
    age,
    (@currank := if(@preage = age, @currank, @incrank)) as my_rank, -- 如果 age 不变,排名不变,否则 把递增的值 赋值 给当前排名
    @incrank := @incrank + 1, -- 不管条件,只管自增
    @preage := age -- 每一次都要把 age 赋值一次
    from players p, 
    (select @currank :=0, @preage := null, @incrank := 1) r 
    order by age
) s;
```
### 4.合并版
```mysql
-- 实现 rank 过程中的变量 @incrank 包装一下可以当成 row_numer ,dense_rank 必须单独弄个变量,不然都会乱掉
select 
    age,
    my_rank,
    my_dense_rank,
    my_row_number
from (
    select 
    age,
    @currank := if(@preage = age, @currank, @incrank) as my_rank,
    ((@incrank := @incrank + 1) - 1) as my_row_number,
    (case when @preage = age then @dense_rank else @dense_rank := @dense_rank + 1 end) as my_dense_rank,
    @preage := age
    from players p, 
    (select @currank :=0, @preage := null, @incrank := 1,@dense_rank:=0) r 
    order by age 
) s;
```
### 5.mysql 8.0 版本已实现
```mysql
select 
    age,
    row_number() over(order by age) as my_row_number,
    rank() over(order by age) as my_row_rank,
    dense_rank() over(order by age) as my_dense_rank
from players;
```