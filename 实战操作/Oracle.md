# 查看表结构
```sql
select 
	owner
	,table_name
	,column_name
	,column_id
	,data_type
	,data_length 
from all_tab_columns 
where table_name in
('wskh_base_dictionary','wskh_base_organization','wskh_base_sys_user','wskh_mess_task','wskh_user_id_info','wskh_user_presence')
order by table_name,column_id;
```