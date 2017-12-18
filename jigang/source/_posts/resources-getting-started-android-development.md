---
title: 现代ANDROID开发入门资源
date: 2017-11-14 09:54:31
tags:
- Android
categories:
- Android

---

![ResourcesForModernAndroidDev.png](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2017/08/ResourcesForModernAndroidDev.png?w=717&ssl=1)

[**原文地址**](https://riggaroo.co.za/resources-getting-started-android-development/)

作者：[Rebecca](https://riggaroo.co.za/author/rebecca/)


在为[DVT](http://dvt.co.za/)的移动开发者研究生项目汇集了资源之后，我意识到我所引用的内容可以成为现代Android开发入门的重要指导。

值得注意的是，我们有一个非常**实际的方法**来培养DVT的毕业生。我们举办工作坊和实际项目，以确保我们的毕业生对Android的发展有一个很好的理解。

随着网上大量的内容可用，当我开始Android开发时，我不知道在开发应用程序时应该注意哪些地方或什么。有一个简洁的列表，就像这个博客文章中的那个，对我来说是非常宝贵的。我希望你也能找到它的价值。

这里是一个链接，代码实验室和参考资料的列表，这对任何想要开始Android开发的开发人员都是有用的。

<!-- more -->

## 一般编程实践

一般编程实践是开发职业生涯开始成功的关键。这些做法包括：

* [源代码管理（Git）](https://git-scm.com/) - 源代码管理是一种管理代码版本的工具，它可以协同编写软件。

* [Git工作流程](https://www.atlassian.com/git/tutorials/comparing-workflows) - 在使用源代码管理时，有很多不同的方式来管理软件。流行的方法包括：Gitflow工作流程，集中式工作流程，分叉工作流程等

* [持续集成](https://www.thoughtworks.com/continuous-integration) - 持续集成可确保您的代码在不属于您自己的计算机的服务器上构建。看看使用像Jenkins，Buddybuild，Circle CI，Travis等构建服务器。

* [合并请求](https://www.atlassian.com/blog/git/written-unwritten-guide-pull-requests)  - 合并请求是获得您开发的代码非常详细的反馈的好方法。

* [敏捷/Scrum方法](https://www.scrumalliance.org/why-scrum/scrum-guide)  - 大多数现代软件开发团队遵循Scrum方法进行工作。

* 代码质量工具 - 公司用许多工具来衡量代码质量和代码库的健康状况。诸如测试覆盖范围的行数或代码库所具有的技术债务等度量标准是可见的。一些经常使用的工具：[Sonar](https://www.sonarqube.org/)，[FindBugs](http://findbugs.sourceforge.net/)，[Checkstyle](https://github.com/checkstyle/checkstyle)和[Android Lint](https://developer.android.com/studio/write/lint.html)。

## Android基础介绍

有一大堆网站提供了Android开发入门的基础知识。我的建议是遵循官方文档来理解基础知识，然后在进入应用程序设计的更多技术方面（参见本文后面的部分）中查看其他资源（如博客等）以获取更多信息。

一些入门资源：

* [Android应用基础](https://developer.android.google.cn/guide/components/fundamentals.html)
* Android中的一些主要组件： [Activities](https://developer.android.google.cn/guide/components/activities/index.html), [Fragments](https://developer.android.google.cn/guide/components/fragments.html), [Services](https://developer.android.google.cn/guide/components/services.html), [Broadcast Receivers](https://developer.android.google.cn/guide/components/broadcasts.html)。
* [Android应用程序清单](https://developer.android.google.cn/guide/topics/manifest/manifest-intro.html)
* [代码实验室 - 建立您的第一个Android应用程序](https://codelabs.developers.google.com/codelabs/build-your-first-android-app/index.html)

## 掌握Android中的布局

Android中有很多不同的布局类型，从FrameLayout到RelativeLayout到ConstraintLayout。确保你对这些常用的布局类型感到满意：[FrameLayout](https://developer.android.google.cn/reference/android/widget/FrameLayout.html)，[RelativeLayout](https://developer.android.google.cn/guide/topics/ui/layout/relative.html)，[LinearLayout](https://developer.android.google.cn/guide/topics/ui/layout/linear.html)，[ConstraintLayout](https://developer.android.google.cn/reference/android/support/constraint/ConstraintLayout.html)，[CoordinatorLayout](https://developer.android.google.cn/reference/android/support/design/widget/CoordinatorLayout.html)。

**资源**：

* [支持不同的屏幕尺寸](https://developer.android.google.cn/training/multiscreen/screensizes.html)
* [代码实验室 - ConstraintLayout](https://codelabs.developers.google.com/codelabs/constraint-layout/index.html)
* [代码实验室 - CoordinatorLayout](https://codelabs.developers.google.com/codelabs/mdc-android/index.html)

## 构建系统 - 使用Gradle

使用Gradle在开发Android应用程序时可能会被忽视。确保你了解基础知识，甚至更好 - 学习如何编写自己的gradle任务！

**资源**：

* [Gradle文档](https://gradle.org/)
* [配置你的构建](https://developer.android.google.cn/studio/build/index.html)

## Android中的网络

尽管大部分[Android文档](https://developer.android.google.cn/training/basics/network-ops/index.html)都没有引用Retrofit或OkHttp，但在Android中进行联网时，这些是最常用的库。熟悉Android Studio中提供的不同分析工具也是很好的做法。

**资源**：

* [了解RESTful服务](https://www.tutorialspoint.com/restful/)
* [Retrofit](http://square.github.io/retrofit/) - 适用于Android和Java的类型安全的HTTP客户端
* [OkHttp](http://square.github.io/okhttp/)  - 用于Android和Java应用程序的HTTP和HTTP/2客户端
* [Android中的网络分析器](https://developer.android.com/studio/profile/network-profiler.html)  - Android Studio中的一款工具，可用于分析您的网络呼叫。
* [Charles Proxy](https://www.charlesproxy.com/)  - 用于在测试时拦截网络呼叫。

## 架构您的Android应用程序

不幸的是，编写代码并让应用程序编译并不是知道如何编写可维护的Android应用程序的结束。大规模的Android应用程序需要遵循良好的架构设计，以使其可维护和可测试。编写Android应用程序时可以遵循许多不同的模式。通常使用MVP，MVVM和Clean Architecture等模式。确保你了解模式之间的差异，因为你会在外遇到许多不同的模式。

**资源：**

* [Android体系结构组件指南](/2017/11/08/AndroidArchitectectureComponents/)
* 我在Android体系结构组件3部分组成的系列（部分[1](https://riggaroo.co.za/android-architecture-components-looking-room-livedata-part-1/)，[2](https://riggaroo.co.za/android-architecture-components-looking-viewmodels-part-2/)，[3](https://riggaroo.co.za/android-architecture-components-looking-lifecycles-part-3/)）
* [Android体系结构组件视频介绍](https://github.com/googlesamples/android-architecture-components)
* [Google Sample App Github存储库](https://github.com/googlesamples/android-architecture-components)
* [代码实验室 - 持久性](https://codelabs.developers.google.com/codelabs/android-persistence/index.html)
* [代码实验室 - 生命周期感知组件](https://codelabs.developers.google.com/codelabs/android-lifecycles/index.html)

## 测试你的Android应用程序

一旦你掌握了创建Android应用程序的机会，你将需要考虑如何测试它们。单元测试和UI测试是非常重要的概念，您需要确保自己理解。有很多不同的工具可以用来编写UI测试。大多数Android开发人员使用Espresso和JUnit编写测试，但是还有很多其他工具，例如Robotium，Calabash，Appium等等。我推荐使用Espresso和JUnit。

**资源：**

* [Android测试支持库](https://developer.android.google.cn/topic/libraries/testing-support-library/index.html)
* [Espresso](https://developer.android.google.cn/training/testing/espresso/basics.html)
* [JUnit](http://junit.org/junit4/)
* [Mockito](http://site.mockito.org/)
* [代码实验 - Android测试](https://codelabs.developers.google.com/codelabs/android-testing/index.html)
* [代码实验 - Android性能测试](https://codelabs.developers.google.com/codelabs/android-perf-testing/index.html)

## 发布您的Android应用程序

* [准备您的应用程序的发布](https://developer.android.google.cn/studio/publish/preparing.html)
* [应用程序签名](https://developer.android.google.cn/studio/publish/app-signing.html)
* [版本化您的应用程序](https://developer.android.google.cn/studio/publish/versioning.html)
* [ProGuard](https://developer.android.google.cn/studio/build/shrink-code.html)

## 安全

为了保护您的应用程序，确保没有人能够访问未经授权的内容，应该做很多事情。确保你正在使用ProGuard（前面提到过）。
了解[中间者](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)是什么样。
了解不同的加密方法以及安全地将信息存储在Android应用程序中的方法，包括保护您的API令牌，证书锁定等。

**资源：**

* Android的[安全提示](https://developer.android.google.cn/training/articles/security-tips.html)
* [证书锁定](https://square.github.io/okhttp/3.x/okhttp/okhttp3/CertificatePinner.html)
* [SafetyNet API](https://developer.android.google.cn/training/safetynet/index.html)
* [Android密钥库系统](https://developer.android.google.cn/training/articles/keystore.html)

## 高级Android主题

一旦你已经涵盖了编写Android应用程序的所有基础知识，有几个高级主题，你可能需要覆盖，以便贡献给一些代码库：

* [Kotlin](https://kotlinlang.org/)  - Kotlin是Android的新编程语言，开发人员正在Kotlin积极编写他们的代码。值得关注Kotlin并穿过Kotlin Koans。还有一个Kotlin[代码实验室](https://codelabs.developers.google.com/codelabs/build-your-first-android-app-kotlin/index.html)。
* [RxJava](https://github.com/ReactiveX/RxJava)  -  RxJava是用于基于事件的异步编程的库。它允许您将操作组合在一起以执行复杂的任务（如将多个网络调用组合在一起），并且可以非常有用地管理您的代码在哪个线程上执行。有一个伟大的[视频](https://www.youtube.com/watch?v=htIXKI5gOQU)来自杰克·沃顿，描述如何使用RxJava和使用它的好处。
* [Dagger](https://google.github.io/dagger/)（依赖注入） - 依赖注入是一种在应用程序中管理对象及其依赖关系的方法。DI的概念不是一个Android的概念，但也可以在许多其他框架中使用。DI可以使您的代码更有效地提高内存并提高可测试性。Dagger 2是最受欢迎的Android DI框架。
* [Material Design](https://material.io/)  - 大多数Android应用程序都遵循Google的材料设计指南。指南是以用户熟悉的标准方式设计您的应用程序的一种方式。
* [Android支持库](https://developer.android.google.cn/topic/libraries/support-library/index.html)  -  Android中的支持库对于确保您的应用在多个Android版本中的外观和行为保持一致非常重要。有几个不同的库有不同的目的。链接的文章描述了图书馆背后的推理。
* **内存泄漏 **-  在Android中，创建内存泄漏非常容易。这可能会导致应用程序中的错误行为（随机崩溃）。阅读有关内存泄漏的[信息](https://riggaroo.co.za/fixing-memory-leaks-in-android-outofmemoryerror/)。很多开发人员在应用程序中使用[LeakCanary](https://github.com/square/leakcanary)来确保没有任何内存泄漏。

## 跟上现代Android开发的最新进展

有很多方法可以跟上Android社区的最新变化和发展。我觉得有用的一些方法是：

* 订阅[Android Weekly](http://androidweekly.net/)简讯
* 在Reddit上关注[/r/androiddev](https://www.reddit.com/r/androiddev/)
* 在Twitter上关注[Android Google Developer Experts](https://developer.android.google.cn/experts/all/technology/android)
* 在Twitter上关注[Android Studio](https://twitter.com/androidstudio)，[Android Dev](https://twitter.com/AndroidDev)
* 听Android开发播客([Fragmented](http://fragmentedpodcast.com/), [Android Developers Backstage](http://androidbackstage.blogspot.co.za/), [The Context](https://player.fm/series/the-context-androiddev), [Android Snacks](https://player.fm/series/android-snacks))


---

作为一名Android开发人员，您会遇到许多概念，例如处理通知，小部件，主题和样式化应用程序等。
此列表涵盖了我认为有必要成为卓越的Android开发人员的主要主题，大部分其他主题你可以自己去理解。
