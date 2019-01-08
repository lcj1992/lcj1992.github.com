---
layout: post
title: 单例模式
categories: clean_code
tags: singleton
---




确保一个类只有一个实例，并提供一个全局访问点。

## 动机

1. 有一些对象其实我们只需要一个，常常被用来管理共享的资源。 如：线程池，数据库连接，Runtime, AppContext, 注册表等
2. 如果对象的初始化非常耗费资源，那么我们就可以在使用它时才进行初始化。eg: 实例化特别消耗资源的，比如从大文件中读取配置等.

## 实现

1. 不对外提供公开构造器，而是提供工厂方法，工厂方法保证线程安全，只会实例化一次
2. 有`lazy`方式和`eager`方式，延迟初始化。

### lazy

1.  instance必须声明为volatile，保证初始化的过程中，不会发生重排序。
2.  保证check and act的原子性。

#### 简单版本 

    public class SimpleVersion {
        private static volatile SimpleVersion instance = null;
        private SimpleVersion() {}
        public static synchronized SimpleVersion getInstance() {
            if (instance == null) {
                instance = new SimpleVersion();
            }
            return instance;
        }
    }

#### 双重检查 

    public class DoubleCheckVersion {
        // 必须声明为volatile, 禁止重排序
        private static volatile DoubleCheckVersion instance;
        private DoubleCheckVersion() {}
        public static DoubleCheckVersion getInstance() {
            // 先判断下，避免读也需要同步，同步是个很重的操作啊
            if (instance == null) {
                // 先check后act，这应该是个原子操作，所以使用synchronized保证原子性
                synchronized (DoubleCheckVersion.class) {
                    if (instance == null) {
                        instance = new DoubleCheckVersion();
                    }
                }
            }
            return instance;
        }
    }

##### 双检查锁的原因

```
instance = new DoubleCheckVersion();
```

这句并不是原子的操作， 事实上在jvm层面大概做了三件事：

1.  给instance分配内存
2.  调用instance的构造函数来初始化成员变量，形成实例
3.  将instance对象指向分配的内存空间（执行完这步，instance才不是null）

如果instance不声明为volatile，则在jvm的即时编译器中可能存在指令重排序的优化，也就是说存在1-3-2的情形发生，这样instance已经不为null，但是并没有初始化成员变量呢，被别的线程引用到那就呵呵了。

#### holder

    public class InitOnDemandHolderVersion {
        private InitOnDemandHolderVersion() {}

        private static class SingletonHolder {
            private static final InitOnDemandHolderVersion instance = new InitOnDemandHolderVersion();
        }
        private static InitOnDemandHolderVersion getInstance() {
            return SingletonHolder.instance;
        }
    }

1.  延迟加载，静态内部类在单例类被加载时并不会立即实例化，而是在在需要实例化时，调用getInstance(),才会装载SingletonHolder，从而完成InitOnDemandHolderVersion的实例化
2.  线程安全，在类进行初始化时，别的线程是无法进入的，jvm层面保证了线程安全。

### eager

1.  instance声明为final的，在构造完成之前引用已经完全确定了

eager的实现方式有enum版本的，还有最简单的直接实例化的,这里就不说了,详见[单例](https://github.com/lcj1992/learn/tree/master/java/designPattern/src/main/java/creational/singleton)。

## 问题

1. 两个类加载器可能有机会各自创建自己的单件实例
2. 反射也是有可能造成两个单件实例的。

```
SimpleVersion singleton = SimpleVersion.getInstance();
MyClassLoader myClassLoader = new MyClassLoader("myClassLoader");
myClassLoader.setClassPath("/Users/lichuangjian/work/learn/java/designPattern/target/classes");
Class singletonClass = myClassLoader.findClass("creational.singleton.lazyInit.SimpleVersion");
System.out.println("singletonClass.getClassLoader() : " + singletonClass.getClassLoader());
System.out.println("Singleton.class==singletonClass : " + (SimpleVersion.class == singletonClass));
System.out.println("Singleton.class.equals(singletonClass) : " + (SimpleVersion.class.equals(singletonClass)));
Constructor constructor1 = SimpleVersion.class.getDeclaredConstructor();
Constructor constructor2 = SimpleVersion.class.getDeclaredConstructor();
Constructor constructor3 = singletonClass.getDeclaredConstructor();
System.out.println("constructor1==constructor2 : " + (constructor1 == constructor2));
System.out.println("constructor1.equals(constructor2) : " + constructor1.equals(constructor2));
System.out.println("constructor1==constructor : " + (constructor1 == constructor3));
System.out.println("constructor1.equals(constructor3) : " + constructor1.equals(constructor3));
constructor1.setAccessible(true);
Object singleton1 = constructor1.newInstance();
constructor2.setAccessible(true);
Object singleton2 = constructor2.newInstance();
constructor3.setAccessible(true);
Object singleton3 = constructor3.newInstance();
System.out.println("singleton : " + singleton);
System.out.println("singleton1 : " + singleton1);
System.out.println("singleton2 : " + singleton2);
System.out.println("singleton3 : " + singleton3);


-----------------------

singletonClass.getClassLoader() : creational.singleton.MyClassLoader@27c170f0
Singleton.class==singletonClass : false
Singleton.class.equals(singletonClass) : false
constructor1==constructor2 : false
constructor1.equals(constructor2) : true
constructor1==constructor : false
constructor1.equals(constructor3) : false
singleton : creational.singleton.lazyInit.SimpleVersion@2626b418
singleton1 : creational.singleton.lazyInit.SimpleVersion@5a07e868
singleton2 : creational.singleton.lazyInit.SimpleVersion@76ed5528
singleton3 : creational.singleton.lazyInit.SimpleVersion@2c7b84de

```

### 参考 

[深入浅出单实例Singleton设计模式]<http://coolshell.cn/articles/265.html>

[wikipedia]<https://en.wikipedia.org/wiki/Singleton_pattern>

[单例模式的八种写法比较]<http://tianweili.github.io/blog/2015/03/02/singleton-pattern/>

[如何正确地写出单例模式]<http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/>
