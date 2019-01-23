---
layout: post
title: 建造者模式
date: 2016-07-26
categories: clean_code
tags:
- builder
---


## 定义

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。（封装一个产品的构造过程，并允许按步骤构造，经常被用来创建组合结构。）

类图：

![builder](/images/design_pattern/builder.png)

## 使用场景

1. 当创建对象的算法应该独立于对象的组成部分以及他们的装配方式时。
2. 当构造过程必须允许被构造的对象有不同的行为时。

## 应用举例

1. lombok的@Builder注解,就是effective java中讲的。
2. spring mvc中的UriComponentsBuilder


## 优缺点

优点：

1. 将一个复杂对象的创建过程封装起来，和工厂模式一样，将创建对象的职责和对象本身所承担的职责分离。
2. 允许对象通过多个步骤来创建，并且可以改变过程（这和只有一个步骤的工厂模式不同）
3. 产品的实现可以被替换，因为客户只看到一个抽象的接口

缺点：

1. 与工厂模式相比，采用生成器模式创建对象的客户，需要具备更多的领域知识。

## 参考

[wikipedia](https://en.wikipedia.org/wiki/Builder_pattern)
