---
title: iOS Principle：Block
layout: post
date: 2018-06-10 21:56:27
comments: true
categories:
	- Principle
keywords: Principle
tags:
	- iOS
description: 

---
Block 其实就是 C 语言的扩充功能，实现了对 C 的闭包实现，一个带有局部变量的匿名函数~ 

<!-- more -->
------
👨🏻‍💻 [Github Demo](https://github.com/ReverseScale/iOSPrinciple_Block)

### 方便记忆：
* Block 实现了对C的闭包实现，一个带有局部变量的匿名函数
* 源码结构分析
    * 本体部分：block实际的结构体部分
    * 成员变量：impl和Desc
    * 结构体的构造函数：__main_block_impl_0
* 捕获外部变量
    * 局部变量作为Block结构体的成员变量追加到了__main_block_impl_0
    * 根据传递给构造函数的参数对由局部变量追加的成员变量进行初始化
    * __main_block_func_0中进行了变量获取_cself->ch(变量的值-深拷贝)
* 改变Block里的值
    * c方法：静态变量、静态全局变量、全局变量
    * __block修饰符：变成了结构体实例，初始化_main_block_impl_0的时候追加的是_block生成的结构体指针
* 三种Block本体
    * NSConcreteStackBlock：在局部定义的（栈）
    * NSConcreteGlobalBlock：在全局中定义的（数据区域）-不存在对局部变量的捕获
    * NSConcreteMallocBlock：分配在堆中-Block从栈上复制到堆上的方法

---

### 声明一个Block:
#### 使用场景一：
返回值 (^名称) (参数列表) = ^(参数列表) {
};

```objc
int (^name)(int, int) = ^(int a, int b) {
    return (a+b);
};
```
#### 使用场景二：
作为一个函数的参数：

```objc
- (void)testBlock:(NSString *(返回类型) (^)(int a))s (block名字） {
    NSString *a = s(1);
}

{
    [self testBlock:^NSString *(int a) {
        a = 5;
        return @"1";
    }];
}
```

### Demo 中文件对照

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef4658d95a21?w=427&h=75&f=png&s=13706)

* mainTestBlock.cpp -> 基本的block转译
* mainTestBlockValue.cpp -> block捕获外部变量
* mainTestBlockValueC.cpp -> 通过C语言变量访问值
* mainTestBlockValueValueBlock.cpp -> 通过__block变量访问值

### 从底层分析Block的实现
#### 先从最简单的看起

```objc
void (^block)(void) = ^(void) {
    
};
block();  
```

使用 Clang（LLVM）转化为 C++ 的实现
```
clang -rewrite-objc 文件名
```
注意：由于 ViewController 中使用 UIKit 库，编译时会出现找不到文件的情况。

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef465e456757?w=465&h=59&f=png&s=14449)

转化后的代码如下：
```c++
struct __block_impl {  
    void *isa;  
    int Flags;  
    int Reserved;  
    void *FuncPtr;  
};  
struct __main_block_impl_0 {  
    struct __block_impl impl;  
    struct __main_block_desc_0* Desc;  
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {  
        impl.isa = &_NSConcreteStackBlock;  
        impl.Flags = flags;  
        impl.FuncPtr = fp;  
        Desc = desc;  
    }  
};  
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {  
    printf("aaa\n");
}  
static struct __main_block_desc_0 {  
    size_t reserved;  
    size_t Block_size;  
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};  
int main() {  
    void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));  
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);  
    return 0;  
}  
```

#### 源码结构分析
1.block实际的结构体部分（本体）
```c++
struct __main_block_impl_0 {  
    struct __block_impl impl; // 结构体 impl，见下 2
    struct __main_block_desc_0* Desc; // 结构体 Desc，见下 3
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {  // 构造函数，见下 4
        impl.isa = &_NSConcreteStackBlock;  
        impl.Flags = flags;  
        impl.FuncPtr = fp;  
        Desc = desc;  
    }  
};  
```

2.再看看第一个成员变量impl
```c++
struct __block_impl {  
    void *isa; // struct 中，首地址是 *isa，证明 block 是 objc 中的对象
    int Flags; // 标记
    int Reserved; // 标记
    void *FuncPtr; // 函数指针，block所需要执行的代码段
};  
```
他的第一个属性也是一个结构__block_impl，而第一个参数也是一个isa的指针。

