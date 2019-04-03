---
layout: post
title: jvm内存结构
date: 2015-09-03
categories: java
tags:
    - jvm
    - byte_code
    - thread
    - stack
---

## 概述

本文在介绍jvm的内存模型之前，我们先介绍下jvm的模型，然后分线程私有和线程共享两个方面详细阐述jvm内存各部分的内容，针对jdk 7 hotspot虚拟机。

## jvm的组织架构

![jvm的组织架构](/images/java/JvmSpec7.png)

1.  classloader:详见[load class](/2015/01/26/load_class)
2.  程序计数器: 线程私有，当前线程所执行代码的行号指示器
3.  栈：线程私有，生命周期和线程相同。每个栈可能包含多个栈帧，每个方法在执行时都会创建一个栈帧。
4.  本地(native)方法栈：线程私有，与jvm栈锁发挥的作用非常相似
5.  堆：线程共享，很大，GC的主要区域，几乎所有的对象实例及数组都要在堆上分配
6.  方法区：有个别名（non-heap）非堆。存储已被jvm加载的类信息，常量、静态变量、即使编译器编译后的代码。
7.  执行引擎：执行字节码，具体可以参照这个[例子](/2015/01/06/jvm_execute_engine)

## jvm内存模型

然后我们以线程私有和线程共享两个方面来详细说下jvm内存各部分的细节

![jvm memory](/images/java/JVM_Internal_Architecture.png)

### 线程私有

jvm允许一个应用程序拥有多线程，并发地运行。在hotspot虚拟机中在java线程和操作系统的线程有一个直接的映射关系。在准备好java线程所有的状态（state）诸如thread-local的存储，分配的buffers，同步的对象，栈和程序计数器之后，会创建本地线程。一旦java线程终止了，本地线程就会被回收。操作系统调度所有的线程并分发给任意可用的cpu。一旦本地的线程被初始化后，它调用java线程的run()方法。当run方法返回时，未被捕获的异常将被处理，线程终止，然后本地线程确认jvm是否需要被终止（比如如果该线程是最终的非守护线程的话，jvm就应该被teminated）。当线程被终止时，本地线程和jvm线程所需要的所有资源都会被释放。      

每个线程的执行都有下列的组成部分:`程序计数器`，`栈`。每个栈有许多`栈帧`组成；每个栈帧又包含`局部变量表`,`操作数栈`,`当前方法所在类的运行时常量池引用`等。

#### 程序计数器（program counter PC）

程序计数器是当前的指令instruction或者叫操作码opcode的地址，native除外。如果当前的方法是native，程序计数器是空的(undefined)，所有的cpu都有一个程序计数器，一般情况下，在一条指令执行完后程序计数器加一，然后指向下一条将要被执行的指令的地址。jvm使用pc来追踪程序现在在哪执行指令，可以看做是当前线程所执行的字节码的行号指示器。字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖程序计数器来完成。 事实上，pc指向的是方法区中的一个内存地址.  

由于jvm的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数。

程序计数器所在的内存区域是jvm规范唯一一个没有规定任何OutOfMemoryError情况的区域。

#### 栈

每个线程都有它自己的栈（线程私有），持有这个线程要执行的每个方法对应的栈帧。栈是后进先出的数据结构，所以要被执行的方法是位于栈顶的。当一个方法被调用时，一个新的栈帧将被创建并add（push）到栈顶。当这个方法return或者在执行过程中未捕获的异常被throw，这个栈帧将会被remove（pop）出栈。

native栈,不是所有的java虚拟机都支持本地方法，支持的会创建一个本地方法栈。如果一个jvm用C的连接模型来实现本地方法调用（java native Invocation，本地方法栈就是一个C的栈。这种情况下，本地方法栈中的参数的顺序和返回值将和典型的c程序一样。通常情况下，一个本地方法可以call back into jvm，并且调用java方法。这样的本地到java的调用将发生在栈上（通常是java栈）。线程将离开native栈，并创建一个新的栈帧在栈上（通常是java栈）

一个栈可以是动态的，也可以固定大小的。如果一个线程需要超过允许的大小，就会报栈溢出错误StackOverFlowError，如果一个栈需要一个新的栈帧，但是没有足够的内存来分配它，就会报内存溢出OutOfMemoryError。

每个方法被执行时都会创建一个栈帧（Stack Frame）。

