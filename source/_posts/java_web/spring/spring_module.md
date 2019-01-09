---
layout: post
title: spring基础
date: 2016-01-07
categories: java_web
tags: spring
---

#### spring架构

![spring架构](/images/java_web/spring.png)

#### spring各jar包

|jar包|说明|
|-|-|
|spring-core|spring的核心工具类，其他组件都要用到这个包,在3.2.0之后spring-aspects中的相关类已经被包含在spring-core中了|
|spring-beans|所有应用都要用到的，它包含访问配置文件，创建和管理bean以及进行控制反转/依赖注入操作相关的所有类，如果应用只需基本的Ioc/DI支持，引入spring-core.jar及spring-beans即可|
|spring-aop|包含应用中使用spring的aop特性时所需要的类，使用基于AOP的spring特性，如声明型事务管理|
|spring-context|为spring核心提供了大量的扩展，可以找到使用spring　ApplicationContext特性所需的全部类，JDNI所需要的全部类，模板引擎集成的类，以及校验相关的类|
|spring-dao|包含spring dao 和spring 事务进行数据访问的所有类,为了声明事务性支持，还需要在自己的应用里包含spring-aop|
|spring-jdbc|包含spring对jdbc数据访问进行封装的所有类|
|spring-web|包含web应用开发时，用到spring框架时所需的核心类，保罗自动载入webApplicationContext特性的类,Filter类和大量工具辅助类|
|spring-webmvc|包含spring mvc框架相关的所有类，包含国际化，标签，主题，视图展现．|
|spring-mock|包含spring一整套mock类来辅助应用的测试，spring测试套件使用了其中大量mock类，这样测试就更加简单了，模拟HttpServletRequest和HttpServletResponse类在web应用单元测试是很方便的|
|spring-support|包含缓存,JMX,邮件，任务计划Scheduling(Timer,Quartz)方面的类|

#### 典型web应用程序应用场景

![典型web架构](/images/java_web/webStruct.jpg)

#### spring模块说明

*   aop 面向切面编程,提供比如日志记录、权限控制、性能统计等通用功能和业务逻辑分离的技术

#### 参考 

[1]<http://www.importnew.com/17474.html>
