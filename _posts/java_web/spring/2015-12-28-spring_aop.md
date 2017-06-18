---
layout: post
title: spring aop
categories: java_web
tags: spring proxy aop
---

* TOC
{:toc}

要讲spring的aop，就不能不讲代理。

### 核心类图

![动态代理类图](/images/java_web/spring_aop_proxy.png)

### 动态代理 {#dynamic_proxy}

spring Aop使用`jdk动态代理`或者`cglib代理`来为实例创建代理。(AopProxy接口的两个实现类分别是`JdkDynamicAopProxy`和`CglibAopProxy`)

如果目标对象实现了至少一个接口，那么jdk动态代理将会被使用。所有被这个目标类实现的接口都将被代理。如果目标对象没有实现任何的接口，那么cglib代理将被创建。

如果你想强制使用cglib代理（例如，想代理目标对象中所有的方法，而不仅仅是实现的接口的方法），你可以做这些，但是有一些问题需要考虑：

final 方法不能被advised，因为他们不能被重写
从3.20版本，cglib被包含org.srpingframework的spring-croe jar包中
为了强制使用cglib代理，将<aop:config>中proxy-target-class设为true

    <aop:config proxy-target-class="true">
        <!-- other beans defined here... -->
    </aop:config>

在使用@Aspectj自动代理时，为了强制使用cglib，需要设置<aop:aspectj-autoproxy>的属性`proxy-target-class`为true。

    <aop:aspectj-autoproxy proxy-target-class="true"/>

使用proxy-target-class=true 在`<tx:annotation-driven/>`,`<aop:aspectj-autoproxy/>`,`<aop:config/>`任一，会使cglib代理在三者都生效。

### 静态代理 {#static_proxy}

aspectj

### aop

#### aop术语

* ***Advice***: `action taken by an aspect at a particular join point.` Different types of advice include "around," "before" and "after" advice. (Advice types are discussed below.)` Many AOP frameworks, including Spring, model an advice as an interceptor, maintaining a chain of interceptors around the join point.` 切面在某一连接点采取的动作
    * Before: 在目标方法被调用之前调用通知功能,下类似
    * After:
    * After-returing
    * After-throwing
    * Around
* ***JoinPoint*** : `a point during the execution of a program`, such as the execution of a method or the handling of an exception. `In Spring AOP, a join point always represents a method execution。`
* ***PointCut***: `a predicate that matches join points.` Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.
* ***Aspect***: a modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented using regular classes (the schema-based approach) or regular classes annotated with the @Aspect annotation (the @AspectJ style).切面是通知和切点的结合，通知和切点共同定义了切面的全部内容--它是什么，在何时和何处完成其功能
* ***Introduction***:  declaring additional methods or fields on behalf of a type. Spring AOP allows you to introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an IsModified interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)
* ***Weaving***: linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime. 织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点织入到目标对象。在目标对象的生命周期里有多个点可以进行织入：
    * 编译期:切面在目标类编译时被织入，这种方式需要特殊的编译器，Aspectj的织入编译器就是以这种方法织入切面的
    * 类加载期: 切面在目标类加载到jvm时被织入。这种方式需要特殊的类加载器，它可以在目标类在被引入应用之前增强该目标类的字节码，Aspectj5的加载时织入(load-time weaving,LTW)就支持以这种方式织入切面
    * 运行时: 切面在应用运行的某个时刻被织入，一般情况下，在织入切面时，aop容器会为目标对象动态创建一个代理对象。spring aop就是以这种方式织入切面的。

以实战中例子说明如下：

![aop例子说明](/images/java_web/aop_action.png)

#### 通过切点来选择连接点

正如前面所述，切点用于准确定位应该在什么地方应用切面的通知。通知和切点是切面的最基本元素。因此了解如何编写切点非常重要。

在spring aop中，要使用aspectj的切点表达式语言来定义切点，spring aop仅支持aspectj切点指示器的一个子集。

|aspectj指示器|描述|
|-|-|
|arg()|限制连接点匹配参数为指定类型的执行方法|
|@arg()|限制连接点匹配参数由指定注解标注的执行方法|
|execution()|用于匹配是连接点的执行方法|
|this()|限制连接点匹配aop代理的bean引用为指定类型的类|
|target()|限制连接点匹配目标对象为指定类型的类|
|@target()|限制连接点匹配特定的执行对象，这些对象对应的类要具有指定类型的注解|
|within()|限制连接点匹配指定的类型|
|@within()|限制连接点匹配指定胡姐所标注的类型(当使用spring aop时，方法定义在由指定注解所标注的类里)|
|@annotation()|限制匹配带有指定注解的连接点|

ps:

1. 在spring中尝试使用aspectj其他指示器时，将会抛出IIlegalArgument Exception异常。
2. 只有execution指示器是实际执行匹配的，而其他的指示器都是用来限制匹配的。

举例如下：
![aspectj_expression](/images/java_web/aspectj_expression.jpeg)

[spring aop通知顺序](http://www.uml.org.cn/sjms/201211023.asp)

[《Spring设计思想》AOP实现原理（基于JDK和基于CGLIB）](http://blog.csdn.net/luanlouis/article/details/51155821)

[Aop设计基本原理](http://blog.csdn.net/luanlouis/article/details/51095702)

[Spring AOP源码（十四） 分析ProxyFactoryBean](http://blog.csdn.net/linuu/article/details/50972036)

[spring aop](http://docs.spring.io/spring/docs/5.0.0.RC2/spring-framework-reference/core.html#aop)  
