---
title: iOS RAC 的使用总结
layout: post
date: 2017-07-16 22:56:27
comments: true
categories:
	- Summary
keywords: RAC
tags:
	- iOS
description: 

---
Reactive Cocoa(简称RAC),是 GitHub 上开源的一个应用于 iOS 和 OS X 开发的一个新框架，RAC具有函数式编程和响应者编程的特性~                                                                                                                        

<!-- more -->

![](https://user-gold-cdn.xitu.io/2018/3/12/16218ce0892a1ed4?w=647&h=164&f=png&s=46414)

> ReactiveCocoa解决的问题:
* 1.传统iOS开发过程中,状态以及状态之间依赖过多的问题
* 2.传统MVC架构的问题:Controller比较复杂,可测试性差
* 3.提供统一的消息传递机制

![](https://user-gold-cdn.xitu.io/2018/3/12/16218ccd7902b852?w=671&h=322&f=png&s=122960)

#### 键值观察--监听 TF 的值发生变化

```OC
- (void)demo1{
    @weakify(self);
    [self.tF.rac_textSignal subscribeNext:^(NSString *value) {
        @strongify(self);
        self.value = value;
    }];

    //当self.value的值变化时调用Block，这是用KVO的机制，RAC封装了KVO
    [RACObserve(self, value) subscribeNext:^(NSString *value) {
        NSLog(@"%@",value);
    }];
}
```

#### map 的使用
```OC
- (void)demo2{
  //创建一个信号
    RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
      //这个信号里面有一个Next事件的玻璃球和一个complete事件的玻璃球
        [subscriber sendNext:@"唱歌"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //对信号进行改进,当信号里面流的是唱歌.就改进为'跳舞'返还给self.value
    RAC(self, tF.text) = [signalA map:^id(NSString *value) {
        if ([value isEqualToString:@"唱歌"]) {
            return @"跳舞";
        }
        return @"";
    }];     
}
```

#### filter 使用,你向西，他就向东，他向左，你就向右
```OC
- (void)demo3{
    //创建两个通道,一个从A流出的通道A,和一个从B流出的通道B
    RACChannelTerminal *channelA = RACChannelTo(self, value);
    RACChannelTerminal *channelB = RACChannelTo(self, value2);
   
    //改造通道A,使通过通道A的值,如果等于'西',就改为'东'
    [[channelA map:^id(NSString *value) {
        if ([value isEqualToString:@"西"]) {
            NSLog(@"东");
            return @"东";
        }
        NSLog(@"====== %@",value);
        return value;
    }] subscribe:channelB];//通道A流向B
   
    //改造通道B,使通过通道B的值,如果等于'左',就改为'右'传出去
    [[channelB map:^id(id value) {
        if ([value isEqualToString:@"左"]) {
            NSLog(@"右");
            return @"右";
        }
        NSLog(@"====== %@",value);
        return value;
    }] subscribe:channelA];//通道B流向通道A
   
    //KVO监听valueA的值的变化,过滤valueA的值,返回Yes表示通过
    //只有value有值,才可通过
    [[RACObserve(self, value) filter:^BOOL(id value) {
        return value ? YES : NO;
    }] subscribeNext:^(id x) {
        NSLog(@"你向%@",x);
    }];
   
    //KVO监听value2的变化
    [[RACObserve(self, value2) filter:^BOOL(id value) {
        return value ? YES: NO;
    }] subscribeNext:^(id x) {
        NSLog(@"他向%@",x);
    }];
   
    //下面使value的值和value2的值发生改变
    self.value = @"西";
    self.value2 = @"左"; 
}
```

#### 代理

1)代理的第一种写法

.m文件
```OC
- (void)demo4{
    [self.delegate makeAnApp:@"12345上山打老虎" String:@"老虎不在家,怎么办"];
}
```
.h文件
```OC
- (void)makeAnApp:(NSString *)string String:(NSString *)string;
@end

@interface Base5Controller : UIViewController
@property (nonatomic, assign)id<ProgrammerDelegate> delegate;
```

第一个控制器的.h
```OC
Base5Controller *base = [[Base5Controller alloc] init];
    //  base.delegate = self;
	    [self demo4];
    // 这里是个坑,必须将代理最后设置,否则信号是无法订阅到的
    // 雷纯峰大大是这样子解释的:在设置代理的时候，系统会缓存这个代理对象实现了哪些代码方法
    // 如果将代理放在订阅信号前设置,那么当控制器成为代理时是无法缓存这个代理对象实现了哪些代码方法的
	    base.delegate = self;
	    [self.navigationController pushViewController:base animated:YES];
    } else {
        [self.navigationController pushViewController:[cl new] animated:YES];
    }
}

#pragma mark---demo4
//使用RAC代替代理时,rac_signalForSelector: fromProtocol:这个代替代理的方法使用时,切记要将self设为代理这句话放在订阅代理信号的后面写,否则会无法执行
- (void)demo4{
    //为self添加一个信号,表示代理ProgrammerDelegate的makeAnApp;
    //RACTuple 相当于swift中的元祖
    [[self rac_signalForSelector:@selector(makeAnApp:String:) fromProtocol:@protocol(ProgrammerDelegate)] subscribeNext:^(RACTuple *x) {
        //这里可以立即为makeAnApp的方法要执行的代码
        NSLog(@"%@ ",x.first);
        NSLog(@"%@",x.second);
    }];
}
```

2)方法2 使用RACSubject替代代理
```OC
/**
*  RACSubject:信号提供者,自己可以充当信号,又能发送信号
创建方法:
1.创建RACSubject
2.订阅信号
3.发送信号
工作流程:
1.订阅信号,内部保存了订阅者,和订阅者相应block
2.当发送信号的,遍历订阅者,调用订阅者的nextBolck

注:如果订阅信号,必须在发送信号之前订阅信号,不然收不到信号,有利用区别RACReplaySubject
*/

-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    RacSubjectController *racsub = [[RacSubjectController alloc] init];
    racsub.subject = [RACSubject subject];
    [racsub.subject subscribeNext:^(id x) {
        NSLog(@"被通知了%@",x);
    }];
    [self.navigationController pushViewController:racsub animated:YES];
}
```

