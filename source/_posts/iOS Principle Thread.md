---
title: iOS Principle：Thread
layout: post
date: 2018-06-09 21:56:27
comments: true
categories:
	- Principle
keywords: Principle
tags:
	- iOS
description: 

---
进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础~ 

<!-- more -->
------
👨🏻‍💻 [Github Demo](https://github.com/ReverseScale/iOSPrinciple_Thread)

### 方便记忆：
* 进程和线程：进程是饭馆、线程是工作人员
* 线程的执行：1个线程中任务的执行是串行的
* 多线程并发执行：CPU快速地在多条线程之间调度
* iOS 多线程
    * pthread：c语言的通用api，跨平台、难度大、非自动管理生命周期
    * NSThread：面向对象的非自动管理生命周期
    * GCD：利用设备的多核自动管理生命周期
    * NSOperation、NSOperationQueue：封装GCD，加强依赖关系、监控状态、并发管理
* 常用安全锁：@synchronized{}、[self.lock lock]

---
### 相关概念

#### 进程

进程是指在系统中正在运行的一个应用程序。每个进程之间是独立的，每个进程均运行在其专用且受保护的内存空间内。

#### 线程

*线程和进程的关系*

1个进程要想执行任务，必须得有线程（每1个进程至少要有1条线程），线程是进程的基本执行单元，一个进程（程序）的所有任务都在线程中执行。

*线程内部是串行执行的？*

1个线程中任务的执行是串行的，如果要在1个线程中执行多个任务，那么只能一个一个地按顺序执行这些任务。也就是说，在同一时间内，1个线程只能执行1个任务。

#### 多线程

即1个进程中可以开启多条线程，每条线程可以并行（同时）执行不同的任务。比如同时开启3条线程分别下载3个文件（分别是文件A、文件B、文件C）。

*多线程并发执行的原理？*

在同一时间里，CPU只能处理1条线程，只有1条线程在工作（执行）。多线程并发（同时）执行，其实是CPU快速地在多条线程之间调度（切换），如果CPU调度线程的时间足够快，就造成了多线程并发执行的假象。

*多线程的优缺点？*

优点：
* 1）能适当提高程序的执行效率。
* 2）能适当提高资源利用率（CPU、内存利用率）

缺点：
* 1）开启线程需要占用一定的内存空间（默认情况下，主线程占用1M，子线程占用512KB），如果开启大量的线程，会占用大量的内存空间，降低程序的性能。
* 2）线程越多，CPU在调度线程上的开销就越大。
* 3）程序设计更加复杂：比如线程之间的通信、多线程的数据共享

#### 多线程在 iOS 开发中的应用

*iOS 的主线程？*

一个 iOS 程序运行后，默认会开启1条线程，称为“主线程”或“UI线程”，用于刷新显示UI,处理UI事件。
 
 > 不要将耗时操作放到主线程中去处理，会卡住线程。

*iOS 的多线程？*

1）第一种：pthread

特点：
* 一套通用的多线程API
* 适用于Unix\Linux\Windows等系统
* 跨平台\可移植
* 使用难度大

使用语言：c语言

使用频率：几乎不用

线程生命周期：由程序员进行管理



2）第二种：NSThread

特点：
* 使用更加面向对象
* 简单易用，可直接操作线程对象

使用语言：OC语言

使用频率：偶尔使用

线程生命周期：由程序员进行管理



3）第三种：GCD

特点：
* 旨在替代NSThread等线程技术
* 充分利用设备的多核(自动)

使用语言：OC语言

使用频率：经常使用

线程生命周期：自动管理



4）第四种：NSOperation

特点：
* 基于GCD（底层是GCD）
* 比GCD多了一些更简单实用的功能
* 使用更加面向对象

对比 GCD 的优点：
* 在NSOperationQueue中，可以建立各个NSOperation之间的依赖关系
* 有KVO，可以监测operation是否正在执行（isExecuted）、是否结束（isFinished），是否取消（isCanceld）
* NSOperationQueue可以方便的管理并发、NSOperation之间的优先级

使用语言：OC语言

使用频率：经常使用

线程生命周期：自动管理


### 代码示例

由于要使用一些 c 来演示，先说明些规范，防懵逼 😳
* 1.在c语言中，没有对象的概念，对象类型是以-t/Ref结尾的;
* 2.c语言中的void * 和OC的id是等价的;
* 3.在混合开发时，如果在 C 和 OC 之间传递数据，需要使用 __bridge 进行桥接，桥接的目的就是为了告诉编译器如何管理内存，MRC 中不需要使用桥接;
* 4.在 OC 中，如果是 ARC 开发，编译器会在编译时，根据代码结构， 自动添加 retain/release/autorelease。但是，ARC 只负责管理 OC 部分的内存管理，而不负责 C 语言 代码的内存管理。因此，开发过程中，如果使用的 C 语言框架出现retain/create/copy/new 等字样的函数，大多都需要 release，否则会出现内存泄漏。


#### pthread 基本使用

需要引入 pthread 头文件
```objc
#import <pthread.h>
```
使用 pthread_create 方法
```objc
/**
pthread_create(<#pthread_t  _Nullable *restrict _Nonnull#>, <#const pthread_attr_t *restrict _Nullable#>, <#void * _Nullable (* _Nonnull)(void * _Nullable)#>, <#void *restrict _Nullable#>)
参数：
1.指向线程标识符的指针，C 语言中类型的结尾通常 _t/Ref，而且不需要使用 *;
2.用来设置线程属性;
3.指向函数的指针,传入函数名(函数的地址)，线程要执行的函数/任务;
4.运行函数的参数;
*/
```
完整代码
```objc
{
    //1.创建线程对象
    pthread_t thread;

    //2.创建线程
    NSString *param = @"参数";
    int result = pthread_create(&thread, NULL, task, (__bridge void *)(param));
    result == 0?NSLog(@"success"):NSLog(@"failure");

    //3.设置子线程的状态设置为detached,则该线程运行结束后会自动释放所有资源，或者在子线程中添加 pthread_detach(pthread_self()),其中pthread_self()是获得线程自身的id
    pthread_detach(thread);
}
void *task(void * param) {
    //在此做耗时操作
    NSLog(@"new thread : %@  参数是: %@",[NSThread currentThread],(__bridge NSString *)(param));
    return NULL;
}
```

#### NSThread 基本使用

1.1）第一种 创建线程的方式：alloc init.

特点：需要手动开启线程，可以拿到线程对象进行详细设置

* 第一个参数：目标对象
* 第二个参数：选择器，线程启动要调用哪个方法
* 第三个参数：前面方法要接收的参数（最多只能接收一个参数，没有则传nil）

```objc
NSThread *thread = [[NSThread alloc]initWithTarget:self selector:@selector(run) object:@"wendingding"];
//启动线程
[thread start];
```

1.2）第二种 创建线程的方式：分离出一条子线程

特点：自动启动线程，无法对线程进行更详细的设置

* 第一个参数：线程启动调用的方法
* 第二个参数：目标对象
* 第三个参数：传递给调用方法的参数

```objc
[NSThread detachNewThreadSelector:@selector(run) toTarget:self withObject:@"我是分离出来的子线程"];
```

1.3）第三种 创建线程的方式：后台线程

特点：自动启动县城，无法进行更详细设置

```objc
[self performSelectorInBackground:@selector(run) withObject:@"我是后台线程"];
```

2）常用的控制线程方法

为线程命名，以便区分
```objc
//设置线程的名称
thread.name = @"线程A";
```

设置优先级，取值范围为0.0~1.0之间，1.0表示线程的优先级最高,如果不设置该值，那么理想状态下默认为0.5
```objc
//设置线程的优先级
thread.threadPriority = 1.0;
```

退出当前线程
```objc
//退出当前线程
[NSThread exit];
```

线程的各种状态：新建-就绪-运行-阻塞-死亡，阻塞线程方法来了
```objc
[NSThread sleepForTimeInterval:2.0]; //阻塞线程
[NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:2.0]]; //阻塞线程
//注意：线程死了不能复生
```

3）线程安全
* 前提：多个线程访问同一块资源会发生数据安全问题
* 解决方案：加互斥锁
* 相关代码：@synchronized(self){}
* 专业术语-线程同步
* 原子和非原子属性（是否对setter方法加锁）

死锁：是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。

产生死锁的四个必要条件：
* 互斥条件：一个资源每次只能被一个进程使用。
* 占有且等待：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
* 不可强行占有:进程已获得的资源，在末使用完之前，不能强行剥夺。
* 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

产生死锁的原因主要是：
* 因为系统资源不足。
* 进程运行推进的顺序不合适。
* 资源分配不当等。

售票员售票问题
```objc
@interface NSThreadSafeViewController ()
//三只售票员🐱🐶🐭
@property (nonatomic, strong) NSThread *thread01;
@property (nonatomic, strong) NSThread *thread02;
@property (nonatomic, strong) NSThread *thread03;
//总票数
@property(nonatomic, assign) NSInteger totalticket;
@end

@implementation NSThreadSafeViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    //假设有10张票
    self.totalticket = 10;

    //创建线程
    self.thread01 =  [[NSThread alloc]initWithTarget:self selector:@selector(saleTicket) object:nil];
    self.thread01.name = @"小🐱";

    self.thread02 =  [[NSThread alloc]initWithTarget:self selector:@selector(saleTicket) object:nil];
    self.thread02.name = @"小🐶";

    self.thread03 =  [[NSThread alloc]initWithTarget:self selector:@selector(saleTicket) object:nil];
    self.thread03.name = @"小🐭";
}
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    //启动线程
    [self.thread01 start];
    [self.thread02 start];
    [self.thread03 start];
}
//售票
- (void)saleTicket {
    while (1) {
        //2.加互斥锁
        @synchronized(self) {
            [NSThread sleepForTimeInterval:0.03];
            //1.先查看余票数量
            NSInteger count = self.totalticket;
            if (count > 0) {
                self.totalticket = count - 1;
                NSLog(@"%@卖出去了一张票,还剩下%zd张票",[NSThread currentThread].name,self.totalticket);
            } else {
                NSLog(@"%@发现当前票已经买完了--",[NSThread currentThread].name);
                break;
            }
        }
    }
}
```

运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef141663cfa2?w=658&h=239&f=png&s=164477)

