---
title: Android 自动化测试 - 第1部分 介绍
date: 2017-11-17 17:29:12
tags:
- Android
- 自动化测试
- Android自动化测试
categories:
- Android

---

<img src="http://szimg.mukewang.com/5850bc4500015ecd05400300-360-202.jpg" width="88%" height="320" align=center/>


很多人对如何在Android中进行测试感到困惑和不确定。
在过去，测试Android应用程序是非常困难的，并没有太多的方向。在这个系列中，将尝试让你的测试更容易一些。
这第一篇文章只是为了让你开始测试，接下来的几章将深入Android的测试。
让我们开始吧！

<!-- more -->

## 为什么要测试呢？

* 测试迫使你以不同的方式思考，并隐含地使你的代码更清晰。
* 如果您的代码有测试，您会对自己的代码更有信心。
* 闪亮的绿色状态栏和详细的报告详细，说明编写测试的后果覆盖了你多少的代码。
* `回归测试`变得容易很多，因为自动化测试会首先收集错误。

回归测试对我来说是最大的好处。如果你重构代码，且测试仍然通过，你会相信你没有造成任何破坏。
**测试的问题在于，您可能不会立即看到测试的好处**，因为真正的价值只会在需要重构的几个月后才会显现出来。

## Android中有哪些类型的测试？

### 单元测试

单元测试通常以可重复的方式实现尽可能小的代码单元（可以是方法，类或组件）的功能。

用于执行此测试的工具：

* [JUnit](http://junit.org/)  - 正常的测试断言。
* [Mockito](http://site.mockito.org) - 模拟其他未经测试的类。
* [PowerMock](https://github.com/jayway/powermock)  - 模拟静态类，如Android环境类等。

### UI测试 - 交互测试

UI测试或交互测试模拟典型的用户与您的应用程序的交互。
点击按钮，输入文本是UI测试可以完成的一些事情。

用于执行此测试的工具：

* [Espresso](https://google.github.io/android-testing-support-library/docs/espresso/)  - 用于在你的应用中进行测试，选择项目，确保某些东西是可见的。
* [UIAutomator](https://developer.android.google.cn/training/testing/ui-testing/uiautomator-testing.html)  - 用于测试不同应用之间的交互。

还有其他工具可用于此类测试，如  [Robotium](http://robotium.com/)，  [Appium](http://appium.io/)，  [Calabash](http://calaba.sh/)，  [Robolectric](http://robolectric.org/)。

## 开始自动化测试我需要做什么?

为了让你在应用中开始自动测试，你应该遵循某种架构模式，帮助你以一种简洁、可测试的方式测试和构建你的应用程序。很容易进行测试的一个模式是[模型视图展示者(MVP)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93presenter)，用于查看网络和数据库访问的视图和[存储库模式](http://martinfowler.com/eaaCatalog/repository.html)。

当然，这不是您作为Android开发者唯一的选择，它是您可以探索的许多方法之一，以便获得测试覆盖率和简洁的架构。

我发现在没有定义结构的情况下实现测试非常困难，通常我的测试是无用的，我不知道要测试什么，在大型应用程序中获取代码也很困难。
我的UI测试也不是很可靠，因为他们正在对生产服务器进行测试。希望我们可以在这个博客系列中解决这些问题。

## 以可测试的方式构建应用程序

下面是一个图表，描述了我将用作博客系列文章的其余部分的指导原则:

![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/06/BasicStructureOfAndroidApp.png?w=276&ssl=1)

*  **视图(Views)** - `Activities`和`Fragments`。
这是我们设置文本并进行UI更改的地方。我通常喜欢在这部分中保留Android特定的代码，并尽量不传递比视图更深入的内容，显然这并不总是容易的，但这是一个很好的指导，可以尝试遵循。
视图应该只与`presenters`对话。

* **展示者(Presenters)** - 这是决定在视图中应该显示什么内容的业务逻辑。
`Presenters`与`Repositories`交互以获取信息，并将信息传达给视图。
如果你可以避免使用Android特有的代码，尽量避免使用Android的代码，因为这会使单元测试变得更加困难。

* **存储库(Repositories)** — 决定从哪里获取数据，它们是否来自本地持久化的数据?还是应该从网络中获得数据?
`Repositories`和`presenters`交谈

* **模型(Models)** — 通常是POJOs，这些模型被Presenter和View使用，以便将信息从Presenter中传递给View。

单元测试将测试Presenters和Repositories。UI测试将测试视图一直到Repositories。

有很多很好的文章描述了MVP和其他架构选项——这篇博客文章不会详细介绍这些细节。请阅读更多关于Android架构的文章，[这里](https://labs.ribot.co.uk/android-application-architecture-8b6e34acda65#.qbyl367c7),[这里](https://github.com/googlesamples/android-architecture)和[这里](https://github.com/android10/Android-CleanArchitecture)。

## 在我的Android应用中，自动测试在哪里?

在你的Android应用文件夹结构中，有两个文件夹可以存放测试: `test`和`androidTest`

![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/06/Automated-testing-in-Android.png?resize=768%2C904&ssl=1)

* `androidTest` —— Android交互测试就在这里。这个文件夹中的测试需要运行在Android模拟器或物理设备上。

* `test/`  – 单元测试被放置在这个文件夹中。
单元测试运行在本地机器上的JVM上，不运行在Android设备或模拟器上。
这意味着他们不能访问Android类(例如`Context`类)。


---

现在我们已经知道了不同种类的测试以及在我们的应用中放置它们的位置。
下一篇文章我们将深入介绍如何构造代码，以便更好地进行单元测试。请看这里的第2部分！
