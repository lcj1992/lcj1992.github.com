---
layout: post
title: java序列化解析
categories: java
tags: serial
---

字节码中包含的内容

1.  类的元数据：类名，序列化id，域的个数，域类型，域名长度等，

2.  父类的这些信息，直到没有父类。

3.  开始解析数据，父类的数据，子类的数据，如果有引用类型，会先解析引用的对象对应的类的元数据，然后其数据。

解析顺序：

![serial](/images/java/serial.png)

<http://www.javaworld.com/article/2072752/the-java-serialization-algorithm-revealed.html>