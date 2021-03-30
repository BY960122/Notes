```mysql
create definer=`mysql16`@`%` procedure `cq_datamap`.`update_datasource_grid`()
begin
    declare id varchar(100);
    declare yd_max_time varchar(30);
    declare count1 int;
    declare done int default 0;
    
    -- 移动
    declare yd cursor for  
    select 
        code,
        person_count 
    from datasource_grid b 
    where operators_name ='移动' and b.person_count > 0 
    and not exists (select 1 from datasource_grid a where a.code = b.code and a.operators_name = b.operators_name and a.count_time > b.count_time and a.person_count > 0);

    declare continue handler for sqlstate '02000' set done = 1;
    select @yd_max_time:=max(count_time)  from datasource_grid where operators_name ='移动';   

    open yd;
        yd_xxx:loop
        fetch yd into id,count1;
        if done = 1 then 
            leave yd_xxx;
        end if;
        update datasource_grid set person_count=count1 where code=id and count_time=yd_max_time and operators_name ='移动';
        set done = 0;
    end loop yd_xxx;
    close yd;
end
```