# 多线程技术

## 概念理解

### 进程、线程与队列的关系与区别

#### 进程与线程

- 进程是系统中正在运行的一个应用程序，每个进程之间是独立的，每个进程均运行在专用且受保护的内存空间内。
- 线程是进程的基本执行单元，一个进程中的任务都在线程中执行，所以一个进程由至少一个线程组成。

#### 主线程

主线程主要负责显示和刷新UI界面，处理UI事件。

#### 队列的类型

- 主队列：`dispatch_get_main_queue`
- 全局并发队列：`dispatch_get_global_queue`
- 自己创建队列：`dispatch_queue_create`




### 多线程的 并行 和 并发 有什么区别？

- **并行**：充分利用计算机的多核，在多个线程上同步进行
- **并发**：在一条线程上通过快速切换，让人感觉在同步进行




### 什么是多线程？

一个线程中的任务是串行的，同一时间内，一个线程只能执行一个任务。一个进程可以开启多条线程，每条线程可以并行执行不同的任务。同一时间，CPU 只能处理一条线程，多线程并发执行是 CPU 快速地在多条线程之间调度。




### iOS 中实现多线程的几种方案，各自有什么特点？

iOS 的多线程技术有：`pthread`、`NSThread`、`GCD`、`NSOperation`。

- NSThread 面向对象的，需要程序员手动创建线程，但不需要手动销毁。子线程间通信很难。
- GCD c语言，充分利用了设备的多核，自动管理线程生命周期。比 NSOperation 效率更高。
- NSOperation 基于 GCD 封装，更加面向对象，比 GCD 多了一些功能。

#### GCD 和 NSOperation 区别

- GCD 是纯 C 语言的 API；NSOperation 是基于 GCD 的 OC 版本封装
- GCD 只支持 FIFO 的队列；NSOperation 可以很方便地调整执行顺序，设置最大并发数量
- NSOperationQueue 可以轻松在 operation 间设置依赖关系，而 GCD 需要些很多代码才能实现
- NSOperationQueue 支持 KVO，可以检测 operation 是否正在执行(isExecuted)，是否结束(isFinish)，是否取消(isCancel)
- GCD 的执行速度比 NSOperation 快




### GCD 的队列（`dispatch_queue_t`）分哪两种类型？

1. 串行队列 Serial Dispatch Queue
2. 并行队列 Concurrent Dispatch Queue




### 什么是死锁？ 如何避免？


死锁就是队列引起的循环等待：在串行队列 A 中向队列 A 添加一个同步任务，例如主队列同步：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    dispatch_sync(dispatch_get_main_queue(), ^{ // 👈死锁在这一行
        // NSLog(@"在主队列同步执行");
    });
}
```

或者在自定义线程中：


```objc
// NOT OK
- (void)test1 {
    NSLog(@"在主线程添加一个串行队列queue");
    dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_SERIAL);
    dispatch_async(queue, ^{
        NSLog(@"在串行队列queue中添加一个同步任务");
        dispatch_sync(queue, ^{ // 👈死锁在这一行
            NSLog(@"OK");
        });
    });
}

// 这样就不会死锁了
- (void)test2 {
    NSLog(@"在主线程添加一个串行队列queue");
    dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t queue2 = dispatch_queue_create("test", DISPATCH_QUEUE_SERIAL);
    dispatch_async(queue, ^{
        NSLog(@"在串行队列queue2中添加一个同步任务");
        dispatch_sync(queue2, ^{
            NSLog(@"OK");
        });
    });
}
```



### 什么是信号量（dispatch_semaphore）？

就是一种可用来控制访问资源的数量的标识，设定了一个信号量，在线程访问之前，加上信号量的处理，则可告知系统按照我们指定的信号量数量来执行多个线程。

```objc
// 这value是初始化多少个信号量
dispatch_semaphore_create(long value);
// 这个方法是P操作对信号量 -1 ，dsema这个参数表示对哪个信号量进行减一，如果该信号量为0则等待，timeout这个参数则是传入等待的时长。
dispatch_semaphore_wait(dispatch_semaphore_t dsema, dispatch_time_t timeout);
// 这个方法是V操作对信号量 +1 ，dsema这个参数表示对哪个信号量进行加一
dispatch_semaphore_signal(dispatch_semaphore_t dsema);
```

注意，正常的使用顺序是先降低然后再提高，这两个函数通常成对使用。

```objc
- (void)getToken {
    // 以上请求的设置忽略
    NSURLSessionDataTask *task = [mySession dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (data) {
            NSLog(@"get Token");
            // 拿到token，传给request请求做参数
            [self request:token];
        } else {
            NSLog(@"token error:%@", error.description);
        }
    }];
    [task resume];
}

