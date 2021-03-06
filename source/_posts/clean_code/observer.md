---
layout: post
title: 观察者模式
date: 2016-07-26
categories: clean_code
tags:
    - observer
    - 观察者
    - 发布订阅
---

## 定义

定义对象间的一种一对多的依赖关系，当一个对象的状态发生变化时，所有依赖于它的对象都得到通知并被自动更新。

## 类图

![类图](/images/design_pattern/observer.png)

1.  Subject 目标、主题
    * 目标知道它的观察者。可以有任意多个观察者观察同一个目标
2.  Observer 观察者
    * 为那些在目标发生改变时需获得通知的对象定义一个更新接口
3.  ConcreteSubject
    * 将有关状态存入各ConcreteObserver对象
    * 当它的状态发生改变时，向它的各个观察者发出通知
3.  ConcreteObserver 具体观察者
    * 维护一个指向ConcreteSubject对象的引用
    * 存储有关状态，这些状态应与目标的状态保持一致
    * 实现Observer的更新接口以使自身状态与目标的状态保持一致

## 推模型 vs 拉模型

推模型：主题对象向观察者推送主题的详细信息，不管观察者是否需要。推送的信息通常是主题对象的全部或部分数据。

拉模型：主题对象在通知观察者的时候，除最小通知外什么也不送出。如果观察者需要更具体的信息，由观察者主动显式地向主题对象询问细节。


## 使用场景

1. 当一个抽象模型有两个方面，其中一个方面依赖于另一方面。将这二者封装在独立的对象中以使它们可以各自独立地改变和复用
2. 当一个对象的改变需要同时改变其它对象，而不知道具体有多少对象有待改变。
3. 当一个对象必须通知其它对象，而它又不能假定其它对象是谁。换言之 ，你不希望这些对象是紧密耦合的。

## 优缺点

1. 松耦合

## 实例

1. java.util.Obsever Observable
2. org.springframework.context.ApplicationListener
mq


## 参考

[wikipedia](https://en.wikipedia.org/wiki/Observer_pattern)

[《JAVA与模式》之观察者模式](http://www.cnblogs.com/java-my-life/archive/2012/05/16/2502279.html)
o

[消息队列设计精要](https://tech.meituan.com/mq-design.html)

[分布式队列编程：模型、实战](https://tech.meituan.com/distributed_queue_based_programming.html)
