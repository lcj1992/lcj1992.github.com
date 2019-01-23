---
layout: post
title: 享元模式
date: 2016-07-26
categories: clean_code
tags:
    - flyweight
---

## 定义

运用共享技术有效地支持大量细粒度的对象

类图
![类图](/images/design_pattern/flyweight.png)

1. flyweight
    * 描述一个接口，通过这个接口flyweight可以接受并作用于外部状态
2. ConcreteFlyweight
    * 实现Flyweight接口，并为内部状态（如果有的话）增加存储空间，ConcreteFlyweight对象必须是可共享的。它所存储的状态必须是内部的，即它必须独立于ConcreteFlyweight对象的场景
3. UnsharedConcreteFlyweight
    * 并非所有的Flyweight子类都需要被共享。Flyweight接口使共享成为可能，但它并不强制共享。在Flyweight对象结构的某些层次，UnsharedConcreteFlyweight对象通常将ConcreteFlyweight对象作为子节点
4. FlyweightFactory
    * 创建并管理flyweight对象
    * 确保合理地共享flyweight。当用户请求一个flyweight时，FlyweightFactory对象提供一个已创建的实例或者创建一个（如果不存在的话）。

## 使用场景

1. 一个应用程序使用了大量的对象
2. 完全由于使用了大量的对象，造成很大的存储开销
3. 对象的大多数状态都可变为外部状态
4. 如果删除对象的外部状态，那么可以用相对较少的共享对象取代很多组独享
5. 应用程序不依赖于对象标识。由于FlyWeight对象可以被共享，对于概念上明显有别的对象，标识测试将返回真值。

## 协作

1. flyweight执行时所需的状态必定是内部的或外部的。内部状态存储于ConcreteFlyweight对象之中；而外部对象则由Client对象存储或计算。当用户调用flyweight对象的操作时，将该状态传递给它。
2. 用户不应该直接对ConcreteFlyweight类进行实例化，而只能从FlyweightFactory对象得到ConcreteFlyweight对象，这可以保证对它们适当地进行共享。

## 优缺点  

1. 减少运行时对象实例的个数，节省内存
2. 将许多“虚拟”独享的状态集中管理
3. 一旦你实现了它，那么单个的逻辑实例将无法拥有独立而不同的行为。

## 实例

Integer

String

## 参考

[wikipedia](https://en.wikipedia.org/wiki/Flyweight_pattern)

[设计模式：享元模式（Flyweight）](http://blog.csdn.net/u013256816/article/details/51009522)

[设计模式读书笔记----享元模式](http://www.cnblogs.com/chenssy/p/3330555.html)
