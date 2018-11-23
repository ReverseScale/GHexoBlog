---
title: iOS Principle：ReactNative
layout: post
date: 2018-06-11 21:56:27
comments: true
categories:
	- Principle
keywords: Principle
tags:
	- iOS
description: 

---
ReactNative 利用了移动平台能够运行 JavaScript (脚本语言)代码的能力，并且发挥了 JavaScript 不仅仅可以传递配置信息，还可以表达逻辑信息的优点~ 

<!-- more -->
------
👨🏻‍💻 [Github Demo](https://github.com/ReverseScale/iOSPrinciple_ReactNative)

### 方便记忆：

* React原理：一套可以用简洁的语法高效绘制 DOM 的框架
* React特点：
    * 简洁：不单单指它的 HTML 和 CSS 语法，更因为可以单用 JavaScript 构造页面
    * 高效：因为 React 独创了 Virtual DOM 机制，两大特征
        * 它存在于内存中的 JavaScript 对象，并且与 DOM 是对应关系
        * 使用高效的 DOM Diff 算法不需要对 DOM 进行重新绘制
* React Native原理：通过 JS 对 OC 的 JavaScript Core 框架的交互来实现对原生的调用
    * rn 在 OC 和 JS 两端都保存了一份配置表，里面标记了所有 OC 暴露给 JS 的模块和方法 ，js对oc的调用通过block方式实现回调
    * AppDelegate初始化过程中创建bridge，内部通过setUp创建BatchedBridge来批量读取 JS 对 OC 的方法调用并通过JavaScriptExecutor执行 JS 代码
* 创建 BatchedBridge 步骤
    * 读取 JS 源码：把 JSX 代码转成 JS 加载进内存中
    * 初始化模块信息：找到所有需要暴露给 JS 的类
    * 初始化 JS 代码的执行器：即 RCTJSCExecutor 对象
    * 生成模块列表并写入 JS 端：接受 ModuleName 并且生成模块信息
    * 执行 JavaScript 源码：通过RCTJSCExecutor执行代码，写入信息
* 相互调用方法：
    * OC调用JS：OC会通过executeBlockOnJavaScriptQueue方法在单独的线程上运行 JS 代码
        * 处理参数：_executeJSCall:(NSString *)method方法
        * 实际调用：sendAppEventWithName和body方法
    * JS调用OC：JS 会解析出方法的类、方法和方法参数并放入到 MessageQueue 中，等待 OC 调用或超时发送
        * 使用RCT_EXPORT_METHOD 宏，用来注册模块表
        * JS中使用NativeModules.CryptoExport.注册方法调用
* rn 的更新机制：React 状态机，不停地检查确认更新
    * 文本元素：ReactDOMTextComponent 比较替换文本元素
    * 基本元素：updateComponent方法分属性、节点替换基本元素
    * 自定义元素：_performComponentUpdate判断-先卸载再安装子节点
    
---

> 20180522更新：React Native 原理解析

### 准备工作，首先要有个解剖对象
从 HelloWord 看起，我们来分析RN的实现原理

```js
import React, { Component } from 'react';
import { AppRegistry, Text } from 'react-native';
class HelloWorldApp extends Component {
  render() {
    return (
      <Text>Hello world!</Text>
    );
  }
}
// 注意，这里用引号括起来的'HelloWorldApp'必须和你init创建的项目名一致
AppRegistry.registerComponent('HelloWorldApp', () => HelloWorldApp);
```

可以创建一个新的项目

```
react-native init ProjectName
```

创建完成你可以手动打开项目，也可以在项目根目录执行

```
// 启动 iOS
react-native run-ios
// 启动 Android
react-native run-android
```

准备工作完成了

----
### React 原理探究
首先我们聊聊 React，我们注意到这条数据源代码

```js
return (
      <Text>Hello world!</Text>
);
```

*“为什么 JavaScript 代码里面出现了 HTML 的语法？”*

React Native 把一组相关的 HTML 标签，也就是 app 内的 UI 控件，封装进一个组件(Component)中，这种语法被称为 JSX，它是一种 JavaScript 语法拓展。

JSX 允许我们写 HTML 标签或 React 标签，它们终将被转换成原生的 JavaScript 并创建 DOM。

在 React 框架中，除了可以用 JavaScript 写 HTML 以外，我们甚至可以写 CSS。

> 总之 React 是一套可以用简洁的语法高效绘制 DOM 的框架

* 简洁：不单单指它的 HTML 和 CSS 语法，更因为可以单用 JavaScript 构造页面；
* 高效：因为 React 独创了 Virtual DOM 机制，Virtual DOM 有两大特征，一它存在于内存中的 JavaScript 对象，并且与 DOM 是一一对应的关系；二使用高效的 DOM Diff 算法不需要对 DOM 进行重新绘制。

当然，React 并不是前端开发的全部。从之前的描述也能看出，它专注于 UI 部分，对应到 MVC 结构中就是 View 层。

要想实现完整的 MVC 架构，还需要 Model 和 Controller 的结构。在前端开发时，我们可以采用 Flux 和 Redux（基于Flux） 架构，它们并非框架(Library)，而是和 MVC 一样都是一种架构设计(Architecture)。

---
### React Native 原理探究
#### 谈谈 RN 的故事背景
React 在前端取得突破性成功以后，JavaScript 开始试图一统三端。

最终，一个基于 JavaScript，具备动态配置能力，面向前端开发者的移动端开发框架 —— React Native

----
#### 谈谈 RN 的原理
即使使用了 React Native，我们依然需要 UIKit 等框架，调用的是 Objective-C 代码，JavaScript 只是提供了配置信息和逻辑的处理结果。

而 JavaScript 是一种脚本语言，它不会经过编译、链接等操作，而是在运行时才动态的进行词法、语法分析，生成抽象语法树(AST)和字节码，然后由解释器负责执行或者使用 JIT 将字节码转化为机器码再执行。

苹果提供了一个叫做 JavaScript Core 的框架，这是一个 JavaScript 引擎。整个流程由 JavaScript 引擎负责完成。 

```objc
JSContext *context = [[JSContext alloc] init];  
JSValue *jsVal = [context evaluateScript:@"21+7"];  
int iVal = [jsVal toInt32];  
```

JavaScript 是一种单线程的语言，它不具备自运行的能力，因此总是被动调用。很多介绍 React Native 的文章都会提到 “JavaScript 线程” 的概念，实际上，它表示的是 Objective-C 创建了一个单独的线程，这个线程只用于执行 JavaScript 代码，而且 JavaScript 代码只会在这个线程中执行。

---
下面将 JavaScript 👉 OC

由于 JavaScript Core 是一个面向 Objective-C 的框架，在 Objective-C 这一端，我们对 JavaScript 上下文知根知底，可以很容易的获取到对象，方法等各种信息，当然也包括调用 JavaScript 函数。

真正复杂的问题在于，JavaScript 不知道 Objective-C 有哪些方法可以调用。

React Native 解决这个问题的方案是在 Objective-C 和 JavaScript 两端都保存了一份配置表，里面标记了所有 Objective-C 暴露给 JavaScript 的模块和方法。

这样，无论是哪一方调用另一方的方法，实际上传递的数据只有 

* ModuleId 类
* MethodId 方法
* Arguments  方法参数

当 Objective-C 接收到这三个值后，就可以通过 runtime 唯一确定要调用的是哪个函数，然后调用这个函数。

对于 Objective-C 来说，执行完 JavaScript 代码再执行 Objective-C 回调毫无难度，难点依然在于 JavaScript 代码调用 Objective-C 之后，如何在 Objective-C 的代码中，回调执行 JavaScript 代码。

目前 React Native 的做法是：在 JavaScript 调用 Objective-C 代码时，注册要回调的 Block，并且把 Block Id 作为参数发送给 Objective-C，Objective-C 收到参数时会创建 Block，调用完 Objective-C 函数后就会执行这个刚刚创建的 Block。

Objective-C 会向 Block 中传入参数和 Block Id，然后在 Block 内部调用 JavaScript 的方法，随后 JavaScript 查找到当时注册的 Block 并执行。

> 简单的表示就是：JS 👉 OC (Block 👉 JS)
---
### 继续看项目-初始化

除了 index.js 中的 JavaScript 代码，留给我们的还有 AppDelegate 中的入口方法：

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  NSURL *jsCodeLocation;

  jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index" fallbackResource:nil];

  RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                      moduleName:@"demo"
                                               initialProperties:nil
                                                   launchOptions:launchOptions];
  rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];

  self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
  UIViewController *rootViewController = [UIViewController new];
  rootViewController.view = rootView;
  self.window.rootViewController = rootViewController;
  [self.window makeKeyAndVisible];
  return YES;
}
```

实际我们操作的视图就是这个 RootView ，但是 RootView 是依托于 Bridge 对象，它是 Objective-C 与 JavaScript 交互的桥梁，后续的方法交互完全依赖于它，而整个初始化过程的最终目的其实也就是创建这个桥梁对象。

```objc
- (instancetype)initWithBundleURL:(NSURL *)bundleURL
                       moduleName:(NSString *)moduleName
                initialProperties:(NSDictionary *)initialProperties
                    launchOptions:(NSDictionary *)launchOptions {
  RCTBridge *bridge = [[RCTBridge alloc] initWithBundleURL:bundleURL
                                            moduleProvider:nil
                                             launchOptions:launchOptions];

  return [self initWithBridge:bridge moduleName:moduleName initialProperties:initialProperties];
}
```

初始化方法的核心是 setUp 方法，而 setUp 方法的主要任务则是创建 BatchedBridge。

```objc
- (void)setUp {
  RCT_PROFILE_BEGIN_EVENT(0, @"-[RCTBridge setUp]", nil);

  _performanceLogger = [RCTPerformanceLogger new];
  [_performanceLogger markStartForTag:RCTPLBridgeStartup];
  [_performanceLogger markStartForTag:RCTPLTTI];

  Class bridgeClass = self.bridgeClass;

  #if RCT_DEV
  RCTExecuteOnMainQueue(^{
    RCTRegisterReloadCommandListener(self);
  });
  #endif

  // Only update bundleURL from delegate if delegate bundleURL has changed
  NSURL *previousDelegateURL = _delegateBundleURL;
  _delegateBundleURL = [self.delegate sourceURLForBridge:self];
  if (_delegateBundleURL && ![_delegateBundleURL isEqual:previousDelegateURL]) {
    _bundleURL = _delegateBundleURL;
  }

  // Sanitize the bundle URL
  _bundleURL = [RCTConvert NSURL:_bundleURL.absoluteString];

  self.batchedBridge = [[bridgeClass alloc] initWithParentBridge:self];
  [self.batchedBridge start];

  RCT_PROFILE_END_EVENT(RCTProfileTagAlways, @"");
}
```

BatchedBridge 的作用是批量读取 JavaScript 对 Objective-C 的方法调用，同时它内部持有一个 JavaScriptExecutor，顾名思义，这个对象用来执行 JavaScript 代码。

#### 创建 BatchedBridge 的关键是 start 方法，它可以分为五个步骤：
* 读取 JavaScript 源码
* 初始化模块信息
* 初始化 JavaScript 代码的执行器，即 RCTJSCExecutor 对象
* 生成模块列表并写入 JavaScript 端
* 执行 JavaScript 源码

逐个分析上面每一步完成的操作：

*1.读取JavaScript源码* 
这一部分的具体代码实现没有太大的讨论意义。我们只要明白，JavaScript 的代码是在 Objective-C 提供的环境下运行的，所以第一步就是把 JavaScript 加载进内存中，对于一个空的项目来说，所有的 JavaScript 代码大约占用 1.5 Mb 的内存空间。

需要说明的是，在这一步中，JSX 代码已经被转化成原生的 JavaScript 代码。

*2.初始化模块信息* 
这一步在方法 initModulesWithDispatchGroup: 中实现，主要任务是找到所有需要暴露给 JavaScript 的类。

每一个需要暴露给 JavaScript 的类(也成为 Module，以下不作区分)都会标记一个宏：RCT_EXPORT_MODULE，这个宏的具体实现并不复杂

```objc
#define RCT_EXPORT_MODULE(js_name) \
RCT_EXTERN void RCTRegisterModule(Class); \
+ (NSString *)moduleName { return @#js_name; } \
+ (void)load { RCTRegisterModule(self); }
```

这样，这个类在 load 方法中就会调用 RCTRegisterModule 方法注册自己：

```objc
void RCTRegisterModule(Class moduleClass) {
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    RCTModuleClasses = [NSMutableArray new];
  });
  [RCTModuleClasses addObject:moduleClass];
}
```

因此，React Native 可以通过 RCTModuleClasses 拿到所有暴露给 JavaScript 的类。下一步操作是遍历这个数组，然后生成 RCTModuleData 对象：

```objc
for (Class moduleClass in RCTGetModuleClasses()) {
    RCTModuleData *moduleData = [[RCTModuleData alloc]initWithModuleClass:moduleClass                                                                      bridge:self];
    [moduleClassesByID addObject:moduleClass];
    [moduleDataByID addObject:moduleData];
}
```

可以想见，RCTModuleData 对象是模块配置表的主要组成部分。如果把模块配置表想象成一个数组，那么每一个元素就是一个 RCTModuleData 对象。

这个对象保存了 Module 的名字，常量等基本信息，最重要的属性是一个数组，保存了所有需要暴露给 JavaScript 的方法。

暴露给 JavaScript 的方法需要用 RCT_EXPORT_METHOD 这个宏来标记，它的实现原理比较复杂，有兴趣的读者可以自行阅读。简单来说，它为函数名加上了 **rct_export** 前缀，再通过 runtime 获取类的函数列表，找出其中带有指定前缀的方法并放入数组中:

```objc
- (NSArray<id<RCTBridgeMethod>> *)methods{
    unsigned int methodCount;
    Method *methods = class_copyMethodList(object_getClass(_moduleClass), &methodCount); // 获取方法列表
    for (unsigned int i = 0; i < methodCount; i++) {
        RCTModuleMethod *moduleMethod = /* 创建 method */
        [_methods addObject:moduleMethod];
      }
    }
    return _methods;
}
```

因此 Objective-C 管理模块配置表的逻辑是：Bridge 持有一个数组，数组中保存了所有的模块的 RCTModuleData 对象，RCTModuleData又保存了类的方法、常亮、类名等信息。只要给定 ModuleId 和 MethodId 就可以唯一确定要调用的方法。 

*3.初始化JavaScript执行器（RCTJSCExecutor）*
通过查看源码可以看到，初始化 JavaScript 执行器的时候，会调用
```objc
+ (instancetype)initializedExecutorWithContextProvider:(RCTJSContextProvider *)JSContextProvider 
applicationScript:(NSData *)applicationScript 
sourceURL:(NSURL *)sourceURL 
JSContext:(JSContext **)JSContext 
error:(NSError **)error;
```
返回的 excuter 对象是已经被同步执行的
```c++
// 执行对应的方法
- (void)callFunctionOnModule:(NSString *)module method:(NSString *)method arguments:(NSArray *)args jsValueCallback:(RCTJavaScriptValueCallback)onComplete
{
  [self _callFunctionOnModule:module method:method arguments:args returnValue:NO unwrapResult:NO callback:onComplete];
}
```
这里需要关注 nativeRequireModuleConfig 和 nativeFlushQueueImmediate 这两个block。

在这两个 block 中会通过 bridge 调用 oc 的方法。
```objc
[self executeBlockOnJavaScriptQueue:^{
    if (!self.valid) {
      return;
    }

    JSContext *context = nil;
    if (self->_jscWrapper) {
      RCTAssert(self->_context != nil, @"If wrapper was pre-initialized, context should be too");
      context = self->_context.context;
    } else {
      [self->_performanceLogger markStartForTag:RCTPLJSCWrapperOpenLibrary];
      self->_jscWrapper = RCTJSCWrapperCreate(self->_useCustomJSCLibrary);
      [self->_performanceLogger markStopForTag:RCTPLJSCWrapperOpenLibrary];

      RCTAssert(self->_context == nil, @"Didn't expect to set up twice");
      context = [self->_jscWrapper->JSContext new];
      self->_context = [[RCTJavaScriptContext alloc] initWithJSContext:context onThread:self->_javaScriptThread];
      [[NSNotificationCenter defaultCenter] postNotificationName:RCTJavaScriptContextCreatedNotification
                                                          object:context];

      configureCacheOnContext(context, self->_jscWrapper);
      installBasicSynchronousHooksOnContext(context);
    }

    __weak RCTJSCExecutor *weakSelf = self;

    context[@"nativeRequireModuleConfig"] = ^NSString *(NSString *moduleName) {
      RCTJSCExecutor *strongSelf = weakSelf;
      if (!strongSelf.valid) {
        return nil;
      }

      RCT_PROFILE_BEGIN_EVENT(RCTProfileTagAlways, @"nativeRequireModuleConfig", nil);
      NSArray *config = [strongSelf->_bridge configForModuleName:moduleName];
      NSString *result = config ? RCTJSONStringify(config, NULL) : nil;
      RCT_PROFILE_END_EVENT(RCTProfileTagAlways, @"js_call,config", @{ @"moduleName": moduleName });
      return result;
    };

    context[@"nativeFlushQueueImmediate"] = ^(NSArray<NSArray *> *calls){
      RCTJSCExecutor *strongSelf = weakSelf;
      if (!strongSelf.valid || !calls) {
        return;
      }

      RCT_PROFILE_BEGIN_EVENT(RCTProfileTagAlways, @"nativeFlushQueueImmediate", nil);
      [strongSelf->_bridge handleBuffer:calls batchEnded:NO];
      RCT_PROFILE_END_EVENT(RCTProfileTagAlways, @"js_call", nil);
    };

#if RCT_PROFILE
    __weak RCTBridge *weakBridge = self->_bridge;
    context[@"nativeTraceBeginAsyncFlow"] = ^(__unused uint64_t tag, __unused NSString *name, int64_t cookie) {
      if (RCTProfileIsProfiling()) {
        [weakBridge.flowIDMapLock lock];
        int64_t newCookie = [_RCTProfileBeginFlowEvent() longLongValue];
        CFDictionarySetValue(weakBridge.flowIDMap, (const void *)cookie, (const void *)newCookie);
        [weakBridge.flowIDMapLock unlock];
      }
    };

    context[@"nativeTraceEndAsyncFlow"] = ^(__unused uint64_t tag, __unused NSString *name, int64_t cookie) {
      if (RCTProfileIsProfiling()) {
        [weakBridge.flowIDMapLock lock];
        int64_t newCookie = (int64_t)CFDictionaryGetValue(weakBridge.flowIDMap, (const void *)cookie);
        _RCTProfileEndFlowEvent(@(newCookie));
        CFDictionaryRemoveValue(weakBridge.flowIDMap, (const void *)cookie);
        [weakBridge.flowIDMapLock unlock];
      }
    };
#endif

#if RCT_DEV
    RCTInstallJSCProfiler(self->_bridge, context.JSGlobalContextRef);

    // Inject handler used by HMR
    context[@"nativeInjectHMRUpdate"] = ^(NSString *sourceCode, NSString *sourceCodeURL) {
      RCTJSCExecutor *strongSelf = weakSelf;
      if (!strongSelf.valid) {
        return;
      }

      RCTJSCWrapper *jscWrapper = strongSelf->_jscWrapper;
      JSStringRef execJSString = jscWrapper->JSStringCreateWithUTF8CString(sourceCode.UTF8String);
      JSStringRef jsURL = jscWrapper->JSStringCreateWithUTF8CString(sourceCodeURL.UTF8String);
      jscWrapper->JSEvaluateScript(strongSelf->_context.context.JSGlobalContextRef, execJSString, NULL, jsURL, 0, NULL);
      jscWrapper->JSStringRelease(jsURL);
      jscWrapper->JSStringRelease(execJSString);
    };
#endif
  }];
}
```

*4.生成模块配置表并写入JavaScript端* 

复习一下 nativeRequireModuleConfig 这个 Block，它可以接受 ModuleName 并且生成详细的模块信息，但在前文中我们没有提到 JavaScript 是如何知道 Objective-C 要暴露哪些类的(目前只是 Objective-C 自己知道)。

这一步的操作就是为了让 JavaScript 获取所有模块的名字

```objc
- (NSString *)moduleConfig {
  NSMutableArray<NSArray *> *config = [NSMutableArray new];
  for (RCTModuleData *moduleData in _moduleDataByID) {
    if (self.executorClass == [RCTJSCExecutor class]) {
      [config addObject:@[moduleData.name]];
    } else {
      [config addObject:RCTNullIfNil(moduleData.config)];
    }
  }

  return RCTJSONStringify(@{
    @"remoteModuleConfig": config,
  }, NULL);
}
```

*5.执行JavaScript代码*

这一步也没什么技术难度可以，代码已经加载进了内存，该做的配置也已经完成，只要把 JavaScript 代码运行一遍即可。

运行代码时，第三步中所说的那些 Block 就会被执行，从而向 JavaScript 端写入配置信息。

至此，JavaScript 和 Objective-C 都具备了向对方交互的能力，准备工作也就全部完成了。

---
### 方法调用
如前文所述，在 React Native 中，Objective-C 和 JavaScript 的交互都是通过传递 ModuleId、MethodId 和 Arguments 进行的。以下是分情况讨论 
#### OC 调用 JavaScript
也许你在其他文章中曾经多次听说 JavaScript 代码总是在一个单独的线程上面调用，它的实际含义是 Objective-C 会在单独的线程上运行 JavaScript 代码

```objc
- (void)executeBlockOnJavaScriptQueue:(dispatch_block_t)block {
  if ([NSThread currentThread] != _javaScriptThread) {
    [self performSelector:@selector(executeBlockOnJavaScriptQueue:)
                 onThread:_javaScriptThread withObject:block waitUntilDone:NO];
  } else {
    block();
  }
}
```

调用JavaScript的核心代码如下

```objc
- (void)_executeJSCall:(NSString *)method
             arguments:(NSArray *)arguments
              callback:(RCTJavaScriptCallback)onComplete{
    [self executeBlockOnJavaScriptQueue:^{
        // 获取 contextJSRef、methodJSRef、moduleJSRef
        resultJSRef = JSObjectCallAsFunction(contextJSRef, (JSObjectRef)methodJSRef, (JSObjectRef)moduleJSRef, arguments.count, jsArgs, &errorJSRef);
        objcValue = /*resultJSRef 转换成 Objective-C 类型*/
        onComplete(objcValue, nil);
    }];
}
```

需要注意的是，这个函数名是我们要调用 JavaScript 的中转函数名，比如 callFunctionReturnFlushedQueue。也就是说它的作用其实是处理参数，而非真正要调用的 JavaScript 函数。 

在实际使用的时候，我们可以这样发起对 JavaScript 的调用：

```objc
[_bridge.eventDispatcher sendAppEventWithName:@"greeted"
                                         body:@{ @"name": @"nmae"}];
