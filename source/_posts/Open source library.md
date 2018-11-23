---
title: 使用 CocoaPods 对公有库开源和私有库组件
layout: post
date: 2017-01-22 21:56:27
comments: true
categories:
	- Tips
keywords: CocoaPods
tags:
	- iOS
description: 

---
最近在研究使用 CocoaPods 对 iOS 工程组件化，创建公有 Pod 库和私有 Pod 库方法，为了方便整理和学习就整理了这篇文章~

<!-- more -->
------
![](https://user-gold-cdn.xitu.io/2018/3/21/16247d0c402e7c6a?w=516&h=306&f=png&s=163758)

创建公有 Pod 库或者私有 Pod 库，实际上原理是一样的，都是基于 git 服务和 repo 协议，不一样的是，两者的版本索引查询方式不一样，公有库的 podspec 由 CocoaPods/Specs 管理，而内部私有使用的 pod 库需要自己建立一个仓库来管理 podspec。

## 实用：开源公有库

例子: 我有封装过一个工具CollectionIndexTools，CollectionIndexTools 可以给 Collection 添加一个类似 TableView 右侧的索引条，我想通过 Podfile 中添加 pod 'CollectionIndexToolsLib' 即可使用.

### 注册 CocoaPods 账户信息

想要创建一个开源 pod 库，首先我们需要注册 CocoaPods, 这里使用 trunk 方式，作为一个 iOS 开发人员你一定安装了 CocoaPods，那么只需要在终端执行：

```
pod trunk register 邮箱地址 '用户名' --verbose
```

这里我们一般使用 Github 邮箱和用户名，然后在你的邮箱中会收到确认邮件，在浏览器中点击链接确认即注册成功，成功之后可以终端执行：

```
pod trunk me
```

查看自己的注册信息，以后当你有了自己的开源Pod库，也可以用此方式随时查看自己发布过的Pods：
![](https://user-gold-cdn.xitu.io/2018/3/21/16247c3dcdcd82f4?w=549&h=245&f=png&s=40028)

### 创建共享库文件并上传到公有仓库

共享库需要三个必不可少的部分:

* 1.共享文件夹(文件夹存放着你要共享的内容, 也就是其他人pod得到的文件, .podspec文件中的source_files需要指定此文件路径及文件类型);

* 2.LICENSE文件(默认一般选择MIT);

* 3.库描述文件.podspec(本库的各项信息描述, 需要提交给CocoaPods, pod通过这个文件查找到你共享的库, .podspec文件的格式见第3点).

这一步分两种场景:

场景一：如果你已经有了现成的想要共享的文件,你只需要满足上面三个部分,即可上传到公有仓库即可继续其他的步骤;

场景二：你想要创建一个全新的工程去做自己的共享, 可以使用终端命令:

```
pod lib create CollectionIndexToolsLib
```
需要输入一些模板参数：

![](https://user-gold-cdn.xitu.io/2018/3/21/16247c3dcf41cd99?w=567&h=586&f=png&s=73469)

Cocoapods 会自动生成一个模板项目，目录结构：

```
CollectionIndexToolsLib
├── Example                                  #demo APP
│  ├── CollectionIndexToolsLib
│  ├── CollectionIndexToolsLib.xcodeproj
│  ├── CollectionIndexToolsLib.xcworkspace
│  ├── Podfile                              #demo APP 的依赖描述文件
│  ├── Podfile.lock
│  ├── Pods                                  #demo APP 的依赖文件
│  └── Tests
├── LICENSE                              #开源协议 默认MIT
├── Pod                                      #组件的目录
│  ├── Assets                            #资源文件
│  └── Classes                              #类文件
├── PodCollectionIndexToolsLib.podspec          #第三步要创建的podspec文件
└── README.md                                #markdown格式的README
```

### 编辑.podspec文件

以CollectionIndexToolsLib.podspec为例:
```ruby
Pod::Spec.new do |s|
  s.name             = 'CollectionIndexToolsLib'
  s.version          = '0.1.0'
  s.summary          = 'Custom IndexTools similar to TableViews index bar'

  s.description      = <<-DESC
  I believe you must have thought about adding an index like Table View to Collection View. I will give you one today.
                       DESC

  s.homepage         = 'https://github.com/ReverseScale/CollectionIndexToolsLib'
  s.license          = 'MIT'
  s.author           = { 'ReverseScale' => 'reversescale@icloud.com' }
  s.source           = { :git => 'https://github.com/ReverseScale/CollectionIndexToolsLib.git', :tag => s.version.to_s }

  s.ios.deployment_target = '8.0'

  s.source_files = 'CollectionIndexToolsLib/Classes/**/*'
  
  s.requires_arc = true

end
```

理论上前面的设置就可以通过验证，下面是注释参照：
```ruby
Pod::Spec.new do |s|
  s.name            = "PodTestLibrary"    #名称
  s.version          = "0.1.0"            #版本号
  s.summary          = "Just Testing."    #简短介绍，下面是详细介绍
  s.description      = <<-DESC
                      Testing Private Podspec.

                      * Markdown format.
                      * Don't worry about the indent, we strip it!
                      DESC
  s.homepage        = "https://coding.net/u/wtlucky/p/podTestLibrary"                          #主页,这里要填写可以访问到的地址，不然验证不通过
  # s.screenshots    = "www.example.com/screenshots_1", "www.example.com/screenshots_2"          #截图
  s.license          = 'MIT'              #开源协议
  s.author          = { "wtlucky" => "wtlucky@foxmail.com" }                  #作者信息
  s.source          = { :git => "https://coding.net/wtlucky/podTestLibrary.git", :tag => "0.1.0" }      #项目地址，这里不支持ssh的地址，验证不通过，只支持HTTP和HTTPS，最好使用HTTPS
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'                      #多媒体介绍地址

  s.platform    = :ios, '7.0'            #支持的平台及版本
  s.requires_arc = true                  #是否使用ARC，如果指定具体文件，则具体的问题使用ARC

  s.source_files = 'Pod/Classes/**/*'    #代码源文件地址，**/*表示Classes目录及其子目录下所有文件，如果有多个目录下则用逗号分开，如果需要在项目中分组显示，这里也要做相应的设置
  s.resource_bundles = {
    'PodTestLibrary' => ['Pod/Assets/*.png']
  }                                      #资源文件地址

  s.public_header_files = 'Pod/Classes/**/*.h'  #公开头文件地址
  s.frameworks = 'UIKit'                  #所需的framework，多个用逗号隔开
  s.dependency 'AFNetworking', '~> 2.3'  #依赖关系，该项目所依赖的其他库，如果有多个需要填写多个s.dependency
end
```
编写完成后, 我们需要验证.podspec文件的合法性, 这里需要终端cd到.podspec文件所在文件夹, 执行:
忽视警告：--allow-warnings

```
pod lib lint CollectionIndexToolsLib.podspec
```

如有警告或者错误请重新检查你的编写正确性，如果没有问题会出现：

```
-> CollectionIndexToolsLib (0.1.0)

CollectionIndexToolsLib passed validation.
```

### 打tag, 发布一个release版本

一切准备就绪后, 我们需要在你的git仓库里面存在一个与.podspec文件中一致的version, 这里你可以在你的git仓库中的releases一项去手动发布, 也可以在当前文件夹下使用终端命令:

```
git tag -m '🔖:Releasing tags.' '0.1.0'
git push --tag #推送tag到远端仓库
```

成功之后即可在你的 releases 里面看到这个 tag 的版本.

### 发布自己的库描述文件podspec给cocoapods

同样在这个文件夹下, 终端执行:
忽视警告：--allow-warnings

```
pod trunk push CollectionIndexToolsLib.podspec
```

将你的库文件.podspec文件提交到公有的specs上面, 这一步做的操作是验证你的podspec文件是否合法+提交到specs中(等同于fork;commit;push)+将上传的podspec文件转成json格式文件)，成功后会出现Congrats信息。

![](https://user-gold-cdn.xitu.io/2018/3/21/16247c3dcf1156ca?w=566&h=143&f=png&s=23100)

成功上传后等待片刻就可以用查找命令找到你的库：

```
pod search CollectionIndexToolsLib
```

![](https://user-gold-cdn.xitu.io/2018/3/21/16247c3dc90824fe?w=540&h=112&f=png&s=22004)

### 日后维护更新开源库
如果有错误或者需要迭代版本,修改工程文件后推送到远端仓库后, 需要修改podspec中的版本号, 并重新打tag上传, 再进行新一轮的验证和发布.
如果在开发过程中发现某基础组件存在 bug 需要更新 Pod，具体操作步骤如下：

- 修改 podspec 文件中的 s.version;
- 修复 bug 并对项目打 tag，tag 名称和 s.version 一直并 push 到远程仓库。
- 验证 podspec 文件的有效性；
- 推送 podspec 文件到远程仓库；
- 执行 pod search RRCache 验证结果；


## 实用：组件化私有库

组件化的实用之处请参考《移动端 iOS 年终工作总结-纯干货请自备酒水》（https://juejin.im/post/5a934dfa6fb9a0634514d8a9）

私有Pod库和公有Pod库的创建方式没有什么区别, 不一样的是管理他们的 spec repo 不一样.

所以我们需要自己创建一个跟CocoaPods/Specs类似的仓库来管理内部创建的Pod库的podspec文件, 供内部人员更新和依赖使用内部Pod组件库.

私有repo的构建形式有两种, 一种是私有git服务器上面创建，一种是本机创建.

本机创建请参考官方文档:Private Pods,

这里介绍的是在公司内部搭建的git服务器上面创建整个服务的方式.

### 创建一个git仓库用来做内部私有库的Spec Repo

在私有服务器一个仓库,一个用来存放所有共享库的podspec, 这里创建好之后的内部SSH协议地址是:https://gitee.com/WhatsXie/LibComponent.git, 花钱买git的私有仓库或者使用其他免费的第三方git服务(如Bitbucket等)创建的私有仓库给到的http/https地址也一样.终端输入命令:
```
pod repo add LibComponent https://gitee.com/WhatsXie/LibComponent.git
```

将LibComponent添加到本地repo, 添加成功后可以在/.cocoapods/repos/目录下可以看到官方的specs:master和刚刚加入的specs:LibComponent

如果有其他合作人员共同使用这个私有Spec Repo的话在他有对应Git仓库的权限的前提下执行相同的命令添加这个Spec Repo即可.

### 创建私有Pod组件库

继续创建一个私有仓库,用来建立需要共享的内部组件, 以RSGuidePageLib为例:https://gitee.com/WhatsXie/RSGuidePageLib.git 可以创建示例工程, 像创建公有的库一样, 填写自己的podspec文件

```ruby
Pod::Spec.new do |s|

s.name = 'RSGuidePageLib'

s.version = '0.3.0'

s.summary = 'Custom guide page package'

s.description = <<-DESC

Swift implementation of the guide page package, support for multiple pictures and video guide page

DESC

s.homepage = 'https://gitee.com/WhatsXie/RSGuidePageLib.git'

s.license = { :type => 'MIT', :file => 'LICENSE' }

s.author = { 'ReverseScale@icloud.com' => 'reversescale' }

s.source = { :git => 'https://gitee.com/WhatsXie/RSGuidePageLib.git', :tag => s.version.to_s }

s.ios.deployment_target = '8.0'

s.swift_version = '3.2'

s.source_files = 'RSGuidePageLib/Classes/**/*'

s.requires_arc = true
```

值得注意的是:podspec文件中的homepage和source不支持ssh协议地址,所以我们得放入http/https地址.

与公有库的创建方式一样, pod lib lint Category.podspec验证成功之后push到仓库, 然后打tag发布release版本.

### 然后将podspec加入私有Sepc repo中

公有库使用trunk方式将.podspec文件发布到CocoaPods/Specs, 内部的pod组件库则是添加到我们第一步创建的私有Spec repo中去, 在终端执行:
--allow-warnings 忽略警告
--private 私有库

```
pod repo push LibComponent RSGuidePageLib.podspec
```
添加成功之后LibComponent中会包含RSGuidePageLib库的podspec信息, 可以前往~/.cocoapods/repos下的LibComponent文件夹中查看, 同时git服务器中的远端也更新了.

移除私有Repo

```
pod repo remove [name]
```

### 查找和使用内部组件库

执行pod search Category就能查到刚刚创建好的Category库了, 然后在想要使用此组件的工程的Podfile中加入pod 'Category', '~>1.0.1'即可使用内部组件啦！

值得注意的是:必须在Podfile前面需要添加你的私有Spec repo的git地址source, pod install时, 才能在私有repo中查找到私有库, 像这样:

```
# Uncomment the next line to define a global platform for your project

source 'https://github.com/CocoaPods/Specs.git'
source 'https://gitee.com/WhatsXie/LibComponent.git'

# platform :ios, '9.0'
target 'Demo' do

pod 'RSGuidePageLib', '~>0.3.0’

end
```

经过测试, 这种方式可以把你的所有可以拆分出来的组件, 甚至是业务都来使用Pod管理, 这样达到了解耦和单项更新优化。
某些组件不影响老版本的依赖使用, 出现问题修改Podfile中的依赖版本即可随时回滚, 给开发了带来极大的便利。


### 参考链接
* CocoaPods创建公有和私有Pod库方法总结（https://www.aliyun.com/jiaocheng/376300.html）
* CocoaPods Guides(https://guides.cocoapods.org)
* Private Pods(https://guides.cocoapods.org/making/private-cocoapods.html)
* 手把手教你发布代码到CocoaPods(Trunk方式)(http://www.cnblogs.com/wengzilin/p/4742530.html)
* 使用Cocoapods创建私有podspec(http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)
* COCOAPODS创建私有PODS(http://www.cnblogs.com/tufeibo/p/5654268.html)
* CocoaPods 组件化实践 - 私有Pod(https://www.jianshu.com/p/475d6b6d5600)