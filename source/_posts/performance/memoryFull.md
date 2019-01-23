---
layout: post
title: 内存满了
date: 2015-12-17
categories: performance
tags:
    - xmx
    - oomkiller
    - Res
    - Rss
---

### 基础


#### 常用命令

1.  free

2.  jmap -dump:live,file=heap.bin pid

3.  vmstat

[详见](/2015/12/27/memory_command)

#### jvm的内存


[详见](/2015/09/03/jvm_internal)

### 案例一

现象:行程单系统内存经常吃满,jstat查看基本不fgc,老年代和永久代都远不满呢.

处理与原理: 对于java进程,其内存分为堆内存和堆外内存.可以推断是堆外内存占用太多,而公司的tomcat的配置`-XX:+DisableExplicitGC`是不让显式gc的,
在申请ByteBuffer内存时,是调用了System.gc()的,干掉`-XX:+DisableExplicitGC`,然后就fgc了,前后内存情况如下

![内存1](/images/performance/memory_full_1.png)

#### 参考

[程物理内存远大于Xmx的问题分析]<http://lovestblog.cn/blog/2015/08/21/rssxmx/>

[JVM源码分析之堆外内存完全解读]<http://lovestblog.cn/blog/2015/05/12/direct-buffer/>

[JVM源码分析之SystemGC完全解读]<http://lovestblog.cn/blog/2015/05/07/system-gc/>

[Linux与jvm的内存关系分析]<http://tech.meituan.com/linux-jvm-memory.html>
