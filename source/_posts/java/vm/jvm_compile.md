---
layout: post
title: 如何得到jvm的本地指令
date: 2016-08-06
categories: java
tags: assembly jit jvm
---

### 安装汇编插件

1.  下载汇编插件：https://kenai.com/projects/base-hsdis/downloads
2.  把hsdis-*.so放到$JAVA_HOME/jre/lib/amd64/server下（也有可能在$JAVA_HOME/jre/lib/server目录下）
3.  javac xx.java
4.  java -XX:CompileThreshold=1 -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -XX:CompileCommand="compileonly classname methodname" classname

其中

-XX:CompileThreshold=1是执行一次就触发jit编译（不知道我这怎么不好使）

-XX:+UnlockDiagnosticVMOptions 解锁诊断jvm选项的一些指令，比如如果这个不打开的话，printAssembly是不好使的。

-XX:+PrintAssembly 打印汇编指令

-XX:CompleCommand="compleonly VolatileBarrierExample readAndWrite VolatileBarrierExample" 只汇编这个方法VolatileBarrierExample#readAndWrite这个方法。
更多请参照[tool reference](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)

eg：

    public class VolatileBarrierExample {
        long a;
        volatile long v1=1;

        void readAndWrite(){
            v1 = 32;
        }

        public static void main(String[] args){
            final VolatileBarrierExample ex=new VolatileBarrierExample();
            for(int i=0;i<10000000;i++)
                ex.readAndWrite();
        }
    }

然后按3，4步所说再编译，执行，(我的本是64位的)可以得到如下（截取的是一小段）

    # {method} 'readAndWrite' '()V' in 'VolatileBarrierExample'
    #           [sp+0x20]  (sp of caller)
    0x00007fb59d061bc0: mov    0x8(%rsi),%r10d
    0x00007fb59d061bc4: cmp    %r10,%rax
    0x00007fb59d061bc7: jne    0x00007fb59d037960  ;   {runtime_call}
    0x00007fb59d061bcd: xchg   %ax,%ax
    [Verified Entry Point]
    0x00007fb59d061bd0: sub    $0x18,%rsp
    0x00007fb59d061bd7: mov    %rbp,0x10(%rsp)
    0x00007fb59d061bdc: movq   $0x20,0x18(%rsi)
    0x00007fb59d061be4: lock addl $0x0,(%rsp)     ;*putfield v1
                                                  ; - VolatileBarrierExample::readAndWrite@4 (line 6)
    0x00007fb59d061be9: add    $0x10,%rsp
    0x00007fb59d061bed: pop    %rbp
    0x00007fb59d061bee: test   %eax,0x9c6440c(%rip)        # 0x00007fb5a6cc6000
                                                  ;   {poll_return}

v1声不声明volatile，会有lock addl $0x0,(%rsp)这句的差别。

![volatile_diff](/images/java/volatile_diff.png)
在赋值之后(movq $0x20,0x18(%rsi))多执行了一句`lock addl $0x0,(%rsp)`操作，这个操作相当于一个内存屏障（memory barrier或者memory fence），重排序时，不能把后面的指令重排序到内存屏障之前的位置。`lock addl $0x0,(%rsp)`的作用是使得本cpu的cache写入了内存，该写入动作也会引起别的cpu或者别的内核无效化其cache。所以通过这一空操作，可以让volatile变量的修改对其他cpu立即可见。

todo看懂上边汇编

从AMD Opteron处理器开始，x86架构扩展32位寄存器到64位寄存器。其中R前缀代表的是64位的寄存器（RAX,RBX,RCX,RDX,RSI,RDI,RBP,RSP,RFLAGS,RIP）,同时引入了八个通用寄存器(R8-R15)。

关于64位下寄存器的相关知识可参照[x86#64bit](https://en.wikipedia.org/wiki/X86#64-bit)

### 参考 

[x86]<https://en.wikipedia.org/wiki/X86>

[tool reference]<https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html>

[CPU中的主要寄存器]<http://share.onlinesjtu.com/mod/tab/view.php?id=233>