在RacSubjectController.h里面声明属性
```OC
@property (nonatomic, strong) RACSubject *subject;
```

.m里面进行数据的传递
```OC
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
	if (self.subject) {
	    [self.subject sendNext:@1];
	}
}
```

#### 广播
```OC
//发送通知
- (void)demo5{
    NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
    //发送广播通知
    [center postNotificationName:@"妇女之友" object:nil userInfo:@{@"技巧":@"用心听"}];
}

//接收通知
NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
//RAC的通知不需要我们手动移除
//注册广播通知
RACSignal *siganl = [center rac_addObserverForName:@"妇女之友" object:nil];
//设置接收通知的回调处理
[siganl subscribeNext:^(NSNotification *x) {
    NSLog(@"技巧: %@",x.userInfo[@"技巧"]);
}];
```

#### 两个信号串联,两个管串联,一个管处理完自己的东西,下一个管才开始处理自己的东西
```OC
- (void)demo6{
    //创建一个信号管A
    RACSignal *siganlA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"吃饭"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //创建一个信号管B
    RACSignal *siganlB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"吃的饱饱的,才可以睡觉的"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //串联管A和管B
    RACSignal *concatSiganl = [siganlA concat:siganlB];
    //串联后的接收端处理 ,两个事件,走两次,第一个打印siggnalA的结果,第二次打印siganlB的结果
    [concatSiganl subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
}
```

#### 并联,只要有一个管有东西,就可以打印
```OC
- (void)demo7{
  //创建信号A
    RACSignal *siganlA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"纸厂污水"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //创建信号B
    RACSignal *siganlB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"电镀厂污水"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //并联两个信号,根上面一样,分两次打印
    RACSignal *mergeSiganl = [RACSignal merge:@[siganlA,siganlB]];
    [mergeSiganl subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
}
```

#### 组合,只有两个信号都有值,才可以组合
```OC
- (void)demo8{
    //定义2个自定义信号
    RACSubject *letters = [RACSubject subject];
    RACSubject *numbers = [RACSubject subject];
   
    //组合信号
    [[RACSignal combineLatest:@[letters,numbers] reduce:^(NSString *letter, NSString *number){
        return [letter stringByAppendingString:number];
    }] subscribeNext:^(id x) {
        NSLog(@"%@",x);
       
    }];
   
    //自己控制发生信号值
    [letters sendNext:@"A"];
    [letters sendNext:@"B"];
    [numbers sendNext:@"1"]; //打印B1
    [letters sendNext:@"C"];//打印C1
    [numbers sendNext:@"2"];//打印C2
}
```

