---
title: Android 架构组件简介
date: 2017-11-08 09:35:45
tags:
- Android
- 架构设计
categories:
- Android架构组件

---


![arch-hero.png](https://developer.android.google.cn/topic/images/arch/hero/arch-hero.png)

一组帮助您设计`健壮的`、`可测试的`和`可维护的`应用程序库。
管理`UI组件生命周期`和处理`数据持久性`。

* [应用架构指南](/2017/11/08/GuidetoAppArchitecture/)
* [添加组件到你的项目](/2017/11/08/adding-components/)
* [使用Lifecycle组件来处理生命周期](/2017/11/08/architecture-lifecycle/)
* [LiveData](/2017/11/08/architecture-livedata/)
* [ViewModel](/2017/11/09/architecture-viewmodel/)
* [保存UI状态](/2017/11/09/architecture-saving-states/)
* [Room持久性库](/2017/11/09/architecture-room/)
* [分页库](/2017/11/09/architecture-paging/)

### 轻松管理app的生命周期

新的lifecycle组件感知生命周期，可以帮助您管理您的活动和片段生命周期。
保存配置更改，避免内存泄漏，并使用[LiveData](https://developer.android.google.cn/topic/libraries/architecture/livedata.html)、[ViewModel](https://developer.android.google.cn/topic/libraries/architecture/viewmodel.html)、[LifecycleObserver](https://developer.android.google.cn/topic/libraries/architecture/lifecycle.html)和[LifecycleOwner](https://developer.android.google.cn/topic/libraries/architecture/lifecycle.html)，轻松地将数据加载到UI中。

* [试一试codelab](https://codelabs.developers.google.com/codelabs/android-lifecycles/#0)
* [查看文档](https://developer.android.google.cn/topic/libraries/architecture/lifecycle.html)
* [获得示例项目](https://github.com/googlesamples/android-architecture-components)

### Room:SQLite对象映射库

使用[Room](https://developer.android.google.cn/topic/libraries/architecture/room.html) 避免使用样板代码，并轻松地将SQLite表数据转换为Java对象。
Room提供了SQLite语句的编译时间检查，并可以返回RxJava、Flowable和LiveData observables。

* [试一试codelab](https://codelabs.developers.google.com/codelabs/android-persistence/#0)
* [查看文档](https://developer.android.google.cn/topic/libraries/architecture/room.html)
* [获得示例项目](https://github.com/googlesamples/android-architecture-components)
