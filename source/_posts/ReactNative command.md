---
title: ReactNative 开发常用命令行（持续更新）
layout: post
date: 2018-03-29 22:56:27
comments: true
categories:
	- Tips
keywords: React
tags:
	- Tips
description: 

---
记录一些 ReactNative 常用的命令行，包括了例如安装、启动服务、插件管理、热更新等一些日常经常使用的命令，会随着使用的深入不断的补充 ~                                                                                                                        
<!-- more -->
------

![](https://user-gold-cdn.xitu.io/2018/3/29/1626fc2e0b185158?w=673&h=364&f=png&s=99722)

### Node 相关
查看所有安装的node版本信息：
```
nvm list
```
查看更新了的node的版本(可能需要翻墙)：
```
nvm ls-remote 
```
安装node：
```
nvm install v7.4.0
```
设置默认的node版本(这里设置成了7.4.0)，解决有些版本有些兼容性的问题：
```
nvm alias default v7.4.0 
```

---
### React 相关
卸载命令：
```
npm uninstall -g react-native-cli 
```
安装命令：
```
npm install -g react-native-cli 
```
查看某个模块最新发布版本信息(这里查看react-native发布的版本信息)：
```
npm info react-native 
```
升级或者降级react-native的版本并且更新package.json，需要用在react-native项目目录下：
```
npm install --save react-native@0.41.1 
```
新建 react-native 项目并指定版本：
```
react-native init demo --version 0.40.0
```
开启服务：
```
react-native start 
```
运行Android：
```
react-native run-android 
```
运行iOS：
```
react-native run-ios
```
版本查看：
```
react-native --version 
```
项目版本查看：
```
react-native -v 
```
查看react-native的帮助信息：
```
react-native --help 
```

---
### 使用 Cocoapods 管理 ReactNative
Podfile 文件格式：
```
pod 'React', :path => './node_modules/react-native', :subspecs => [
  'Core',
  'RCTText',
  'RCTImage',
  'RCTActionSheet',
  'RCTGeolocation',
  'RCTNetwork',
  'RCTSettings',
  'RCTVibration',
  'RCTWebSocket',
  ]
```
ReactNative 0.42.0 以上版本需在 Podfile 配置 yoga：
```
# 如果你的RN版本 >= 0.42.0，请加入下面这行
pod "yoga", :path => "./node_modules/react-native/ReactCommon/yoga"
```

---
### 开源组件库：
安装最新版本：
```
npm install react-native-tab-navigator --save
```
安装指定版本：
```
npm install --save react-native-tab-navigator@0.4.0
```
react-native 集成组件绑定(ReactNative 0.27以后，自集成RNPM)：
```
react-native link react-native-splash-screen
```
常用开源库：
```
npm install --save react-native-tab-navigator@0.4.0
npm install --save react-native-scrollable-tab-view@0.7.0
npm install --save react-native-check-box@1.0.4
npm install --react-native-easy-toast@1.0.9
npm install --save GitHubTrending@2.0.0
npm install --save react-native-htmlview@0.5.0
npm install --save react-native-popover@0.5.0
npm install --react-native-splash-screen@2.0.0
```

---
### Code-Push 常用命令
Code-Push 推包命令：
```
code-push release-react <appName> <platform> [options]
```
示例：

```
code-push release-react RNAPPGithub ios --t 1.0.2 --dev false --d Staging --des "1.热更新相关设置" --m true
```

Code-Push 线上查看更新：
```
code-push deployment ls RNAPPGithub
```
Code-Push 查看项目Key：
```
code-push deployment ls RNAPPGithub -k
```
Code-Push iOS更新打包方法：
```
react-native bundle --platform ios --entry-file index.js --bundle-output release_ios/main.jsbundle --assets-dest release_ios/ --dev fasle
```

---
### IDE 技巧
《使用VS Code调试React-Native程序》（https://jingyan.baidu.com/article/ad310e80fb13fc1849f49ed1.html）