#### 合流压缩
```OC
- (void)demo9{
    //创建信号A
    RACSignal *siganlA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"红"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //创建信号B
    RACSignal *siganlB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"白"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //合流后处理的是压缩包,需要解压后才能取到里面的值
    [[siganlA zipWith:siganlB] subscribeNext:^(id x) {
        //解压缩
        RACTupleUnpack(NSString *stringA, NSString *stringB) = x;
        NSLog(@"%@ %@",stringA, stringB);
    }];
}
```

#### 映射,我可以点石成金
```OC
- (void)demo10{
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:nil];
        [subscriber sendCompleted];
        return nil;
    }];
    //对信号进行改造,改"石"成"金"
    siganl = [siganl map:^id(NSString *value) {
        if ([value isEqualToString:@"石"]) {
            return @"金";
        }
        return value;
    }];
   
    //打印,不论信号发送的是什么,这一步都会走的
    [siganl subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
}
```

#### 过滤,未满18岁,禁止入内
```OC
- (void)demo11{
    RACSignal *singal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@(15)];
        [subscriber sendNext:@(17)];
        [subscriber sendNext:@(21)];
        [subscriber sendNext:@(14)];
        [subscriber sendNext:@(30)];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //过滤信号,打印
    [[singal filter:^BOOL(NSNumber *value) {
        //大于18岁的,才可以通过
        return value.integerValue >= 18;//return为yes可以通过
    }] subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
}
```

#### 秩序(flattenMap 方法也可以换成 then 方法,效果一样)
```OC
-(void)demo12{
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSLog(@"打蛋液");
        [subscriber sendNext:@"蛋液"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //对信号进行秩序秩序的第一步
    siganl = [siganl flattenMap:^RACStream *(NSString *value) {
        //处理上一步的RACSiganl的信号value.这里的value=@"蛋液"
        NSLog(@"把%@倒进锅里面煎",value);
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            [subscriber sendNext:@"煎蛋"];
            [subscriber sendCompleted];
            return nil;
        }];
    }];
    //对信号进行第二步处理
    siganl = [siganl flattenMap:^RACStream *(id value) {
        NSLog(@"把%@装载盘里",value);
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            [subscriber sendNext:@"上菜"];
            [subscriber sendCompleted];
            return nil;
        }];
    }];
   
    //最后打印 最后带有===上菜
    [siganl subscribeNext:^(id x) {
        NSLog(@"====%@",x);
    }];
}
```

#### 命令
```OC
-(void)demo13{
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        //打印：今天我投降了
      //命令执行代理
        NSLog(@"%@我投降了",input);
        //返回一个RACSignal信号
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            return nil;
        }];
    }];
    //执行命令
    [command execute:@"今天"];
}
```

#### 延迟
```OC
- (void)demo14{
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSLog(@"等等我,我还有10s就到了");
        [subscriber sendNext:@"北极"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //延迟10s接受next的玻璃球
    [[siganl delay:10] subscribeNext:^(id x) {
        NSLog(@"我到了%@",x);
    }];
}
```

#### 重放
```OC
- (void)demo15{
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSLog(@"电影");
        [subscriber sendNext:@"电影"];
        [subscriber sendCompleted];
        return nil;
    }];
   
    //创建该普通信号的重复信号
    RACSignal *replaySiganl = [siganl replay];
    //重复接受信号
    [replaySiganl subscribeNext:^(NSString *x) {
        NSLog(@"小米%@",x);
    }];
    [replaySiganl subscribeNext:^(NSString *x) {
        NSLog(@"小红%@",x);
    }];
}
```

#### 定时---每隔 8 小时服用一次药
```OC
- (void)demo16{
    //创建定时器信号.定时8小时
    RACSignal *siganl = [RACSignal interval:60 * 60 * 8 onScheduler:[RACScheduler mainThreadScheduler]];
    //定时器执行代码
    [siganl subscribeNext:^(id x) {
        NSLog(@"吃药");
    }];
}
```

#### 超时
```OC
- (void)demo17{
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSLog(@"我快到了");
        RACSignal *sendSiganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            [subscriber sendNext:nil];
            [subscriber sendCompleted];
            return nil;
        }];
       
        //发生信号要1个小时10分钟才到
        [[sendSiganl delay:60 * 70] subscribeNext:^(id x) {
          //这里才发送next玻璃球到siganl
            [subscriber sendNext:@"我到了"];
            [subscriber sendCompleted];
        }];
        return nil;
    }];
   
    [[siganl timeout:60 * 60 onScheduler:[RACScheduler mainThreadScheduler]] subscribeNext:^(id x) {
        NSLog(@"等了你一个小时,你一直没来,我走了");
    }];
}
```