- (void)request:(NSString *)params {
    // 请求的设置忽略
    NSURLSessionDataTask *task = [mySession dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (data) {
            NSLog(@"request success");
        } else {
            NSLog(@"request error:%@----", error.description);
        }
    }];
    [task resume];
}

// 指定调用
- (IBAction)buttonPress:(UIButton *)sender {
    // 创建一个并行队列
    dispatch_queue_t queque = dispatch_queue_create("GoyakodCreated", DISPATCH_QUEUE_CONCURRENT);
    // 异步执行
    dispatch_async(queque, ^{
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
        [self getToken:semaphore];
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        [self request];
    });

    NSLog(@"main thread");
}

- (void)getToken:(dispatch_semaphore_t)semaphore {
    // 以上请求的设置忽略
    NSURLSessionDataTask *task = [mySession dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (data) {
            NSLog(@"get Token");
           // 成功拿到token，发送信号量:
            dispatch_semaphore_signal(semaphore);
        } else {
            NSLog(@"token error:%@", error.description);
        }
    }];
    [task resume];
}
```

<!-- ** -->



## 应用能力

### 如何实现多个网络请求完成后执行下一步？

#### GCD 设置栅栏方式

```objc
dispatch_queue_t queue = dispatch_queue_create("test", DISPATCH_QUEUE_CONCURRENT);
dispatch_group_t group = dispatch_group_create();
for (int i = 0; i < 10; i++) {
    dispatch_group_async(group, queue, ^{
        sleep(1);
        NSLog(@"网络请求   ---- %d",i);
    });
}
dispatch_barrier_sync(queue, ^{
    NSLog(@"主线程刷新UI");
});
```

#### NSOperationQueue 设置栅栏方式

```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
for (int i = 0; i < 10; i++) {
    NSOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        sleep(1);
        NSLog(@"网络请求   ---- %d",i);
    }];
    [queue addOperation:op];
}
[queue addBarrierBlock:^{
    NSLog(@"刷新页面");
}];
```

#### GCD 线程组方式

```objc
dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_CONCURRENT);
dispatch_group_t group = dispatch_group_create();
for (int i = 0; i < 10; i++) {
    dispatch_group_async(group, queue, ^{
        NSLog(@"网络请求   ---- %d",i);
    });
}
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    NSLog(@"刷新页面");
});
```

#### NSOperation 设置依赖方式

```objc
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
NSOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
    sleep(1);
    NSLog(@"下载图片1");
}];
NSOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
    sleep(1);
    NSLog(@"下载图片2");
}];
NSOperation *combine = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"合成图片");
}];
[combine addDependency:op1];
[combine addDependency:op2];

[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:combine];
```


### 如何进行线程通信？

#### NSThread 方式

```objc
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(nullable id)arg waitUntilDone:(BOOL)wait;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(nullable id)arg waitUntilDone:(BOOL)wait API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
- (void)performSelectorInBackground:(SEL)aSelector withObject:(nullable id)arg API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
```

<!-- ** -->

#### GCD 方式

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // 在这里执行耗时操作
    dispatch_async(dispatch_get_main_queue(), ^{
         // 回到主线程，执行UI刷新操作
    });
});
```
```objc
// 线程延迟调用 通信
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    NSLog(@"## 在主线程延迟5秒调用 ##");
});
```

