---
layout: post
title: equal & hashcode
categories: java
tags: java equal hashcode
---





*   [Q&A](#qAnda)
    *   [hashCode为什么能够提升hash-based的Collection和Map的性能](#hashCode_hash_based)



### 概述 
equals() 和 hashCode()是java中Object类中两个基本的方法

equals():   

*   == (引用相等)是比较两个对象的在内存中的地址是否相同
*   equals() (逻辑相等)比较的是两个对象的数据是否想等

hashCode():

对于包含容器类型的程序设计语言来说,基本上都会设计到hashCode.在java中也一样,hashCode方法的主要作用是为了配合基于散列的集合一起正常运行,如hashSet,hashMap等
用于快速定位对象.

###  hashCode和equals关系 

1.  如果你重写了equals方法,你必须重写hashCode方法
2.  关系:
    *   ***equals(逻辑相等)的对象,hashCode()一定相等;其逆否命题当然也是成立的,hashCode()不等的对象,一定不equals***
    *   ***hashCode()相等的对象,不一定equals;***
3.  equals和hashCode应使用相同的域(`same set of "significant" fields`),当然不一定是所有域,你看着办.
4.  当使用基于hash的Collection和Map如 HashSet,LinkedHashSet,HashMap,HashTable,WeakHashMap,确保key的hashCode()是不可变的

### 实现equals 

当实现equals,根据类型不同,采用不同的比较方法

1.  对象域包括collections :equals
2.  类型安全的枚举 : equals或者==
3.  可能为空的对象域 : ==和equals
4.  数据域 : Arrays.equals
5.  除了float和equals的基本类型 : ==
6.  float : 通过Float.floatToIntBits转化为int ,然后使用 ==
6.  double : 通过Double.doubleToLongBits转化为int ,然后使用 ==

很多人认为hashCode是一个对象的唯一标识,其实这是不对的.在java中equals方法必须满足`自反性`,`对称性`,`传递性`,`一致性`,且obj.equals(null)肯定返回false
重写equals的最加实践是:

*   使用==来检查引用的相等性
*   使用instanceof 来检测参数类型
*   cast参数到正确的类型
*   按照上边的比较方法比较你觉得有意义的域的相等性

看下String的equals方法

    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String) anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                            return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }

还可以使用apache的工具类[EqualsBuilder](http://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/builder/EqualsBuilder.html)

### 实现hashCode 

1.  把某个非零常数值保存在int变量result中
2.  对于对象的每一个关键域f(equals方法中考虑的每一个域)
    *   boolean 计算 `f ? 0 :1`
    *   byte,char,short 计算`(int) f`
    *   long    计算`(int)(f^(f\>\>\>32))`
    *   float   计算`Float.floatToIntBits(f)`
    *   double  计算`Double.doubleToLongBits(f)`得到一个long,然后计算`(int)(f^(f\>\>\>32))`;
    *   对象引用 调用它的hashCode方法`f.hashCode()`
    *   数组域,对其中每个元素调用它的hashCode方法.
3.  将上面计算得到的散列码保存到int变量c中,然后执行`result = 37 * result + c`
4.  返回result

ps:
用37这样的素数，可以让各种对象的hashcode值分布散列一些，为了减少下面这种情况的发生，不同对象虽然每个实例变量不同，还是可能计算出来的hashCode值相同
[详见](http://stackoverflow.com/questions/8577582/on-integer-multiplication-overflow-and-information-loss)

还可以使用apache的工具类[HashCodeBuilder](http://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/builder/HashCodeBuilder.html)

### Q&A 


#### hashCode为什么能够提升基于hash的Collection和Map的性能 

以HashMap为例,想象一下如果没有hash,你会如何查找一个map中的某个entry. 遍历HashMap中所有entry,逐一equals?这肯定是可以的,但是...

看下hashMap#getEntry的实现

1.  `hash(key)` 都是移位运算
2.  `indexFor()` hash值与table.length - 1 进行与运算,找到hash(key)所在table的index.
3.  又因为table的length是2的n次幂,保证table数据可以均匀分配.
4.  然后遍历该table\[index\]的entry,最好情况下时间复杂度为O(1),最坏才是O(n)，平均也是O(1)

高下立判

    int hash = (key == null) ? 0 : hash(key);
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        // 这段逻辑是否熟悉,忘了的话看上边,再贴一遍吧
        // e.hash值等于上hash并且e.key与key逻辑相等或就是同一个引用,满足的话,就是同一个entry,返回改entry
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;


### 一道题目 

[看这里](/2016/03/12/equals_hashcode_eg)

### 参考 

[1]<http://crunchify.com/how-to-override-equals-and-hashcode-method-in-java/>

[2]<http://zhangjunhd.blog.51cto.com/113473/71571>

[3]<http://stackoverflow.com/questions/27581/what-issues-should-be-considered-when-overriding-equals-and-hashcode-in-java>

[4]<http://www.javapractices.com/topic/TopicAction.do?Id=29>

[5]<http://www.cnblogs.com/dolphin0520/p/3681042.html>