```

这里的 Name 和 Body 参数分别表示要调用的 JavaScript 的函数名和参数。

---
#### JavaScript调用OC

在调用 Objective-C 代码时，如前文所述，JavaScript 会解析出方法的 ModuleId、MethodId 和 Arguments 并放入到 MessageQueue 中，等待 Objective-C 主动拿走，或者超时后主动发送给 Objective-C。

Objective-C 负责处理调用的方法是 handleBuffer，它的参数是一个含有四个元素的数组，每个元素也都是一个数组，分别存放了 ModuleId、MethodId、Params，第四个元素目测用处不大。

函数内部在每一次方调用中调用 _handleRequestNumber:moduleID:methodID:params 方法，通过查找模块配置表找出要调用的方法，并通过 runtime 动态的调用：

演示JavaScript调用OC方法:

```objc
//.h文件
#import <Foundation/Foundation.h>
#import "RCTBridge.h"
#import "RCTLog.h"
#import "EncryptUtil.h"
#import "RSA.h"
@interface CryptoExport : NSObject<RCTBridgeModule>
@end

//.m文件
#import "CryptoExport.h"
@implementation CryptoExport
RCT_EXPORT_MODULE()//必须定义的宏
RCT_EXPORT_METHOD(rsaEncryptValue:(NSString *)src withKey:(NSString *)rsaKey successCallback:(RCTResponseSenderBlock)successCallback){
  NSString *rsaValue = [RSA encryptString:src publicKey:rsaKey];
  successCallback(@[rsaValue]);
}
@end
```

每个oc的方法前必须加上 RCT_EXPORT_METHOD 宏，用来注册模块表。 

在JavaScript中的调动如下

```js
NativeModules.CryptoExport.rsaEncryptValue(value, rsaKey,function (rsaValue) {
        console.log(rsaValue)
});
```
---
### React Native 更新机制
![](https://user-gold-cdn.xitu.io/2018/6/8/163de72bf327c5e2?w=1265&h=1181&f=png&s=100866)
之前我们说过，React是状态机，就是不停的去检查当前的状态，判断是否需要刷新。

调用this.setState
```js
ReactClass.prototype.setState = function(newState) {
    this._reactInternalInstance.receiveComponent(null, newState);
}
```

调用内部receiveComponent方法，这里在接受元素的时候主要分三种情况：
* 文本元素
* 基本元素
* 自定义元素

文本元素
```js
ReactDOMTextComponent.prototype.receiveComponent(nextText, transaction) {
     //跟以前保存的字符串比较
    if (nextText !== this._currentElement) {
      this._currentElement = nextText;
      var nextStringText = '' + nextText;
      if (nextStringText !== this._stringText) {
        this._stringText = nextStringText;
        var commentNodes = this.getHostNode();
        // 替换文本元素
        DOMChildrenOperations.replaceDelimitedText(
          commentNodes[0],
          commentNodes[1],
          nextStringText
        );
      }
    }
  }
