# 内存管理

## 关键字


### 什么时候用 copy 关键字？

1. NSString、NSArray、NSDictionary 等等经常使用 copy 关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary
2. block 也经常使用 copy 关键字，具体原因见官方文档：[Objects Use Properties to Keep Track of Blocks](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8-SW12)



### 为什么 block 用 copy 关键字

Block 在没有使用外部变量时，内存存在全局区，然而，当 Block 在使用外部变量的时候，内存是存在于栈区，当 Block copy 之后，是存在堆区的。存在于栈区的特点是对象随时有可能被销毁，一旦销毁在调用的时候，就会造成系统的崩溃。所以 Block 要用 copy 关键字。




### NSString 为什么要用 copy 关键字，如果用 strong 会有什么问题？

1. 因为父类指针可以指向子类对象，使用 copy 的目的是为了让本对象的属性不受外界影响，使用 copy 无论给我传入是一个可变对象还是不可对象，我本身持有的就是一个不可变的副本。
2. 如果我们使用是 strong 的话，这个属性就有可能指向一个可变对象，如果这个可变对象在外部被修改了，那么会影响该属性。

copy 此特质所表达的所属关系与 strong 类似。然而设置方法并不保留新值，而是将其“拷贝” (copy)。
当属性类型为 NSString 时，经常用此特质来保护其封装性，因为传递给设置方法的新值有可能指向一个 NSMutableString 类的实例。这个类是 NSString 的子类，表示一种可修改其值的字符串，此时若是不拷贝字符串，那么设置完属性之后，字符串的值就可能会在对象不知情的情况下遭人更改。所以，这时就要拷贝一份“不可变” (immutable)的字符串，确保对象中的字符串值不会无意间变动。只要实现属性所用的对象是“可变的” (mutable)，就应该在设置新属性值时拷贝一份。

#### 对非集合类对象的 copy 操作

在非集合类对象中：对 immutable 对象进行 copy 操作，是指针复制，mutableCopy 操作是内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。用代码简单表示如下：

```objc
- [immutableObject copy] // 浅复制
- [immutableObject mutableCopy] // 深复制
- [mutableObject copy] // 深复制
- [mutableObject mutableCopy] // 深复制
```

> 用 @property 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。

#### 集合类对象的 copy 与 mutableCopy

集合类对象是指 NSArray、NSDictionary、NSSet ... 之类的对象。下面先看集合类 immutable 对象使用 copy 和 mutableCopy 的一个例子：

```objc
NSArray *array = @[@[@"a", @"b"], @[@"c", @"d"]];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];
```

查看内容，可以看到 copyArray 和 array 的地址是一样的，而 mCopyArray 和 array 的地址是不同的。说明 copy 操作进行了指针拷贝，mutableCopy 进行了内容拷贝。但需要强调的是：此处的内容拷贝，仅仅是拷贝 array 这个对象，array 集合内部的元素仍然是指针拷贝。这和上面的非集合 immutable 对象的拷贝还是挺相似的，那么mutable对象的拷贝会不会类似呢？我们继续往下，看 mutable 对象拷贝的例子：

```objc
NSMutableArray *array = [NSMutableArray arrayWithObjects:[NSMutableString stringWithString:@"a"],@"b",@"c",nil];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];
```

查看内存，如我们所料，copyArray、mCopyArray和 array 的内存地址都不一样，说明 copyArray、mCopyArray 都对 array 进行了内容拷贝。同样，我们可以得出结论：

在集合类对象中，对 immutable 对象进行 copy，是指针复制，mutableCopy 是内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。但是：集合对象的内容复制仅限于对象本身，对象元素仍然是指针复制。用代码简单表示如下：

```objc
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] // 单层深复制
[mutableObject copy] // 单层深复制
[mutableObject mutableCopy] // 单层深复制
```

这个代码结论和非集合类的非常相似。

> 参考链接：[iOS 集合的深复制与浅复制](https://www.zybuluo.com/MicroCai/note/50592)



### 用 copy 修饰可变类型会有什么问题？

添加、删除、修改数组内的元素的时候，程序会因为找不到对应的方法而崩溃，因为 copy 就是复制一个不可变 NSArray/NSDictionary/NSString 的对象。



### 什么情况使用 weak 关键字，相比 assign 有什么不同？

#### 什么情况使用 weak 关键字

- 在 ARC 中，在有可能出现循环引用的时候，往往要通过让其中一端使用 weak 来解决，比如: delegate 代理属性

- 自身已经对它进行一次强引用，没有必要再强引用一次，此时也会使用 weak 关键字，自定义 IBOutlet 控件属性一般也使用 weak 关键字；当然，也可以使用 strong 关键字。

#### weak 与 assign 的区别

- weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同 assign 类似，然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。 而 assign 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 NSlnteger 等)的简单赋值操作。

