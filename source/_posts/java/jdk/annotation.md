---
layout: post
title: 注解
date: 2014-11-20
categories: java
tags:
    - annotation
    - java
---

####  基础

##### 1.java中内置的三种注解

`@Override`: 重写方法，或者实现接口  

`@Deprecated`:不鼓励使用，很危险或者是有更好的替代。

`@SuppressWarnings`: 压制警告

#####   2.元注解

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它annotation类型作说明。

1.@Target

表示该注解可以用于什么地方，可能的ElementType参数包括：

1.  CONSTRUCTOR：构造器的声明
2.  FIELD：域声明（包括enum实例）
3.  LOCAL_VARIABLE:局部变量声明
4.  METHOD：方法声明
5.  PACKAGE：包声明
6.  PARAMETER：参数声明
7.  TYPE：类，接口（包括注解类型）或enum声明

2.@Retention

表示需要在什么级别保存该注解信息。可选的RetentionPolicy参数包括：

1.  SOURCE：注解将被编译器丢弃
2.  CLASS：注解在class文件中可用，但会被VM丢弃
3.  RUNTIME：VM将在运行期也保留注解，因此可以通过反射机制读取注解的信息。

3.@Documented

将此注解包含在JavaDoc中

4.@Inherited

允许子类继承父类中的注解

Class，Method，Field等类都实现了AnnotatedElement接口

#####   3.注解元素可用的类型

八大基本类型,String,Class,enum,Annotaion,以上的数组,注解允许嵌套

#### Effective java

1.  注解优先于命名模式  

    >   a.既然有了注解,就完全没有理由再使用命名模式了

2.  坚持使用`@Override`注解
3.  用标记接口定义类型: 标记接口和注解一样,只是告诉jvm此类具备标记接口所标示的能力,如Serializable,Cloneable

    >   a.标记接口是没有包含方法的接口,而只是标明一个类实现了具有某种属性的接口,eg Serializable,通过实现这个接口,类表明它的实例可以被写到ObjectOutputStream  
    >   b.标记接口定义的类型是由被标记类的实例实现的,标记注解则没有定义这样的类型,这个类型允许你在`编译时`捕捉,在使用标记注解的情况下要到`运行时`才能捕捉到的错误  
    >   c.标记接口胜过标记注解的另一个优点是,它们可以被更加精确地进行锁定  
    >   d.标记注解可以通过默认的方式添加一个或者多个注解类型元素,给已被使用的注解类型添加更多地信息  

4.  什么时候该使用标记注解,什么时候应该使用标记接口?

    >   a.如果标记是应用到任何程序元素而不是类或者接口,就必须使用注解  
    >   b.如果标记只是应用到类和接口,并且需要编写一个或多个只针对这种标记的方法,就要考虑使用标记接口,可以提供编译时的类型检查.  
    >   c.如果标记只是应用到类和接口,并且我要永远限制这个标记只用于特殊接口的元素.最好将标记定义成该接口的一个子接口(eg Set)  


#### 自定义注解

        //Age
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Age {
            public int value() default 0;
        }

        //Hometown
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Hometown {
            public String value() default "";
        }

        //Name
        @Target({ ElementType.METHOD, ElementType.FIELD })
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Name {
            public String value() default "";
        }

        //User
        public class User {
            @Name(value = "lcj")
            private String name;

            @Hometown(value = "henan")
            private String hometown;

            @Age(value = 22)
            private Integer age;
            ...

            getter/setter
            @Override
            public String toString() {
                return MoreObjects.toStringHelper(User.class).add("name", this.getName()).add("hometown", this.getHometown())
                    .add("age", this.getAge()).toString();
            }
        }

        @Test
        public void getUserAnnotation() {
            User user = new User();
            Field[] fields = User.class.getDeclaredFields();
            for (Field field : fields) {
                field.setAccessible(true);
                if ("name".equals(field.getName())) {
                    Name name = field.getAnnotation(Name.class);
                    user.setName(name.value());
                    System.out.println(name.value());
                }
                if ("hometown".equals(field.getName())) {
                    Hometown hometown = field.getAnnotation(Hometown.class);
                    user.setHometown(hometown.value());
                    System.out.println(hometown.value());
                }
                if ("age".equals(field.getName())) {
                    Age age = field.getAnnotation(Age.class);
                    user.setAge(age.value());
                    System.out.println(age.value());
                } else {
                    continue;
                }
            }
            System.out.println(user);
        }
