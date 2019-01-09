---
layout: post
title: 一个题目来说equals和hashCode
date: 2016-03-12
categories: java
tags: equals hashCode
---


*   [HashMap#put](#put)
*   [HashMap#hash](#hash)
*   [HashMap#indexFor](#indexFor)
*   [HashMap#get](#get)
*   [HashMap#getEntry](#getEntry)


概念原理相关 [参见这里](/2016/03/10/equals_hashcode)

### 下边程序结果是什么? 

    ...
    @Override
    public boolean equals(Object obj) {
        //类型检查
        //...
        return this.name.equals(((People) obj).name) && this.age == ((People) obj).age;
    }

    //@Override
    //public int hashCode() {
    //    return name.hashCode() * 37 + age;
    //}

    public static void main(String[] args) {
        People p1 = new People("foolchild", 24);
        System.out.println(p1.hashCode());

        Map<People, Integer> hashMap = new HashMap<People, Integer>();
        hashMap.put(p1, 1);

        System.out.println(hashMap.get(new People("foolchild", 24)));
    } 
    ...
  
扒扒HashMap的put和get

### HashMap#put()  

    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        // 对put进来的key做hash
        int hash = hash(key);
        // 根据hash值确定在table中的index位置
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            // e.hash值等于hash并且e.key与key逻辑相等或就是同一个引用,满足的话,就是同一个key,value替换为新值
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        //这里并发条件下可能造成死循环,resize,可能导致entry的next指针反转
        addEntry(hash, key, value, i);
        return null;
    }

然后来看看hash(key) 和 indexFor()做了什么?

### HashMap#hash()  

***根据key的hashCode***再算出一个hashCode,在hashMap resize时,该值不变.为什么要这么算,其数学原理暂时不懂.

    final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }


### HashMap#indexFor()  

根据hashCode计算该key在table中的index

对于table length 为2^n(n > 0),h & (length - 1) 其实就是 h % length,不过位运算比模运算效率高(之所以采用table采用2^n,这样可以保证平均分配,`2^n - 1每一位都是1,然后又是与运算..`)

    static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        // 对于table.length,为2^n(n > 0)(这个必须的)
        // h & (length - 1) 其实就是 h % length,不过前者效率高的多,这样可以保证分配平均
        return h & (length-1);
    }
    
最后来看看

### hashMap#get()  

    public V get(Object key) {
        if (key == null)
            return getForNullKey();
        //这里我们讨论的是非空的key,直接看getEntry    
        Entry<K,V> entry = getEntry(key);

        return null == entry ? null : entry.getValue();
    }
    
### HashMap#getEntry 

只有e.hash == hash(key),且e.key == key 或者e.key equals key才返回e.

    final Entry<K,V> getEntry(Object key) {
        if (size == 0) {
            return null;
        }
        
        //hash(key) 是否熟悉,忘了的话看上边
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
    }

### 结论 

最后一行的方法调用链如下:

`hashMap.get(new People("foolchild", 24)) -> HashMap#get() -> HashMap#getEntry() -> HashMap#hash(Object k)`

因为只重写了equals没重写hashCode,`e.hash == hash` && ((k = e.key) == key \|\| (key != null && key.equals(k))) hash值的条件肯定过不去啊,

所以上述例子返回null
