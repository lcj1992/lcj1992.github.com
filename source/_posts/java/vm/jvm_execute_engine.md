---
layout: post
title: jvm执行引擎
date: 2015-01-06
categories: java
tags:
    - jvm
    - byte_code
    - operand_stack
    - local_var_table
    - class_file
---

### class文件格式

一个编译后的class文件格式如下（javap -verbose xx.class）：

    ClassFile {
        u4          magic;
        u2          minor_version;
        u2          major_version;
        u2          constant_pool_count;
        cp_info     contant_pool[constant_pool_count – 1];
        u2          access_flags;
        u2          this_class;
        u2          super_class;
        u2          interfaces_count;
        u2          interfaces[interfaces_count];
        u2          fields_count;
        field_info      fields[fields_count];
        u2          methods_count;
        method_info     methods[methods_count];
        u2          attributes_count;
        attribute_info  attributes[attributes_count];
    }

|字段|说明|
|-|-|
|magic  |魔数CAFEBABE
|minor_version major_version| 这个class编译的版本|
|constant_pool  |相当于一个符号表
|access_flags   |提供这个类的一系列的标识
|this_class |类名索引
|super_class|   父类名的索引
|interfaces |所有实现了的接口的符号链接的索引
|fields |...|
|methods|   ...|
|attributes|    ...|

#### 运行时常量池

jvm维护一个常量池，一个运行时的数据结构，它和符号表很类似，但是包含更多的数据。字节码需要数据，但是数据直接以字节码存储又太大，于是它会被存放在常量池中，字节码仅仅是持有对常量池的引用。

常量池中包含的几种类型：

>1.Integer               4字节的常量    
>2.Long                  8字节的常量    
>3.Float                 4字节的常量    
>4.Double                8字节的常量    
>5.String                指向一个utf8类型的条目，那里边才是String真实的bytes    
>6.utf8                  一段utf8格式的字节流    
>7.Class                 指向一个utf8格式的条目，那里是class的名字（jvm格式，这会在动态链接的过程中用到）    
>8.NameAndType           冒号分割的一对值，都指向常量池中utf8类型的一个条目，冒号前是方法名或者域名，对于域来说，冒号后是全类名，对于方法来说，冒号后是类名加参数    
>9.Fieldref，Methodref  逗号分割的一对之，逗号前指向的常量池中的Class类型的一个条目，逗号后指向的常量池中NameAndType类型的一个条目。    

例如下边的代码

`Object foo = new Object();`
将会被编译成如下的字节码

    0:  new #2          // Class java/lang/Object
    1:  dup
    2:  invokespecial #3    // Method java/ lang/Object "<init>"( ) V

new 操作符后跟#2,#2代表的是常量池的索引，第二条记录，第二条记录是一个类的引用，这个引用又指向常量池中另一条记录，是一个utf8的字符串，值为// Class java/lang/Object。这个符号链接会被用来找类java.lang.Object。new操作符会创建一个类的实例并初始化它的值。一个新的类实例的引用会被添加到操作栈上。dup指令创建一个栈顶元素的copy，并将其push到栈顶。最后一个类的实例化方法会被调用invokespecial，这个操作数栈也持有常量池的引用。初始化方法会消费（pop）栈顶元素的值，并将其作为参数传给初始化方法（这里会新建一个栈帧）。最后，栈顶会有一个新建的并初始化好的对象。

#### 方法

>a.签名和访问标志     
>b.字节码      
>c.行数表：这个为debugger提供信息,将**行号**和**字节码**顺序照应，如上边java文件的第六行对应于与字节码0 ,第七行对应于字节码8     
>d.局部变量表：列举了这个栈帧里所有的局部变量，例子中的局部变量只有this引用      
>e.异常表

##### 局部变量表

1.  start是局部变量声明的位置(第几条字节码)；  

2.  length是从起始位置作用的长度；从0开始，长度为9,也就是说this作用的范围是字节码0到字节码8  

3.  slot是插槽位置，除了long和double占两个插槽，其他都占一个；  

4.  Name是变量名；  

5.  Signature是变量类型。


##### 异常表

异常表存储每个异常handler的信息，诸如：

1.  开始点  

2.  结束点  

3.  程序计数器偏移for handler code  

4.  常量池中被捕获的异常类的索引  

如果一个方法中定义了try-catch和try-finally异常handler，然后一个`异常表`就会被建立。异常表包含每个异常处理handler的信息，包括`作用的范围`，将要`被处理的异常的类型`和`异常处理的具体字节码的位置`。当一个异常被抛出时，jvm会在当前方法中寻找一个符合的hadnler，如果在当前方法中没有找到，方法会被终止，并将当前栈帧pop出栈，异常会被抛出到调用的方法中（一个新的栈帧）。如果所有的栈帧都被pop出栈，仍然没有找到异常handler，当前线程会被终止。如果在最后一个非守护线程中（例如如果这个线程是主线程），异常被抛出，那么会造成jvm自己终止terminate。finally异常handlers匹配所有类型的异常，所有总是执行，无论何时异常被抛出。这个例子中，没有异常被抛出，finally块在一个方法的最后仍然会被执行，方法会在return之前，执行finally块的代码。

