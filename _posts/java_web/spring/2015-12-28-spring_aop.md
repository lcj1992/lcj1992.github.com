---
layout: post
title: spring aop
categories: java_web
tags: spring proxy aop
---

* [动态代理](#dynamic_proxy)
* [静态代理](#static_proxy)
* [aop](#aop)

要讲spring的aop，就不能不讲代理。

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

在使用@Aspectj自动代理时，为了强制使用cglib，需要设置<aop:aspectj-autoproxy>的属性‘proxy-target-class’为true。

    <aop:aspectj-autoproxy proxy-target-class="true"/>

使用proxy-target-class=true 在`<tx:annotation-driven/>`,`<aop:aspectj-autoproxy/>`,`<aop:config/>任一，会使cglib代理在三者都生效。

### 静态代理 {#static_proxy}

aspectj

### aop

#### aop术语

* 通知(Advice): 定义了切面是什么以及何时使用，除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题
    * 前置通知(Before): 在目标方法被调用之前调用通知功能,下类似
    * 后置通知(After):
    * 返回通知(After-returing)
    * 异常通知(After-throwing)
    * 环绕通知(Around)
* 连接点(JoinPoint) :在应用执行过程中能够插入切面的一个点
* 切点(PointCut): 切点的定义会匹配通知所要织入的一个或多个连接点
* 切面(Aspect): 切面是通知和切点的结合，通知和切点共同定义了切面的全部内容--它是什么，在何时和何处完成其功能
* 引入(Introduction): 引入允许我们向现有的类添加新方法或属性
* 织入(Weaving): 织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点织入到目标对象。在目标对象的生命周期里有多个点可以进行织入：
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
