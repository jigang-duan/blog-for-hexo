---
title: Android 架构组件 - 保存界面状态
date: 2017-11-09 13:12:41
tags:
- 架构设计
- Android架构组件
categories:
- Android

---


<img src="https://developer.android.google.cn/topic/images/arch/icons/icons_cards_viewmodel.svg" width="240" height="240" align=center/>

您所做的或者不做的，保持UI状态是用户体验的一个关键部分。
无论用户是旋转设备，用户重新启动应用，还是系统关闭应用，重要的是你的activity保持了用户所期望的状态

* 如果要保存的UI数据是`简单而轻量级的`，那么您可以单独使用onSaveInstanceState()来保存您的状态数据。
* 如果要保存的是`复杂数据`，可以使用ViewModel对象、onSaveInstanceState()方法和持久化本地存储的组合。

本页讨论了这些方法中的每一个。

<!-- more -->

## 管理简单的情况: onSaveInstanceState()

[onSaveInstanceState()](https://developer.android.google.cn/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle))回调被设计用来存储相对少量的数据，以便轻松地重新加载UI控制器的状态，如activity或fragment，如果系统停止并稍后重新创建该控制器。
这个回调是为了处理两种情况:

* **由于内存限制**，该应用程序在后台运行时会[杀死应用程序的进程](https://developer.android.google.cn/guide/components/activities/activity-lifecycle.html#asem)。
* **发生配置更改**，例如屏幕旋转或更改为输入语言。

由于这两种情况都暗示，在系统中activity停止，但未完成的情况下，onSaveInstanceState()将被调用。
例如，如果用户离开应用程序几个小时，系统会从内存中弹出相关的进程，系统会调用onSaveInstanceState()的默认实现来保存具有ID的每个UI控制器。
稍后，当用户返回到应用程序时，系统将用保存的状态恢复activity。

> 注意:onSaveInstanceState()在用户显式关闭该活动时，或者在调用finish()时，不会调用onSaveInstanceState()。

系统自动保存并为您恢复大量的UI数据:onSaveInstanceState()的默认实现保存了activity的视图层次结构的信息，比如[EditText](https://developer.android.google.cn/reference/android/widget/EditText.html)小部件中的文本或[ListView](https://developer.android.google.cn/reference/android/widget/ListView.html)小部件的滚动位置。
您还可以通过重写onSaveInstanceState()回调来将自定义数据保存到这个包中。
如果您重写该方法，以保存每个单独实体未捕获的附加信息，那么您应该调用缺省实现，除非您准备好保存每个实体的状态。

`onSaveInstanceState()不是用来存储大量数据的`，比如位图，或者复杂的数据结构，这些数据结构需要长时间的序列化或反序列化。
如果序列化的对象是复杂的，串行化可以消耗大量的内存。
由于这个过程在配置更改期间发生在主线程上，因此序列化可能会导致删除帧和视觉口吃，如果时间过长的话。
因此，不要在复杂的数据结构中使用onSaveInstanceState()，而是要确保将这些结构存储在本地持久存储中;在创建数据的时候存储数据是一个好主意，这样可以最小化丢失数据的机会。
然后，使用onSaveInstanceState()为每个对象存储唯一的id。

本文档的下一部分提供了关于保存更复杂数据的更多细节。

## 管理更复杂的状态:分而治之

当一个activity结束时需要保存更复杂的数据结构，可以通过将工作划分为几种类型的存储机制，从而有效地保存和恢复UI状态。

有两种一般的方法可以让用户离开activity，
导致用户可能期望的两种不同结果:

* **用户完全关闭该activity**。
用户可以完全关闭该activity，如果他们将activity从屏幕上划掉，导航出去，或者退出activity。
在这些情况下，假设用户已经永久地离开了activity，并且如果他们重新打开了activity，他们将期望从一个干净的状态开始。

* 用户可以旋转手机，或者把activity放在后台，**然后再回到它上面**。
例如，用户执行搜索，然后按下home键或接听一个电话。
当他们返回到搜索activity时，他们期望找到搜索关键字和结果，就像以前一样。

要在这两种情况下实现复杂数据结构的行为，您可以使用本地持久性、ViewModel类和onSaveInstanceState()方法。
每种方法都存储了activity中使用的不同类型的数据。

* 本地持久性:如果您打开并关闭该activity，则存储所有您不想丢失的数据。
	* 示例:一个包含音频文件和元数据的歌曲对象集合。
* ViewModel:存储在内存中的所有数据，以显示相关的UI控制器。
	* 示例:最近搜索的歌曲对象和最近的搜索查询。
* onSaveInstanceState():如果系统停止，然后重新创建UI控制器，就需要存储少量的数据，以便轻松地重新加载活动状态。这里不再存储复杂的对象，而是将复杂的对象持久化到本地存储中，并在onSaveInstanceState()中为这些对象存储一个惟一的ID。
	* 示例:存储最近的搜索查询。

举个例子，考虑一个允许你搜索你的歌曲库的activity。
下面是如何处理不同的事件：

当用户添加一首歌时，ViewModel会立即代表本地保存这些数据。
如果这个新添加的歌曲是应该在UI中显示的，那么您还应该更新ViewModel对象中的数据，以反映这首歌的添加。
记住要从主线程中执行所有的数据库插入操作。

当用户搜索歌曲时，无论从数据库中加载的用于UI控制器的复杂歌曲数据都应该立即存储在ViewModel对象中。
您还应该在ViewModel对象中保存搜索查询本身。

当activity进入后台时，系统调用onSaveInstanceState()。
您应该在onSaveInstanceState()包中保存搜索查询。
这少量的数据很容易保存。
它也是将activity恢复到当前状态所需的所有信息。

## 恢复复杂状态: 重组部件

当用户返回activity的时候，有两种可能的场景来重建activity:

* 该activity在被系统停止之后重新创建。
该activity在onSaveInstanceState()包中保存了查询，并且应该将查询传递给ViewModel。
ViewModel看到它没有缓存的搜索结果，并且使用给定的搜索查询来装载搜索结果。

* 该activity是在配置更改之后创建的。
该activity在onSaveInstanceState()包中保存了查询，并且ViewModel已经缓存了搜索结果。
您将查询从onSaveInstanceState()bundle传递到ViewModel，这就决定了它已经加载了必要的数据，并且不需要重新查询数据库

> ⚠️:当一个activity最初创建时，onSaveInstanceState() bundle不包含数据，ViewModel对象是空的。
当您创建ViewModel对象时，您将传递一个空查询，该查询将告诉ViewModel对象，目前还没有数据可加载。
因此，activity从一个空的状态开始。

根据您的activity实现，您可能不需要使用onSaveInstanceState()。
例如，浏览器可能会将用户返回到他们在浏览器退出之前所看到的精确的网页。
如果您的activity以这种方式运行，您可以放弃使用onSaveInstanceState()，而是在本地保存所有内容。
在搜索示例中，这可能意味着在共享首选项中持久化最近的查询。

此外，当您从一个intent打开一个activity时，当配置更改和系统恢复activity时，附加的附加部分将被交付到activity中。
如果将搜索查询作为一个intent extra传递，您可以使用extras  bundle而不是onSaveInstanceState() bundle。

在这两种情况中，您仍然会使用ViewModel来避免在配置更改期间从数据库中重新加载数据的循环。
