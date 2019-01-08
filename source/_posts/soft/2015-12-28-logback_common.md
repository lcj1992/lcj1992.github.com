---
layout: post
title: logback日常总结
categories: soft java
tags: logback asyncAppender
---

*   打印异常

eg:

    LOGGER.error("aa{}",e); 是错误的,异常被吞掉是多么恐怖的一件事啊
    LOGGER.error("aa{}",var,e);  这样是可以打印出异常信息的(slf4j老版本的是不支持这种打印的)

*   性能

eg:

    logger.debug("Entry number: " + i + " is " +  String.valueOf(entry[i]));
    logger.debug("Entry number: {} is {}", i, entry[i]);
    前者即使在日志关闭的情况下也会进行字符串拼接,

*   异步appender

在十线程并发下，输出200字符的INFO日志，AsyncAppender的吞吐量最高能是FileAppender的3.7倍,这是下边文章说的,我没考证.

但是官网代购的一次故障就是跟这个有关的.

    <appender name="search" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${catalina.base}/logs/search.log</File>
        <encoder>
            <pattern>%d|%X{sys_name}_web|%X{_monitor_cookie}|%m%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${catalina.base}/logs/search.log.%d{yyyy-MM-dd-HH}</fileNamePattern>
            <maxHistory>5</maxHistory>
        </rollingPolicy>
    </appender>


    <!-- 异步输出 -->
    <appender name ="asyncSearch" class= "ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold >100</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>512</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref ="search"/>
    </appender>

*   分布式系统中一个请求可能会经过多个不同的子系统，这是最好生成一个UUID附在请求中，每个子系统在大一日志时都将给UUID放在MDC里，便于以后打日志

*   测试环境中打印sql

    <logger name="com.ibatis" level="debug">
    </logger>
    <logger name="java.sql" level="debug">
    </logger>
    <logger name="org.springframework.jdbc" level="debug">
    </logger>

[1]<http://www.infoq.com/cn/articles/things-of-java-log-performance>
