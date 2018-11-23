---
title: Swift 4.0 新特征汇总及演示
layout: post
date: 2018-03-09 21:16:27
comments: true
categories:
	- Project
keywords: Swift
tags:
	- Swift
description: 

---
Swift 在不断的版本迭代中，由于其 ABI 尚不稳定，所以给开发者带来许多的挑战，但也正因为如此，这门语言才有无限可能~                                                                                                                        

<!-- more -->
------
![](https://user-gold-cdn.xitu.io/2018/3/20/16241654c9724f1d?w=792&h=270&f=png&s=31206)

👨🏻‍💻 [Github Demo](https://github.com/ReverseScale/Swift4.0NewFeature)


### Key Paths 新语法
key-path 通常是用在键值编码（KVC）与键值观察（KVO）上的，KVC、KVO 相关内容可以参考我之前写的这篇文章：Swift - 反射（Reflection）的介绍与使用样例（附KVC介绍）

1.Swift 3 之前使用的是 String 类型的 key-Path
```
//用户类
class User: NSObject{
    @objc var name:String = ""  //姓名
    @objc var age:Int = 0  //年龄
}
 
//创建一个User实例对象
let user1 = User()
user1.name = "hangge"
user1.age = 100
 
//使用KVC取值
let name = user1.value(forKey: "name")
print(name)
 
//使用KVC赋值
user1.setValue("hangge.com", forKey: "name")
```
具体显示如下：
![](https://user-gold-cdn.xitu.io/2018/3/9/1620949944021d59?w=331&h=41&f=png&s=9756)

2.到了 Swift 3 新增了 #keyPath() 写法
使用 #keyPath() 写法，可以避免我们因为拼写错误而引发问题。
```
//用户类
class User: NSObject{
    @objc var name:String = ""  //姓名
    @objc var age:Int = 0  //年龄
}
 
//创建一个User实例对象
let user1 = User()
user1.name = "hangge"
user1.age = 100
 
//使用KVC取值
let name = user1.value(forKeyPath: #keyPath(User.name))
print(name)
 
//使用KVC赋值
user1.setValue("hangge.com", forKeyPath: #keyPath(User.name))
```
3.Swift 4 中直接用 \ 作为开头创建 KeyPath
新的方式不仅使用更加简单，而且有如下优点：
* 类型可以定义为 class、struct
* 定义类型时无需加上 @objc 等关键字
* 性能更好
* 类型安全和类型推断，例如：user1.value(forKeyPath: #keyPath(User.name)) 返回的类型是 Any，user1[keyPath: \User.name] 直接返回 String 类型
* 可以在所有值类型上使用
（1）比如上面的样例在 Swift4 中可以这么写：
```
//用户类
class User: NSObject{
    var name:String = ""  //姓名
    var age:Int = 0  //年龄
}
 
//创建一个User实例对象
let user1 = User()
user1.name = "hangge"
user1.age = 100
 
//使用KVC取值
let name = user1[keyPath: \User.name]
print(name)
 
//使用KVC赋值
user1[keyPath: \User.name] = "hangge.com"
```
（2）keyPath 定义在外面也是可以的：
```
let keyPath = \User.name
 
let name = user1[keyPath: keyPath]
print(name)
 
user1[keyPath: keyPath] = "hangge.com"
```
（3）可以使用 appending 方法向已定义的 Key Path 基础上填加新的 Key Path。
```
let keyPath1 = \User.phone
let keyPath2 = keyPath1.appending(path: \.number)
```

----
### 类与协议的组合类型

在 Swift 4 中，可以把类（Class）和协议（Protocol）用 & 组合在一起作为一个类型使用。

使用样例1：
```
protocol MyProtocol { }
 
class View { }
 
class ViewSubclass: View, MyProtocol { }
 
class MyClass {
    var delegate: (View & MyProtocol)?
}
 
let myClass = MyClass()
myClass.delegate = ViewSubclass() //这个编译正常
myClass.delegate = View() //这个编译报错:
```
具体错误信息如下：

![](https://user-gold-cdn.xitu.io/2018/3/9/162094994523a249?w=661&h=54&f=png&s=36843)

使用样例2：
```
protocol Shakeable {
    func shake()
}
 
extension UIButton: Shakeable {
    func shake() {
        /* ... */
    }
}
 
extension UISlider: Shakeable {
    func shake() {
        /* ... */
    }
}
 
func shakeEm(controls: [UIControl & Shakeable]) {
    for control in controls where control.isEnabled {
        control.shake()
    }
}
```
----
### 下标支持泛型
1.下标的返回类型支持泛型
有时候我们会写一些数据容器，Swift 支持通过下标来读写容器中的数据。但是如果容器类中的数据类型定义为泛型，过去下标语法就只能返回 Any，在取出值后需要用 as? 来转换类型。现在 Swift 4 定义下标也可以使用泛型了。
```
struct GenericDictionary<Key: Hashable, Value> {
    private var data: [Key: Value]
     
    init(data: [Key: Value]) {
        self.data = data
    }
     
    subscript<T>(key: Key) -> T? {
        return data[key] as? T
    }
}
 
//字典类型: [String: Any]
let earthData = GenericDictionary(data: ["name": "Earth", "population": 7500000000, "moons": 1])
 
//自动转换类型，不需要在写 "as? String"
let name: String? = earthData["name"]
print(name)
 
//自动转换类型，不需要在写 "as? Int"
let population: Int? = earthData["population"]
print(population)
```
![](https://user-gold-cdn.xitu.io/2018/3/9/162094994476f137?w=333&h=44&f=png&s=13584)

2.下标类型同样支持泛型
```
extension GenericDictionary {
    subscript<Keys: Sequence>(keys: Keys) -> [Value] where Keys.Iterator.Element == Key {
        var values: [Value] = []
        for key in keys {
            if let value = data[key] {
                values.append(value)
            }
        }
        return values
    }
}
 
// Array下标
let nameAndMoons = earthData[["moons", "name"]]        // [1, "Earth"]
// Set下标
let nameAndMoons2 = earthData[Set(["moons", "name"])]  // [1, "Earth"]
```
----
### Codable 序列化

如果要将一个对象持久化，需要把这个对象序列化。过去的做法是实现 NSCoding 协议，但实现 NSCoding 协议的代码写起来很繁琐，尤其是当属性非常多的时候。
Swift 4 中引入了 Codable 协议，可以大大减轻了我们的工作量。我们只需要让需要序列化的对象符合 Codable 协议即可，不用再写任何其他的代码。
```
struct Language: Codable {
    var name: String
    var version: Int
}
```
1.Encode 操作
我们可以直接把符合了 Codable 协议的对象 encode 成 JSON 或者 PropertyList。
```
let swift = Language(name: "Swift", version: 4)
 
//encoded对象
let encodedData = try JSONEncoder().encode(swift)
 
//从encoded对象获取String
let jsonString = String(data: encodedData, encoding: .utf8)
print(jsonString)
```
![](https://user-gold-cdn.xitu.io/2018/3/9/1620949968e93e6f?w=363&h=50&f=png&s=15143)

2.Decode 操作
```
let decodedData = try JSONDecoder().decode(Language.self, from: encodedData)
print(decodedData.name, decodedData.version)
```
![](https://user-gold-cdn.xitu.io/2018/3/9/162094994414e30b?w=205&h=39&f=png&s=8642)

----
### Subtring
Swift 4 中有一个很大的变化就是 String 可以当做 Collection 来用，并不是因为 String 实现了 Collection 协议，而是 String 本身增加了很多 Collection 协议中的方法，使得 String 在使用时看上去就是个 Collection。
```
let str = "hangge.com"
 
print(str.prefix(5)) // "hangg"
print(str.suffix(5)) // "e.com"
 
print(str.dropFirst()) // "angge.com"
print(str.dropLast()) // "hangge.co"
```
比如上面的样例，我们使用一些 Collection 协议的方法对字符串进行截取，只不过它们的返回结果不是 String 类型，而是 Swift 4 新增的 Substring 类型。

1.为何要引入 Substring？
既然我们想要的到的就是字符串，那么直接返回 String 就好了，为什么还要多此一举返回 Substring。原因只有一个：性能。具体可以参考下图：
当我们用一些 Collection 的方式得到 String 里的一部分时，创建的都是 Substring。Substring 与原 String 是共享一个 Storage。这意味我们在操作这个部分的时候，是不需要频繁的去创建内存，从而使得 Swift 4 的 String 相关操作可以获取比较高的性能。
而当我们显式地将 Substring 转成 String 的时候，才会 Copy 一份 String 到新的内存空间来，这时新的 String 和之前的 String 就没有关系了。

2.使用 Substring 的注意事项
由于 Substring 与原 String 是共享存储空间的，只要我们使用了 Substring，原 String 就会存在内存空间中。只有 Substring 被释放以后，整个 String 才会被释放。
而且 Substring 类型无法直接赋值给需要 String 类型的地方，我们必须用 String() 包一层。当然这时系统就会通过复制创建出一个新的字符串对象，之后原字符串就会被释放。

3.使用样例
这里对 String 进行扩展，新增一个 subString 方法。直接可以根据起始位置（Int 类型）和需要的长度（Int 类型），来截取出子字符串。
```
extension String {
    //根据开始位置和长度截取字符串
    func subString(start:Int, length:Int = -1) -> String {
        var len = length
        if len == -1 {
            len = self.count - start
        }
        let st = self.index(startIndex, offsetBy:start)
        let en = self.index(st, offsetBy:len)
        return String(self[st ..< en])
    }
}
```
使用样例：
```
let str1 = "欢迎访问hangge.com"
let str2 = str1.subString(start: 4, length: 6)
print("原字符串：\(str1)")
print("截取出的字符串：\(str2)")
```
运行结果如下：

![](https://user-gold-cdn.xitu.io/2018/3/9/16209499445602df?w=285&h=55&f=png&s=16118)

注意：这个方法最后我们会将 Substring 显式地转成 String 再返回。

----
### 废除 swap 方法

（1）过去我们会使用 swap(_:_:) 来将两个变量的值进行交换：
```
var a = 1
var b = 2
swap(&a, &b)
print(a, b)
```
（2）后面 swap() 方法将会被废弃，建议使用 tuple（元组）特性来实现值交换，也只需要一句话就能实现：
```
var a = 1
var b = 2
(b, a) = (a, b)
print(a, b)
```
使用 tuple 方式的好处是，多个变量值也可以一起进行交换：
```
var a = 1
var b = 2
var c = 3
(a, b, c) = (b, c, a)
print(a, b, c)
```
（3）补充一下：现在数组增加了个 swapAt 方法可以实现两个元素的位置交换。
```
var fruits = ["apple", "pear", "grape", "banana"]
//交换元素位置（第2个和第3个元素位置进行交换）
fruits.swapAt(1, 2)
print(fruits)
```
![](https://user-gold-cdn.xitu.io/2018/3/9/162094993a3445b9?w=270&h=36&f=png&s=9367)

----
### 减少隐式 @objc 自动推断

1.过去的情况（Swift 3）
（1）在项目中如果想把 Swift 写的 API 暴露给 Objective-C 调用，需要增加 @objc。在 Swift 3 中，编译器会在很多地方为我们隐式的加上 @objc。
（2）比如当一个类继承于 NSObject，那么这个类的所有方法都会被隐式的加上 @objc。
```
class MyClass: NSObject {
    func print() { } // 包含隐式的 @objc
    func show() { } // 包含隐式的 @objc
}
```
（3）但这样做很多并不需要暴露给 Objective-C 也被加上了 @objc。而大量 @objc 会导致二进制文件大小的增加。

2.现在的情况（Swift 4）
（1）在 Swift 4 中隐式 @objc 自动推断只会发生在下面这种必须要使用 @objc 的情况：
* 覆盖父类的 Objective-C 方法
* 符合一个 Objective-C 的协议

（2）大多数地方必须手工显示地加上 @objc。
```
class MyClass: NSObject {
    @objc func print() { } //显示的加上 @objc
    @objc func show() { } //显示的加上 @objc
}
```
（3）如果在类前加上 @objcMembers，那么它、它的子类、扩展里的方法都会隐式的加上 @objc。
```
@objcMembers
class MyClass: NSObject {
    func print() { } //包含隐式的 @objc
    func show() { } //包含隐式的 @objc
}
 
extension MyClass {
    func baz() { } //包含隐式的 @objc
}
```
（4）如果在扩展（extension）前加上 @objc，那么该扩展里的方法都会隐式的加上 @objc。
```
class SwiftClass { }
 
@objc extension SwiftClass {
    func foo() { } //包含隐式的 @objc
    func bar() { } //包含隐式的 @objc
}
```
（5）如果在扩展（extension）前加上 @nonobjc，那么该扩展里的方法都不会隐式的加上 @objc。
```
@objcMembers
class MyClass : NSObject {
    func wibble() { } //包含隐式的 @objc
}
 
@nonobjc extension MyClass {
    func wobble() { } //不会包含隐式的 @objc
}
```

