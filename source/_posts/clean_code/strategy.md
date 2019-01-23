---
layout: post
title: 策略模式
date: 2016-07-26
categories:
    - clean_code
---




## 定义

定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。本模式使得算法可独 立于使用它的客户而变化。

![strategy](/images/design_pattern/strategy.jpg)

1. Strategy
    * 定义所有支持的算法的公共接口。Context使用这个接口来调用某ConcreteStrategy定义的算法。
2. ConcreteStrategy
    * 以strategy接口实现某具体算法。
3. Context
    * 用一个ConcreteStrategy对象来配置 。
    * 维护一个对strategy对象的引用 。
    * 可定义一个接口来让Strategy访问它的数据。

## 使用场景

1. 需要使用一个算法的不同变体。例如你可能会定义一些反映不同的空间/时间权衡的算法。当这些变体实现为一个算法的类层次时 ,可以使用策略模式。
2. 算法使用客户不应该知道的数据。可使用策略模式以避免暴露复杂的、与算法相关的数据结构。
3. 一个类定义了多种行为,并且这些行为在这个类的操作中以多个条件语句的形式出现。将相关的条件分支移入它们各自的Strategy类中以代替这些条件语句。

## 优缺点

1. 策略可重用，逻辑内聚，
2. 可读性好，易于扩展，易于切换，添加新的策略很easy
3. 消除if-else语句

## 实例

1. 订单中心支付回调处理（首次支付、关单支付、重复支付、重复回调）
2. 门票下单策略的选择（同步下单、异步下单、统一下单）

## 参考

[wikipedia](https://en.wikipedia.org/wiki/Strategy_pattern)
