---
title: SpringBoot(4)——日志管理
date: 2018-10-24 11:58:16
tags: 
  - SpringBoot
categories: 
  - SpringBoot
---

SpringBoot日志管理的[官方文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-logging.html "官方文档")。根据这个文档，我们知道SpringBoot在所有内部日志中使用[Commons Logging](http://commons.apache.org/proper/commons-logging/ "Commons Logging")，但是默认配置也提供了对常用日志的支持，如java.util.logging、Log4j和Logback。
<!--more-->
每一种日志支持，默认都是使用控制台输出日志信息，日志文件输出可选。默认配置下，使用Logback管理日志，并用INFO级别输出到控制台。

#### 格式化日志
默认的日志输出格式：
```shell
# 输出格式是：时间(精确到毫秒) 日志级别(FATAL, ERROR, WARN, INFO, DEBUG, TRACE) 进程ID --- [线程名] Logger名(一般使用源代码的类名) : 日志内容
2018-03-21 19:50:57.892  INFO 22798 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
```
#### 控制台输出
在SpringBoot中默认配置了ERROR、WARN和INFO级别的日志输出到控制台。
我们可以通过两种方式切换至DEBUG级别：
- 在控制台执行命令后加入--debug级别，如：$ java -jar myapp.demo --debug
- 在application.properties中配置debug=true，该属性置为true的时候，核心Logger(包含嵌入式容器、hibernate、spring)会输出更多内容，但是你自己应用的日志并不会输出为DEBUG级别。

#### 多彩输出
如果你的终端支持ANSI，设置多彩输出会让日志更具可读性。通过在application.properties中设置spring.output.ansi.enabled参数来支持。
- NEVER：禁用ANSI-colored输出(默认项)
- DETECT：会检查终端是否支持ANSI，是的话就采用多彩输出(推荐项)。
- ALWAYS：总是使用ANSI-colored格式输出，若终端不支持的时候，会有很多干扰信息，不推荐使用。

#### 文件输出
SpringBoot默认配置只会输出到控制台，并不会记录到文件中，但是我们在实际生产环境中都是要用文件记录日志。
在application.properties中配置logging.file或者logging.path属性。注意：二者不能同时使用，如果同时使用，则只有logging.file生效。
- logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：logging.file=my.log
- logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容，如：logging.path=/var/log
  默认情况下，日志文件会在10MB大小的时候被截断，产生新的日志文件。如果要修改大小，可以通过logging.file.max-size来配置，单位为MB。

#### 级别控制
在application.properties中进行日志的级别控制。
配置格式：
`logging.level.*=LEVEL`
- logging.level: 日志级别控制前缀， * 为包名或Logger名
- LEVEL: 选项TRACE，DEBUG，INFO，WARN，ERROR，FATAL，OFF
  举例：
- logging.level.com.didispace=DEBUG    com.didispace包下所有class以DEBUG级别输出
- logging.level.root=WARN         root日志以WARN级别输出

#### 自定义日志配置
由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。
根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：
- Logback：logback-spring.xml, logback-spring.groovy, logback.xml, logback.groovy
- Log4j：log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml
- Log4j2：log4j2-spring.xml, log4j2.xml
- JDK (Java Util Logging)：logging.properties
  Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml）

#### 自定义输出格式
在Spring Boot中可以通过在application.properties配置如下参数控制输出格式：
- logging.pattern.console：定义输出到控制台的样式（不支持JDK Logger）
- logging.pattern.file：定义输出到文件的样式（不支持JDK Logger）

下面详细介绍SpringBoot支持的Logback日志组件和log4j日志组件。
### Logback的配置和使用
#### 1.介绍
logback是由log4j创始人设计的又一个开源日志组件。logback当前分成三个模块：logback-core,logback-classic和logback-access。logback-core是其它两个模块的基础模块。logback-classic是log4j的一个改良版本。此外logback-classic完整实现SLF4J API使你可以很方便地更换成其它日志系统如log4j或JDK14 Logging。logback-access访问模块与Servlet容器集成提供通过Http来访问日志的功能(用的少)。
#### 2.添加依赖
SpringBoot项目中在官方文档中说明，默认已经依赖了一些日志框架。而其中推荐使用的就是Logback，SpringBoot已经依赖了Logback所以不需要手动添加依赖。
#### 3.详解logback.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!-- 级别从高到低 OFF 、 FATAL 、 ERROR 、 WARN 、 INFO 、 DEBUG 、 TRACE 、 ALL -->
<!-- 日志输出规则 根据当前ROOT 级别，日志输出时，级别高于root默认的级别时 会输出 -->
<!-- 以下 每个配置的 filter 是过滤掉输出文件里面，会出现高级别文件，依然出现低级别的日志信息，通过filter 过滤只记录本级别的日志 -->
<!-- scan 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。 -->
<!-- scanPeriod 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
<!-- debug 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <!-- 动态日志级别 -->
    <jmxConfigurator/>

    <!-- 定义日志文件 输出位置 -->
    <property name="log.home_dir" value="F:\logs"/>

    <!-- 日志最大的历史 30天 -->
    <property name="log.maxHistory" value="30"/>

    <property name="log.level" value="debug"/>


    <!-- ConsoleAppender 控制台输出日志 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>
                <!-- 设置日志输出格式 -->
                %d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %logger - %msg%n
            </pattern>
        </encoder>
    </appender>

    <!-- ERROR级别日志 -->
    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 RollingFileAppender -->
    <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤器，只记录WARN级别的日志 -->
        <!-- 果日志级别等于配置级别，过滤器会根据onMath 和 onMismatch接收或拒绝日志。 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 设置过滤级别 -->
            <level>ERROR</level>
            <!-- 用于配置符合过滤条件的操作 -->
            <onMatch>ACCEPT</onMatch>
            <!-- 用于配置不符合过滤条件的操作 -->
            <onMismatch>DENY</onMismatch>
        </filter>
        <!-- 最常用的滚动策略，它根据时间来制定滚动策略.既负责滚动也负责触发滚动 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志输出位置 可以是相对和绝对路径 -->
            <fileNamePattern>
                ${log.home_dir}/error/%d{yyyy-MM-dd}/my-demo-%i.log
            </fileNamePattern>
            <!-- 可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件假设设置每个月滚动，且<maxHistory>是6， 则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除 -->
            <maxHistory>${log.maxHistory}</maxHistory>
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            <!-- 单个log文件超过该大小就会重新建一个log文件 -->
                <MaxFileSize>2MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>
                <!-- 设置日志输出格式 -->
                %d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %logger - %msg%n
            </pattern>
        </encoder>
    </appender>


    <!-- WARN级别日志 appender -->
    <appender name="WARN" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 过滤器，只记录WARN级别的日志 -->
        <!-- 果日志级别等于配置级别，过滤器会根据onMath 和 onMismatch接收或拒绝日志。 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- 设置过滤级别 -->
            <level>WARN</level>
            <!-- 用于配置符合过滤条件的操作 -->
            <onMatch>ACCEPT</onMatch>
            <!-- 用于配置不符合过滤条件的操作 -->
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志输出位置 可相对、和绝对路径 -->
            <fileNamePattern>${log.home_dir}/warn/%d{yyyy-MM-dd}/my-demo-%i.log</fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>2MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %logger - %msg%n</pattern>
        </encoder>
    </appender>


    <!-- INFO级别日志 appender -->
    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.home_dir}/info/%d{yyyy-MM-dd}/my-demo-%i.log</fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>2MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %logger - %msg%n</pattern>
        </encoder>
    </appender>


    <!-- DEBUG级别日志 appender -->
    <appender name="DEBUG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.home_dir}/debug/%d{yyyy-MM-dd}/my-demo-%i.log</fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>2MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %logger - %msg%n</pattern>
        </encoder>
    </appender>


    <!-- TRACE级别日志 appender -->
    <appender name="TRACE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>TRACE</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.home_dir}/trace/%d{yyyy-MM-dd}/my-demo-%i.log</fileNamePattern>
            <maxHistory>${log.maxHistory}</maxHistory>
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <MaxFileSize>2MB</MaxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %logger - %msg%n</pattern>
        </encoder>
    </appender>


    <!-- root级别   DEBUG -->
    <root>
        <!-- 打印debug级别日志及以上级别日志 -->
        <level value="${log.level}"/>
        <!-- 控制台输出 -->
        <appender-ref ref="console"/>
        <!-- 文件输出 -->
        <appender-ref ref="ERROR"/>
        <appender-ref ref="INFO"/>
        <appender-ref ref="WARN"/>
        <appender-ref ref="DEBUG"/>
        <appender-ref ref="TRACE"/>
    </root>