- assign 可以用于非 OC 对象，而 weak 必须用于 OC 对象




### IBOutlet 连出来的视图属性为什么可以被设置成 weak

因为既然有外链那么视图在 xib 或者 storyboard 中肯定存在，视图已经对它有一个强引用了。
使用 storyboard（xib不行）创建的 vc，会有一个叫 `_topLevelObjectsToKeepAliveFromStoryboard` 的私有数组强引用所有 top level 的对象，所以这时即便 outlet 声明成 weak 也没关系。

> 参考链接：[Should IBOutlets be strong or weak under ARC?](http://stackoverflow.com/questions/7678469/should-iboutlets-be-strong-or-weak-under-arc)




### 如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter 方法？

需实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying 与 NSMutableCopying 协议。

#### 具体步骤

1. 声明该类遵从 NSCopying 协议

2. 实现 NSCopying 协议。该协议只有一个方法:

  ```objc
  - (id)copyWithZone:(NSZone *)zone;
  ```

  !> 注意：一提到让自己的类用 copy 修饰符，我们总是想覆写 copy 方法，其实真正需要实现的却是 `copyWithZone` 方法。

3. 重写带 copy 关键字的 setter 方法，例如：

  ```objc
  - (void)setName:(NSString *)name {
      _name = [name copy];
  }
  ```

<!-- ** -->




### 深拷贝与浅拷贝

浅拷贝只是对指针的拷贝，拷贝后两个指针指向同一个内存空间，深拷贝不但对指针进行拷贝，而且对指针指向的内容进行拷贝，经深拷贝后的指针是指向两个不同地址的指针。

当对象中存在指针成员时，除了在复制对象时需要考虑自定义拷贝构造函数，还应该考虑以下两种情形：

- 当函数的参数为对象时，实参传递给形参的实际上是实参的一个拷贝对象，系统自动通过拷贝构造函数实现；
- 当函数的返回值为一个对象时，该对象实际上是函数内对象的一个拷贝，用于返回函数调用处。

`copy` 方法：如果是非可扩展类对象，则是浅拷贝。如果是可扩展类对象，则是深拷贝。

`mutableCopy` 方法：无论是可扩展类对象还是不可扩展类对象，都是深拷贝。





## 内存泄漏

### 经典内存泄漏及解决方案？

#### 僵尸对象和野指针

僵尸对象是指内存已经被回收的对象，指向僵尸对象的指针就是野指针，向野指针发送消息会导致崩溃：

```c
EXC_BAD_ACCESS
```

解决方案：当对象释放后，应该将其置为nil。

#### 循环引用

ARC 时代最常出现的内存管理问题是循环引用，即对象 A 强引用对象 B，对象 B 强引用对象 A，或者多个对象强引用形成一个闭环。block 会对内部使用的对象进行强引用，因此在使用的时候应该确定不会引起循环引用。

解决方案：使用弱引用或主动断开循环。

#### 循环产生的内存峰值

循环内产生大量的临时对象，直至循环结束才释放，可能导致内存泄漏。

解决方案：在循环中创建自己的autoreleasepool，及时释放占用内存大的临时变量，减少内存占用峰值。

```objc
for (int i=0; i<5000; i++) {
    @autoreleasepool {
        NSNumber *num = [NSNumber numberWithInt:i];
        [num performOperationOnNumber];
    }
}
```

<!-- ** -->





### 如何解决循环引用？

循环引用即对象 A 强引用对象 B，对象 B 强引用对象 A，或者多个对象强引用形成一个闭环。block 会对内部使用的对象进行强引用，因此在使用的时候应该确定不会引起循环引用。

解决方案：使用弱引用或主动断开循环。

> 详见这篇文章：[iOS 常见内存泄漏及解决方案](https://xaoxuu.com/blog/2018-10-15-ios-memory-leak/)



### 使用 CADisplayLink、NSTimer 有什么注意点？

CADisplayLink、NSTimer 会造成循环引用，可以使用 YYWeakProxy 或者为 CADisplayLink、NSTimer 添加 block 方法解决循环引用。




### 使用系统的某些 block api（如 UIView 的 block 版本写动画时），是否也考虑引用循环问题？

系统的某些 block api中，UIView 的 block 版本写动画时不需要考虑，但也有一些 api 需要考虑：

所谓“引用循环”是指双向的强引用，所以那些“单向的强引用”（block 强引用 self ）没有问题，比如这些：

```objc
[UIView animateWithDuration:duration animations:^{
    [self.superview layoutIfNeeded];
}];
```

```objc
[[NSOperationQueue mainQueue] addOperationWithBlock:^{
    self.someProperty = xyz;
}];
```

```objc
[[NSNotificationCenter defaultCenter] addObserverForName:@"someNotification"
                                                  object:nil
                           queue:[NSOperationQueue mainQueue]
                                              usingBlock:^(NSNotification *notification) {
                                                    self.someProperty = xyz;
                                                  }
];
```

这些情况不需要考虑“引用循环”。

但如果你使用一些参数中可能含有 ivar 的系统 api ，如 GCD 、NSNotificationCenter 就要小心一点：比如 GCD 内部如果引用了 self，而且 GCD 的其他参数是 ivar，则要考虑到循环引用：

```objc
__weak __typeof__(self) weakSelf = self;
dispatch_group_async(_operationsGroup, _operationsQueue, ^
{
__typeof__(self) strongSelf = weakSelf;
[strongSelf doSomething];
[strongSelf doSomethingElse];
} );
```

类似的：

```objc
  __weak __typeof__(self) weakSelf = self;
  _observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"testKey"
                                                                object:nil
                                                                 queue:nil
                                                            usingBlock:^(NSNotification *note) {
      __typeof__(self) strongSelf = weakSelf;
      [strongSelf dismissModalViewControllerAnimated:YES];
  }];
```

self --> _observer --> block --> self 显然这也是一个循环引用。

> 检测代码中是否存在循环引用问题，可使用 Facebook 开源的一个检测工具 [FBRetainCycleDetector](https://github.com/facebook/FBRetainCycleDetector)




### iOS 内存分区情况

#### 栈区（Stack）

- 存放函数的参数，局部变量的值等。
- 栈是向低地址扩展的数据结构，是一块连续的内存区域。
- 由编译器自动分配释放。

#### 堆区（Heap）

- 是向高地址扩展的数据结构，是不连续的内存区域。
- 由程序员分配释放。

#### 全局区

- 全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域，未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。
- 程序结束后由系统释放。

#### 常量区

- 常量字符串就是放在这里。
- 程序结束后由系统释放。

#### 代码区

- 存放函数体的二进制代码。

> **注意**
> - 在 iOS 中，堆区的内存是应用程序共享的，堆中的内存分配是系统负责的。
> - 系统使用一个链表来维护所有已经分配的内存空间（系统仅仅记录，并不管理具体的内容）。
> - 变量使用结束后，需要释放内存，OC 中是判断引用计数是否为 0，如果是就说明没有任何变量使用该空间，那么系统将其回收。
> - 当一个 app 启动后，代码区、常量区、全局区大小就已经固定，因此指向这些区的指针不会产生崩溃性的错误。而堆区和栈区是时时刻刻变化的（堆的创建销毁，栈的弹入弹出），所以当使用一个指针指向这个区里面的内存时，一定要注意内存是否已经被释放，否则会因为访问了野指针而导致程序崩溃。



## 内存管理

### objc 使用什么机制管理对象内存？

通过 retainCount 的机制来决定对象是否需要释放。
每次 runloop 的时候，都会检查对象的 retainCount，如果 retainCount 为 0，说明该对象没有地方需要继续使用了，可以释放掉了。

### alloc 方法中做了什么

alloc 内部调用 malloc 函数，计算对象所需要的内存空间，分配相应的空间，返回内存空间的首地址。
init 对分配的内存进行初始化。

### iOS 内存管理方式

#### Tagged Pointer

- Tagged Pointer 专门用来存储小的对象，例如 NSNumber 和 NSDate。

- Tagged Pointer 指针的值不再是地址了，而是真正的值。所以，实际上它不再是一个对象了，它只是一个披着对象皮的普通变量而已。所以，它的内存并不存储在堆中。

- `objc_msgSend` 能识别 Tagged Pointer，比如 NSNumber 的 intValue 方法，直接从指针提取数据。

- 使用 Tagged Pointer 后，指针内存储的数据变成了 Tag + Data，也就是将数据直接存储在了指针中。

#### NONPOINTER_ISA

- 苹果将 isa 设计成了联合体，在 isa 中存储了与该对象相关的一些内存的信息，并不需要 64 个二进制位全部都用来存储指针。

```objc
// x86_64 架构
struct {
    uintptr_t nonpointer        : 1;  // 0:普通指针，1:优化过，使用位域存储更多信息
    uintptr_t has_assoc         : 1;  // 对象是否含有或曾经含有关联引用
    uintptr_t has_cxx_dtor      : 1;  // 表示是否有C++析构函数或OC的dealloc
    uintptr_t shiftcls          : 44; // 存放着 Class、Meta-Class 对象的内存地址信息
    uintptr_t magic             : 6;  // 用于在调试时分辨对象是否未完成初始化
    uintptr_t weakly_referenced : 1;  // 是否被弱引用指向
    uintptr_t deallocating      : 1;  // 对象是否正在释放
    uintptr_t has_sidetable_rc  : 1;  // 是否需要使用 sidetable 来存储引用计数
    uintptr_t extra_rc          : 8;  // 引用计数能够用 8 个二进制位存储时，直接存储在这里
};

// arm64 架构
struct {
    uintptr_t nonpointer        : 1;  // 0:普通指针，1:优化过，使用位域存储更多信息
    uintptr_t has_assoc         : 1;  // 对象是否含有或曾经含有关联引用
    uintptr_t has_cxx_dtor      : 1;  // 表示是否有C++析构函数或OC的dealloc
    uintptr_t shiftcls          : 33; // 存放着 Class、Meta-Class 对象的内存地址信息
    uintptr_t magic             : 6;  // 用于在调试时分辨对象是否未完成初始化
    uintptr_t weakly_referenced : 1;  // 是否被弱引用指向
    uintptr_t deallocating      : 1;  // 对象是否正在释放
    uintptr_t has_sidetable_rc  : 1;  // 是否需要使用 sidetable 来存储引用计数
    uintptr_t extra_rc          : 19;  // 引用计数能够用 19 个二进制位存储时，直接存储在这里
};
```

#### SideTables

引用计数要么存放在 isa 的 extra_rc 中，要么存放在引用计数表中，而引用计数表包含在一个叫 SideTable 的结构中，它是一个散列表，也就是哈希表。而 SideTable 又包含在一个全局的 StripeMap 的哈希映射表中，这个表的名字叫 SideTables。

散列表（Hash table，也叫哈希表），是根据建（Key）而直接访问在内存存储位置的数据结构。也就是说，它通过一个关于键值得函数，将所需查询的数据映射到表中一个位置来访问记录，这加快了查找速度。这个映射函数称作散列函数，存放记录的数组称作散列表。

![](https://i.loli.net/2020/04/06/qTR3eXCrbONwu64.png)

当一个对象访问 SideTables 时：

- 首先会取得对象的地址，将地址进行哈希运算，与 SideTables 中 SideTable 的个数取余，最后得到的结果就是该对象所要访问的 SideTable。
- 在取得的 SideTable 中的 RefcountMap 表中再进行一次哈希查找，找到该对象在引用计数表中对应的位置。
- 如果该位置存在对应的引用计数，则对其进行操作，如果没有对应的引用计数，则创建一个对应的 `size_t` 对象，其实就是一个 `uint` 类型的无符号整型。

弱引用表也是一张哈希表的结构，其内部包含了每个对象对应的弱引用表 `weak_entry_t`，而 `weak_entry_t` 是一个结构体数组，其中包含的则是每一个对象弱引用的对象所对应的弱引用指针。

#### 引用计数

引用计数（Reference Count）是一个简单而有效的管理对象生命周期的方式。当我们创建一个新对象的时候，它的引用计数为 1，当有一个新的指针指向这个对象时，我们将其引用计数加 1，当某个指针不再指向这个对象时，我们将其引用计数减 1，当对象的引用计数变为 0 时，说明这个对象不再被任何指针指向了，这个时候我们就可以将对象销毁，回收内存。




### 对象什么时候会被释放？

当对象没有被任何变量引用（也可以说是没有指针指向该对象）的时候，就会被释放。

那怎么知道对象已经没有被引用了呢？OC 采用引用计数来进行管理：

- 每个对象都有一个关联的整数，称为引用计数器
- 当代码需要使用该对象时，则将对象的引用计数加 1
- 当代码结束使用该对象时，则将对象的引用计数减 1
- 当引用计数的值变为 0 时，表示对象没有被任何代码使用，此时对象将被释放。

当不再使用一个对象时应该将其释放，但是在某些情况下，我们很难理清一个对象什么时候不再使用，需要使用 autoreleasepool。对象接收到 autorelease 消息时，它会被添加到了当前的自动释放池中，当自动释放池被销毁时，会給池里所有的对象发送 release 消息。autoreleasepool 一般是在事件处理之前创建，在事件处理之后释放。



### 苹果是如何实现 autoreleasepool 的？

autoreleasepool 以一个队列数组的形式实现，主要通过下列三个函数完成：

```objc
objc_autoreleasepoolPush
objc_autoreleasepoolPop
objc_autorelease
```

看函数名就可以知道，对 autorelease 分别执行 push 和 pop 操作。销毁对象时执行 release 操作。
