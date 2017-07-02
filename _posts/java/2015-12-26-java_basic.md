---
layout: post
title: java基础
categories:  java
tags: java autoboxing transient thread jdbc shallow_deep_copy final
---
* [初始化顺序](#class_init)
* [自动装箱拆箱](#box_unbox)
* [transient](#transient)
* [shallow_deep_copy](#shallow_deep_copy)
* [object#wait VS thread#sleep](#object_wait_thread_sleep)
* [unSafe](#unsafe)
* [final](#final)
* [SimpleDateFormat](#simple_date_format)
* [lock_tryLock_lockInterruptibly](#lock_tryLock_lockInterruptibly)    

#### 初始化顺序 {#class_init}

1.  静态变量（类变量）、静态初始化块 > 实例变量、 初始化块 > 构造器

2.  父类 > 子类  子类的静态初始化在父类的实例变量初始化之前

3.  静态代码块是在类加载时主动执行的，静态方法是在被调用的时候被动执行的 todo

#### 自动装箱拆箱 {#box_unbox}

装箱：基本类型 -> 引用类型
拆箱：引用类型 -> 基本类型

##### 什么时候

1.  接受一个对象类型的参数，但是我们传入的是基本类型，发生自动装箱；相反情形，发生自动拆箱
2.  赋值时
3.  引用类型不参与计算，比较等操作，会发生自动拆箱。

eg:

    List<Integer> list = new ArrayList<Integer>();
    list.add(1); //auto boxing对应情况1

    Integer i = 10; //auto boxing 对应情况2
    int j = i; // auto unboxing
    if(i > 11){ // auto unboxing
      System.out.println("fuck");
    }


##### 注意事项

1.  避免在循环参与计算时，使用引用类型，会造成不必要的装箱拆箱操作，可能生成大量无用对象，增大GC压力。
2.  缓存的对象：基本类型true,false,\u0000到\u007f的byte或char，-128到127的int或short会被缓存起来。
3.  如果这个对象没有进行初始化或者为Null，在自动拆箱过程中会报空指针异常

eg:

    Integer sum = 0;
    for(int i=1000; i<5000; i++){
        sum+=i;
    }

    // Example 1: == comparison pure primitive – no autoboxing
    int i1 = 1;
    int i2 = 1;
    System.out.println("i1==i2 : " + (i1 == i2)); // true

    // Example 2: equality operator mixing object and primitive
    Integer num1 = 1; // autoboxing
    int num2 = 1;
    System.out.println("num1 == num2 : " + (num1 == num2)); // true

    // Example 3: special case - arises due to autoboxing in Java
    Integer obj1 = 1; // autoboxing will call Integer.valueOf()
    Integer obj2 = 1; // same call to Integer.valueOf() will return same cached Object
    System.out.println("obj1 == obj2 : " + (obj1 == obj2)); // true

    // Example 4: equality operator - pure object comparison
    Integer one = new Integer(1); // no autoboxing
    Integer anotherOne = new Integer(1);
    System.out.println("one == anotherOne : " + (one == anotherOne)); // false

参考

[Java中的自动装箱与拆箱](http://droidyue.com/blog/2015/04/07/autoboxing-and-autounboxing-in-java/index.html)

#### transient

1.  一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2.  transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

3.  被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。


#### Object#wait() &&  Thread.sleep() {#object_wait_thread_sleep}

Object#wait()

1.  Causes the current thread to wait until another thread invokes the Object#notify() method or the Object#notifyAll() method for this object。(本线程wait，直到另一个线程调用notifyAll或者notify方法)
2.  The current thread must own this object's monitor. The thread releases ownership of this monitor and waits until another thread notifies threads waiting on this object's monitor to wake up either through a call to the notify method or the notifyAll method. The thread then waits until it can re-obtain ownership of the monitor and resumes execution.(当前线程必须拥有该object的monitor，调用wait方法后，当前线程释放该object的monitor，直到另一个线程notify该线程，重新获得该object的monitor，重新开始执行)
3.  响应中断

Thread.sleep()

1.  Causes the currently executing thread to sleep (temporarily cease execution) for the specified number of milliseconds, subject to the precision and accuracy of system timers and schedulers. The thread does not lose ownership of any monitors.(暂时停止执行x ms，线程不丢失任何monitor的所有权)
2. 也响应中断

[1]<http://javaconceptoftheday.com/difference-between-wait-and-sleep-methods-in-java/>


#### Unsafe

[1]<http://blog.csdn.net/fenglibing/article/details/17138079>

#### final

不想做改变可能出于两种理由：1.`设计`；2.`效率`

可能用到final的三种情况：1.`数据`;2.`方法`;3.`类`


##### final数据

一经确定就不能再改了：

1.  在编译时已经确定的常量

2.  在运行时初始化(static、非static、构造器)，但在初始化（对象创建）后，不能再被改变了。

3.  空白final： 在构造器中初始化final变量的值，这样就可以做到final域根据对象的不同而有所不同，却又保持其恒定不变的特性。

4.  final参数： 无法再方法中更改参数引用所指的对象。这一特性，主要用于向匿名内部类传递参数。

##### final方法

1.  防止继承类override重写该方法

2.  类中所有的private方法都隐式地指定为是final的了，类似的还有interface的方法隐式都指定为public，想想就知道了。

##### final类

1.  不希望该类有任何改动，不希望有子类

2.  由于final类禁止继承，所以final类中的所有方法都隐式的指定为final的。

ps: static的变量都是放在方法区的。这个我觉得说的很对[27楼说的挺对](http://bbs.csdn.net/topics/370001490#post-371813857)

所有的Serializable的类必须含有serialVersionUID属性，这是出于性能考虑，如果没有serialVersionUID属性，jre会自己计算一个值，这个值的计算很消耗资源。

#### SimpleDateFormat {#simple_date_format}

Calendar calendar 可变且共享的 -> 非线程安全

那么如何正确的使用参见[常用的线程安全处理方法](http://foolchild.cn/2015/11/25/concurrent#how_to_handle)

1. 线程封闭： 每次使用实例化一个simpleDateFormat，局部变量； threadLocal
2. 同步：每次使用时，加锁
3. 使用joadTime （推荐）

#### lock_tryLock_lockInterruptibly

1. 若lock被thread A取得，thread B會進入block狀態，直到取得lock。不响应终端
2. tryLock尝试取获取锁，获取不到返回false。
3. lockInterruptibly类lock，响应中断。当别的线程调用threadx.interrupt() 会使threadx抛出InterruptedException

参考[lock、tryLock和lockInterruptibly的差別](https://pandaforme.ghost.io/java-lock-trylockhe-lockinterruptibly/)
