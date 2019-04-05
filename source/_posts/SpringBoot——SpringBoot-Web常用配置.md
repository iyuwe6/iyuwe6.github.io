---
title: SpringBoot——SpringBoot Web常用配置
date: 2018-09-27 22:04:20
tags: 
  - SpringBoot
categories: 
  - SpringBoot
---

SpringBoot使用“习惯优于配置”的理念让你的SpringBoot项目能快速搭建并运行起来。但是SpringBoot也允许你进行自定义的配置，所以我们就需要详细了解如何进行配置。

<!--more-->

首先是创建项目后默认生成的application.properties配置文件，这是一个全局的配置文件，放在src/main/resources目录下或者类路径的/config下。Sping Boot的全局配置文件的作用是对一些默认配置的配置值进行修改。注:如果你工程没有这个application.properties，那就在src/main/java/resources目录下新建一个。

下面是一些常用的配置项：
```
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