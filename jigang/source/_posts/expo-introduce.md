---
title: Expo 介绍
date: 2018-02-08 11:38:08
tags:
- how-to-become-a-react-native-developer
- JavaScript
- 'React Native'
- Expo
categories:
- Expo

---


![](https://d30j33t1r58ioz.cloudfront.net/static/images/spots/docs.gif?f560592076320441267576e0dae10968)

- [介绍](#)
[Expo](http://expo.io/)是一套工具、库和服务，让你可以通过编写JavaScript来构建本地的iOS和Android应用程序。

- 使用Expo工作
详细介绍如何使用Expo开发环境和工具的各个方面。
如果你对Expo完全陌生，一定要确保你已经[安装好了工具](/2018/02/06/expo-installation/)并[四处看看](/2018/02/06/expo-xde-tour/)。在此之后，您可能想要阅读并[运行](/2018/02/07/expo-up-and-running/)以创建您的第一个项目。

<!-- more -->

# 快速入门

## 介绍

Expo应用程序是带有[Expo SDK](https://docs.expo.io/versions/latest/sdk/index.html#expo-sdk)的本地应用程序。SDK是一个native-and-JS库，它提供了对设备的系统功能的访问(像照相机、联系人、本地存储和其他硬件)。这意味着您不需要使用Xcode或Android Studio，也不需要编写任何本地代码，而且它也使您的纯js项目非常便于移植，因为它可以在包含世博会SDK的任何本地环境中运行。

Expo还提供了UI组件来处理各种各样的用例，这些用例几乎所有的应用都能覆盖，但没有被融入到React Native中，比如图标、模糊视图等等。

最后，Expo SDK提供了对服务的访问，这些服务通常是一种很难管理的服务，但几乎每个应用都需要它。
其中最受欢迎的是:Expo可以为你管理你的资产，它可以帮你处理推送通知，它还可以构建原生二进制文件，这些二进制文件已经准备好部署到app store中。

## 考虑使用Expo吗?

- 如果你想了解一下Expo提供了什么，你可能想要熟悉一下[Expo项目的生命周期]()，它描述了你是如何进入一个产品的iOS和Android应用的场景。
- 为了进一步解释，检查[经常被问到的问题]()也很好。

## 准备好开始了吗?

- 首先[安装]()工具，看看周围的情况。
- 通过[跟踪和运行指南]()来完成你的第一个项目。
- 如果您还不熟悉React和React Native，那么您可以使用[React Native Express](http://www.reactnativeexpress.com/)来引导您的知识。

# 安装

开发应用程序你需要有两种工具——一个桌面开发工具和一个移动客户端来打开你的应用程序。

## 桌面开发工具:XDE

XDE代表Expo开发环境。它是一个独立的桌面应用程序，包含了所有需要启动的依赖项。

为[macOS](https://xde-updates.exponentjs.com/download/mac)、[Windows(64位)](https://xde-updates.exponentjs.com/download/win32)或[Linux](https://xde-updates.exponentjs.com/download/linux-x86_64)下载最新版本的XDE。

在Linux上，打开chmod a+x xde*.AppImage 和 ./xde*.AppImage。

## 移动客户端:iOS和Android Expo

Expo客户端就像一个用Expo搭建的应用的浏览器。当你在你的项目上启动XDE时，它会为你生成一个独特的开发URL，你可以从iOS或Android的世博会客户端访问它，无论是在真实的设备上，还是在模拟器上。

### 在你的设备

[下载Android从Play Store](https://play.google.com/store/apps/details?id=host.exp.exponent) 或 [iOS从App Store](https://itunes.com/apps/exponent)

> 所需的Android和iOS版本:最低的Android版本支持是Android 4.4，最小的iOS版本是iOS 9.0。

您不需要在模拟器上手动安装世博会客户端，因为XDE会自动完成这个操作。请参阅本指南的下一部分。

### iOS模拟器

通过苹果App Store安装Xcode。需要一段时间，去打个盹。接下来，打开Xcode，进入首选项并单击组件选项卡，从列表中安装模拟器。

一旦模拟器打开，你在XDE中打开了一个项目，你可以在XDE上打开iOS模拟器，它会将Expo客户端安装到模拟器上，并在里面打开你的应用。

> **不工作吗?** 有时XDE会自动安装世博客户端，通常是由于你的环境或Xcode工具链上的小差异造成的。如果您需要手动在模拟器上安装世博会客户端，您可以遵循以下步骤:
- 下载最新的模拟器构建。
- 提取存档的内容。你应该得到一个像指数.x.xxx这样的目录。
- 确保模拟器正在运行。
- 在终端中，运行 `xcrun simctl install booted [path to extracted directory]`

### Android模拟器

[下载Genymotion](https://www.genymotion.com/fun-zone/)(免费版)，并遵循[Genymotion安装指南](https://docs.genymotion.com/Content/01_Get_Started/Installation.htm)。一旦你安装了Genymotion，创建了一个虚拟设备——我们推荐Nexus 5，Android版本就会由你来决定。在准备好时启动虚拟设备。

一旦模拟器打开，你在XDE中打开了一个项目，你就可以在XDE的Android平台上发布开放项目，它会将Expo客户端安装到模拟器上，并在里面打开你的应用程序。如果你遇到任何问题，请遵循我们的[Genymotion指南](https://docs.expo.io/versions/latest/guides/genymotion.html#genymotion)。

## Node.js

要想开始Expo，你不一定需要安装了Node.js，但一旦你开始构建东西，你就会想要拥有它。[下载最新版本的Node.js](https://nodejs.org/en/)。

## Watchman

一些macOS用户会遇到问题，如果他们没有安装在他们的机器上，那么我们建议您安装Watchman。当用户改变时，他们会观察文件和记录，然后触发相应的动作，并在内部进行反应。[下载并安装Watchman](https://facebook.github.io/watchman/docs/install.html)。

# XDE之旅

## 登录屏幕

当你第一次打开XDE时，你会在屏幕上看到这个标志。如果你已经有了一个账户，请继续登录。如果你不这样做，要么和Github签约，要么注册一个账户。

![](https://docs.expo.io/static/3e6b623e795fdb7aa26ac4ea708ceb03-d94ef.png)

## 主屏幕

成功,你签署!在这个屏幕上，您可能想要创建一个新项目，或者打开一个现有的项目。为了方便，我们列出了一些最近开放的项目。

![](https://docs.expo.io/static/3b06fec065fc43a319c50f3bf6bdfe29-d94ef.png)

## 项目对话框

点击项目，你就会看到你能做的一切。当然，你不能关闭一个项目或者在finder中显示它，因为你还没有打开一个项目。

![](https://docs.expo.io/static/54eabb01a078ee0a37fd20f2c4e12709-d94ef.png)

## 退出,如果你想要

在任何时候，你都可以点击右上方的用户名并退出。或注销。谁能真正同意这一看法呢?

![](https://docs.expo.io/static/aa6d11a7cb8249b27c608177824c9c52-d94ef.png)

## 项目屏幕

所以我们已经开放了一个新项目。左边的窗格是React包装器，您可以在它的[启动和运行]()中了解更多关于[Expo的工作]()。右边的面板是用于设备日志的，您可以在[查看日志]()中看到更多的内容。

![](https://docs.expo.io/static/ba5e8f002e9c4763c6c47d903e6a7952-d94ef.png)

## 分享

向任何有互联网连接的人发送你的应用程序的链接。如果你的设备没有连接到你的电脑上，这也很有用。

![](https://docs.expo.io/static/e0769f3e3d13565ff8d0a38c1a544e98-d94ef.png)

## 设备上打开

设备按钮可以让你在设备或模拟器上快速打开你的应用。更多的阅读[启动和运行]()。

![](https://docs.expo.io/static/0af285921a2776b12f3d758cf811b529-d94ef.png)

## 开发模式

您通常希望在开发模式中使用您的项目。这使它运行得稍微慢一些，因为它增加了很多运行时验证的代码来警告您潜在的问题，但是它也提供了实时重载、热重载、远程调试和元素检查器的访问权。如果你想测试与性能相关的任何东西，禁用开发模式并重新加载你的应用。

![](https://docs.expo.io/static/a69ee61b65016412b4e2b91249cc524d-d94ef.png)

## 项目对话框(与项目打开)

除了主屏幕提供的选项外，我们还提供了一些快捷方式，比如在finder中显示项目目录。

![](https://docs.expo.io/static/ba5e8f002e9c4763c6c47d903e6a7952-d94ef.png)

## 发布

当你在你的项目上工作的时候，它会得到一个临时的URL，从你的电脑上提供。当你准备好与他人分享这个项目时，你可以 **发布** 这个项目来获得一个永久的URL(类似expo.io/@your-username/your-app-slug)，任何人都可以和Expo的客户一起开放。

当您点击XDE中的Publish按钮时，您将被要求确认您希望您的项目对公众开放。XDE需要花一些时间来生成你的迷你JS包，并将你的资产上传到我们的服务器上，一旦完成，打印出你的应用发布的URL。你可以读到更多的细节，关于在[Expo如何运作]()和[发布指南]()方面，发布是如何运作的。

![](https://docs.expo.io/static/cea8b37eddf1a301c7be3677971231c8-d94ef.png)

# 项目生命周期

Expo让你很容易开始编写应用程序，但Expo也可以将你的项目带向生产。以下是您可能在此过程中使用的工具和服务的概述。

本指南旨在对Expo提供的内容进行高层次的解释。出于好奇，这些主题的技术实现将在[这里](https://docs.expo.io/versions/latest/guides/how-expo-works.html)更详细地介绍。

![](https://docs.expo.io/static/9b488f03170d123590d9ebc0fbf67f64-cca32.png)

## 创建一个世博会项目

您可以使用我们的桌面工具和文本编辑器创建一个新的Expo项目。查看[启动并运行]()一个快速指南来创建一个项目，在一个设备上运行它，并进行更改。

Expo应用程序是本地应用程序。最快的方法是使用[启动并运行]()指南，但是你也可以转换一个已有的React Native应用，或者使用一个由create-react-native-app生成的项目。

## 本地开发

当你在一个Expo项目上工作时，我们会从你的本地电脑上为你的项目提供一个实例。如果你关闭了这个项目或者关闭了你的电脑，你的开发项目就会停止。

在这段时间里，你可以使用一个叫做[Expo Client]()的预构建的iOS/Android应用来测试你的项目。它要求您的计算机为您的项目的本地副本(通过localhost、LAN或隧道)，下载它，并运行它。您可以利用各种开发工具，如[调试]()、[流设备日志]()、检查元素、热模块重新加载等等。

## 发布您的项目

如果你点击XDE的 **发布** 按钮，我们就会将你的应用的一个迷你版复制到我们的CDN上，并给你一个可分享的网址，你的地址，`expo.io/@your-username/your-app-slug`。

你可以立即与任何拥有Expo Client应用程序的人分享这个链接，[阅读更多关于这里的发布]()。

## 更新应用程序

您可以在不影响用户的情况下继续在本地进行更改。任何时候你发布应用程序的修改，你的新版本就会立即被任何有链接的人使用。

我们经常发布对[Expo SDK]()的更新。每次更新包括说明如何升级你的项目。如果您决定更新我们的SDK的更新版本，那么旧版本的副本将继续正常工作。用户将下载他们的客户端支持的最新版本。

## 部署到苹果应用商店和Google Play商店

当你准备在苹果应用商店和Google Play商店正式发布你的应用时，世博会就可以为你带来部署了`.ipa`和`.apk`，准备提交给苹果和谷歌。我们在我们的服务器上生成它们，所以你仍然不需要任何苹果或谷歌软件。参见关于分发应用程序的文档。

## 更改本地代码

你可以在app Store和Play商店中使用你的应用程序，同时只编写JS。然而，如果你遇到了Expo所没有的特殊的高级需求，我们就提供了分离到ExpoKit的能力，这是你的Expo项目的本地Xcode和Android工作室的代表。

> ⚠️:如果您选择与Expo分开，一些Expo服务将不复存在。例如，我们不能再为您生成独立的构建。您需要自己管理本机构建。

# 额外的资源

以下资源对于学习Expo和它所依赖的一些项目是非常有用的。

## Expo 博客

- [Exposition](https://blog.expo.io/)——官方博客，每个月我们都会发布新闻稿和其他与Expo相关的内容。

## 课程 使用Expo

- [repl.it - React Native - 在接下来的5分钟内构建你的第一个应用程序](https://repl.it/site/react_native) (免费)
- [React Europe - React Native工作室视频在YouTube的介绍](https://www.youtube.com/playlist?list=PLCC436JpVnK2RFms3NG9ubPToWCNbMLbT) (免费)
- [RMOTR - React Native & Expo 移动开发](https://rmotr.com/introduction-to-react-native-and-expo) (付费)
- [Udemy - React Native: Stephen Grider的高级概念](https://www.udemy.com/react-native-advanced/) (付费)
- [Udacity - React Nanodegree](https://www.udacity.com/course/react-nanodegree--nd019) (付费)

## React Native

- [React Native Express](http://www.reactnativeexpress.com/) - React Native最佳实践入门! 这是对 React 和 React Native的构建块的一次演练.
- [官方 React Native 文档](https://facebook.github.io/react-native/docs/sample-application-movies.html)
- [React Native 基本原理](https://egghead.io/courses/react-native-fundamentals) (egadad.io视频课程)
- [动态 React Native UI 元素](https://egghead.io/courses/animate-react-native-ui-elements) (egadad.io的视频课程)
- [学习 React Native](http://shop.oreilly.com/product/0636920041511.do) (书)
- [Jani Eväkallio的聊天教程](https://github.com/jevakallio/react-native-chat-tutorial)

## 会谈

- [React Native Europe - Snack的建立](https://www.youtube.com/watch?v=Uk45O6AygH8)
- [React Europe - Expo Snack](https://www.youtube.com/watch?v=U0vnAW4UNXE)
- [React Conf - 创建 React Native App](https://www.youtube.com/watch?v=9baaVjGdBqs)
- [Reactive - 从React web 到本地移动](https://www.youtube.com/watch?v=-XxSCi8TKuk)
- [React Europe - 建造 Android的 li.st以 Expo 和 React Native](https://www.youtube.com/watch?v=cI9bDvDEsYE)
- [哈佛大学CS50课程 - 很容易就能建立React Native应用](https://www.youtube.com/watch?v=uFrAZfPW9JY)
- [Apollo Day - GraphQL at Expo](https://www.youtube.com/watch?v=E398q4HGRBA)

## React

- [React Express](http://www.react.express/)
- [React 课程 Egghead.io](https://egghead.io/technologies/react) - 几个播放列表，包括React和Redux基本原理.
- [官方 React 文档](https://facebook.github.io/react/docs/getting-started.html)

## JavaScript

- [ES6 Katas](http://es6katas.org/) - 为了让自己熟悉新JavaScript特性，可以让自己熟悉使用React Native的新JavaScript特性
