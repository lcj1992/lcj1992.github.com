---
layout: post
title: 命令模式
date: 2016-07-26
categories: clean_code
tags:
    - command
---

## 定义

将一个请求封装成一个对象，从而使你可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作。（将请求复用）

![类图](/images/design_pattern/command.jpg)
<!-- more -->

1. command
    * 声明执行操作的接口
2. ConcreteCommand
    * 将一个接收者对象绑定与一个事件
    * 调用接收者相应的操作，以实现Execute
3. Invoker
    * 要求该命令执行这个请求
4. Receiver
    * 知道如何实施与执行一个请求相关的操作，任何类都可能成为一个接收者。
5. Client
    * 创建一个具体命令对象并设定它的接收者

## 动机

有时必须向某对象提交请求，但并不知道关于被请求的操作或请求的接受者的任何信息。Runnable，Method。

（命令模式通过将请求本身变成一个对象来使工具箱对象可向未指定的应用对象提出请求，这个对象可被存储并像其他的对象一样被传递）

## 使用场景

1. 抽象出待执行的动作以参数化某对象，Command模式是回调机制（学过c的都知道[函数指针](/2015/01/25/function_pointer)）的一个面向对象的替代品。

2. 在不同的时刻指定、排列和执行请求，（想象一下ThreadPoolExecutor，排队策略）

3. 支持取消操作，Command的Execute操作可在实施操作前将状态存储起来，在取消操作时将这个状态用来消除该操作的影响。

4. 支持修改日志，这样当系统崩溃时，这些修改可以被重做一遍，在command接口中添加装载操作和存储操作，可以用来记录变动的一个一致的修改日志，从崩溃中回归的过程包括从磁盘中重新读入记录下来的命令并用Execute操作重新执行它们

5. 用构建在原语操作上的高层操作构造一个系统，这样一种结构在支持事务的信息系统很常见，一个事务封装了对数据的一组变动，command模式提供了对事务进行建模的方法，command有一个公共结构，使得你可以用同一种方式调用所有的事务，同时使该模式也易于添加新事务以扩展系统。

## 优缺点

1. 将调用操作的对象与指导如何实现该操作的对象解耦，(对请求、调用或者说操作 进行了抽象，复用)

2. Command是头等的对象，它们可像其他的对象一样被操纵和扩展

3. 你可以将多个命令装配成一个复合命令，复合命令是Composite模式的一个实例

4. 符合开闭原则： 增加新的Command很容易,无需改变已有的类

## 实例

1. 最常见的就是Runnable(command), Executor(invoker)。

2. Invocation、MehondInvocation(command)

3. Method(command)

4. 执行框架：LogicUnitExecutor(invoker)，LogicUnitQueueGroup(command)

[代码示例](https://github.com/lcj1992/learn/blob/master/java/designPattern/src/main/java/behavioral/command/CommandTest.java)

## 参考

[wikipedia]<https://en.wikipedia.org/wiki/Command_pattern>
