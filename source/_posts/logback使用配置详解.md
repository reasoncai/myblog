---
title: logback使用配置详解
date: 2017-04-25 16:42:49
tags: 日志
categories: java框架
---

## 1.介绍
　Logback是由log4j创始人设计的另一个开源日志组件,它当前分为下面下个模块：

<!-- more -->

- 　　logback-core：其它两个模块的基础模块
- 　　logback-classic：它是log4j的一个改良版本，同时它完整实现了slf4j API使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging
- 　　logback-access：访问模块与Servlet容器集成提供通过Http来访问日志的功能（非必须的包）
- 　　注意：还要一个slf4j的包

## 2.配置介绍
### 2.1Logger、appender及layout
Logger作为日志的记录器，把它关联到应用的对应的context上后，主要用于存放日志对象，也可以定义日志类型、级别。
Appender主要用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、PostreSQL、 Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。 
Layout 负责把事件转换成字符串，格式化的日志信息的输出。
### 2.2logger context
各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory 类的静态方法getLogger取得。 getLogger方法以 logger名称为参数。用同一名字调用LoggerFactory.getLogger 方法所得到的永远都是同一个logger对象的引用。
### 2.3有效级别及级别的继承
Logger 可以被分配级别。级别包括：TRACE、DEBUG、INFO、WARN 和 ERROR，定义于ch.qos.logback.classic.Level类。如果 logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger 默认级别是 DEBUG。
### 2.4打印方法与基本的选择规则
打印方法决定记录请求的级别。例如，如果 L 是一个 logger 实例，那么，语句 L.info("..")是一条级别为 INFO的记录语句。记录请求的级别在高于或等于其 logger 的有效级别时被称为被启用，否则，称为被禁用。记录请求级别为 p，其 logger的有效级别为 q，只有则当 p>=q时，该请求才会被执行。
该规则是 logback 的核心。级别排序为： TRACE < DEBUG < INFO < WARN < ERROR

## 3.logback的默认配置
如果配置文件 logback-test.xml 和 logback.xml 都不存在，那么 logback 默认地会调用BasicConfigurator ，创建一个最小化配置。最小化配置由一个关联到根 logger 的ConsoleAppender 组成。输出用模式为%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n 的 PatternLayoutEncoder 进行格式化。root logger 默认级别是 DEBUG。
### 3.1Logback的配置文件
Logback 配置文件的语法非常灵活。正因为灵活，所以无法用 DTD 或 XML schema 进行定义。尽管如此，可以这样描述配置文件的基本结构：以<configuration>开头，后面有零个或多个<appender>元素，有零个或多个<logger>元素，有最多一个<root>元素。
### 3.2Logback默认配置的步骤
　　1. JVM参数指定 -Dlogback.configurationFile=D:\PSS\PssCenter\config\logback.xml
　　2. 尝试在 classpath下查找文件logback-test.xml；
　　3. 如果文件不存在，则查找文件logback.xml；
　　4. 如果两个文件都不存在，logback用BasicConfigurator自动对自己进行配置，这会导致记录输出到控制台。