4）NSThread 线程间通信（子线程加载图片完成通知主线程更新UI）
```objc
- (void)touchesBegan:(nonnull NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event {
    //开启一条子线程来下载图片
    [NSThread detachNewThreadSelector:@selector(downloadImage) toTarget:self withObject:nil];
}
- (void)downloadImage {
    //1.确定要下载网络图片的url地址，一个url唯一对应着网络上的一个资源
    NSURL *url = [NSURL URLWithString:@"http://p6.qhimg.com/t01d2954e2799c461ab.jpg"];

    //2.根据url地址下载图片数据到本地（二进制数据
    NSData *data = [NSData dataWithContentsOfURL:url];

    //3.把下载到本地的二进制数据转换成图片
    UIImage *image = [UIImage imageWithData:data];

    //4.回到主线程刷新UI
    //4.1 第一种方式
    //    [self performSelectorOnMainThread:@selector(showImage:) withObject:image waitUntilDone:YES];

    //4.2 第二种方式
    //    [self.imageView performSelectorOnMainThread:@selector(setImage:) withObject:image waitUntilDone:YES];

    //4.3 第三种方式
    [self.imageView performSelector:@selector(setImage:) onThread:[NSThread mainThread] withObject:image waitUntilDone:YES];
}
```

5）特殊用法（计算代码段的执行时间）
```objc
NSURL *url = [NSURL URLWithString:@"http://p6.qhimg.com/t01d2954e2799c461ab.jpg"];

//第一种方法
NSDate *start = [NSDate date];
//2.根据url地址下载图片数据到本地（二进制数据）
NSData *data = [NSData dataWithContentsOfURL:url];

NSDate *end = [NSDate date];
NSLog(@"第二步操作花费的时间为%f",[end timeIntervalSinceDate:start]);


//第二种方法
CFTimeInterval start = CFAbsoluteTimeGetCurrent();
NSData *data = [NSData dataWithContentsOfURL:url];

CFTimeInterval end = CFAbsoluteTimeGetCurrent();
NSLog(@"第二步操作花费的时间为%f",end - start);
```

