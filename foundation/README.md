# Foundation

## 基本概念

### 类方法和实例方法有什么本质区别和联系？

**类方法**

1. 类方法是属于类对象的
2. 类方法只能通过类对象调用
3. 类方法中的 self 是类对象
4. 类方法可以调用其他的类方法
5. 类方法中不能访问成员变量
6. 类方法中不能直接调用对象方法

**实例方法**

1. 实例方法是属于实例对象的
2. 实例方法只能通过实例对象调用
3. 实例方法中的 self 是实例对象
4. 实例方法中可以访问成员变量
5. 实例方法中直接调用实例方法
6. 实例方法中也可以通过类名调用类方法


### nil、NIL、NSNULL 有什么区别？

- nil、NIL 可以说是等价的，都代表内存中一块空地址。
- NSNULL 代表一个指向 nil 的对象。


### 如何实现一个线程安全的 NSMutableArray？

NSMutableArray 是线程不安全的，当有多个线程同时对数组进行操作的时候可能导致崩溃或数据错误。

**线程锁**

使用线程锁对数组读写时进行加锁：

```objc
- (void)addObject:(NSObject *)obj {
    @synchronized(self) {
        [_array addObject:obj];
    }
}
```

**GCD**

使用串行同步队列将读取操作及写入操作都安排在同一个队列里，即可保证数据同步。而通过并发队列，结合 GCD 的栅栏块（barrier）来不仅实现数据同步线程安全，还比串行同步队列方式更高效。

### 实现 isEqual 和 hash 方法时要注意什么？

**hash**

- 对关键属性的 hash 值进行位或运算作为 hash 值

**isEqual**

- 使用 `==` 运算符判断是否是同一对象，因为同一对象必然完全相同
- 判断是否是同一类型，这样不仅可以提高判等的效率，还可以避免隐式类型转换带来的潜在风险
- 判断对象是否是 nil，做参数有效性检查
- 各个属性分别使用默认判等方法进行判断
- 返回所有属性判等的与结果




### id 和 instancetype 有什么区别？

#### 相同点

- id 和 instancetype 都是万能指针，指向对象。

#### 不同点

- id 在编译的时候不能判断对象的真实类型，instancetype 在编译的时候可以判断对象的真实类型。
- id 可以用来定义变量，可以作为返回值类型，可以作为形参类型；instancetype 只能作为返回值类型。




### self 和 super 的区别

#### 基本区别

- self 调用自己方法，super 调用父类方法

- self 是类，super 是预编译指令

- `[self class]` 和 `[super class]` 输出是一样的

#### 底层区别

- 当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找；而当使用 super 时，则从父类的方法列表中开始找，然后调用父类的这个方法。

- 当使用 self 调用时，会使用 `objc_msgSend` 函数：
  ```c
  id objc_msgSend(id theReceiver, SEL theSelector, ...)
  ```

  第一个参数是消息接收者，第二个参数是调用的具体类方法的 selector ，后面是 selector 方法的可变参数。以 `[self setName:]` 为例，编译器会替换成调用 `objc_msgSend` 的函数调用，其中 `theReceiver` 是 self， `theSelector` 是 `@selector(setName:)` ，这个 selector 是从当前 self 的 class 的方法列表开始找的 `setName` ，当找到后把对应的 selector 传递过去。

- 当使用 super 调用时，会使用 `objc_msgSendSuper` 函数：
  ```c
  id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
  ```

  第一个参数是个 `objc_super` 的结构体，第二个参数还是类似上面的类方法的 selector
  ```c
  struct objc_super {
  	id receiver;
  	Class superClass;
  };
  ```




### @synthesize 和 @dynamic 分别有什么作用？

- `@property` 有两个对应的词，一个是 `@synthesize` ，一个是 `@dynamic` 。如果 `@synthesize` 和 `@dynamic` 都没写，那么默认的就是 `@syntheszie var = _var` 。

- `@synthesize` 的语义是如果你没有手动实现 `setter` 方法和 `getter` 方法，那么编译器会自动为你加上这两个方法。

