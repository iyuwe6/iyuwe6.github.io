---
title: SpringBoot(5)——AOP的配置和应用于日志管理
date: 2018-10-25 21:04:10
tags: 
  - SpringBoot
categories: 
  - SpringBoot
---

Spring有两个最核心的理念，一是控制反转(IOC)，二是面向切面编程(AOP)。这里我们来看一下如何配置AOP，并在SpringBoot中使用AOP统一处理web请求日志。
<!--more-->

### AOP简介
AOP全称Aspect Oriented Programming，意为“面向切面编程”，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。AOP是Spring框架中的一个重要内容，它通过对既有程序定义一个切入点，然后在其前后切入不同的执行内容，比如常见的有：打开数据库连接/关闭数据库连接、打开事务/关闭事务、记录日志等。基于AOP不会破坏原来程序逻辑，因此它可以很好的对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。
比如，若是需要一个记录日志的功能，首先想到的是在方法中通过log4j或其他框架来进行记录日志，但写下来发现一个问题，在整个业务中其实核心的业务代码并没有多少，都是一些记录日志或其他辅助性的一些代码。而且很多业务有需要相同的功能，比如都需要记录日志，这时候又需要将这些记录日志的功能复制一遍，即使是封装成框架，也是需要调用之类的。在此处使用复杂的设计模式又得不偿失。所以就需要面向切面出场了

### AOP的配置
在pom.xml中加入如下依赖：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
在引入了AOP依赖后，一般来说并不需要去做其他设置，使用默认配置即可。
### 使用AOP统一处理web请求日志
实现AOP的切面主要有以下几点：
- 使用@Aspect注解将一个java类定义为切面类
- 使用@Pointcut定义一个切入点，可以是一个规则表达式，比如下面例子中的某个package下的所有函数，也可以是一个注解等。
- 根据需要在切入点不同位置的切入内容
- 1.使用@Before在切入点开始处切入内容
- 2.使用@After在切入点结尾处切入内容
- 3.使用@AfterReturning在切入点return内容之后切入内容（可以用来对处理返回值做一些加工处理）
- 4.使用@Around在切入点前后切入内容，并自己控制何时执行切入点自身的内容
- 5.使用@AfterThrowing用来处理当切入内容部分抛出异常之后的处理逻辑

下面我们来写一个定义切面的类：
```java
@Aspect
@Component
public class WebLogAspect {
    private Logger logger = Logger.getLogger(getClass().getName());

    @Pointcut("execution ( public * com.xd.springboot..*.*(..))")
    public void webLog(){}

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable{
        //接收到请求，记录请求内容
        ServletRequestAttributes attributes =(ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        //记录下请求内容
        logger.info("URL: " + request.getRequestURI().toString());
        logger.info("HTTP_METHOD: " + request.getMethod());
        logger.info("IP: " + request.getRemoteAddr());
        logger.info("CLASS_METHOD: " + joinPoint.getSignature().getDeclaringTypeName() +
                "." + joinPoint.getSignature().getName());
        logger.info("ARGS: " + Arrays.toString(joinPoint.getArgs()));
    }

    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable{
        //处理完请求，返回内容
        logger.info("RESPONSE: " + ret);
    }
}
```

@Pointcut("execution(public * com.xd.springboot..*.*(..))")
上面这段注解的意思如下：
1) execution(): 表达式主体
2) 第一个public * 号：表示返回类型， * 号表示所有的类型。
3) 包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包，com.king.controller包、子孙包下所有类的方法。
4) 第二个 * 号：表示类名， * 号表示所有的类。
5) * (..):最后这个星号表示方法名， * 号表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数

