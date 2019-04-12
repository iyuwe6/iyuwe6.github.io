---
title: 学习JVM(2)——类的加载机制
date: 2019-03-19 10:29:06
tags: 
  - JVM
categories: 
  - JVM
---

## 1.类的加载机制
类的加载指的是将类.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区的数据结构。类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。
<!--more-->
类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到.class文件缺失或错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError错误）如果该类一直没有被程序主动使用，那么类加载器就不会报告错误。
**加载.class文件的方式**

- 从本地系统中直接加载
- 通过网络下载.class文件
- 从zip,jar等归档文件中加载.class文件
- 从专有数据库中提取.class文件
- 将Java源文件动态编译为.class文件
## 2.类的生命周期

![](https://ws1.sinaimg.cn/large/006aBttAly1g1h2j37savj30ja069jrt.jpg)

类从被加载到虚拟机内存中，到写在出内存为止，整个的生命周期包括：加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)7个阶段。其中准备、验证、解析3个部分统称为连接(Linking)。加载、验证、准备、初始化和卸载这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定（也称为动态绑定或晚期绑定）。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或者完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。

### 1. 加载
查找并加载类的二进制数据是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：

1. 通过一个类的全限定名来获取其定义的二进制字节流（并没有指明要从一个Class文件中获取，可以从其他渠道，譬如：网络、动态生成、数据库等）；
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构；
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

加载和连接阶段的部分内容（如一部分字节码文件格式验证动作）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些夹在加载阶段之中进行的动作，仍然属于连接阶段的内容，但这两个阶段的开始时间仍然保持固定的先后顺序。

### 2. 验证

验证是连接阶段的第一步，这一阶段的目的是为了确保class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

验证阶段大致会完成4个阶段的检验动作：

1. 文件格式验证：验证字节流是否符合class文件格式的规范；例如：是否以魔术0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围内、常量池中的常量是否有不被支持的类型。
2. 元数据验证：对字节码描述的信息进行语义分析，以保证其描述的信息符合Java语言规范的要求；例如，这个类是否有父类，除了java.lang.Object之外。
3. 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
4. 符号引用验证：确保解析动作能正确执行。

验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用-Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

### 3. 准备

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用的内存都将在方法区中进行分配。这时候进行内存分配的仅包括类变量（被static修饰的变量），而不包括实例变量，实例变量将会在对象实例化时随着对象一起分配在堆中。这里所说的初始值“通常情况”下是数据类型的零值，例如，假设一个类变量定义为：
> public static int value=123;

那变量value在准备阶段完成后的初始值为0而不是123，因为这时候尚未开始执行任何java方法，而把value赋值为123的putstatic指令是程序被编译之后，存放于类构造器<client>()方法之中，所以把value赋值为123的动作将在初始化阶段才会执行，至于“特殊情况”是指：
> public static final int value = 123;

即当类字段的字段属性是ConstantValue时，会在准备阶段初始化为指定的值，所以标注为final后，value的值在准备阶段初始化为123而非0。

需要注意以下几点：

- 对基本数据类型来说，对于类变量(static)和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值（如0，0L，null，false等），而对于局部变量，在使用前必须显式地为其赋值，否则编译时不通过。
- 对于同时被static和final修饰的常量，必须在声明的时候就为其显式地赋值，否则编译时不通过；而只被final修饰的常量则既可以在声明时显式地为其赋值，也可以在类初始化时显式地为其赋值，总之，在使用前必须为其显式地赋值，系统不会为其赋予默认零值。
- 对于引用数据类型reference来说，如数组引用、对象引用等，如果没有对其进行显式地赋值而直接引用，系统都会为其赋予默认的零值，即null。
- 如果在数组初始化时没有对数组中的各元素赋值，那么其中的元素将根据对应的数据类型而被赋予默认的零值。

### 4. 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。符号引用就是class文件中的CONSTANT_Class_info/CONSTANT_Field_info/CONSTANT_Method_info等类型的变量。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。

下面解释一下符号引用和直接引用的概念：

1. 符号引用就是一组符号来描述目标，可以是任何字面量。符号引用与虚拟机实现的布局无关，引用的目标并不一定要已经加载到内存中。各种虚拟机实现的内存布局可以各不相同，但是它们能接受的符号引用必须是一致的，因为符号引用的字面量形式明确定义在java虚拟机规范的Class文件格式中。
2. 直接引用可以是指向目标的指针，相对偏移量或是一个能间接定位到目标的句柄。如果有了直接引用，那引用的目标必定已经在内存中存在。

### 5. 初始化

