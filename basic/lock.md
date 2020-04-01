# 锁

### NSLock

低级别的锁，[lock lock] 意味着获取锁，之后为线程安全，此处永远只允许一个线程访问，直到 [lock unlock] 后，其他线程才可以获取锁。

NSLock 必须在锁定的线程中进行解锁。

调用 lock 之前，必须先调用 unlock，以防未解锁。


### NSRecursiveLock

递归锁，在调用 unlock 之前，可以多次调用 lock， 如果 unlock 与 lock 次数相匹配，则认为锁被释放了，其他线程可以获取锁。

```
NSRecursiveLock *lock;

-(void)safeMethod1 { 
	// 获取锁
	[self->lock lock];
	// 调用方法2
	[self safeMethod2];
	// 此处为真正解锁
	[self->lock unlock];
}

-(void)safeMethod2 {
	// 方法2可以继续获取锁
	[self->lock lock];
	/*线程安全的代码*/
	// 方法2解锁，但是方法1尚未解锁
	[self->lock unlock];
}
```

### NSCondition

条件锁

解决标准的生产者消费者问题

一个线程需要等待其他线程返回结果，NSCondition 可以原子性的释放锁，其他等待的线程可以获取锁，原线程继续等待。

一个线程会等待释放锁的条件变量，另一个线程会通知条件变量释放锁，并唤醒等待中的线程。

### 并发读写用锁

读取数据可以是并行的，但是写入操作是需要互斥的，同一时间只允许一个写入。

`dispatch_barrier` 将写入操作放到栅栏中。