#### 重试
```OC
- (void)demo18{
    __block int failedCount = 0;
    //创建信号
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        if (failedCount < 100) {
            failedCount ++;
            NSLog(@"我失败了");
            [subscriber sendError:nil];
        }else{
            NSLog(@"经历了数百次后,我成功了");
            [subscriber sendNext:nil];
        }
        return nil;
    }];
   
    //重试
    RACSignal *retrySiganl = [siganl retry];
    //直到发生next的玻璃球
    [retrySiganl subscribeNext:^(id x) {
        NSLog(@"重要成功了");
    }];
}
```

#### 节流,不好意思,这里每一秒只能通过一个人,如果 1s 内发生多个,只通过最后一个
```OC
- (void)demo19{
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
      //即使发送一个next的玻璃球
        [subscriber sendNext:@"A"];
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [subscriber sendNext:@"B"];
        });
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [subscriber sendNext:@"C"];
            [subscriber sendNext:@"D"];
            [subscriber sendNext:@"E"];
        });
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [subscriber sendNext:@"F"];
        });
        return nil;
    }];
   
    //对信号进行节流,限制时间内一次只能通过一个玻璃球
    [[signal throttle:1] subscribeNext:^(id x) {
        NSLog(@"%@通过了",x);
    }];
    /*
    [2015-08-16 22:08:45.677]旅客A
    [2015-08-16 22:08:46.737]旅客B
    [2015-08-16 22:08:47.822]旅客E
    [2015-08-16 22:08:48.920]旅客F
    */
}
```

#### 条件(takeUntil 方法,当给定的 signal 完成前一直取值)
```OC
- (void)demo20{
    RACSignal *takeSiganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
      //创建一个定时器信号,每一秒触发一次
        RACSignal *siganl = [RACSignal interval:1 onScheduler:[RACScheduler mainThreadScheduler]];
        [siganl subscribeNext:^(id x) {
          //在这里定时发送next玻璃球
            [subscriber sendNext:@"直到世界尽头"];
        }];
        return nil;
    }];

    //创建条件信号
    RACSignal *conditionSiganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
      //设置5s后发生complete玻璃球
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"世界的今天到了,请下车");
            [subscriber sendCompleted];
        });
        return nil;
    }];
   
    //设置条件,takeSiganl信号在conditionSignal信号接收完成前,不断取值
    [[takeSiganl takeUntil:conditionSiganl] subscribeNext:^(id x) {
        NSLog(@"%@",x);
    }];
}
```

#### RACReplaySubject 使用
```OC
/**
*  RACReplaySubject创建方法
        1.创建RACSubject
        2.订阅信号
        3.发送信号
    工作流程:
    1.订阅信号,内部保存了订阅者,和订阅者相应的block
    2.当发送信号的,遍历订阅者,调用订阅者的nextBlock
    3.发送的信号会保存起来,当订阅者订阅信号的时候,会将之前保存的信号,一个个作用于新的订阅者,保存信号的容量由capacity决定,这也是有别于RACSubject的
*/

-(void)RACReplaySubject{
    RACReplaySubject *replaySubject = [RACReplaySubject subject];
    [replaySubject subscribeNext:^(id x) {
        NSLog(@" 1 %@",x);
    }];
   
    [replaySubject subscribeNext:^(id x) {
        NSLog(@"2 %@",x);
    }];
    [replaySubject sendNext:@7];
    [replaySubject subscribeNext:^(id x) {
        NSLog(@"3 %@",x);
    }];
}
```

#### rac_liftSelector:withSignals 使用
```OC
//这里的rac_liftSelector:withSignals 就是干这件事的，它的意思是当signalA和signalB都至少sendNext过一次，接下来只要其中任意一个signal有了新的内容，doA:withB这个方法就会自动被触发
-(void)test{
    RACSignal *sigalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        double delayInSeconds = 2.0;
        dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delayInSeconds *NSEC_PER_SEC));
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(popTime * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [subscriber sendNext:@"A"];
        });
        return nil;
    }];
   
    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [subscriber sendNext:@"B"];
        [subscriber sendNext:@"Another B"];
        [subscriber sendCompleted];
        return nil;
    }];
    [self rac_liftSelector:@selector(doA:withB:) withSignals:sigalA,signalB, nil];
}
```

---
来源：简书作者未魏雨辰 《iOS RAC的使用总结》