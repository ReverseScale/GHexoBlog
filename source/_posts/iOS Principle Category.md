---
title: iOS Principle：Category
layout: post
date: 2018-06-03 21:56:27
comments: true
categories:
	- Principle
keywords: Principle
tags:
	- iOS
description: 

---
分类就是对一个类的功能进行扩展,让这个类能够适应不不同情况的需求.在一般的实际开发中,我们都会对系统的一些常用类进行扩展,比如,NSString,Button,Label等等,简单来说类别是一种为现有的类添加新方法的方式~ 

<!-- more -->
------
👨🏻‍💻 [Github Demo](https://github.com/ReverseScale/iOSPrinciple_Category)

### 方便记忆
* 调用顺序：先调用类的load方法，再调用分类的load方法
* 为分类添加属性：
    * 实现：RunTime 为 Category 动态关联对象，objc_setAssociatedObject方法，内部调用_object_set_associative_reference函数
    * 原理：关联对象并不是放在了原来的对象里面，而是自己维护了一个全局的map用来存放每一个对象及其对应关联属性表格

---

### 目录结构
![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271018ebeb0?w=233&h=122&f=png&s=15513)

我们之前讲到过实例对象的isa指针指向类对象，类对象的isa指针指向元类对象，当p调用run方法时，类对象的isa指针找到类对象的isa指针，然后在类对象中查找对象方法，如果没有找到，就通过类对象的superclass指针找到父类对象，接着去寻找run方法。

![](https://user-gold-cdn.xitu.io/2018/5/30/163b027141542dcc?w=454&h=475&f=png&s=74810)

### Category 的底层实现

将Preson+Test.m文件转化为c++文件，查看其中的编译过程

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc Person+Test.m
```

在分类转化为c++文件中可以看出_category_t结构体中，存放着类名，对象方法列表，类方法列表，协议列表，以及属性列表

```c++
struct _category_t {
    const char *name;
    struct _class_t *cls;
    const struct _method_list_t *instance_methods;
    const struct _method_list_t *class_methods;
    const struct _protocol_list_t *protocols;
    const struct _prop_list_t *properties;
};
```

紧接着，我们可以看到_method_list_t类型的结构体

```c++
static struct /*_method_list_t*/ {
    unsigned int entsize;  // sizeof(struct _objc_method)
    unsigned int method_count;
    struct _objc_method method_list[3];
} _OBJC_$_CATEGORY_INSTANCE_METHODS_Person_$_Test __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_objc_method),
    3,
    {{(struct objc_selector *)"test", "v16@0:8", (void *)_I_Person_Test_test},
    {(struct objc_selector *)"setAge:", "v20@0:8i16", (void *)_I_Person_Test_setAge_},
    {(struct objc_selector *)"age", "i16@0:8", (void *)_I_Person_Test_age}}
};
```

从中我们发现这个结构体_OBJC_CATEGORY_INSTANCE_METHODS_Preson_Test从名称可以看出是INSTANCE_METHODS对象方法，并且一一对应为上面结构体内赋值。我们可以看到结构体中存储了方法占用的内存，方法数量，以及方法列表。并且找到分类中我们实现对应的对象方法，test , setAge, age三个方法

```c++
static struct /*_method_list_t*/ {
    unsigned int entsize;  // sizeof(struct _objc_method)
    unsigned int method_count;
    struct _objc_method method_list[1];
} _OBJC_$_CATEGORY_CLASS_METHODS_Person_$_Test __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_objc_method),
    1,
    {{(struct objc_selector *)"abc", "v16@0:8", (void *)_C_Person_Test_abc}}
};
```

同上面对象方法列表一样，这个我们可以看出是类方法列表结构体 _OBJC_CATEGORY_CLASS_METHODS_Preson_Test，同对象方法结构体相同，同样可以看到我们实现的类方法，abc

接下来是协议方法列表

```c++
static struct /*_method_list_t*/ {
    unsigned int entsize;  // sizeof(struct _objc_method)
    unsigned int method_count;
    struct _objc_method method_list[1];
} _OBJC_PROTOCOL_INSTANCE_METHODS_NSCopying __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_objc_method),
    1,
    {{(struct objc_selector *)"copyWithZone:", "@24@0:8^{_NSZone=}16", 0}}
};

