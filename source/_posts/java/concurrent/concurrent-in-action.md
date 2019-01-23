---
layout: post
title: java线程间通信方式
date: 2015-11-25
categories: java
tags:
    - synchronized
    - CountDownLatch
    - CyclicBarrier
    - Exchanger
    - Semaphore
---




本文重在实战，不讲原理。

### synchronized

1. java语言层面的同步，性能在1.6之后很ok了。
2. static方法，非static方法，代码块。
3. 可重入

### ReentrantLock

1. 可重入
2. 公平锁&非公平锁
3. lock、tryLock、lockInterruptibly、newCondition
4. [ReentranctLock](/2016/08/22/ReentrantLock)

### CountDownLatch：某个线程等待其他线程

1. 某个线程等待其他线程执行完任务后，它才执行。
2. 不可重用
3. await无限等待或等待超时，直到count为0.

eg. 多线程获取所有待执行task，获取完成之后执行之。具体见代购proxy MessageReadTask

### CyclicBarrier：一组线程互相等待

1. 一组线程`互相`等待至某个状态,然后执行对应的barrierCommand
2. 可重用（回环栅栏),barrierCommand执行完成之后，会进入nextGeneration，count会被重置成parities

eg. CountDownLatch的例子同样适用。

### Exchanger：一对线程交换数据

1. exchange方法帮一对线程交换数据
2. exchange方法会阻塞调用方线程直至另一方线程参与交换

### Semaphore：信号量

1. 通过计数器控制对共享资源的访问

### LockSupport#park && LockSupport#unpark  

1. LockSupport#park()针对thread进行阻塞处理，可以指定阻塞队列的目标对象
2. LockSupport#unpark()可以指定具体的线程唤醒。
3. 实现机制和object.wait()不同，两者的阻塞队列并不交叉。

### Object#wait && Object#notify

1. obj#wait**当前线程**释放对obj的锁，直到**别的**线程调用该obj#notify或者notifyAll才可能被唤醒。

### Condition#await && Condition#notify

1. 与object#wait和notify不同的是Condition可以有多个谓词条件。

### Thread#join

1. threadx.join(): 当前线程等待threadx执行完成
2. threadx.join(1000): 当前线程等待threadx执行完成或超时，超时的话，不会影响threadx的执行。

### 参考

[java线程阻塞中断和LockSupport的常见问题](http://agapple.iteye.com/blog/970055)

[java并发-ReentrantLock的lock和lockInterruptibly的区别](http://blog.csdn.net/wojiushiwo945you/article/details/42387091)
