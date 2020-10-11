# Runtime

## 概念理解

### Category 的实现原理？

Category 实际上是 `Category_t` 的结构体，在运行时，新添加的方法，都被以倒序插入到原有方法列表的最前面，所以不同的 Category，添加了同一个方法，执行的实际上是最后一个。

Category 在刚刚编译完的时候，和原来的类是分开的，只有在程序运行起来后，通过 Runtime，Category 和原来的类才会合并到一起。



### isa 指针的理解，对象的 isa 指针指向哪里？ isa 指针有哪两种类型？

#### isa 指针的理解

在 Objective-C 中，任何类的定义都是对象。类和类的实例（对象）没有任何本质上的区别。任何对象都有 isa 指针，指向对象的类。

- 每一个对象本质上都是一个类的实例。其中类定义了成员变量和成员方法的列表。对象通过对象的isa指针指向类。
- 每一个类本质上都是一个对象，类其实是元类（metaclass）的实例。元类定义了类方法的列表。类通过类的isa指针指向元类。
- 所有的元类最终继承一个根元类，根元类isa指针指向本身，形成一个封闭的内循环。

#### isa 指针指向

- 实例对象的 isa 指向类对象
- 类对象的 isa 指向元类对象
- 元类对象的 isa 指向元类的基类

#### isa 有两种类型

- 纯指针，指向内存地址
- NON_POINTER_ISA，除了内存地址，还存有一些其他信息



### 讲一下 OC 的消息机制

OC 中的方法调用其实都是转成了 `objc_msgSend` 函数的调用，给 receiver（方法调用者）发送了一条消息（selector方法名）

objc_msgSend 底层有3大阶段，消息发送（当前类、父类中查找）、动态方法解析、消息转发



### OC 方法调用流程是怎样的？

OC 是运行时语言，方法调用的本质，就是让对象发送消息：

```c
id objc_msgSend ( id self, SEL op, ... );
```

具体的流程如下：

1. 通过 isa 指针找到所属类
2. 查找类的 cache 列表，如果没有则下一步
3. 查找类的是“方法列表”
4. 如果能找到与选择子名称相符的方法，就跳至其实现代码
5. 找不到，就沿着继承体系继续向上查找
6. 如果能找到与选择子名称相符的方法，就跳至其实现代码
7. 找不到，执行“消息转发”。



### 消息转发机制的理解和应用？

如果收到无法解读的消息，会尝试做消息转发，即以下三种补救措施：

#### 1. 动态方法解析

```objc
+ (BOOL)resolveInstanceMethod:(SEL)sel;
+ (BOOL)resolveClassMethod:(SEL)sel;
```

#### 2. 备用接收者

```objc
- (id)forwardingTargetForSelector:(SEL)aSelector;
```

#### 3. 完整的消息转发

```objc
- (void)forwardInvocation:(NSInvocation *)anInvocation;
```



### 如何让多个方法指向同一个实现？





### 在分类中重写了原类中的方法，实际调用情况是怎样的？

只调用分类中的实现而不调用原类中的实现。

因为 Category 的底层实现是在加载的时候，把 Category 中的方法添加到原类的方法列表中，当调用方法时会遍历方法列表找到对应的响应子就返回，不再向下遍历。因为 Category 的优先级高于类的优先级，使得原类中的选择子遍历不到。



### 成员变量的本质是什么？

#### 成员变量的定义

**Ivar**: 实例变量类型，是一个指向objc_ivar结构体的指针

```
typedef struct objc_ivar *Ivar;
```

操作函数：

```c
// 获取所有成员变量
class_copyIvarList

// 获取成员变量名
ivar_getName

// 获取成员变量类型编码
ivar_getTypeEncoding

// 获取指定名称的成员变量
class_getInstanceVariable

// 获取某个对象成员变量的值
object_getIvar

// 设置某个对象成员变量的值
object_setIvar
```

获取成员变量的方法：

```c
unsigned int count = 0;
Ivar * ivars = class_copyIvarList([Model class], &count);
for (unsigned int i = 0; i < count; i ++) {
    Ivar ivar = ivars[i];
    const char * name = ivar_getName(ivar);
    const char * type = ivar_getTypeEncoding(ivar);
    NSLog(@"类型为 %s 的 %s ",type, name);
}
free(ivars);
```

#### 属性的定义

objc_property_t：声明的属性的类型，是一个指向objc_property结构体的指针

```
typedef struct objc_property *objc_property_t;
```

操作函数：

