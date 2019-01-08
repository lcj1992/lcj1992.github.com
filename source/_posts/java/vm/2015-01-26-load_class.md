---
layout: post
title: 类加载机制
categories: java
tags: memory byteCode
---

### classloader

字节码的组织单元是一个class。一个class loader的实现必须能够识别和加载一切符合java class文件格式的东东(.class只是其中一种哟)。类从加载到虚拟机中开始，到卸载出内存为止，它的整个生命周期包括：加载（load）、验证（verification）、准备（preparation）、解析（resolution）、初始化（initialization）、使用（using）和卸载（unloading）。


class loader加载类时按照顺序执行三个基本步骤。

1.   加载（load）：查找并导入类的二进制数据
2.   链接（link）：验证（verification）、准备（preparation）、解析(可选)(resolution)
     1.   验证： 确保导入的类的正确性  



jvm通过bootstrap classloader加载一个初始化类。这个类在调用public static void main方法之前被链接并初始化。这个方法的执行会反过来驱动其他类和接口的加载，链接，初始化。

加载是通过唯一的名字找到代表这个类或者接口的class文件，并以字节数组的方式读入内存的过程。接下来这些字节流会被解析来确认它代表一个class对象，有正确的major（45是jdk1，52是jdk8）和minor版本号。它的直接父类也会被加载。一旦这个过程完成了，一个类或者接口对象就从二进制的格式被创建在jvm中。

链接验证类或者接口并准备它的直接父类或者父接口的过程。链接包含验证verifying，准备preparing和选择性的解析resolving。

验证是确认class或者接口有正确的结构，符合java语言规范，符合jvm规范的过程。例如下边的会被check。

>final的方法或者类没被重写
>方法没有不合理的操作栈  
>变量在使用之前被初始化  
>变量是该类型正确的值  

在验证阶段执行这些意味着不需要在运行时进行这些验证。链接过程中的验证会减慢类的加载过程，但是它避免了执行字节码时的重复验证。准备会为静态存储和jvm用到的任意的数据结构（如方法表）分配内存。静态域会被创建并初始化为默认值，然而这时候没有进行任何其他的初始化，也没有任何的代码被执行。
解析会加载相关的类或者接口来验证符号链接是否正确。如果不在这时候，符号链接的解析会被延迟，直到它被用到。初始化类和接口的初始化就是调用类和接口的初始化方法。
在jvm中有多个不同角色的classloaders。每个classloader代理它的父classloader（父classloader加载子classloader），除了bootstrap classloader它是最顶层的classloader。
![class_load_link_init](http://blog.jamesdbloom.com/images_2013_11_17_17_56/Class_Loading_Linking_Initializing.png)
Bootstrap Classloader是由本地代码实现的，因为它很早的，随着jvm的启动就被初始化了。bootstrap classloader负责加载基本的java apis，如rt.jar。它仅仅加载boot classpath路径下的classes，由于这个路径下的class具有很高的可靠性，所以这里省去了常规的class需要进行的一些验证的步骤。Extension Classloader加载标准java扩展apis，如security extension functions System Classloader是默认的应用classloader，它load应用classes从classpath中User Defined Classloaders可以被用来选择性的加载应用classes。为了一些特殊的目的，我们需要自定义classloader，例如在运行时reload classes，web servers如tomcat实现不同groups下加载的classes之间的隔离
![class_loader_hierarchy](http://blog.jamesdbloom.com/images_2013_11_17_17_56/class_loader_hierarchy.png)

所有已被加载的类都包含有它对classloader的引用。classloader也会包含所有的它加载的类。

## 参考

[java内存分配]<http://www.cnblogs.com/redcreen/archive/2011/05/04/2036387.html>

[jvm internal]<http://blog.jamesdbloom.com/JVMInternals.html>
