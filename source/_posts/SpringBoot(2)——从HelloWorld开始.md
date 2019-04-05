---
title: SpringBoot(2)——从HelloWorld开始
date: 2018-10-18 21:40:26
tags: 
  - SpringBoot
categories:
  - SpringBoot
---

在开始编写项目代码前，我们首先来看一下用Maven来构建SpringBoot时，自动生成的默认配置文件pom.xml。pom.xml主要描述了项目的maven坐标，依赖关系，开发者需要遵循的规则，缺陷管理系统，组织和licenses，以及其他所有的项目相关因素，是项目级别的配置文件。
<!--more-->

###  1. pom.xml文件详解
一个典型的pom.xml文件如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
	<!-- 模型版本。maven2.0必须是这样写，现在是maven2唯一支持的版本 -->
	<modelVersion>4.0.0</modelVersion>
 
	<!-- 公司或者组织的唯一标志，并且配置时生成的路径也是由此生成， 如com.winner.trade，maven会将该项目打成的jar包放本地路径：/com/winner/trade -->
	<groupId>com.winner.trade</groupId>
 
	<!-- 本项目的唯一ID，一个groupId下面可能多个项目，就是靠artifactId来区分的 -->
	<artifactId>trade-core</artifactId>
 
	<!-- 本项目目前所处的版本号 -->
	<version>1.0.0-SNAPSHOT</version>
 
	<!-- 打包的机制，如pom,jar, maven-plugin, ejb, war, ear, rar, par，默认为jar -->
	<packaging>jar</packaging>
 
	<!-- 帮助定义构件输出的一些附属构件,附属构件与主构件对应，有时候需要加上classifier才能唯一的确定该构件 不能直接定义项目的classifer,因为附属构件不是项目直接默认生成的，而是由附加的插件帮助生成的 -->
	<classifier>...</classifier>
 
	<!-- 定义本项目的依赖关系 -->
	<dependencies>
 
		<!-- 每个dependency都对应这一个jar包 -->
		<dependency>
 
			<!--一般情况下，maven是通过groupId、artifactId、version这三个元素值（俗称坐标）来检索该构件， 然后引入你的工程。如果别人想引用你现在开发的这个项目（前提是已开发完毕并发布到了远程仓库），--> 
			<!--就需要在他的pom文件中新建一个dependency节点，将本项目的groupId、artifactId、version写入， maven就会把你上传的jar包下载到他的本地 -->
			<groupId>com.winner.trade</groupId>
			<artifactId>trade-test</artifactId>
			<version>1.0.0-SNAPSHOT</version>
 
			<!-- maven认为，程序对外部的依赖会随着程序的所处阶段和应用场景而变化，所以maven中的依赖关系有作用域(scope)的限制。 -->
			<!--scope包含如下的取值：compile（编译范围）、provided（已提供范围）、runtime（运行时范围）、test（测试范围）、system（系统范围） -->
			<scope>test</scope>
 
			<!-- 设置指依赖是否可选，默认为false,即子项目默认都继承:为true,则子项目必需显示的引入，与dependencyManagement里定义的依赖类似  -->
			<optional>false</optional>
 
			<!-- 屏蔽依赖关系。 比如项目中使用的libA依赖某个库的1.0版，libB依赖某个库的2.0版，现在想统一使用2.0版，就应该屏蔽掉对1.0版的依赖 -->
			<exclusions>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-api</artifactId>
				</exclusion>
			</exclusions>
 
		</dependency>
 
	</dependencies>
 
	<!-- 为pom定义一些常量，在pom中的其它地方可以直接引用 使用方式 如下 ：${file.encoding} -->
	<properties>
		<file.encoding>UTF-8</file.encoding>
		<java.source.version>1.5</java.source.version>
		<java.target.version>1.5</java.target.version>
	</properties>
 
	...
</project>
```
上述的配置项对于任何项目来说，应该都是必不可少的，它定义了项目的基本属性。对于pom.xml文件更详细的解释可以参考这篇文章[pom.xml配置文件详解](https://blog.csdn.net/u012152619/article/details/51485297# "pom.xml配置文件详解")。

默认的pom.xml中指定了parent为spring-boot-starter-parent. Parent中已经定义了spring的基本属性。指定了spring的版本号，在dependencies中无需再指定版本号。Parent中还包含其他大量默认配置，大大简化了我们的开发。然后默认添加了spring-boot-starter-test(测试模块，包括JUnit、Hamcrest、Mockito)依赖。
我们想要开发一个web应用的话，需要添加spring-boot-starter-web来集成SpringMVC功能。在pom.xml中的dependencies配置项中添加下面这段：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
### 2. 编写Hello World
完成以上步骤后，我们姐可以编写一个简单的HelloWorld了。
1.首先，编写一个Controller类。
```java
package com.xd.springboot;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MyController {
    @RequestMapping("/helloworld")
    public String index(){
        return "Hello World";
    }
}
```
2.这样，我们就写好了一个HelloWorld。我们直接运行项目的启动类SpringbootApplication，以后我们运行我们的项目都可以直接通过运行这个类启动。
3.在浏览器中访问http://localhost:8080/helloworld 。就可以看到页面显示出了Hello World。

我们还可以通过修改默认生成的那个SpringbootApplicationTest测试类，来测试我们写的这个HelloWorld。代码如下：
```java
package com.xd.springboot;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootApplicationTests {

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception{
        mvc = MockMvcBuilders.standaloneSetup(new MyController()).build();
    }

    @Test
    public void testHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/helloworld").accept(MediaType.APPLICATION_JSON)).andExpect(status().isOk()).andExpect(content().string(equalTo("Hello World")));
    }

}

```