---
title: 使用 ngrok 快速实现内网穿透
layout: post
date: 2018-08-13 11:56:27
comments: true
categories:
	- Tips
keywords: 内网穿透
tags:
	- Tips
description: 

---
实现内网穿透的工具很多，之所以介绍 ngrok，主要还是因为它使用便捷，不用搭建服务器等等麻烦的工序，适合前段开发过程中，快速评估检测项目~                                                                                                                        

<!-- more -->
------
### Ngrok是什么

ngrok 是一个反向代理，通过在公共的端点和本地运行的 Web 服务器之间建立一个安全的通道。ngrok 可捕获和分析所有通道上的流量，便于后期分析和重放。

而我用来在家里快速访问 Jenkins，偷个懒🤪

![](https://user-gold-cdn.xitu.io/2018/8/13/16531b4060d824c0?w=700&h=438&f=png&s=167025)

### 搭建方法

由于要完成一个网页优化的作业找了很久ngrok的使用方法，都不够简便易行最后终于发现了一个好方法。

#### 下载 MAC 版的 ngrok：https://ngrok.com/download

![](https://user-gold-cdn.xitu.io/2018/8/13/16531b47946c06f4?w=873&h=721&f=png&s=120693)

#### 解压到指定目录：
Safari 浏览器下载 Mac OS X 环境一般直接解压（反正我是自动解压的），将 ngrok 放进项目目录。

![](https://user-gold-cdn.xitu.io/2018/8/13/16531b4d12c6f8ce?w=193&h=211&f=png&s=14935)

#### 进入到 ngrok 所在路径：

```
cd /tmp
```

#### 开启服务

```
./ngrok http localhost:8080
```

会出现如下 ngrok 控制台

![](https://user-gold-cdn.xitu.io/2018/8/13/16531b626fe2b14b?w=570&h=475&f=png&s=85834)

等待 Session Status 状态为 online（变绿），就可以在外网通过 Forwarding 的地址进行连接了。

> 注意：Forwarding地址中的 c33faf1b 不是固定的，在每次开始 ngrok 服务的时候都会变更，想固定？ ngrok:😛要钱。

### 测试一下

例如：

ngrok 穿透前在局域网中访问地址：
http://localhost:8080

ngrok 穿透后局域网 + 公网访问地址：
https://c33faf1b.ngrok.io
