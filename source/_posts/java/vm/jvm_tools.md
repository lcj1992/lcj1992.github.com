---
layout: post
title: jvm性能工具(notes)
date: 2015-12-26
categories: java
tags:
    - jvm
    - tool
    - jps
    - jstat
    - jstack
    - jmap
    - jhar
    - jcmd
---

### all

|名称|	主要作用|
|--|--|
|jps	JVM Process Status Tool |显示指定系统内所有的HotSpot虚拟机进程|
|jstat	JVM Statistics Monitoring Tool |用于收集HotSpot虚拟机各方面的运行数据|
|jinfo	Configuration Info for Java |显示虚拟机配置信息|
|jmap	Memory Map for Java|生成虚拟机的内存存储快照（headpdump文件）|
|jhat	JVM Heap Dump Browser|用于分析heapDump文件，它会建立一个HTTP/HTML服务器，让用户可以在浏览器上查看分析结果|
|jstack	Stack Trace for java |显示虚拟机的线程快照|
|jcmd	|上边的集合，据说oracle以后要推这个，废弃上边的|
|jvisualvm|可视化，也是上边的集合|

### jps

1.  功能与ps命令类似：可以列出正在运行的虚拟机进程，并显示虚拟机执行主类名（main函数所在的类）以及这些进程的本地虚拟机唯一ID（Local Vitural Machine Identifier，LVMID）。

2.  虽然功能比较单一，但它却是使用频率最高的JDK命令行工具，因为其他的JDK工具大多需要输入它查询到的LVMID来确定要监控的哪一个虚拟机进程。

3.  对于本地虚拟机进程来说，LVMID与操作系统的进程id（Process Identifier，PID）是一致的.

4.  jps可以通过RMI协议查询开启了RMI服务的远程虚拟机进程状态

选项
jps   [options]    [hostid]

|选项|	作用|
|--|--|
|-q|只输出LVMID，省略主类的名称|
|-m|输出虚拟机启动时传递给主类main（）方法的参数|
|-l|输出主类的全名，如果进程执行的是jar包，输出jar路径｜
|-v|输出虚拟机进程启动是JVM参数|

eg:

1.  jps -l pid

### jstat

用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据，在没有GUI图形界面，只提供纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的首选工具。

`jstat  [options vmid [interval[s|ms]  [count] ] ]`

对于本地虚拟机进程，VMID与LVMID是一致的。interval 代表查询间隔和次数，如果省略这两个参数，说明只查询一次，

假设需要每250毫秒查询一次进程2764垃圾收集状况，一共查询20次： jstat -gc 2764 250 20

选项option代表用户希望查询的虚拟机信息，主要分为三类：类装载、垃圾收集、运行时编译状况

|选项|	作用|
|--|--|
|-class	|监视类装载，卸载数量，总空间以及类装载所耗费的时间
|-gc|	监视Java堆状况，包括Eden区、两个survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息
|-gccapacity|	监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间
|-gcutil	|监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
|-gccause|	与-gcutil功能一样，但会额外输出导致上一次GC产生的原因
|-gcnew	|监视新生代GC状况
|-gcnewcapacity	|监视内容与-gcnew基本相同，输出主要关注使用到的最小、最大空间
|-gcold	|监视老年代GC状况
|-gcoldcapacity	|监视内容与-gcold基本相同，输出主要关注使用到的最小、最大空间
|-gcmetacapacity|	输出元数据区使用到的最小、最大空间
|-compiler|	输出JIT编译器编译过的方法、耗时等信息
|-printcomipilation	|输出已经被JIT编译的方法

eg:

1.  jstat -gcutil pid 1000\[s\|ms\] 10 每隔一秒(不写单位默认是ms)打印一次各区内存使用比例,打印十次
2.  jstat -class pid 1s 10  每隔一秒打印,打印十次类装载卸载的情况和大小

### jinfo

实时得查看和调整虚拟机各项参数。

1.  使用jps命令的-v参数可以查看虚拟机启动时的显式指定的参数列表，但是如果想知道未被显式指定的参数的系统默认值，使用jinfo -flag 或者java -XX：PrintFlagsFinal

2.  jinfo -syspros 把虚拟机进程的System.getProperties()的内容打印出来

`jinfo [option] pid`

eg：

1.  jinfo -flags xxx
2.  jinfo -sysprops xxx