类初始化阶段是类加载过程的最后一步，前面的类加载阶段，除了在加载阶段可以自定义类加载器以外，其他操作都由JVM主导。到了初始化阶段，才真正开始执行类中定义的java程序代码。在准备阶段，变量已经赋过一次系统要求的初始值，而在初始化阶段，则根据程序员通过程序制定的主观计划去初始化类变量和其他资源，或者说：初始化阶段是执行类构造器clinit()方法的过程。

clinit()方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块static{}中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问。如下：

```
public class Test
{
    static
    {
        i=0;
        System.out.println(i);//这句编译器会报错：Cannot reference a field before it is defined（非法向前应用）
    }
    static int i=1;
    
    public static void main(String args[])
    {
        System.out.println(i);
    }
}
```

当去掉报错的语句后，输出的结果为1。在准备阶段i=0，然后类初始化阶段按照顺序执行，首先执行static块中的i=0，接着执行static赋值操作i=1，最后在main方法中获取i的值为1。

clinit()方法与实例构造器init()方法不同，它不需要显式的调用父类构造器，虚拟机会保证在子类clinit()方法执行之前，父类的clinit()方法已经执行完毕。由于父类的clinit()方法先执行，也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作。

clinit()方法对于类或者接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生产clinit()方法。

接口中不能使用静态语句块，但仍然有变量初始化的赋值操作，因此接口与类一样都会生成clinit()方法。但接口与类不同的是，执行接口的clinit()方法不需要先执行父接口的clinit()方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的clinit()方法。

虚拟机会保证一个类的clinit()方法在多线程环境中被正确的加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的clinit()方法，其他线程都需要阻塞等待，直到活动线程执行clinit()方法完毕。如果在一个类的clinit()方法中有耗时很长的操作，就可能造成多个线程阻塞，在实际应用中这种阻塞往往是隐藏的。例如：

```java
package jvm.classload;

public class DealLoopTest
{
    static class DeadLoopClass
    {
        static
        {
            if(true)
            {
                System.out.println(Thread.currentThread()+"init DeadLoopClass");
                while(true)
                {
                }
            }
        }
    }

    public static void main(String[] args)
    {
        Runnable script = new Runnable(){
            @Override
            public void run()
            {
                System.out.println(Thread.currentThread()+" start");
                DeadLoopClass dlc = new DeadLoopClass();
                System.out.println(Thread.currentThread()+" run over");
            }
        };

        Thread thread1 = new Thread(script);
        Thread thread2 = new Thread(script);
        thread1.start();
        thread2.start();
    }
}
```

运行结果：（即一条线程在死循环以模拟长时间操作，另一条线程在等待）

```
Thread[Thread-0,5,main] start
Thread[Thread-1,5,main] start
Thread[Thread-0,5,main]init DeadLoopClass
```

需要注意的是，其他线程虽然会被阻塞，但如果执行clinit()方法的那条线程退出clinit()方法后，其他线程唤醒之后不会再次进入clinit()方法。同一个类加载器，一个类型只会初始化一次。

将上述代码中的静态代码部分改为：

```java
static
{
    System.out.println(Thread.currentThread() + "init DeadLoopClass");
    try
    {
        TimeUnit.SECONDS.sleep(10);
    }
    catch (InterruptedException e)
    {
        e.printStackTrace();
    }
}
```

运行结果：

```
Thread[Thread-0,5,main] start
Thread[Thread-1,5,main] start
Thread[Thread-1,5,main]init DeadLoopClass (之后sleep 10s)
Thread[Thread-1,5,main] run over
Thread[Thread-0,5,main] run over
```

虚拟机规范严格规定了有且只有5种情况(jdk1.7)必须对类进行“初始化”（而加载、验证、准备自然需要在此之前开始）：

1. 遇到new,getstatic,putstatic,invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
2. 使用java.lang.reflect包的方法对类进行反复调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
3. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
5. 当使用jdk1.7动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getstatic,REF_putstatic,REF_invokestatic的方法句柄，并且这个方法句柄所对应的类没有进行初始化，则需要先触发其初始化。

注意以下几种情况不会执行类“初始化”：

1. 通过子类引用父类的静态字段，只会触发父类的初始化，而不会触发子类的初始化。
2. 定义对象数组，不会触发该类的初始化。
3. 常量在编译期间会存入调用类的常量池中，本质上并没有直接引用定义常量的类，不会触发定义常量所在的类的初始化。
4. 通过类名获取Class对象，不会触发类的初始化。
5. 通过Class.forName加载指定类时，如果指定参数initialize为false时，也不会触发此类初始化，其实这个参数时告诉虚拟机，是否要对类进行初始化。
6. 通过ClassLoader默认的loadClass方法，也不会触发初始化动作。

