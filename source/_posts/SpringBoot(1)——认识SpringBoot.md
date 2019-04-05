---
title: SpringBoot(1)——认识SpringBoot
date: 2018-09-13 20:30:18
tags: 
  - SpringBoot
categories:
  - SpringBoot
---

Spring Boot不是一个新的框架，它只是用来简化Spring应用开发的方式。它提供了默认的配置，自动化配置以方便快速启动新的Spring项目。它的主要动机在于简化配置和快速部署Spring应用。

<!--more-->

在使用SpringBoot之前，开发一个SpringMVC项目，需要完成以下操作：
1. 引入Spring依赖，一般都会引入大量的Spring依赖，如：spring-core/spring-context/spring-context-support/spring-webmvc等；
2. 配置web.xml，要配置contextConfigLocation参数，要配置Spring的监听器ContextLoaderListener，要配置SpringMVC的分发ServletDispatcherServlet
3. 配置spring-service.xml，配置SpringMVC对应的内部资源视图识别器InternalResourceViewResolver的前缀、后缀等；
4. 配置applicationContext.xml，配置Spring的核心配置文件
5. 配置好数据库的连接和Spring事务；
6. 配置文件的读取并开启注解；
7. 配置日志组件；
8. 编写代码，然后部署到Tomcat上进行调试；

可见，使用SpringMVC开发项目，需要在pom.xml中添加大量的依赖，还需要配置很多的配置文件，但其实这些配置在每个项目中都是大同小异，还需要配置外部Tomcat，并在运行时启动Tomcat。
Spring缺点： 配置繁琐，各种XML、Annotation配置，如果出错了也很难找出原因。

### 快速搭建SpringBoot的Demo
#### 1.使用Spring[官方网站](http://spring.io/projects/spring-boot)的[SPRING INITIALIZR](https://start.spring.io/)的方式
如图：
![](https://ws1.sinaimg.cn/large/006aBttAly1g1jsrhivfkj311v0hl76e.jpg)

选择项目构建模式(Maven/Gradle)，开发语言Java和SpringBoot的版本。填好Group和Artifact，并选择好需要用到的依赖模块。最后点击Generate Project即可生成并下载一个demo project。通过Eclipse或者IDEA等IDE导入这个demo，即可运行。
#### 2.直接使用IDEA搭建SpringBoot的Demo
打开IDEA，Create New Project，选择Spring Initializr，如图
![](https://ws1.sinaimg.cn/large/006aBttAly1g1jssg29luj30mt0giwf0.jpg)

![](https://ws1.sinaimg.cn/large/006aBttAly1g1jst0yq3kj311v0hl76e.jpg)

选择好JDK版本，勾选Default，直接下一步，如图
![](https://ws1.sinaimg.cn/large/006aBttAly1g1jt08saohj30mu0gf3yp.jpg)

填好Group（应用所属的公司或组织）和Artifact（应用名称），选择Type构建类型为Maven Project，其余的可以保持默认，直接下一步，如图
![](https://ws1.sinaimg.cn/large/006aBttAly1g1jt0mvbd4j30mt0ggq3d.jpg)

在这一步，右上角选择SpringBoot的版本，勾选需要的依赖。你可以根据项目的需求，勾选需要用到的依赖，也可以后续在配置文件中自行添加相应的依赖。直接下一步
![](https://ws1.sinaimg.cn/large/006aBttAly1g1jt12m3pij30mp0gedfz.jpg)

填写好项目名称和项目存储路径，点击Finish，即完成了一个SpringBoot的创建。

SpringBoot项目的目录结构:
- /src/main                                项目根目录
- /java                                    Java源代码目录
- /resources                               资源目录
- /resources/static                        静态资源目录
- /resources/templates                     表示层页面记录
- /resources/application.properties        SpringBoot配置文件
- /test                                    测试文件目录
  按照上述目录结构，在刚创建的项目中添加相应的目录。

基本的Spring Boot项目需要引用一下这些依赖项，包括：
- spring-boot-starter-parent  所有Spring Boot组件的基础引用
- spring-boot-starter-web  对Spring Boot应用提供Web支持
- spring-boot-starter-thymeleaf(可选)  提供thymeleaf模板引擎支持
- spring-boot-maven-plugin  提供Spring Boot应用构建打包的功能
  配置Spring Boot的依赖项和插件都是在项目的pom.xml文件中，添加好上述的四个依赖项。

**那么，传统的基于SpringMVC的web项目和基于SpringBoot的web项目有那些区别呢？**
1.项目打包方式不同

- 前者使用的maven-archetype-webapp架构，使用的是war包
- 后者使用的是jar包

2.pom.xml中引入的依赖不同
- 前者是每次引入单独的依赖
- 后者引入的依赖经常是多个依赖集成的一个依赖，这些依赖之间已经确保兼容，而且大部分依赖不需要指定version，因为版本号已经在spring-boot-starter-parent中定义过了。

3.项目结构不同
- 前者项目中的src/main/java在项目刚创建完成时是空的，而后者项目中有一个启动类（项目名+Application），而且在src/test/java中也有一个测试类（项目名+ApplicationTest）。
- 前者有web.xml文件，后者没有
- 前者项目的resources的目录是空的，后者resource中有static、templates目录和一个配置文件application.properties

4.项目运行方式不同

- 前者是启动Tomcat运行
- 后者是直接运行main方法或者直接运行jar(java -jar 项目名.jar)

**SpringBoot的优点：**

- 集成框架很简单，比如集成SpringMVC，只需集成spring-boot-starter-web这一个依赖，也不需要做任何配置，集成快速方便。SpringBoot支持很多常用框架，如log、test、mybatis、nosql、mq、模板技术(thymeleaf、freemark)、jpa、aop、actuator等；
- 引入的依赖很少，因为多个依赖被集成为一个依赖，比如要引入测试依赖JUnit、Hamcrest、Mockito，只需要引入spring-boot-starter-test这一个依赖；
- 自动化配置，使用默认配置，无需配置applicationContext.xml等配置文件
- 运行简单，直接使用java -jar命令，或者直接在IDE中运行main方法

**SpringBoot的缺点：**
- 高度封装，出现问题不易排查，适合有经验的开发人员，不适合初学者，上手容易，但是出问题很难排查；
- 蒋传统的Spring项目转成SpringBoot项目困难且耗时。仅适用于新的Spring项目；
- SpringBoot面世不久，文档不如传统Spring全面，快速发着，可能版本变化较大