```

基本元素
```js
ReactDOMComponent.prototype.receiveComponent = function(nextElement, transaction, context) {
    var prevElement = this._currentElement;
    this._currentElement = nextElement;
    this.updateComponent(transaction, prevElement, nextElement, context);
}
```

updateComponent 方法
```js
ReactDOMComponent.prototype.updateComponent = function(transaction, prevElement, nextElement, context) {
    // 略.....
    //需要单独的更新属性
    this._updateDOMProperties(lastProps, nextProps, transaction, isCustomComponentTag);
    //再更新子节点
    this._updateDOMChildren(
      lastProps,
      nextProps,
      transaction,
      context
    );
    // ......
}
```

自定义元素
```js
ReactCompositeComponent.prototype.receiveComponent = function(nextElement, transaction, nextContext) {
    var prevElement = this._currentElement;
    var prevContext = this._context;
    this._pendingElement = null;
    this.updateComponent(
      transaction,
      prevElement,
      nextElement,
      prevContext,
      nextContext
    );
}
```

updateComponent 方法
```
ReactCompositeComponent.prototype.updateComponent = function(
    transaction,
    prevParentElement,
    nextParentElement,
    prevUnmaskedContext,
    nextUnmaskedContext
){
//省略
}
```

调用内部 _performComponentUpdate 方法
```
ReactCompositeComponent.prototype._updateRenderedComponentWithNextElement = function() {

    // 判定两个element需不需要更新
    if (shouldUpdateReactComponent(prevRenderedElement, nextRenderedElement)) {
      // 如果需要更新，就继续调用子节点的receiveComponent的方法，传入新的element更新子节点。
      ReactReconciler.receiveComponent(
        prevComponentInstance,
        nextRenderedElement,
        transaction,
        this._processChildContext(context)
      );
    } else {
      // 卸载之前的子节点，安装新的子节点
      var oldHostNode = ReactReconciler.getHostNode(prevComponentInstance);
      ReactReconciler.unmountComponent(
        prevComponentInstance,
        safely,
        false /* skipLifecycle */
      );
      var nodeType = ReactNodeTypes.getType(nextRenderedElement);
      this._renderedNodeType = nodeType;
      var child = this._instantiateReactComponent(
        nextRenderedElement,
        nodeType !== ReactNodeTypes.EMPTY /* shouldHaveDebugID */
      );
      this._renderedComponent = child;

      var nextMarkup = ReactReconciler.mountComponent(
        child,
        transaction,
        this._hostParent,
        this._hostContainerInfo,
        this._processChildContext(context),
        debugID
      );
    }
```

this._updateDOMChildren 方法内部调用diff算法

![](https://user-gold-cdn.xitu.io/2018/6/8/163de72bf39b9c2b?w=691&h=889&f=png&s=50345)

实现过程
```js
_updateChildren: function(nextNestedChildrenElements, transaction, context) {
    var prevChildren = this._renderedChildren;
    var removedNodes = {};
    var mountImages = [];

    // 获取新的子元素数组
    var nextChildren = this._reconcilerUpdateChildren(
      prevChildren,
      nextNestedChildrenElements,
      mountImages,
      removedNodes,
      transaction,
      context
    );

    if (!nextChildren && !prevChildren) {
      return;
    }

    var updates = null;
    var name;
    var nextIndex = 0;
    var lastIndex = 0;
    var nextMountIndex = 0;
    var lastPlacedNode = null;

    for (name in nextChildren) {
      if (!nextChildren.hasOwnProperty(name)) {
        continue;
      }
      var prevChild = prevChildren && prevChildren[name];
      var nextChild = nextChildren[name];
      if (prevChild === nextChild) {
          // 同一个引用，说明是使用的同一个component,所以我们需要做移动的操作
          // 移动已有的子节点
          // NOTICE：这里根据nextIndex, lastIndex决定是否移动
        updates = enqueue(
          updates,
          this.moveChild(prevChild, lastPlacedNode, nextIndex, lastIndex)
        );

        // 更新lastIndex
        lastIndex = Math.max(prevChild._mountIndex, lastIndex);
        // 更新component的.mountIndex属性
        prevChild._mountIndex = nextIndex;

      } else {
        if (prevChild) {
          // 更新lastIndex
          lastIndex = Math.max(prevChild._mountIndex, lastIndex);
        }

        // 添加新的子节点在指定的位置上
        updates = enqueue(
          updates,
          this._mountChildAtIndex(
            nextChild,
            mountImages[nextMountIndex],
            lastPlacedNode,
            nextIndex,
            transaction,
            context
          )
        );

        nextMountIndex++;
      }

      // 更新nextIndex
      nextIndex++;
      lastPlacedNode = ReactReconciler.getHostNode(nextChild);
    }

    // 移除掉不存在的旧子节点，和旧子节点和新子节点不同的旧子节点
    for (name in removedNodes) {
      if (removedNodes.hasOwnProperty(name)) {
        updates = enqueue(
          updates,
          this._unmountChild(prevChildren[name], removedNodes[name])
        );
      }
    }
  }
```


> 以下是针对 Demo 项目的通信原理解释
### 通信基本原理

首先，我们来看一下在iOS中Native如何调用JS。从iOS7开始，系统进一步开放了WebCore SDK，提供JavaScript引擎库，使得我们能够直接与引擎交互拥有更多的控制权。其中，有两个最基础的概念：

```objc
JSContext // JS代码的环境，一个JSContext是一个全局环境的实例
JSValue // 包装了每一个可能的JS值：字符串、数字、数组、对象、方法等
```

通过这两个类，我们能够非常方便的实现Javascript与Native代码之间的交互，首先我们通过一个简单示例来观察Native如何调用Javascript代码：

🌰：Native -> JavaScript

```objc
// 头文件
#import <JavaScriptCore/JSContext.h>
#import <JavaScriptCore/JSValue.h>
- (void)createJSContext {
    JSContext *context = [[JSContext alloc] init];
    [context evaluateScript:@"var num = 5 + 5"];
    [context evaluateScript:@"var names = ['Grace', 'Ada', 'Margaret']"];
    [context evaluateScript:@"var triple = function(value) { return value * 3 }"];
    JSValue *tripleNum = [context evaluateScript:@"triple(num)"];
    JSValue *tripleFunction = context[@"triple"];
    JSValue *result = [tripleFunction callWithArguments:@[@5]];
    // 打印结果
    NSLog(@"JSContext function \ntripleNum:%@ \nresult:%@", tripleNum, result);
}
```

那么，JSContext如何访问我们本地客户端OC代码呢？答案是通过Blocks和JSExports协议两种方式。
我们来看一个通过Blocks来实现JS访问本地代码的示例：

🌰：JavaScript -> Native

```objc
context[@"testSay"] = ^(NSString *input) {
    NSMutableString *mutableString = [input mutableCopy];
    CFStringTransform((__bridge CFMutableStringRef)mutableString, NULL, kCFStringTransformToLatin, NO);
    CFStringTransform((__bridge CFMutableStringRef)mutableString, NULL, kCFStringTransformStripCombiningMarks, NO);
    return mutableString;
};
NSLog(@"%@", [context evaluateScript:@"testSay('hello world')"]);
```

关于JSCore库的更多学习介绍，请看JavaScriptCore。

> Java​Script​Core 相关介绍 http://nshipster.cn/javascriptcore/


### React Native 初始化过程解析

在了解React-Native中JS->Native的具体调用之前，我们先做一些准备工作，看看框架中Native app的启动过程。打开FB提供的AwesomeProject定位到appDelegate的didFinishLaunchingWithOptions方法中：

```objc
// 指定JS页面文件位置
jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.ios.bundle?platform=ios&dev=false"];
// 创建React Native视图对象
RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
moduleName:@"ReactExperiment"
initialProperties:nil
launchOptions:launchOptions];
self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
// 创建VC，并且把React Native Root View赋值给VC
UIViewController *rootViewController = [UIViewController new];
rootViewController.view = rootView;
self.window.rootViewController = rootViewController;
[self.window makeKeyAndVisible];
```

可以看到使用集成非常简单，那么RCTRootView到底做了哪些事情最后渲染将视图呈现在用户面前呢？
我们继续跟着代码往下分析就会看到我们今天的主角RCTBridge。

🥟：RCTBridge

```objc
- (instancetype)initWithBundleURL:(NSURL *)bundleURL
moduleName:(NSString *)moduleName
initialProperties:(NSDictionary *)initialProperties
launchOptions:(NSDictionary *)launchOptions {
    RCTBridge *bridge = [[RCTBridge alloc] initWithBundleURL:bundleURL
    moduleProvider:nil
    launchOptions:launchOptions];
    return [self initWithBridge:bridge moduleName:moduleName initialProperties:initialProperties];
}
```

RCTBridge是Naitive端的bridge，起着桥接两端的作用 。事实上具体的实现放置在RCTBatchedBridge中，在它的start方法中执行了一系列重要的初始化工作。这部分也是ReactNative SDK的精髓所在，基于GCD实现一套异步初始化组件框架。大致的工作流程如下图所示：

![](https://user-gold-cdn.xitu.io/2018/6/8/163de72bed8ca4d1?w=940&h=370&f=png&s=440483)

#### Load JS Source Code（并行）

加载页面源码阶段。该阶段主要负责从指定的位置（网络或者本地）加载React Native页面代码。与initModules各模块初始化过程并行执行，通过GCD分组队列保证两个阶段完成后才会加载解析页面源码。

#### Init Module（同步）

初始化加载React Native模块。该阶段会将所有注册的Native模块类整理保存到一个以Module Id为下标的数组对象中（同时还会保存一个以Module Name为Key的Dictionary，用于做索引方便后续的模块查找）。

整个模块的基础初始化和注册过程在系统Load Class阶段就会完成。React Native对模块注册的实现还是比较巧妙、方便，只需要对目标类添加相应的宏即可。

* 1.注册模块。实现RCTBridgeModule协议，并且在响应的Implemention文件中添加RCT_EXPORT_MODULE宏，该宏会为所在类自动添加一个+load方法，调用RCTBridge的RCTRegisterModule实现在Load Class阶段就完成模块注册工作。
* 2.注册函数。待注册函数所在的类必须是已注册模块，在需要注册的函数前添加RCT_EXPORT_MODULE宏即可。

当然这里需要注意的问题是模块初始化是一个同步任务，它必须被同步加载，所以当模块较多时势必会带来高延迟的问题，也是在新的版本中SDK将Module Method改为Lazy Load的原因之一。

#### Setup JS Executor（并行）

初始化JS引擎。React Native在0.18中已经很好的抽象了原来了JSExecutor，目前实现了RCTWebSocketExecutor和RCTJSCExecutor两个脚本引擎的封装，前者用于通过WebSocket链接到Chrome调试，后者则是内置默认引擎直接通过IOS SDK JSContext来实现相关的逻辑。

另外，在本阶段还会通过block hook的方式注册部分核心API

* 1.nativeRequireModuleConfig：用于在JS端获取对应的Native Module，在0.14后的版本React Native已经对初始化模块做了部分优化，把关于Native Module Method部分的加载工作放置在requireModuleConfig时才做
* 2.nativeLoggingHook：调用Native写入日志
* 3.nativeFlushQueueImmediate：手动触发执行当前Native Call队列中所有的Native处理请求
* 4.nativePerformanceNow：用于性能统计，获取当前Native的绝对时间（毫秒）

对于模块类中想要声明的方法，需要添加RCT_EXPORT_METHOD宏。它会给方法名添加” rct_export “前缀。

🌰：React 调用 Native 的 SVProgressHUD 提示窗

在 Native 中声明方法
```objc
RCT_EXPORT_METHOD(calliOSActionWithOneParams:(NSString *)name) {
    [SVProgressHUD setDefaultMaskType:SVProgressHUDMaskTypeBlack];
    [SVProgressHUD showSuccessWithStatus:[NSString stringWithFormat:@"参数：%@",name]];
}
```

在 React 中调用 calliOSActionWithOneParams 方法
```js
<TouchableOpacity style={styles.calltonative}
    onPress={()=>{
        RNCalliOSAction.calliOSActionWithOneParams('hello');
    }}>
    <Text>点击调用 Native 方法, 并传递一个参数</Text>