下面用一个代码示例来说明：

```java
class SSClass
{
    static
    {
        System.out.println("SSClass");
    }
}    
class SuperClass extends SSClass
{
    static
    {
        System.out.println("SuperClass init!");
    }

    public static int value = 123;

    public SuperClass()
    {
        System.out.println("init SuperClass");
    }
}
class SubClass extends SuperClass
{
    static 
    {
        System.out.println("SubClass init");
    }

    static int a;

    public SubClass()
    {
        System.out.println("init SubClass");
    }
}
public class NotInitialization
{
    public static void main(String[] args)
    {
        System.out.println(SubClass.value);
    }
}
```

运行结果：

```
SSClass
SuperClass init!
123
```

也许有人会疑问为什么没有输出SubClass init。因为：对于静态字段，只有直接定义这个字段的类才会被初始化，因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。

再举两个例子：

1. 通过数组定义来引用类，不会触发此类的初始化：（SuperClass类已在上述代码定义）

```java
public class NotInitialization
{
    public static void main(String[] args)
    {
        SuperClass[] sca = new SuperClass[10];
    }
}
```

运行结果：（无）

2. 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化

```
public class ConstClass
{
    static
    {
        System.out.println("ConstClass init!");
    }
    public static final String HELLOWORLD = "hello world";
}
public class NotInitialization
{
    public static void main(String[] args)
    {
        System.out.println(ConstClass.HELLOWORLD);
    }
}
```

运行结果：hello world 

最后一个例子：

```java
package jvm.classload;

public class StaticTest
{
    public static void main(String[] args)
    {
        staticFunction();
    }

    static StaticTest st = new StaticTest();

    static
    {
        System.out.println("1");
    }

    {
        System.out.println("2");
    }

    StaticTest()
    {
        System.out.println("3");
        System.out.println("a="+a+",b="+b);
    }

    public static void staticFunction(){
        System.out.println("4");
    }

    int a=110;
    static int b =112;
}
```

一般来说，java中的赋值顺序是这样的：

1. 父类的静态变量赋值
2. 自身的静态变量赋值
3. 父类成员变量赋值和父类块赋值
4. 父类构造函数赋值
5. 自身成员变量赋值和自身块赋值
6. 自身构造函数赋值

按照这个理论，上述代码的输出是：1 4。但是实际输出并不是这样的，但不是说上面的规则不正确，只是这段代码并不能简单地套用规则。

正确的输出是：

```
2
3
a=110,b=0
1
4
```

这段代码关键的点在于：实例初始化不一定要在类初始化结束之后才开始初始化。

类的生命周期是：加载->验证->准备->解析->初始化->使用->卸载，只有在准备阶段和初始化阶段才会涉及类变量的初始化和赋值，因此只针对这两个阶段进行分析；

类的准备阶段需要做的是为类变量分配内存并设置默认值，因此类变量st为null、b为0；（需要注意的是如果类变量是final，编译时javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将变量设置为指定的值，如果这里定义：static final int b=112，那么在准备阶段b的值就是112，而不是0了。）

类的初始化阶段需要做的是执行类构造器（类构造器是编译器收集所有静态语句块和类变量的赋值语句，按语句在源码中的顺序合并成类构造器，对象的构造方法是init()，类的构造方法是clinit()，可以在堆栈信息中看到），因此先执行第一条静态变量的赋值语句即st = new StaticTest()，此时会进行对象的初始化，对象的初始化是先初始化成员变量再执行构造方法，因此设置a为110->打印2->执行构造方法（打印3，此时a已经赋值为110，但是b只是设置了默认值0，并未完成赋值操作），等对象的初始化完成后继续执行之前的类构造器的语句，接下来就不详细说了，按照语句在源码中的顺序执行即可。

这里面还牵涉到一个在嵌套初始化时的特别逻辑，特别是内嵌的这个变量恰好是个静态成员，而且是本类的实例。这会导致“实例初始化竟然出现在静态初始化之前”。

在以下几种情况下，Java虚拟机将结束生命周期：

- 执行了System.exit()方法
- 程序正常执行结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统出现错误而导致Java虚拟机进程终止

## 3. 类加载器

### 1. 类加载器种类

从Java虚拟机的角度说，只有两种不同的类加载器：

- 启动类加载器，使用C++实现（这里仅限于Hotspot，也就是JDK1.5之后默认的虚拟机，有很多其他的虚拟机是用Java语言实现的），是虚拟机自身的一部分；
- 所有其他的类加载器，这些类加载器都由Java语言实现，独立于虚拟机之外，并且全部继承自抽象类java.lang.ClassLoader，这些类加载器需要由启动类加载器加载到内存中之后才能去加载其他的类。

