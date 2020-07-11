## 锁

- 执行时间短，数量多，用自旋锁 CAS Lock

- 执行时间长，数量少，用系统锁 OS Lock

## volatile 关键字

- 保证线程的可见性
  - MESI
  - CPU缓存一致性协议

- 禁止指令重排序
  - DCL单例
  - 双重检查锁
  

## 计数效率

LongAdder （分段锁）> AtomicInterger > Synchronize

## ReentrantLock

- cas
- lock 死等
- tryLock 可以设置超时时间
- fair / non fair

## CountDownLatch

- latch.await
- latch.countDown

## CyclicBarrier

```java
CyclicBarrier barrier = new CyclicBarrier(20, new Runnable() {
  	@Override
    public void run(){
  	System.out.println("满人，发车")
  }
});
```

## Phaser

每个阶段都得等所有线程都结束才能执行下一个阶段

## ReadWriteLock

- 共享锁
- 排它锁

## Semaphore

允许N个线程同时运行

- 限流
- fair / non fair

## Exchange

不常用