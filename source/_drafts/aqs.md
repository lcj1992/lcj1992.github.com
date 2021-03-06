---
layout: post
title: AbstractQueuedSynchronizer
date: 2016-08-24
categories: java
tags:
    - concurrent
    - aqs
    - juc
---

## 概述

AbstractQueuedSynchronizer是整个juc中locks类的核心了，包含有一个声明为volatile的变量state保证了内存可见性，通过原子的CAS操作来保证原子性，使用CLH队列来存储waiters。

1.  AQS继承自继承自`AbstractOwnableSynchronizer`（juc locks包中的根本了），含有一个属性`exclusiveOwnerThread`，同步器的属主线程。
2.  AQS内部类`Node`, AQS内部实现两个FIFO队列（同步队列sync queue和条件队列condition queue），Node是队列的节点。
3.  AQS内部类`ConditionObject`继承自`Condition`
4.  `state` 同步器的状态，0为free，>0 重入数。
5.  AQS有独占功能和共享功能，它的所有子类中，要么实现并使用它独占功能的API（ReentrantLock中的Sync），要么共享功能，而不会同时使用两套API。
6.  AQS提供模板方法，其子类实现
    1. 独占 `tryAcquire`、`tryRelease`；
    2. 共享 `tryAcquireShared`、`tryReleaseShared`
7.  支持非阻塞同步尝试，如`tryAcquire`
8.  支持超时时间，应用可以放弃等待`tryAcquireNanos`
9.  可取消`acquireInterruptibly`
10. Node的waitStatus（todo）
    1.  CANCELLED ： waitStatus value to indicate thread has cancelled
    2.  SIGNAL ： waitStatus value to indicate successor's thread needs unparking
    3.  CONDITION ： waitStatus value to indicate thread is waiting on condition
    4.  PROPAGATE ：waitStatus value to indicate the next acquireShared should unconditionally propagate


ps:
tryAcquire 抛出异常而不是声明为抽象方法的原因：

面向两种使用场景，需要给子类定义的方法都是抽象方法了，会导致子类无论如何都需要实现另外一种场景的抽象方法，显然，这对子类来说是不友好的。

## 硬件层面支持的原子操作  

一些看似非原子的原子指令

|操作|x86对应指令|
|-|-|
|测试并设置(Test-and-Set)|[BTS](http://www.felixcloutier.com/x86/BTS.html)|
|获取并增加(Fetch-and-Increment)|[INC](http://www.felixcloutier.com/x86/INC.html)|
|交换(Swap)|[BSWAP](http://www.felixcloutier.com/x86/BSWAP.html)|
|比较并交换(Compare-and-Swap,下文简称CAS)|[CMPXCHG](http://www.felixcloutier.com/x86/CMPXCHG.html)|
|加载链接/条件存储(Load-linked/Store-Conditional,下问称LL/SC)|这个x86系列的处理器好像木有，参见[这里](https://en.wikipedia.org/wiki/Load-link/store-conditional#Implementations)|

详情请垂询[x86的指令集](http://www.felixcloutier.com/x86/)

## 参考

[x86的指令集]<http://www.felixcloutier.com/x86/>

[AQS,老爷子的论文]<http://gee.cs.oswego.edu/dl/papers/aqs.pdf>

[AbstractQueuedSynchronizer的实现分析（上）]<http://www.infoq.com/cn/articles/jdk1.8-abstractqueuedsynchronizer>

[AbstractQueuedSynchronizer的实现分析（下）]<http://www.infoq.com/cn/articles/java8-abstractqueuedsynchronizer>

[]<http://www.cnblogs.com/leesf456/p/5350186.html>