每个栈帧包含：

1.  局部变量表
2.  返回值
3.  操作数栈
4.  动态链接等

##### 局部变量表

局部变量表包含执行该方法时，所有用到的局部变量，包括this引用，所有的方法参数和一些定义的局部变量。对于类方法（如static方法）来说，方法参数从0开始，对于实例方法，0 slot为this引用。  

一个局部变量可以是：   

1.  基本数据类型:boolean、byte、char、long、short、int、float、double
2.  reference（它不等同于对象本身）
3.  returnAddress（指向了一条字节码指令的地址）

在局部变量表除了long和double外，所有的类型占用一个slot，long和double占用连续的两个slot因为这些类型是64bit而不是32bit。

局部变量表的所需的内存空间在编译后已经确定了的，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

##### 操作数栈

操作数栈在字节码执行的过程中使用，作用和cpu的寄存器差不多。大多数jvm的字节码在入栈，出栈，复制栈顶元素，交换，执行产生值消费值的操作。因此，在字节码中，交换局部变量表和操作数栈的指令是非常频繁的。例如，一个简单的变量初始化编译成字节码后是两条与操作数栈交互的字节码。   

int i;
编译成字节码为：

    0:iconst_0    // 将常量池0入操作数栈
    1:istore_1    // 将栈顶元素出栈，并存入局部变量表中的变量1。

##### 动态链接

每个栈帧都包含一个运行时常量池的引用。这个引用指向当前栈帧所对应的方法所在的类。这个引用为动态链接提供支持。

C/C++代码首先是被编译成一个object文件(xxx.o),然后这些.o文件被链接成一个可执行文件如（executable或者dll）。在链接的过程中，每个xxx.o文件中的符号链接被真实的内存地址（当然这个地址还是相对的，相对于最终的可执行文件。可执行文件加载到内存中还是会有相对地址，线性地址，绝对地址的转换）在java中，这个过程是在运行时动态进行的。  

当一个java的class文件被编译，所有指向变量的引用，方法名都被作为符号链接存储在class的常量池中。一个符号链接是逻辑引用而不是指向真实的内存地址。jvm实现可以选择在什么时候来处理这些符号链接，可以在class文件加载之后，被验证的时候，这叫做急切的或者静态的解析（eager or static resolution）；或者这可以发生在符号链接第一次被使用的时候，这叫做延迟解析（lazy or late resolution）。然而在每个引用第一次被使用的时候，jvm必须表现的好像这个解析发生过，不能抛出任何处理异常？？？（However the JVM has to behave as if the resolution occurred when each reference is first used and throw any resolution errors at this point）。绑定是这些有符号链接标识的域，方法或者类被一个直接引用代替的过程，这仅仅发生一次，因为符号链接已经完全有直接引用代替。如果指向类的符号链接没有被处理，这个类会被加载。每个直接引用作为一个相对于存储结构的偏移地址，这个存储结构和运行时的变量和方法相关。

#### 本地方法栈

本地方法栈（native method stack）与jvm栈所发挥的作用是非常相似的，不同的是jvm栈为jvm执行字节码服务，而本地方法栈则为jvm使用到的native方法服务。在jvm规范中对本地方法使用的语言、使用的方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。对于hotspot，本地方法栈和虚拟机栈是合二为一的。

### 线程共享


#### 堆

所有的对象实例以及数组都要在堆上分配。数组和对象不能存储在栈中，因为一个栈帧在创建之后就不能改变大小。栈帧仅仅存储堆中对象或者数组的引用。不像基本类型和局部变量表中的引用（在每个栈帧中），对象总是存储在堆中，因此当一个方法结束时,它们不被移除。对象仅会被垃圾收集器收集。  

ps：随着jit编译器的发展与逃逸分析技术逐渐成熟，栈上分配，标量替换优化技术将会导致一些微妙的变化发生，所有对象都分配在堆上也渐渐变得不那么“绝对了”。

为了支持垃圾回收，堆被划分为这三个部分：      

1.  年轻代(经常被划分为eden和survivor)
2.  老年代

对象和数组只会被垃圾回收器回收。  
典型的过程如下：       

