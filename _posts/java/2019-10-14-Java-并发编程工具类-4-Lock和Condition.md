# Lock&Condition

Java SDK 并发包通过 Lock 和 Condition 两个接口来实现管程, 其中 Lock 用于解决互斥问题, Condition 用于解决同步问题.

## 再造管程的理由

synchronized 无法解决 **不可抢占条件**.

## 破坏 不可抢占条件 的三个方法
1. 能够响应中断. 使阻塞状态线程能够响应中断信号 Lock.lockInterruptibly()
2. 支持超时. 使线程阻塞一定时间后, 退出阻塞 Lock.tryLock(long time, TimeUnit unit)
3. 非阻塞地获取锁. 无法获得锁后, 不进入阻塞 Lock.tryLock()

## 保证可见性
利用 volatile 相关的 Happens-Before 规则
一个 volatile 修饰的变量 state

## 用锁的最佳实践
1. 永远只在更新对象的成员变量的时候加锁
2. 永远只在访问可变的成员变量的时候加锁
3. 永远不在调用其他对象的方法时加锁

## 同步和异步
调用方需要等待结果是同步
调用方不需要等待结果是异步

# Semaphore
信号量
一个计数器, 一个等待队列, 三个方法

## 三个方法
1. init() 设置计数器的初始值 0-初始值 代表有多少线程在等待 
2. down() 计数器值减 1, 如果此时计数器的值小于0, 当前线程将被阻塞, 否则当前线程可以继续执行
3. up() 计数器值加 1, 如果此时计数器的值小于等于0, 则唤醒等待队列中的一个线程, 并将其移除等待队列

## acquire()
down()操作

## release()
up()操作

## 限流功能
可以允许多个线程访问一个临界区

# ReadWriteLock
读写锁
1 允许多个线程同时读共享变量
2 只允许一个线程写共享变量
3 如果有一个线程正在执行写操作, 此时禁止读线程读共享变量

## 缓存的初始化
分为一次性加载和按需加载

### 一次性加载
适合缓存数据量不大的情况, 在应用启动的时候加载

### 按需加载
适合源头数据量非常大的时候, 也叫懒加载

#### 实现按需加载
```java
class Cache<K,V> {
  final Map<K, V> m =
    new HashMap<>();
  final ReadWriteLock rwl = 
    new ReentrantReadWriteLock();
  final Lock r = rwl.readLock();
  final Lock w = rwl.writeLock();
 
  V get(K key) {
    V v = null;
    // 读缓存
    r.lock();         ①
    try {
      v = m.get(key); ②
    } finally{
      r.unlock();     ③
    }
    // 缓存中存在，返回
    if(v != null) {   ④
      return v;
    }  
    // 缓存中不存在，查询数据库
    w.lock();         ⑤
    try {
      // 再次验证
      // 其他线程可能已经查询过数据库
      v = m.get(key); ⑥
      if(v == null){  ⑦
        // 查询数据库
        v= 省略代码无数
        m.put(key, v);
      }
    } finally{
      w.unlock();
    }
    return v; 
  }
}
```

# StampedLock
不可重入
java1.8 读多写死场景, 性能比读写锁好 
StampedLock 支持三种模式, 写锁/ 悲观读锁和乐观读锁
写锁与悲观读锁互斥. 写锁和悲观读锁加锁成功之后, 都会返回一个stamp; 解锁的时候需要传入这个stamp

```java
final StampedLock sl = 
  new StampedLock();
  
// 获取 / 释放悲观读锁示意代码
long stamp = sl.readLock();
try {
  // 省略业务相关代码
} finally {
  sl.unlockRead(stamp);
}

// 获取 / 释放写锁示意代码
long stamp = sl.writeLock();
try {
  // 省略业务相关代码
} finally {
  sl.unlockWrite(stamp);
}

```
StampedLock 的性能之所以比 ReadWriteLock 还要好, 其关键是 StampedLock 支持乐观读的方式, 其允许一个线程获取写锁, 也就是说不是所有写操作都被阻塞.

## 读模板
```java
final StampedLock sl = 
  new StampedLock();

// 乐观读
long stamp = 
  sl.tryOptimisticRead();
// 读入方法局部变量
......
// 校验 stamp
if (!sl.validate(stamp)){
  // 升级为悲观读锁
  stamp = sl.readLock();
  try {
    // 读入方法局部变量
    .....
  } finally {
    // 释放悲观读锁
    sl.unlockRead(stamp);
  }
}
// 使用方法局部变量执行业务操作
......
```

## 写模板
```java
long stamp = sl.writeLock();
try {
  // 写共享变量
  ......
} finally {
  sl.unlockWrite(stamp);
}

```

# CountDownLatch和CyclicBarrier
CountDownLatch 有个计数器, 可实现主线程等待 N 个线程执行完再执行
CyclicBarrier 可以实现线程同步, 可以实现一组线程互相等待



