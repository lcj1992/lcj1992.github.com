---
layout: post
title: load average
categories: unix
tags: load
---

#### 基本含义

    man uptime
    DESCRIPTION
    ...
    System load averages is the average number of processes that are either in a runnable or uninterruptable  state.   A  process  in  a
    runnable  state  is  either  using  the  CPU  or waiting to use the CPU.  A process in uninterruptable state is waiting for some I/O
    access, eg waiting for disk.  The averages are taken over the three time intervals.  Load averages are not normalized for the number
    of CPUs in a system, so a load average of 1 means a single CPU system is loaded all the time while on a 4 CPU system it means it was
    idle 75% of the time.
翻译：

系统负载均值是一个可运行状态和不可中断状态的进程的平均值。可运行状态包含运行着占着cpu的，还有在进程队列中等待着的去占cpu的，还有处于不可中断状态的（为什么叫不可中断状态？）如等待磁盘的。

负载均值有三个时间间隔，1分，5分，15分。负载均值为1在单个cpu的机器上负载满了，但是在4个cpu的表明其在75%的时间内是空闲的。

#### 计算
那么均值是如何均的呢？

    #define FSHIFT   11		/* nr of bits of precision */
    #define FIXED_1  (1<<FSHIFT)	/* 1.0 as fixed-point */
    #define LOAD_FREQ (5*HZ)	/* 5 sec intervals */
    #define EXP_1  1884		/* 1/exp(5sec/1min) as fixed-point */
    #define EXP_5  2014		/* 1/exp(5sec/5min) */
    #define EXP_15 2037		/* 1/exp(5sec/15min) */

    #define CALC_LOAD(load,exp,n) \
       load *= exp; \
       load += n*(FIXED_1-exp); \
       load >>= FSHIFT;

    unsigned long avenrun[3];

    static inline void calc_load(unsigned long ticks)
    {
       unsigned long active_tasks; /* fixed-point */
       static int count = LOAD_FREQ;

       count -= ticks;
       if (count < 0) {
          count += LOAD_FREQ;
          active_tasks = count_active_tasks();
          CALC_LOAD(avenrun[0], EXP_1, active_tasks);
          CALC_LOAD(avenrun[1], EXP_5, active_tasks);
          CALC_LOAD(avenrun[2], EXP_15, active_tasks);
       }
    }

在linux系统上，负载均值不是每个时钟周期tick都计算的，但是会被一个基于Hz频度设置的变量值触发,也就是调用上边的calc_load函数中if(count < 0)中的代码块（这个值是可以设置的，是linux内核活动的心率，默认情况下1Hz等于1tick，也就是10ms）。虽然Hz的值在某些内核版本中是可配，不过它通常被设为100(就是一秒啦，这里的Hz给我们高中学的频率的单位好像不一样哎)，通过这个值来计算内核负载的计算频度。一般情况是每5Hz触发一次这个算法

（每个时钟周期都检测调用calc_load函数，但是5s才触发一次计算CALC_LOAD）

    1 HZ = 100 ticks
    5 HZ = 500 ticks
    1 tick = 10 milliseconds
    500 ticks = 5000 milliseconds (or 5 seconds)
    So, 5 HZ means that CALC_LOAD is called every 5 seconds.

每个时钟周期都调用一下calc_load方法。LOAD_FREQ是个宏，值为5Hz，count初始值为LOAD_FREQ,没调用一次calc_load，其值减少1ticks，当其减少到小于0时候，触发CALC_LOAD中算法，计算load值，其中avenrun数组分别保存系统当前1分，5分，15分的负载均值。

active_tasks的计算源码,即使遍历tasklist的链表，累加所有task状态为running的，runnable的和uninterruptible的进程的个数。

/* Nr of active tasks - counted in fixed-point numbers
*/
static unsigned long count_active_tasks (void) {
    struct task struct *p;
    unsigned long nr = 0;

    read_lock (&tasklist_lock);
    for_each_task(p) {
        if ( (p->state == TASK_RUNNING ||
                (p->state & TASK_UNINTERRUPTIBLE ) ) )
        nr += FIXED_1;
        }
    read_unlock(&tasklist_lock);
    return nr;
}

下边是推导的CALC_LOAD的函数，至于exp为什么要那么计算，其数学背景是什么，等我知道了再补上。

![load计算](/images/unix/load_compute.png)

#### 结论

`load_m = load_m * exp(-5 / 60m) + n * (1 – exp(-5 / 60m))`

其中m为1或5或15

注：外链二中源码有误（第10页代码631行）

#### 参考文献
load维基百科

<http://www.teamquest.com/import/pdfs/whitepaper/ldavg1.pdf>

<http://www.gracecode.com/posts/2973.html>

<http://www.penglixun.com/tech/system/how_to_calc_load_cpu.html>