#### NSOperationQueue 方式

```objc
[[NSOperationQueue new] addOperationWithBlock:^{
    NSLog(@"子线程下载: %@", NSOperationQueue.currentQueue);
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        NSLog(@"主线程刷新UI: %@", NSOperationQueue.currentQueue);
    }];
}];
```




### 如何控制最大并发数？

#### NSOperationQueue

NSOperationQueue 提供了 `maxConcurrentOperationCount` 属性设置最大并发数（该属性需要在任务添加到队列中之前进行设置）其默认值是 `-1`；如果值设为 `0`，那么不会执行任何任务；如果值设为 `1`，那么该队列是串行的；如果大于 `1`，那么是并行的。

#### GCD

GCD 可以用信号量来实现最大并发数：

```objc
dispatch_queue_t workQueue = dispatch_queue_create("workQueue", DISPATCH_QUEUE_CONCURRENT);
dispatch_queue_t waitQueue = dispatch_queue_create("waitQueue",DISPATCH_QUEUE_SERIAL);
dispatch_semaphore_t semaphore = dispatch_semaphore_create(10);

for (NSInteger i = 0; i < 10; i++) {
  dispatch_async(waitQueue, ^{
      dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
      dispatch_async(workQueue, ^{
          NSLog(@"thread-info: %@开始执行任务%d", [NSThread currentThread], (int)i);
          sleep(1);
          NSLog(@"thread-info: %@结束执行任务%d", [NSThread currentThread], (int)i);
          dispatch_semaphore_signal(semaphore);
      });
  });
}
```

### 如何处理线程安全（资源竞争）问题？

在 iOS 上进行多线程开发，为了保证线程安全，防止资源竞争，需要给线程加锁，通常用到的线程锁分为<u>7</u>种：信号量、互斥锁、自旋锁、递归锁、条件锁、读写锁、分布式锁。

#### 1. 信号量

dispatch_semaphore 是 GCD 用来同步的一种方式，`dispatch_semephore_create` 方法用户创建一个 `dispatch_semephore_t` 类型的信号量，初始的参数必须大于0，该参数用来表示该信号量有多少个信号，简单的说也就是同事允许多少个线程访问。

`dispatch_semaphore_wait` 方法是等待一个信号量，该方法会判断 signal 的信号值是否大于0，如果大于0则不会阻塞线程，消耗点一个信号值，执行后续任务。如果信号值等于0那么就和 NSCondition 一样，阻塞当前线程进入等待状态，如果等待时间未超过 timeout 并且 `dispatch_semaphore_signal` 释放了一个信号值，那么就会消耗掉一个信号值并且向下执行。如果期间一直不能获得信号量并且超过超时时间，那么就会自动执行后续语句。

```objc
- (void)semaphoreSync {
    NSLog(@"semaphore---begin");
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    // 创建初始信号量为0，阻塞所有线程
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    __block int number = 0;
    dispatch_async(queue, ^{
        // 追加任务A
        [NSThread sleepForTimeInterval:2];              // 模拟耗时操作
        NSLog(@"1---%@",[NSThread currentThread]);      // 打印当前线程
        number = 100;
        // 执行完线程，信号量加 1，信号总量从 0 变为 1
        dispatch_semaphore_signal(semaphore);
    });
    // 原任务B
    // 若计数为0则一直等待，直到接到总信号量变为 >0 ，继续执行后续代码
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"semaphore---end,number = %d",number);
}
```

#### 2. 互斥锁

互斥锁的实现原理与信号量非常相似，不是使用忙等，而是阻塞线程并睡眠，需要进行上下文切换。当一个线程获得这个锁之后，其他想要获得此锁的线程将会被阻塞，直到该锁被释放。当临界区加上互斥锁以后，其他的调用方不能获得锁，只有当互斥锁的持有方释放锁之后其他调用方才能获得锁。调用方在获得锁的时候发现互斥锁已经被其他方持有，那么该调用方只能进入睡眠状态，这样不会占用 CPU 资源。但是会有时间的消耗，系统的运行时基于 CPU 时间调度的，每次线程可能有 100ms 的运行时间，频繁的 CPU 切换也会消耗一定的时间。

