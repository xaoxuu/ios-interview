# Swift

## 基本理解

### Swift 相比 OC 有哪些优点

- Swift 是类型安全的语言，注重安全，OC注重灵活
- Swift 注重面向协议编程、函数式编程、面向对象编程，OC注重面向对象编程
- Swift 注重值类型，OC 注重指针和引用
- Swift 是静态类型语言，OC 是动态类型语言
- Swift 容易阅读，文件结构和大部分语法简易化，只有 `.swift` 文件，结尾不需要分号
- Swift 中的可选类型，是用于所有数据类型，而不仅仅局限于类。相比于 OC 中的 `nil` 更加安全和简明
- Swift 中的泛型类型更加方便和通用，而非 OC 中只能为集合类型添加泛型
- Swift 中各种方便快捷的[高阶函数]（函数式编程）（Swift的标准数组支持三个高阶函数：`map`, `filter` 和 `reduce` 以及 `map` 的扩展 `flatMap`）
- Swift 新增了两种权限，细化权限。 `open` > `public` > `internal`(默认) > `fileprivate` > `private`
- Swift 中独有的元组类型(tuples)，把多个值组合成复合值。元组内的值可以是任何类型，并不要求是相同类型的。
- Swift 支持函数式编程，ObjC 本身是不支持的，需要通过引入 ReactiveCocoa 这个库才可支持函数式编程。

## Swift 的高阶函数

### map

Swift 中的 map 函数的作用就是接受一个闭包作为规则，自动遍历集合的每一个元素，使用闭包的规则去处理这些元素，生成一个结构相同的集合。

#### 将 Int 类型数组中的元素乘以 2 ，然后转换为 String 类型的数组

```swift
let ints = [1, 2, 3, 4]
let strs = ints.map { "\($0 * 2)" }

print(strs) // 打印结果：["2", "4", "6", "8"]
```

#### 生成一个新的 Int 数组，元素是多少元素就重复多少个

```swift
let nums = [1, 2, 3, 4]
let mapNums = nums.map { Array(repeating: $0, count: $0) }

print(mapNums) // 打印结果：[[1], [2, 2], [3, 3, 3], [4, 4, 4, 4]]
```

#### 将 String 类型的数组转换为 Int 类型的数组

```swift
let someAry = ["12", "ad", "33", "cc", "22"]
// var nilAry: [Int?]
let nilAry = someAry.map { Int($0) }

print(nilAry) // 打印结果：[Optional(12), nil, Optional(33), nil, Optional(22)]
```


#### 对 NSDate 做 format 操作

不使用 map 的写法

```swift
let date: Date? = Date()
let formatter = DateFormatter()
formatter.dateFormat = "YYYY-MM-dd"
var formatted: String? = nil
if let d = date {
    formatted = formatter.string(from: d)
}

print(formatted) // 打印结果：Optional("2019-07-16")
```

使用 map 函数

```swift
let date: Date? = Date()
let formatter = DateFormatter()
formatter.dateFormat = "YYYY-MM-dd"
let formatted = date.map { formatter.string(from: $0) }

print(formatted) // Optional("2019-07-24")
```


### flatMap

flatMap 的实现与 map 非常类似，也是数组中每个元素通过某种规则（闭包实现）进行转换，最后返回一个新的数组。

不过 flatMap 能把数组中存有数组的数组（二维数组、N维数组）一同打开变成一个新的数组，称为降维，通俗一点就是把多维数组都会拍扁为一维数组，通过后面的例子一看就明白了。

#### 示例

生成一个新的 Int 数组，元素是多少元素就重复多少个

```swift
let nums = [1, 2, 3, 4]
let mapNums = nums.flatMap { Array(repeating: $0, count: $0) }

print(mapNums) // flatMap打印结果：[1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
```

可以看到，flatMap 把数组中的数组都打开了，最终返回的是一个一维数组。而 map 返回的是一个二维数组，没有降维。

### reduce

Swift 中的 reduce 函数的作用就是接受一个初始化值，并且接受一个闭包作为规则，自动遍历集合的每一个元素，使用闭包的规则去处理这些元素，合并处理结果。

```
let packages = [
    Package(name: "Swift高阶函数编程", number: 1, price: 80.0, address: "中关村"),
    Package(name: "Swift面向协议编程", number: 2, price: 88.0, address: "西二旗"),
    Package(name: "Swift基础", number: 3, price: 35.0, address: "798"),
    Package(name: "Swift进阶", number: 4, price: 50.0, address: "望京soho")
]

let reduceName = packages.reduce("") {$0 + $1.name}
```
