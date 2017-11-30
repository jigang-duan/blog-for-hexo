---
title: CocoaPods 简介
date: 2017-11-30 13:03:18
tags:
- iOS
- CocoaPods
categories:
- iOS

---

CocoaPods是Swift和Objective-C Cocoa项目的一个依赖关系管理器。它有超过4万个libraries，在超过280万个应用程序中被使用。CocoaPods可以帮助你优雅地扩展你的项目。

<!-- more -->

## 安装

CocoaPods是用Ruby构建的，在OS X上默认安装有Ruby，我们建议你使用默认的Ruby。

使用默认的Ruby安装要求您在安装gems时使用sudo。[指南](https://guides.cocoapods.org/using/getting-started.html#getting-started)中还有进一步的安装说明。

```bash
# Xcode 8 + 9
$ sudo gem install cocoapods
```

## 开始

在Xcode项目目录中列出一个名为`Podfile`的文本文件中的依赖项:

```Ruby
platform :ios, '8.0'
use_frameworks!

target 'MyApp' do
  pod 'AFNetworking', '~> 2.6'
  pod 'ORStackView', '~> 3.0'
  pod 'SwiftyJSON', '~> 2.3'
end
```

提示: CocoaPods提供了一个`pod init`命令来创建一个带有智能默认值的Podfile。你应该使用它。

现在您可以在您的项目中安装依赖项:

```bash
$ pod install
```

在构建项目时，确保始终打开Xcode工作区，而不是项目文件:

```bash
$ open App.xcworkspace
```

现在您可以导入您的依赖项了:

```Objective-C
#import <Reachability/Reachability.h>
```

## 创建pod

有时候，对于你的一个依赖项，CocoaPods还没有一个pod。幸运的是，创建pod非常简单:

```bash
$ pod spec create Peanut
$ edit Peanut.podspec
$ pod spec lint Peanut.podspec
```

你可以在指南中找到很多关于这个[过程的信息](https://guides.cocoapods.org/)。当你完成的时候，你可以得到一个账户，把你的pod推到[CocoaPods的树干](https://guides.cocoapods.org/making/getting-setup-with-trunk.html)上。