struct _protocol_t _OBJC_PROTOCOL_NSCopying __attribute__ ((used)) = {
    0,
    "NSCopying",
    0,
    (const struct method_list_t *)&_OBJC_PROTOCOL_INSTANCE_METHODS_NSCopying,
    0,
    0,
    0,
    0,
    sizeof(_protocol_t),
    0,
    (const char **)&_OBJC_PROTOCOL_METHOD_TYPES_NSCopying
};
struct _protocol_t *_OBJC_LABEL_PROTOCOL_$_NSCopying = &_OBJC_PROTOCOL_NSCopying;

static struct /*_protocol_list_t*/ {
    long protocol_count;  // Note, this is 32/64 bit
    struct _protocol_t *super_protocols[1];
} _OBJC_CATEGORY_PROTOCOLS_$_Person_$_Test __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    1,
    &_OBJC_PROTOCOL_NSCopying
};
```

通过上述源码可以看到先将协议方法通过_method_list_t结构体存储，之后通过_protocol_t结构体存储在_OBJC_CATEGORY_PROTOCOLS_Preson_Test中同_protocol_list_t结构体一一对应，分别为protocol_count 协议数量以及存储了协议方法的_protocol_t结构体

最后我们可以看到属性列表

```c++
static struct /*_prop_list_t*/ {
    unsigned int entsize;  // sizeof(struct _prop_t)
    unsigned int count_of_properties;
    struct _prop_t prop_list[1];
} _OBJC_$_PROP_LIST_Person_$_Test __attribute__ ((used, section ("__DATA,__objc_const"))) = {
    sizeof(_prop_t),
    1,
    {{"age","Ti,N"}}
};
```

属性列表结构体_OBJC_PROP_LIST_Preson_Test同_prop_list_t结构体对应，存储属性的占用空间，属性属性数量，以及属性列表，可以看到我们自己写的age属性。

最后我们可以看到定义了_OBJC_CATEGORY_Preson_Test结构体，并且将我们上面着重分析的结构体一一赋值，我们通过两张图片对照一下。

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271e542a619?w=689&h=186&f=png&s=39496)

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271ac5f40c6?w=1194&h=311&f=png&s=106186)

上下两张图一一对应，并且我们看到定义_class_t类型的OBJC_CLASS_Preson结构体，最后将_OBJC_CATEGORY_Preson_Test的cls指针指向OBJC_CLASS_Preson结构体地址。我们这里可以看出，cls指针指向的应该是分类的主类类对象的地址。

通过以上分析我们发现。分类源码中确实是将我们定义的对象方法，类方法，属性等都存放在catagory_t结构体中。通过 runtime 源码查看catagory_t存储的方法，属性，协议等我们得知，分类的实现原理是将category中的方法，属性，协议数据放在category_t结构体中，然后将结构体内的方法列表拷贝到类对象的方法列表中。

Category可以添加属性，但是并不会自动生成成员变量及set/get方法。因为category_t结构体中并不存在成员变量。通过之前对对象的分析我们知道成员变量是存放在实例对象中的，并且编译的那一刻就已经决定好了。而分类是在运行时才去加载的。那么我们就无法再程序运行时将分类的成员变量中添加到实例对象的结构体中。因此分类中不可以添加成员变量。

### load 和 initialize
load方法会在程序启动就会调用，当装载类信息的时候就会调用。 调用顺序看一下源代码。

![](https://user-gold-cdn.xitu.io/2018/5/30/163b027197f8e9f7?w=819&h=652&f=png&s=125623)

通过源码我们发现是优先调用类的load方法，之后调用分类的load方法

我们通过代码验证一下： 我们添加Student继承Presen类，并添加Student+Test分类，分别重写只+load方法，其他什么都不做通过打印发现

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271c22a8480?w=794&h=127&f=png&s=34340)

确实是优先调用类的load方法之后调用分类的load方法，不过调用类的load方法之前会保证其父类已经调用过load方法。
之后我们为Preson、Student 、Student+Test 添加initialize方法。

我们知道当类第一次接收到消息时，就会调用initialize，相当于第一次使用类的时候就会调用initialize方法。调用子类的initialize之前，会先保证调用父类的initialize方法。如果之前已经调用过initialize，就不会再调用initialize方法了。当分类重写initialize方法时会先调用分类的方法。但是load方法并不会被覆盖，首先我们来看一下initialize的源码。

![](https://user-gold-cdn.xitu.io/2018/5/30/163b02710017ea4c?w=669&h=137&f=png&s=18874)

上图中我们发现，initialize是通过消息发送机制调用的，消息发送机制通过isa指针找到对应的方法与实现，因此先找到分类方法中的实现，会优先调用分类方法中的实现

我们再来看一下load方法的调用源码

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271d138b646?w=768&h=585&f=png&s=97761)

我们看到load方法中直接拿到load方法的内存地址直接调用方法，不在是通过消息发送机制调用

![](https://user-gold-cdn.xitu.io/2018/5/30/163b02712e3850a4?w=711&h=412&f=png&s=71504)

我们可以看到分类中也是通过直接拿到load方法的地址进行调用。因此正如我们之前试验的一样，分类中重写load方法，并不会优先调用分类的load方法，而不调用本类中的load方法了。


#### RunTime 为 Category 动态关联对象

使用RunTime给系统的类添加属性，首先需要了解对象与属性的关系。我们通过之前的学习知道，对象一开始初始化的时候其属性为nil，给属性赋值其实就是让属性指向一块存储内容的内存，使这个对象的属性跟这块内存产生一种关联。

那么如果想动态的添加属性，其实就是动态的产生某种关联就好了。而想要给系统的类添加属性，只能通过分类。

这里给NSObject添加name属性，创建NSObject的分类

我们可以使用@property给分类添加属性

```objc
@property(nonatomic,strong)NSString *name;
```

通过探寻Category的本质我们知道，虽然在分类中可以写@property
添加属性，但是不会自动生成私有属性，也不会生成set,get方法的实现，只会生成set,get的声明，需要我们自己去实现。

方法一：我们可以通过使用静态全局变量给分类添加属性

```objc
static NSString *_name;
- (void)setName:(NSString *)name {
    _name = name;
}
- (NSString *)name {
    return _name;
}
```

但是这样_name静态全局变量与类并没有关联，无论对象创建与销毁，只要程序在运行_name变量就存在，并不是真正意义上的属性。

方法二：使用RunTime动态添加属性
RunTime提供了动态添加属性和获得属性的方法。

```objc
-(void)setName:(NSString *)name {
    objc_setAssociatedObject(self, @"name",name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
-(NSString *)name {
    return objc_getAssociatedObject(self, @"name");    
}
```

* 1.动态添加属性

objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);

参数一：id object : 给哪个对象添加属性，这里要给自己添加属性，用self。

参数二：void * == id key : 属性名，根据key获取关联对象的属性的值，在**objc_getAssociatedObject中通过次key获得属性的值并返回。

参数三：id value** : 关联的值，也就是set方法传入的值给属性去保存。

参数四：objc_AssociationPolicy policy : 策略，属性以什么形式保存。

有以下几种
```c++
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,  // 指定一个弱引用相关联的对象
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, // 指定相关对象的强引用，非原子性
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  // 指定相关的对象被复制，非原子性
    OBJC_ASSOCIATION_RETAIN = 01401,  // 指定相关对象的强引用，原子性
    OBJC_ASSOCIATION_COPY = 01403     // 指定相关的对象被复制，原子性   
};
```
key值只要是一个指针即可，我们可以传入@selector(name)

* 2.获得属性
```c++
objc_getAssociatedObject(id object, const void *key);
```
参数一：id object : 获取哪个对象里面的关联的属性。

参数二：void * == id key : 什么属性，与**objc_setAssociatedObject**中的key相对应，即通过key值取出value。

* 3.移除所有关联对象

```objc
- (void)removeAssociatedObjects {
    // 移除所有关联对象
    objc_removeAssociatedObjects(self);
}
```

此时已经成功给NSObject添加name属性，并且NSObject对象可以通过点语法为属性赋值。

```objc
NSObject *objc = [[NSObject alloc]init];
objc.name = @"xx_cc";
NSLog(@"%@",objc.name);
```

可以看出关联对象的使用非常简单，接下来我们来探寻关联对象的底层原理

*objc_setAssociatedObject函数*

来到runtime源码，首先找到objc_setAssociatedObject函数，看一下其实现

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271bfaff32c?w=1030&h=132&f=png&s=28294)

我们看到其实内部调用的是_object_set_associative_reference函数，我们来到_object_set_associative_reference函数中

*_object_set_associative_reference函数*

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271011fa4b2?w=949&h=983&f=png&s=249605)

_object_set_associative_reference函数内部我们可以全部找到我们上面说过的实现关联对象技术的核心对象。接下来我们来一个一个看其内部实现原理探寻他们之间的关系。

*AssociationsManager*

通过AssociationsManager内部源码发现，AssociationsManager内部有一个AssociationsHashMap对象。

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271647c8b15?w=850&h=293&f=png&s=68604)

*AssociationsHashMap*

我们来看一下AssociationsHashMap内部的源码。

![](https://user-gold-cdn.xitu.io/2018/5/30/163b027100cc5661?w=1042&h=469&f=png&s=182251)

通过AssociationsHashMap内部源码我们发现AssociationsHashMap继承自unordered_map首先来看一下unordered_map内的源码

![](https://user-gold-cdn.xitu.io/2018/5/30/163b02713af27171?w=962&h=425&f=png&s=129415)

从unordered_map源码中我们可以看出_Key和_Tp也就是前两个参数对应着map中的Key和Value，那么对照上面AssociationsHashMap内源码发现_Key中传入的是disguised_ptr_t，_Tp中传入的值则为ObjectAssociationMap*。

紧接着我们来到ObjectAssociationMap中，上图中ObjectAssociationMap已经标记出，我们发现ObjectAssociationMap中同样以key、Value的方式存储着ObjcAssociation。

接着我们来到ObjcAssociation中

![](https://user-gold-cdn.xitu.io/2018/5/30/163b02713750dec6?w=898&h=283&f=png&s=60731)

我们发现ObjcAssociation存储着_policy、_value，而这两个值我们可以发现正是我们调用objc_setAssociatedObject函数传入的值，也就是说我们在调用objc_setAssociatedObject函数中传入的value和policy这两个值最终是存储在ObjcAssociation中的。

现在我们已经对AssociationsManager、 AssociationsHashMap、 ObjectAssociationMap、ObjcAssociation四个对象之间的关系有了简单的认识，那么接下来我们来细读源码，看一下objc_setAssociatedObject函数中传入的四个参数分别放在哪个对象中充当什么作用。

重新回到_object_set_associative_reference函数实现中

![](https://user-gold-cdn.xitu.io/2018/5/30/163b027186e67b97?w=949&h=983&f=png&s=249605)

细读上述源码我们可以发现，首先根据我们传入的value经过acquireValue函数处理获取new_value。acquireValue函数内部其实是通过对策略的判断返回不同的值

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271777e843d?w=648&h=207&f=png&s=45734)

之后创建 AssociationsManager manager 以及拿到manager内部的AssociationsHashMap即associations。
之后我们看到了我们传入的第一个参数object，object经过DISGUISE函数被转化为了disguised_ptr_t类型的disguised_object。

![](https://user-gold-cdn.xitu.io/2018/5/30/163b02718374b51e?w=786&h=84&f=png&s=30558)

DISGUISE函数其实仅仅对object做了位运算

之后我们看到被处理成new_value的value，同policy被存入了ObjcAssociation中。

而ObjcAssociation对应我们传入的key被存入了ObjectAssociationMap中。

disguised_object和ObjectAssociationMap则以key-value的形式对应存储在associations中也就是AssociationsHashMap中。

![](https://user-gold-cdn.xitu.io/2018/5/30/163b027100d8e5b5?w=699&h=114&f=png&s=37575)

如果我们value设置为nil的话那么会执行下面的代码

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271a59c0f7e?w=830&h=219&f=png&s=51262)

从上述代码中可以看出，如果我们设置value为nil时，就会将关联对象从ObjectAssociationMap中移除。

最后我们通过一张图可以很清晰的理清楚其中的关系

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0270ff216783?w=1016&h=644&f=png&s=169323)

通过上图我们可以总结为：一个实例对象就对应一个ObjectAssociationMap，而ObjectAssociationMap中存储着多个此实例对象的关联对象的key以及ObjcAssociation，为ObjcAssociation中存储着关联对象的value和policy策略。

由此我们可以知道关联对象并不是放在了原来的对象里面，而是自己维护了一个全局的map用来存放每一个对象及其对应关联属性表格。

*objc_getAssociatedObject函数*

objc_getAssociatedObject内部调用的是_object_get_associative_reference

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271ae37e617?w=732&h=98&f=png&s=21314)

*_object_get_associative_reference函数*

![](https://user-gold-cdn.xitu.io/2018/5/30/163b02713e4432f8?w=997&h=590&f=png&s=191847)

从_object_get_associative_reference函数内部可以看出，向set方法中那样，反向将value一层一层取出最后return出去。

*objc_removeAssociatedObjects函数*

objc_removeAssociatedObjects用来删除所有的关联对象，objc_removeAssociatedObjects函数内部调用的是_object_remove_assocations函数

![](https://user-gold-cdn.xitu.io/2018/5/30/163b0271963fb5bc?w=566&h=146&f=png&s=24944)

*_object_remove_assocations函数*

![](https://user-gold-cdn.xitu.io/2018/5/30/163b027171706923?w=1125&h=508&f=png&s=144958)

上述源码可以看出_object_remove_assocations函数将object对象向对应的所有关联对象全部删除。

关联对象并不是存储在被关联对象本身内存中，而是存储在全局的统一的一个AssociationsManager中，如果设置关联对象为nil，就相当于是移除关联对象。

此时我们我们在回过头来看objc_AssociationPolicy policy 参数: 属性以什么形式保存的策略。

```c++
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,  // 指定一个弱引用相关联的对象
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, // 指定相关对象的强引用，非原子性
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  // 指定相关的对象被复制，非原子性
    OBJC_ASSOCIATION_RETAIN = 01401,  // 指定相关对象的强引用，原子性
    OBJC_ASSOCIATION_COPY = 01403     // 指定相关的对象被复制，原子性   
};
```

我们会发现其中只有RETAIN和COPY而为什么没有weak呢？
总过上面对源码的分析我们知道，object经过DISGUISE函数被转化为了disguised_ptr_t类型的disguised_object。

```c++
disguised_ptr_t disguised_object = DISGUISE(object);
```

而同时我们知道，weak修饰的属性，当没有拥有对象之后就会被销毁，并且指针置位nil，那么在对象销毁之后，虽然在map中既然存在值object对应的AssociationsHashMap，但是因为object地址已经被置位nil，会造成坏地址访问而无法根据object对象的地址转化为disguised_object了。

#### 相关问题

问：Category中有load方法吗？load方法是什么时候调用的？load 方法能继承吗？

答：Category中有load方法，load方法在程序启动装载类信息的时候就会调用。load方法可以继承。调用子类的load方法之前，会先调用父类的load方法

问：load、initialize的区别，以及它们在category重写的时候的调用的次序。

答：区别在于调用方式和调用时刻

* 调用方式：load是根据函数地址直接调用，initialize是通过objc_msgSend调用
* 调用时刻：load是runtime加载类、分类的时候调用（只会调用1次），initialize是类第一次接收到消息的时候调用，每一个类只会initialize一次（父类的initialize方法可能会被调用多次）
* 调用顺序：先调用类的load方法，先编译那个类，就先调用load。在调用load之前会先调用父类的load方法。分类中load方法不会覆盖本类的load方法，先编译的分类优先调用load方法。initialize先初始化父类，之后再初始化子类。如果子类没有实现+initialize，会调用父类的+initialize（所以父类的+initialize可能会被调用多次），如果分类实现了+initialize，就覆盖类本身的+initialize调用。

> 以上文章整理自：https://juejin.im/post/5aef0a3b518825670f7bc0f3、https://juejin.im/post/5af86b276fb9a07aa34a59e6