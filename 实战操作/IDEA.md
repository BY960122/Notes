### Failed to locate the winutils binary in the hadoop binary path,java.io.IOException: Could not locate executable null\bin\winutils.exe in the Hadoop binaries.
```http
<!-- 下载winutils的windows版本,增加用户变量HADOOP_HOME,值是下载的zip包解压的目录,然后在系统变量path里增加%HADOOP_HOME%\bin 即可 -->
https://github.com/steveloughran/winutils
```
### java 11 运行报错:WARNING: An illegal reflective access operation has occurred
```txt
先看下是哪些包非法访问,注意看第四行
--illegal-access=warn
**to field java.lang.String.value
于是添加JVM启动参数
--add-opens java.base/java.io=ALL-UNNAMED
```

### log4j升级为log4j2,中途还是会打印找不到log4j配置
```xml
<!-- 用maven helper插件查看 All Dependencies,把log4j,log4j-extras全部排除 -->
<!-- log4j2-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>${log4j2.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>${log4j2.version}</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>${slf4j.version}</version>
</dependency>
<!-- Bridge log4j to log4j2 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-1.2-api</artifactId>
    <version>${log4j2.version}</version>
</dependency>
<!-- Bridge slf4j to log4j2 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>${log4j2.version}</version>
</dependency>
```
### java.lang.NoSuchMethodError: io.netty.bootstrap.Bootstrap.config()Lio/netty/bootstrap/BootstrapConfig;
```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.51.Final</version>
</dependency>
```
### com.fasterxml.jackson.databind.JsonMappingException: Scala module 2.10.0 requires Jackson Databind version >= 2.10.0 and < 2.11.0
```xml
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-scala_2.11</artifactId>
    <version>2.11.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.11.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.11.1</version>
</dependency>
```
### Unable to instantiate SparkSession with Hive support because Hive classes are not found
```xml
<dependency>
    <groupId>commons-collections</groupId>
    <artifactId>commons-collections</artifactId>
    <version>3.2.2</version>
</dependency>
```
### Scala module 2.10.0 requires Jackson Databind version >= 2.10.0 and < 2.11.0-->
```xml
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-scala_2.12</artifactId>
    <version>2.11.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.11.1</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.11.1</version>
</dependency>
```
### java.lang.NoSuchMethodError: io.netty.bootstrap.Bootstrap.config()Lio/netty/bootstrap/BootstrapConfig
```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.51.Final</version>
</dependency>
```
### java.lang.NoClassDefFoundError: org/antlr/runtime/RecognitionException
```xml
<dependency>
    <groupId>org.antlr</groupId>
    <artifactId>antlr4</artifactId>
    <version>4.7.1</version>
</dependency>
```
### com.google.common.util.concurrent.MoreExecutors.sameThreadExecutor()Lcom/google/common/util/concurrent/ListeningExecutorService
```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.3.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.3.0</version>
</dependency>
```
### Spark3.0 read.json报错: error reading Scala signature of org.apache.spark.sql.package: unsafe symbol Unstable (child of package annotation) in runtime reflection universe
```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-tags_2.12</artifactId>
    <version>3.0.1</version>
</dependency>
```

### org.xerial.snappy not found
```xml
<dependency>
    <groupId>org.xerial.snappy</groupId>
    <artifactId>snappy-java</artifactId>
    <version>1.1.7.7</version>
</dependency>
```

### Caused by:java.lang.ClassNotFoundException: clojure.lang.RT
```xml
<dependency>
    <groupId>org.clojure</groupId>
    <artifactId>clojure</artifactId>
    <version>1.10.1</version>
</dependency>
```

### NoSuchMethodError:org.apache.hadoop.hbase.security.token.TokenUtil.addTokenForJob
```xml
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-hbase-handler</artifactId>
    <version>2.0.0</version>
    <!--            <version>1.2.1</version>-->
</dependency>
```

### idea跑spark,最后无法删除临时目录
```sh
# log4j 配置
log4j.logger.org.apache.spark.util.ShutdownHookManager=OFF
log4j.logger.org.apache.spark.SparkEnv=ERROR
```