#### GCD 基本使用

1）GCD 核心概念
* 队列和任务
* 同步函数和异步函数

2）GCD 组合拳 🤣

异步函数：
* 异步函数+并发队列：开启多条线程，并发执行任务
* 异步函数+串行队列：开启一条线程，串行执行任务

同步函数：
* 同步函数+并发队列：不开线程，串行执行任务
* 同步函数+串行队列：不开线程，串行执行任务

主队列：
* 异步函数+主队列：不开线程，在主线程中串行执行任务
* 同步函数+主队列：不开线程，串行执行任务（注意死锁发生）

3）GCD 线程间通信
```objc
//0.获取一个全局的队列
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
//1.先开启一个线程，把下载图片的操作放在子线程中处理
dispatch_async(queue, ^{
    //2.下载图片
    NSURL *url = [NSURL URLWithString:@"http://h.hiphotos.baidu.com/zhidao/pic/item/6a63f6246b600c3320b14bb3184c510fd8f9a185.jpg"];
    NSData *data = [NSData dataWithContentsOfURL:url];
    UIImage *image = [UIImage imageWithData:data];

    NSLog(@"下载操作所在的线程--%@",[NSThread currentThread]);

    //3.回到主线程刷新UI
    dispatch_async(dispatch_get_main_queue(), ^{
        self.imageView.image = image;
        //打印查看当前线程
        NSLog(@"刷新UI---%@",[NSThread currentThread]);
    });
});
```

4）GCD 常用小功能

栅栏函数（控制任务的执行顺序）
```objc
dispatch_barrier_async(queue, ^{
    NSLog(@"--dispatch_barrier_async-");
});
```

延迟执行（延迟·控制在哪个线程执行）
```objc
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_global_  queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"---%@",[NSThread currentThread]);
});
```

一次性代码（注意不能放到懒加载）
```objc
- (void)once {
    //整个程序运行过程中只会执行一次
    //onceToken用来记录该部分的代码是否被执行过
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        NSLog(@"-----");
    });
}
```

快速迭代（开多个线程并发完成迭代操作）
```objc
dispatch_apply(subpaths.count, queue, ^(size_t index) {
});
```

队列组（同栅栏函数）
```objc
//创建队列组
dispatch_group_t group = dispatch_group_create();
dispatch_queue_t globalQueue = dispatch_get_global_queue(0, 0);

dispatch_group_enter(group);
//模拟多线程耗时操作
dispatch_group_async(group, globalQueue, ^{
    sleep(3);
    NSLog(@"%@---block1结束。。。",[NSThread currentThread]);
    dispatch_group_leave(group);
});
NSLog(@"%@---1结束。。。",[NSThread currentThread]);

dispatch_group_enter(group);
//模拟多线程耗时操作
dispatch_group_async(group, globalQueue, ^{
    sleep(3);
    NSLog(@"%@---block2结束。。。",[NSThread currentThread]);
    dispatch_group_leave(group);
});
NSLog(@"%@---2结束。。。",[NSThread currentThread]);

dispatch_group_notify(group, globalQueue, ^{
    NSLog(@"%@---全部结束。。。",[NSThread currentThread]);
});
```

#### NSOperation 基本使用

1）NSOperation、NSOperationQueue 简介

