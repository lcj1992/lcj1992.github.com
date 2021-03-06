---
layout: post
title: 泛型
date: 2014-11-20
categories: java
---

### 概述

1. 声明中具有一个或者多个类型参数的类或者接口,就是泛型类或者接口,简称`泛型`.

2. eg List<E>就读作E的列表.

3. 每个泛型都定义`一组参数化的类型<K,V>`,定义`一个原生态类型Map`.

4. 声明在`类名之后`或者`方法返回值之前` `public class Map <K,V>{...}`

5. 学过模版方法吧，好好想想联系下。

1.几个术语:

|术语|示例|
|-|-|
|参数化的类型  | `List<String>`
| 实际类型参数|`String`|
|  泛型|`List<E>`|
|  形式类型参数|`E`|
|  无限制通配符类型|`List<?>`|
|  原生态类型|`List`|
|  有限制类型参数|`<E extends Number>`|
|递归类型限制|`<T extends Comparable<T>>`|
|  有限制通配符类型|`List<? extends Number>`|
| 泛型方法|`static <E> List<E> asList(E[] a)`|
| 类型令牌|`String.class`|

2.例子:

    class AClass<T,K,V>{}   
    <T>String aMethod(T t){}    
    <T extends Aclass>String aMethod(T t){}
    List<? extends AClass> aMethod(){}

### Effective Java中关于泛型的几个建议

1.  请不要再新代码中使用原生态类型
2.  消除非受检警告
3.  列表优先于数组
4.  优先考虑泛型
5.  优先考虑泛型方法
6.  使用有限制通配符来提升API灵活性

### List,List<?>和List<E>的区别

|类型|编译检查|元素|
|-|-|-|
|原生态类型(List)|编译时不做检查|`List元素可能不一致,这就是坑的所在`|
|无限制通配类型(List<?>)|编译时做检查|List的元素必须是一致的,不能增加除null外的元素|
|泛型(List<E>)|在编译时做类型推导,然后做检查|List的元素必须是一致的,且可以在后边引用使用|

通过以上对比,可以明白`建议一`了,前者在`运行时`的转换才会发现错误.`泛型`可以告诉编译器代码块(类,方法)接受哪些类型的对象,在`编译时`通过`类型推导`,就告知是否插入了错误类型的对象,
泛型可以保证在同一代码块内,类型一致.

### 参考

[泛型的内部原理：类型擦除以及类型擦除带来的问题](http://blog.csdn.net/lonelyroamer/article/details/7868820)

[类型推导](http://blog.csdn.net/zerro99/article/details/6118218)

[无限制的通配符](http://bbs.csdn.net/topics/390349747)

[static方法不能使用类上的泛型](https://bbs.csdn.net/topics/390216551)
