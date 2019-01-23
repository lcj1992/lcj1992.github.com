---
layout: post
title: ReentrantLock源码解析
date: 2016-08-22
categories: java
tags:
    - aqs
    - ReentrantLock
    - CLH
    - JUC
---

## 概述  

ReentrantLock的结构体如下

![reentrantLock](/images/java/reentrantLock.png)

1.  内部类`Sync`继承自`AbstractQueuedSynchronizer` 参见[aqs](/2016/08/24/aqs)
2.  Sync两个子类 `NonfairSync`实现非公平锁、`FairSync`实现公平锁
2.  使用AQS的独占API

## 非公平锁

idea debug多线程，将断点设置为线程级别就可以了,如下图所示
![多线程debug](/images/java/multi_thread_debug.png)

然后在debug时，在这里切换线程
![切换线程](/images/java/switch_thread.png)

设置以下几个断点
![断点位置](/images/java/break_points.png)

本文使用的例子

    // 1.线程1首先拿到锁，但不释放锁呢。
    // 2.线程2，3，4依次入等待队列
    // 3.线程4自旋一次，会将pred的waitStatus改为signal，观察线程3释放锁时的unparkSuccessor操作。
    @Test
    public void howToLockTest() throws InterruptedException {
        ReentrantLock lock = new ReentrantLock();
        new Thread(new MyRunnable(lock), "thread1").start();
        Thread.sleep(100);
        new Thread(new MyRunnable(lock), "thread2").start();
        new Thread(new MyRunnable(lock), "thread3").start();
        new Thread(new MyRunnable(lock), "thread4").start();
        Thread.sleep(1000000000);
    }

    private class MyRunnable implements Runnable {
        private ReentrantLock lock;

        MyRunnable(ReentrantLock lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            lock.lock();
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }

### 加锁过程

#### NonfairLock#lock

ReentrantLock#lock() -> NonfairLock#lock()

     // 执行加锁，立即闯入，如果失败的话执行正常的acquire
    final void lock() {
        // CAS操作保证操作的原子性
        // state声明为 volatile保证内存可见性,state=0表明可以争用。
        // 非公平的，刚进来就大家随便争，可以对比下公平锁的lock这里怎么搞的。
        if (compareAndSetState(0, 1))
            // 如果state 0->1设置成功,设置当前线程为同步器的属主线程
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

线程1因为比线程2，3，4执行提前0.1s，所以肯定走不到else分支，
因为线程1拿到锁之后并不释放锁，所以线程2，3，4肯定走else分支，acquire(1)

#### AQS#acquire

    // 尝试获取锁，获取不到则创建一个waiter（当前线程）后放到队列中
    public final void acquire(int arg) {
        // 以独占模式acquire失败
        if (!tryAcquire(arg) &&
        // 包装一个当前线程的Node，入队列，然后尝试去acquire
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            // 如果该线程被中断了，中断当前线程
            selfInterrupt();
    }

#### Sync#nonfairTryAcquire

NonfairSync#tryAcquire(arg) -> Sync#nonfairTryAcquire(arg)

    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        // 获取state标志位，如果为0，没有线程持有锁
        if (c == 0) {
            // CAS尝试去获取锁。
            if (compareAndSetState(0, acquires)) {
                // 如果获取成功，设置属主线程为当前线程。
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        // 如果不为0，判断当前线程是否为属主线程，这里实现可重入
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            // 设置state的值，state的值表明当前线程的重入数。
            setState(nextc);
            return true;
        }
        return false;
    }

如果tryAcquire失败，把当前线程包装为一个Node入队列，并再次尝试acquire

包装当前线程为一个node

#### AQS#addWaiter

AbstractQueuedSynchronizer#addWaiter()

    private Node addWaiter(Node mode) {
       Node node = new Node(Thread.currentThread(), mode);
       // Try the fast path of enq; backup to full enq on failure
       Node pred = tail;
       // 如果队列不为空
       if (pred != null) {
           // node的先驱节点设置为tail
           node.prev = pred;
           // 设置当前节点为tail
           if (compareAndSetTail(pred, node)) {
               // 原有的tail的后继节点指向node
               pred.next = node;
               return node;
           }
       }
       // 如果pred为null，说明之前队列为空，执行enq方法，自旋方式入队列
       // 或者设置当前节点为tail失败，表明有别的线程并发入队列，执行enq方法，自旋方式入队列。
       enq(node);
       return node;
   }

#### AQS#enq

AbstractQueuedSynchronizer#enq()

    private Node enq(final Node node) {
        // 自旋方式入队列
        for (;;) {
            Node t = tail;
            // !!!!重要
            // 如果队列为空，初始化head和tail节点
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                // 如果队列不为空，将该节点的前驱节点设置为tail，
                node.prev = t;
                // 同时cas设置tail为node，如果设置失败，说明有并发修改tail
                if (compareAndSetTail(t, node)) {
                    // 设置tail的后继节点为node
                    t.next = node;
                    return t;
                }
            }
        }
    }

入队列后，然后尝试从队列中acquire

#### AQS#acquireQueued

AbstractQueuedSynchronizer#acquireQueued()

    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
                // 如果先驱节点为head，尝试acquire
                if (p == head && tryAcquire(arg)) {
                    // acquire成功，将head节点置为node
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 如果先驱节点不是head，或者是head，但是尝试acquire失败
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

AbstractQueuedSynchronizer#setHead()

    private void setHead(Node node) {
        head = node;
        // 头节点不绑定线程
        node.thread = null;
        node.prev = null;
    }

#### AQS#shouldParkAfterFailedAcquire

     /** waitStatus value to indicate thread has cancelled */
     // 当前线程已经被取消
     static final int CANCELLED =  1;
     /** waitStatus value to indicate successor's thread needs unparking */
     // 后继节点线程等待被触发
     static final int SIGNAL    = -1;
     /** waitStatus value to indicate thread is waiting on condition */
     // 线程等待条件
     static final int CONDITION = -2;
     /**
      * waitStatus value to indicate the next acquireShared should
      * unconditionally propagate
      */
     // 节点状态需要向后传播
     static final int PROPAGATE = -3;

    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            // 只有当前节点的前一个节点为SIGNAL时，当前节点才能被挂起
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }

### 解锁过程

解锁过程较简单点

#### AbstractQueuedSynchronizer#release

ReentrantLock#unlock() -> AbstractQueuedSynchronizer#release()

    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            // 属主线程可能是head节点也可能不是head节点哟。
            if (h != null && h.waitStatus != 0)
                // 如果是head节点，并且waitStatus不为0(SIGNAL、CONDITION、PROPAGATE）,则唤醒后继节点。
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

#### ReentrantLock#tryRelease

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        // 可重入，只有减为0时，才是free，别的线程才能抢得到。
        if (c == 0) {
            free = true;
            // 设置属主线程为null
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

#### AQS#unparkSuccessor

        private void unparkSuccessor(Node node) {
       /*
        * If status is negative (i.e., possibly needing signal) try
        * to clear in anticipation of signalling.  It is OK if this
        * fails or if status is changed by waiting thread.
        */
       // 如果状态是负的（例如，可能等待着被signal）
       int ws = node.waitStatus;
       if (ws < 0)
           compareAndSetWaitStatus(node, ws, 0);

       /*
        * Thread to unpark is held in successor, which is normally
        * just the next node.  But if cancelled or apparently null,
        * traverse backwards from tail to find the actual
        * non-cancelled successor.
        */
       Node s = node.next;
       if (s == null || s.waitStatus > 0) {
           s = null;
           for (Node t = tail; t != null && t != node; t = t.prev)
               if (t.waitStatus <= 0)
                   s = t;
       }
       if (s != null)
           LockSupport.unpark(s.thread);
   }

## 公平锁

### 加锁过程

整体流程和非公平加锁是一样的。

#### FairLock#lock

    final void lock() {
        // if (compareAndSetState(0, 1))
        //    setExclusiveOwnerThread(Thread.currentThread());
        // else
        // 必须严格按照队列顺序来
        acquire(1);
    }

#### FairSync#tryAcquire

    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            // 比非公平加锁，多了这个判定条件 hasQueuedPredecessors，必须没有前驱节点才让你acquire，否则的话，好好等着，轮到你再来。
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

解锁过程和非公平锁是一样一样的。

## 参考

[x86的指令集]<http://www.felixcloutier.com/x86/>

[AQS,老爷子的论文]<http://gee.cs.oswego.edu/dl/papers/aqs.pdf>

[AbstractQueuedSynchronizer的实现分析（上）]<http://www.infoq.com/cn/articles/jdk1.8-abstractqueuedsynchronizer>

[AbstractQueuedSynchronizer的实现分析（下）]<http://www.infoq.com/cn/articles/java8-abstractqueuedsynchronizer>
