# ThreadPoolExecutor 源码

## 核心成员变量

```java
/**
 * 线程状态和数量
 * 32 位二进制的数字，前三位表示线程池状态，后29位表示线程数量
 */
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));

// 线程工厂
private volatile ThreadFactory threadFactory;

// 拒绝策略
private volatile RejectedExecutionHandler handler;

// 线程空闲时间
private volatile long keepAliveTime;

// 核心线程数量
private volatile int corePoolSize;

// 最大线程数量
private volatile int maximumPoolSize;

// 拒绝策略
private static final RejectedExecutionHandler defaultHandler = new AbortPolicy();

// ThreadPoolExecutor 级别的锁
private final ReentrantLock mainLock = new ReentrantLock();

// 工作线程容器
private final HashSet<Worker> workers = new HashSet<Worker>();
```

## 执行任务

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    int c = ctl.get();
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
```

> 1. 首先会核心创建线程执行任务，当核心线程数量达到 `corePoolSize` 后会将任务入队，当队列满了后将工作线程数量增加到 `maximumPoolSize`，如果此时还有任务则会走拒绝策略
> 2. `addWorker`中的 `compareAndIncrementWorkerCount(c)` 基于 CAS 创建线程保证线程数量

## 四种拒绝策略

- **AbortPolicy** - 抛出异常
- **CallerRunsPolicy** - 直接在execute方法的调用线程中运行被拒绝的任务
- **DiscardPolicy** - 丢弃任务
- **DiscardOldestPolicy** - 丢弃最早的未处理请求，然后重试execute 