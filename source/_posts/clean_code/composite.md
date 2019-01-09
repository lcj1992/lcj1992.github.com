---
layout: post
title: 组合模式
date: 2016-07-26
categories: clean_code
tags: composite
---




## 定义

### 动机

1. 顺序->树形，表示整体/部分（迭代器-> 组合）

2. 统一处理整体和部分

### 定义

将对象组合成树形结构以表示“部分-整体”的层次结构。Composite使得用户对单个对象和组合对象的使用具有一致性。

### 类图

![类图](/images/design_pattern/composite.png)

1. Component
    * 为组合中的对象声明接口
    * 在适当的情况下，实现所有类共有接口的缺省行为
    * 在递归结构中定义一个接口，用于访问一个父部件，并在合适的情况下实现它（可选）
2. Leaf
    * 在组合中表示叶节点对象、叶节点没有子节点
    * 在组合中定义图元对象的行为
3. Composite
    * 定义有子部件的那些部件的行为
    * 存储子部件
    * 在Component接口中实现与子部件有关的操作

## 使用场景

1. 你想表示对象的部分-整体层次结构
2. 你希望用户忽略组合对象与单个对象的不同，用户将统一地使用结构中的所有对象。
3. 可以结合责任链模式，迭代器，装饰器，享元，访问者模式一块使用。

![门票下单流程](/images/design_pattern/order_composite.png)

## 优点

1. 平铺直叙 -> 层次
2. 各叶子节点足够内聚
3. 简化客户端代码，一致的方式使用组合对象和单个对象
4. 开闭，面向接口，方便地装载卸载子对象。
5. 职责单一，叶子节点足够内聚简单，便于测试
6. 可以面向顶层接口，统一做一些事情(eg.是否可降级，是否可异步处理，日志aop等等)

## 参考

[wikipedia](https://en.wikipedia.org/wiki/Composite_pattern)

[Web 服务编排实践](https://www.ibm.com/developerworks/cn/webservices/ws-choreography/)

[用于实现 Web 服务的 SOA 编程模型，第 3 部分: 流程编排和业务状态机](http://www.ibm.com/developerworks/cn/webservices/ws-soa-progmodel3/)