```c
// 获取所有属性，不会获取无@property声明的成员变量
class_copyPropertyList

// 获取属性名
property_getName

// 获取属性特性描述字符串
property_getAttributes

// 获取所有属性特性
property_copyAttributeList
```

参考资料：https://www.jianshu.com/p/d361f169423b




### load 和 initialize 的区别？

#### 相同点

- 两者都会自动调用父类的，不需要 super 操作，且仅会调用一次（不包括外部显示调用)。

- load 和 initialize 方法内部使用了锁，因此它们是线程安全的。实现时要尽可能保持简单，避免阻塞线程，不要再使用锁。

#### 不同点

- load 和 initialize 方法都会在实例化对象之前调用，以 main 函数为分水岭，前者在 main 函数之前调用，后者在之后调用。这两个方法会被自动调用，不能手动调用它们。

- load 和 initialize 方法都不用显示的调用父类的方法而是自动调用，即使子类没有 initialize 方法也会调用父类的方法，而 load 方法则不会调用父类。

- load 方法通常用来进行 Method Swizzle，initialize 方法一般用于初始化全局变量或静态变量。




### 怎么理解 OC 是运行时语言？

主要是将数据类型的确定由编译时，推迟到了运行时。这个问题其实浅涉及到两个概念：运行时和多态。

简单来说，运行时机制使我们直到运行时才去决定一个对象的类别，以及调用该类别对象指定方法。

多态：不同对象以自己的方式响应相同的消息的能力叫做多态。



### 向一个 nil 对象发送消息将会发生什么？

在 Objective-C 中向 nil 发送消息是完全有效的，只是在运行时不会有任何作用。

具体原因如下：

- 向一个对象发送消息时，runtime 库会根据对象的 isa 指针找到该对象实际所属的类，然后在该类中的方法列表以及其父类方法列表中寻找方法运行，然后在发送消息的时候，`objc_msgSend` 方法不会返回值，所谓的返回内容都是具体调用时执行的。

- 如果向一个 nil 对象发送消息，首先在寻找对象的 isa 指针时就是0地址返回了，所以不会出现任何错误。



### runtime 如何通过 selector 找到对应的 IMP 地址？

每一个类对象中都一个方法列表，方法列表中记录着方法的名称，方法实现，以及参数类型，其实 selector 本质就是方法名称，通过这个方法名称就可以在方法列表中找到对应的方法实现。



### 能否向编译后得到的类中添加实例变量？能否向运行时创建的类中添加实例变量？

- 不能向编译后得到的类中添加实例变量；
- 能向运行时创建的类中添加实例变量；

解释下：

- 因为编译后的类已经注册在 runtime 中，类结构体中的 `objc_ivar_list` 实例变量的链表和 `instance_size` 实例变量的内存大小已经确定，同时 runtime 会调用 `class_setIvarLayout` 或 `class_setWeakIvarLayout` 来处理 strong weak 引用。所以不能向存在的类中添加实例变量；
- 运行时创建的类可以添加实例变量，调用 `class_addIvar` 函数。但是得在调用 `objc_allocateClassPair` 之后，`objc_registerClassPair` 之前，原因同上。




## 应用能力


主要有以下几种：关联对象，给分类增加属性、动态添加方法、交换方法、拦截系统方法的调用、自动归档解档、字典转模型、万能控制器跳转、KVO 的底层实现。

### 关联对象，给分类增加属性

```c
// 关联对象
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
// 获取关联的对象
id objc_getAssociatedObject(id object, const void *key)
// 移除关联的对象
void objc_removeAssociatedObjects(id object)
```

例如给一个名为 Person 的类的分类添加一个 books 属性：

```objc
static const void *KEY_BOOKS = &KEY_BOOKS;

@implementation Person (Books)
- (void)setBooks:(NSArray *)books{
    objc_setAssociatedObject(self, KEY_BOOKS, books, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (NSArray *)books{
    return objc_getAssociatedObject(self, KEY_BOOKS);
}
@end
```


### 动态添加方法

```c
BOOL class_addMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, const char * _Nullable types);
```

通过 `performSelector` 调用某个方法，如果在运行时找不到 Selector 对应的实现，会执行消息转发，在 `resolveInstanceMethod` 或 `resolveClassMethod` 中将函数与对象的 Selector 关联起来。

```objc
void eat(id self, SEL sel) {
    NSLog(@"%@ %@",self, NSStringFromSelector(sel));
}

@implementation Person

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    if (sel == @selector(eat)) {
        class_addMethod(self, @selector(eat), eat, "v@:");
    }
    return [super resolveInstanceMethod:sel];
}

@end
```


