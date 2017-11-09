---
title: Android 架构组件 - ViewModel
date: 2017-11-09 11:26:10
tags:
- Android
- 架构设计
categories:
- Android架构组件

---


<img src="https://developer.android.google.cn/topic/images/arch/icons/icons_cards_viewmodel.svg" width="240" height="240" align=center/>

[ViewModel](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html)类旨在以生命周期的方式存储和管理与ui相关的数据。
`ViewModel`类允许数据在诸如屏幕旋转之类的配置更改中存活。

> 注意:要将ViewModel导入到您的Android项目中，请参见[向您的项目添加组件](/2017/11/08/adding-components/)。

Android框架管理UI控制器的生命周期，比如activities和fragments。
该框架可能会决定销毁或重新创建UI控制器，以响应完全超出您控制的某些用户操作或设备事件。

如果系统破坏或重新创建一个UI控制器，那么存储在其中的任何与UI相关的数据都将丢失。
例如，你的应用可能会在其中一个activity中包含一个用户列表。
当为配置更改重新创建activity时，新activity必须重新获取用户列表。
对于简单的数据，该activity可以使用[onSaveInstanceState()](https://developer.android.google.cn/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle))方法，并从[onCreate()](https://developer.android.google.cn/reference/android/app/Activity.html#onCreate(android.os.Bundle))中的bundle中恢复其数据，但是这种方法只适用于少量的数据，这些数据可以序列化然后反序列化，而不是像用户列表或位图那样的潜在的大量数据。

另一个问题是UI控制器经常需要进行异步调用，这可能需要一些时间才能返回。
UI控制器需要管理这些调用，并确保系统在被销毁后清除它们，以避免潜在的内存泄漏。
这种管理需要大量的维护，并且在为配置更改重新创建对象的情况下，由于对象可能不得不重新发出已经发出的调用，这是对资源的浪费。

诸如activities和fragments之类的UI控制器主要是用来显示UI数据、对用户操作作出反应，或者处理操作系统通信，比如权限请求。
要求UI控制器也负责从数据库或网络加载数据，这增加了类的膨胀。
给UI控制器分配过多的责任会导致一个单独的类，它试图独自处理应用程序的所有工作，而不是将工作委托给其他类。
以这种方式为UI控制器分配过多的责任也会使测试变得更加困难。

将视图数据所有权与UI控制器逻辑分离是更容易、更高效的。

<!-- more -->

## 实现ViewModel

架构组件为UI控制器提供ViewModel助手类，负责为UI准备数据。
在配置更改期间，`ViewModel对象将自动保留`，以便它们保存的数据立即可用到下一个activity或fragment实例。
例如，如果您需要在应用程序中显示一个用户列表，请确保将责任分配给一个ViewModel，而不是一个activity或fragment，如下面的示例代码所示:

```java
public class MyViewModel extends ViewModel {
    private MutableLiveData<List<User>> users;
    public LiveData<List<User>> getUsers() {
        if (users == null) {
            users = new MutableLiveData<List<Users>>();
            loadUsers();
        }
        return users;
    }

    private void loadUsers() {
        // 执行一个异步操作来获取用户.
    }
}
```

然后，您可以从以下activity中访问该列表:

```java
public class MyActivity extends AppCompatActivity {
    public void onCreate(Bundle savedInstanceState) {
        // Create a ViewModel the first time the system calls an activity's onCreate() method.
        // Re-created activities receive the same MyViewModel instance created by the first activity.

        MyViewModel model = ViewModelProviders.of(this).get(MyViewModel.class);
        model.getUsers().observe(this, users -> {
            // update UI
        });
    }
}
```

如果该activity被重新创建，它将收到第一个activity创建的`MyViewModel`实例。
当所有者activity完成后，框架调用[ViewModel](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html)对象的[onCleared()](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html#onCleared())方法，以便它可以清理资源。

> ⚠️:`ViewModel`永远不能引用视图、[Lifecycle](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html)或任何可能引用activity上下文的类。

ViewModel对象的设计是为了比具体的视图实例或LifecycleOwners实例活得更活。
这个设计还意味着您可以更容易地编写测试来覆盖ViewModel，因为它不知道视图和生命周期对象。
ViewModel对象可以包含LifecycleObservers，比如LiveData对象。
然而，**ViewModel对象永远不能观察对生命周期敏感的观察对象的变化**，例如LiveData对象。
如果ViewModel需要Application context，例如找到一个系统服务，它可以扩展[AndroidViewModel](https://developer.android.google.cn/reference/android/arch/lifecycle/AndroidViewModel.html)类，并拥有一个在构造函数中接收到Application的构造函数，因为Application类扩展了Context。

## ViewModel的生命周期

ViewModel对象是在获得ViewModel时被传递到ViewModelProvider的生命周期的范围。
ViewModel仍然存在于内存中，直到它被限定的生命周期永久地消失:在activity的情况下，当它finishes时，在一个fragment中，当它被detached时。

图1演示了一个activity的各个生命周期状态，当它进行轮转，然后结束。
插图还显示了与相关活动生命周期相邻的ViewModel的生命周期。
这个特殊的图说明了activity的状态。
同样的基本状态也适用于fragment的生命周期。

![viewmodel-lifecycle.png](https://developer.android.google.cn/images/topic/libraries/architecture/viewmodel-lifecycle.png)

在系统调用activity对象的`onCreate()`方法时，通常需要一个ViewModel。
系统可以在activity的整个生命周期中多次调用onCreate()，例如在设备屏幕被旋转时。
ViewModel存在于您第一次请求ViewModel时，`直到activity完成并销毁`。

## fragments之间共享数据

在一个activity中，两个或多个fragments需要相互通信是很常见的。
设想一个常见的master-detail fragments，其中有一个fragment，用户从一个列表中选择一个项目，另一个fragment显示所选项的内容。
这个案例并不简单，因为两个fragments都需要定义一些接口描述，而所有者activity必须将两者结合在一起。
此外，两个fragments必须处理另一个fragment尚未创建或可见的场景。

这个常见的痛点可以通过使用ViewModel对象来解决。
这些fragments可以使用它们的activity范围来共享一个视图模型，以处理这种通信，如下面的示例代码所示:

```java
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<Item> selected = new MutableLiveData<Item>();

    public void select(Item item) {
        selected.setValue(item);
    }

    public LiveData<Item> getSelected() {
        return selected;
    }
}

public class MasterFragment extends Fragment {
    private SharedViewModel model;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        itemSelector.setOnClickListener(item -> {
            model.select(item);
        });
    }
}

public class DetailFragment extends Fragment {
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        SharedViewModel model = ViewModelProviders.of(getActivity()).get(SharedViewModel.class);
        model.getSelected().observe(this, { item ->
           // Update the UI.
        });
    }
}
```

请注意，在获取ViewModelProvider时，两个fragments都使用getActivity()。
因此，这两个fragments都接收相同的SharedViewModel实例，该实例的作用域是activity。

这种方法提供了以下好处:

* activity不需要做任何事情，也不需要知道任何关于这种通信的信息。

* 除了SharedViewModel合同之外，Fragments不需要相互了解。
如果其中一个fragments消失了，另一个则继续正常工作。

* 每个fragment都有自己的生命周期，并且不受另一个生命周期的影响。
如果一个fragment替换另一个fragment，UI将继续工作，没有任何问题。

## 用ViewModel替换Loaders

像[CursorLoader](https://developer.android.google.cn/reference/android/content/CursorLoader.html)这样的Loader类经常用于同步数据库数据保持应用程序的UI。
您可以使用ViewModel，以及其他一些类来替换loader。
使用ViewModel将UI控制器与数据加载操作分离，这意味着类之间的强引用较少。

在使用loaders的一种常见方法中，应用程序可能使用一个CursorLoader来观察数据库的内容。
当数据库中的值发生变化时，loader会自动触发数据的重新加载，并更新UI:

![viewmodel-loader.png](https://developer.android.google.cn/images/topic/libraries/architecture/viewmodel-loader.png)

ViewModel使用Room和LiveData来替换loader。
ViewModel确保数据在设备配置更改中得以保存。
当数据库发生变化时，Room会通知您的LiveData，而LiveData则会用修改后的数据更新您的UI。

![viewmodel-replace-loader.png](https://developer.android.google.cn/images/topic/libraries/architecture/viewmodel-replace-loader.png)

[这篇博客文章](https://medium.com/google-developers/lifecycle-aware-data-loading-with-android-architecture-components-f95484159de4)描述了如何使用一个带有LiveData的ViewModel来替换一个[AsyncTaskLoader](https://developer.android.google.cn/reference/android/content/AsyncTaskLoader.html)。

随着您的数据变得越来越复杂，您可能会选择一个单独的类来加载数据。
ViewModel的目的是封装UI控制器的数据，使数据能够在配置更改中存活。
有关如何在配置更改中加载、持久化和管理数据的信息，请参见[保存UI状态](https://developer.android.google.cn/topic/libraries/architecture/saving-state.html)。

[Android应用程序架构的指南](/2017/11/08/GuidetoAppArchitecture/)建议构建一个存储库类来处理这些函数。
