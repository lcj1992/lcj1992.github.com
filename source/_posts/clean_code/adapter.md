---
layout: post
title: 适配器模式
date: 2016-07-26
categories: clean_code
tags:
    - adapter
---

## 定义

将一个类的接口转换成客户希望的另外一个接口，Adapter模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

对象适配器
![对象适配器](/images/design_pattern/object_adapter.png)

类适配器
![类适配器](/images/design_pattern/class_adapter.png)

1. Target
    * 定义Client使用的与特定领域相关的接口
2. Adaptee
    * 定义一个已经存在的接口，这个接口需要适配
3. Adapter
    * 对Adaptee的接口与Target接口进行适配

<!-- more -->

## 优点
类适配器

1. 用一个具体的Adapter类对Adaptee和Target进行匹配，结果是当我们想要匹配一个雷以及所有它的子类时，类Adapter将不能胜任
2. 使得Adapter可以重定义 Adaptee的部分行为，因为Adapter是Adaptee的一个子类
3. 仅仅引入了一个对象，并不需要额外的指针以间接得到adaptee

对象适配器

1. 允许一个Adapter与多个Adaptee---即Adaptee本身以及它的所有子类（如果有子类的话）同时工作，Adapter也可以一次给所有的Adaptee添加功能
2. 使得重定义Adaptee的行为比较困难。这就需要生成Adaptee的子类并且使得Adapter引用这个子类而不是Adaptee本身

## 使用场景

1. 你想使用给一个已经存在的类，而它的接口不符合你的需求
2. 你想创建一个可以复用的类，该类可以与其他不相关的类或不可预见的类（即那些接口可能不一定兼容的类）协同工作
3. （仅适用于对象Adapter）你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口。对象适配器可以适配它的父类接口。

## 实例

RunnableAdapter

![RunnableAdapter](/images/design_pattern/adapter.png)

## 参考

[wikipedia](https://en.wikipedia.org/wiki/Adapter_pattern)
