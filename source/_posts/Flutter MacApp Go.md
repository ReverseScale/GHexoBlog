---
title: 使用 Flutter 开发 macOS App（二）
layout: post
date: 2019-04-29 09:29:16
comments: true
categories:
	- Project
keywords: Flutter
tags:
	- Flutter
description: 

---

之前一篇博客介绍了借助 Feather 平台在 MacOS 和 Windows 上运行我们的 Flutter 应用程序，并且可以直接发布在其平台上，但实际使用下来发现 Feather 极不稳定，因此就继续探索别的方法，进而整理了这篇文章~

<!-- more -->
------

👨🏻‍💻 [Github Demo](https://github.com/ReverseScale/FlutterDemo)

使用 Google 框架构建智能手机和台式机的应用程序。

<img width="356" height="116" src="https://s2.ax1x.com/2019/04/28/EQuSAI.png"/>


## 介绍
如果您正在为智能手机开发应用程序，可能您已经听说过新的开发框架，Google 的 Flutter。它是一个框架，允许您使用 Dart 语言（也来自 Google）开发一套代码的跨平台应用程序，并为Android 和 iOS 平台发布它们。

该框架现已达到 v1.4.9 版本，并且自从今年 5 月发布的 beta 3 版本以来，谷歌已经宣布准备好用于生产。此外，还可以使用 Flutter 为桌面环境（Windows，macOS 和 Linux）构建应用程序，这是本文将要讨论的主题。

目前通过两个仍在开发中的项目提供对桌面平台的支持。其中一个甚至是谷歌开发的，但有迹象表明该公司不支持它。可以在以下 Github 存储库中找到这些项目：

* 【官方】Google 开源的 Flutter 嵌入桌面的 API 实现：[flutter-desktop-embedding](https://github.com/google/flutter-desktop-embedding)
* 【推荐】Go 语言实现工具：[hover](https://github.com/go-flutter-desktop/hover) 

这两个项目都归类为 Custom Flutter Engine Embedders，也就是说，它们是 Flutter API 的实现，因此在此框架中开发的项目可以在 Flutter 项目（Android，Fuchsia 和 iOS）正式支持的操作系统之外运行。

<img width="356" height="116" src="https://s2.ax1x.com/2019/04/28/EQc1zV.png"/>


他们通过 GLFW 库使用 OpenGL（除了 Google 项目的 macOS 版本），它提供了一个 API，用于在桌面平台上创建窗口以及处理键盘和鼠标输入等功能。因此，正在开发的应用程序运行的平台必须安装 OpenGL 驱动程序。

## 安装 Flutter

要安装 Flutter，请根据您的操作系统，按照官方安装页面上列出的步骤进行操作。[单击此处](https://flutter.io/get-started/install/)访问安装页面。将需要在本文后面运行项目。不要忘记在您的 PATH 环境变量中包含 Flutter。

> 也可以查看我之前的[博客文章](https://github.com/ReverseScale/FlutterDemo)进行安装。

### 方法一：Flutter 的桌面嵌入（Google）
该项目目前存在一些限制，这些限制在存储库文档的“[注意事项](https://github.com/google/flutter-desktop-embedding/blob/master/README.md#caveats)”部分中指出。

与将在本文后面介绍的第二个项目不同，该项目没有二进制文件，可以快速运行示例项目或设置自己的 Flutter 项目以在桌面平台上运行它。

您需要在操作系统中编译此项目的源代码，然后将生成的库包含到将成为应用程序一部分的可执行文件中。将来，此过程将简化，如当前存储库文档中所述。

要通过此项目编译项目源代码并使用 Flutter 创建桌面应用程序，请按照[此处](https://github.com/google/flutter-desktop-embedding/blob/master/README.md)的说明进行操作。

### 方法二：Go Flutter 桌面嵌入器（Drakirus）
第二个项目是通过 Google 的 Go（Golang）编程语言开发的。

在项目存储库中有下载版本，您可以通过已编译的可执行文件测试桌面应用程序的示例。此外，创建新的 Flutter 桌面应用程序非常简单。

以下各项说明如何在项目存储库中运行现有示例项目，以及如何运行在 Flutter 中开发的自己的项目，现在作为桌面应用程序。

#### 执行示例项目：
1.在[此处访问](https://github.com/Drakirus/go-flutter-desktop-embedder/releases)项目版本。
2.根据您的平台，在 Linux，Windows 和 MacOS 的 vpr.2.1-alpha 版本的预建二进制文件示例部分中下载示例文件。
3.将下载的文件解压缩到您选择的位置。
4.在未压缩文件集的根目录中运行stocks可执行文件（macOS 和 Linux）或 socks.exe（Windows）。它将像您使用的任何其他操作系统应用程序一样运行（下图中的示例）。

#### 效果演示

在 Windows 10 桌面环境中运行的 Flutter 中开发的示例应用程序：
![Windows 10 下运行截图](https://s2.ax1x.com/2019/04/28/EQ293t.png)

在 Flutter 中开发的示例应用程序在 macOS Mojave 桌面环境中运行：
![macOS Mojave 下运行截图](https://s2.ax1x.com/2019/04/28/EQ2kDS.png)

在 Flutter 中开发的示例应用程序在 Ubuntu Budgie 18.04 桌面环境中运行：
![Ubuntu Budgie 下运行截图](https://s2.ax1x.com/2019/04/28/EQ2ZNj.png)

> 注意：在以下指南中，桌面应用程序的 macOS 版本以黑屏开始，但应用程序在最大化屏幕后正常加载。一旦我找到解决这个问题的方法，我就会更新这篇文章。

### 操作步骤
#### 使用预编译文件

1. 在此处下载适用于您的操作系统的预编译文件。
2. 将内容解压缩到您选择的目录中。
3. 通过根据环境和应用程序的信息更改参数的值来编辑模板目录的 config.json 文件。 FlutterPath： Flutter 安装的目录。 FlutterProjectPath：您的项目（在Flutter中开发）目录。IconPath：桌面应用程序图标文件的路径（可以保留默认值）。ScreenHeight：桌面应用程序的初始屏幕高度。ScreenWidth：桌面应用程序的初始屏幕宽度。如果您使用的是 Windows，请在目录路径中使用双反斜杠（\\）而不是简单的反斜杠（\）。
4. 运行扑桌面模板的可执行文件，在 MacOS 或 Linux，或扑桌面 -template.exe 可执行文件，在 Windows 和应用程序将启动。它将像您使用的任何其他操作系统应用程序一样运行。

#### 编译 Go 桌面项目

1. 下载并通过其下载页面下载安装围棋编程语言在这里（用于安装并提供了下载页面上的 PATH 环境变量的工具配置说明）。
2. 如果您的计算机上没有安装 GCC，则需要安装它以从 Go 中的桌面项目编译一些代码文件。对于 Windows，我建议使用 tdm64-gcc。对于 macOS，使用 Xcode Command Line Tools 提供的那个。对于 Linux，可能您已经在系统上安装了 GCC。
3. 如果您没有在系统中安装 Git，请在此处下载并安装 Git。
4. 通常通过 `flutter create` 命令或 IDE 创建Flutter 项目。
5. 在 Flutter 项目的 main.dart 文件中包含以下导入：`import 'package:flutter/foundation.dart' show debugDefaultTargetPlatformOverride`;
6. 在 Flutter 项目的 main.dart 文件的 main 方法开头（在调用 runApp 方法之前）中包含以下代码（此代码是必需的，因为 Flutter 项目不正式支持桌面平台）： `debugDefaultTargetPlatformOverride = TargetPlatform.fuchsia`;
7. 在终端中，flutter build bundle 在 Flutter 项目的目录中运行该命令（将使用将在以下步骤中下载和配置的模板在 Flutter 中运行应用程序所需的文件创建目录）。
8. 同样在终端中，运行 `go get -u github.com/go-gl/glfw/v3.2/glfw` 命令以下载 Go 语言的 GLFW 库（如果出现问题，请查看本文末尾的“ 故障排除”项）。在安装之前，您必须验证此链接中是否包含所有依赖项。
9. 在 Go here 下载桌面项目（当前正在运行并支持 Windows，macOS 和 Linux 的版本）。
10. 将下载文件的内容解压缩到操作系统上的任何目录。
11. 将从 `go-flutter-desktop-embedder-0.2.1-alpha` 中提取的目录重命名为 `go-flutter-desktop-embedder`。
12. 将提取的（现在重命名的）目录复制到 src\github.com\Drakirus 目录（或 Windows 上的 src/github.com/Drakirus），该目录必须在 GOPATH 中创建（GOPATH 是一个环境变量，指示安装的位置为Go语言下载的库;在 Windows 和 macOS 上，通常是用户个人文件夹中的 go 目录）。
13. 下载或复制需要此演练模板库在[这里](https://github.com/marceloneppel/flutter-desktop-template)（它会被用来制造更容易，而不必在 Go 语言编程运行应用程序）。如果您已选择下载文件而不是克隆存储库，请将下载的文件解压缩到您首选的目录中。
14. 使用 [Linux](https://storage.googleapis.com/flutter_infra/flutter/74625aed323d04f2add0410e84038d250f51b616/linux-x64/linux-x64-embedder)， 链接下载 Flutter 库（模板操作所需）。 [Windows](https://storage.googleapis.com/flutter_infra/flutter/74625aed323d04f2add0410e84038d250f51b616/windows-x64/windows-x64-embedder.zip) 上的 （替换粗体文本，这是测试版 v0.9.4，使用与您在项目中使用的 Flutter 版本对应，通过查看文件可以获得什么在 Flutter 安装路径中的 bin\internal\engine.version）。
15. 解压缩下载的文件并将 Linux 上的 libflutter_engine.so 文件，FlutterEmbedder.framework（FlutterEmbedder.framework.zip 文件的内容，文件名为FlutterEmbedder.framework，目录名为FlutterEmbedder.framework）复制到 macOS 上，或者将 Windows 上的 flutter_engine.dll 复制到目录模板中，main.go 文件所在的位置，以及之前复制到 GOPATH 的 go-flutter-desktop-embedder 目录。
16. 在终端上，转到 go-flutter-desktop-embedder 目录并 `export CGO_LDFLAGS="-L${PWD}"` 在 Linux 上运行命令，`set CGO_LDFLAGS=-L%cd%` 在 Windows 上运行命令或 `export CGO_LDFLAGS="-F${PWD} -Wl,-rpath,@executable_path"`在 macOS 上运行命令。保持终端开放。
17. 在同一终端上，`go install` 在 go-flutter-desktop-embedder 目录中运行命令。仍然保持终端开放。
18. 通过根据环境和应用程序的信息更改参数的值来编辑模板目录的 config.json 文件。FlutterPath： Flutter 安装的目录。FlutterProjectPath：您的项目（在 Flutter 中开发）目录。IconPath：桌面应用程序图标文件的路径（可以保留默认值）。ScreenHeight：桌面应用程序的初始屏幕高度。ScreenWidth：桌面应用程序的初始屏幕宽度。如果您使用的是 Windows，请在目录路径中使用双反斜杠（\\）而不是简单的反斜杠（\）。
19. 仍然在同一个终端中，重复步骤 16 并执行 `go build` 命令，但现在在解压缩模板的目录中，main.go 文件所在的位置。在 Windows 上，使用该 `go build -ldflags -H=windowsgui` 命令而不是 `go build` 在运行时终端不与应用程序一起显示。在 macOS 上，可能需要根据平台标准打包目录，以便终端不与应用程序一起显示。
20. 可执行文件将被创建（这将具有相同的名称作为项目目录，扑桌面模板在 Linux 和 Mac 或 Hover，桌面 template.exe 在 Windows 上，如果模板目录是 Hover 桌面模板）。
21.运行创建的可执行文件，应用程序将启动。它将像您使用的任何其他操作系统应用程序一样运行。

* 当对 Flutter 项目进行任何更改时，请重复步骤 7（在项目目录中运行 `flutter build bundle` 命令）。
* 项目中使用的 Flutter 版本应与步骤 14 中下载的版本相同。
* 每当您对模板的 config.json 文件进行任何更改时，请重复步骤 16 和 19。

### 打包应用程序以进行分发
要分发在 Flutter 中开发的桌面应用程序，以便它可以在未安装 Flutter 的计算机上运行，请在运行上述指南之后按照以下步骤运行您自己的项目。

1. 确保使用的是当前版本的模板，版本为 v1.1.0。
2. 创建一个目录来存储运行应用程序所需的所有文件。在以下步骤中，此目录将被称为应用程序目录。
3. 在应用程序目录中包含 flutter-desktop-template 可执行文件（在 Linux 和 macOS 上）或 flutter-desktop-template.exe（在 Windows 上）。根据应用程序的名称重命名可执行文件。
4. 还包括应用程序目录中的 Flutter 库。这个库是 libflutter_engine.so Linux 上的文件时，FlutterEmbedder.framework 在 MacOS 目录，或 flutter_engine.dll 上的 Windows 文件。
5. 将 config.json 文件复制到应用程序目录。通过将 FlutterPath 和 FlutterProjectPath 的值更改为空白来编辑此文件（将值保留为空）。
6. 在应用程序目录中包含带有图标图像的 assets 目录。如果已选择其他映像，请将该映像复制到应用程序目录，并在 config.json 文件的 IconPath 值中更正其路径。
7. 将 flutter_assets 目录复制到应用程序目录，该目录位于 Flutter 项目的构建目录中。
8. 最后，复制位于 bin\cache\artifacts\engine\windows-x64 (在 Windows 平台), bin/cache/artifacts/engine/linux-x64 (在 Linux 平台) 或者 bin/cache/artifacts/engine/darwin-x64 (在 macOS 平台) 上的 icudtl.dat 文件从您的 Flutter 安装到应用程序目录的路径。
9. 在 macOS 上，可能需要根据平台标准打包目录，以便终端不与应用程序一起显示。

现在，您可以分发应用程序并将其安装在环境中，而无需同时安装 Flutter。

## 结论
本文中介绍的两个项目允许使用 Flutter 框架开发的应用程序在桌面环境中运行。
* 第一个是谷歌自己开发的，但不提供支持。
* 第二个方法既可以更轻松地运行基于 Flutter 和桌面应用程序构建的应用程序，也可以更加熟悉使用谷歌制作的 Go（Golang）编程语言的开发人员。

对于应该为桌面和移动环境开发应用程序以便可以重用代码的情况，本文中介绍的两个项目都是很好的解决方案。

翻译整理自：https://medium.com/flutter-community/flutter-from-mobile-to-desktop-93635e8de64e