NSOperation、NSOperationQueue 是苹果提供给我们的一套多线程解决方案。实际上 NSOperation、NSOperationQueue 是基于 GCD 更高一层的封装，完全面向对象。但是比 GCD 更简单易用、代码可读性也更高。

*为什么要使用 NSOperation、NSOperationQueue？*

* 1.可添加完成的代码块，在操作完成后执行。
* 2.添加操作之间的依赖关系，方便的控制执行顺序。
* 3.设定操作执行的优先级。
* 4.可以很方便的取消一个操作的执行。
* 5.使用 KVO 观察对操作执行状态的更改：isExecuteing、isFinished、isCancelled。


2）NSOperation、NSOperationQueue 操作和操作队列

既然是基于 GCD 的更高一层的封装。那么，GCD 中的一些概念同样适用于 NSOperation、NSOperationQueue。在 NSOperation、NSOperationQueue 中也有类似的任务（操作）和队列（操作队列）的概念。

*操作（Operation）：*
* 执行操作的意思，换句话说就是你在线程中执行的那段代码。
* 在 GCD 中是放在 block 中的。在 NSOperation 中，我们使用 NSOperation 子类 NSInvocationOperation、NSBlockOperation，或者自定义子类来封装操作。

*操作队列（Operation Queues）：*
* 这里的队列指操作队列，即用来存放操作的队列。不同于 GCD 中的调度队列 FIFO（先进先出）的原则。NSOperationQueue 对于添加到队列中的操作，首先进入准备就绪的状态（就绪状态取决于操作之间的依赖关系），然后进入就绪状态的操作的开始执行顺序（非结束执行顺序）由操作之间相对的优先级决定（优先级是操作对象自身的属性）。
* 操作队列通过设置最大并发操作数（maxConcurrentOperationCount）来控制并发、串行。
* NSOperationQueue 为我们提供了两种不同类型的队列：主队列和自定义队列。主队列运行在主线程之上，而自定义队列在后台执行。

3）NSOperation、NSOperationQueue 使用步骤

NSOperation 需要配合 NSOperationQueue 来实现多线程。因为默认情况下，NSOperation 单独使用时系统同步执行操作，配合 NSOperationQueue 我们能更好的实现异步执行。

NSOperation 实现多线程的使用步骤分为三步：

* 1.创建操作：先将需要执行的操作封装到一个 NSOperation 对象中。
* 2.创建队列：创建 NSOperationQueue 对象。
* 3.将操作加入到队列中：将 NSOperation 对象添加到 NSOperationQueue 对象中。

之后呢，系统就会自动将 NSOperationQueue 中的 NSOperation 取出来，在新线程中执行操作。

下面我们来学习下 NSOperation 和 NSOperationQueue 的基本使用。

4）NSOperation 和 NSOperationQueue 基本使用

4.1）创建操作

NSOperation 是个抽象类，不能用来封装操作。我们只有使用它的子类来封装操作。我们有三种方式来封装操作。

* 使用子类 NSInvocationOperation
* 使用子类 NSBlockOperation
* 自定义继承自 NSOperation 的子类，通过实现内部相应的方法来封装操作。

在不使用 NSOperationQueue，单独使用 NSOperation 的情况下系统同步执行操作，下面我们学习以下操作的三种创建方式。

4.1.1）使用子类 NSInvocationOperation

```objc
/**
* 使用子类 NSInvocationOperation
*/
- (void)useInvocationOperation {
    // 1.创建 NSInvocationOperation 对象
    NSInvocationOperation *op = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task1) object:nil];

    // 2.调用 start 方法开始执行操作
    [op start];
}

/**
* 任务1
*/
- (void)task1 {
    for (int i = 0; i < 2; i++) {
        [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
        NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
    }
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef141b2eb525?w=759&h=55&f=png&s=31082)

> 可以看到：在没有使用 NSOperationQueue、在主线程中单独使用使用子类 NSInvocationOperation 执行一个操作的情况下，操作是在当前线程执行的，并没有开启新线程。

如果在其他线程中执行操作，则打印结果为其他线程

```objc
// 在其他线程使用子类 NSInvocationOperation
[NSThread detachNewThreadSelector:@selector(useInvocationOperation) toTarget:self withObject:nil];
```

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef141c69bbf1?w=700&h=58&f=png&s=37536)

> 可以看到：在其他线程中单独使用子类 NSInvocationOperation，操作是在当前调用的其他线程执行的，并没有开启新线程。

4.1.2）使用子类 NSBlockOperation
```objc
/**
* 使用子类 NSBlockOperation
*/
- (void)useBlockOperation {
    // 1.创建 NSBlockOperation 对象
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    // 2.调用 start 方法开始执行操作
    [op start];
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef141b47445e?w=747&h=54&f=png&s=30954)

> 注意：和上边 NSInvocationOperation 使用一样。因为代码是在主线程中调用的，所以打印结果为主线程。如果在其他线程中执行操作，则打印结果为其他线程。

但是，NSBlockOperation 还提供了一个方法 addExecutionBlock:，通过 addExecutionBlock: 就可以为 NSBlockOperation 添加额外的操作。这些操作（包括 blockOperationWithBlock 中的操作）可以在不同的线程中同时（并发）执行。只有当所有相关的操作已经完成执行时，才视为完成。