## 4.logback.xml常用配置详解
![](http://ooxz0ztfx.bkt.clouddn.com/logback.png)
### 4.1根节点configuration，包含下面三个属性：
　　　　scan: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
　　　　scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
　　　　debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
　　例如：
```xml　　
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
<!--其他配置省略--> 
</configuration>
```
### 4.2子节点contextName：用来设置上下文名称，每个logger都关联到logger上下文，默认上下文名称为default。但可以使用<contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。
　　例如：
```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
	<contextName>myAppName</contextName> 
	<!--其他配置省略-->
</configuration>
```
### 4.3子节点property：用来定义变量值，它有两个属性name和value，通过property定义的值会被插入到logger上下文中，可以使“${}”来使用变量。
　　　　name: 变量的名称
　　　　value: 的值时变量定义的值
　　例如：
```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
	<property name="APP_Name" value="myAppName" /> 
	<contextName>${APP_Name}</contextName> 
	<!--其他配置省略--> 
</configuration>
```
### 4.4子节点timestamp：获取时间戳字符串，他有两个属性key和datePattern
　　　　key: 标识此timestamp的名字；
　　　　datePattern: 设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式。
　　例如：
```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 
	<timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/> 
	<contextName>${bySecond}</contextName> 
	<!-- 其他配置省略--> 
</configuration>
```
### 4.5子节点appender：负责写日志的组件，它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名
#### 4.5.1ConsoleAppender 把日志输出到控制台，有以下子节点：
　　　　　　encoder：对日志进行格式化。（具体参数稍后讲解 ）
　　　　　　target：字符串System.out(默认)或者System.err（区别不多说了）
　　　　例如：
```xml
<configuration> 
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 		<encoder> 
　			<pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern> 
		</encoder> 
	</appender> 

　　<root level="DEBUG"> 
		<appender-ref ref="STDOUT" /> 
　　</root> 
</configuration>
```
　　　　上述配置表示把>=DEBUG级别的日志都输出到控制台
#### 4.5.2FileAppender：把日志添加到文件，有以下子节点：
　　　　　　file：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
　　　　　　append：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
　　　　　　encoder：对记录事件进行格式化。（具体参数稍后讲解 ）
　　　　　　prudent：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。
　　　　例如：
```xml
　　　　<configuration> 
　　　　　　<appender name="FILE" class="ch.qos.logback.core.FileAppender"> 
　　　　　　　　<file>testFile.log</file> 
　　　　　　　　<append>true</append> 
　　　　　　　　<encoder> 
　　　　　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
　　　　　　　　</encoder> 
　　　　　　</appender> 

　　　　　　<root level="DEBUG"> 
　　　　　　<appender-ref ref="FILE" /> 
　　　　　　</root> 
　　　　</configuration>
```
　　　　上述配置表示把>=DEBUG级别的日志都输出到testFile.log
　　　　
#### 4.5.3RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：
　　　　　　file：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
　　　　　　append：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。	
　　　　　　rollingPolicy:当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类
　　　　　　class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。有以下子节点：
　　　　　fileNamePattern：必要节点，包含文件名及“%d”转换符，“%d”可以包含一个java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。
如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；
如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。	
　　　　　　　　maxHistory:
可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且maxHistory是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。

　　　　　　class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy"： 查看当前活动文件的大小，如果超过指定大小会告知RollingFileAppender 触发当前活动文件滚动。只有一个节点:
　　　　　　　　maxFileSize:这是活动文件的大小，默认值是10MB。
　　　　　　　　prudent：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。	

　　　　　　triggeringPolicy : 告知 RollingFileAppender 合适激活滚动。
　　　　　　class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy" 根据固定窗口算法重命名文件的滚动策略。有以下子节点：
　　　　　　　　minIndex:窗口索引最小值
　　　　　　　　maxIndex:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
　　　　　　　　fileNamePattern:必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip
　　　　　　例如：
```xml
<configuration> 
	<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
　　　　　　　　　　　　<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"> 
　　　　　　　　　　　　　　<fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern> 
　　　　　　　　　　　　　　<maxHistory>30</maxHistory> 
　　　　　　　　　　　　</rollingPolicy> 
　　　　　　　　　　　　<encoder> 
　　　　　　　　　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
　　　　　　　　　　　　</encoder> 
	</appender> 

　　　　　　　　　　<root level="DEBUG"> 
　　　　　　　　　　　　<appender-ref ref="FILE" /> 
　　　　　　　　　　</root> 
</configuration>
```
　　　　　　　　上述配置表示每天生成一个日志文件，保存30天的日志文件。
```xml
　　　　　　　　<configuration> 
　　　　　　　　　　<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender"> 
　　　　　　　　　　　　<file>test.log</file> 

　　　　　　　　　　　　<rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy"> 
　　　　　　　　　　　　　　<fileNamePattern>tests.%i.log.zip</fileNamePattern> 
　　　　　　　　　　　　　　<minIndex>1</minIndex> 
　　　　　　　　　　　　　　<maxIndex>3</maxIndex> 
　　　　　　　　　　　　</rollingPolicy> 

　　　　　　　　　　　　<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy"> 
　　　　　　　　　　　　　　<maxFileSize>5MB</maxFileSize> 
　　　　　　　　　　　　</triggeringPolicy> 
　　　　　　　　　　　　<encoder> 
　　　　　　　　　　　　　　<pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern> 
　　　　　　　　　　　　</encoder> 
　　　　　　　　　　</appender> 

　　　　　　　　　　<root level="DEBUG"> 
　　　　　　　　　　　　<appender-ref ref="FILE" /> 
　　　　　　　　　　</root> 
　　　　　　　　</configuration>
```
　　　　　　　　上述配置表示按照固定窗口模式生成日志文件，当文件大于20MB时，生成新的日志文件。窗口大小是1到3，当保存了3个归档文件后，将覆盖最早的日志。
　　　　　　encoder：对记录事件进行格式化。负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。
PatternLayoutEncoder 是唯一有用的且默认的encoder ，有一个pattern节点，用来设置日志的输入格式。使用“%”加“转换符”方式，如果要输出“%”，则必须用“\”对“\%”进行转义。
#### 4.5.4还有SocketAppender、SMTPAppender、DBAppender、SyslogAppender、SiftingAppender，并不常用，这里就不详解了。
大家可以参考官方文档http://logback.qos.ch/documentation.html，还可以编写自己的Appender。	
### 4.6、子节点loger：用来设置某一个包或具体的某一个类的日志打印级别、以及指定appender。loger仅有一个name属性，一个可选的level和一个可选的addtivity属性。
可以包含零个或多个appender-ref元素，标识这个appender将会添加到这个loger
　　　　name: 用来指定受此loger约束的某一个包或者具体的某一个类。
　　　　level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。 如果未设置此属性，那么当前loger将会继承上级的级别。
addtivity: 是否向上级loger传递打印信息。默认是true。同loger一样，可以包含零个或多个appender-ref元素，标识这个appender将会添加到这个loger。
### 4.7、子节点root:它也是loger元素，但是它是根loger,是所有loger的上级。只有一个level属性，因为name已经被命名为"root",且已经是最上级了。
　　　　level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，不能设置为INHERITED或者同义词NULL。 默认是DEBUG。

## 5.常用logger配置
```xml
<!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
<logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" />
<logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG" />
<logger name="org.hibernate.SQL" level="DEBUG" />
<logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
<logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />

<!--myibatis log configure-->
<logger name="com.apache.ibatis" level="TRACE"/>
<logger name="java.sql.Connection" level="DEBUG"/>
<logger name="java.sql.Statement" level="DEBUG"/>
<logger name="java.sql.PreparedStatement" level="DEBUG"/>
```

## 6.Demo
　1、添加依赖包logback使用需要和slf4j一起使用，所以总共需要添加依赖的包有slf4j-api
logback使用需要和slf4j一起使用，所以总共需要添加依赖的包有slf4j-api.jar，logback-core.jar，logback-classic.jar，logback-access.jar这个暂时用不到所以不添加依赖了，maven配置
```xml
　　<properties>
　　　　<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
　　　　<logback.version>1.1.7</logback.version>
　　　　<slf4j.version>1.7.21</slf4j.version>
　　</properties>

　　<dependencies>
　　　　<dependency>
　　　　　　<groupId>org.slf4j</groupId>
　　　　　　<artifactId>slf4j-api</artifactId>
　　　　　　<version>${slf4j.version}</version>
　　　　　　<scope>compile</scope>
　　　　</dependency>
　　　　<dependency>
　　　　　　<groupId>ch.qos.logback</groupId>
　　　　　　<artifactId>logback-core</artifactId>
　　　　　　<version>${logback.version}</version>
　　　　</dependency>
　　　　<dependency>
　　　　　　<groupId>ch.qos.logback</groupId>
　　　　　　<artifactId>logback-classic</artifactId>
　　　　　　<version>${logback.version}</version>
　　　　　　</dependency>
　　</dependencies>
```
　　2、logback.xml配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
<property name="LOG_HOME" value="/home" />
<!-- 控制台输出 -->
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
</encoder>
</appender>
<!-- 按照每天生成日志文件 -->
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
<!--日志文件输出的文件名-->
<FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>
<!--日志文件保留天数-->
<MaxHistory>30</MaxHistory>
</rollingPolicy>
<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
</encoder>
<!--日志文件最大的大小-->
<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
<MaxFileSize>10MB</MaxFileSize>
</triggeringPolicy>
</appender>

<!-- 日志输出级别 -->
<root level="INFO">
<appender-ref ref="STDOUT" />
</root>
</configuration>
```
　　3、java代码
```java
　　/**
　　* Hello world!
　　*/
　　public class App {

　　private final static Logger logger = LoggerFactory.getLogger(App.class);

　　　　public static void main(String[] args) {
　　　　　　logger.info("logback 成功了");
　　　　　　logger.error("logback 成功了");
　　　　　　logger.debug("logback 成功了");
　　　　}
　　}
```
我的logback.xml常用配置：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds">
<property name="LOG_HOME" value="../log2" />
	<!-- Simple file output -->
	<appender name="FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">
		<!-- encoder defaults to ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
		<encoder>
			<pattern>
				[ %-5level] [%date{yyyy-MM-dd HH:mm:ss.SSS}] %logger{96} [%line] [%thread]- %msg%n
			</pattern>

			<charset>UTF-8</charset> <!-- 此处设置字符集 -->
		</encoder>
		<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
			<!-- rollover daily 配置日志所生成的目录以及生成文件名的规则 -->
			<fileNamePattern>${LOG_HOME}/log_%d{yyyyMMdd}.%i.log</fileNamePattern>
			<timeBasedFileNamingAndTriggeringPolicy
				class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
				<!-- or whenever the file size reaches 64 MB -->
				<maxFileSize>64 MB</maxFileSize>
			</timeBasedFileNamingAndTriggeringPolicy>
			<maxHistory>30</maxHistory>
		</rollingPolicy>


		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>INFO</level>
		</filter>
		<!-- Safely log to the same file from multiple JVMs. Degrades performance! -->
		<prudent>false</prudent>
	</appender>


	<!-- Console output -->
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<!-- encoder defaults to ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
		<encoder>
			<pattern>
				[ %-5level] [%date{yyyy-MM-dd HH:mm:ss.SSS}] %logger{96} [%line] [%thread]- %msg%n
			</pattern>
			<charset>UTF-8</charset> <!-- 此处设置字符集 -->
		</encoder>
		<!-- Only log level WARN and above -->
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>INFO</level>
		</filter>
	</appender>
	
	<!-- 邮件监控异常 -->
<!--	<appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">
		<smtpHost>smtp.163.com</smtpHost>
		<username>cairs-2w010@163.com</username>
		<password>zxc1s2dsa46</password>
		<from>cairs-2010@163.com</from>
		<to>403411d876@qq.com</to>
		<subject>【web-ext-Error】: %logger</subject>
	 <layout class="ch.qos.logback.classic.html.HTMLLayout"/>	
		<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
			<level>ERROR</level>
		</filter>
	</appender>-->

	<!-- Enable FILE and STDOUT appenders for all log messages. By default, 
		only log at level INFO and above. -->
	<root level="INFO">
		<appender-ref ref="FILE" />
		<!--<appender-ref ref="STDOUT" />-->
		<!--<appender-ref ref="EMAIL" />-->
	</root>

	<!-- For loggers in the these namespaces, log at all levels. -->
	<!-- <logger name="pedestal" level="ALL" /> <logger name="hammock-cafe" 
		level="ALL" /> <logger name="user" level="ALL" /> -->
</configuration>
```



　　　　