**NSLock**

NSLock 遵循 NSLocking 协议，同时也是互斥锁，提供了 `lock` 和 `unlock` 方法来进行加锁和解锁。NSLock 内部是封装了 `pthread_mutext`，类型是 `PTHREAD_MUTEXT_ERRORCHECK`，它会损失一定的性能换来错误提示。

**@synchronized**

这其实是一个 OC 层面的锁，防止不同的线程同时执行同一段代码,相比于使用 NSLock 创建锁对象、加锁和解锁来说，@synchronized 用着更方便，可读性更高。
大体上，想要明白 @synchronized，需要知道在 @synchronized 中 `objc_sync_enter` 和 `objc_sync_exit` 的成对调用，而且每个传入的对象，都会为其分配一个递归锁并存储在哈希表中。在 `objc_sync_enter` 中加锁，在 `objc_sync_exit` 中解锁。

```objc
@synchronized(self) {  
    // 数据操作  
}
```

**os_unfair_lock**

os_unfair_lock 是苹果官方推荐的替换自旋锁 OSSpinLock 的方案，用于解决 OSSpinLock 优先级反转问题，但是它在 iOS10.0 以上的系统才可以调用。

#### 3. 自旋锁

自旋锁的目的是为了确保临界区只有一个线程可以访问。当一个线程获得锁之后，其他线程将会一直循环在哪里查看是否该锁被释放，是用于多线程同步的一种锁，线程反复检查锁变量是否可用。由于线程在这一过程中保持执行，因此是一种忙等待。一旦获取了自旋锁，线程会一直保持该锁，直至显式释放自旋锁。自旋锁避免了进程上下文的调度开销，因此对于线程只会阻塞很短时间的场合是有效的。由于调用方会一直循环看该自旋锁的的保持者是否已经释放了资源，所以总的效率来说比互斥锁高。但是自旋锁只用于短时间的资源访问，如果不能短时间内获得锁，就会一直占用着CPU，造成效率低下。

**OSSpinLock**

自旋锁的一种，由于在某些场景下不安全已被弃用。（使用互斥锁 os_unfair_lock 来代替）

#### 4. 递归锁

需要使用递归锁的情况：

```objc
NSLock *lock = [[NSLock alloc] init];
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    static void (^RecursiveLock)(int);
    RecursiveLock = ^(int value) {
        [lock lock];
        if (value > 0) {
            NSLog(@"value = %d", value);
            sleep(2);
            RecursiveLock(value - 1);
        }
        [lock unlock];
    };
    RecursiveLock(5);
});
```

递归锁也是通过 `pthread_mutex_lock` 函数来实现，在函数内部会判断锁的类型，如果显示是递归锁，就允许递归调用，仅仅将一个计数器加一，锁的释放过程也是同理。一个锁可以被同一线程多次请求，而不会引起死锁。这主要是用在循环或递归操作中。

**NSRecursiveLock**

NSRecursiveLock 与 NSLock 的区别在于内部封装的 `pthread_mutex_t` 对象的类型不同，前者的类型为 `PTHREAD_MUTEX_RECURSIVE`。

```objc
NSRecursiveLock *lock = [[NSRecursiveLock alloc] init]; // 创建递归锁
[lock lockBeforeDate:date]; // 在给定的时间之前去尝试请求一个锁
[lock tryLock]; // 尝试去请求一个锁，并会立即返回一个布尔值，表示尝试是否成功
```

#### 5. 条件锁

**NSCondition**