如果添加的操作多的话，blockOperationWithBlock: 中的操作也可能会在其他线程（非当前线程）中执行，这是由系统决定的，并不是说添加到 blockOperationWithBlock: 中的操作一定会在当前线程中执行。（可以使用 addExecutionBlock: 多添加几个操作试试）。

```objc
/**
 * 使用子类 NSBlockOperation
 * 调用方法 AddExecutionBlock:
*/
- (void)useBlockOperationAddExecutionBlock {
    // 1.创建 NSBlockOperation 对象
    NSBlockOperation *op = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];

    // 2.添加额外的操作
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"4---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"5---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"6---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"7---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"8---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];

    // 3.调用 start 方法开始执行操作
    [op start];
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef141b10bb3e?w=750&h=132&f=png&s=70358)

> 可以看出：使用子类 NSBlockOperation，并调用方法 AddExecutionBlock: 的情况下，blockOperationWithBlock:方法中的操作 和 addExecutionBlock: 中的操作是在不同的线程中异步执行的。而且，这次执行结果中 blockOperationWithBlock:方法中的操作也不是在当前线程（主线程）中执行的。从而印证了blockOperationWithBlock: 中的操作也可能会在其他线程（非当前线程）中执行。

一般情况下，如果一个 NSBlockOperation 对象封装了多个操作。NSBlockOperation 是否开启新线程，取决于操作的个数。如果添加的操作的个数多，就会自动开启新线程。当然开启的线程数是由系统来决定的。

4.1.3）使用自定义继承自 NSOperation 的子类

如果使用子类 NSInvocationOperation、NSBlockOperation 不能满足日常需求，我们可以使用自定义继承自 NSOperation 的子类。可以通过重写 main 或者 start 方法 来定义自己的 NSOperation 对象。重写main方法比较简单，我们不需要管理操作的状态属性 isExecuting 和 isFinished。当 main 执行完返回的时候，这个操作就结束了。

先定义一个继承自 NSOperation 的子类，重写main方法。

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef141c59d0b8?w=212&h=39&f=png&s=10741)

CustomOperation.h
```objc
#import <Foundation/Foundation.h>
@interface CustomOperation : NSOperation
@end
```

CustomOperation.m
```objc
#import "CustomOperation.h"
@implementation CustomOperation
- (void)main {
    if (!self.isCancelled) {
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2];
            NSLog(@"1---%@", [NSThread currentThread]);
        }
    }
}
@end
```

使用示例
```objc
/**
* 使用自定义继承自 NSOperation 的子类
*/
- (void)useCustomOperation {
    // 1.创建 YSCOperation 对象
    CustomOperation *op = [[CustomOperation alloc] init];
    // 2.调用 start 方法开始执行操作
    [op start];
}
```

> 可以看出：在没有使用 NSOperationQueue、在主线程单独使用自定义继承自 NSOperation 的子类的情况下，是在主线程执行操作，并没有开启新线程。

4.2）创建队列

NSOperationQueue 一共有两种队列：主队列、自定义队列。其中自定义队列同时包含了串行、并发功能。下边是主队列、自定义队列的基本创建方法和特点。

* 主队列

凡是添加到主队列中的操作，都会放到主线程中执行。
```objc
// 主队列获取方法
NSOperationQueue *queue = [NSOperationQueue mainQueue];
```

* 自定义队列（非主队列）

添加到这种队列中的操作，就会自动放到子线程中执行（同时包含了：串行、并发功能）
```objc
// 自定义队列创建方法
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
```

4.3）将操作加入到队列中

上边我们说到 NSOperation 需要配合 NSOperationQueue 来实现多线程。

那么我们需要将创建好的操作加入到队列中去。总共有两种方法：

* 1.-(void)addOperation:(NSOperation *)op; 需要先创建操作，再将创建好的操作加入到创建好的队列中去。

```objc
/**
* 使用 addOperation: 将操作加入到操作队列中
*/
- (void)addOperationToQueue {
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];

    // 2.创建操作
    // 使用 NSInvocationOperation 创建操作1
    NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task1) object:nil];

    // 使用 NSInvocationOperation 创建操作2
    NSInvocationOperation *op2 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(task2) object:nil];

    // 使用 NSBlockOperation 创建操作3
    NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [op3 addExecutionBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"4---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];

    // 3.使用 addOperation: 添加所有操作到队列中
    [queue addOperation:op1]; // [op1 start]
    [queue addOperation:op2]; // [op2 start]
    [queue addOperation:op3]; // [op3 start]
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef1442982f62?w=756&h=109&f=png&s=57893)

> 可以看出：使用 NSOperation 子类创建操作，并使用 addOperation: 将操作加入到操作队列后能够开启新线程，进行并发执行。

* 2.-(void)addOperationWithBlock:(void (^)(void))block; 无需先创建操作，在 block 中添加操作，直接将包含操作的 block 加入到队列中

```objc
/**
* 使用 addOperationWithBlock: 将操作加入到操作队列中
*/
- (void)addOperationWithBlockToQueue {
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];

    // 2.使用 addOperationWithBlock: 添加操作到队列中
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
}
```

运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef14428ecf18?w=754&h=132&f=png&s=71804)

> 可以看出：使用 addOperationWithBlock: 将操作加入到操作队列后能够开启新线程，进行并发执行。

5）NSOperationQueue 控制串行执行、并发执行

