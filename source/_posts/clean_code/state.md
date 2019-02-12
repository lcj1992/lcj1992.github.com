---
layout: post
title: 状态模式
date: 2016-07-26
categories: clean_code
tags:
    - state
---

## 定义

允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类

![类图](/images/design_pattern/state.jpg)

1. Context
    * 定义客户感兴趣的接口
    * 维护一个ConcreteState子类的实例，这个实例定义当前状态
2. State
    * 定义一个接口以封装与Context的一个特定状态相关的行为
3. ConcreteState
    * 每一个子类实现一个与Context的一个状态相关的行为

## 使用场景

1. 一个对象的行为取决于它的状态，并且它必须在运行时刻根据状态改变它的行为
2. 一个操作中含有庞大的多分支的条件语句，且这些分支依赖于该对象的状态，这个状态通常用一个或多个枚举变量表示。通常，有多个操作包含这一相同的条件结构 。State模式将每一个条件分支放入一个独立的类中，这使得你可以根据对象自身的情况将对象的状态作为一个对象，这一对象可以不依赖于其他对象而独立变化。

## 优缺点

1. 将与特定状态相关的行为局部化
2. 使得状态转化显式化。
3. State对象可被共享


## 参考

[wikipedia](https://en.wikipedia.org/wiki/State_pattern)