1.  新的对象和数组在年轻代创建
2.  minor gc将在年轻代发生。仍然存活的对象会从eden区移入survivor区
3.  major gc（full gc）这通常会造成应用线程暂停，会将对象在代之间移动。仍然存活的对象会被从年轻代移入老年代（old 或者tenured）
4.  当老年代被回收的时候，永久代也会被回收，他们任一满的时候都会触发full gc来对他们进行回收。

#### 非堆内存

非堆内存(方法区)与java堆一样，是各个线程共享的内存区域，存储jvm加载的类信息、常量、静态变量、即时编译器编译后的代码（code cache）等
。虽然jvm规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫non-heap，目的是与java堆区分开来（从jconsole可以看出来）。

![方法区](/images/java/method_area.png)

ps: 对于hotspot，很多人更愿意把方法区称为“永久代”，本质上两者并不等价，或者说用永久代来时间方法区而已，这样hotspot的垃圾回收器就可以像管理java堆一样管理这部分内存了，对于其他虚拟机（如jRockit，IBM J9）来说不存在永久代的概念。对于hotspot，更容易遇到内存溢出的问题（-XX:MaxPermSize的上限）。而J9和JRockit只要没有触碰到进程可用内存上限就不会溢出。根据hotspot的官方发布路线图也有放弃永久代并逐步改为采用native memory来实现方法区的规划了。

`在1.7的hotspot中，已经把原本放在永久代的字符串常量池移出了。1.8已经把永久代完全移除了`。具体参见[Java永久代去哪儿了](http://www.infoq.com/cn/articles/Java-PERMGEN-Removed)

即时编译(just in time（jit）compilation): java字节码被解释执行，然而它没有本地代码快，为了提高性能，hotspot jvm寻找字节码中的“hot”区域，并把他们编译成本地代码，这个本地代码被保存在code cache在非堆内存中。

## 直接内存

直接内存并不是虚拟机运行时数据区的一部分呢，要不是jvm规范中定义的区域，但是这部分内存也被频繁的使用，而且也可能导致OutOfMemoryError异常出现。

在jdk1.4中新加入了NIO（new input/output）类，引入了一种基于通道（ Channel）和缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过存储在java堆中的DirectByteBuffer对象作为这块内存的引用进行操作，这样能在一些场合显著提高性能，因此避免了java堆和native堆中来回复制数据。

关于DirectByteBuffer的gc

1.  虽然是堆外内存，这部分内存也是有jvm的垃圾回收器负责回收的。Hotspot在gc时会扫描DirectByteBuffer对象是否有引用，如没有则同时也会回收其占用的堆外内存。

2.  DirectByteBuffer对象晋升到old区，这时候就只能等full gc触发了（cms gc的情况下等cms gc），因此在DirectByteBuffer使用较多，存活时间较长的情况下，有可能会导致堆外内存耗光（因为DirectByteBuffer本身对象占用的空间是很小的）

3.  对于2中所说的情况，最好的启动参数中增加`XX:MaxDirectMemorySize=x[m|g],例如X:MaxDirectMemorySize=500m`，这个参数默认的大小是不是64M，也不是-Xmx的大小，是`Runtime.getRuntime.maxMemory()`的大小（可参照`sun.misc.VM#saveAndRemoveProperties`），参数的含义是在DirectByteBuffer分配的堆外内存到达指定大小后，触发Full GC

很多时候我们会看到java进程占用的内存超过-Xmx的大小，原因就是类似DirectByteBuffer、Unsafe、GC、编译、自己写的JNI模块等这些是需要占用堆外内存的。

遇到java进程占用内存超过-Xmx大小的情况，可以尝试强制执行Full GC（强制执行的方位为执行jmap -histo:live）看看，多执行两次，看堆外内存下降的话，很有可能就是DirectByteBuffer造成的，对于这种情况，通常加上上面的启动参数就可解决。

## 参考

[The Java® Virtual Machine Specification]<https://docs.oracle.com/javase/specs/jvms/se8/html/>

[jvm internals]<http://blog.jamesdbloom.com/JVMInternals.html>

[Java virtual machine]<https://en.wikipedia.org/wiki/Java_virtual_machine>

[Java永久代去哪儿了]<http://www.infoq.com/cn/articles/Java-PERMGEN-Removed>

[CMS GC会不会回收Direct ByteBuffer的内存]<http://hellojava.info/?p=56>

[jvm参数]<https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html>

[JDK-4391499]<http://bugs.java.com/bugdatabase/view_bug.do?bug_id=4391499>