之前我们说过，NSOperationQueue 创建的自定义队列同时具有串行、并发功能，上边我们演示了并发功能，那么他的串行功能是如何实现的？

这里有个关键属性 maxConcurrentOperationCount，叫做最大并发操作数。用来控制一个特定队列中可以有多少个操作同时参与并发执行。

> 注意：这里 maxConcurrentOperationCount 控制的不是并发线程的数量，而是一个队列中同时能并发执行的最大操作数。而且一个操作也并非只能在一个线程中运行。

maxConcurrentOperationCount 最大并发操作数：
* maxConcurrentOperationCount 默认情况下为-1，表示不进行限制，可进行并发执行。
* maxConcurrentOperationCount 为1时，队列为串行队列。只能串行执行。
* maxConcurrentOperationCount 大于1时，队列为并发队列。操作并发执行，当然这个值不应超过系统限制，即使自己设置一个很大的值，系统也会自动调整为 min{自己设定的值，系统设定的默认最大值}。

```objc
/**
* 设置 MaxConcurrentOperationCount（最大并发操作数）
*/
- (void)setMaxConcurrentOperationCount {
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];

    // 2.设置最大并发操作数
    queue.maxConcurrentOperationCount = 1; // 串行队列
    // queue.maxConcurrentOperationCount = 2; // 并发队列
    // queue.maxConcurrentOperationCount = 8; // 并发队列

    // 3.添加操作
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"3---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    [queue addOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"4---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
}
```
maxConcurrentOperationCount 设置为 1，运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef1442851adc?w=753&h=211&f=png&s=110267)

maxConcurrentOperationCount 设置为 2，运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef144468a6f0?w=756&h=212&f=png&s=109339)

> 可以看出：当最大并发操作数为1时，操作是按顺序串行执行的，并且一个操作完成之后，下一个操作才开始执行。当最大操作并发数为2时，操作是并发执行的，可以同时执行两个操作。而开启线程数量是由系统决定的，不需要我们来管理。


6）NSOperation 操作依赖

NSOperation、NSOperationQueue 最吸引人的地方是它能添加操作之间的依赖关系。通过操作依赖，我们可以很方便的控制操作之间的执行先后顺序。NSOperation 提供了3个接口供我们管理和查看依赖。

* -(void)addDependency:(NSOperation *)op; 添加依赖，使当前操作依赖于操作 op 的完成。
* -(void)removeDependency:(NSOperation *)op; 移除依赖，取消当前操作对操作 op 的依赖。
* @property (readonly, copy) NSArray<NSOperation *> *dependencies; 在当前操作开始执行之前完成执行的所有操作对象数组。

当然，我们经常用到的还是添加依赖操作。现在考虑这样的需求，比如说有 A、B 两个操作，其中 A 执行完操作，B 才能执行操作。

如果使用依赖来处理的话，那么就需要让操作 B 依赖于操作 A。具体代码如下：

```objc
/**
* 操作依赖
* 使用方法：addDependency:
*/
- (void)addDependency {
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];

    // 2.创建操作
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
        }
    }];

    // 3.添加依赖
    [op2 addDependency:op1]; // 让op2 依赖于 op1，则先执行op1，在执行op2

    // 4.添加操作到队列中
    [queue addOperation:op1];
    [queue addOperation:op2];
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef1444b73cbd?w=759&h=107&f=png&s=56815)

> 可以看到：通过添加操作依赖，无论运行几次，其结果都是 op1 先执行，op2 后执行。


7）NSOperation 优先级

NSOperation 提供了queuePriority（优先级）属性，queuePriority属性适用于同一操作队列中的操作，不适用于不同操作队列中的操作。默认情况下，所有新创建的操作对象优先级都是NSOperationQueuePriorityNormal。但是我们可以通过setQueuePriority:方法来改变当前操作在同一队列中的执行优先级。

```objc
// 优先级的取值
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
NSOperationQueuePriorityVeryLow = -8L,
NSOperationQueuePriorityLow = -4L,
NSOperationQueuePriorityNormal = 0,
NSOperationQueuePriorityHigh = 4,
NSOperationQueuePriorityVeryHigh = 8
};
```

上边我们说过：对于添加到队列中的操作，首先进入准备就绪的状态（就绪状态取决于操作之间的依赖关系），然后进入就绪状态的操作的开始执行顺序（非结束执行顺序）由操作之间相对的优先级决定（优先级是操作对象自身的属性）。

那么，什么样的操作才是进入就绪状态的操作呢？

当一个操作的所有依赖都已经完成时，操作对象通常会进入准备就绪状态，等待执行。
举个例子，现在有4个优先级都是 NSOperationQueuePriorityNormal（默认级别）的操作：op1，op2，op3，op4。其中 op3 依赖于 op2，op2 依赖于 op1，即 op3 -> op2 -> op1。现在将这4个操作添加到队列中并发执行。

因为 op1 和 op4 都没有需要依赖的操作，所以在 op1，op4 执行之前，就是处于准备就绪状态的操作。
而 op3 和 op2 都有依赖的操作（op3 依赖于 op2，op2 依赖于 op1），所以 op3 和 op2 都不是准备就绪状态下的操作。
理解了进入就绪状态的操作，那么我们就理解了queuePriority 属性的作用对象。

queuePriority 属性决定了进入准备就绪状态下的操作之间的开始执行顺序。并且，优先级不能取代依赖关系。
如果一个队列中既包含高优先级操作，又包含低优先级操作，并且两个操作都已经准备就绪，那么队列先执行高优先级操作。比如上例中，如果 op1 和 op4 是不同优先级的操作，那么就会先执行优先级高的操作。

如果，一个队列中既包含了准备就绪状态的操作，又包含了未准备就绪的操作，未准备就绪的操作优先级比准备就绪的操作优先级高。那么，虽然准备就绪的操作优先级低，也会优先执行。优先级不能取代依赖关系。如果要控制操作间的启动顺序，则必须使用依赖关系。


8）NSOperation、NSOperationQueue 线程间的通信

在 iOS 开发过程中，我们一般在主线程里边进行 UI 刷新，例如：点击、滚动、拖拽等事件。我们通常把一些耗时的操作放在其他线程，比如说图片下载、文件上传等耗时操作。而当我们有时候在其他线程完成了耗时操作时，需要回到主线程，那么就用到了线程之间的通讯。

```objc
/**
* 线程间通信
*/
- (void)communication {
    // 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];

    // 2.添加操作
    [queue addOperationWithBlock:^{
        // 异步进行耗时操作
        for (int i = 0; i < 2; i++) {
            [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
            NSLog(@"1---%@", [NSThread currentThread]); // 打印当前线程
        }
        
        // 回到主线程
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            // 进行一些 UI 刷新等操作
            for (int i = 0; i < 2; i++) {
                [NSThread sleepForTimeInterval:2]; // 模拟耗时操作
                NSLog(@"2---%@", [NSThread currentThread]); // 打印当前线程
            }
        }];
    }];
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef14615756a6?w=758&h=112&f=png&s=56793)

