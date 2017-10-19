---
title: TCLMOVE 架构设计与技术选型
date: 2017-10-18 14:28:43
tags:
	- iOS
	- 架构设计
categories:
	- iOS
---

##  架构设计 ##

`架构`，就是对`软件复杂度`的`管理`。

**层次划分**

**三层结构框架**

* *UI Layer*
* *Logic Layer*
* *Persistence Layer*

![TCLMOVE框架图](http://oxwfu3w0v.bkt.clouddn.com/2017/10/18/mt30_tclmove_ios.svg)

### UI Layer 展现层

UI Layer 采用MVVM框架。

***View***

V对应View，责任只负责显示，不保存任何状态，保证View的`无状态`性，`states` 只能来源于外部，同样的`states`设置，View 显示显示内容一样。

View最主要的是实现组件化，与其它组件和模块保持最大化的隔离，对内的接口只有`states`,对外的接口只有`events`.

**比如** 基于Rx的方式 :

```
view.rx.states
view.rx.events.click
```

View的复杂度在于自身的实现(复杂的绘制／动画)，以及Views之间的组合，组件之间的最大化的隔离是防止复杂度的膨胀。

***ViewModel***

VM对应ViewModel, 负责绑定数据和Views。MVVM的机制是VM实现V和M的数据绑定，双向绑定或单向绑定，由于要实现View的无状态，在此只作单向的数据绑定，数据来源于M，单向流于view的states。

ViewModel对应一个ViewController,此Scene中所有的views状态设置都集中在其中，`统一处理`，避免代码分散在ViewController各处，造成逻辑混乱；views的enents也是经由此地，产生Actions.

ViewModel会分担大部分ViewController上的逻辑代码，但也只是UI层面的，主要工作是states和events的分发和中间状态的转化，一些简单的限制判断和states的组合／分解，仅此而已。

***Model***

M对应Model，UI Layer以下对应MVVM的Model.

V和VM负责展现层上的部分，作到无状态化。而所有的状态都在Model之中，状态改变UI呈现改变。


### Logic Layer 业务层

Logic Layer会根据业务划分出许多功能模块，业务层应该最先进入详细设计，对比需求分析有效的分离出独立的功能模块。

业务层主要有三个角色：

* **Manager** `管理者`
* **Worker Protocl** `工作协议`
* **Worker** `工作者`

以下我们将以公司组织结构类比的方式介绍业务层和三个角色。

业务层会根据业务需求划分出相应的功能模块，就像公司的各个不同的职能部门，各自有自己的分工与协作。

各部门组织成员都有`Manager`和`Worker`，有自己的职责。

`Manager` 主要职责是 管理Workers, 分发任务， 协调任务， 与上层交互。`Manager` 不会处理具体的工作，具体工作由`Worker`来实现。

`Worker`只关心一个单独的任务，其它的事情不涉及，而且只对他自己的`Manager`负责。`Worker`所需要的资源和工具来源于下游。

工作任务由`Manager`下派，通过工作清单即`Worker Protocl`。`Worker Protocl`是工作规范（即接口也叫协议），指明了需要实现的操作。而能够完成这些规范的`Workers`，不只一个，比如经理(`Manager`)需要存储一份文档，一个小秘(`Worker`)说她可以存储在本地，一个码农(`Worker`)说他可以上传存储到服务器，至于使用那个`Worker`由`Manager`决定。

部门之间的协作也是由`Manager`完成，部门经理会向其它部门获取结果。如果一项工作需要几个部门协作完成，可以产生一个更高级的`Manager`，管理这几个部门的`Managers`。

### Persistence Layer 持久层

持久层是对接基础设施，资源的出入口，也可以叫资源层。

其主要包括：

* HTTP Client
* 数据库
* BLE
* 本地文件
* 资源文件
* ...

**HTTP Client**

服务器端采用HTTP/RESTful，移动端对应采用API层和Client层。

* `Client层` 对应HTTP客户端功能，完成HTTP网络请求功能。
* `API层` 对应Rest，完成接口定义，让上层通过接口直接操作资源。

还需要完成本地缓存功能，配合服务端的HTTP缓存机制，和自己的离线缓存。

**数据库**

数据库实现DAO（数据访问对象），统一数据库的数据操作标准。

数据库采用ORM(对象关系映射)框架，屏蔽SQL的操作步骤。

Change Events,当数据库数据改变时，产生相应的通知事件。

**BLE**

**File Manage**

**Resource Manage**

### 三层之间的桥梁

三层结构`UI Layer`，`Logic Layer`，`Persistence Layer`如上所述，各自有各自的职责，彼此之间也是隔离的，这样就可以分配给不同的开发人员，独立的单元测试。而它们之间需求建立沟通的桥梁。

### App States

`App States`的思想来源与Flux架构，实现它的有前端的Redux。【[可参考 Redux](http://www.redux.org.cn)】

`App States`是一个状态管理器，如上所述UI层是无状态的，而它的状态来源是`App States`，并且作状态的统一管理。

`App States`保存了所有的状态，通过View Model绑定View，当状态发生变化时相应的View会发生改变。

这里最大的难度是`states`。归纳`states`需要即要`全面`也要`精准`,那些值可以是`state`那些不是请慎重考虑。比如业务数据中的小孩子信息是一定在`states`中，还有像 _正在获取销毁信息的状态_ 也是一个布尔的`state`，但像_小孩子的数目_这种值，已经可以从组织size中知道，就不应该在`states`，这些值会照成`states`冗余的，使得管理复杂化。

然后，记住以下几点：
> `State` 是只读的
> 
> 惟一改变 `State` 的方法就是触发 `action`，通过 `reducers` 改变 `states`
> 
> `states`的结构尽量扁平化，不要有太多的嵌套。
> 

### Repository

`Repository`采用外观模式。

`Repository`对接Persistence Layer，提供一组一致的接口，定义一个高层接口，上层不关心数据来源于那里（那个网络服务器或蓝牙或数据库），它只关心它要的数据。

---

## 技术选型 ##

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

Kingfisher是一个轻量级的、纯Swift的库，用于从web上下载和缓存图像。这个项目深受流行的SDWebImage的启发。

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

