### 一些小技巧
```sh
# idea debug 日志 VM options
-Dlog4j.debug
```

### pyspark3.1异常: Python worker failed to connect back
```txt
win10增加系统环境变量：
key: PYSPARK_PYTHON
value: python
```

### idea 运行问题:Command line is too long.Shorten command line
```xml
<!-- 修改项目下 .idea\workspace.xml找到标签 <component name="PropertiesComponent"> -->
<property name="dynamic.classpath" value="true" />
```

### idea 初次分享代码仓到 github 报错: github代码仓为空,本地无法分析
```txt
之前提交代码保存过账号信息(用户名和密码)所以导致这次提交提示403错误
打开本地Windows的cmd命令窗口输入 rundll32.exe keymgr.dll,KRShowKeyMgr 将之前存储的git提交时保留的用户名密码删除
```

### idea 初次提交代码到 github 报错: Can't finish GitHub sharing process,Successfully created project '--' on GitHub,but initial push failed:git@github.com: Permission denied (publickey).
```txt
看本地的.git/config设置的仓库url地址和github使用的链接地址是否一致,如use https,则url需要用https的仓库地址
统一改为github的http模式,例如
https://github.com/ByDylan-YH/srping-boot_hadoop.git
```

### idea 关闭一直显示 close project...

```txt
主页 Help -> Find Action -> 输入 Registry -> 禁用 ide.await.scope.completion
```
### maven 添加jar包到本地仓库
```sh
# groupId,artifactId,version 照着这格式随便填
mvn install:install-file -Dfile=D:/WorkSpace/ideaProject/data_compare/lib/hadoop-common.jar -DgroupId=org.apache.hadoop -DartifactId=hadoop-common -Dversion=1.0.0 -Dpackaging=jar
```

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
--illegal-access=deny --add-opens java.base/java.lang=ALL-UNNAMED
```

### log4j升级为log4j2,中途还是会打印找不到log4j配置
```xml
<!-- 用maven helper插件查看 All Dependencies,把log4j,log4j-extras全部排除 -->
<!-- log4j2-->
<dependencys>
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
</dependencys>
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
<dependencys>
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
</dependencys>
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
<dependencys>
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
</dependencys>
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
<dependencys>
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
</dependencys>
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

### maven打包包含其他目录文件
```xml
        <resources>
<!--            <resource>-->
<!--                <directory>lib</directory>-->
<!--                <targetPath>lib/</targetPath>-->
<!--                <includes>-->
<!--                    <include>**/*.jar</include>-->
<!--                </includes>-->
<!--            </resource>-->
<!--这一行是默认的,如果写了别的就必须写上这一个,否则会漏掉它-->
            <resource>
                <directory>src/main/resources</directory>
                <!--                <targetPath>BOOT-INF/classes/</targetPath>-->
                <includes>
                    <include>**/*.*</include>
                </includes>
            </resource>
        </resources>
```

### maven插件1: 将依赖复制到一个目录下
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/lib</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### maven插件2: 打包
```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.3.0</version>
    <configuration>
        <!--                    这部分可有可无,加上的话则直接生成可运行jar包-->
        <archive>
            <manifest>
                <mainClass>main.SpringStartMain</mainClass>
            </manifest>
        </archive>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
</plugin>
```

### maven插件3: 打包,能防止报错 Configuration problem: Unable to locate Spring NamespaceHandler for XML schema namespace [http://www.springframework.org/schema/context]
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <!--                            <artifactSet>-->
                <!--                                <excludes>-->
                <!--                                    <exclude>junit:junit</exclude>-->
                <!--                                    <exclude>log4j:log4j:jar:</exclude>-->
                <!--                                </excludes>-->
                <!--                            </artifactSet>-->
                <transformers>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>META-INF/spring.handlers</resource>
                    </transformer>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                        <resource>META-INF/spring.schemas</resource>
                    </transformer>
                    <transformer
                            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>main.SpringStartMain</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### maven插件4: 
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <archive>
            <addMavenDescriptor>false</addMavenDescriptor>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
                <mainClass>main.SpringStartMain</mainClass>
            </manifest>
            <manifestEntries>
                <Class-Path>./</Class-Path>
            </manifestEntries>
        </archive>
        <!-- 过滤掉不希望包含在jar中的文件  -->
        <excludes>
            <exclude>*.xml</exclude>
            <exclude>spring/**</exclude>
            <exclude>config/**</exclude>
        </excludes>
        <!-- 这里不做举例了 -->
        <includes>
            <include>**</include>
        </includes>
    </configuration>
</plugin>
```