### jmap
生成堆存储快照（一般成为headdump或者dump文件）。如果不使用jmap命令，要想获取Java堆转储快照，还有一些比较“暴力”的手段：譬如在第二章中用过的-XX:+HeapDumpOnOutOfMemoryError参数，可以让虚拟机在OOM异常出现之后自动生成dump文件，通过-XX:+HeapDumpOnCtrlBreak参数则可以使用【Ctrl】 + 【Break】键，让虚拟机生成dump文件，又或者在linux系统下通过kill -3命令发送进程退出信号“吓唬”一下虚拟机，也能拿到dump文件。

jmap的作用并不仅仅是为了获取dump文件，它还恶意查询finalize执行队列，Java堆和永久代的详细信息，如空间使用率，当前用的是哪种收集器等。

`jmap   [option] vmid`

|选项	|作用|
|-|-|
|-dump|	生成java堆转储快照。格式为：-dump[live,]format=b,file=<filename>,其中live子参数说明只dump出存活的对象
|-finalizerinfo	|显示在F-Queue中等待Fianlizer线程执行finalize方法的对象。
|-heap|显示java堆详细信息，如使用那种回收器，参数配置，分代状况等
|-histo|显示堆中对象统计信息，包括类、实例变量、合计容量
|-permstat|以ClassLoader为统计口径显示永久代内存状态
|-F|当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照。


eg:

1.  jmap -dump:live,format=b,file=hehe.hporf pid dump出pid进程的堆内存,然后可以通过jvisualvm进行分析
2.  jmap -heap pid pid堆的统计信息
3.  jmap -histo pid

### jhat

jhat命令与jmap搭配使用来分析jmap生成的堆转储快照。jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，可以在浏览器中查看。不过实际工作中不会使用它来干活，一耗时且消耗硬件资源二功能简陋，可以通过visualVm 和Eclipse Memory Analyzer。

eg:

1.  jhat  xx.hprof

### jstack

用于生成虚拟机当前时刻的线程快照（一般成为threaddump或者javacore文件）。线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程死锁，死循环，请求外部资源导致的长时间等待等都是导致线程长时间停顿的常见原因。线程出现停顿的时候通过jstack来查看哥哥线程的调用堆栈，就可以知道没有相应的线程到底在后台做些什么事情或者等待着什么资源。

`jstack  [option] vmid`

|选项|	作用|
|-|-|
|-F|	当正常输出的请求不被响应时，强制输出线程堆栈
|-l	|除堆栈外，显示关于锁的附加信息
|-m|	如果调用到本地方法，可以显示c/c++的堆栈

1.  Deadlock – 死锁
2.  Runnable – 执行中
3.  Suspended – 暂停
4.  Blocked – 阻塞
5.  Parked – 停止
6.  Waiting on condition – 等待资源
7.  Waiting on monitor entry – 等待获取监视器
8.  Object.wait() 或 TIMED_WAITING – 对象等待中

eg:

1.  jstack pid
如何找到占用最多cpu的线程[cpu高了](/2015/12/17/cpuHigh)

### jvisualvm

文件-装入-

1.  应用程序快照 *.apps
2.  线程dump *.tdump
3.  堆dump *.hprof , \*.\*
4.  profiler快照 \*.nps,\*.npss


### oql

oql是一个类似sql的查询java堆的查询语言，oql允许过滤／查询java堆的信息。oql是一个基于javascript表达式的语言。

查询语句类似下边格式：

         select <JavaScript expression to select>
         [ from [instanceof] <class name> <identifier>
         [ where <JavaScript boolean expression to filter> ] ]

类名可以是`java.net.URL`具名的类或者`char[]`(or [C)数组。但是className并不能唯一确定一个java类。同样名字的class可以被不同的类加载器加载。`instanceof` 表明可以是子类型，否则的话必须是该类的实例。`from`和`where`都是可选的。

`select`和`where`使用的都是javascript语法规则，java堆对象呗包装成方便的script对象，更接近自然的语法规则，例如obj.field_name或者array[index].

### ps

JDK自带工具如jmap和jstack来dump JVM heap和JVM 线程栈的log来分析问题，可能碰到这个问题：well-known file is not secure

执行jdk的工具时，会先检查执行该命令的用户和/tmp/hsperfdata_$USER/$PID 中的user一致不，不一致的话就会报上述错误。
[问题原址](http://stackoverflow.com/questions/9100149/jstack-well-known-file-is-not-secure?rq=1)

### 参考

[1]<https://visualvm.java.net/oqlhelp.html>

https://blog.csdn.net/maosijunzi/article/details/46049117