从Java开发人员的角度来说，虚拟机设计团队把加载动作放到JVM外部实现，以便让应用程序决定如何获取所需的类，JVM提供了3种类加载器：

1. 启动类加载器（Bootstrap ClassLoader）：负责加载存放在JDK\jre\lib目录中的，或通过-Xbootclasspath参数指定路径中的，且被虚拟机识别（按文件名识别，如rt.jar，所有的java.开头的类）的类。启动类加载器是无法被Java程序直接引用的。启动类加载器是无法被Java程序直接引用的。
2. 扩展类加载器（Extension ClassLoader）：该加载器由sun.misc.Launcher$ExtClassLoader实现，负责加载JDK\jre\lib\ext目录中的，或通过java.ext.dirs系统变量指定路径中的类库（如javax.开头的类），开发者可以直接使用扩展类加载器。
3. 应用程序类加载器（Application ClassLoader）：该类加载器由sun.misc.Launcher$AppClassLoader实现，负责加载用户路径上的类库。开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中的默认类加载器。

应用程序都是由这三种类加载器互相配合进行加载的，如果有必要，我们还可以加入自定义的类加载器，因为JVM自带的ClassLoader只会从本地文件系统加载标准的java.class文件，因此如果编写了自己的ClassLoader，便可以做到如下几点：

- 在执行非置信代码之前，自动验证数字签名
- 动态地创建符合用户特定需要的定制化构建类
- 从特定的场所取得java class，例如数据库或网络中

### 2. 类的加载

JVM类加载机制：

- 全盘负责：当一个类加载器负责加载某个class时，该class所依赖的和引用的其他class也将由该类加载器负责载入，除非显示使用另外的类加载器来载入
- 父类委托：先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类。
- 缓存机制：缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class之后，必须重启JVM，程序的修改才会生效。

类加载的三种方式：

- 命令行启动应用时候由JVM初始化加载
- 通过Class.forName()方法动态加载
- 通过ClassLoader.loadClass()方法动态加载

示例：

```java
package com.neo.classloader;
public class loaderTest {
    public static void main(String[] args) throws ClassNotFoundException {
        ClassLoader loader = Test2.class.getClassLoader();
        System.out.println(loader);
        //使用ClassLoader.loadClass()来加载类，不会执行初始化块 
        loader.loadClass("com.neo.classloader.Test2");
        //使用Class.forName()来加载类，默认会执行初始化块 
        //Class.forName("com.neo.classloader.Test2"); 
        //使用Class.forName()来加载类，并指定ClassLoader，false表示初始化时不执行静态块
        //Class.forName("com.neo.classloader.Test2", false, loader); 
    }
}
class Test2 {
    static {
        System.out.println("静态初始化块执行了！");
    }
}
```

分别切换加载方式，会有不同的输出结果。

这里说一下Class.forName()和ClassLoader.loadClass()区别：

- Class.forName()：将类的.class文件加载到jvm中，还会对类进行解释，执行类中的static块；
- ClassLoader.loadClass()：只干一件事情，就是将.class文件加载到jvn中，不会执行static中的内容，只有在new Instance时才会去执行static块。
- Class.forName(name, initialize, loader)带参函数也可控制是否加载static块，并且只有调用了new Instance()方法采用调用构造函数，创建类的对象。

### 3. 双亲委派模型

JVM通过双亲委派模型进行类的加载，当然我们也可以通过继承java.lang.ClassLoader实现自定义的类加载器。注意：这里父类加载器并不是通过继承关系来实现的，而是采用组合实现的。

