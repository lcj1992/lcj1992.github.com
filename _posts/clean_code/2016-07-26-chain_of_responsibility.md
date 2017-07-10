---
layout: post
title: 责任链模式
categories: clean_code
tags: chain_of_responsibility
---

* TOC
{:toc}


## 定义

使多个对象都有机会处理请求，从而避免请求的发送者与接受者之间的耦合关系，将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

![类图](/images/design_pattern/chain_of_responsibility.png)

1. Handler
    * 定义一个处理请求的接口
    * 实现后继链（可选）
2. ConcreteHandler  
    * 处理它所负责的请求
    * 可访问它的后继者
    * 如果可处理该请求，就处理之，否则将该请求转发给它的后继者。

## 使用场景

1. 有多个的对象可以处理一个请求，哪个对象处理该请求运行时刻自动确定。
2. 你想在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。
3. 可处理一个请求的对象集合应被动态指定。
4. 责任链常与组合模式一起使用。

## 优缺点

1. 降低耦合度  该模式使得一个对象无需知道是其他哪一个对象处理其请求。对象仅需知道该请求会被“正确”地处理。接收者和发送者都没有对方明确的信息，且链中的对象不需知道链的结构。

2. 增强了给对象指派职责的灵活性  当在对象中分派职责时，责任链给你更多的灵活性。你可以通过在运行时刻对该链进行动态的增加或修改来增加或改变处理一个请求的那些职责。你可以将这种机制与静态的特例化处理对象的继承机制集合起来使用。

3. 不保证被接受  既然一个请求没有明确的接收者，那么久不能保证它一定会被处理，一个请求可能一直到链的末端都得不到处理，一个请求也可能因该链没有被正确配置而得不到处理。

## 实例

1. logic执行框架（责任链+模板方法） -> 组合模式 + 责任链 + 模板方法

2. order处理订单中心的pushData(责任链+模板方法+组合模式)，详见[组合模式](/2016/07/26/composite)

3. Filter FilterChain

4. HandlerInterceptor

[代码示例](https://github.com/lcj1992/learn/blob/master/java/designPattern/src/main/java/behavioral/chainOfResponsibility/ChainOfResponseTest.java)

## 参考
[wikipedia](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)
