---
layout: post
title: jvm常用配置
date: 2015-12-26
categories: java
tags:
    - jvm
---

*   -Xms2048m:初始堆大小
*   -Xmx2048m:最大堆大小
*   -Xss1m: 一个线程栈的size
*   -XX:NewRatio: 新老代占比，默认是2（新/老=2）
*   -XX:SurvivorRatio=3 新生代 survivor与eden的比例
*   -XX:NewSize=256m:新生代大小
*   -XX:PermSize=256m:永久代大小
*   -server :服务器应用
*   -XX:+DisableExplicitGC:不能显式的GC，System.gc()无效。
*   -XX:+PrintGC（-verbose:gc): 打印垃圾回收
*   -XX:+PrintGCDetails :？
*   -XX:+PrintGCDateStamps:垃圾回收的时候打印时间戳
*   -XX:+PrintTenuringDistribution  查看每次minor GC后新的存活周期的阈值， Desired survivor size 1048576 bytes, new threshold 7 (max 15)
new threshold 7即标识新的存活周期的阈值为7
*   -XX:MaxTenuringThreshold
*   -Xloggc:gc日志
*   -Dcom.sun.management.jmxremote.port=12999:（Java Management Extensions）jConsole远程监控管理端口
*   -Dcom.sun.management.jmxremote.ssl=false:不开启ssl认证
*   -Dcom.sun.management.jmxremote.authenticate=false: 不输入用户名，密码
*   -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8788 开启远程调试，监听的端口号为8788

[jvm参数]<https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html>

[JVM系列三:JVM参数设置、分析]<http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html>