- `@dynamic` 告诉编译器：属性的 `setter` 与 `getter` 方法由用户自己实现，不自动生成。（当然对于 `readonly` 的属性只需提供 `getter` 即可）。假如一个属性被声明为 `@dynamic var` ，然后你没有提供 `setter` 方法和 `getter` 方法，编译的时候没问题，但是当程序运行到 `instance.var = someVar` ，由于缺 `setter` 方法会导致程序崩溃；或者当运行到 `someVar = var` 时，由于缺 `getter` 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。




### __typeof__ / __typeof / typeof 的区别？

这三个意思是相同的，但没有一个是标准 C，不同的编译器会按需选择符合标准的写法。 `__typeof__()` 和 `__typeof()` 是 C 语言的编译器特定扩展，因为标准 C 不包含这样的运算符。 标准 C 要求编译器用双下划线前缀语言扩展（这也是为什么你不应该为自己的函数，变量等做这些） `typeof()` 与前两者完全相同的，只不过去掉了下划线，同时现代的编译器也可以理解。





### struct 和 class 的区别

- struct 结构体：值类型（实例直接位于栈中）

- class 类：引用类型（位于栈上面的指针（引用）和位于堆上的实体对象）


## KVC & KVO

### KVC 的实现原理

KVC，键-值编码，使用字符串直接访问对象的属性。底层实现，当一个对象调用 `setValue` 方法时，方法内部会做以下操作：

1. 检查是否存在相应 `key` 的 `set` 方法，如果存在，就调用 `set` 方法。

2. 如果 `set` 方法不存在，就会查找与 `key` 相同名称并且带下划线的成员变量，如果有，则直接给成员变量赋值。

3. 如果没有找到 `_key`，就会查找相同名称的属性 `key`，如果有就直接赋值。

4. 如果还没找到，则调用 `valueForUndefinedKey:` 和 `setValue: forUndefinedKey:` 方法。




### KVO 的实现原理

KVO 是键-值观察机制，原理如下：

1. 当给 A 类添加 KVO 的时候，runtime 动态的生成了一个子类 `NSKVONotifying_A`，让 A 类的 isa 指针指向 `NSKVONotifying_A` 类，重写 class 方法，隐藏对象真实类信息

2. 重写监听属性的 setter 方法，在 setter 方法内部调用了 Foundation 的 `_NSSetObjectValueAndNotify` 函数

3. `_NSSetObjectValueAndNotify` 函数内部

  3.1 首先会调用 `willChangeValueForKey`
  3.2 然后给属性赋值
  3.3 然后调用 `didChangeValueForKey`
  3.4 最后调用 observer 的 `observeValueForKeyPath` 去告诉监听器属性值发生了改变

4. 重写了 `dealloc` 做一些 KVO 内存释放




### 如何手动触发 KVO 方法？

手动调用 `willChangeValueForKey` 和 `didChangeValueForKey` 两个方法。

键值观察通知依赖于 NSObject 的两个方法：`willChangeValueForKey:` 和 `didChangeValueForKey:`。在一个被观察属性发生改变之前， `willChangeValueForKey:` 一定会被调用，这就会记录旧的值。而当改变发生后，`didChangeValueForKey:` 会被调用，继而 `observeValueForKey:ofObject:change:context:` 也会被调用。如果可以手动实现这些调用，就可以实现“手动触发”了。

!> 只调用 `didChangeValueForKey:` 方法不可以触发 KVO 方法。



### KVC 的 keyPath 中的集合运算符如何使用？

1. 必须用在集合对象上或普通对象的集合属性上
2. 简单集合运算符有 `@avg`、`@count`、`@max`、`@min`、`@sum`
3. 格式 `@"@sum.age"` 或 `@"集合属性.@max.age"`


## 属性

### 属性中有哪些属性关键字？

