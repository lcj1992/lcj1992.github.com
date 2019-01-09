---
layout: post
title: 方法
date: 2016-04-09
categories: java
tags: 方法 参数校验 保护性拷贝 方法签名 重载 可变参数 null
---

*  检查参数的有效性

*  必要时进行保护性拷贝

    >   a.我们编写程序应该假设类的客户端会尽其所能来破坏这个类的约束条件,进行保护性的设计.    
    >   b.对于构造器的每个可变参数都要进行保护性拷贝  
    >   c.保护性拷贝实在检查参数的有效性之前进行的,并且有效性检查针对的是拷贝之后的对象(Time-Of-Check/Time-Of-Use攻击)     
    >   d.对于参数类型可以被不可信任方子类化的参数,请不要使用clone方法进行保护性拷贝   
    >   e.访问方法,返回可变内部域的保护性拷贝.    
    >   f.参数的保护性拷贝并不仅仅针对不可变类,每当编写方法或构造器,如果它要允许客户提供的对象进入内部数据接口,则需要考虑下,这个对象是否是可变的,如果是,还要考虑下,你的类是否能够容忍对象进入数据结构之后发生变化,如果不允许,就要进行保护性拷贝.
    
        private final Class Period{
            private final Date start;
            private final Date end;
            
            public Peroid(Date start,Date end){
                // don't do this
                // if(start.compareTo(end) > 0){
                //     throw new IllegalArgumentException(start + "after" + end);
                // this.start = start;
                // this.end = end;
                
                // don't do this  
                //因为Date不是final的,不能保证clone方法一定返回类为java.util.Date的对象,它有可能返回专门出于恶意目的设计的不可信子类,
                //例如,这个子类在new实例时,把实例引用保存在自己私有的静态列表中,并且允许攻击者访问这个列表,这将使得攻击者可以自由的控制所有实例.
                // this.start = start.clone();
                
                // 保护性拷贝 Date是可变参数,需进行保护性拷贝,对应规则b
                this.start = new Date(start.getTime());
                this.end = new Date(end.getTime());
                
                // 在保护性拷贝之后进行参数有效性检查,你可以试想这换换位置,在参数检查通过后,别的线程有改变了你的可变对象,参数检查白搭了.
                if(this.start.compareTo(this.end) > 0){
                    throw new IllegalArgumentException(start + "after" + end);
                }    
            }
                        
            public Date getStart(){
                // return this.date;
                // 返回可变内部域的保护性拷贝
                return new Date(start.getTime());
            }
            ...
        }    
    
*   谨慎设计方法签名

    >a.避免过长的参数列表(目标是4个参数,或者更少)    
    >b.对于参数类型,要优先使用接口而不是类(对客户端限制小,避免不必要的拷贝工作)      
    >c.对于boolean参数,要优先使用两个元素的枚举类型(可读性好,便于代码编写)  
    
*   慎用重载

    >a.对于重载`overload`方法的选择是静态的,而对于被覆盖`override`的方法的选择则是动态的.
    
*   慎用可变参数

*   返回零长度的数组或者集合,而不是null
       
