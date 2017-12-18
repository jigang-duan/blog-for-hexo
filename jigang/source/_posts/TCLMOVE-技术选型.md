---
title: TCLMOVE 技术选型
date: 2017-10-19 19:50:15
tags:
- iOS
- 架构设计
- TCLMOVE
categories:
- iOS

---

![TCLMOVE](http://oxwfu3w0v.bkt.clouddn.com/2017/10/20/tclmove1.png)

* Swift
* Storyboard
* RxSwfit
* Rleam
* R.Swfit
* Moya+Alamofire+ObjectMapper
* SwiftyBeaver
* Kingfisher
* CocoPod
* ...

<!-- more -->

#### 编程语言选择

* Swift
* Object-C

Swift的优势：

> 1. 趋势 苹果已经明确，Swift将是未来的主力开发语言
> 2. Swift定位是安全，快速，跨平台的语言
> 3. gitHub上新增的第三方开源库，Swift版本要多余Object-C的版本

Swift的缺点：

> 1. 不支持Runtime
> 2. 编译速度慢
> 3. Swift核心库会打包在APP中，增加了APP体积

***建议：***  对于新项目尽量采用Swift，对于业务复杂的旧项目Object-C继续维持

#### 代码手写 UI 和 Storyboard 之间的取舍

构建 UI 方式的争论就在 Cocoa 开发者社区里一直发生着，Storyboard 被诟病最多的 是`冲突风险`和`加载速度`。

**冲突风险**

Storyboard 一直在进步，在 Xcode 7 引入了 SB reference 以后，「SB 容易冲突」已经得到很好的解决了。

**加载速度**

编译过程中，项目里用到的 SB 文件也会被编译，并以 storyboardc 为扩展名保存在最终的 app 包内。这个是一个文件夹，里面存储了一系列 .nib 文件。SB 中的每个对象将会被编译为一个单独的 .nib。

**Storyboard 优势**

Storyboard可以简化UI的开发，属性设置和布局都可以简单化，实现和逻辑代码的分离，不会污染代码。

***建议：***  采用Storyboard

#### RxSwift

RxSwift是Swift函数响应式编程的一个开源库，由Github的ReactiveX组织开发，维护。

**RxSwift的优点**

* Composable 可组合，在设计模式中有一种模式叫做组合模式，你可以方便的用不同的组合实现不同的类
* Reusable 代码可重用，原因很简单，对应RxSwift，就是一堆Obserable
* Declarative 响应式的，因为状态不可变，只有数据变化
* Understandable and concise 简洁，容易理解。
* Stable 稳定，因为RxSwift写出的代码，单元测试时分方便
* Less stateful “无”状态性，因为对于响应式编程，你的应用程序就是一堆数据流
* Without leaks 没有泄漏，因为资源管理非常简单

#### Rleam

[Realm](https://realm.io)是由Y Combinator孵化的创业团队开源出来的一款可以用于iOS(同样适用于Swift&Objective-C)和Android的跨平台移动数据库。

* 跨平台：支持的平台包括Java，Objective-C，Swift，React Native，Xamarin。
* 简单易用：Core Data 和 SQLite 冗余、繁杂的知识和代码，而Realm，可以极大地减少学习成本，立即学会本地化存储的方法。
* 可视化：Realm 还提供了一个轻量级的数据库查看工具，可以查看数据库当中的内容，执行简单的插入和删除数据的操作。

#### R.Swfit

[R.Swfit](https://github.com/mac-cain13/R.swift) 在Swift项目中自动生成资源（像图片，字体，转场）相关的强类型变量，可以优雅的获取资源，仿Android资源文件使用的方法。

相关资源：

* [Swift-颜色设置技巧和(.clr)文件的创建和使用](http://blog.csdn.net/mazy_ma/article/details/73024576)
* [Sip：做屏幕取色没人比我强](https://www.waerfa.com/sip-color-picker)

#### Moya+Alamofire+ObjectMapper


[Moya](https://github.com/Moya/Moya)的基本思想是，提供一些网络抽象层，它们被充分的封装了且实际上直接调用了Alamofire. 它不仅在普通的简单的事情上很容易使用，而且在综合的复杂的事情上也容易使用

如果你使用 Alamofire 来抽象 URLSession, 那为什么不使用一些方式来抽象URLs和parameters等等的本质呢?
Moya的一些特色功能:

- 对正确的API端点访问进行编译时检查.
- 让您使用关联的枚举值定义不同端点的清晰用法.
- 把test stub作为一等公民，所以单元测试超级简单.

![Moya Overview](https://raw.githubusercontent.com/Moya/Moya/master/web/diagram.png)

[Alamofire](https://github.com/Alamofire/Alamofire) 是一个用Swift编写的HTTP网络库.

特性：

* 链式的请求/响应方法
* URL/JSON/plist参数编码
* 上传文件/数据/流/MultipartFormData
* 使用请求或恢复数据下载文件
* 身份验证使用URLCredential
* HTTP响应验证
* 上传和下载进度闭包
* cURL命令输出
* 动态调整和重试请求
* TLS证书和公钥固定
* 网络可达性
* 综合单元和集成测试覆盖
* 完整的文档


[ObjectMapper](https://github.com/Hearst-DD/ObjectMapper)是一个用Swift编写的框架，它使您可以很容易地将模型对象(类和结构)转换为JSON。

特性：

* JSON对象映射
* 将对象映射到JSON
* 嵌套对象(在数组或字典中单独使用)
* 自定义转换期间映射
* 结构支持
* 常量的支持

#### SwiftyBeaver

[SwiftyBeaver](https://github.com/SwiftyBeaver/SwiftyBeaver)，Swift多彩日志记录。

特性:

* 开发期间: 彩色记录到Xcode控制台
* 开发期间: 彩色记录文件
* 发布期间: 加密记录到SwiftyBeaver平台
* 通过Mac App来浏览、搜索和过滤

#### Kingfisher

[Kingfisher](https://github.com/onevcat/Kingfisher)是一个轻量级的、纯Swift的库，用于从web上下载和缓存图像。这个项目深受流行的SDWebImage的启发。

特性:

* 异步图像下载和缓存。
* 基于URLSession。基本的图像处理器和过滤器。
* 用于内存和磁盘的多层缓存。
* 可取消下载和处理任务以提高性能。
* 独立的组件。根据需要分别使用下载程序或缓存系统。
* 预取图像并在必要时从缓存中显示它们。
* 用于UIImageView、NSImage和UIButton的扩展，可以直接从URL中设置图像。
* 设置映像时内置的转换动画。
* 可定制的占位符，同时载入图片。
* 可扩展的图像处理和图像格式支持

#### CocoPods

[CocoaPods](https://cocoapods.org) iOS端的依赖管理工具。

使用CocoPods作项目依赖库的管理工具，包括第三方和本地的。
