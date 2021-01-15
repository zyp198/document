## JVM学习总结



![1610694002262](C:\Users\intple\IdeaProjects\leetcode\src\code\shangguigu\Typora Note\JVM学习总结.assets\1610694002262.png)

### 一、JVM结构概念

#### 1、谈谈你对JVM的认识？

需要了解Java虚拟机就不得不了解 java的类加载机制。我们知道我们编写的类，也就是一个.java文件，首先是通过javac编译成.class文件【也就是字节码文件】，然后通过java的类加载器【ClassLoader】加载到jvm中，类加载时符合双亲委派原则。jvm在内存中的结构主要分为五个区域。按照线程私有和线程共享来进行分类，其中线程共享分为方法区和堆，并且垃圾回收【GC】是针对于这两个区域。线程私有分为java栈，本地方法栈，程序计数器。

#### 2、有哪几种类加载器？

系统中由上至下依次分为：1、Bootstrap【C++】启动类加载器，2、可拓展类加载器【Extension 】3、应用类加载器【Application classLoader】或者叫系统类加载器

用户自定义加载器 extend ClassLoader,通过继承ClassLoader,用户可以自定义类的加载方式。

#### 3、什么是双亲委派原则

当一个类收到了类加载请求之后，类加载器自己不会尝试去加载这个类，而是把这个请求传递给父类，父类也是如此，因此，所有的类加载请求都会传递到启动类加载器。只有父类无法完成这个请求时，子类就会尝试去加载这个类。当所有的类都无法完成加载的时候，就会报ClassNotFoundException异常。

​	优势：保证了在使用不同的类加载器时最终得到的都是同一个对象。判断是否是同一个对象，不但要判断它的全限定类名，而且还要判断是否是同一个类加载器。

#### 4、如何破坏双亲委派模型？

在某些情况下需要由子类去加载class文件，这时就需要破坏双亲委派模型。可以通过重写ClassLoader类的loadClass方法来实现。

#### 5、介绍一下JVM中的五大结构。

​	1、方法区

​	2、堆

​	3、JVM栈

​	4、本地方法栈

​	5、程序计数器

```java
3、native关键字

	3.1 native是一个关键字
        本地接口的作用是融合不同的编程语言为 Java 所用，它的初衷是融合 C/C++程序，Java 诞生的时候是 C/C++横行的时候，要想立足，必须有调用 C/C++程序，于是就在内存中专门开辟了一块区域处理标记为native的代码，它的具体做法是 Native Method Stack中登记 native方法，在Execution Engine 执行时加载native libraies。
        目前该方法使用的越来越少了，除非是与硬件有关的应用，比如通过Java程序驱动打印机或者Java系统管理生产设备，在企业级应用中已经比较少见。因为现在的异构领域间的通信很发达，比如可以使用 Socket通信，也可以使用Web Service等等，不多做介绍。
	3.2 native只有声明没有实现
	3.3 native method stack
	    它的具体方式是做native method stack 中登记native方法，在Execution engine 执行时加载本地方法库。


4、PC寄存器
    4.1 程序计数器
        4.1.1 线程私有，实际上是一个指针（指针，指向方法区中的字节码）
        4.1.2 用来完成分支、循环、跳转、异常处理、线程恢复等操作
        4.1.3 所需要得内存空间非常小，不会发生OOM
        4.1.4 如果是一个Naive方法，那么这个计数器是空的

5、java栈
	5.1 栈管运行，堆管存储。
	5.2 栈也叫栈内存，主要负责java程序的运行，是线程创建的时候创建，他的生命周期是和线程一样的
	5.3 对于栈来说不存在垃圾回收的问题。（原因：线程结束栈内存也就释放）
	5.4 线程私有
	5.5 8种基本类型的变量+对象的引用+实例方法
	    5.5.1 什么是栈帧？等同于java方法。在栈中的方法叫做栈帧
	    5.5.2 栈的运行原理
	    5.5.3 Exception in thread "main" java.lang.StackOverflowError

6、方法区(java7 永久带 ; java8 元空间 mataspace) 永久代使用的是堆内存 元空间使用的是本机物理内存
    6.1 线程共享区域
    6.2 存储一个类的结构信息，例如运行时常量池，字段和方法数据、构造方法和普通方法的字节码内容
    6.3 方法区是一套规范。它的实现有两种方式，分别是永久代【jdk7】和元空间【jdk8】
    6.4 实例变量存放在堆内存中【可以GC】，和方法区无关
        什么是实例变量？引用变量？实例方法？
7、堆
    7.1 一个JVM的实例只存在一个堆内存，堆内存的大小是可以调节的。类加载器读取了类文件后，需要把类、方法、常变量放到堆内存中。保存所有引用类型的真实信息。
    7.2 堆内存分为 新生区【伊甸区 幸存者0区 幸存者1区 = 8 1 1】、养老区、永久区
    7.3 Java.lang.OutOfMemoryError:java heap space?
        7.3.1 java 堆内存不够用
        7.3.2 代码中创建了大量的大对象，并且长时间不被GC收集
    7.4 第一次GC什么时候触发？MinorGC过程 （复制-》交换-》清空）
        7.4.1 首先在eden区满的时候会触发第一次GC，把活着的对象拷贝到SurvivorFrom区
        7.4.2 当再次触发GC的时候会扫描Eden和SurvivorFrom区，对这两个区域进行垃圾回收，经过这次回收还存活的对象，则直接复制到SurvivorTo区域 复制的过程是什么样的？
        7.4.3 清空Eden、SurvivorFrom区
        7.4.4 SurvivorTo 和 SurvivorFrom交换
              最后SurvivorTo和SurvivorFrom互换，原先SurvivorTo成为下一次GC扫描时的SurvivorFrom 原则是谁空谁是SurvivorTo区
```

