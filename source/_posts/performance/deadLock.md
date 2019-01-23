---
layout: post
title: 死锁了
date: 2015-12-27
categories: performance
tags:
    - deadlock
---

#### 代码

    public class MonitorThreadDeadLockTest {
        static class SynAddRunable implements Runnable{
            int a,b;
            public SynAddRunable(int a,int b){
                this.a = a;
                this.b = b;
            }
            @Override
            public void run() {
                synchronized (Integer.valueOf(a)){
                    synchronized (Integer.valueOf(b)){
                        System.out.println(a + b);
                    }
                }
            }
        }
        public static void main(String[] args) {
            for (int i = 0; i < 1000; i++) {
                new Thread(new SynAddRunable(1,2)).start();
                new Thread(new SynAddRunable(2,1)).start();
            }
        }
    }

#### 可能的现象

1.大量block的线程,可能很多对象都要拿那把锁.

#### 检测

1.拿到异常的java进程　top,sar,jps等.公司的监控系统都能看出来.

![top](/images/performance/top_dead_lock.png)

2.查看其线程dump

    sudo  -u tomcat jstack 6123 > stack.log

3.大量线程blocked,并发现deadlock

![dead_lock](/images/performance/dead_lock_block.png)

4.卧槽，死锁了

### 工作遇到的

//todo
