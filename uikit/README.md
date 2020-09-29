# UIKit

## 基本控件

### UIView 和 CALayer 是什么关系？

- UIView 继承 UIResponder，而 UIResponder 是响应者对象，可以对 iOS 中的事件响应及传递，CALayer 没有继承自 UIResponder，所以 CALayer 不具备响应处理事件的能力。CALayer 是 QuartzCore 中的类，是一个比较底层的用来绘制内容的类，用来绘制 UI 。

- UIView 对 CALayer 封装属性，对 UIView 设置 frame、center、bounds 等位置信息时，其实都是UIView 对 CALayer 进一层封装，使得我们可以很方便地设置控件的位置。

- UIView 是 CALayer 的代理，UIView 持有一个 CALayer 的属性，并且是该属性的代理，用来提供一些 CALayer 行的数据，例如动画和绘制。

### Bounds 和 Frame 的区别？

- Bounds 一般是相对于自身来说的，是控件的内部尺寸。如果你修改了 Bounds，那么子控件的相对位置也会发生改变。

- Frame 是相对于父控件来说的，是控件的外部尺寸。




### setNeedsDisplay 和 layoutIfNeeded 两者是什么关系？

UIView 的 `setNeedsDisplay` 和 `setNeedsLayout` 两个方法都是异步执行的。而 `setNeedsDisplay` 会自动调用 `drawRect` 方法，这样可以拿到 `UIGraphicsGetCurrentContext` 进行绘制；而 `setNeedsLayout` 会默认调用 `layoutSubViews` ，给当前的视图做了标记； `layoutIfNeeded` 查找是否有标记，如果有标记及立刻刷新。

只有 `setNeedsLayout` 和 `layoutIfNeeded` 这二者合起来使用，才会起到立刻刷新的效果。





### loadView 的作用？

`loadView` 方法会在每次访问 UIViewController 的 view (比如 `controller.view`、`self.view`) 而且 view 为 nil 时会被调用，此方法主要用来负责创建 UIViewController 的 view (重写 `loadView` 方法，并且不需要调用`[super loadView]`)

这里要提一下 `[super loadView]` ，`[super loadView]` 做了下面几件事。

它会先去查找与 UIViewController 相关联的 xib 文件，通过加载 xib 文件来创建 UIViewController 的 view ，如果在初始化 UIViewController 指定了 xib 文件名，就会根据传入的 xib 文件名加载对应的 xib 文件，如果没有明显地传 xib 文件名，就会加载跟 UIViewController 同名的 xib 文件

如果没有找到相关联的 xib 文件，就会创建一个空白的 UIView ，然后赋值给 UIViewController 的 view 属性

综上，在需要自定义 UIViewController 的 view 时，可以通过重写 `loadView` 方法且不需要调用 `[super loadView]` 方法。




### 使用 drawRect 有什么影响？

`drawRect` 方法依赖 Core Graphics 框架来进行自定义的绘制

**缺点**：它处理 touch 事件时每次按钮被点击后，都会用 `setNeddsDisplay` 进行强制重绘；而且不止一次，每次单点事件触发两次执行。这样的话从性能的角度来说，对 CPU 和内存来说都是欠佳的。特别是如果在我们的界面上有多个这样的 UIButton 实例，那就会很糟糕了。这个方法的调用机制也是非常特别. 当你调用 `setNeedsDisplay` 方法时, UIKit 将会把当前图层标记为 `dirty` ，但还是会显示原来的内容，直到下一次的视图渲染周期，才会将标记为 `dirty` 的图层重新建立 Core Graphics 上下文,然后将内存中的数据恢复出来，再使用 CGContextRef 进行绘制。




### keyWindow 和 appDelegate 中的 window 有何区别

appDelegate 中的 window 是程序启动时设置的 window 对象。

keyWindow 这个属性保存了`[windows]` 数组中的 `[UIWindow]` 对象，该对象最近被发送了 `[makeKeyAndVisible]` 消息

一般情况下 appDelegate 中的 window 和 keyWindow 是同一个对象，但不能保证 keyWindow 就是 appDelegate 中的 window，因为 keyWindow 会因为 `makeKeyAndVisible` 而变化，例如，程序中添加了一个悬浮窗口，这个时候 keywindow 就会变化。



## 响应者链

### 什么是响应者链？

