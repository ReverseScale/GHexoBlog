---
title: Swift 4.0 中对 Dictionary 的改进
layout: post
date: 2018-03-13 21:56:27
comments: true
categories:
	- Tips
keywords: Swift
tags:
	- Tips
description: 

---
Swift 4 发布已经有一段时间了，不知道大家有没有切换到 4.0 版本。 这次 4.0 更新给我最大的感受就是没有了前几次升级的跳跃式变化。 不用为了更新语言版本，完全推翻已有的项目，这也是 Swift 慢慢趋向于稳定的标志~

<!-- more -->
咱们这次说说 Swift 4.0 对 Dictionary 这个经常会用到的类的改进。

### 自动根据 key 分组
Dictionary 新增了一个构造方法，可以将给定的一个数组，根据指定的条件进行分组。 来看一个例子:

```Swift
struct Person {
    var name: String
    var gender: Gender
    var age: Int
    enum Gender {
        case male
        case female
    }
}
```

这里有一个 Person 结构, 然后我们初始化一个数组：

```Swift
let p1 = Person(name: "aa", gender: .female, age: 22)
let p2 = Person(name: "bb", gender: .male, age: 24)
let p3 = Person(name: "cc", gender: .male, age: 21)

let persons = [p1, p2, p3]
```

现在用新的 Dictionary 构造方法，可以立即将这组 Person 实例根据他们的 gender 属性进行分组：

```Swift
let groupedDict = Dictionary(grouping: persons) { p in
    return p.gender
}
```

这个构造方法，第一个参数 grouping 接收的是 persons 数组， 第二个参数是一个闭包，用于返回根据进行分组的依据，我们这里返回的是 p.gender。结果一目了然：

```Swift
[
    .male: [
        Person(name: "bb", gender: .male, age: 24), 
        Person(name: "cc", gender: .male, age: 21)
    ], 
    .female: [
        Person(name: "aa", gender: .female, age: 22)
    ]
]
```

以往要实现这样的功能，就需要手动遍历整个数组，取出 key， 然后生成字典， 现在方便很多。

### value 无缝转换
另外一个比较有用的特性是，Dictionary 提供的 mapValues 方法。 还以我们刚才生成的 groupedDict 为例，可以进行这样的操作：

```Swift
let count = groupedDict.mapValues { persons in
    persons.count
}
```

先看一下输出，大家可能就猜到这个方法的用途是什么了：

```Swift
[
    .male: 2, 
    .female: 1
]
```

上面的输出可以看到， mapValues 会遍历每一个 key 对应的 values， 然后传递给闭包进行自定义转换。 我们例子中的闭包返回的就是每个 key 对应的 Person 集合的数量，最生成了一个新的 Dictionary，里面的 key 和之前一样，只是对应的值变成了我们闭包中自定义的了。

mapValues 方法同样是一个帮助我们解决繁杂操作的工具方法。

### 从键值对元组中直接构建
假如我们有这样一个数组:

```Swift
let personsTuples = [("group 1", [p1, p2]),  ("group 2", [p3])]
```

然后可以调用 uniqueKeysWithValues 构造方法直接初始化字典：

```Swift
let dict = Dictionary(uniqueKeysWithValues: personsTuples)
```

personsTuples 数组中，每一个元素都是一个元组(Tuple)， 这个构造方法把元组中的第一项当做 key， 第二项当做 value， 生成一个新的 Dictionary，如下所示：

```Swift
[
    "group 1": [
        Person(name: "aa", gender: .female, age: 22), 
        Person(name: "bb", gender: .male, age: 24)
    ], 
    "group 2": [
        Person(name: "cc", gender: .male, age: 21)
    ]
]
```

使用 uniqueKeysWithValues 构造方法时候需要注意，就是传入的数组中，不能有重复的 key， 否则会报运行时错误，比如这个数组就会报错：

```Swift
let personsTuples = [("group 1", [p1, p2]),  ("group 2", [p3]), ("group 2", [p4])]
```

上面数组中 group 2 出现了两次。 如果用它来初始化的话，就会出错。 所以 Swift 4.0 还提供了另外一个初始化方法， 对于上面这个数组，可以调用：

```Swift
let dict = Dictionary(personsTuples) { old, new in
    return new
}
```

第二个闭包参数，会在遇到重复的 key 时候调用。 它提供两个参数，一个是同样这个 key 的上一个值 old， 还有当前的值 new。 我们这里直接返回 new，意思就是 每次遇到重复的 key， 就用新的值代替老的值。 这样初始化后，生成的 Dictionary 结构如下：

```Swift
[
    "group 1": [
            p1,
            p2
    ], 
    "group 2": [
            p4
    ]
]
```

group 2 中的值， 是第二次 key 所对应的 p4。

### 提供默认值
如果我们访问了一个字典中不存在的 key， 会返回 nil：

```Swift
dict["group 3"]
```

因为上面字典中，不存在 group 3 这个 key， 所以它返回了 nil。 Swift 4 中新增了指定默认值的能力：

```Swift
dict["group 3", default: []]
```

这样调用，如果 group 3 这个 key 不存在的话， 就会返回我们指定的默认值空数组， 而不是 nil 了。 这个特性在我们处理 JSON 数据解析这类的问题上很实用。

### 过滤器
过滤器对于集合类来说是比较常用的功能。 Swift 4 中对 Dictionary 类型也提供了过滤器的支持：

```Swift
let filteredDict = dict.filter { key, val in
    return val.count >= 2
}
```

filter 方法接收一个闭包，它会遍历 Dictionary 中所有的元素，并且作为闭包的参数传入。 我们需要通过闭包的返回值确定这个元素是否被保留。 上面的例子中，比如我们 dict 中的元素是 ：

```Swift
[
    "group 1": [
            p1,
            p2
    ], 
    "group 2": [
            p4
    ]
]
```

调用 filter 后，我们只保留数量大于 2 的集合：

```Swift
[
    "group 1": [
            p1,
            p2
    ]
]
```

### 总结
以上就是 Swift 4.0 对于 Dictionary 的主要改进。 这些新增的工具方法对于提高我们开发效率和代码质量都有帮助，希望这里的介绍对你有帮助。 关于 Dictionary 更完整的信息，大家还可以参考苹果官方的文档，还有 Swift Blog 中的介绍。

### 参考文献
Dictionary 官方文档：https://developer.apple.com/documentation/swift/dictionary 
Swift Blog: https://swift.org/blog/dictionary-and-set-improvements/
