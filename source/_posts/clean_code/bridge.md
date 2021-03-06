---
layout: post
title: 桥接器模式
date: 2016-07-26
categories: clean_code
tags:
    - bridge
---

## 定义

将抽象部分与它的实现部分分离，使它们都可以独立地变化。

![桥接器](/images/design_pattern/bridge_uml.png)

<!-- more -->
1. Abstraction
    * 定义抽象类的接口。
    * 维护一个指向Implementor类型对象的指针。
2. RefinedAbstraction
    * 扩充由Abstraction定义的接口。
3. Implementor
    * 定义实现类的接口，该接口不一定要与Abstraction的接口完全一致;事实上这两个接口可以完全不同。一般来讲，Implementor接口仅提供基本操作，而Abstraction则定义了基于这些基本操作的较高层次的操作。
4. ConcreteImplementor
    * 实现Implementor接口并定义它的具体实现。

## 使用场景

你不希望在抽象和它的实现部分之间有一个固定的绑定关系。例如这种情况可能是因为，在程序运行时刻实现部分应可以被选择或者切换。

##  优缺点
1. 分离接口及其实现部分
2. 可扩展性
3. 实现细节对客户透明

### 参考

[桥接模式](http://alicharles.com/article/jdk-bridge-pattern/)

[wikipedia](https://en.wikipedia.org/wiki/Bridge_pattern)