在上面这段代码中，通过@Pointcut定义的切入点为com.xd.springboot包下的所有函数（对web层所有请求处理做切入点），然后通过@Before实现，对请求内容的日志记录，最后通过@AfterReturning记录请求返回的对象。
运行我们的示例项目，并访问：http://localhost:8090/users/hello ，可以看到如下的日志输出：
```
2018-10-25 19:11:59.142 [INFO ] com.xd.springboot.other.aop.WebLogAspect - URL: /users/hello
2018-10-25 19:11:59.143 [INFO ] com.xd.springboot.other.aop.WebLogAspect - HTTP_METHOD: GET
2018-10-25 19:11:59.143 [INFO ] com.xd.springboot.other.aop.WebLogAspect - IP: 0:0:0:0:0:0:0:1
2018-10-25 19:11:59.146 [INFO ] com.xd.springboot.other.aop.WebLogAspect - CLASS_METHOD: com.xd.springboot.controller.MyController.index
2018-10-25 19:11:59.147 [INFO ] com.xd.springboot.other.aop.WebLogAspect - ARGS: []
2018-10-25 19:11:59.154 [INFO ] com.xd.springboot.other.aop.WebLogAspect - RESPONSE: Hello World
```

#### 优化：AOP切面中的同步问题
在WebLogAspect切面中，分别通过doBefore和doAfterReturning两个独立函数实现了切点头部和切点返回后执行的内容，若我们想统计请求的处理时间，就需要在doBefore处记录时间，并在doAfterReturning处通过当前时间与开始处记录的时间计算得到请求处理的消耗时间。
那么我们是否可以在WebLogAspect切面中定义一个成员变量来给doBefore和doAfterReturning一起访问呢？是否会有同步问题呢？
的确，如果我们直接在这里定义基本类型会有同步问题，所以我们可以引入ThreadLocal对象，对于ThreadLocal，可以参考这篇博文《[Java并发编程：深入剖析ThreadLocal](http://www.cnblogs.com/dolphin0520/p/3920407.html "Java并发编程：深入剖析ThreadLocal")》，修改代码如下：
```java
@Aspect
@Component
public class WebLogAspect {

    private Logger logger = Logger.getLogger(getClass().getName());

    ThreadLocal<Long> startTime = new ThreadLocal<>();

    @Pointcut("execution ( public * com.xd.springboot.controller.*.*(..))")
    public void webLog(){}

    @Before("webLog()")
    public void doBefore(JoinPoint joinPoint) throws Throwable{
        //开始时间
        startTime.set(System.currentTimeMillis());

        //接收到请求，记录请求内容
        ServletRequestAttributes attributes =(ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        //记录下请求内容
        logger.info("URL: " + request.getRequestURI().toString());
        logger.info("HTTP_METHOD: " + request.getMethod());
        logger.info("IP: " + request.getRemoteAddr());
        logger.info("CLASS_METHOD: " + joinPoint.getSignature().getDeclaringTypeName() +
                "." + joinPoint.getSignature().getName());
        logger.info("ARGS: " + Arrays.toString(joinPoint.getArgs()));
    }

    @AfterReturning(returning = "ret", pointcut = "webLog()")
    public void doAfterReturning(Object ret) throws Throwable{
        //处理完请求，返回内容
        logger.info("RESPONSE: " + ret);
        //记录请求处理时间
        logger.info("SPEND_TIME: " + (System.currentTimeMillis() - startTime.get()));
    }
}
```

#### 优化：AOP切面的优先级
由于通过AOP实现，程序得到了很好的解耦，但是也会带来一些问题，比如：我们可能会对web层做多个切面，检验用户，检验头信息等等，这个时候经常会碰到切面的处理顺序问题。
所以，我们需要定义每个切面的优先级，我们需要@Order(i)注解来标识切面的优先级。**i的值越小，优先级越高。**假设我们还有一个切面是CheckNameAspect用来检验name，我们将其设置@Order(10),而上文中WebLogAspect设置为@Order(5)，所以WebLogAspect有更高的优先级，这个时候执行顺序是这样的：
- 在@Before中优先执行@Order(5)的内容，再执行@Order(10)的内容
- 在@After和@AfterReturning中优先执行@Order(10)的内容，再执行@Order(5)的内容

对于AOP的详细用法，可以参考这篇博客：https://www.cnblogs.com/lic309/p/4079194.html