9）NSOperation、NSOperationQueue 线程同步和线程安全

线程安全：如果你的代码所在的进程中有多个线程在同时运行，而这些线程可能会同时运行这段代码。如果每次运行结果和单线程运行的结果是一样的，而且其他的变量的值也和预期的是一样的，就是线程安全的。

若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作（更改变量），一般都需要考虑线程同步，否则的话就可能影响线程安全。

线程同步：可理解为线程 A 和 线程 B 一块配合，A 执行到一定程度时要依靠线程 B 的某个结果，于是停下来，示意 B 运行；B 依言执行，再将结果给 A；A 再继续操作。

举个简单例子就是：两个人在一起聊天。两个人不能同时说话，避免听不清(操作冲突)。等一个人说完(一个线程结束操作)，另一个再说(另一个线程再开始操作)。

下面，我们模拟火车票售卖的方式，实现 NSOperation 线程安全和解决线程同步问题。
> 场景：总共有50张火车票，有两个售卖火车票的窗口，一个是北京火车票售卖窗口，另一个是上海火车票售卖窗口。两个窗口同时售卖火车票，卖完为止。

9.1）NSOperation、NSOperationQueue 非线程安全

先来看看不考虑线程安全的代码：

```objc
/**
* 非线程安全：不使用 NSLock
* 初始化火车票数量、卖票窗口(非线程安全)、并开始卖票
*/
- (void)initTicketStatusNotSave {
    NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
    self.ticketSurplusCount = 50;

    // 1.创建 queue1,queue1 代表北京火车票售卖窗口
    NSOperationQueue *queue1 = [[NSOperationQueue alloc] init];
    queue1.maxConcurrentOperationCount = 1;

    // 2.创建 queue2,queue2 代表上海火车票售卖窗口
    NSOperationQueue *queue2 = [[NSOperationQueue alloc] init];
    queue2.maxConcurrentOperationCount = 1;

    // 3.创建卖票操作 op1
    __weak typeof(self) weakSelf = self;
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        [weakSelf saleTicketNotSafe];
    }];

    // 4.创建卖票操作 op2
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        [weakSelf saleTicketNotSafe];
    }];

    // 5.添加操作，开始卖票
    [queue1 addOperation:op1];
    [queue2 addOperation:op2];
}

/**
* 售卖火车票(非线程安全)
*/
- (void)saleTicketNotSafe {
    while (1) {
        if (self.ticketSurplusCount > 0) {
            //如果还有票，继续售卖
            self.ticketSurplusCount--;
            NSLog(@"%@", [NSString stringWithFormat:@"剩余票数:%d 窗口:%@", self.ticketSurplusCount, [NSThread currentThread]]);
            [NSThread sleepForTimeInterval:0.2];
        } else {
            NSLog(@"所有火车票均已售完");
            break;
        }
    }
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef1464986694?w=665&h=118&f=png&s=75328)

> 可以看到：在不考虑线程安全，不使用 NSLock 情况下，在0票后依然出票，数据混乱，这样显然不符合我们的需求，所以我们需要考虑线程安全问题。

9.2）NSOperation、NSOperationQueue 非线程安全

线程安全解决方案：可以给线程加锁，在一个线程执行该操作的时候，不允许其他线程进行操作。iOS 实现线程加锁有很多种方式。@synchronized、 NSLock、NSRecursiveLock、NSCondition、NSConditionLock、pthread_mutex、dispatch_semaphore、OSSpinLock、atomic(property) set/ge等等各种方式。这里我们使用 NSLock 对象来解决线程同步问题。

NSLock 对象可以通过进入锁时调用 lock 方法，解锁时调用 unlock 方法来保证线程安全。

```objc
/**
* 线程安全：使用 NSLock 加锁
* 初始化火车票数量、卖票窗口(线程安全)、并开始卖票
*/
- (void)initTicketStatusSave {
    NSLog(@"currentThread---%@",[NSThread currentThread]); // 打印当前线程
    self.ticketSurplusCount = 50;

    self.lock = [[NSLock alloc] init];  // 初始化 NSLock 对象

    // 1.创建 queue1,queue1 代表北京火车票售卖窗口
    NSOperationQueue *queue1 = [[NSOperationQueue alloc] init];
    queue1.maxConcurrentOperationCount = 1;

    // 2.创建 queue2,queue2 代表上海火车票售卖窗口
    NSOperationQueue *queue2 = [[NSOperationQueue alloc] init];
    queue2.maxConcurrentOperationCount = 1;

    // 3.创建卖票操作 op1
    __weak typeof(self) weakSelf = self;
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        [weakSelf saleTicketSafe];
    }];

    // 4.创建卖票操作 op2
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        [weakSelf saleTicketSafe];
    }];

    // 5.添加操作，开始卖票
    [queue1 addOperation:op1];
    [queue2 addOperation:op2];
}