### 二、JVM 垃圾回收算法及垃圾回收器

#### **1、JVM 【GC算法】**

​	1、引用计数法

​		实现原理：后台搭建一个引用计数器，用来记录对象引用了几次，如果该对象引用了0次【即判断该对象没有被引用】，即可进行GC。

​		优势：实现方式简单。

​		劣势：每次对象进行赋值时，都需要维护引用计数器，且计数器本身也是一种消耗；

​			    难以处理循环引用。

​		目前JVM GC算法一般不会采用这种方式

​	2、复制算法

![1610699394546](C:\Users\intple\IdeaProjects\leetcode\src\code\shangguigu\Typora Note\JVM学习总结.assets\1610699394546.png)

​				MinorGC的过程（复制->清空->互换）

​		2.1 当伊甸区满的时候触发第一次GC，【针对于eden区】，将还活着的对象拷贝到survivorFrom区,当eden区再次触发GC的时候会扫描eden区和survivorFrom区，对两个区域的垃圾进行回收，经过这次回收还活着的对象，则直接复制到to区域【如果有对象的年龄达到了老年的标准，则直接复制到老年代】，同时把对象的年龄+1

​		2.2 清空eden、survivorFrom区

​		2.3 SurvivorTo和SurvivorFrom互换位置【互换位置的标准谁空谁是To】

​			

​	3、标记清除算法

![1610700368884](C:\Users\intple\IdeaProjects\leetcode\src\code\shangguigu\Typora Note\JVM学习总结.assets\1610700368884.png)

​	4、标记整理算法

![1610700383862](C:\Users\intple\IdeaProjects\leetcode\src\code\shangguigu\Typora Note\JVM学习总结.assets\1610700383862.png)

#### 2、GC相关的题目

1、JVM垃圾回收的时候如何确定垃圾？什么是GCRoots？

​	什么是垃圾？简单的说就是内存中已经不在被使用到的空间就是垃圾

​	要进行垃圾回收，如何判断一个对象是否可以被回收？两种方式：1、引用计数法【不使用，无法解决循环依赖的问题】2、枚举根节点做可达性分析（根搜索路径）

2、你说你做过JVM的调优，请问如何盘点查看JVM系统的默认值

-XX:+PrintFlagsInitial 查看初始默认值

-XX:+printFlagsFinal 查看修改更新

-XX:+printCommadLineFlags

3、枚