</TouchableOpacity>
```

#### Module Config（并行）

这步将第2步中的Native模块类转换成Json，保存为remoteModuleConfig。注意在这里获取到的列表并非含有完整模块信息，而仅仅是一个Module List而已。

```js
{
"remoteModuleConfig":[
[
"HTSimpleAPI", // module
],
[
"RCTViewManager",
],
[
"HTTestView",
],
[
"RCTAccessibilityManager",
],
...
],
}
```

#### JS Source Code代码分析

JS的主入口index.ios.js在我们看来只有短短数十行，然而这不是最终执行的代码。React-Native页面源码需要通过Transform Server转换处理，并把转化后的模块一起合并为一个bundle.js，这个过程称为buildBundle。转换后的index.ios.bundle才是最终可被Javascript引擎直接解释运行的代码。下面我们按照主程序的逻辑来分析源码几个核心模块实现原理。

在React Server中需要查看Bundle的模块映射关系可以直接访问：http://localhost:8081/index.ios.bundle.map，查看相关依赖和Bundle的缓存则可以访问： http://localhost:8081/debug

1.BatchedBridge

在上一部分我们知道，Native完成模块初始化后会通过Inject Json Config将配置信息同步至JS里中的全局变量__fbBatchedBridgeConfig，打开BatchedBridge.js我们可以看到如下代码。

```js
__d('BatchedBridge',function(global, require, module, exports) { 'use strict';
var MessageQueue=require('MessageQueue');
var BatchedBridge=new MessageQueue(
__fbBatchedBridgeConfig.remoteModuleConfig,
__fbBatchedBridgeConfig.localModulesConfig);
//......
Object.defineProperty(global,'__fbBatchedBridge',{value:BatchedBridge});
module.exports = BatchedBridge;
});
```

对于这段代码，我们可以得出以下几个结论：

* 1.在JS端也存在一个bridge模块BatchedBridge，也是与Native建立双向通信的关键所在
* 2.BatchedBridge是一个MessageQueue实例，它在创建时传入了__fbBatchedBridgeConfig值保存Native端支持的模块列表配置

BatchedBridge在创建时将自己写入全局变量__fbBatchedBridge上，这样Native可以通过JSContext[@”__fbBatchedBridge”]访问到JS bridge对象。

2.MessageQueue

接着我们继续看MessageQueue，它在整个通讯链路的机制上面有着重要作用，首先我们来观察一下它的构造函数。

```js
constructor(remoteModules, localModules) {
this.RemoteModules = {};
this._callableModules = {};
this._queue = [[], [], [], 0];
this._moduleTable = {};
this._methodTable = {};
this._callbacks = [];
this._callbackID = 0;
this._callID = 0;
//......
let modulesConfig = this._genModulesConfig(remoteModules);
this._genModules(modulesConfig);
//......
}
```
从构造函数，我们大致能了解MessageQueue的几个信息：

* 1.RemoteModules属性，用于保存Native端模块配置
* 2.Callbacks属性缓存js的回调方法
* 3.Queue事件队列用于处理各类事件等

在构造函数中，解析Native传入的remoteModules JSON，转换成JS对象

3.Config Modules

根据上一步MessageQueue的逻辑，继续往下跟踪_genModules函数，可以看到在MessageQueue已经对Native注入的Module Config做了一次预处理，如果debug模式可以看到大致的数据结构会转换成如下表中所示结构（其中HTSimepleAPI是一个自定义模块）。

```js
config = ["HTSimpleAPI", Array[1]], moduleID = 0
config = null, moduleID = 1
config = null, moduleID = 2
config = ["RCTAccessibilityManager", Array[3]], moduleID = 3
```

至于这样的预处理有什么作用，我们继续往下分析，后面再来总结。

4.Lazily Config Methods

对于NativeModule，它们在上一步之后只有一个包含Module Name等简单信息的Module List的对象，只有在实际调用了该模块之后才会加载该模块的具体信息（比如暴露的API等）。

```js
const NativeModules = {};
Object.keys(RemoteModules).forEach((moduleName) => {
Object.defineProperty(NativeModules, moduleName, {
enumerable: true,
get: () => {
let module = RemoteModules[moduleName];
if (module && typeof module.moduleID === 'number' && global.nativeRequireModuleConfig) {
const json = global.nativeRequireModuleConfig(moduleName);
const config = json && JSON.parse(json);
module = config && BatchedBridge.processModuleConfig(config, module.moduleID);
RemoteModules[moduleName] = module;
}
return module;
},
});
});
```

这段代码定义了一个全局模块NativeModules，遍历之前取到的remoteModules，将每一个module在NativeModules对象上扩展了一个getter方法，该方法中通过nativeRequireModuleConfig进一步加载模块的详细信息，通过processModuleConfig对模块信息进行预处理。进一步分析代码就可以发现这个方法其实是Native中定义的全局JS Block（nativeRequireModuleConfig）。

接下来我们继续看processModuleConfig中具体的代码逻辑，如下表所示：

```js
processModuleConfig(config, moduleID) {
    const module = this._genModule(config, moduleID);
    return module;
}
_genMethod(module, method, type) {
//......
    fn = function(...args) {
    return self.__nativeCall(module, method, args, onFail, onSucc);
};
//......
return fn;
}
```

processModuleConfig方法的主要工作是生成methods配置，并对每一个method封装了一个闭包fn，当调用method时，会转换成成调用self.__nativeCall(moduleID, methodID, args, onFail, onSucc)方法

预处理完成后，在JavaScript环境中的Moudle Config信息才算完整，包含Module Name、Native Method等信息，具体信息如下所示。

```js
config = ["HTSimpleAPI", Array[1]], moduleID = 0
methodName = "test", methodID = 0
config = null, moduleID = 1
config = null, moduleID = 2
config = ["RCTAccessibilityManager", Array[3]], moduleID = 3
methodName = "setAccessibilityContentSizeMultipliers", methodID = 0
methodName = "getMultiplier", methodID = 1
methodName = "getCurrentVoiceOverState", methodID = 2
```

还记得第二部分第5步中Native端生成的模块配置表吗？结合它的结构，我们可以得知：对于Module&Method，在Native和JS端都以数组的形式存放，数组下标即为它们的ModuleID和MethodID。

5.__nativeCall

分析完Bridge部分的映射关系以及模块加载，那么我们再来看看最终调用Native代码是如何实现的。当JS调用module.method时，其实调用了self.__nativeCall(module, method, args, onFail, onSucc)，对于__nativeCall方法：

```js
__nativeCall(module, method, params, onFail, onSucc) {
    if (onFail || onSucc) {
    ......
    onFail && params.push(this._callbackID);
    this._callbacks[this._callbackID++] = onFail;
    onSucc && params.push(this._callbackID);
    this._callbacks[this._callbackID++] = onSucc;
    }
this._queue[MODULE_IDS].push(module);
this._queue[METHOD_IDS].push(method);
this._queue[PARAMS].push(params);
global.nativeFlushQueueImmediate(this._queue);
......
}
```

这段代码为每个method创建了一个闭包fn，在__nativeCall方法中，并且在这里做了两件重要的工作：

* 1.把onFail和onSucc缓存到_callbacks中，同时把callbackID添加到params
* 2.把moduleID, methodID, params放入队列中，回调Native代码.

__nativeCall如何做到回调Native代码呢？看第二部分第3步，在初始化JS引擎JSExecutor Setup时，Native端注册一个全局block回调nativeFlushedQueueImmediate，nativeCall在处理完毕后，通过该回调把队列作为返回值传给Native。nativeFlushedQueueImmediate的实现如下所示。

```js
[self addSynchronousHookWithName:@"nativeFlushQueueImmediate" usingBlock:^(NSArray *calls){
RCTJSCExecutor *strongSelf = weakSelf;
    if (!strongSelf.valid || !calls) {
        return;
    }
[strongSelf->_bridge handleBuffer:calls batchEnded:NO];
}];
```

这里的handleBuffer就是Native端解析JS的模块调用最后通过NSInvocation机制调用Native代码对应的逻辑。有兴趣的朋友继续跟踪handleBuffer代码会发现，他的实现和React在JS端定义的MessageQueue有惊人的相似之处。

6.Call JS function & Callbacks

最后，我们回过头来看看Native端是如何调用JS端的相关逻辑的，这部分我们需要回到MessageQueue.js代码中来，可以看到MessageQueue暴露了3个核心方法：’invokeCallbackAndReturnFlushedQueue’、’callFunctionReturnFlushedQueue’、’flushedQueue’。

```js
// 将API暴露到全局作用域中
[
'invokeCallbackAndReturnFlushedQueue',
'callFunctionReturnFlushedQueue',
'flushedQueue',
].forEach((fn) => this[fn] = this[fn].bind(this));
…
// 声明带有返回值的函数
callFunctionReturnFlushedQueue(module, method, args) {
guard(() => {C
this.__callFunction(module, method, args);
this.__callImmediates();
});
return this.flushedQueue();
}
// 声明带有Callback的函数
invokeCallbackAndReturnFlushedQueue(cbID, args) {
guard(() => {
this.__invokeCallback(cbID, args);
this.__callImmediates();
});
return this.flushedQueue();
}
```

callFunctionReturnFlushedQueue用于实现Native调用带有返回值的JS端函数（这里的返回值也是通过Queue来模拟）；
invokeCallbackAndReturnFlushedQueue用于实现Native调用带有Call的JS端函数（可以将Native的Callback作为JS端函数的入参，JS端执行完后调用Native的Callback）。

对于callFunctionReturnFlushedQueue方法，它最终调用的是__callFunction：

```js
__callFunction(module, method, args) {
......
var moduleMethods = this._callableModules[module];
......
moduleMethods[method].apply(moduleMethods, args);
}
```

可以看到，此处会根据Native传入的module, method，调用JS端相应的模块并传入参数列表args.
同时我们又可以获得对于MessageQueue的另一条推测，_callableModules用来存放JS端暴露给Native的模块，进一步分析我们可以发现SDK中正是通过registerCallableModules方法注册JS端暴露API模块。

对于JS bridge提供的调用回调方法invokeCallbackAndReturnFlushedQueue，原理上和callFunction差不多，不再细说。

#### JS <-> Native 通信原理
1.Native->JS

综上所述，在JS端提供callFunctionReturnFlushedQueue，Native bridge调用JS端方法时，应该使用这个方法。查看Native代码实现可知，RCTBridge封装了enqueueJSCall方法调用JS，梳理Native->JS的整体交互流程如下图所示。

![](https://user-gold-cdn.xitu.io/2018/6/8/163de72bf259629a?w=940&h=1168&f=png&s=123372)

之前已经论述过，如果在NATIVE端需要自定义模块提供给JS端使用那么该类需要实现RCTBridgeModule协议 。

此外，React-Native提供了另一种基于通知的方式，通过RCTEventDispatcher发送消息通知 。eventDispatcher作为Native Bridge的属性，封装了sendEventWithName:body:方法。使用时，Native中类同样需要实现RCTBridgeModule协议，通过self.bridge发送通知，JS端对应事件的EventEmitter添加监听处理调用。

> 查看sendEvent方法的代码可以发现，这种方式本质上还是调用enqueueJSCall方法。官方推荐我们使用通知的方式来实现 Native->JS，这样可以减少模块初始化加载解析的时间。

2.JS->Native

最后，我们来看一下JS如何调用Native。答案是JS不会主动传递数据给Native，也不能直接调用Native（一种情况除外，在入口直接通过NativeModules调用API），只有在Native调用JS时才会通过返回值触发调用。因为Native是基于事件响应机制的，比如触摸事件、启动事件、定时器事件、回调事件等。

当事件发生时，Native会调用JS相应模块处理，完毕后再通过返回值把队列传递给Native执行对应的代码。

![](https://user-gold-cdn.xitu.io/2018/6/8/163de72bf3201885?w=940&h=1168&f=png&s=123372)

如上图所示，整个调用过程可以归纳为：

* 1.JS把需要Module, Method, args(CallbackID)保存在队列中， 作为返回值通过blocks回调Native
* 2.Native调用相应模块方法，完成
* 3.Native通过CallbackID调用JS回调

### 总结

React Native的通讯基础建立在传统的JS Bridge之上，不过对于Bridge处理的MessageQueue机制、模块定义、加载机制上的巧妙处理指的借鉴。对于上述的整个原理解析可以概括为以下四个部分：

* 1.在启动阶段，初始化JS引擎，生成Native端模块配置表存于两端，其中模块配置是同步取得，而各模块的方法配置在该方法被真正调用时懒加载。
* 2.Native和JS端分别有一个bridge，发生调用时，调用端bridge查找模块配置表将调用转换成{moduleID, methodID, args(callbackID)}，处理端通过同一份模块配置表转换为实际的方法实现。
* 3.Native->JS，原理上使用JSCore从Native执行JS代码，React-Native在此基础上给我们提供了通知发送的执行方式。
* 4.JS->Native，原理上JS并不主动调用Native，而是把方法和参数(回调)缓存到队列中，在Native事件触发并访问JS后，通过blocks回调Native。

> 以上原理解析文章来源：http://i.dotidea.cn/2016/05/react-native-communication-principle-for-ios/、https://blog.csdn.net/passionhe/article/details/52498061、https://blog.csdn.net/xiangzhihong8/article/details/54425807