/**
* 售卖火车票(线程安全)
*/
- (void)saleTicketSafe {
    while (1) {
        // 加锁
        [self.lock lock];

        if (self.ticketSurplusCount > 0) {
            //如果还有票，继续售卖
            self.ticketSurplusCount--;
            NSLog(@"%@", [NSString stringWithFormat:@"剩余票数:%d 窗口:%@", self.ticketSurplusCount, [NSThread currentThread]]);
            [NSThread sleepForTimeInterval:0.2];
        }

        // 解锁
        [self.lock unlock];

        if (self.ticketSurplusCount <= 0) {
            NSLog(@"所有火车票均已售完");
            break;
        }
    }
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/5/163cef04224f6476?w=684&h=181&f=png&s=105710)

> 可以看出：在考虑了线程安全，使用 NSLock 加锁、解锁机制的情况下，得到的票数是正确的，没有出现混乱的情况。我们也就解决了多个线程同步的问题。


10）NSOperation、NSOperationQueue 常用属性和方法归纳

10.1）NSOperation 常用属性和方法

取消操作方法
* -(void)cancel; 可取消操作，实质是标记 isCancelled 状态。
判断操作状态方法
* -(BOOL)isFinished; 判断操作是否已经结束。
* -(BOOL)isCancelled; 判断操作是否已经标记为取消。
* -(BOOL)isExecuting; 判断操作是否正在在运行。
* -(BOOL)isReady; 判断操作是否处于准备就绪状态，这个值和操作的依赖关系相关。
操作同步
* -(void)waitUntilFinished; 阻塞当前线程，直到该操作结束。可用于线程执行顺序的同步。
* -(void)setCompletionBlock:(void (^)(void))block; completionBlock 会在当前操作执行完毕时执行 completionBlock。
* -(void)addDependency:(NSOperation *)op; 添加依赖，使当前操作依赖于操作 op 的完成。
* -(void)removeDependency:(NSOperation *)op; 移除依赖，取消当前操作对操作 op 的依赖。
* @property (readonly, copy) NSArray<NSOperation *> *dependencies; 在当前操作开始执行之前完成执行的所有操作对象数组。

10.2 NSOperationQueue 常用属性和方法

取消/暂停/恢复操作
* -(void)cancelAllOperations; 可以取消队列的所有操作。
* -(BOOL)isSuspended; 判断队列是否处于暂停状态。 YES 为暂停状态，NO 为恢复状态。
* -(void)setSuspended:(BOOL)b; 可设置操作的暂停和恢复，YES 代表暂停队列，NO 代表恢复队列。
操作同步
* -(void)waitUntilAllOperationsAreFinished; 阻塞当前线程，直到队列中的操作全部执行完毕。
添加/获取操作`
* -(void)addOperationWithBlock:(void (^)(void))block; 向队列中添加一个 NSBlockOperation 类型操作对象。
* -(void)addOperations:(NSArray *)ops waitUntilFinished:(BOOL)wait; 向队列中添加操作数组，wait 标志是否阻塞当前线程直到所有操作结束
* -(NSArray *)operations; 当前在队列中的操作数组（某个操作执行结束后会自动从这个数组清除）。
* -(NSUInteger)operationCount; 当前队列中的操作数。
获取队列
* +(id)currentQueue; 获取当前队列，如果当前线程不是在 NSOperationQueue 上运行则返回 nil。
* +(id)mainQueue; 获取主队列。

>注意：
这里的暂停和取消（包括操作的取消和队列的取消）并不代表可以将当前的操作立即取消，而是当当前的操作执行完毕之后不再执行新的操作。
暂停和取消的区别就在于：暂停操作之后还可以恢复操作，继续向下执行；而取消操作之后，所有的操作就清空了，无法再接着执行剩下的操作。


> 以上文章整理自：https://blog.csdn.net/sunnyboy9/article/details/19848031、https://www.jianshu.com/p/4b1d77054b35