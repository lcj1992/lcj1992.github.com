---
layout: post
title: 工厂（简单工厂、工厂方法、抽象工厂）
date: 2016-07-26
categories: clean_code
tags:
    - factory
---

## 简单工厂（静态工厂）

`创建对象，而不向客户端暴露实例化细节`

### 示例

1. BeanFactory#getBean()： IMeituanSmsService
2. Integer#valueOf()
3. Proxy#newProxyInstance()
4. CommonResponse#buildSuccess()
5. java.sql.DriverManager#getDriver()
6. CreateOrderServiceFactory#getCreateOrderService()
7. AllItemsPriceCheckLogic#createErrorResult()

### 优点

1. 分离创建对象的职责与对象本身所承担的职责，收敛创建对象的职责到工厂类。
2. 可读性强，有名字，根据方法的意图来命名。Lists#newArrayList() Lists#newLinkedList()
3. 对象可复用,(享元模式) Integer#valueOf(), Boolean#valueOf()
4. 创建参数化实例时，代码更简洁 Maps#newHashMap()
5. 可以返回更抽象的类型
6. 低耦合，将创建对象的职责和使用对象的职责中分开，客户端不需要知道具体的产品类，只需要关心所需产品对应的工厂

### 使用场景

1. 实例化时有一定逻辑的，我都倾向于对外提供工厂方法，可以是在本类内提供，也可以单独搞个类，承担创建对象的职责。（extract method,extract class）
2. 当类的创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候。

## 工厂方法

`定义一个用于创建对象的接口，让子类决定实例化哪一个类，工厂方法使一个类的实例化延迟到其子类。`

简单工厂封装了变化，但是当变化发生时，还是需要修改代码

### 类图

![类图](/images/design_pattern/factory.png)

1. IProduct: 定义工厂方法所创建对象的接口
2. ConcreteProduct： 实现Product接口
3. Factory：声明工厂方法，返回一个Product类型对象，也可以定义一个工厂方法的缺省实现；可以调用工厂方法以创建一个Product对象
4. ConcreteFactory：重定义工厂方法，返回一个ConcreteProduct实例

### 优点

1. 简单工厂的优点
2. 扩展性高，扩展遵从开闭原则，新增产品通过添加产品类和对应产品的工厂类来实现，不需要修改原有代码

[代码举例](https://github.com/lcj1992/learn/blob/master/java/designPattern/src/main/java/creational/facotry/FactoryTest.java)

### 使用场景

1. 同简单工厂，当类的创建对象的职责委托给多个帮助子类中的某一个，并且你希望将哪一个帮助子类是代理者这一信息局部化的时候。
2. 当一个类不知道它所必须创建的对象的类的时候
3. 当一个类希望由它的子类来指定它所创建的对象的类的时候

### 实例

1. ILoggerFactory todo 实例化过程分析

## 抽象工厂

![产品族](/images/Design_pattern/abstract_factory.jpg)

定义： 提供一个创建一系列相关或者相互依赖对象的接口，而无需指定它们具体的类。

动机： 产品 -> 产品族, 工厂 -> 抽象工厂

### 类图

![类图](/images/design_pattern/abstract_factory.png)

1. Factory: 声明创建抽象产品对象的操作接口
2. ConcreteFactory: 实现创建具体产品对象的操作，最好实现为singleton的。一个应用中一般每个产品系列只需要一个实例。
3. AbstractProduct: 为一类产品对象声明一个接口
4. ConcreteProduct: 定义一个被具体工厂创建的产品对象；实现AbstractProduct接口

### 优点

1. 有利于产品的一致性
2. 开闭原则：
    * 遵从：添加新的产品组合时，新增对应的工厂类即可。当需要更改一个产品组合时，修改配置即可。
    * 违反：添加新的产品时，所有的工厂类都需要添加对应的创建方法。
    * 定义可扩展的工厂：一种灵活但不太安全的设计是给创建对象的操作增加一个参数，AbstractFactory只需要一个工厂方法和元创建对象的种类的参数。

### 适用场景

1. 产品之间有相互依赖，可以通过抽象工厂来处理产品的依赖，维护其一致性。

## 参考

[工厂方法](https://en.wikipedia.org/wiki/Factory_method_pattern)

[抽象工厂](https://en.wikipedia.org/wiki/Abstract_factory_pattern)

[创建对象与使用对象——谈谈工厂的作用](http://blog.csdn.net/lovelion/article/details/7523392)
