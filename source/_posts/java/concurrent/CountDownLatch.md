---
layout: post
title: CountDownLatch源码分析
date: 2016-08-26
categories: java
tags:
    - countDownLatch
    - juc
    - aqs
---


以核心函数为突破口countDown和await

## CountDownLatch构造器

    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

    Sync(int count) {
        // 设置state为count
        setState(count);
    }

## CountDownLatch#countDown

    public void countDown() {
        sync.releaseShared(1);
    }

### AbstractQueuedSynchronizer#releaseShared

    public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
            // 如果state为0，走不到这里；如果cas进行state-1后state不为0，也走不到这里。详看tryReleaseShared
            doReleaseShared();
            return true;
        }
        return false;
    }

### CountDownLatch#tryReleaseShared

    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            // 如果线程持有锁
            int nextc = c-1;
            // 可能并发的改state的值，所以这里需要cas
            if (compareAndSetState(c, nextc))
                // 如果nextc不为0，返回false
                return nextc == 0;
        }
    }

### AbstractQueuedSynchronizer#doReleaseShared

// 如果state不为0，并且cas进行state - 1后，state不为0，则执行doReleaseShared

    private void doReleaseShared() {
       for (;;) {
           Node h = head;
           if (h != null && h != tail) {
               int ws = h.waitStatus;

               // signal : waitStatus value to indicate successor's thread needs unparking    
               // 后继节点等待被触发
               if (ws == Node.SIGNAL) {
                   // 如果节点状态为signal，cas重置节点状态为0
                   if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                       continue;            // loop to recheck cases
                   // 如果重置成功，唤醒后继节点       
                   unparkSuccessor(h);
               }
               // 如果头节点为状态为初始状态，尝试设置waitStatus为propagate
               // propagate: waitStatus value to indicate the next acquireShared should unconditionally propagate
               else if (ws == 0 &&
                        !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                   continue;                // loop on failed CAS
           }
           if (h == head)                   // loop if head changed
               break;
       }
   }

### AbstractQueuedSynchronizer#unparkSuccessor

    private void unparkSuccessor(Node node) {
       int ws = node.waitStatus;
       // 唤醒后继节点，需要当前节点的waitStatus置为0或1
       if (ws < 0)
           compareAndSetWaitStatus(node, ws, 0);

       Node s = node.next;
       if (s == null || s.waitStatus > 0) {
           // 如果s为null或者s为取消状态    
           s = null;
           // 从后向前遍历node，直到第一个waitStatus不是cancel的node
           for (Node t = tail; t != null && t != node; t = t.prev)
               if (t.waitStatus <= 0)
                   s = t;
       }
       // 然后unpark之。
       if (s != null)
           LockSupport.unpark(s.thread);
   }

##  await
使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断

### doAcquireSharedInterruptibly

    private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        // 包装为共享节点，然后添加进等待队列
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    // 参照下doAcquireInterruptibly,这里是不一样滴。
                    // 试图在共享模式下获取对象状态
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        // 这里需要state为0

                        // 这里对比doAcquireInterruptibly
                        // 设置头节点 并传播
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

### setHeadAndPropagate

    private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; // Record old head for check below
        setHead(node);
        /*
         * Try to signal next queued node if:
         *   Propagation was indicated by caller,
         *     or was recorded (as h.waitStatus either before
         *     or after setHead) by a previous operation
         *     (note: this uses sign-check of waitStatus because
         *      PROPAGATE status may transition to SIGNAL.)
         * and
         *   The next node is waiting in shared mode,
         *     or we don't know, because it appears null
         *
         * The conservatism in both of these checks may cause
         * unnecessary wake-ups, but only when there are multiple
         * racing acquires/releases, so most need signals now or soon
         * anyway.
         */
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                // 后继为空或者为共享模式
                doReleaseShared();
        }
    }