</configuration>

```

### log4j的配置和使用
#### 1.添加依赖
在创建SpringBoot工程时，引入了spring-boot-starter，其中包含了spring-boot-starter-logging，该依赖默认使用了logback，所以我们在引入log4j之前，需要排除logback的依赖，再引入log4j的依赖。
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion> 
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j</artifactId>
</dependency>
```
#### 2.log4j配置
在引入了log4j依赖后，只需要在src/main/resources目录下加入log4j.properties配置文件，就可以开始对应用的日志进行配置使用。
#### 控制台输出
```
# LOG4J配置
log4j.rootCategory=INFO, stdout

# 控制台输出
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %5p %c{1}:%L - %m%n
```
#### 文件输出
在实际情况下，我们不能只将日志输出到控制台，还要持久化日志内容，方便后续出了问题时可以通过日志查找原因。添加如下appender内容，按天输出到不同的文件中，同时还需要为log4j.rootCategory添加名为file的appender，这样root日志就可以输出到logs/all.log文件中了。
```
#
log4j.rootCategory=INFO, stdout, file

# root日志输出
log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.file=logs/all.log
log4j.appender.file.DatePattern='.'yyyy-MM-dd
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %5p %c{1}:%L - %m%n
```
#### 分类输出
对不同的文件日志输出级别往往是不同的，我们可以对不同的包的日志输出级别进行单独设置。
- 可以按不同的package进行输出。通过定义输出到logs/my.log的appender，并对com.didispace包下的日志级别设定为DEBUG级别、appender设置为输出到logs/my.log的名为didifile的appender。

```
# com.didispace包下的日志配置
log4j.category.com.didispace=DEBUG, didifile

# com.didispace下的日志输出
log4j.appender.didifile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.didifile.file=logs/my.log
log4j.appender.didifile.DatePattern='.'yyyy-MM-dd
log4j.appender.didifile.layout=org.apache.log4j.PatternLayout
log4j.appender.didifile.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %5p %c{1}:%L ---- %m%n
```
- 可以对不同级别进行分类，比如对ERROR级别输出到特定的日志文件中，具体配置可以如下。

```
log4j.logger.error=errorfile
# error日志输出
log4j.appender.errorfile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.errorfile.file=logs/error.log
log4j.appender.errorfile.DatePattern='.'yyyy-MM-dd
log4j.appender.errorfile.Threshold = ERROR
log4j.appender.errorfile.layout=org.apache.log4j.PatternLayout
log4j.appender.errorfile.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %5p %c{1}:%L - %m%n
```