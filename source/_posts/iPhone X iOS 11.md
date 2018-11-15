---
title: iPhone X + iOS 11 适配指南
layout: post
date: 2017-09-30 17:56:27
comments: true
categories:
	- Tips
keywords: 适配
tags:
	- Tips
description: 

---
北京时间今天凌晨1点，苹果再一次让全世界沸腾。iPhone X 带给我们的最大改变：全屏 Super Retina显示屏。它提供了更多的内容显示空间，同时也营造了更加深入的沉浸感。作为 iOS 开发者，在为强大的 Face ID 和全面屏欣喜的同时，我更担忧“齐刘海”的适配！ 下面结合官方的人机交互指南，来了解下如何设计 App 才能在iPhone X 和其他所有 iOS 设备上都看起来很棒~                                                                                                                        

<!-- more -->
----

### 前言

北京时间今天凌晨1点，苹果再一次让全世界沸腾。iPhone X 带给我们的最大改变：全屏 Super Retina显示屏。它提供了更多的内容显示空间，同时也营造了更加深入的沉浸感。作为 iOS 开发者，在为强大的 Face ID 和全面屏欣喜的同时，我更担忧“齐刘海”的适配！ 下面结合官方的人机交互指南，来了解下如何设计 App 才能在iPhone X 和其他所有 iOS 设备上都看起来很棒。