![](https://ws1.sinaimg.cn/large/006aBttAly1g1ynxck7v7j30ge0db3zn.jpg)

当一个类加载器收到类加载任务时，会先交给其父类加载器去完成，因此最终加载任务都会传递到顶层的启动类加载器，只有到父类加载器无法完成加载任务时，子类加载器才会尝试执行加载任务。

双亲委派机制：

- 当 `AppClassLoader`加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器`ExtClassLoader`去完成。

- 当 `ExtClassLoader`加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给`BootStrapClassLoader`去完成。
- 如果 `BootStrapClassLoader`加载失败（例如在 `$JAVA_HOME/jre/lib`里未查找到该class），会使用 `ExtClassLoader`来尝试加载；
- 若`ExtClassLoader`也加载失败，则会使用 `AppClassLoader`来加载，如果 `AppClassLoader`也加载失败，则会报出异常 `ClassNotFoundException`。

采用双亲委派的一个好处是比如加载位于rt.jar包中的类java.lang.Object，不管是哪个加载器加载这个类，最终都是委派给顶层的启动类加载器进行加载，这样就能保证了使用不同的类加载器最终得到的都是同洋一个object对象。

双亲委派模型意义：

- 系统类防止内存中出现多份同样的字节码
- 保证Java程序安全稳定运行

在有些情境中可能会出现要求我们自己来实现一个类加载器的需求。这里可以看一下jdk中的ClassLoader源码实现：

```
protected synchronized Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException {
    // First, check if the class has already been loaded
    // 首先判断该类型是否已经被加载
    Class c = findLoadedClass(name);
    if (c == null) {
        //如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
        try {
            if (parent != null) {
                //如果存在父类加载器，就委派给父类加载器加载
                c = parent.loadClass(name, false);
            } else {
                //如果不存在父类加载器，就检查是否是由启动类加载器加载的类，通过调用本地方法native Class findBootstrapClass(String name)
                c = findBootstrapClass0(name);
            }
        } catch (ClassNotFoundException e) {
            // If still not found, then invoke findClass in order
            // to find the class.
            // 如果父类加载器和启动类加载器都不能完成加载任务，才调用自身的加载功能
            c = findClass(name);
        }
    }
    if (resolve) {
        resolveClass(c);
    }
    return c;
}
```

1. 首先通过Class c = findLoadedClass(name)判断一个类是否已经被加载过；
2. 如果没有被加载过，则执行if(c == null)中的程序，遵循双亲委派的模型，首先会通过递归从父加载器开始找，直到父类加载器是Bootstrap ClassLoader为止；
3. 最后根据resolve的值，判断这个class是否需要解析。

而上面的findClass()的实现如下，直接抛出一个异常，并且方法是protected，很明显这是留给我们开发者自己去实现的。

```
protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}
```

### 4. 自定义类加载器

一般情况下，我们都是直接使用系统类加载器，但是，有的时候，我们也需要自定义类加载器。比如，应用是通过网络来传输的java类的字节码，为保证安全性，这些字节码经过了加密处理，这是系统类加载器就无法对其进行加载，这样则需要自定义类加载器来实现。自定义类加载器一般都是继承自ClassLoader类，从上面对loadClass方法来分析来看，我们只需要重写findClass方法即可。

下面用一个示例来演示自定义类加载器的流程：

```java
package jvm.classload;
import java.io.*;

public class MyClassLoader extends ClassLoader{
    private String root;

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }
    private byte[] loadClassData(String className){
        String fileName = root + File.separatorChar + className.replace('.', File.separatorChar) + ".class";
        InputStream ins = null;
        try {
            ins = new FileInputStream(fileName);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length = 0;
            while ((length = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, length);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (ins != null) {
                try {
                    ins.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }

    public String getRoot() {
        return root;
    }

    public void setRoot(String root) {
        this.root = root;
    }

    public static void main(String[] args) {
        MyClassLoader classLoader = new MyClassLoader();
        classLoader.setRoot("D:\\workspace\\jvmTest\\out\\production\\jvmTest");
        //D:\workspace\jvmTest\out\production\jvmTest\jvm\classload\Test2.class
        Class<?> testClass = null;
        try {
            testClass = classLoader.findClass("jvm.classload.loaderTest");
            Object object = testClass.newInstance();
            System.out.println(object.getClass().getClassLoader());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```

运行结果：

```java
jvm.classload.MyClassLoader@4554617c
```

自定义类加载器的核心在于对字节码文件的获取，如果是加密的字节码则需要在该类中对文件进行解密。由于这里只是演示，我并未对class文件进行加密，因此没有解密的过程。这里有几点需要注意：

- 这里传递的文件名需要是类的全限定名，即com.paddx.test.classloading.Test格式的，因为defineClass方法是按这种格式进行处理的。
- 最好不要重写loadClass方法，因为这样容易破坏双亲委派模式。
- 这类Test类本身可以被AppClassLoader类加载，因此我们不能把com/paddx/test/classloading/Test.class放在类路径下。否则，由于双亲委派模式的存在，会直接导致该类有AppClassLoader加载，而不会通过我们自定义类加载器来加载。

## 参考文档：

1. [Java虚拟机类加载机制](<https://blog.csdn.net/u013256816/article/details/50829596>)
2. [Java虚拟机类加载机制——案例分析](<https://blog.csdn.net/u013256816/article/details/50837863>)
3. [JVM 类加载机制详解](<http://www.importnew.com/25295.html>)
4. [java类的加载机制](<https://www.cnblogs.com/ityouknow/p/5603287.html>)