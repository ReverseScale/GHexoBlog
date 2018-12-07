---
title: iOS Principle：Notification
layout: post
date: 2018-06-12 21:56:27
comments: true
categories:
	- Principle
keywords: Principle
tags:
	- iOS
description: 

---
Cocoa 中使用 NSNotification、NSNotificationCenter 和 KVO 来实现观察者模式，实现对象间一对多的依赖关系~ 

<!-- more -->
------
👨🏻‍💻 [Github Demo](https://github.com/ReverseScale/iOSPrinciple_Notification)

### 方便记忆：
* 设计模式：观察者模式，实现对象间一对多的依赖关系
* NSNotification：方便 NSNotificationCenter 广播到其他对象的封装对象
* NSNotificationCenter：管理通知调度表中对象的发送和接收通知
* 发送时机：
    * NSPostWhenIdle：空闲发送通知 (当运行循环处于等待或空闲状态时，发送通知，对于不重要的通知可以使用)
    * NSPostASAP：尽快发送通知 (当前运行循环迭代完成时，通知将会被发送，有点类似没有延迟的定时器)
    * NSPostNow ：同步发送通知 (如果不使用合并通知和postNotification:一样是同步通知)
* 合并通知（系统的drawRect方法）：
    * NSNotificationNoCoalescing：不合并通知
    * NSNotificationCoalescingOnName：合并相同名称的通知
    * NSNotificationCoalescingOnSender：合并相同通知和同一对象的通知
* 异步通知：NSNotificationQueue实现对通知的控制管理
* 多线程通知：使用runloop维持多个线程使用NSPort进行通知（NSPort的runloop兼容mode）

---

### 前言 

本篇文章主要来讨论NSNotification、NSNotificationCenter、同步异步通知、通知中心底层原理

### NSNotification、NSNotificationCenter 等
#### NSNotification

NSNotification 是方便 NSNotificationCenter 广播到其他对象的封装对象，通知中心(NSNotificationCenter)对通知调度表中的对象广播时发送NSNotification对象

```objc
@interface NSNotification : NSObject <NSCopying, NSCoding>
@property (readonly, copy) NSNotificationName name; //标识通知的标记
@property (nullable, readonly, retain) id object; //要通知的对象
@property (nullable, readonly, copy) NSDictionary *userInfo; //存储发送通知时附带的信息
```

#### NSNotificationCenter

NSNotificationCenter是类似一个广播中心站，使用defaultCenter来获取应用中的通知中心，它可以向应用任何地方发送和接收通知。

在通知中心注册观察者，发送者使用通知中心广播时，以NSNotification的name和object来确定需要发送给哪个观察者。

为保证观察者能接收到通知，所以应先向通知中心注册观察者，接着再发送通知这样才能在通知中心调度表中查找到相应观察者进行通知。

#### 发送者

发送者其实就是对post的使用，后面单独讲，发送通知可使用以下方法发送通知

```objc
- (void)postNotification:(NSNotification *)notification;
- (void)postNotificationName:(NSNotificationName)aName object:(nullable id)anObject;
- (void)postNotificationName:(NSNotificationName)aName object:(nullable id)anObject userInfo:(nullable NSDictionary *)aUserInfo;
```

三种方式实际上都是发送NSNotification对象给通知中心注册的观察者。

发送通知通过name和object来确定来标识观察者,name和object两个参数的规则相同即当通知设置name为kChangeNotifition时，那么只会发送给符合name为kChangeNotifition的观察者

同理object指发送给某个特定对象通知

* 如果只设置了name，那么只有对应名称的通知会触发。
* 如果同时设置name和object参数时就必须同时符合这两个条件的观察者才能接收到通知。

#### 观察者

你可以使用以下两种方式注册观察者

```objc
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(nullable NSNotificationName)aName object:(nullable id)anObject;

- (id <NSObject>)addObserverForName:(nullable NSNotificationName)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block NS_AVAILABLE(10_6, 4_0);
```

* 第一种方式是比较常用的添加Oberver的方式，接到通知时执行aSelector。
* 第二种方式是基于Block来添加观察者，往通知中心的调度表中添加观察者，这个观察者包括一个queue和一个block,并且会返回这个观察者对象。当接到通知时执行block所在的线程为添加观察者时传入的queue参数，queue也可以为nil，那么block就在通知所在的线程同步执行。

> 这里需要注意的是如果使用第二种的方式创建观察者需要弱引用可能引起循环引用的对象,避免内存泄漏。

#### 移除观察者

在对象被释放前需要移除掉观察者，避免已经被释放的对象还接收到通知导致崩溃。

移除观察者有两种方式：

```objc
- (void)removeObserver:(id)observer;
- (void)removeObserver:(id)observer name:(nullable NSNotificationName)aName object:(nullable id)anObject;
```

传入相应的需要移除的observer 或者使用第二种方式三个参数来移除指定某个观察者。

如果使用基于-[NSNotificationCenter addObserverForName:object:queue:usingBlock:]方法在获取方法返回的观察者进行释放。基于这个方法我们还可以让观察者接到通知后只执行一次：

```objc
__block __weak id<NSObject> observer = [[NSNotificationCenter defaultCenter] addObserverForName:kChangeNotifition object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification * _Nonnull note) {
    NSLog(@"-[NSNotificationCenter addObserverForName:object:queue:usingBlock:]");
    [[NSNotificationCenter defaultCenter] removeObserver:observer];
}];
```

![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d50b490c5?w=700&h=44&f=png&s=29155)

> 在iOS9中使用-[NSNotificationCenter addObserverForName:object:queue:usingBlock:]方法需要手动释放

#### NSNotificationQueue

NSNotificationQueue通知队列，用来管理多个通知的调用。通知队列通常以先进先出（FIFO）顺序维护通。

NSNotificationQueue就像一个缓冲池把一个个通知放进池子中，使用特定方式通过NSNotificationCenter发送到相应的观察者。下面我们会提到特定的方式即合并通知和异步通知。

1.创建通知队列方法:

```objc
- (instancetype)initWithNotificationCenter:(NSNotificationCenter *)notificationCenter NS_DESIGNATED_INITIALIZER;
// 或者直接 defaultQueue
NSNotificationQueue * notificationQueue = [NSNotificationQueue defaultQueue];
```

2.往队列加入通知方法:

```objc
- (void)enqueueNotification:(NSNotification *)notification postingStyle:(NSPostingStyle)postingStyle;
- (void)enqueueNotification:(NSNotification *)notification postingStyle:(NSPostingStyle)postingStyle coalesceMask:(NSNotificationCoalescing)coalesceMask forModes:(nullable NSArray<NSRunLoopMode> *)modes;
```

3.移除队列中的通知方法:
```objc
- (void)dequeueNotificationsMatching:(NSNotification *)notification coalesceMask:(NSUInteger)coalesceMask;
```

4.发送方式

控制通知发送的时机

NSPostingStyle包括三种类型：
```objc
typedef NS_ENUM(NSUInteger, NSPostingStyle) {
    NSPostWhenIdle = 1,
    NSPostASAP = 2,
    NSPostNow = 3  
};
```

* NSPostWhenIdle：空闲发送通知 (当运行循环处于等待或空闲状态时，发送通知，对于不重要的通知可以使用)
* NSPostASAP：尽快发送通知 (当前运行循环迭代完成时，通知将会被发送，有点类似没有延迟的定时器)
* NSPostNow ：同步发送通知 (如果不使用合并通知和postNotification:一样是同步通知)

5.合并通知

通过合并我们可以用来保证相同的通知只被发送一次

NSNotificationCoalescing包括三种类型：

```objc
typedef NS_OPTIONS(NSUInteger, NSNotificationCoalescing) {
    NSNotificationNoCoalescing = 0,
    NSNotificationCoalescingOnName = 1,
    NSNotificationCoalescingOnSender = 2
};
```

* NSNotificationNoCoalescing：不合并通知。
* NSNotificationCoalescingOnName：合并相同名称的通知。
* NSNotificationCoalescingOnSender：合并相同通知和同一对象的通知。

> forModes:(nullable NSArray<NSRunLoopMode> *)modes可以使用不同的NSRunLoopMode配合来发送通知，可以看出实际上NSNotificationQueue与RunLoop的机制以及运行循环有关系，通过NSNotificationQueue队列来发送的通知和关联的RunLoop运行机制来进行的。


### 通知管理：同(异)步、单(多)线程
#### 同步通知
先写一个简单的通知示例

```objc
- (void)viewDidLoad {
    [super viewDidLoad];

    //object：指定接受某个对象的通知，为nil表示可以接受任意对象的通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleNotifi:) name:@"EdisonNotif" object:nil];
}
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [self sendNotification];
}
- (void)sendNotification {
    NSLog(@"发送通知before：%@", [NSThread currentThread]);
    [[NSNotificationCenter defaultCenter] postNotificationName:@"EdisonNotif" object:nil];
    NSLog(@"发送通知after：%@", [NSThread currentThread]);
}
- (void)handleNotifi:(NSNotification*)notif {
    NSLog(@"接收到通知了:%@", [NSThread currentThread]);
}
```

![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d558eeba0?w=952&h=52&f=png&s=47684)

从 log 结果看出，通知处理都是同步的

> 发送通知before -> 接收到通知了 -> 发送通知after

说明消息发完之后要等处理了消息才跑发送消息之后的代码，这跟多线程中的同步概念相似

#### 异步通知

发送完之后就继续执行下面的代码，不需要去等待接受通知的处理，这里用到通知对列 NSNotificationQueue

```objc
- (void)sendNotificationQueue {
    //每个线程都默认又一个通知队列，可以直接获取，也可以alloc
    NSNotificationQueue * notificationQueue = [NSNotificationQueue defaultQueue];
    NSNotification * notification = [NSNotification notificationWithName:@"EdisonNotif" object:nil];
    NSLog(@"异步发送通知before:%@",[NSThread currentThread]);
    [notificationQueue enqueueNotification:notification postingStyle:NSPostWhenIdle coalesceMask:NSNotificationCoalescingOnName forModes:nil];
    NSLog(@"异步发送通知after:%@",[NSThread currentThread]);
}
```

![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d55b94760?w=979&h=50&f=png&s=49949)

从 log 结果看出，通知处理实现了异步

> 发送通知before -> 发送通知after -> 接收到通知了 

#### 多线程通知

点击屏幕直接发送通知，开启一个线程发送通知

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    [NSThread detachNewThreadSelector:@selector(sendNotification) toTarget:self withObject:nil];
}
```

而用线程发送同步的是可以接受到通知的，并且处理也是在线程里处理的
，这说通知队列跟线程是有关系的，再继续改代码，回到线程发送异步通知，只是把发送时机改成马上发送

![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d56f8836e?w=979&h=81&f=png&s=77932)

```objc
[notificationQueue enqueueNotification:notification postingStyle:NSPostNow coalesceMask:NSNotificationNoCoalescing forModes:nil];
```
![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d55ce8eb2?w=947&h=49&f=png&s=47572)

这又能处理通知，所以可以说NSPostNow就是同步，其实呢，[[NSNotificationCenter defaultCenter]通知中心这句代码的意思就是：你在哪个线程里面就是获取当前线程的通知队列并且默认采用NSPostNow发送时机

```objc
[notificationQueue enqueueNotification:notification postingStyle:NSPostWhenIdle coalesceMask:NSNotificationNoCoalescing forModes:nil];
```
![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d56f53724?w=946&h=49&f=png&s=48213)

那么通知队列到底和线程有什么关系呢：每个线程都有一个通知队列，当线程结束了，通知队列就被释放了，所以当前选择发送时机为NSPostWhenIdle时也就是空闲的时候发送通知，通知队列就已经释放了，所以通知发送不出去了

如果线程不结束，就可以发送通知了，用runloop让线程不结束

```objc
- (void)sendAsyncNotification {
    //每个线程都默认又一个通知队列，可以直接获取，也可以alloc
    NSNotificationQueue * notificationQueue = [NSNotificationQueue defaultQueue];
    NSNotification * notification = [NSNotification notificationWithName:@"EdisonNotif" object:nil];
    NSLog(@"异步发送通知before:%@",[NSThread currentThread]);
    [notificationQueue enqueueNotification:notification postingStyle:NSPostWhenIdle coalesceMask:NSNotificationNoCoalescing forModes:nil];
    NSLog(@"异步发送通知after:%@",[NSThread currentThread]);
    NSPort * port = [NSPort new];
    [[NSRunLoop currentRunLoop] addPort:port forMode:NSRunLoopCommonModes];
    [[NSRunLoop currentRunLoop] run];
}
```

![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d7c6999d2?w=978&h=59&f=png&s=49945)

这样通知就被发送出去了，而且发送和处理也在线程中，这还没有达到真正的异步是吧，应该发送在一个线程，处理在另一个线程

消息合并处理

```objc
- (void)sendAsyncNotificationTwo {
    NSNotificationQueue * notificationQueue = [NSNotificationQueue defaultQueue];
    NSNotification * notification = [NSNotification notificationWithName:@"EdisonNotif" object:nil];
    NSNotification * notificationtwo = [NSNotification notificationWithName:@"EdisonNotif" object:nil];
    NSLog(@"异步发送通知before:%@",[NSThread currentThread]);
    [notificationQueue enqueueNotification:notification postingStyle:NSPostWhenIdle coalesceMask:NSNotificationCoalescingOnName forModes:nil];
    [notificationQueue enqueueNotification:notificationtwo postingStyle:NSPostWhenIdle coalesceMask:NSNotificationCoalescingOnName forModes:nil];
    [notificationQueue enqueueNotification:notification postingStyle:NSPostWhenIdle coalesceMask:NSNotificationCoalescingOnName forModes:nil];
    NSLog(@"异步发送通知after:%@",[NSThread currentThread]);
    NSPort * port = [NSPort new];
    [[NSRunLoop currentRunLoop] addPort:port forMode:NSRunLoopCommonModes];
    [[NSRunLoop currentRunLoop] run];
}
```

![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d81aaa7e5?w=972&h=50&f=png&s=47729)

设置成NSNotificationCoalescingOnName按名称合并，此时我连续发送三条,但是只处理了一次，
再继续，上面代码就只是把发送时机改成NSPostNow

```objc
[notificationQueue enqueueNotification:notification postingStyle:NSPostNow coalesceMask:NSNotificationCoalescingOnName forModes:nil];
[notificationQueue enqueueNotification:notificationTwo postingStyle:NSPostNow coalesceMask:NSNotificationCoalescingOnName forModes:nil];
[notificationQueue enqueueNotification:notification postingStyle:NSPostNow coalesceMask:NSNotificationCoalescingOnName forModes:nil];
```

打印结果

```
/*
2017-08-30 16:30:04.114 通知的底层解析[4442:164620] 异步发送通知before:<NSThread: 0x600000269a40>{number = 3, name = (null)}
2017-08-30 16:30:04.115 通知的底层解析[4442:164620] 接收到通知了:<NSThread: 0x600000269a40>{number = 3, name = (null)}
2017-08-30 16:30:04.115 通知的底层解析[4442:164620] 接收到通知了:<NSThread: 0x600000269a40>{number = 3, name = (null)}
2017-08-30 16:30:04.115 通知的底层解析[4442:164620] 接收到通知了:<NSThread: 0x600000269a40>{number = 3, name = (null)}
2017-08-30 16:30:04.116 通知的底层解析[4442:164620] 异步发送通知after:<NSThread: 0x600000269a40>{number = 3, name = (null)}
*/
```

结果就打印了处理了三次通知，这个应该好理解吧，就跟dispatch_sync原理一样，就是得发送因为NSPostNow是同步的，所以发送第一条通知，得等处理完第一条通知，才跑发送第二条通知，这样肯定就没有合并消息一说了，因为这有点类似线程阻塞的意思，只有异步，就是三个发送通知全部跑完，在处理通知的时候看是否需要合并和怎么合并，再去处理

> 系统的很多方法，如 drawRect，就是默认消息合并处理，多次方法只响应一次

### 实现原理

#### 先猜想一下
首先，信息的传递就依靠通知(NSNotification),也就是说，通知就是信息(执行的方法，观察者本身(self),参数)的包装。

通知中心(NSNotificationCenter)是个单例，向通知中心注册观察者，也就是说，这个通知中心有个集合，这个集合存放着观察者。

可以想象的是，发送通知需要name参数，添加观察者也有个name参数，这两个name一样的时候，当发送通知时候，观察者对象就能接受到信息，执行对应的操作。那么这个集合很容易想到就是NSDictionary! 

key就是name，value就是NSArray(存放数据模型)，里面存放观察者对象。如下图 

![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d81c74cfe?w=790&h=539&f=png&s=30252)

#### 实现探究

根据NSNotification&NSNotificationCenter接口给出实现代码,创建两个新类YFLNotification,YFLNotificationCenter，这两个类的接口和苹果提供的接口完全一样，我将根据接口提供的功能给出实现代码。 

要点是通知中心是单例类，并且通知中心维护了一个包含所有注册的观察者的集合，这里我选择了动态数组来存储所有的观察者，源码如下：

```objc
+ (YFLNotificationCenter*)defaultCenter {
    static YFLNotificationCenter *singleton;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        singleton = [[self alloc] initSingleton];
    });
    return singleton;
}
- (instancetype)initSingleton {
    if ([super init]) {
        _obsetvers = [[NSMutableDictionary alloc] init];
    }
    return self;
}
```

还定义了一个观察者模型用于保存观察者，通知消息名，观察者收到通知后执行代码所在的操作队列和执行代码的回调，模型源码如下：

```objc
@interface YFLObserverModel: NSObject
@property (nonatomic, strong) id observer;  //观察者对象
@property (nonatomic, assign) SEL selector;  //执行的方法
@property (nonatomic, copy) NSString *notificationName; //通知名字
@property (nonatomic, strong) id object;  //携带参数
@property (nonatomic, strong) NSOperationQueue *operationQueue;//队列
@property (nonatomic, copy) OperationBlock block;  //回调
@end
```

向通知中心注册观察者，源码如下：
```objc
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(nullable NSString*)aName object:(nullable id)anObject {
    //如果不存在，那么即创建
    if (![self.obsetvers objectForKey:aName]) {
        NSMutableArray *arrays = [[NSMutableArray alloc]init];
        // 创建数组模型
        YFLObserverModel *observerModel = [[YFLObserverModel alloc]init];
        observerModel.observer = observer;
        observerModel.selector = aSelector;
        observerModel.notificationName = aName;
        observerModel.object = anObject;
        [arrays addObject:observerModel];
        //填充进入数组
        [self.obsetvers setObject:arrays forKey:aName];
    } else {
        //如果存在，取出来，继续添加减去即可
        NSMutableArray *arrays = (NSMutableArray*)[self.obsetvers objectForKey:aName];
        // 创建数组模型
        YFLObserverModel *observerModel = [[YFLObserverModel alloc]init];
        observerModel.observer = observer;
        observerModel.selector = aSelector;
        observerModel.notificationName = aName;
        observerModel.object = anObject;
        [arrays addObject:observerModel];
    }
}
- (id <NSObject>)addObserverForName:(nullable NSString *)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(YFLNotification *note))block {
    //如果不存在，那么即创建
    if (![self.obsetvers objectForKey:name]) {
        NSMutableArray *arrays = [[NSMutableArray alloc]init];
        // 创建数组模型
        YFLObserverModel *observerModel = [[YFLObserverModel alloc]init];
        observerModel.block = block;
        observerModel.notificationName = name;
        observerModel.object = obj;
        observerModel.operationQueue = queue;
        [arrays addObject:observerModel];
        //填充进入数组
        [self.obsetvers setObject:arrays forKey:name];
    } else {
        //如果存在，取出来，继续添加即可
        NSMutableArray *arrays = (NSMutableArray*)[self.obsetvers objectForKey:name];
        // 创建数组模型
        YFLObserverModel *observerModel = [[YFLObserverModel alloc]init];
        observerModel.block = block;
        observerModel.notificationName = name;
        observerModel.object = obj;
        observerModel.operationQueue = queue;
        [arrays addObject:observerModel];
    }
    return nil;
}
```

发送通知有三种方式，最终都是调用- (void)postNotification:(YFLNotification *)notification，源码如下：

```objc
- (void)postNotification:(YFLNotification *)notification {
    //name 取出来对应观察者数组，执行任务
    NSMutableArray *arrays = (NSMutableArray*)[self.obsetvers objectForKey:notification.name];
    [arrays enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        //取出数据模型
        YFLObserverModel *observerModel = obj;
        id observer = observerModel.observer;
        SEL secector = observerModel.selector;
        if (!observerModel.operationQueue) { 
            #pragma clang diagnostic push
            #pragma clang diagnostic ignored "-Warc-performSelector-leaks"
            [observer performSelector:secector withObject:notification];
            #pragma clang diagnostic pop
        } else {
            //创建任务
            NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
                //这里用block回调出去
                observerModel.block(notification);
            }];
            // 如果添加观察者 传入 队列，那么就任务放在队列中执行(子线程异步执行)
            NSOperationQueue *operationQueue = observerModel.operationQueue;
            [operationQueue addOperation:operation];
        }
    }];
}
```

#### 底层通信 port

通知队列也可以实现异步，但是真正的异步还是得通过port

底层所有的消息触发都是通过端口 NSPort 来进行操作的

NSPort 接口通信实现代码

```objc
@interface ViewController ()<NSPortDelegate>
{
NSPort *_port;
}
@end
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    [self testPortDemo];
}
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    // NSPort
    //    [self sendPort];
    [NSThread detachNewThreadSelector:@selector(sendPort) toTarget:self withObject:nil];
}
- (void)testPortDemo {
    _port  =[[NSPort alloc] init];
    //消息处理通过代理来处理的
    _port.delegate = self;
    //把端口加在哪个线程里，就在哪个线程进行处理，下面：加在当前线程的runloop里
    [[NSRunLoop currentRunLoop] addPort:_port forMode:NSRunLoopCommonModes];
}
//发送消息
- (void)sendPort {
    NSLog(@"port发送通知before:%@",[NSThread currentThread]);
    [_port sendBeforeDate:[NSDate date] msgid:1212 components:nil from:nil reserved:0];
    NSLog(@"port发送通知after:%@",[NSThread currentThread]);
}
//处理消息
- (void)handlePortMessage:(NSPortMessage *)message {
    NSLog(@"port处理任务:%@",[NSThread currentThread]);
    NSObject * messageObj = (NSObject*)message;
    NSLog(@"=%@",[messageObj valueForKey:@"msgid"]);
}
```
运行结果

![](https://user-gold-cdn.xitu.io/2018/6/8/163de75d83ae2321?w=993&h=63&f=png&s=58865)

发送和处理在不同线程，实现通知的效果

> 以上原理解析文章来源：https://www.jianshu.com/p/051a9a3af1a4，https://www.jianshu.com/p/087a35d5f778，https://blog.csdn.net/qq_18505715/article/details/76146575