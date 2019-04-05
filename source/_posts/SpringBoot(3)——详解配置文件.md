---
title: SpringBoot(3)——详解配置文件
date: 2018-10-22 17:28:32
tags: 
  - SpringBoot
categories:
  - SpringBoot
---

SpringBoot使用“习惯优于配置”的理念让你的项目快速构建起来。SpringBoot在pom.xml中引入模块化的Starter POMs，其中各个模块都有自己的默认配置。根据项目的需要，我们可能需要修改默认配置。我们可以在很多地方进行配置来覆盖掉默认的配置，覆盖默认的配置有很多方式，每种方式的优先级也不同。
<!--more-->
如下，优先级由高到低：
1. 在命令行输入的参数
2. SPRING_APPLICATION_JSON中的属性。SPRING_APPLICATION_JSON是以json格式配置在系统环境变量中的内容
3. java: comp/env 里的JNDI属性
4. Java的系统属性，可以通过System.getProperties()获得的内容
5. 操作系统环境变量
6. RandomValuePropertySource属性类生成的random. * 属性
7. 位于当前应用jar包之外，针对不同的profile换将的配置文件内容，例如application-{profile}.properties或者yaml定义的配置文件
8. 位于当前应用jar包之内，针对不同的profile换将的配置文件内容，例如application-{profile}.properties或者yaml定义的配置文件
9. 位于当前应用jar包之外的application.properties和yaml配置内容
10. 位于当前应用jar包之内的application.properties和yaml配置内容
11. 在应用@Configuration配置类中，用@PropertySource注解声明的属性文件
12. 应用默认属性，SpringApplication.setDefaultProperties声明的默认属性

SpringBoot使用了一个全局配置文件application.properties，放在src/main/resources目录下。这个文件的作用是对一些默认配置项进行修改。
### 自定义属性
application.properties提供自定义属性的支持，我们可以在这儿配置一些常量：
```yaml
author.name=Jack
author.email=jack@gmail.com
contact.information=请通过${author.email}联系${author.name}
#在配置文件中，可以通过${key}来相互引用，key可以在当前属性文件中也可以在其他属性文件中
#注意：application.properties 配置中文值的时候，读取出来的属性值会出现乱码问题。
#但是application.yml不会出现乱码问题。原因是，Spring Boot 是以iso-8859的编码方式读取 application.properties配置文件。
```
在要使用的地方通过注解@Value(value="${config.name}")就可以绑定到你想要的属性上面。

在一些情况下，我们希望能生成随机的参数值，比如密钥/服务端口等。SpringBoot的属性配置文件中可以通过${random}来产生int值、long值或者string字符串，来支持属性的随机值。举例：
```yaml
#随机字符串
com.xd.blog.value=${random.value}
#随机int
com.xd.blog.number=${random.int}
#随机long
com.xd.blog.bignumber=${random.long}
#10以内的随机数
com.xd.blog.test1=${random.int(10)}
#10-20以内的随机数
com.xd.blog.test2=${random.int[10,20]}
```
通过@ConfigurationProperties(prefix="com.xd.blog")注解，将配置文件中以com.xd.blog为前缀的属性值自动绑定到对应的字段中。同时用@Component作为Bean注入到Spring容器中。

### 多环境配置
在实际的项目中，我们需要对不同的环境分别配置，比如开发环境/测试环境/生产环境，就需要分别配置以下三个文件：
    application-dev.properties: 开发环境
    application-test.properties: 测试环境
    application-prod.properties: 生产环境
然后通过在application.properties文件中，设置spring.profiles.active属性，比如：
```yaml
#激活dev配置文件
spring.profiles.active=dev
```
需要注意的是，properties格式的文件在配置中文值时，读取出来的属性值会出现乱码问题，因为SpringBoot是以ISO-8859的编码格式读取properties配置文件。但是以yml格式的文件不会出现乱码问题。实际开发中，我们会用yml文件的格式编写配置文件，详情见[YAML详解](http://x-d.xyz/SpringBoot/46.html "YAML详解")。
注意这里，还有一个坑：
如果定义一个键值对 user.name=xxx ,这里会读取不到对应写的属性值。为什么呢？Spring Boot 的默认StandardEnvironment首先将会加载 “systemEnvironment" 作为首个PropertySource. 而 source 即为System.getProperties().当 getProperty时,按照读取顺序,返回 “systemEnvironment" 的值.即 System.getProperty("user.name")。即系统的登录账号。

下面介绍几种常见的配置方式
### 在命令行输入参数
```shell
# 例如，通过使用server.port属性来设置应用的端口为8888
# 启动调试模式debug，打印日志更为详细
java -jar xxx.jar --server.port=8888 --debug
```
### 应用程序属性文件application.properties
常见的配置项如下：
```yaml
#配置Tomcat启动的端口号
server.port=80
#开启/关闭调试模式
debug=true
#应用上下文
#默认是/，若有多个项目，/地址将会产生冲突
#一般情况下，小项目通常都是在Tomcat下部署多个webapp，通过这个配置项进行区分
#在集群或者大中型项目中，通常一个Tomcat对应一个webapp，然后通过不同的端口进行区分（如8080/8091/8082）
server.servlet.context-path=/myspringboot
#默认字符集编码，UTF-8只包含了20000+个中文字符，无法显示生僻字
spring.http.encoding.charset=UTF-8
#开启/关闭thymeleaf页面缓存
#关闭thymeleaf页面缓存，同时在IDEA Setting->Build,...->Compiler->勾选Build Project Automatically
#即可进行thymeleaf的热部署。即打开Tomcat服务器，后续改动页面后，每次请求都是实时显示改动后的页面，而无须重启Tomcat
spring.thymeleaf.cache=false
#设置日期输出格式
spring.mvc.date-format=yyyy-MM-dd
```

想要了解完整的配置项，可以看SpringBoot的官方文档介绍：
https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html