在运行时，NSObject和block的isa指针都是指向对象的一个8字节。
NSObject及派生类对象的isa指向Class的prototype，而block的isa指向了_NSConcreteStackBlock这个指针。
就是说当一个block被声明的时候他都是一个_NSConcreteStackBlock类的对象。

3.第二个成员变量Desc
```c++
static struct __main_block_desc_0 {  
    size_t reserved; // 版本升级所需的区域
    size_t Block_size;  // Block的占用空间大小
}
```

4.第三个就是这个结构体的构造函数（可以理解为对象的初始化方法）
```c++
__main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {  
    impl.isa = &_NSConcreteStackBlock; // block 的本体类型，后面专门讲解
    impl.Flags = flags;  
    impl.FuncPtr = fp; // void *fp的指针赋值给了FuncPtr指针
    Desc = desc; // desc 结构体的指针传入的初始化
} 
```

> 以下是源码调用分析

5.现在来看看Main函数中调用的基本转换（初始化转换）
```c++
void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA)); 
```
看起来这个函数好吓人啊，没事，咱们把类型都去了，而且分开两步来写
```c++
/* 调用结构体函数初始化   
struct __main_block_impl_0 impBlock = __main_block_impl_0(__main_block_func_0,&__main_block_desc_0_DATA);  

/* 赋值给该结构体类型指针变量  
struct __main_block_impl_0 *block = &impBlock;  
```
把栈上生成的__main_block_impl_0的结构体实例的指针，赋值给__main_block_impl_0结构体指针类型的变量block

__main_block_func_0就是转换成的函数指针，这样void (*block)(void) = ^{}这句简单的代码最终就是上面的结构体初始化函数的内部实现逻辑

6.再来细看下初始化函数内部的实现__block_main_impl_0
```c++
__main_block_impl_0(__main_block_func_0,&__main_block_desc_0_DATA);  
```

第一个参数是由Block块语法转换的正真内部函数指针，第二个参数就是作为静态全局变量初始化__main_block_desc_0的结构体实例指针，初始化如下
```c++
__main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)}; 
```

最终，初始化内部进行赋值
```c++
isa = &_NSConcreteStackBlock
Flags = 0
Reversed = 0
FuncPtr = __main_blcok_func_0 // 就是Block块代码转换成的C语言函数指针
Desc = &__ main_block_desc_0_DATA
```

7.最终 block() 调用的内部实现
```c++
((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);  
```
老规矩 ，去掉类型就是
```c++
(*block->imp.FuncPtr)(block);
```
打印换行
```c++
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {  
printf("aaa\n");} 
```
没错，最后就是直接调用函数指针进行最终的调用，由上面所描述的，FuncPtr就是由__main_block_func_0的函数指针所赋值的指针，而且可以看出，这个函数的参数正是指向block本身结构体实例，block作为参数进行了传递

综上所述，block的实质，就是一个对象，包含了一个指向函数首地址的指针，和一些与自己相关的成员变量。


### Block 访问外部变量是怎么回事
```objc
int main(){  
    int a = 100;  
    int b = 200;  
    const char *ch = "b = %d\n";  
    void (^block)(void) = ^{   
    };  

    b = 300;  
    ch = "value had changed.b = %d\n";  
    block();  
    return 0;  
}  
```
Clang 转化后代码
```c++
struct __block_impl {
    void *isa;
    int Flags;
    int Reserved;
    void *FuncPtr;
};
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    const char *ch;
    int b;
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, const char *_ch, int _b, int flags=0) : ch(_ch), b(_b) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
};
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    const char *ch = __cself->ch; // bound by copy
    int b = __cself->b; // bound by copy
    printf(ch,b);
}
static struct __main_block_desc_0 {
    size_t reserved;
    size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
int main(){
    int a = 100;
    int b = 200;
    const char *ch = "b = %d\n";
    void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, ch, b));
    b = 300;
    ch = "value had changed.b = %d\n";
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
    return 0;
}
```

1.首先看看Block的内部结构本尊和没有截获的区别
```c++
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    const char *ch;
    int b;
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, const char *_ch, int _b, int flags=0) : ch(_ch), b(_b) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
};
```
根据main函数里面的Block语法，这里把Block块表达式里面的局部变量作为了这个Block结构体的成员变量追加到了__main_block_impl_0的结构体中

值得注意的是：
* 结构体内声明的成员变量类型与局部变量类型完全相同
* 语法中没有使用的局部变量（例如咱们这里的a变量）不会被追加
* 细节需要注意，这里截获的ch是不可修改的，而且捕捉的b只是截获了该局部变量的值而已（下面在将如何截获指针）

