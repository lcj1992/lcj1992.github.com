---
layout: post
title: 装饰器模式
date: 2016-07-26
categories: clean_code
tags:
    - decorator
---

## 定义

动态地给一个对象添加一些额外的职责。就增加功能来说，Decorator模式相比生成子类更加灵活

![装饰器](/images/design_pattern/decorator.jpg)

<!-- more -->

1. Component 定义一个对象接口，可以给这些对象动态的添加指责
2. ConcreteComponent 定义一个对象，可以给这个对象添加一些职责
3. Decorator 维持一个指向Component对象的指针，并定义一个与Component接口一致的接口
4. ConcreteDecorator 向组件添加职责

## 使用场景

1. 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责
2. 处理那些可以撤销的职责
3. 当不能采用生成子类的方法进行扩充时，一种情况是，可能有大量独立的扩展为支持每一种组合将产生大量的子类，使得子类数目呈爆炸性增长。另一种情况可能是因为类定义被隐藏，或类定义不能用于 生成子类。

## 优缺点

1. 比静态继承更灵活 与对象的静态继承（多重继承）相比。Decorator模式提供了更加灵活的向对象添加职责的方式，可以用添加和分离的方法，用装饰在运行时刻增加和删除职责，相比之下，继承机制要求为每个添加的职责创建一个新的子类。这会产生许多新的类，这就使得你可以对一些职责进行混合和搭配
2. 避免在层次结构高层的类有太多的特性。
3. Decorator与它的Component不一样，Decorator是一个透明的包装，如果我们从对象标识的观点出发，一个被装饰了的组件与这个组件是有差别的，因此，使用装饰时不应该依赖对象标识
4. 有许多小对象

## 实例

1. InputStream
2. Collections

## 参考

[wikipedia](https://en.wikipedia.org/wiki/Decorator_pattern)
