# 项目架构

## 概念理解

### MVC

![MVC 架构示意图](https://i.loli.net/2020/04/07/fOmcvCHXYPER8nD.png)

MVC（Model、View、Controller）
MVC是比较直观的架构模式，最核心的就是通过Controller层来进行调控，首先看一下官方提供的MVC示意图：


Model和View永远不能相互通信，只能通过Controller传递

Controller可以直接与Model对话（读写调用Model），Model通过NOtification和KVO机制与Controller间接通信

Controller可以直接与View对话，通过IBoutlet直接操作View，IBoutlet直接对应View的控件（例如创建一个Button：需声明一个 IBOutlet UIButton * btn），View通过action向Controller报告时间的发生(用户点击了按钮)。Controller是View的直接数据源

优点：对于混乱的项目组织方式，有了一个明确的组织方式。通过Controller来掌控全局，同时将View展示和Model的变化分开

缺点：愈发笨重的Controller，随着业务逻辑的增加，大量的代码放进Controller，导致Controller越来越臃肿，堆积成千上万行代码，后期维护起来费时费力



### MVP

![MVP 架构示意图](https://i.loli.net/2020/04/07/lnm4Dw31HVqhCEs.png)

MVP（Model、View、Presenter）
MVP模式是MVC模式的一个演化版本，其中Model与MVC模式中Model层没有太大区别，主要提供数据存储功能，一般都是用来封装网络获取的json数据；View与MVC中的View层有一些差别，MVP中的View层可以是viewController、view等控件；Presenter层则是作为Model和View的中介，从Model层获取数据之后传给View。

从上图可以看出，从MVC模式中增加了Presenter层，将UIViewController中复杂的业务逻辑、网络请求等剥离出来。

优点 模型和视图完全分离，可以做到修改视图而不影响模型；更高效的使用模型，View不依赖Model，可以说VIew能做到对业务逻辑完全分离

缺点 Presenter中除了处理业务逻辑以外，还要处理View-Model两层的协调，也会导致Presenter层的臃肿



### MVVM

MVVM（Model、Controller/View、ViewModel）
在MVVM中，view和ViewCOntroller联系在一起，我们把它们视为一个组件，view和ViewController都不能直接引用model，而是引用是视图模型即ViewModel。 viewModel是一个用来放置用户输入验证逻辑、视图显示逻辑、网络请求等业务逻辑的地方，这样的设计模式，会轻微增加代码量，但是会减少代码的复杂性

优点 VIew可以独立于Model的变化和修改，一个ViewModel可以绑定到不同的View上，降低耦合，增加重用

缺点 过于简单的项目不适用、大型的项目视图状态较多时构建和维护成本太大

合理的运用架构模式有利于项目、团队开发工作，但是到底选择哪个设计模式，哪种设计模式更好，就像本文开头所说，不同的设计模式，只是让不同的场景有了更多的选择方案。根据项目场景和开发需求，选择最合适的解决方案。



<br>

> **参考资料**
> - [【长篇高能】ReactiveCocoa 和 MVVM 入门](http://www.cocoachina.com/articles/11930)
> - [不要用子类！Swift的核心是面向协议](http://www.cocoachina.com/articles/12881)
