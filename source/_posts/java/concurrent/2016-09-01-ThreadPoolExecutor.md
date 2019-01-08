---
layout: post
title: ThreadPoolEexcutor
categories: java
tags: ThreadPoolExecutor FutureTask RunnableFuture ExecutorService
---

首先需要明确的是ThreadPoolExecutor虽然是1.5引入的，但是底层还是一些jdk1.0的东西，Thread和Runnable，然后加以管理.

### ctl

线程池最重要的一个状态了。一个int型，包含两个域：1.  线程池数量(workCount后29位) 2.  运行状态(runState前3位)   

    // 后29位保存线程池数量
    private static final int COUNT_BITS = Integer.SIZE - 3;
    // 最大的线程池数量是2的29次方-1
    private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

    // runState is stored in the high-order bits
    // 111000..000
    private static final int RUNNING    = -1 << COUNT_BITS;
    // 000000..000
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    // 001000..000
    private static final int STOP       =  1 << COUNT_BITS;
    // 010000..000
    private static final int TIDYING    =  2 << COUNT_BITS;
    // 011000..000
    private static final int TERMINATED =  3 << COUNT_BITS;hreadPoolExecutor几个重要方法为突破口

    // 提供ctl、runState、workCount的转换方法
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    private static int workerCountOf(int c)  { return c & CAPACITY; }
    private static int ctlOf(int rs, int wc) { return rs | wc; }

### FutureTask

因为所有提交给线程池的Runnable或者Callable都会被newTaskFor()包装成一个个的FutureTask，然后交给一个个的Worker，所以有必要提下。

    FutureTask<V> implement RunnableFuture<V>
    RunnableFuture<V> extend Runnable,Future<V>

首先看FutureTask的继承体系，继承自RunnableFuture，实现了Runnable和Future接口。

也就是说FutureTask可以作为Runnable扔给线程池去执行，然后通过Future.get()拿到返回的结果。

然后看下FutureTask几个关键的域：关联的`callable`，`outcome`运行结果、`runner`运行这个futureTask的线程.

正常的流程:通过runner.start()方法启动线程，调用run()方法、run()方法里会调用callable()的call方法，并把结果保存在outcome中。然后通过future.get()拿到outcome中的结果。这是Callable可以返回结果的原因。

ps:注意线程池里并不是通过runner.start()来启动线程的，通过Worker里的thread来启动的，下边有讲。

    /** The underlying callable; nulled out after running */
    // 所关联的callable
    private callable<V> callable;

    /** The result to return or exception to throw from get() */
    // 结果
    private Object outcome; // non-volatile, protected by state reads/writes

    /** The thread running the callable; CASed during run() */
    // 运行这个futureTask的线程
    private volatile Thread runner;

### Worker

Worker继承自AbstractQueuedSynchronizer和Runnable。有两个关键的域需提下：持有执行任务的线程thread，以及要执行的任务firstTask.

     /** Thread this worker is running in.  Null if factory fails. */
     // worker绑定的执行任务的线程
     final Thread thread;

     /** Initial task to run.  Possibly null. */
     // worker绑定的要执行的任务
     Runnable firstTask;

     /** Per-thread task counter */
     volatile long completedTasks;

在构造Worker时，会初始化这两个域，并指定thread的Runnable为this(worker自己)，run() -> runWorker() -> firstTask.run().

     Worker(Runnable firstTask) {
         setState(-1); // inhibit interrupts until runWorker
         this.firstTask = firstTask;
         this.thread = getThreadFactory().newThread(this);
     }


### invokeAll(callableList)

源码注解invokeAll

    public <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException {
        if (tasks == null)
            throw new NullPointerException();
        ArrayList<Future<T>> futures = new ArrayList<Future<T>>(tasks.size());
        boolean done = false;
        try {
            for (Callable<T> t : tasks) {
                // 将callable包装成RunnableFuture。
                RunnableFuture<T> f = newTaskFor(t);
                futures.add(f);
                // 执行runnableFuture
                execute(f);
            }
            for (int i = 0, size = futures.size(); i < size; i++) {
                Future<T> f = futures.get(i);
                // 先判定任务是否完成
                if (!f.isDone()) {
                    try {
                        // 如果没有完成，这里会自旋阻塞当前结果。
                        f.get();
                    } catch (CancellationException ignore) {
                    } catch (ExecutionException ignore) {
                    }
                }
            }
            done = true;
            return futures;
        } finally {
            if (!done)
                for (int i = 0, size = futures.size(); i < size; i++)
                    futures.get(i).cancel(true);
        }
    }

execute()

    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        // 如果池中线程小于corePoolSize，会一直添加，即时有空余线程。
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }

addWorker()

    private boolean addWorker(Runnable firstTask, boolean core) {
        retry:
        for (;;) {
            int c = ctl.get();
            int rs = runStateOf(c);

            // Check if queue empty only if necessary.
            if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN &&
                   firstTask == null &&
                   ! workQueue.isEmpty()))
                return false;

            for (;;) {
                int wc = workerCountOf(c);
                if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                    return false;
                if (compareAndIncrementWorkerCount(c))
                    // workCount原子自增，自增成功break出for循环
                    break retry;
                c = ctl.get();  // Re-read ctl
                if (runStateOf(c) != rs)
                    continue retry;
                // else CAS failed due to workerCount change; retry inner loop
            }
        }

        boolean workerStarted = false;
        boolean workerAdded = false;
        Worker w = null;
        try {
            // 这里包赚firstTask，同时实例化运行该task的Thread。
            w = new Worker(firstTask);
            // t为运行该task的线程
            final Thread t = w.thread;
            if (t != null) {
                final ReentrantLock mainLock = this.mainLock;
                mainLock.lock();
                try {
                    // Recheck while holding lock.
                    // Back out on ThreadFactory failure or if
                    // shut down before lock acquired.
                    int rs = runStateOf(ctl.get());

                    if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                        if (t.isAlive()) // precheck that t is startable
                            throw new IllegalThreadStateException();
                        workers.add(w);
                        int s = workers.size();
                        if (s > largestPoolSize)
                            largestPoolSize = s;
                        workerAdded = true;
                    }
                } finally {
                    mainLock.unlock();
                }
                if (workerAdded) {
                    // 启动线程
                    t.start();
                    workerStarted = true;
                }
            }
        } finally {
            if (! workerStarted)
                addWorkerFailed(w);
        }
        return workerStarted;
    }

f.get()

    public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING)
            // 这里自旋直到拿到结果。
            s = awaitDone(false, 0L);
        // 返回结果或异常
        return report(s);
    }
