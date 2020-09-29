# 调试与发布

## 常见异常


### BAD_ACCESS 在什么情况下出现？ 如何调试？

访问了野指针，比如对一个已经释放的对象执行了 release、访问已经释放对象的成员变量或者发消息。

#### EXC_BAD_ACCESS 的本质

在 C 和 Objective-C 中，指针是存储另一个变量的内存地址的变量。向一个对象发送消息时，指向该对象的指针将会被引用。这意味着，获取了指针所指的内存地址，并访问该存储区域的值。

当该存储器区域不再映射到您的应用时，或者换句话说，该内存区域在你认为使用的时候却没有使用，该内存区域是无法访问的。 这时内核会抛出一个异常（EXC），表明你的应用程序不能访问该存储器区域（BAD ACCESS）。

简而言之，碰到 EXC_BAD_ACCESS，这意味着你试图发送消息到的内存块，但内存块无法执行该消息。但是，在某些情况下，EXC_BAD_ACCESS 是由被损坏的指针引起的。每当你的应用程序尝试引用损坏的指针，一个异常就会被内核抛出。

#### 调试方法

启用僵尸对象，这意味着被释放的对象将会以僵尸的形式被保留。通过保留已释放的对象，Xcode 可以告诉你你试图访问哪个对象，这使的查找问题原因容易得多。

单击左上角的项目名，并选中 <kbd>Edit Schemes</kbd>。在左侧选中 <kbd>Run</kbd>，在上方打开 <kbd>Diagnostics</kbd> 选项。要启用僵尸对象，勾选 <kbd>Zombie Objects</kbd> 选框。


在 Xcode 中启用僵尸对象：
![](https://gitee.com/xaoxuu/cdn-assets/raw/master/blog/2020-0406a@2x.jpg)




### 什么时候会报 unrecognized selector 的异常？ 如何解决？

当调用该对象上某个方法，而该对象上没有实现这个方法的时候，会报 unrecognized selector 的异常。

#### 解决方法

可以通过“消息转发”进行解决。objc 在向一个对象发送消息时，runtime 库会根据对象的 isa 指针找到该对象实际所属的类，然后在该类中的方法列表以及其父类方法列表中寻找方法运行，如果，在最顶层的父类中依然找不到相应的方法时，程序在运行时会挂掉并抛出异常 `unrecognized selector sent to xxx` 。但是在这之前，objc 的运行时会给出三次拯救程序崩溃的机会：

1. Method resolution

  objc 运行时会调用 `+resolveInstanceMethod:` 或者 `+resolveClassMethod:`，让你有机会提供一个函数实现。如果你添加了函数，那运行时系统就会重新启动一次消息发送的过程，否则，运行时就会移到下一步，消息转发（Message Forwarding）。

2. Fast forwarding

  如果目标对象实现了 `-forwardingTargetForSelector:`，Runtime 这时就会调用这个方法，给你把这个消息转发给其他对象的机会。
只要这个方法返回的不是 nil 和 self，整个消息发送的过程就会被重启，当然发送的对象会变成你返回的那个对象。否则，就会继续 Normal Fowarding。
这里叫Fast，只是为了区别下一步的转发机制。因为这一步不会创建任何新的对象，但下一步转发会创建一个NSInvocation对象，所以相对更快点。

3. Normal forwarding

 这一步是 Runtime 最后一次给你挽救的机会。首先它会发送 `-methodSignatureForSelector:` 消息获得函数的参数和返回值类型。如果 `-methodSignatureForSelector:` 返回 nil，Runtime 则会发出 `-doesNotRecognizeSelector:` 消息，程序这时也就挂掉了。如果返回了一个函数签名，Runtime 就会创建一个 NSInvocation 对象并发送 `-forwardInvocation:` 消息给目标对象。



### 宏定义只有 debug 没有 release 会怎样？

#### 现象

编译、运行都没有错误，但是项目打包失败，也没有失败原因、错误提示。

#### 原因

由于疏忽，在写宏定义的时候，只在 `#ifdef DEBUG` 里定义，而忘了在 `#else` 里面定义。正确的写法应该是 `#ifdef DEBUG` 和 `#else` 里面都要有：

```
#ifdef DEBUG
#define AXLogError(NSError) NSLog((@"\n➤ func:%s " "line:%d" "\n🔴error: \n%@" "\n\n"), __FUNCTION__, __LINE__, NSError.description)
#else
#define AXLogError(NSError)
#endif
```



## 调试技巧

### LLDB 常用的调试命令？

- `breakpoint` 设置断点定位到某一个函数
- `n` 断点指针下一步
- `po` print object 的缩写，表示显示对象的文本描述，如果对象不存在则打印 nil
- `p` 可以用来打印基本数据类型
- `call` 执行一段代码，如：`call NSLog(@"%@", @"yang")`
- `expr` 动态执行指定表达式
- `bt` 打印当前线程堆栈信息（`bt all` 打印所有线程堆栈信息）
- `image` 常用来寻找栈地址对应代码位置，如：`image lookup --address 0xxxx`

> 更多 lldb（gdb） 调试命令可查看
> 1. [ ***The LLDB Debugger*** ](http://lldb.llvm.org/lldb-gdb.html)；
> 2. 苹果官方文档：[ ***iOS Debugging Magic*** ](https://developer.apple.com/library/ios/technotes/tn2239/_index.html)。



### 断点调试

条件断点

打上断点之后，对断点进行编辑，设置相应过滤条件。下面简单的介绍一下条件设置：

Condition：返回一个布尔值，当布尔值为真触发断点，一般里面我们可以写一个表达式。

Ignore：忽略前N次断点，到N+1次再触发断点。

Action：断点触发事件，分为六种：

AppleScript：执行脚本。

Capture GPU Frame：用于OpenGL ES调试，捕获断点处GPU当前绘制帧。

Debugger Command：和控制台中输入LLDB调试命令一致。

Log Message：输出自定义格式信息至控制台。

Shell Command：接收命令文件及相应参数列表，Shell Command是异步执行的，只有勾选“Wait until done”才会等待Shell命令执行完在执行调试。

Sound：断点触发时播放声音。

Options(Automatically continue after evaluating actions选项)：选中后，表示断点不会终止程序的运行。

异常断点

异常断点可以快速定位不满足特定条件的异常，比如常见的数组越界，这时候很难通过异常信息定位到错误所在位置。这个时候异常断点就可以发挥作用了。

Exception：可以选择抛出异常对象类型：OC或C++。

Break：选择断点接收的抛出异常来源是Throw还是Catch语句。

符号断点

符号断点的创建方式和异常断点一样一样的，在符号断点中可以指定要中断执行的方法：

Symbol:[类名 方法名]可以执行到指定类的指定方法中开始断点。



### 如何使用 Instruments 工具检查内存泄漏？


打开 Instruments：

![](https://cdn.jsdelivr.net/gh/xaoxuu/cdn-assets/2018/827fb4fa7c3203ca4690109766ae23754c1f0c.jpg)

内存泄漏时：

![](https://cdn.jsdelivr.net/gh/xaoxuu/cdn-assets/2018/88e108233106f8172dc603758739887ec29925.jpg)