- 非原子的 `nonatomic`
- 读/写权限 `readwrite(读写)`、`readonly (只读)`
- 内存管理语义 `assign`、`strong`、 `weak`、`unsafe_unretained`、`copy`
- 方法名 `getter=<name>` 、`setter=<name>`

  ```objc 例如
  @property (nonatomic, getter=isOn) BOOL on;
  ```

**setter 一般用在特殊的情境下**

在数据反序列化、转模型的过程中，服务器返回的字段如果以 `init` 开头，所以你需要定义一个 `init` 开头的属性，但默认生成的 `setter` 与 `getter` 方法也会以 `init` 开头，而编译器会把所有以 `init` 开头的方法当成初始化方法，而初始化方法只能返回 self 类型，因此编译器会报错。

这时你就可以使用下面的方式来避免编译器报错：

```objc
@property(nonatomic, strong, getter=p_initBy, setter=setP_initBy:)NSString *initBy;
```

另外也可以用关键字进行特殊说明，来避免编译器报错：

```objc
@property(nonatomic, readwrite, copy, null_resettable) NSString *initBy;
- (NSString *)initBy __attribute__((objc_method_family(none)));
```



- 非空/可空类型 `nonnull`、`null_resettable`、`nullable`
- 类属性 `class`




### ARC 下，不显式指定任何属性关键字时，默认的关键字都有哪些？

- 基本数据类型默认关键字是 `atomic`, `readwrite`, `assign`

- 普通的 ObjC 对象默认关键字是 `atomic`, `readwrite`, `strong`



### 属性的本质是什么？ ivar、getter、setter 是如何生成并添加到这个类中的？

#### 属性的本质

属性的本质是实例变量（`ivar`）+ 存取方法（`getter`、`setter`）

属性（property）作为 Objective-C 的一项特性，主要的作用就在于封装对象中的数据。 Objective-C 对象通常会把其所需要的数据保存为各种实例变量。实例变量一般通过“存取方法”来访问。

#### ivar、getter、setter 是如何生成并添加到这个类中的？

完成属性定义后，编译器会自动编写访问这些属性所需的方法，此过程叫做“自动合成”（autosynthesis）。需要强调的是，这个过程由编译器在编译期执行，所以编辑器里看不到这些“合成方法”的源代码。除了生成方法代码 getter、setter 之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字，也可以在类的实现代码里通过 `@synthesize` 语法来指定实例变量的名字。



### atomic 修饰的属性是绝对安全的吗？为什么？

不是，所谓的安全只是局限于 Setter、Getter 的访问器方法而言的，你对它做 Release 的操作是不会受影响的。这个时候就容易崩溃了。


### iOS 开发中，对象属性不写 nonatomic 会有什么后果？

**会默认使用 atomic 属性，影响性能。**

该属性使用了自旋锁，会在创建时生成一些额外的代码用于帮助编写多线程程序，这会带来性能问题，通过声明 nonatomic 可以节省这些虽然很小但是不必要额外开销。

一般情况下并不要求属性必须是“原子的”，因为这并不能保证线程安全，若要实现线程安全的操作，还需采用更为深层的锁定机制才行。例如，一个线程在连续多次读取某属性值的过程中有别的线程在同时改写该值，那么即便将属性声明为 atomic，也还是会读到不同的属性值。

因此，开发 iOS 程序时一般都会使用 nonatomic 属性。




### protocol 和 category 中如何使用 @property

在 protocol 中使用 property 只会生成 setter 和 getter 方法声明，我们使用属性的目的，是希望遵守我协议的对象能实现该属性。

在 category 使用 property 也是只会生成 setter 和 getter 方法的声明，如果我们真的需要给 category 增加属性的实现，需要借助于运行时的两个函数：`objc_setAssociatedObject` 和 `objc_getAssociatedObject` 来实现。




### @synthesize 有哪些使用场景？

1. 同时重写了 setter 和 getter 时
2. 重写了只读属性的 getter 时
3. 使用了 @dynamic 时
4. 在 @protocol 中定义的所有属性
5. 在 category 中定义的所有属性
6. 重载的属性




