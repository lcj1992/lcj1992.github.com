---
layout: post
title: 原型模式
date: 2016-07-26
categories: clean_code
tags:
    - prototype
---




## 动机

用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

当创建给定类的实例的过程很昂贵或很复杂时，就使用原型模式，通过一个原型对象克隆出多个一模一样的对象。


## 使用场景

## 优缺点

1. 减少子类的构造，工厂方法经常产生一个与产品类层次平行的工厂类层次，原型模式使得你克隆一个原型而不是请求一个工厂方法去产生一个新的对象。

## 实例

## Object#clone()

java中所有的类都继承自java.lang.Object。Object类提供一个clone()方法，可以将一个java对象复制一份（浅克隆），需要注意的是能够实现克隆方法的java类必须实现一个标识接口Cloneable，表示这个java类支持被复制。如果一个类没有实现这个接口但是调用了clone()方法，Java编译器将抛出一个CloneNotSupportedException异常。

java语言的clone()方法满足：

1. 对任何对象x，都有x.clone() != x，即克隆对象与原型对象不是同一个对象；
2. 对任何对象x，都有x.clone().getClass() == x.getClass()，即克隆对象与原型对象的类型一样；
3. 如果对象x的equals()方法定义恰当，那么x.clone().equals(x)应该成立。

为了获取对象的一份拷贝，我们可以直接利用Object类的clone()方法，具体步骤如下：

1. 在派生类中覆盖基类的clone()方法，并声明为public；
2. 在派生类的clone()方法中，调用super.clone()；
3. 派生类需实现Cloneable接口。

### 深克隆vs浅克隆

浅克隆: 对于基本类型没有问题，独立的一份；对于引用，只是拷贝了引用，共用同一片内存区域（注意String可能在常量池也可能在堆内存中），Object#clone()就是这种。

深克隆: 对值类型的成员变量进行值的复制,对引用类型的成员变量也进行引用对象的复制，两对象独立。可利用串行化来做深复制。

## 参考

[原型模式](https://quanke.gitbooks.io/design-pattern-java/content/%E5%8E%9F%E5%9E%8B%E6%A8%A1%E5%BC%8F-Prototype%20Pattern.html)

[wikipedia](https://en.wikipedia.org/wiki/Prototype_pattern)  

[Difference Between Shallow Copy Vs Deep Copy In Java](http://javaconceptoftheday.com/difference-between-shallow-copy-vs-deep-copy-in-java/)

[deepCopying](https://www.cs.utexas.edu/~scottm/cs307/handouts/deepCopying.htm)

[Object copying](https://en.wikipedia.org/wiki/Object_copying)
