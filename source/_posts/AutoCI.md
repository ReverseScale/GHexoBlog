---
title: 移动端自动化 CI Review 解决方案
layout: post
date: 2019-01-26 12:55:38
comments: true
categories:
	- Summary
keywords: CI
tags:
	- Script
description: 
---

介绍一下通过 CodeBeat 对移动端代码质量的自动化管理，顺带提及一下，第三方持续集成环境 Travis CI 的基本使用~                                                                                                           
<!-- more -->

之前一篇文章介绍如何通过 Jenkins + Fastlane 脚本自动打包发布测试包。

*  [Jenkins + Fastlane 自动打包脚本](https://reversescale.github.io/2017/06/20/AutoBuildScript/)

这次主要介绍通过 CodeBeat 对移动端代码质量的自动化管理，顺带提及一下，第三方持续集成环境 Travis CI 的基本使用。

![kEed4f.png](https://s2.ax1x.com/2019/01/23/kEed4f.png)


█◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢█

## 🚠 第三方 CI 时代

> [🚠 第三方 CI 时代] Travis CI 是一个专门为开源项目打造的持续集成环境（付费可以支持私有项目），目前已经支持绝大部分主流语言，它的配置文件采用 yaml 格式，简洁清新独树一帜。]

### 持续集成环境能做什么？

![kEML90.png](https://s2.ax1x.com/2019/01/23/kEML90.png)

持续集成是涵盖代码变更后对项目进行自动运行、测试及反馈运行结果的服务。通过自动化的监管，以最大程度确保代码变更符合开发预期。

不仅如此，一个完善的持续集成环境还能够清晰细化开发进度，优化迭代周期，提高项目质量。

但是搭建持续集成环境及配套服务通常需要涉及多方面知识，而且后期维护也需要时间成本。

### Travis CI 服务

![kEKkI1.png](https://s2.ax1x.com/2019/01/23/kEKkI1.png)

Travis CI 提供的是持续集成服务（Continuous Integration，简称 CI）。它绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。

![kElwee.png](https://s2.ax1x.com/2019/01/23/kElwee.png)

而且 Travis 能与 GitHub 的很好的结合起来，为我们提供友好的开发体验。

[![kElTWq.md.png](https://s2.ax1x.com/2019/01/23/kElTWq.md.png)](https://imgchr.com/i/kElTWq)

### 开始集成

#### 添加 Travis-CI 配置文件

在项目根目录下新建 .travis.yml 文件，配置信息如下：

> 在 macOS 下，以 `.` 开头的文件默认为隐藏文件，在 Finder 中使用快捷键 `Command + Shift + .` 可以显示 / 隐藏对隐藏文件的显示。

```
# Xcode 的大版本号，根据实际版本配置，如果是 Swift 项目要注意不同 Xcode 版本对 Swift 的支持
osx_image: xcode10

# 默认设置 objective-c 即可，Swift 项目可以设置 `swift`，但是并不影响运行
language: objective-c

# 环境变量
  # XCODE_PROJECT：Xcode 工程文件路径
  # SCHEME：需要执行的 Scheme
env:
  global:
    - LANG=en_US.UTF-8
    - LC_ALL=en_US.UTF-8
    - XCODE_PROJECT=CollectionIndexTools/CollectionIndexTools.xcodeproj
  matrix:
    - SCHEME="CollectionIndexTools"

# 在 before_install 步骤中，您可以安装项目所需的其他依赖项，如 Ubuntu 包或自定义服务
before_install:
  - gem install xcpretty --no-rdoc --no-ri --no-document --quiet

# 需要执行的构建脚本，使用 CocoaPods 进行库管理可以加一句 `pod install`，build 命令使用 `- xcodebuild -workspace "$XCODE_PROJECT" -scheme "$SCHEME" -configuration Debug -sdk iphonesimulator clean`
script:
  - set -o pipefail
  - xcodebuild -project "$XCODE_PROJECT" -scheme "$SCHEME" -configuration Debug clean build CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO | xcpretty -c

# 构建通过后的自定义命令
  # sleep 3：延时 3 秒，规避 Travis-CI 的历史 Bug
after_success:
  - sleep 3
```

如果是 Swift 项目，需要额外新建 .swift-version 文件配置 Swift 版本号。

```
4.2
```

#### 注册账户

访问 [Travis CI 官网](https://travis-ci.org) 创建账户或者通过 Github 账户直接登录，Travis CI 对 Github 开源项目提供免费服务。

![kEUieP.png](https://s2.ax1x.com/2019/01/23/kEUieP.png)

点击 `+` 进入 GitHub 项目同步设置列表，切换需要同步的项目。

[![kEU1oT.md.png](https://s2.ax1x.com/2019/01/23/kEU1oT.md.png)](https://imgchr.com/i/kEU1oT)

默认情况下 Travis CI 会定时同步开通 Github 绑定的项目代码。

[![kEUROI.md.png](https://s2.ax1x.com/2019/01/23/kEUROI.md.png)](https://imgchr.com/i/kEUROI)

通过 Travis CI 控制面板可以获取到项目的运行结果、log 日志和配置文件。

[![kEaic9.md.png](https://s2.ax1x.com/2019/01/23/kEaic9.md.png)](https://imgchr.com/i/kEaic9)

设置完成后可以在 Github 对应项目的 Setting - Integrations & services 中管理绑定状态。

#### 运行流程

从上面的 `.travis.yml` 配置可知，项目会分为 install、script 两个阶段。

* install 阶段：安装依赖
* script 阶段：运行脚本

1)install 字段用来指定安装脚本。

```
install: ./install-dependencies.sh
```

如果有多个脚本，可以写成下面的形式。

```
install:
  - command1
  - command2
```

如果不需要安装，即跳过安装阶段，就直接设为true。

```
install: true
```

> 在 install 阶段，如果 command1 失败了，整个构建就会停止退出进程。

2)script 字段用来指定构建或测试脚本。

```
script: bundle exec thor build
```

如果有多个脚本，可以写成下面的形式。

```
script:
  - command1
  - command2
```

> script 阶段，如果 command1 失败，command2 会继续执行。但是，整个构建阶段的状态是失败。

如果 command2 需要依赖 command1 成功后才能执行需要如下设置：

```
script: command1 && command2
```

#### 运行状态

Travis 每次运行返回的四种状态：
* passed：运行成功，退出码返回`0`
* canceled：用户取消执行
* errored：before_install、install、before_script 有非零退出码，运行会立即停止
* failed ：script 有非零状态码，会继续运行

█◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢█

## 🤖 自动化 Review 时代

> [🤖 自动化 Review 时代] CodeBeat 是一个免费为开源项目进行代码质量管理的工具（付费可以支持私有项目），无需对原有项目进行任何修改即可获取针对项目的完整质量分析，方便快捷。]

### Codebeat

![kErStx.png](https://s2.ax1x.com/2019/01/23/kErStx.png)

Codebeat 是一个开源的自动代码审查工具，支持Ruby、Go、Swift、Java 代码等开发语言。并且支持与Github，Bitbucket和slack的集成。

和 Travis CI 一样，Codebeat 会在代码仓库发生变更后，执行分析操作，对项目代码质量进行评分，并在完成后刷新项目评级 / 评分的状态或结果。

[![kEr6ER.md.png](https://s2.ax1x.com/2019/01/23/kEr6ER.md.png)](https://imgchr.com/i/kEr6ER)

#### 注册账户
访问 [CodeBeat 官网](https://codebeat.co/)进行注册，具体和 Travis CI 一致，这里不多赘述。

CodeBeat 服务对开源项目是免费的，所以你的私有项目无法享受到免费的持续构建服务。

当然，每月支付 20 美刀成为付费用户后可以解锁无限数量私有库的功能。

登录好账号后点击 `Add Repository` 可以很方便的添加关联项目来开启代码质量管理。

同样关联成功会在 Github 设置页面的 Webhooks 中看到关联信息。

![kE70Ff.png](https://s2.ax1x.com/2019/01/23/kE70Ff.png)

### GPA 指数

点击具体项目进入对应的评分结果页面，可以详细查看 Code Review 结果。

[![kEsCan.md.png](https://s2.ax1x.com/2019/01/23/kEsCan.md.png)](https://imgchr.com/i/kEsCan)

Codebeat 计算全局项目并给出一个 GPA 指数，范围0（最差）~ 4（最佳），大致映射到美国GPA量表。

![kEs3RK.png](https://s2.ax1x.com/2019/01/23/kEs3RK.png)

> GPA 指数的计算规则见[官方文档](https://hub.codebeat.co/docs/gpa-explained)

### Review 

以我的 `AutoAlignButton` 项目为例，审查报告中给 `LabelTagsViewController` 评级为 `D` 😤。

[![kEgplV.md.png](https://s2.ax1x.com/2019/01/23/kEgplV.md.png)](https://imgchr.com/i/kEgplV)

点击进入报告详情，可以看到分为三个分组：
* Complexity 代码层级评估
* Duplication 重复代码评估
* Style 代码风格评估

可以看到我的项目中 `LabelTagsViewController.m` 和 `CollectionTagsViewController.m` 有一个 `- (NSMutableArray *)dataArr` 方法代码重复，虽然很 MMP，但是我们也是可以优化的。

![kEqFFP.png](https://s2.ax1x.com/2019/01/23/kEqFFP.png)

对分析出的代码质量问题进行修改后直接提交，，Webhook 会通知 CodeBeat 进行刷新，以重新进行代码评分。



![kZptr6.png](https://s2.ax1x.com/2019/01/24/kZptr6.png) 

当然自动化的 Review 一定会出现对你高深代码架构的误解，这时候我们可以选择手动干预它的认知🤣。

来来来，让我们对报告结果进行 Mute、Report、Help 操作...

当然还有其他的报告指标等待您的发现🤪。

> 官方文档中关于[功能级指标](https://hub.codebeat.co/docs/software-quality-metrics)的解释


## 😬  联系

* 微信 : WhatsXie
* 邮件 : ReverseScale@iCloud.com
* 博客 : https://reversescale.github.io