2.再来看看该结构体实例化的构造函数以及调用和没有截获的区别
```c++
__main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, const charchar *_ch, int _b, int flags=0) : ch(_ch), b(_b) {  
    impl.isa = &_NSConcreteStackBlock;  
    impl.Flags = flags;  
    impl.FuncPtr = fp;  
    Desc = desc;  
}  
void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, ch, b));  
```
根据传递给构造函数的参数对由局部变量追加的成员变量进行初始化。（这里传递的参数仅仅只是ch和b的值而已）

理解为把Block语法块里面截获的局部变量对__main_block_impl_0的成员追加并且传递赋值，如：
```c++
impl.isa = &_NSConcreteStackBlock;
impl.Flags = 0;
impl.FuncPtr = __main_block_func_0;
Desc = &__main_block_desc_0_DATA;
ch = "b=%d\n";
b = 200;
```
由此可以看出，__main_block_impl_0的结构体（即Block）对自动变量进行了截获

3.再来看看最终调用Block的时候和没有截获的区别
```c++
((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);  
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {  
    const charchar *ch = __cself->ch; // bound by copy  
    int b = __cself->b; // bound by copy  
    printf(ch,b);  
}  
```

可以看出block()这句代码转换的调用方式没有任何变化
还是转换为一下这句话，而且把本身结构体作为参数进行了传递
```c++
(*block-impl.FuncPtr)(block)
```
现在注意看__main_block_func_0这个C语言的函数，和之前相比这里对传递过来的block结构体参数进行了变量获取__cself->ch 和 __cself->b（这两个变量已经在Block表达式之前进行了声明定义），并最终打印出来

但是，Block中所捕获的变量就犹如“带有局部变量值的匿名函数”所说，仅仅截获局部变量的值而已，如果在截获的Block里面重写局部变量也不会改变原先所截获的局部变量

block 引用外部对象时候，不是简单的指针引用（浅复制），而是一种重建（深复制）方式（括号内外分别对于基本数据类型和对象分别描述）

所以如果在 block 中对外部对象进行修改，无论是值修改还是指针修改，自然是没有任何效果

例如：
```objc
int a = 0；
void (^block)(void) = ^{a = 1};
```

这样写直接就编译出错了！！！

可以直接看__block_main_block_impl_0的实现上，并不能改写其捕获变量的值，因此直接报错了

综上所述，所谓的“截获自动变量”，无非就是在执行Block语法的时候，Block语法所用到的局部变量值被保存到Block的结构体实例当中（即所谓的Block本体）


### 要在 Block 中改变里面的值
#### 第一种方法：运用C语言中的变量
* 静态变量
* 静态全局变量
* 全局变量

```objc
int global_val = 1; // 全局变量  
static int static_global_val = 2; // 静态全局  
int main(){  
    static int static_val = 3; // 静态局部  
    void (^block)(void) = ^{  
        global_val *= 1;  
        static_global_val *= 2;  
        static_val *= 3;  
    };  
    block(); 
}
```
这里全局和静态全局问题不大，出了作用于照样能访问，但是静态局部的话如果去掉Static按上面那种写法，就直接报错了，原因就是实现上不能改变捕获的局部变量的值。

没错，应该能想到了，能改变的情况就是直接让Block截获局部变量的指针，看Clang的源码
```c++
int global_val = 1;  
static int static_global_val = 2;  
struct __main_block_impl_0 {  
    struct __block_impl impl;  
    struct __main_block_desc_0* Desc;  
    intint *static_val;  
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, intint *_static_val, int flags=0) : static_val(_static_val) {  
        impl.isa = &_NSConcreteStackBlock;  
        impl.Flags = flags;  
        impl.FuncPtr = fp;  
        Desc = desc;  
    }  
};  
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {  
    intint *static_val = __cself->static_val; // bound by copy  
    global_val *= 1;  
    static_global_val *= 2;  
    (*static_val) *= 3;  
}  
static struct __main_block_desc_0 {  
    size_t reserved;  
    size_t Block_size;  
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};  
int main(){  
    static int static_val = 3;  
    void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, &static_val));  
    ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);  
    printf("%d\n",global_val);  
    printf("%d\n",static_global_val);  
    printf("%d\n",static_val);  
}  
```
很简单就能看出区别了
* 在__main_block_impl_0这个结构体中的追加的成员变量变成了int *_static_val指针了
* 在结构体实例化函数中__main_block_impl_0参数也变为了&static_val地址了

