---
layout: post
title: idea调试技巧
categories: tool
tags: idea debug
---

*   [Condition Breakpoint](#condition) 
*   [Exception Breakpoint](#exception) 
*   [Watch Point](#watch) 
*   [evaluation](#evaluation) 
*   [change Variable values](#change) 
*   [Environment Variables](#env) 
*   [drop frame](#dropFrame) 
*   [step over,step into,step out,run to cursor](#general) 
*   [multi threads](#multiThread) 

<http://javapapers.com/core-java/top-10-java-debugging-tips-with-eclipse>

这是个eclipse版的，此人.. 知道大名鼎鼎的core java么.对照着来个idea版的

#### Condition Breakpoint {#condition}

<https://www.jetbrains.com/idea/help/creating-field-watchpoints.html>

ctrl+shift+f8 打开Breakpoints

Condition 设置断点打开的条件，当条件满足时，断点才生效，才中断。

#### Exception Breakpoint {#exception}

<https://www.jetbrains.com/idea/help/creating-exception-breakpoints.html>

ctrl+shift+f8 打开Breakpoints

添加java Exception breakpoint，当发生指定异常时，中断。

#### Watch Point [filed] {#watch}

<https://www.jetbrains.com/idea/help/creating-field-watchpoints.html)、[method](https://www.jetbrains.com/idea/help/creating-method-breakpoints.html>

ctrl+shift+f8 打开Breakpoints

添加 java field watchpoints，这样当执行到域的赋值操作时会中断

同样的还有java method breakpoint，会在进入方法的第一步和出方法的最后一步中断

#### Evaluation(Display or inspect or watch) {#evaluation}

alt + f8 运行时执行表达式判定值，

#### change Variable values {#change}

f2 运行时改变值，继续走，对于多if，简直赞。

#### stop in main

找不到对应的。

#### Environment Variables {#env}

run->Edit configurations->environment variables  设置环境变量

然后System.getEnv()

#### drop frame {#dropFrame}
扔掉帧,

每个方法维护一个栈帧

如果你不小心从main方法中跳到了f1方法,你就可以使用drop  frame来回到main，对于跑一次需要很长时间的大项目，这个功能简直赞。

#### step filter
//TODO

#### step over,step into,step out,run to cursor {#general}

step over F8

step into F7（不会进入官方类库的方法）

step out shift + F8

force step into alt + shift + F7

run to cursor F9

####  多线程debug {#multiThread} 

idea 默认run的级别是all 改为Thread就可以debug多线程了

run - view Breakpoints - suspend 将All改为Thread就可以了

###  参考

[1]<http://javapapers.com/core-java/top-10-java-debugging-tips-with-eclipse>