---
layout: post
title: 一些编程决定（题目todo）
categories: clean_code
tags: 阿里巴巴java开发手册  
---

* TOC
{:toc}

>旨在介绍一些编程中正确的决定，并解释清楚这些，与[实现模式](/2017/01/29/implementation-patterns)不同的是，实现模式最大的层次是类，本文意在类间，模块间的。

## 命名

1. 类名取名时，取为对象，英语中一般为er，or，ist等后缀

2. 方法取名时，取为动作，doExecute, createOrder等

3. 减少耦合性，好多都是中间加了一层代理层，比如ioc、桥接、观察者等。

### 分层架构

### 防御式编程

enum和字面量
范型
框架该做的事
接口幂等

作为一个技术负责人的你，你当然有权利通过使用非技术的手段来达到你的目的。比如：你在团队内部明文规定，“XX类只能有一个全局实例，如果某人使用两次以上，那么该人将被处于2000元的罚款！”（呵呵），你当然有权这么做。但是如果你的设计的是东西是一个类库，或是一个需要提供给用户使用的API，恐怕你的这项规定将会失效。因为，你无权要求别人会那么做。所以，这就是为什么，我们希望通过使用技术的手段来达成这样一个目的的原因。

防御式编程，不要相信任何人和服务。别相信你的下游说，我就调你三次，你放心吧，没事的。别信，肯定有鬼，你要做好对自身的保护，也不要相信下游说别人的提供的服务放心地使，哥向你保证五个9的可靠性，没有一个服务能做到100%的可靠的，这是必然的，即使是5个9，也有损失的时候，别相信他，要做好对下游的依赖和熔断；

错误异常是一定会发生的

容错机制

failback

failfast

failsafe

failover

forking

broadcast

## 参考

[重构](https://book.douban.com/subject/4262627/)

[代码整洁之道](https://book.douban.com/subject/4199741/)

[阿里巴巴java开发手册](https://yq.aliyun.com/articles/69327?utm_content=m_10088)

[美团外卖系统架构演进与稳定性的探索](http://www.infoq.com/cn/articles/evolution-and-stability-of-meituan-waimai-architecture)