```c++
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {  
    intint *static_val = __cself->static_val; // bound by copy  
    global_val *= 1;  
    static_global_val *= 2;  
    (*static_val) *= 3;  
}  
```
根据Block语法转换而来的C静态函数，使用static_val的指针进行访问，上面参数可以看出，初始化的时候将静态变量的static_val指针传递给__main_block_impl_0结构体进行追加成员变量初始化并保存，这样做也是对超出作用于使用变量的最简单的方法

#### 第二种方法：就是大家所熟知的__block修饰符的使用
```objc
__block int a = 0;
void (^block)(void) = ^{a = 1};
```
Clang 转化后代码
```c++
struct __Block_byref_a_0 {  
    void *__isa;  
    __Block_byref_a_0 *__forwarding;  
    int __flags;  
    int __size;  
    int a;  
};  

struct __main_block_impl_0 {  
    struct __block_impl impl;  
    struct __main_block_desc_0* Desc;  
    __Block_byref_a_0 *a; // by ref  
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_a_0 *_a, int flags=0) : a(_a->__forwarding) {  
        impl.isa = &_NSConcreteStackBlock;  
        impl.Flags = flags;  
        impl.FuncPtr = fp;  
        Desc = desc;  
    }  
};  
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {  
    __Block_byref_a_0 *a = __cself->a; // bound by ref  
    (a->__forwarding->a) = 1;
}  
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {
    _Block_object_assign((void*)&dst->a, (void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);
}  
static void __main_block_dispose_0(struct __main_block_impl_0*src) {
    _Block_object_dispose((void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);
}  

static struct __main_block_desc_0 {  
    size_t reserved;  
    size_t Block_size;  
    void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);  
    void (*dispose)(struct __main_block_impl_0*);  
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};  
int main(){  
    __attribute__((__blocks__(byref))) __Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a, 0, sizeof(__Block_byref_a_0), 0};  
    void(*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_a_0 *)&a, 570425344));  
    return 0; 
}
```

代码多了不少，分解一下
```objc
__block int a = 0；
```
Clang 的代码如下
```c++
__Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a,0,sizeof(__Block_byref_a_0),0};
```

这货竟然加了_block变成了结构体实例，在栈上生成了__block_byref_a_0的结构体实例，a变量初始化为0，这个值也出现在了这个结构体成员变量变量中，意味着该结构体持有相当于原局部变量的成员变量

```c++
struct __Block_byref_a_0 {  
 
 *__isa;  
    __Block_byref_a_0 *__forwarding;  
    int __flags;  
    int __size;  
    int a;  
};  
```
这个初始化的原结构，里面的最后一个int a就是原局部变量的成员变量

```objc
void (^block)(void) = ^{a = 1};
```

分解如下

```c++
void(*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_a_0 *)&a, 570425344));  
```

还是调用__main_block_impl_0的初始化结构体，由于int a加了__block修饰符，那么这个初始化函数传递的不再是&a的地址，而是换成__Block_byref_a_0这个结构体实例的指针进行传递（该结构体其实也是个对象，最后的属性放着需要截获的局部变量）

这里的__main_block_func_0对应的block块函数转换为C

```c
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {  
    __Block_byref_a_0 *a = __cself->a; // bound by ref  
    (a->__forwarding->a) = 1;
}  
```

看了两个方案，第一个用C语言的静态变量来解决，那么直接操作静态变量的指针就完事了，但是__block要比这个真的复杂多了。Block的结构体实例持有指向__block变量的 __Block_byref_a_0结构体实例的指针。

__Block_byref_a_0这个结构体实例的成员变量__forwarding持有指向该结构体实例自身的指针。通过__forwarding访问成员变量a。

另外提一点，__block内部的__Block_byhef_val_0这个也是独立的结构体实例，不属于__main_blocl_impl_0结构体中，这样方便了在多个block中都能访问__block变量。

小结一下，不然太乱：

1.普通的局部变量被Block截获只是单纯的截获其值，并追加到__main_block_impl_0结构体中（无法修改重写）

2.要允许Block修改值有两个方法，第一个就是用C语言的变量，代表就是静态局部变量，第二个就是__block修饰符

3.__block修饰的变量，其实这就是变成了个对象结构体__Block_byref_val_0,初始化__main_block_impl_0的时候追加的是__block生成的结构体指针,例如__block int a = 0;这个int不在是普通的数据类型，而是在标识符的引用下变成了对象