简单来说就是：一级一级的找到响应的视图，如果没有就传给 UIWindow 实例和 UIApplication 实例，要是他们也处理不了，就丢弃这次事件。

对于 iOS 设备用户来说，他们操作设备的方式主要有三种：触摸屏幕、晃动设备、通过遥控设施控制设备。对应的事件类型有以下三种：

1. 触屏事件（Touch Event）
2. 运动事件（Motion Event）
3. 远端控制事件（Remote-Control Event）


响应者链概念: iOS 系统检测到手指触摸操作时会将其打包成一个 UIEvent 对象，并放入当前活动 Application 的事件队列，单例的 UIApplication 会从事件队列中取出触摸事件并传递给单例的 UIWindow 来处理，UIWindow 对象首先会使用 `hitTest:withEvent:` 方法寻找此次 Touch 操作初始点所在的视图，即需要将触摸事件传递给其处理的视图。

#### 寻找事件的第一响应者

UIWindow 实例对象会首先在它的内容视图上调用 `hitTest:withEvent:`，此方法会在其视图层级结构中的每个视图上调用 `pointInside:withEvent:`，该方法用来判断点击事件发生的位置是否处于当前视图范围内，以确定用户是不是点击了当前视图，如果 `pointInside:withEvent:` 返回 YES，则继续逐级调用，直到找到 touch 操作发生的位置，这个视图也就是要找的 hit-test view。

**上面找到了事件的第一响应者，接下来就该沿着寻找第一响应者的相反顺序来处理这个事件，如果UIWindow单例和UIApplication都无法处理这一事件，则该事件会被丢弃。**

#### 处理事件

1. 如果最终 hit-test 没有找到第一响应者，或者第一响应者没有处理该事件，则该事件会沿着响应者链向上回溯，如果 UIWindow 实例和 UIApplication 实例都不能处理该事件，则该事件会被丢弃；
2. `hitTest:withEvent:` 方法将会忽略隐藏(`hidden=YES`)的视图，禁止用户操作(`userInteractionEnabled=YES`)的视图，以及(`alpha<0.01`)的视图。如果一个子视图的区域超过父视图的 bound 区域，那么正常情况下对子视图在父视图之外区域的触摸操作不会被识别，因为父视图的 `pointInside:withEvent:` 方法会返回 NO，这样就不会继续向下遍历子视图了。当然，也可以重写 `pointInside:withEvent:` 方法来处理这种情况。
3. 我们可以重写 `hitTest:withEvent:` 来达到某些特定的目的。





### 谈谈对 UIResponder 的理解

UIResponder 类是专门用来响应用户的操作处理各种事件的，包括触摸事件(Touch Events)、运动事件(Motion Events)、远程控制事件(Remote Control Events)。我们知道 UIApplication、UIView、UIViewController 这几个类是直接继承自 UIResponder ，所以这些类都可以响应事件。当然我们自定义的继承自 UIView 的 View 以及自定义的继承自 UIViewController 的控制器都可以响应事件。

响应者对象（Responder Object） 指的是有响应和处理事件能力的对象。响应者链就是由一系列的响应者对象构成的一个层次结构。

UIResponder 是所有响应对象的基类，在 UIResponder 类中定义了处理上述各种事件的接口。我们熟悉的 UIApplication、UIViewController、UIWindow 和所有继承自 UIView 的 UIKit 类都直接或间接的继承自 UIResponder，所以它们的实例都是可以构成响应者链的响应者对象。




## WebKit

### 说一下 JS 和 OC 互相调用的几种方式？

#### JS 调用 OC 的三种方式

- 根据网页重定向截取字符串通过 url scheme 判断

- 替换方法 `context[@"copyText"]`

- 注入对象：遵守协议 NSExport，设置 `context[@"per"] = per` 调用方式类似 Swift 中的链式编程 `per.eat();`

#### OC 调用 JS 代码两种方式

- 通过 webVIew 调用 `webView stringByEvaluatingJavaScriptFromString:` 调用

- 通过 JSContext 调用 `[context evaluateScript:];`




### 在使用 WKWedView 时遇到过哪些问题？

白屏问题，Cookie 问题，在 WKWebView 上直接使用 NSURLProtocol 无法拦截请求，在 WKWebView 上通过 loadRequest 发起的 post 请求 body 数据被丢失，截屏问题等
