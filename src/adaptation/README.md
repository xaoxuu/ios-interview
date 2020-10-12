# 适配、升级、混编

### Objective-C 与 Swift 混编

#### Swift 调用 Objective-C

Swift 调用 Objective-C 文件比较简单。当在 Swift 工程中新建 Objective-C 文件或者在 Objective-C 工程中新建 Swift 文件时，Xcode 会自动提示你是否创建 bridging header 桥接头文件，点击创建后 Xcode 会自动为你创建一个桥接头文件。 在基于 OC 的 OCSwiftDemo 中创建一个 Swift 文件或者在 Swift 项目中创建 OC 文件时，Xcode 会自动创建一个名为 `SwiftProject-Bridging-Header.h` 桥接头文件。


当然你也可以在 Building Settings 中自己设置桥接头文件（一般情况下我们会用系统默认生成的）

创建好 Bridging Header 文件后，在 Bridging Header 文件中即可 import 需要提供给 Swift 的 Objective-C 头文件，Swift 即可调用对应的 Objective-C 文件。 同时 Xcode 可以自动生成 Objective-C 对应的 Swift 接口。

然后就可以在 Swift 中调用 OC 的代码。

#### Objective-C 调用 Swift

在 OC 中导入 `**-Swift.h` 文件



### 适配暗黑模式

#### 颜色适配

iOS 13 之前 UIColor 只能表示一种颜色，从 iOS 13 开始 UIColor 是一个动态的颜色，它可以在 LightMode 和 DarkMode 拥有不同的颜色。
iOS 13 的 UIColor 增加了很多动态颜色，我们来看下用系统提供的颜色能实现怎么样的效果。

```swift
// UIColor 增加的颜色
@available(iOS 13.0, *)
open class var systemBackground: UIColor { get }
@available(iOS 13.0, *)
open class var label: UIColor { get }
@available(iOS 13.0, *)
open class var placeholderText: UIColor { get }
...

view.backgroundColor = UIColor.systemBackground
label.textColor = UIColor.label
placeholderLabel.textColor = UIColor.placeholderText
```

### 如何自己创建一个动态的 UIColor 呢？

```swift
@available(iOS 13.0, *)
public init(dynamicProvider: @escaping (UITraitCollection) -> UIColor)
```

这个方法要求传一个闭包进去，当系统从 LightMode 和 DarkMode 之间切换的时候就会触发这个回调。这个闭包返回一个 UITraitCollection 类，我们要用这个类的 userInterfaceStyle 属性。

```swift
@available(iOS 12.0, *)
public enum UIUserInterfaceStyle : Int {
    case unspecified
    case light
    case dark
}
```

现在我们创建两个 UIColor 并赋值给 view.backgroundColor 和 label，代码如下：

```swift
let backgroundColor = UIColor { (traitCollection) -> UIColor in
    if traitCollection.userInterfaceStyle == .dark {
        return UIColor.black
    } else {
        return UIColor.white
    }
}
view.backgroundColor = backgroundColor

let labelColor = UIColor { (traitCollection) -> UIColor in
    if traitCollection.userInterfaceStyle == .dark {
        return UIColor.white
    } else {
        return UIColor.black
    }
}
label.textColor = labelColor
```



#### 图片适配

打开 Assets.xcassets 把图片拖拽进去，然后我们在右侧工具栏中点击最后一栏，点击 Appearances 选择 <kbd>Any, Dark</kbd> 或者 <kbd>Any, Light, Dark</kbd>。

![](https://i.loli.net/2020/04/14/2a4xvy358BVq61Z.png)

#### 获取当前模式 (Light or Dark)

```swift
if traitCollection.userInterfaceStyle == .dark {
    // Dark
} else {
    // Light
}
```

#### 监听模式变化

```swift
override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
    super.traitCollectionDidChange(previousTraitCollection)
    if traitCollection.hasDifferentColorAppearance(comparedTo: previousTraitCollection) {
        // 适配代码
    }
}
```

#### 改变当前模式

```swift
overrideUserInterfaceStyle = .dark
print(traitCollection.userInterfaceStyle)  // dark
```


<br>

> **参考资料**
> - [iOS13-适配夜间模式/深色外观(Dark Mode)](https://www.jianshu.com/p/e6616e44cf60)