### 交换方法、拦截系统方法的调用

每个类都维护一个方法列表，Method 则包含 SEL 和其对应 IMP 的信息，方法交换做的事情就是把 SEL 和 IMP 的对应关系断开，并和新的 IMP 生成对应关系。

```objc
+ (void)exchangeClassMethodImplementations:(Class)cls selector1:(SEL)selector1 selector2:(SEL)selector2{
    Method m1 = class_getClassMethod(cls, selector1);
    Method m2 = class_getClassMethod(cls, selector2);
    method_exchangeImplementations(m1, m2);
}

+ (void)exchangeInstanceMethodImplementations:(Class)cls selector1:(SEL)selector1 selector2:(SEL)selector2{
    Method m1 = class_getInstanceMethod(cls, selector1);
    Method m2 = class_getInstanceMethod(cls, selector2);
    method_exchangeImplementations(m1, m2);
}
```


### 自动归档解档

遍历 Model 自身所有属性，并对属性进行 encode 和 decode 操作。

```objc
- (id)initWithCoder:(NSCoder *)aDecoder {
    if (self = [super init]) {
        unsigned int outCount;
        Ivar * ivars = class_copyIvarList([self class], &outCount);
        for (int i = 0; i < outCount; i ++) {
            Ivar ivar = ivars[i];
            NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
            [self setValue:[aDecoder decodeObjectForKey:key] forKey:key];
        }
    }
    return self;
}

- (void)encodeWithCoder:(NSCoder *)aCoder {
    unsigned int outCount;
    Ivar * ivars = class_copyIvarList([self class], &outCount);
    for (int i = 0; i < outCount; i ++) {
        Ivar ivar = ivars[i];
        NSString * key = [NSString stringWithUTF8String:ivar_getName(ivar)];
        [aCoder encodeObject:[self valueForKey:key] forKey:key];
    }
}
```


### 访问私有变量

通过 `getIvar` 可以获取到任意已知 name 的成员变量的值。

```
Ivar ivar = class_getInstanceVariable([Model class], "_str1");
NSString * str1 = object_getIvar(model, ivar);
```


### 字典转模型的 KVC 实现

遍历 Model 自身所有属性，从 json 中找到与属性对应的值，通过 KVC 将其赋值。

```objc
// Ivar:成员变量 以下划线开头
// Property:属性
+ (instancetype)modelWithDict:(NSDictionary *)dict {
    id objc = [[self alloc] init];
    unsigned int count = 0;
    // 获取成员变量数组
    Ivar *ivarList = class_copyIvarList(self, &count);

    // 遍历所有成员变量
    for (int i = 0; i < count; i++) {
        // 获取成员变量
        Ivar ivar = ivarList[i];
        // 获取成员变量名字
        NSString *ivarName = [NSString stringWithUTF8String:ivar_getName(ivar)];
        // 获取成员变量类型
        NSString *ivarType = [NSString stringWithUTF8String:ivar_getTypeEncoding(ivar)];
        // @\"User\" -> User
        ivarType = [ivarType stringByReplacingOccurrencesOfString:@"\"" withString:@""];
        ivarType = [ivarType stringByReplacingOccurrencesOfString:@"@" withString:@""];
        // 获取key
        NSString *key = [ivarName substringFromIndex:1];

        // 去字典中查找对应value
        // key:user  value:NSDictionary
        id value = dict[key];

        // 二级转换:判断下value是否是字典,如果是,字典转换层对应的模型
        // 并且是自定义对象才需要转换
        if ([value isKindOfClass:[NSDictionary class]] && ![ivarType hasPrefix:@"NS"]) {
            // 字典转换成模型 userDict => User模型
            Class modelClass = NSClassFromString(ivarType);
            value = [modelClass modelWithDict:value];
        }

        // 给模型中属性赋值
        if (value) {
            [objc setValue:value forKey:key];
        }
    }

    return objc;
}
```


### 实现万能控制器跳转

利用 runtime 动态生成对象、属性、方法这特性，我们可以先跟服务端商量好，定义跳转规则，比如要跳转到 A 控制器，需要传属性 id、type，那么服务端返回字典给我，里面有控制器名，两个属性名跟属性值，客户端就可以根据控制器名生成对象，再用 kvc 给对象赋值。

参考资料：http://www.cocoachina.com/articles/13104



### KVO 的底层实现

KVO 是基于 runtime 机制实现的，当某个类的对象第一次被观察时，系统就会在运行期动态地创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的 setter 方法。