### 若一个类有实例变量 NSString *_foo ，调用 setValue:forKey: 时，可以以 foo 还是 _foo 作为 key？

都可以。


## 值的传递

### 通知和代理有什么区别？

- 通知是观察者模式，适合一对多的场景

- 代理模式适合一对一的反向传值

- 通知耦合度低，代理的耦合度高




### block 和 delegate 的区别

**delegate 运行成本低，block 的运行成本高**

block 出栈需要将使用的数据从栈内存拷贝到堆内存，当然对象的话就是加计数，使用完或者 block 置 nil 后才消除。delegate 只是保存了一个对象指针，直接回调，没有额外消耗。

#### delegate

代理更注重过程信息的传输：比如发起一个网络请求，可能想要知道此时请求是否已经开始、是否收到了数据、数据是否已经接受完成、数据接收失败。这也是很常用的方式，特点是一对一的形式，而且逻辑结构非常清晰。实现起来较为简单：写协议，设置代理这个属性，然后在你想通知代理做事情的方法中调用即可。当然这里面有一些细节，包括：
- 协议定义时，请用关键字 `@required`，和 `@optional` 来明确代理是否必须实现某些方法。
- 代理的类型需用 id 类型，并写明要遵守的协议。
- 就是在调用代理方法的时候需要判断代理是否实现该方法。


#### block

block 注重结果的传输：比如对于一个事件，只想知道成功或者失败，并不需要知道进行了多少或者额外的一些信息。





###  Block 的几种类型

#### 全局 GlobalBlock

- 没有用到外界变量或只用到全局变量、静态变量的 Block
- GlobalBlock 生命周期从创建到应用程序结束
- 不持有对象
- retain, release, copy 操作为空操作


#### 栈类型 StackBlock

- 只用到外部局部变量、成员属性变量，且没有强指针引用的 Block
- StackBlock 的生命周期由系统控制的，超出作用域就会被销毁
- 存储在栈上
- 调用 copy 操作会变成堆类型的 Block


#### 堆类型 MallocBlock

- 有强指针引用或调用 copy 方法的 Block 会被复制一份到堆中成为 MallocBlock
- MallocBlock 的生命周期由程序员控制，没有强指针引用即销毁。
- 调用 copy 操作后，复制效果是：引用计数增加




###  Block 的本质

[iOS-Block本质](https://www.jianshu.com/p/4e79e9a0dd82)



### 在 ARC 环境下，编译器会根据情况自动将栈上的 block 复制到堆上的几种情况？

- block 作为函数返回值时
- block 赋值给 __strong 指针时
- block 作为 Cocoa API 中方法名含有 usingBlock 的方法参数时
- block 作为 GCD API 的方法参数时



### 当 block 内部访问了对象类型的 auto 变量时，是否会强引用？

如果 block 在栈空间，不管外部变量是强引用还是弱引用，block 都会弱引用访问对象
如果 block 在堆空间，如果外部强引用，block 内部也是强引用；如果外部弱引用，block 内部也是弱引用



### ARC 下如何解决 block 循环引用的问题？

三种方式：`__weak`、`__unsafe_unretained`、`__block`

#### __weak

```objc
Person *person = [[Person alloc] init];
__weak typeof(person) weakPerson = person;

person.block = ^{
    NSLog(@"age is %d", weakPerson.age);
};
```

#### __unsafe_unretained

```objc
__unsafe_unretained Person *person = [[Person alloc] init];
person.block = ^{
    NSLog(@"age is %d", weakPerson.age);
};
```

#### __block

```objc
__block Person *person = [[Person alloc] init];
person.block = ^{
    NSLog(@"age is %d", person.age);
    person = nil;
};
person.block();
```

#### 三种方法比较

`__weak`：不会产生强引用，指向的对象销毁时，会自动让指针置为 nil
`__unsafe_unretained`：不会产生强引用，不安全，指向的对象销毁时，指针存储的地址值不变
`__block`：必须把引用对象置位 nil，并且要调用该 block
