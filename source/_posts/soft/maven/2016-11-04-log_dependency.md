---
layout: post
title: 日志各jar依赖
categories: soft
tags: maven slf4j log4j2 logback dependency
---



  * [slf4j + logback](#slf4j_logback)
  * [slf4j + log4j2](#slf4j_log4j2)



### 概述 

很多的冲突都是日志jar引起的，日志实现性能不一，有必要统一下系统中使用的日志.

日志相关的包根据作用分为几类：

1. 日志门面接口包(eg: slf4j-api)

2. 具体的日志实现包(eg: log4j-core)

3. 桥接包:桥接日志接口和日志实现(eg: log4j-slf4j-impl)

4. 适配包:不同日志接口之间的适配(eg: jcl-over-slf4j)

![log_knids](/images/soft/log_kinds.png)

### 实践 

通常我们选择slf4j作为日志接口，log4j2或者logback作为日志实现

为了使我们系统使用到比较高性能的log4j2或者logback，我们需要做三件事

1. 引入slf4j-api包和相关日志实现包(log4j2或者logback)

2. 用各种适配包替换掉其他日志接口包(引入适配包，同时exclude掉原有日志接口包)

3. exclude掉其他日志实现

适配包是怎么来适配的

假设我们日志接口使用slf4j，日志实现使用logback。可是我们的系统中可能引入不同的jar包，会带进来不同的日志包，比如spring使用了commons-logging。为此我们需要exclude掉commons-logging，用jcl-over-slf4j包替换之，这样的话spring还是继续使用commons-logging的api，但是日志实现已经变成slf4j底层的日志实现了（这是logback）。

怎么实现的，适配器模式呗，比对jcl-over-slff4j和commons-logging-api的jar包。log4j-over-slf4j和jul-to-slf4j类似，原理都是一样的，用slf4j的api，适配原有的日志接口。

![jcl-over-slf4j](/images/soft/jcl-over-slf4j.png)
![jcl](/images/soft/jcl.png)


jcl-over-slf4j适配原理

    #// 摘取SLF4JLog的关键代码片段
    public class SLF4JLog implements Log, Serializable {
     
        ...
        // 关键点
        private transient org.slf4j.Logger logger;
     
        SLF4JLog(Logger logger) {
            this.logger = logger;
            this.name = logger.getName();
        }
     
        ...
        public void info(Object message) {
            logger.info(String.valueOf(message));
        }
    }

#### slf4j+logback 

![slf4j+logback](/images/soft/slf4j_logback.png)

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${org.slf4j.version}</version>
    </dependency>
     
    <!--Jakarta Commons Logging redirect to slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>${org.slf4j.version}</version>
        <scope>runtime</scope>
    </dependency>
     
    <!--Java Util Logging redirect to slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jul-to-slf4j</artifactId>
        <version>${org.slf4j.version}</version>
        <scope>runtime</scope>
    </dependency>
      
    <!--Apache log4j redirect to slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>log4j-over-slf4j</artifactId>
        <version>${org.slf4j.version}</version>
        <scope>runtime</scope>
    </dependency>
     
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>${logback.version}</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
        <version>${logback.version}</version>
        <scope>runtime</scope>
    </dependency>


#### slf4j+ log4j2  

![slf4j + log4j2](/images/soft/log4j-1.2-api.png)

    <!--slf4j的配置-->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${org.slf4j.version}</version>
    </dependency>
     
    <!--Jakarta Commons Logging redirect to slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>${org.slf4j.version}</version>
        <scope>runtime</scope>
    </dependency>
     
    <!--Java Util Logging redirect to slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jul-to-slf4j</artifactId>
        <version>${org.slf4j.version}</version>
        <scope>runtime</scope>
    </dependency>
    <!--log4j2的配置-->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.3</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.3</version>
    </dependency>
      
    <!--其实这里使用log4j-over-slf4j也是可以的。无非是多了一层适配-->
    <!--log4j api --> slf4j api -->  slf4j挂的日志实现（这里是log4j2） -->
    <!--log4j api --> log4j2实现 -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-1.2-api</artifactId>
        <version>2.3>
    </dependency>

### 其他好图 

![log4j2+slf4j](/images/soft/whichjar-slf4j-2.x.png)

使用log4j2的api，底层实现使用slf4j的实现（可以挂不同的实现）

![slf4j+other](/images/soft/concrete-bindings.png)

从左向右一次1，2，3，4..

1. 使用slf4j-api，没挂系统实现，不打日志

2. 使用slf4j-api 挂logback日志实现（logback因为是slf4j作者亲生的，所以天然桥接slf4j，不需要任何桥接包）

3. slf4j-api 挂log4j实现，需要桥接包slf4j-log4j12

4. slf4j-api挂java.util.logging日志实现，需要桥接包 slf4j-jdk14

5. 。。。

6. 。。。

7. slf4j 挂log4j2，需要桥接包log4j-slf4j-impl，具体见上 slf4j+log4j2

### 参考 

[slf4j-legacy](http://www.slf4j.org/legacy.html)

[slf4j-manual](http://www.slf4j.org/manual.html)

[log4j2-troubleshooting](https://logging.apache.org/log4j/2.0/faq.html#troubleshooting)

