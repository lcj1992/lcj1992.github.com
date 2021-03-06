---
layout: post
title: logback
date: 2014-11-20
categories: soft
tags:
    - logback
    - java
---

### 实用版
去除wiki的冗杂，能够快速确定项目日志怎么打的。实用为上

#### 常见的appender
RollFileAppender 滚动策略一般为TimeBasedRollingPolicy

SiftingAppender  根据一个运行参数分割日志文件（MDC put进去）

AsyncAppender  异步的

#### 基本pattern 语法
1.MDC put一些运行时参数，用于打日志用

2.File和filenamePattern ${}引用变量，在logback中定义的property，还有mdc put的。

3.encoder中% 格式化输出

|参数|作用|
|-|-|
|d \| date|	时间|
|t  \| thread|	线程名
|L \| line|	行号
|c \| lo | logger	logger名
|p \| le | level	日志级别
|m \| msg | message	日志内容
|n|	换行符
|X	|mdc put的变量  %X{orderId}

#### logger
1.level 不设置，继承上级级别

2.addticity 不设置，向上级logger传递打印信息。

3.root是根logger

### wiki
    appender logger  filter
    <?xml version="1.0" encoding="UTF-8"?>
    <!--
    1.logback首先会试着查找logback.groovy文件
    2.当没有找到时，继续试着查找logback-test.xml文件
    3.当没有找到时，继续试着查找logback.xml文件
    4.如果仍然没有找到，则使用默认配置（打印到控制台）。
    -->

    <!--根节点-->
    <configuration scan="true" scanPeriod="60 seconds" debug="false">
        <!--
        scan:当此属性为true时，配置文件如果发生改变，将会被重现加载，默认值为true
        scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒，当scan为true时，此属性生效，默认的时间间隔为1分钟
        debug:当此属性为true时,将打印出logback内部日志信息，实时查看logback运行状态，默认值为false
        -->

        <!--子节点-->
        <property name="appName" value="lcj-web"/>
        <!--定义变量，两个属性name 键,value 值,使用${}来使用-->

        <contextName>${appName}</contextName>
        <!--每个logger都关联到logger上下文，默认上下文名称为"default"。但可以使用<contextName>设置成其他名字，用于区分不用应用程序的记录.一旦设置，不能修改-->

        <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/>
        <!--两个属性key表示次timestamp的名字，datePattern 设置将当前时间（解析配置文件的时间）转换成字符串的模式，遵循java.txt.SimpleDateFormat的格式
            例如将解析配置文件的时间作为上下文名称：
                <configuration scan="true" scanPeriod="60 seconds" debug="false">
                    <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/>
                    <contextName>${bySecond}</contextName>
                </configuration>
        -->

        <!--<appender>是<configuration>的子节点，是负责写日志的组件-->
        <!--appender有两个必要的属性，name和class。name指定appender名称，class指定appender的全限定名-->

        <!--RollingFileAppender-->
        <appender name="ROLL" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <!--滚动记录文件，先将日志文件记录到指定文件，当符合某个条件时，将日志记录到其他文件，有一下子节点-->
            <!--encoder:负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流
            目前PatternLayoutEncoder是唯一有用的且默认的encoder，有一个<pattern>节点，用来设置日志的输入格式，
            使用"%"+"转换符"的方式，如果要输出%,则必须用"\"转义"\%"

            -->
            <!--
            triggeringPolicy:告知RollingFileAppender何时激活滚动
            prudent：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空
            -->

            <file>xx.log</file>
            <!--file:被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值-->
            <append>true</append>
            <!--true：追加，false：覆盖-->
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">

                <fileNamePattern>xx.%d{yyy-MM-dd}.log</fileNamePattern>
                <!--必要节点,包含文件名以及%d转换符,%d可以包含一个java.text.SimpleDateFormat指定的时间格式，如果直接是用%d,默认格式是??
                此时RollingFileAppender的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），
                活动文件的名字不会改变；如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。-->

                <maxHistory>30</maxHistory>
                <!--可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件,<maxHistory>是6，假设设置每月滚动，
                则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。-->
            </rollingPolicy>

            <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
                <!--根据固定窗口算法重命名文件的滚动策略，有一下子节点-->
                <minIndex>10</minIndex>
                <!--窗口索引最小值-->
                <maxIndex>12</maxIndex>
                <!--窗口索引最大值,当用户指定的窗口过大时，会自动将窗口设置为12-->
                <fileNamePattern>xx%i.log</fileNamePattern>
                <!--必须包含%i,假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。
                还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip-->
            </rollingPolicy>

            <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
                <maxFileSize>10MB</maxFileSize>
                <!--只有一个节点，活动文件的大小-->
            </triggeringPolicy>
            <prudent>false</prudent>

        </appender>


        <appender name="flightguess" class="ch.qos.logback.classic.sift.SiftingAppender">
            <discriminator>
                <key>site</key>
                <defaultValue>guess</defaultValue>
            </discriminator>
            <sift>
                <appender name="rolling" class="ch.qos.logback.core.rolling.RollingFileAppender">
                    <File>${catalina.base}/logs/${site}.log</File>
                    <encoder>
                        <pattern>%d [%p] [%t] %-17C:%L %X %m%n</pattern>
                    </encoder>
                    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                        <fileNamePattern>
                            ${catalina.base}/logs/${site}.log.%d{yyyy-MM-dd-HH}
                        </fileNamePattern>
                    </rollingPolicy>
                </appender>
            </sift>
        </appender>

        <!--ConsoleAppender-->
        <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
            <!--有encoder和targer两个子节点-->

            <encoder charset="UTF-8">
                <pattern>%d %t %p %-17c{2} %m%n</pattern>
            </encoder>
            <!--对日志进行格式化-->

            <!--fliter-->
            <!--执行一个过滤器会有返回个枚举值，即DENY，NEUTRAL，ACCEPT其中之一。
                返回DENY，日志将立即被抛弃不再经过其他过滤器；
                返回NEUTRAL，有序列表里的下个过滤器过接着处理日志；
                返回ACCEPT，日志会被立即处理，不再经过剩余过滤器。
                过滤器被添加到<Appender> 中，
                为<Appender> 添加一个或多个过滤器后，可以用任意条件对日志进行过滤。
                <Appender> 有多个过滤器时，按照配置顺序执行-->
            <!--levelFilter-->
            <fliter class="ch.qos.logback.classic.filter.LevelFilter">
                <!--
                级别过滤器，根据日志级别进行过滤。如果日志级别等于配置级别，过滤器会根据onMath 和onMismatch接收或拒绝日志。
                有以下子节点：
                    <level>:设置过滤级别
                    <onMatch>:用于配置符合过滤条件的操作
                    <onMismatch>:用于配置不符合过滤条件的操作
                -->
                <level>info</level>
                <onmatch>accept</onmatch>
                <onmismatch>deny</onmismatch>
            </fliter>

            <fliter class="ch.qos.logback.classic.filter.ThresholdFilter">
                <!--临界值过滤器，过滤掉低于指定临界值的日志。
                当日志级别等于或高于临界值时，过滤器返回NEUTRAL；当日志级别低于临界值时，日志会被拒绝
                有节点：
                    <level>:设置过滤级别
                -->
                <level>info</level>
            </fliter>


            <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
                <!--求值过滤器，评估、鉴别日志是否符合指定条件。
                需要额外的两个JAR包，commons-compiler.jar和janino.jar
                有以下子节点：
                    <evaluator>:鉴别器，常用的鉴别器是JaninoEventEvaluato，也是默认的鉴别器.
                    它以任意的java布尔值表达式作为求值条件，求值条件在配置文件解释过成功被动态编译，
                    布尔值表达式返回true就表示符合过滤条件。evaluator有个子标签<expression>，用于配置求值条件。
                    见下：
                    <onMatch>:用于配置符合过滤条件的操作
                    <onMismatch>:用于配置不符合过滤条件的操作
                    <matcher> ：匹配器，尽管可以使用String类的matches()方法进行模式匹配，
                    但会导致每次调用过滤器时都会创建一个新的Pattern对象，为了消除这种开销，可以预定义一个或多个matcher对象，
                    定义后就可以在求值表达式中重复引用。<matcher>是<evaluator>的子标签。
                    <matcher>中包含两个子标签，一个是<name>，用于定义matcher的名字，
                    求值表达式中使用这个名字来引用matcher；另一个是<regex>，用于配置匹配条件。
                -->
                <evaluator> <!-- 默认为 ch.qos.logback.classic.boolex.JaninoEventEvaluator -->
                    <!--<expression>return message.contains("billing");</expression>-->
                    <matcher>
                        <Name>odd</Name>
                        <!-- filter out odd numbered statements -->
                        <regex>statement [13579]</regex>
                    </matcher>

                    <expression>odd.matches(formattedMessage)</expression>
                </evaluator>
                <OnMatch>ACCEPT</OnMatch>
                <OnMismatch>DENY</OnMismatch>
            </filter>
            <target>System.out</target>
            <!--target: 字符串System.out或者System.err,默认是System.out-->
        </appender>

        <!--FileAppender-->
        <appender name="FILE" class="ch.qos.logback.core.FileAppender">
            <!--有file，appender，encoder，prudent四个子节点-->

            <file>xx.log</file>
            <!--被写入的文件名，可以是相对目录，可以是绝对目录，如果上级目录不存在会自动创建，没有默认值-->
            <append>true</append>
            <!--true,日志追加到文件末尾，false，晴空现存文件，默认是true-->
            <encoder>
                <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
            </encoder>
            <prudent>false</prudent>
            <!--true,日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是false-->
        </appender>


        <logger name="com.xxx.guess" level="info">
            <!--用来设置某一个包或者具体的某一类的日志打印级别，以及指定<appender>.<logger>仅有一个name属性，一个可选的level和一个可选的addtivity-->
            <!--
            name:用来之地功能受此约束的某一个包或者具体的某一个类
            level:用来设置打印级别，大小写无关：trace,debug,info,warn,error,all,off还有未设置此属性，那么当前logger将会继承上级的级别
            addtivity:是否向上级logger传递打印信息，默认是true
            logger可以包含零个或者多个<appender-ref>元素，标识这个appender将会添加到这个logger
            -->
            <!--<appender-ref ref="rolling"/>-->
        </logger>

        <root level="info">
            <!--<root>也是logger元素，但是它是根logger,只有一个level属性，因为已经命名为"root"-->
            <!--level:用来设置打印级别，大小写无关：trave,debug,info,warn,error,all和off,不能设置为inherited或者同义词null.默认是debug-->
            <appender-ref ref="flightguess"/>
        </root>
    </configuration>


    encoder
    <!--
                                输出日志的logger名，可有一个整形参数，功能是缩短logger名，设置为0表示只输入logger最右边点符号之后的字符串。 Conversion specifier Logger name Result
                                %logger        mainPackage.sub.sample.Bar mainPackage.sub.sample.Bar
    c {length}                  %logger{0}     mainPackage.sub.sample.Bar Bar
    lo {length }                %logger{5}     mainPackage.sub.sample.Bar s.Bar
    logger {length }            %logger{10}    mainPackage.sub.sample.Bar m.s.s.Bar
                                %logger{15}    mainPackage.sub.sample.Bar m.s.sample.Bar
                                %logger{16}    mainPackage.sub.sample.Bar m.sub.sample.Bar
                                %logger{26}    mainPackage.sub.sample.Bar mainPackage.sub.sample.Bar


    C {length }
    class {length }             输出执行记录请求的调用者的全限定名。参数与上面的一样。尽量避免使用，除非执行速度不造成任何问题。


    contextName
    cn                          输出上下文名称。


                                输出日志的打印日志，模式语法与java.text.SimpleDateFormat 兼容。 Conversion Pattern Result
                                %d 2006-10-20 14:06:49,812
    d {pattern }                %date  2006-10-20 14:06:49,812
    date {pattern }             %date{ISO8601} 2006-10-20 14:06:49,812
                                %date{HH:mm:ss.SSS}    14:06:49.812
                                %date{dd MMM yyyy ;HH:mm:ss.SSS}   20 oct. 2006;14:06:49.812


    F / file                    输出执行记录请求的java源文件名。尽量避免使用，除非执行速度不造成任何问题。


                                输出生成日志的调用者的位置信息，整数选项表示输出信息深度。
                                例如， %caller{2}   输出为：
                                0    [main] DEBUG - logging statement
    caller{depth}               Caller+0   at mainPackage.sub.sample.Bar.sampleMethodName(Bar.java:22)
    caller{depth,               Caller+1   at mainPackage.sub.sample.Bar.createLoggingRequest(Bar.java:17)
    evaluator-1, ... evaluator-n}
                                例如， %caller{3}   输出为：
                    16   [main] DEBUG - logging statement
                                Caller+0   at mainPackage.sub.sample.Bar.sampleMethodName(Bar.java:22)
                                Caller+1   at mainPackage.sub.sample.Bar.createLoggingRequest(Bar.java:17)
                                Caller+2   at mainPackage.ConfigTester.main(ConfigTester.java:38)


    L / line                    输出执行日志请求的行号。尽量避免使用，除非执行速度不造成任何问题。

    m / msg / message           输出应用程序提供的信息。

    M / method                  输出执行日志请求的方法名。尽量避免使用，除非执行速度不造成任何问题。

    n  输出平台无关的分行符“\n”或者“\r\n”。

    p / le / level             输出日志级别。

    r / relative               输出从程序启动到创建日志记录的时间，单位是毫秒

    t / thread                 输出产生日志的线程名。

    replace(p ){r, t}          p 为日志内容，r 是正则表达式，将p 中符合r 的内容替换为t 。例如， "%replace(%msg){'\s', ''}"

    格式修饰符，与转换符共同使用：
    可选的格式修饰符位于“%”和转换符之间。
    第一个可选修饰符是左对齐标志，符号是减号“-”；
    接着是可选的最小宽度修饰符，用十进制数表示。如果字符小于最小宽度，则左填充或右填充，
    默认是左填充（即右对齐），填充符为空格。如果字符大于最小宽度，字符永远不会被截断。
    最大宽度修饰符，符号是点号"."后面加十进制数。如果字符大于最大宽度，则从前面截断。点符号“.”后面加减号“-”在加数字，表示从尾部截断。
    -->


    <!--evaluatorFilter-->
    <!--
    求值表达式作用于当前日志，logback向求值表达式暴露日志的各种字段：

    Name                      Type                Description
    event                  LoggingEvent       与记录请求相关联的原始记录事件，下面所有变量都来自event，
                                              例如，event.getMessage()返回下面"message"相同的字符串

    message                 String            日志的原始消息，例如，设有logger mylogger，"name"的值是"AUB"，
                                              对于 mylogger.info("Hello {}",name); "Hello {}"就是原始消息。

    formatedMessage         String            日志被各式话的消息，例如，设有logger mylogger，"name"的值是"AUB"，
                                              对于 mylogger.info("Hello {}",name); "Hello Aub"就是格式化后的消息。

    logger                  String                 logger 名。

    loggerContext        LoggerContextVO      日志所属的logger上下文。

    level                    int              级别对应的整数值，所以 level > INFO 是正确的表达式。

    timeStamp               long                 创建日志的时间戳。

    marker                  Marker           与日志请求相关联的Marker对象，注意“Marker”有可能为null，所以你要确保它不能是null。

    mdc                      Map             包含创建日志期间的MDC所有值得map。访问方法是： mdc.get("myKey") 。mdc.get()
                                             返回的是Object不是String，要想调用String的方法就要强转，例如，
                                             ((String) mdc.get("k")).contains("val") .MDC可能为null，调用时注意。

    throwable           java.lang.Throwable  如果没有异常与日志关联"throwable" 变量为 null.
                                             不幸的是， "throwable" 不能被序列化。在远程系统上永远为null，
                                             对于与位置无关的表达式请使用下面的变量throwableProxy

    throwableProxy       IThrowableProxy     与日志事件关联的异常代理。如果没有异常与日志事件关联，则变量"throwableProxy"
                                             为 null. 当异常被关联到日志事件时，"throwableProxy" 在远程系统上不会为null
    -->
