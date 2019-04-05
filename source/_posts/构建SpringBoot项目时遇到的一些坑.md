---
title: 构建SpringBoot项目时遇到的一些坑
date: 2018-11-01 17:28:36
tags: 
  - SpringBoot
categories: 
  - SpringBoot
---

平时在写SpringBoot的项目时，难免会遇到各种各样奇奇怪怪的问题，在此几下，以防后面再遇到同样的问题一时找不到解决办法。

<!--more -->

今天在写一个Spring Boot的小项目时，在配置好spring data jpa后，运行测试代码时，遇到一系列的问题。
### 1.mysql-connector-java不同版本对应的驱动问题
在mysql-connector-java5中是com.mysql.jdbc.Driver
    #JDBC连接mysql5配置
    driverClassName=com.mysql.jdbc.Driver
    url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
    username=root
    password=
在mysql-connector-java6中是com.mysql.cj.jdbc.Driver
    #JDBC连接mysql6配置，需要额外指定时区
    driverClassName=com.mysql.cj.jdbc.Driver
    url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
    username=root
    password=
如果你使用的mysql-connector-java用的6.0以上的，例如：
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>6.0.6</version>
    </dependency>
但是你在配置文件中依然使用的是com.mysql.jdbc.Driver，就会报以下错误信息：
    Loading class 'com.mysql.jdbc.Driver'. This is deprecated. The new 
    driver class is 'com.mysql.cj.jdbc.Driver'. 
    The driver is automatically registered via the SPI 
    and manual loading of the driver class is generally unnecessary.

还有一点说明的是，在配置文件中设置useSSL=false。是因为
不推荐不使用服务器身份验证来建立SSL连接。
如果未明确设置，MySQL 5.5.45+, 5.6.26+ and 5.7.6+版本默认要求建立SSL连接。
为了符合当前不使用SSL连接的应用程序，verifyServerCertificate属性设置为’false’。
如果你不需要使用SSL连接，你需要通过设置useSSL=false来显式禁用SSL连接。
如果你需要用SSL连接，就要为服务器证书验证提供信任库，并设置useSSL=true。
注明： SSL – Secure Sockets Layer（安全套接层）

### 2.连接mysql数据库出现CommunicationsException
在运行程序后，出现了com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure错误。后经检查发现是数据库的URL写错了端口号应该是3306错写成了8080，尴尬。

查阅网上的资料发现，出现这种错误还有另外一个原因，这里也一并记下。
mysql5数据库配置引起。mysql5将其连接的等待时间(wait_timeout)缺省为8小时。通过在mysql中执行
    mysql > show global variables like 'wait_timeout'; 
    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | wait_timeout  | 28800 |
    +---------------+-------+
    1 row in set (0.00 sec)
28800sec，也就是8小时。如果在wait_timeout期间，数据库连接(java.sql.Connection)一直处于等待状态，mysql5就将该连接关闭。这时，你的java应用的连接池仍然合法地持有该连接的引用。当用该连接来进行数据库操作时，就碰到上述错误。
**解决办法：**
1.mysql5以前的版本可以直接在jdbc连接url的配置中附加上“autoReconnect=true”。
2.将mysql的全局变量wait_timeout的值修改为最大。查看mysql5的手册，发现windows和linux下wait_timeout的最大值分别是24天和365天。
(1).在文件my.ini的最后增加一行：wait_timeout=1814400。（该文件，windows下在mysql的安装目录下，linux下位置为/etc/my.ini）
(2).重启mysql。
3.如果经过了以上的步骤，你的问题依旧没有的到解决，则建议你修改下你程序中的mysql驱动的版本，将其升级为mysql6。

### 3.java.lang.InstantiationException异常的分析与解决
java.lang.InstantiationException是指不能实例化某个对象。
一般我们在使用java反射机制去创建某个对象的时候实例化到了一个抽象类或者接口(java中抽象类和接口是不能被实例化)，这次的异常是在使用反射机制实例化一个数据库对应的持久化类时，抛出了这个异常。
原因是：Hibernate要求每一个持久化类都必须带一个不带参数的构造方法。如果你在类中声明了带参数的构造函数，会自动覆盖无参数的构造函数，这样系统就无法调用无参数的构造函数实例化类，所以会抛出这个异常。

### 4.关于SpringBoot中jpa org.springframework.dao.InvalidDataAccessResourceUsageException的分析与解决
在SpringBoot项目中添加数据抛出了这个异常。解决的办法是在使用自增长生成策略中声明strategy = GenerationType.IDENTITY即可。如下：
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer categoryId;
下面详细说明一下@GeneratedValue注解，在JPA中，@GeneratedValue注解的作用就是为一个实体生成一个唯一标识的主键(JPA要求每一个实体类，必须有且只有一个主键)，@GeneratedValue提供了主键的生成策略，它有两个属性，分别是strategy和generator属性的值是一个字符串，默认为”“，其声明了主键生成器的名称(对应于同名的主键生成器@SequenceGenerator和@TableGenerator)。
JPA为开发人员提供了四种主键生成策略，它们被定义在枚举类GenerationType中，包括
GenerationType.TABLE
GenerationType.SEQUENCE
GenerationType.IDENTITY
GenerationType.AUTO
下面分别介绍这四种主键生成策略。
1.