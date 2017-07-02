---
layout: post
title: 组合模式
categories: clean_code
tags: composite
---

* TOC
{:toc}

## 定义

### 动机

1. 顺序->树形，表示整体/部分（迭代器-> 组合）

2. 统一处理整体和部分

### 定义

允许你将对象组合成树形接口来表现“整体/部分”层次结构，让客户以一致的方式处理个别对象和对象组合。

### 类图

![类图](/images/design_pattern/composite.png)

## 示例

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
