---
layout: post
title: gc算法
date: 2015-12-26
categories: java
tags: jvm gc
---






### gc流程 

在jvm分代垃圾回收机制中,将应用程序可用的堆空间分为年轻代和年老代,又将年轻代分为Eden区,From区,To区.新建的对象总是在Eden区中被创建,
当Eden区空间已满,就触发一次YoungGC,将还被使用的对象复制到From区这样整个Eden区都是未被使用的空间,可供继续创建对象.当Eden区再次用完,
再触发一次YoungGC,将Eden区和From区还在被使用的对象复制到to区,下一次YoungGC则是将Eden区和To区还被使用的对象复制到From区.
因此经过多次YoungGC,某些对象会在From区和To区多次复制,如果超过某个阈值,对象还未被释放,则将该对象复制到Old区,如果Old区空间也用完了,那么就会触发FullGC.

之前就纳闷为什么要有From和To区,有一个不就ok了么?读了这段话明白了(你得明确新生代采用的是复制算法)

### gc算法 

|算法名称|　说明|
|--|--|
|`标记清扫` |先标记后清扫，容易造成内存碎片|
|`复制` |标记后复制到另一片内存，然后删除本片内存，默认的eden和survivor是8：1，是因为新生代的对象死的快，8在gc到来之前其实能存活的往往还不到1|
|`标记整理`|标记后整理（移动指针指向），然后删除|
|将上述三种组合，分代进行||

### gc收集器 

*  新生代的 采用复制算法

Serial
多线程的Serial （ParNew）
Parallel  Scavenge ，可以jvm自适应调优

*  老年代

Serial Old   Serial的老年代版本，标记整理
Parallel Old  ParNew的老年代版本，标记整理
CMS 标记清除

### 会看gc日志 

        2015-08-05T23:28:42.468+0800: 119660.669: [GC [PSYoungGen: 1042944K->1440K(1044992K)]

        2475082K->1433711K(3142144K), 0.0162150 secs] [Times: user=0.03 sys=0.00, real=0.02 secs]

`119660.669`：自JVM启动到触发这次GC经历的秒数

`GC` ，`Full GC`，`Full GC(System)`：停顿类型，Full的表明发生了STW。

`DefNew`，`Tenured`,`Perm`：GC发生的区域，这里显示的区域名称与使用的GC收集器密切相关。DefNew新生代使用Serial收集器，ParNew新生代使用ParNew收集器，PSYoungGen新生代使用Parallel Scavenge收集器

`1042944K->1440K(1044992K)`：GC前该内存区域已使用容量->GC后该区域使用的容量（该内存区域的总量）

`方括号之外的2475082K->1433711K(3142144K)`:GC前堆使用量->GC后堆使用量（堆总容量）

`0.0162150 secs`：本次GC耗时。

`[Times: user=0.03 sys=0.00, real=0.02 secs]`：用户态消耗的Cpu时间，内核态消耗的Cpu时间，时钟时间（时钟时间还包括各种非运算的时间），Cpu时间是累加各个核的，所以会出现Cpu时间大于时钟时间的。

### 参考
