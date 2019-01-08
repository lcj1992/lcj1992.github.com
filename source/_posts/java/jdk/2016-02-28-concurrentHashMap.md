---
layout: post
title: concurrentHashMap
categories: java
---





### uml图 

![concurrentHashMap](/images/java/concurrentHashMap.png)

### ConcurrentMap

提供四个原子接口方法

*   putIfAbsent(K key, V value)
*   remove(Object key,Object value)
*   replace(K key,V oldValue,V newValue)
*   replace(K key,V value)

### ConcurrentHashMap

ConcurrentHashMap比hashMap多两个内部类

*   Segment ConcurrentHashMap之所以叫做Concurrent,并且性能较好,跟Segment有很大关系
*   WriteThroughEntry

### 参考

[1]<https://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/>
