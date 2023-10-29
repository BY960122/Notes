## 连接Ftp

```bat
echo off
set ftpUser=wqy_1
set ftpPass=wqy@123
set ftpIP=10.253.59.199
set ftpFolder=/busy/
set LocalFolder=E:/test/busy
set ftpFile=%test%/task.txt

echo open %ftpIp% > abc.txt
echo user %ftpUser% %ftpPass% >> abc.txt

echo cd %ftpFolder% >> abc.txt

echo lcd %LocalFolder% >> abc.txt
echo prompt off >> abc.txt
echo bin >> abc.txt
echo mget *.txt >> abc.txt
echo mdelete *.txt >> abc.txt
echo bye >>abc.txt
ftp -n -s:abc.txt
bye

pause

setlocal enabledelayedexpansion

for %%i in (E:\test\busy\*.txt) do ( set var=%%i
set aa=!var:\=\\!
echo !aa!
pause
mysql -u hadoop -phadoop tiny_area --execute="DROP TABLE IF EXISTS temp_fact_jzhmdjcs_busy;CREATE TABLE IF NOT EXISTS temp_fact_jzhmdjcs_busy (PhoneNO bigint(20) NOT NULL,StationID varchar(100) NOT NULL,StationName varchar(20) DEFAULT NULL,RegCnt double DEFAULT NULL,LNG varchar(10) DEFAULT NULL,LAT varchar(10) DEFAULT NULL,rate double DEFAULT NULL,Mon int(11) NOT NULL,Company varchar(6) DEFAULT NULL) ENGINE=myisam DEFAULT CHARSET=utf8 PARTITION BY HASH (Mon) PARTITIONS 12;INSERT INTO temp_fact_jzhmdjcs_busy SELECT a.* FROM fact_jzhmdjcs_busy a INNER JOIN (SELECT mon FROM fact_jzhmdjcs_busy GROUP BY mon,lat ORDER BY CAST(lat AS DECIMAL (10,0)) DESC LIMIT 4) b ON a.mon = b.mon;LOAD DATA LOCAL INFILE '!aa!' INTO TABLE temp_fact_jzhmdjcs_busy FIELDS TERMINATED by '|';ALTER TABLE temp_fact_jzhmdjcs_busy ENGINE = INNODB;ALTER TABLE temp_fact_jzhmdjcs_busy ADD PRIMARY KEY (Mon,StationID,PhoneNO);DROP TABLE IF EXISTS fact_jzhmdjcs_busy_bak;RENAME TABLE fact_jzhmdjcs_busy TO fact_jzhmdjcs_busy_bak;RENAME TABLE temp_fact_jzhmdjcs_busy TO fact_jzhmdjcs_busy;"
)
pause
del /s /Q E:\test\busy\
```

## 文件合并

```bat
@echo off
set dest=D:\KLI\RAW_TO_ODS\ktr.txt
set src=D:\KLI\RAW_TO_ODS\SJCK
echo >%dest%
for /r "%src%" %%i in (*.ktr) do (
rem 添加分割线
echo --------------------------------------- >> %dest%
rem 输出文件路径
echo %%i >> %dest%
rem 输出文件内容
type "%%i" >> %dest%
)
```

## 批量提取指定后缀的文件名

```bat
for /r D:\Schoolbags %%a in (*.pdf) do echo %%~na >>生成文件.txt
```

## 增量拷贝文件夹 /d 为增量

```bat
@echo off
set target_dir=C:\mysoft\Apache-maven-3.6.3\mvnRespo
set src_dir=\\tsclient\D\Mysoft\apache-maven-3.6.3\mvnRespo
xcopy /S /E /H /R /K /Y /D %src_dir% %target_dir%
pause
```

## 快速关机

```bat
shutdown.exe /s /t 0
```

## 清理临时文件

```bat
RD %TEMP% /S/Q
MKDIR %TEMP%
```