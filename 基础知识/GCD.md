# GCD

苹果官方文档显示:
应用程序中编写的线程管理用的代码要在系统级实现.
苹果的系统级别是iOS和OSX的核心XNU内核级上实现.

1. 编程人员所使用的GCD的API全部为包含在libdispatch库中的C语言函数.
2. Dispatch Queue 通过结构体和链表,被实现成FIFO队列.FIFO队列管理是通过dispatch_async等函数所追加的Block.
3. Block 并不是直接加入FIFO队列,而是先加入Disptch_Continuation_t 类型结构体中,然后加入FIFO队列.  该Dispatch Continuation 用于记忆Block所属的Dispatch Group和其他一些信息,相当于一般常说的执行上下文.
4. Dispatch Queue 通过dispatch_set_target_queue 函数进行设定,同时也会优先级的信息.phthread_workqueue_create_np    XNU内核持有 workqueue
5. Dispacth 执行Block的过程. 当在Global Dispatch Queue中执行Block时,Libdispatch 从Global Dispatch Queue自身的FIFO队列中取出Dispatch Continuation,调用pthread_workqueue_additem_np函数.  将Global Dispatch Queue自身信息,符合优先级的work却也信息以及为执行Dispatch Contitnuation的回调函数等传递给参数.
pthread_workqueue_additem_np函数使用workq_kernreturn系统调用,通过workqueue增加应当执行项目,根据该通知,XNU系统会判断是否生产线程.
6. workqueue 的线程执行pthread_workqueue函数,该函数调用libdispatch的回调函数.在回调函数中执行加入到DispatchContinuation的Block.
7. block 执行完了后,进行通知Dispatch Group结束,释放Dispatch Continuation等处理,开始准备执行下一个Block.


## dispatch_once原理

dispatch_once一般用于创建单例，示例代码：
```
static dispatch_once_t onceToken;
static Person * per;
+ (Person *)shared{
    dispatch_once(&onceToken, ^{
        per = [[Person alloc] init];
    });
    return per;
}
```

声明静态变量onceToken，是dispatch_once_t类型（long），并将其地址传给dispatch_once方法。下面是dispatch_once：

```
void _dispatch_once(dispatch_once_t *predicate, DISPATCH_NOESCAPE dispatch_block_t block){
	if (DISPATCH_EXPECT(*predicate, ~0l) != ~0l) {
		dispatch_once(predicate, block);
	} else {
		dispatch_compiler_barrier();
	}
	DISPATCH_COMPILER_CAN_ASSUME(*predicate == ~0l);
}
```
首先判断参数predicate是否不等于```~0l（-1）```，如果为YES，则表明当前为第一次执行，那么就直接执行block，否则就会执行一个```dispatch_compiler_barrier()```，这是一个do-while(0)的一次循环，目前还不清楚这是要做什么。

经测试，onceToken一共有三个值，nil、140734545356784（不定值，可能会变）和-1，生命onceToken时为nil，第一次进入dispatch_once方法中执行时为140734545356784，当dispatch_once方法执行完成之后设置为-1。

猜测，首先根据-1判断是否执行过block，根据140734545356784判断是否正在执行当中，执行中时，会等待返回。

另外测试多个线程同时调用一个初始化时间较长的单例，得出结果：各个线程都会被阻塞，直至执行block的线程返回后，所有线程执行回调，并没有先后顺序，但返回值相同。**结论：**从第一次执行block到执行完成期间，所有的调用初始化方法，都会被阻塞，直到初始化完成，并将onceToken设置为-1。

## Dispatch Queue

Dispatch Queue 是一个类似队列的数据结构，而且是 FIFO队列，因此任务开始执行的顺序，就是你把它们放到 queue 中的顺序。GCD 中的队列有下面三种：

* Serial（串行队列）按照添加进队列的顺序一个一个执行。
* Concurrent(并行队列)也叫 global dispatch queue，可以并发执行多个任务，但是顺序，仍然是按照添加的顺序执行。由GCD控制具体执行的线程和并发数。
* Main Dispatch Queue(主队列)是一个全局可见的**串行**队列。主队列通过与应用程序的 runloop 交互，把任务安插到 runloop 当中执行。

dispatch queue 不是线程，可以管理多个线程。

### 自己创建的队列和系统的队列

事实上，我们自己创建的队列最终会放到系统提供的主队列和四个全局的并发队列上执行。这种操作叫做 Target queues。具体来说，我们创建的串行队列的 target queue 就是系统的主队列，我们创建的并行队列的 target queue 默认是系统 default 优先级的全局并行队列。所有放在我们创建的队列中的任务，最终都会到 target queue 中完成真正的执行。

通过我们自己创建的队列，以及 dispatch_set_target_queue 和 barrier 等操作，可以实现比较复杂的任务之间的同步。