```c++
struct __Block_byref_a_0 {  
    void *__isa;  
    __Block_byref_a_0 *__forwarding;  
    int __flags;  
    int __size;  
    int a;  
};  
```
而int的值就存储在这个对象的一个字段 a中，而Block截获的就是这个结构体对象的指针，可以随意修改结构体内部a的值

4.最终在__main_block_func_0函数中调用__block结构体指针的forwarding指针（这一节指向自己，后面会变）的成员变量a来进行重新赋值

5.其实__block这方法生成的源码，能大致看出来这其实类似OC里面对象的retain和release一系列操作

### 三种 Block 本体

到这个阶段，我们用C的结构编译的代码以及源码能看到Block结构体内部的isa指针是指向_NSConcreteStackBlock的，其实这只是其中的一种，分别还有_NSContreteGlobalBlock 和 _NSContreteMallocBlock,可以根据命名的后缀看出来StackBlock是设置在栈上的，GlobalBlock就类似全局变量，设置在程序的数据区域（.data区域），那么最重要的也是我们写OC代码的时候根本不关注的一种类型NSContreteMallocBlock，没错，他就是和对象一样分类内存块(堆)中。

```objc 
{
    NSConcreteGlobalBlock; // 在全局中定义的（数据区域）
    NSConcreteStackBlock; // 在局部定义的（栈）
    NSConcreteMallocBlock; // 分配在堆中
}
```

#### NSConcreteGlobalBlock 全局定义
上面我们说的都是 NSConcreteStackBlock 在局部定义的（栈），下面我们来看看 NSConcreteGlobalBlock

你可以把它理解为全局变量，反正存储在.data区域的，最直接得写法是这样的

```objc
void (^block)(void) = ^{};  
int main(){
}  
```
Clang 转换过后的源码指针impl.isa = &_NSContreteGlobalBlock类型的

由于在使用全局变量的地方不能使用局部变量，这么说来就根本不存在对局部变量的捕获。那么这个Block的结构体实例的内容压根不会再进行追加成员变量，所以不会依赖于执行状态，所以整个程序运行只有一个实例。因此将Block使用的结构体实例设置在与全局变量相同的数据区域即可。（把它理解为单纯的全局变量就好了，而且不会有任何值的捕获）

存在的两种案例
* 全局变量的地方，用这种Block语法时，如上面所示
* Block语法中表达式不截获任何局部变量时，这个稍后Demo介绍，也很简单

#### NSContreteMallocBlock 堆定义
配置在全局的GlobalBlock可以出了作用域还是能继续访问，但是在栈上的StackBlock就废弃了，因此为了出了作用域能继续使用，Blocks提供了把Block和__block这两个东西从栈上复制到堆上的方法来解决这个问题。而_forwarding其实既可以指向自己，也可以指向复制后的自己，也就是说有了这个指针的存在，无论__block变量配置在堆上还是栈上都能够正确的访问__block变量

一种作为返回值返回的情况
```objc
typedef void(^block)(void);  
block func (int a) {  
    return ^{};  
} 
```
转换后
```c++
block func (int a) {  
    block tmp = ((void (*)())&__func_block_impl_0((void *)__func_block_func_0, &__func_block_desc_0_DATA));  
    tmp = objc_retainBlock(tmp);  
    return objc_autoreleaseReturnValue(tmp);  
} 
```
这里先是通过Block语法生成Block，即生成配置在栈上的Block结构体实例，然后赋值给Block类型的变量tmp中，然后执行objc_retainBlock(tmp)这句其实就是_Block_copy(tmp)，将栈上的的Block复制到堆上，赋值后将堆上的指针赋值给tmp，然后堆上的所有都是对象，需要注册陶autoreleasepool中进行管理，然后返回其对象

其实这种内部调用方式，MallocBlock就是对象，而且这种写法是否很似曾相识，就是类方法实例化

```objc
- (NSArray *) myTestArray {  
    NSArray *array = [[NSArray alloc] initWithObjects: @"a", @"b", @"c", nil nil];  
    return [array autorelease];  
}  
```
简单来说就是第一步copyBlock到堆上，然后和OC对象一样，返回的对象需要进行autorelease防治内存泄露

ARC下面很多都已经自动帮我们Copy成了MallocBlock了，请看一下几种情况


由于Block是默认建立在栈上,所以如果离开方法作用域，Block就会被丢弃，在非ARC情况下，我们要返回一个Block，需要 

```objc
[Block copy];
```

在ARC下,以下几种情况, Block会自动被从栈复制到堆:

* 1.被执行copy方法
* 2.作为方法返回值
* 3.将Block赋值给附有__strong修饰符的id类型的类或者Blcok类型成员变量时
* 4.在方法名中含有usingBlock的Cocoa框架方法或者GDC的API中传递的时候

### 来猜一猜
```objc
int a = 1;  

// 这里的^{}初始化的block赋值给block变量，在OC中没有具体写明的情况下应该就是strong类型的，这就是上面第三点的例子  
// 打印出来 first <__NSMallocBlock__: 0x60800004d290>  
void (^block)(void) = ^{  
   NSLog(@"%d",a);  
};  
NSLog(@"first %@",block);  

// 没有截获变量的时候就是globalBlock  
// second<__NSGlobalBlock__: 0x102f80100>  
NSLog(@"second%@",^{NSLog(@"呵呵");});  

// 截获了变量就是stackBlock  
// third<__NSStackBlock__: 0x7fff5cc7faa0>  
NSLog(@"third%@",^{NSLog(@"%d",a);});  

__block int val = 10;  
__strong blk strongPointerBlock = ^{NSLog(@"val1 = %d", ++val);};  
// strongPointerBlock: <__NSMallocBlock__: 0x600000044e90>  
NSLog(@"strongPointerBlock: %@", strongPointerBlock); //1  

__weak blk weakPointerBlock = ^{NSLog(@"val2 = %d", ++val);};  
// weakPointerBlock: <__NSStackBlock__: 0x7fff5282ea70>  
NSLog(@"weakPointerBlock: %@", weakPointerBlock); //2  

// mallocBlock3: <__NSMallocBlock__: 0x608000046930>  
NSLog(@"mallocBlock3: %@", [weakPointerBlock copy]); //3  

// 截获了test <__NSStackBlock__: 0x7fff5282ea48>  
NSLog(@"test4 %@", ^{NSLog(@"val4 = %d", ++val);}); //4  

// test5 <__NSMallocBlock__: 0x60800005c0b0>  
NSLog(@"test5 %@", [^{NSLog(@"val4 = %d", ++val);} copy]); //5  

// stackBlock经过传参 打印  
NSLog(@"传参后 %@",[self getBlock]);  
}  

- (blk)getBlock {  
int val = 11;  
// 上面已经介绍了，这种直接打印传参前的block，应该是__NSStackBlock__  
NSLog(@"传参前：%@",^{NSLog(@"%d",val);});  

// 那么现在我们直接传出去  
return ^{NSLog(@"%d",val);};  
}  
```

这里把几种情况都打印了一下看看到底是哪个类型

1.第一个和第二个打印的区别在于，第一个生成的Block默认赋值给了block变量，第二个直接打印，由于OC里面没有修饰符默认就是strong，所以这么看来遵循第三条规则之后，第二条条打印就是_NSGlobalBlock_（没有截获变量），第一条打印就是_NSMallocBlock_（自动复制到堆上了）

2.后面用strong修饰符和weak修饰符分别打印的是malloc的和stack的，但是无论哪种，只要copy就是变成malloc类型了

3.最后一种就是上面介绍的，stackBlock经过传参，自动变成了mallocBlock


### 属性修饰符 copy 修饰 Block
来看一张表，看看三种类型copy之后有什么区别

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef465d5b19ef?w=1096&h=83&f=png&s=30210)

```objc
@property (nonatomic,copy) blk blk;
```

当我们这么声明属性的时候，其setter方法就是用了copy方法

```objc
- (void)setBlk:(blk)blk {  
    if (_blk != blk) {  
        [_blk release]; // MRC  
        _blk = [blk copy];  
    }  
}  
```

* 1.超出作用域存在的理由就是生成了MallocBlock对象，即使出栈了还是能继续调用
* 2.forwarding的理由就是无论在堆上还是栈上，我们都能访问Block，而且能保证访问同一个
* 3.GlobalBlock 、MallocBlock和StackBlock的区别以及如何Block如何会被copy到堆上
* 4.特别上作为参数传递时，类似类方法的autoreleasepool注册进去，避免内存泄露
* 5.在正常写代码的时候不需要管理这个，默认百分之99的情况基本都是MallocBlock
* 6.无论什么情况下，copy一下或者强指针引用一下是不会有错的，能保证必然是MallocBlock
* 7.各种迹象表明，他就是一个对象，可以通过copy改变类型的特殊对象

> 以上文章整理自：https://blog.csdn.net/deft_mkjing/article/details/53143076