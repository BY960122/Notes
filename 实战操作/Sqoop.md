## 显示数据库
```sh
sqoop list-databases --connect jdbc:mysql://39.106.8.170:3306/ --username root --password vdata0708
```

## 显示数据库中的表
```sh
sqoop list-tables --connect jdbc:mysql://39.106.8.170:3306/ --username root --password vdata0708
```

## 执行sql并返回结果
```sh
sqoop eval --connect jdbc:mysql://192.168.1.6:3306/ecif_etl_test --driver com.mysql.cj.jdbc.Driver --username root --password By9216446o6 -e "SELECT client_id,client_majorcate FROM t_ods_bib_t_cm_clientinfo_hv" 
```

## MySQL中的表复制到hive
```sh
sqoop import --connect jdbc:mysql://192.168.1.6:3306/ecif_etl_test \
--driver com.mysql.cj.jdbc.Driver --username root --password By9216446o6 \
--table t_ods_bib_t_cm_clientinfo_hv --where 'oc_date = 20191201' \
--fields-terminated-by '|' \
--hive-import --hive-overwrite \
--hive-database ods_bib --hive-table t_ods_bib_t_cm_clientinfo_hv \
--hive-partition-key part_init_date --hive-partition-value 20191201 \
--m 1
    ## 可选参数
--table t_ods_bib_t_cm_clientinfo_hv \
--where 'oc_date = 20191201' \
--query 'select * from t_ods_bib_t_cm_clientinfo_hv where oc_date = 20191201' \  必须要指定 target-dir ????
--create-hive-table
--hive-overwrite 
--hive-partition-key part_init_date
--hive-partition-value 20191201
--as-parquetfile \
    ## 必填参数
--hive-import

    ## 第一次执行中断,第二次执行会说hdfs目录已经存在
hdfs dfs -rm -r /user/root/t_ods_bib_t_cm_clientinfo_hv
```

## MySQL中的查询导入到hive parquent格式的表
```sh
## 注意:
## 1.decimal字段为空map,reduce阶段会报错,设置--columns参数,最好保证两边表结构一致,并严格限定默认值
## 2.parquent表无法覆盖,记得执行 hdfs dfs -rm -rf ....
sqoop import \
--connect jdbc:mysql://192.168.1.6:3306/ecif_etl_test \
--driver com.mysql.cj.jdbc.Driver --username root --password By9216446o6 \
--table t_ods_bib_t_cm_clientinfo_hv --where 'oc_date = 20191202' \
--columns client_id,client_name,oc_date \
--hive-overwrite \
--hcatalog-database ods_bib --hcatalog-table t_ods_bib_t_cm_clientinfo_hv \
--hcatalog-partition-keys part_init_date --hcatalog-partition-values 20191202 \
--m 1

    ## 写不写不影响
## -Dsqoop.avro.decimal_padding.enable=true -Dsqoop.parquet.logical_types.decimal.enable=true \
## -Dsqoop.avro.logical_types.decimal.default.precision=38 \
## -Dsqoop.avro.logical_types.decimal.default.scale=10 \
## --fields-terminated-by '|' \
    ## 亲测无效 ??
## --null-string ' ' \
## --null-non-string ' ' \
```

## MySQL导入数据到HDFS
```sh
sqoop import --connect jdbc:mysql://192.168.0.201/hive --username root --password 962464 --table table_name --columns *** --target-dir /data/ --delete-target-dir --fields-terminated-by '|' -m 1 --as-parquetfile 

    ## 可选参数
--target-dir /user/hive/warehouse/dxyjpt.db/yddt/day=20170601 
```

## 导出数据到mysql关系型数据库
```sh
sqoop export --connect jdbc:mysql://39.106.8.170:3306/Demo \
--username root --password vdata0708 --driver com.mysql.jdbc.Driver 
--table studentdemo  
--export-dir /bingyi/day19/sqoop/demo
```

## 将命令封装进sqoop脚本 myopt.opt 格式必须是一行一行的[参数or内容]
```sh
vim myopt.opt

import 
--connect 
    jdbc:mysql://192.168.1.6:3306/ecif_etl_test
--driver
    com.mysql.cj.jdbc.Driver
--username
    root
--password
    By9216446o6
--table
    t_ods_bib_t_cm_clientinfo_hv
--fields-terminated-by
    '|'
--hive-import
--hive-overwrite
--hive-database 
    ods_bib 
--hive-table 
    t_ods_bib_t_cm_clientinfo_hv_text
--m
    1

sqoop --options-file myopt.opt
```