封装了一个互斥锁和信号量，它把前者的 lock 以及后者的 wait/signal 统一到 NSCondition 对象中，是基于条件变量pthread_cond_t来实现的，和信号量相似，如果当前线程不满足条件，那么就会进入睡眠状态，等待其他线程释放锁或者释放信号之后，就会唤醒线程。
NSCondition 的对象实际上作为一个锁和一个线程检查器：锁主要为了当检测条件时保护数据源，执行条件引发的任务；线程检查器主要是根据条件决定是否继续运行线程，即线程是否被阻塞。

- NSCondition 同样实现了 NSLocking 协议，所以它和 NSLock 一样，也有 NSLocking 协议的 lock 和 unlock 方法，可以当做 NSLock 来使用解决线程同步问题，用法完全一样。
- NSCondition 提供了 wait 和 signal，和条件信号量类似。比如我们要监听 array 数组的个数，当 array 的个数大于0的时候就执行清空操作。思路是这样的，当 array 个数大于0时执行清空操作，否则，wait 等待执行清空操作。当 array 个数增加的时候发生 signal 信号，让等待的线程唤醒继续执行。
- NSCondition 可以给每个线程分别加锁，加锁后不影响其他线程进入临界区。但是正是因为这种分别加锁的方式，NSCondition 使用 wait 并使用加锁后并不能真正的解决资源的竞争。

**NSCoditionLock**

NSConditionLock 也可以像 NSCondition 一样做多线程之间的任务等待调用，而且是线程安全的。
NSConditionLock 同样实现了 NSLocking 协议，性能比较低。


#### 6. 读写锁

读写锁，在对文件进行操作的时候，写操作是排他的，一旦有多个线程对同一个文件进行写操作，后果不可估量，但读是可以的，多个线程读取时没有问题的。

**pthread_rwlock**

读写锁可以有三种状态：
- 读模式下加锁状态，
- 写模式下加锁状态，
- 不加锁状态。

一次只有一个线程可以占有写模式的读写锁，但是多个线程可用同时占有读模式的读写锁。读写锁也叫做共享-独占锁，当读写锁以读模式锁住时，它是以共享模式锁住的，当它以写模式锁住时，它是以独占模式锁住的。因此：
- 当读写锁被一个线程以读模式占用的时候，写操作的其他线程会被阻塞，读操作的其他线程还可以继续进行。
- 当读写锁被一个线程以写模式占用的时候，写操作的其他线程会被阻塞，读操作的其他线程也被阻塞。


#### 7. 分布式锁

NSDistributedLock 分布锁，文件方式实现，可以跨进程 用 tryLock 方法获取锁。 用 unlock 方法释放锁。如果一个获取锁的进行在释放锁之前挂了，那么锁就一直得不到释放了，此时可以通过 breakLock 强行获取锁。

> 注：这种锁在 iOS 上基本用不到，不过多探究。



## 底层原理

### GCD 执行原理？

GCD 有一个底层线程池，这个池中存放的是一个个的线程。之所以称为“池”，很容易理解出这个“池”中的线程是可以重用的，当一段时间后这个线程没有被调用，这个线程就会被销毁。注意：开多少条线程是由底层线程池决定的（线程建议控制再3~5条），池是系统自动来维护，不需要我们程序员来维护，我们只关心的是向队列中添加任务，队列调度即可。

如果队列中存放的是同步任务，则任务出队后，底层线程池中会提供一条线程供这个任务执行，任务执行完毕后这条线程再回到线程池。这样队列中的任务反复调度，因为是同步的，所以当我们用 `currentThread` 打印的时候，就是同一条线程。

如果队列中存放的是异步的任务，（注意异步可以开线程），当任务出队后，底层线程池会提供一个线程供任务执行，因为是异步执行，队列中的任务不需等待当前任务执行完毕就可以调度下一个任务，这时底层线程池中会再次提供一个线程供第二个任务执行，执行完毕后再回到底层线程池中。

这样就对线程完成一个复用，而不需要每一个任务执行都开启新的线程，也就从而节约的系统的开销，提高了效率。



<br>

> **参考资料**
> - [iOS开发基础——线程安全（线程锁）](https://juejin.im/post/5c6a17136fb9a049e660cb20)