![](https://user-gold-cdn.xitu.io/2017/9/30/5d7d451bd75a9ef3a908b8f8da444a52)


----

### 理论部分

#### 屏幕尺寸

![](https://user-gold-cdn.xitu.io/2018/2/7/1616f5876e1d96aa?w=507&h=262&f=png&s=39948)

在竖屏下，iPhone X 上的显示的宽度与 iPhone 6，iPhone 7和 iPhone 8的4.7英寸显示屏的宽度保持一致。然而，iPhone X 比4.7英寸显示屏高了145个点，这导致增加了大约20％的垂直高度内容。


![](https://user-gold-cdn.xitu.io/2017/9/30/177efaedcb9b1c8b56ba54e6a95ddd0c)


大家在为设计师悲伤的同时也不要忘记添加启动图（LaunchImage or LaunchScreen.storyboard）呦~


![](https://user-gold-cdn.xitu.io/2018/2/7/1616f5a36792d391?w=519&h=655&f=png&s=569539)

----

#### 安全区

![](https://user-gold-cdn.xitu.io/2018/2/7/1616f5bd1e16fac9?w=530&h=277&f=png&s=127513)

在 iPhone X 布局中，最关键的是：必须确保布局填满屏幕，同时又不会被设备的圆角，传感器外壳或用于访问主屏幕的指示灯所遮盖，苹果为称此区域为“安全区”。


![](https://user-gold-cdn.xitu.io/2017/9/30/a6a5328fb1d354b865f3eae14b12f4d4)


可喜的是，大多数标准的系统提供的UI元素和控件（如 navigation bars，tables 和 collections）都已经为新外形做了很好的适配。 背景已经延伸到显示器的边缘，并且UI元件被很恰当地插入和定位在安全区域。 因此，对于具有自定义布局的 App，支持iPhone X 也应该比较容易，特别是如果使用了 AutoLayout 并遵守安全区域(safe area)和边距布局(margin layout)指南。这些在上文都已经有过较详细的阐述。 下面说几点需要特别注意的：

![](https://user-gold-cdn.xitu.io/2018/2/7/1616f5d122a3380b?w=518&h=442&f=png&s=206802)

* 在 iPhone X 上预览 App: 在拿到新机之前，也可以先使用 Simulator 来预览和检查下布局问题。 但是一些依赖硬件的功能，如图像效果和交互体验，最好还是在真机上预览。

* 始终保持全屏体验: 确保背景延伸到显示区域的边缘，以及垂直可滚动的布局（如 tables 和 collections）一直延续到底部。

* 防止边缘内容被裁剪: 一般来说，内容应该是居中对称的，这样它在任何方向看起来都会很棒，不会被边角角或设备外壳夹住，或被主屏幕的指示器遮挡。 为了获得最佳效果，请使用标准的系统界面元素和 AutoLayout 构建界面。 所有 App 都应遵循 UIKit 定义的安全区域和布局边距，因为这些区域可以根据设备和上下文进行适当的填充。 安全区域还可以防止内容覆盖status bar, navigation bar, toolbar, 以及 tab bar.

* 注意 status bar 的高度: status bar 在iPhone X 上比在其他 iPhone上更高。 如果假定你固定 status bar 的高度用于将内容定位在 status bar 的下方，那么现在必须更新你的的 App，才能根据用户的设备动态定位内容。 特别需要注意，当后台任务（如录音和位置跟踪）处于活动状态时，iPhone X上的状态栏不会改变高度。

* 重新考虑隐藏 status bar: iPhone X 较之显示高度为4.7“iPhone 的显示屏提供了更多的内容垂直空间，status bar 占据的只是扩展出来的屏幕区域。况且 status bar 更直观的显示用户有用的信息，如果非要隐藏状态栏，那最好用与这些信息同等重要的内容替代。

* 注意长宽比差异: iPhone X 具有不同于4.7“iPhone 的长宽比。因此，全屏4.7英寸iPhone 图形在iPhone X 上全屏显示时出现裁剪或 letterboxing 。同样，全屏iPhone X 图形全屏显示在4.7“iPhone 上时也会被裁剪或 pillarboxing ，因此要确保重要的视觉内容适配这两种尺寸。

![](https://user-gold-cdn.xitu.io/2017/9/30/46d30777de25fd14079a4de0ec107439)


* 避免交互式控件出现在屏幕底部和角落: iPhone X 提供了显示屏底部的滑动手势来访问主屏幕和应用程序切换器的新交互方式，这些手势可能会取消在此区域中实现的自定义手势。 况且屏幕的两个角落过多复杂的交互也不是最佳体验的良好实践。

![](https://user-gold-cdn.xitu.io/2017/9/30/ae04475f4cd2291d0208e210b81615a4)


* 不要遮挡或者特别修饰显示特性来引起用户注意: 请勿尝试隐藏设备的圆角、传感器外壳，或者通过在屏幕顶部和底部放置控件来访问主屏幕的引导。也要特别注意不要试图使用像括号，边框或各种符号等视觉修饰这些特殊区域。

* 为了轻松访问主屏幕允许自动隐藏指示器: 当开启自动隐藏时，如果用户离开屏幕几秒钟，指示器将消失。 当用户再次触摸屏幕时，它会重新出现。 这种行为应该只能用于提升观看体验，如播放视频或照片幻灯片。


----

#### 色彩

iPhone X 的显示器支持 P3 色彩空间，它可以产生比 sRGB 更丰富，更饱和的颜色。

![](https://user-gold-cdn.xitu.io/2017/9/30/39bb14b0ab4c2850a287b0d6f0eb4846)

可以使用 wide color 来增强视觉体验。 它可以让照片和视频更加逼真生动。

更多内容可以参考官网Color management（https://developer.apple.com/ios/human-interface-guidelines/visual-design/color/#color-management）


----

#### 手势

想必大家都在发布会上看到了，iPhone X 上的显示屏可以使用屏幕边缘手势来访问主屏幕，应用程序切换器，通知中心和控制中心。适应这个新变化的同时，对于开发者要特别注意： 避免干扰系统范围的屏幕边缘手势:用户依赖这些手势在每个 App 中操作，所以在极少数情况下，比如游戏这种强调沉浸式体验的 App 可能需要自定义的屏幕边缘手势，优先级高于系统的手势。 这种行为（称为边缘保护）应该谨慎使用，因为它使得用户难以访问系统级的操作。

更多内容参考官网Gestures

（https://developer.apple.com/ios/human-interface-guidelines/user-interaction/gestures/）


----

##### 补充的注意事项


1. 认证方法准确:iPhone X 支持 Face ID进行身份验证。 如果你的 App 集成了 Apple Pay 或其他系统身份验证功能，请务必注意不要在 iPhone X 上引用 Touch ID。同样地，也请确保不要在支持Touch ID 的设备上引用 Face ID。

更详细的内容请参考Authentication(https://developer.apple.com/ios/human-interface-guidelines/user-interaction/authentication/) 

2. 不要重复增加系统提供的键盘功能:在 iPhone X上，即使使用自定义键盘，Emoji / Globe 按钮和 Dictation 按钮也自动显示在键盘的下方。 你的 App 不能影响这些按钮，因此避免在键盘中重复增加这些按钮造成混乱。

3. 由于 iPhone X的屏幕比例发生变化，对于长期靠“等比缩放”完成适配的H5活动页而言也有不小的影响，需要对页面结构进行适当微调。
 
![](https://user-gold-cdn.xitu.io/2018/2/7/1616f5e51321f88d?w=516&h=301&f=png&s=302459)

更详细内容请参阅Custom-keyboards(https://developer.apple.com/ios/human-interface-guidelines/extensions/custom-keyboards/) 

----

### 判断 iPhone X 机型 (Swift)

如何判断当前的设备是 iPhone X 呢？有好几种办法，可以考虑取得「iPhone 10,1」这样的 Module Name 来判断，也可以用屏幕分辨率的形式来判断。我觉得要用屏幕分辨率的方式来做，因为这是目前为止最简单也最不容易出错的。因为 iPhone X 只有一种分辨率，那就是 812pt x 375pt (@3x），且没有任何其他设备用了一样的分辨率，特别是高度。

于是写了一个基于 UIDevice 的扩展（或者其他任意方法也行）：

```Swift
extension UIDevice {
       public func isX() -> Bool {
               if UIScreen.main.bounds.height == 812 {
                       return true
               }
               return false
       }
}
```

在代码中，就可以用 UIDevice.current.isX() 来判断是不是跑在 iPhone X 机型上，然后做一些或不做一些特殊的 Hack 了。


当然如果你习惯用三方库，也可以尝试“DeviceKit”

```Swift
let device = Device()
print(device)     // prints, for example, "iPhone X"
if device == .iPhoneX {
 // Do something
} else {
 // Do something else
}
```


----

### 代码适配部分

当我们能够判断出设备型号就可以配合系统版本进行适配了


#### 适配 UITableView 组件

```Swift
if (@available(iOS 11.0, *)) {
   self.contentInsetAdjustmentBehavior = .never
   self.estimatedRowHeight = 0
   self.estimatedSectionHeaderHeight = 0
   self.estimatedSectionFooterHeight = 0
} else {
   // Fallback on earlier versions
}
```


#### 适配 UIScrollView 组件

```Swift
if (@available(iOS 11.0, *)) {
   scrollView?.contentInsetAdjustmentBehavior = .never
} else {
   // Fallback on earlier versions
}
```


#### UITableView中的sectionHeader或者Footer显示不正常

还有的发现某些界面tableView的sectionHeader、sectionFooter高度与设置不符的问题，在iOS11中如果不实现-tableView: viewForHeaderInSection:和-tableView: viewForFooterInSection:，则-tableView: heightForHeaderInSection:和- tableView: heightForFooterInSection:不会被调用，导致它们都变成了默认高度，这是因为tableView在iOS11默认使用Self-Sizing，tableView的estimatedRowHeight、estimatedSectionHeaderHeight、estimatedSectionFooterHeight三个高度估算属性由默认的0变成了UITableViewAutomaticDimension，解决办法简单粗暴，就是实现对应方法或把这三个属性设为0。

```Swift
if #available(iOS 11.0, *) {
   tableView.estimatedRowHeight = 0;
   tableView.estimatedSectionHeaderHeight = 0;
   tableView.estimatedSectionFooterHeight = 0;
} else {
   automaticallyAdjustsScrollViewInsets = false;
};
```


#### 适配网页加载不全下面有白边

```Swift
if #available(iOS 11.0, *) {
   webView.scrollView.contentInsetAdjustmentBehavior = .never
} else {
};
```


#### 适配iPhoneX不能铺满屏的问题

<1>给Brand Assets添加一张1125*2436大小的图片

打开Assets.xcassets文件夹，找到Brand Assets

右键Show in Finder

添加一张1125*2436大小的图片


<2>修改Contents.json文件,添加如下内容

```
{
"extent" : "full-screen",
"idiom" : "iphone",
"subtype" : "2436h",
"filename" : "1125_2436.png”,
"minimum-system-version" : "11.0",
"orientation" : "portrait",
"scale" : "3x"
}
```


<3>使用 LaunchScreen.storyboard 设置启动图

使用 LaunchScreen.storyboard 文件将简单视图约束定位，实现各种尺寸的自适应。


#### 适配iPhoneX

```Swift
//适配iPhoneX
let LL_iPhoneX = (kScreenW == Double(375.0) && kScreenH == Double(812.0) ?true:false)
let kNavibarH = LL_iPhoneX ? Double(88.0) : Double(64.0)
let kTabbarH = LL_iPhoneX ? Double(49.0+34.0) : Double(49.0)
let kStatusbarH = LL_iPhoneX ? Double(44.0) : Double(20.0)
```


#### Xcode9 打包注意事项

Xcode9 打包版本只能是8.2及以下版本,或者9.0及更高版本

Xcode9 不支持8.3和8.4版本

Xcode9 新打包要在构建版本的时候加入1024*1024 AppSore icon

Xcode9 沿用了之前的分包设计，可以配置打出多种设备的包文件，用户安装时根据设备不同分别安装不同的API包，减小安装包大小。

![](https://user-gold-cdn.xitu.io/2017/9/30/b5e7a87f32ae4546ac736ac0d02b908b)


#### iOS 11 相册权限变更

iOS11以前：

NSPhotoLibraryUsageDescription：访问相册和存储照片到相册（读写），会出现用户授权。

iOS11之后：

NSPhotoLibraryUsageDescription：无需添加。默认开启访问相册权限（读），无需用户授权。

NSPhotoLibraryAddUsageDescription： 添加内容到相册。（写），会出现用户授权。