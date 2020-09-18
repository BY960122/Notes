# 下载 Sqoop
```http
http://mirror.bit.edu.cn/apache/sqoop/
http://archive.apache.org/dist/sqoop/
```
## 1.修改配置文件
### qoop-env.sh
```sh
export HADOOP_COMMON_HOME=/opt/software/hadoop-3.2.1
export HADOOP_MAPRED_HOME=/opt/software/hadoop-3.2.1
export HBASE_HOME=/opt/software/hbase-2.3.1
export HIVE_HOME=/opt/software/apache-hive-3.1.1-bin
export ZOOCFGDIR=/opt/software/apache-zookeeper-3.6.1-bin
```
## 2.拷贝jar至sqoop/lib
```sh
cp /opt/software/apache-hive-3.1.1-bin/lib/mysql-connector-java-8.0.18.jar /opt/software/sqoop-1.4.7.bin__hadoop-2.6.0/lib/
cp /opt/software/apache-hive-3.1.1-bin/lib/* /opt/software/sqoop-1.4.7.bin__hadoop-2.6.0/lib/
cp /opt/software/apache-hive-3.1.1-bin/conf/hive-site.xml /opt/software/sqoop-1.4.7.bin__hadoop-2.6.0/conf/
```
## 3.验证
```sh
sqoop version
sqoop list-databases --connect jdbc:mysql://192.168.1.6/hive --username root --password By9216446o6
```

# 阿里 DataX
```http
https://github.com/alibaba/DataX/blob/master/userGuid.md
```
## 1.范例配置
```json
{
    "job": {
        "setting": {
            "speed": {
                "channel": 3
            }
        },
        "content": [{
            "reader": {
                "name": "mysqlreader",
                "parameter": {
                    "column": [
                        "*"
                    ],
                    "connection": [{
                        "jdbcUrl": ["jdbc:mysql://192.168.1.6:3306/ecif_etl_test"],
                        "table": [
                            "t_ods_bib_t_cm_clientinfo_hv"
                        ]
                    }],
                    "username": "root",
                    "password": "By9216446o6"	
                }
            },
            "writer": {
                "name": "hdfswriter",
                "parameter": {
                    "defaultFS": "hdfs://192.168.1.201:9000",
                    "fieldDelimiter": "|",
                    "fileName": "t_ods_bib_t_cm_clientinfo_hv",
                    "fileType": "text",
                    "encoding": "UTF-8",
                    "path": "/user/hive/warehouse/ods_bib.db/t_ods_bib_t_cm_clientinfo_hv_test2",
                    "writeMode": "append",
                    "column": [
                        "*"
                    ]
                }
            }
        }]
    }
}
```