### 指令

#### 1.  指令类型

存储指令 （例如：aload_0, istore）   
算术与逻辑指令 （例如: ladd, fcmpl）   
类型转换指令 （例如：i2b, d2i）  
对象创建与操作指令 （例如：new, putfield）  
堆栈操作指令 （例如：swap, dup2）  
控制转移指令 （例如：ifeq, goto）  
方法调用与返回指令 （例如：invokespecial, areturn)  

#### 2.前/后缀 操作数类型  

i 整数 l 长整数 s 短整数 b 字节 c 字符 f 单精度浮点数 d 双精度浮点数 z 布尔值 a 引用

### 一个例子

![程序与其字节码](/images/java/jvm_example.jpg)

*   栈的深度为3，局部变量为3，方法参数为0(非静态方法，默认有个参数this)。

        stack=3, locals=3, arg_size=0
*  将常量池中#index为5的string（含有#index指向utf8类型的常量，那才是真的"ab"的bytes）压入操作数栈，然后pop出栈保存在局部变量0中,`栈深为0,局部变量表size为1`.

         0: ldc             #5    // String ab
         2: astore_0    
*  同样，将常量池中#index为6的string保存在局部变量1中，`栈深为0,局部变量表size为2`。

         3: ldc             #6    // Sting b
         5: astore_1
*  新建一个栈帧，在堆中实例化一个StringBuilder对象，并将其引用压入操作数栈中。`栈深为1`

         6: new             #7    // class java/lang/StringBuilder
*  复制栈顶元素，并压入操作数栈,此时`栈深度为2`

         9: dup
*  将栈顶元素（一个StringBuilder的引用）pop出栈，调用其构造方法，创建新的栈帧,进行初始化操作，`栈深为1`

        10: invokesepcial   #8    // Method java/lang/StringBuilder."<init>";()V
*  将常量a压入操作数栈,栈深为2

        13: ldc             #9    // String a
*   栈顶两个元素pop出栈，栈顶的字符串a的引用（方法参数）和StringBuilder对象的引用（相关对象引用）。然后将返回值压入操作数栈，还是那个StringBuilder的引用，只不过现在不是空的了，为“a”;`栈深为1`.

        15: invokevirtual  #10    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
注：invokespecial和invokevirtual等方法调用，`方法参数`和`相关对象的引用`都需要pop出栈，不然jvm如何创建新的栈帧啊。invokestatic就不需要了。

*   将局部变量1压入操作数栈,`栈深为2`

        18: aload_1
*   调用append方法,栈顶元素为StringBuilder引用，”ab“,调用append方法

        19: invokevirtual  #10    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
*   pop栈顶的StringBuilder引用，调用其toString方法，返回String引用

        22: invokevirtual  #11    // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
*   将栈顶的String引用保存在局部变量2中,`局部变量表size为2,栈深为0`

        25: astore_2

*   将类型为Ljava/io/PrintStream的静态域System.out压入栈中

        26: getstatic      #3     // Field java/lang/System.out:Ljava/io/printStream;

*   将局部变量0压入栈中

        29: aload_0

*   将局部变量2压入栈中，此时`栈深为3`!

        30: aload_2
*   pop栈顶两个元素,并比较他们,如果两个对象引用不等,跳到38

        31: if_acmpne      38

*   将常量1入栈

        34: iconst_1

*   然后跳到39

        35: goto           39
*   将常量0入栈

        38: iconst_0
*   pop栈顶两元素PrintStream引用和比较结果

        39: invokevirtual  #4     // Method java/io/PrintStream.println:(Z)V (false)
*   pop栈顶元素(其实栈顶为空,其他类型另说)

        42: return

*   行号表源代码14行,对应于字节码前的序号.

        >a.字节码前的序号为相对于该栈帧的偏移量.
        >b.字节码都是单字节的,为什么字节码前的序号不是连续的?
        >b1.有的指令不需要参数,有的需要多个参数
        >b2.有的参数为单字节的,有的参数是多字节的

        LineNumberTable
           line 14: 0   
           line 15: 3
           line 16: 6

ps：栈和线程，栈帧和方法是对应关系，不要搞混哟。  

#### 参考

[java字节码 wikipedia]<https://en.wikipedia.org/wiki/Java_bytecode>  

[java字节码指令 wikipedia]<https://en.wikipedia.org/wiki/Java_bytecode_instruction_listings>

[java字节码指令 这个感觉更赞]<http://cs.au.dk/~mis/dOvs/jvmspec/ref-Java.html>

[这个是官方的]<https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-7.html>

[class文件]<http://blog.csdn.net/dc_726/article/details/7944154>

[class文件包含的信息]<http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7>
