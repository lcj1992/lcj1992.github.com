---
layout: post
title: 模版方法
date: 2016-07-26
categories: clean_code
tags:
    - template
---




## 定义

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。TemplateMethod使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

类图

![模板方法](/images/design_pattern/template.jpg)

1. AbstractClass
    * 定义抽象的原语操作，具体的子类将重定义他们以实现一个算法的各步骤
    * 实现一个模板方法，定义一个算法的骨架，该模板方法不仅调用原语操作，也调用定义在AbstractClass或其他对象中的操作
2. ConcreteClass
    * 实现原语操作以完成算法中与特定子类相关的步骤

## 优缺点

1. 模板方式是一种代码复用的基本技术，它们在类库中尤为重要，它们提取了类库中的公共行为
2. 模板方法导致一种反向的控制结构，这种结构有时候被称为“好莱坞法则”，即“别找我们，我们找你”，这指的是一个父类调用一个子类操作，而不是相反。

## 相关模式

1. 模板方法使用继承来改变算法的一部分，策略模式使用委托来改变整个算法。

## 实例

太多了，好多的一个抽象类，AbstractCollection、AbstractExecutorService

## 参考

[wikipedia](https://en.wikipedia.org/wiki/Template_method_pattern)
