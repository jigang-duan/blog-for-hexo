---
title: Android 架构组件 - 使用Lifecycle组件来处理生命周期
date: 2017-11-08 16:50:06
tags:
- Android
- 架构设计
categories:
- Android架构组件

---

<img src="https://developer.android.google.cn/topic/images/arch/hero/desktoplifecycles-2x.png" width="320" height="240" align=center/>

Lifecycle组件执行操作以响应另一个组件的生命周期状态的变化，例如activities和fragments。
这些组件可以帮助您生成更有组织的、更轻的代码，这样更容易维护。

一个常见的模式是在activities和fragments的生命周期方法中实现相关组件的操作。
然而，这种模式导致了代码的糟糕组织和错误的扩散。
通过使用lifecycle组件，您可以将依赖组件的代码从生命周期方法中转移到本身组件中

[android.arch.lifecycle](https://developer.android.google.cn/reference/android/arch/lifecycle/package-summary.html)包提供了类和接口，允许您构建基于生命周期的组件，这些组件可以根据activity或fragment的当前生命周期状态自动调整它们的行为。

> ⚠️:导入`android.arch.lifecycle`在您的Android项目，请参见[向您的项目添加组件](/2017/11/08/adding-components/)。

<!-- more -->

大多数在Android框架中定义的应用程序组件都有生命周期。
生命周期由操作系统或在您的过程中运行的框架代码来管理。
它们是Android如何工作的核心，你的应用必须尊重它们。
如果不这样做，可能会触发内存泄漏或应用程序崩溃。

*假设我们有一个activity，在屏幕上显示的设备位置。一个常见的实现可能如下:*

```java
class MyLocationListener {
    public MyLocationListener(Context context, Callback callback) {
        // ...
    }

    void start() {
        // connect to system location service
    }

    void stop() {
        // disconnect from system location service
    }
}

class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    @Override
    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, (location) -> {
            // update UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        myLocationListener.start();
        // manage other components that need to respond
        // to the activity lifecycle
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
        // manage other components that need to respond
        // to the activity lifecycle
    }
}
```

尽管这个示例看起来很好，但在实际应用中，您最终会有太多的调用来管理UI和其他组件以响应生命周期的当前状态。
管理多个组件在生命周期方法中放置了相当数量的代码，例如onStart()和onStop()，这使得它们难以维护。

```java
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, location -> {
            // update UI
        });
    }

    @Override
    public void onStart() {
        super.onStart();
        Util.checkUserStatus(result -> {
            // what if this callback is invoked AFTER activity is stopped?
            if (result) {
                myLocationListener.start();
            }
        });
    }

    @Override
    public void onStop() {
        super.onStop();
        myLocationListener.stop();
    }
}
```

[android.arch.lifecycle](https://developer.android.google.cn/reference/android/arch/lifecycle/package-summary.html)包提供了类和接口，帮助您以一种弹性和隔离的方式来解决这些问题。

## Lifecycle

[Lifecycle](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html)是一个类，它包含组件的生命周期状态的信息(比如activity或fragment)，并允许其他对象观察这个状态。

`Lifecycle`使用两个主要枚举来跟踪其相关组件的生命周期状态:

* **Event**
	* 从框架和生命周期类中分派的lifecycle事件。
这些事件映射到activities和fragments中的回调事件。

* **State**
	* Lifecycle对象跟踪的组件的当前状态。

![lifecycle-states.png](https://developer.android.google.cn/images/topic/libraries/architecture/lifecycle-states.png)

*可以将states看作图和事件的节点，就像这些节点之间的边沿。*

一个类可以通过向其方法添加注释来监视组件的生命周期状态。
然后，您可以通过调用[Lifecycle](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html)类的[addObserver()](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html#addObserver(android.arch.lifecycle.LifecycleObserver))方法添加一个观察者，并传递一个观察者的实例，如下面的例子所示:

```java
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```

在上面的例子中，`myLifecycleOwner`对象实现了[LifecycleOwner](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html)接口，这将在下面的部分中解释。

## LifecycleOwner

[LifecycleOwner](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html)是一个表示类具有[Lifecycle](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.html)的单一方法接口。
它有一个方法[getLifecycle()](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html#getLifecycle())，它必须由类来实现。
如果您试图管理整个应用程序过程的生命周期，请参见[ProcessLifecycleOwner](https://developer.android.google.cn/reference/android.arch/lifecycle/ProcessLifecycleOwner.html)。

这个接口从单个类中抽象出Lifecycle的所有权，例如Fragment和AppCompatActivity，并允许编写与它们一起工作的组件。
任何定制的应用程序类都可以实现LifecycleOwner接口。

实现LifecycleObserver的组件与实现LifecycleOwner的组件无缝地工作，因为所有者可以提供一个lifecycle，一个观察者可以注册观察。

对于位置跟踪示例，我们可以让MyLocationListener类实现LifecycleObserver，然后用onCreate()方法中的activity生命周期初始化它。
这使得MyLocationListener类能够自给自足，这意味着在MyLocationListener中声明了对lifecycle状态变化的响应，而不是activity。
让单独的组件存储它们自己的逻辑，可以使活动和片段逻辑更容易管理。

```java
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, getLifecycle(), location -> {
            // update UI
        });
        Util.checkUserStatus(result -> {
            if (result) {
                myLocationListener.enable();
            }
        });
  }
}
```

一个常见的用例是如果生命周期处于不良好状态，就避免调用某些回调。
例如，如果回调在activity状态保存后运行一个fragment事务，它将触发崩溃，因此我们永远不希望调用那个回调。

为了使这个用例简单，Lifecycle类允许其他对象查询当前状态。

```java
class MyLocationListener implements LifecycleObserver {
    private boolean enabled = false;
    public MyLocationListener(Context context, Lifecycle lifecycle, Callback callback) {
       ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    void start() {
        if (enabled) {
           // connect
        }
    }

    public void enable() {
        enabled = true;
        if (lifecycle.getCurrentState().isAtLeast(STARTED)) {
            // connect if not connected
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    void stop() {
        // disconnect if connected
    }
}
```

有了这个实现，我们的LocationListener类完全是生命周期感知的。
如果我们需要从另一个activity或fragment中使用LocationListener，我们只需要初始化它。
所有的安装和拆卸操作都是由类本身管理的。

如果一个库提供了需要使用Android生命周期的类，我们建议您使用lifecycle组件。
您的库客户端可以轻松地集成这些组件，而无需在客户端进行手动的生命周期管理。

### 实现自定义LifecycleOwner

支持库26.1.0的Fragments和Activities已经实现了LifecycleOwner接口。

如果您有一个定制的类，您想要创建一个LifecycleOwner，您可以使用LifecycleRegistry类，但是您需要将事件转发到这个类中，如下面的代码示例所示:

```java
public class MyActivity extends Activity implements LifecycleOwner {
    private LifecycleRegistry mLifecycleRegistry;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mLifecycleRegistry = new LifecycleRegistry(this);
        mLifecycleRegistry.markState(Lifecycle.State.CREATED);
    }

    @Override
    public void onStart() {
        super.onStart();
        mLifecycleRegistry.markState(Lifecycle.State.STARTED);
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
}
```

## lifecycle组件的最佳实践

Lifecycle组件可以使您在各种情况下更容易地管理生命周期。
几个例子:

* 在粗粒度和细粒度的位置更新之间切换。
使用生命周期感知组件来支持细粒度的位置更新，而你的位置应用是可见的，当应用在后台时切换到粗粒度的更新。
LiveData，一个生命周期感知组件，允许你的应用在你的使用改变位置时自动更新UI

* 停止并启动视频缓冲。
使用生命周期感知的组件尽快启动视频缓冲，但是延迟播放直到应用程序完全启动。
您还可以使用生命周期感知组件在应用程序被销毁时终止缓冲。

* 启动和停止网络连接。
使用生命周期感知组件来支持网络数据的实时更新(流)，当应用处于前台时，当应用进入后台时自动暂停。

* 暂停并恢复动画绘图。
当应用在后台时，使用生命周期感知的组件来处理暂停的动画，当应用程序处于前台时，可以使用它。

## 处理on stop事件

当Lifecycle属于AppCompatActivity或Fragment，onSaveInstanceState()被调用时，生命周期的状态会变化到CREATED，ON_STOP事件将派发。

当一个Fragment或AppCompatActivity的状态通过onSaveInstanceState()保存时，它的UI被认为是不可变的，直到ON_START被调用。
在保存状态后尝试修改UI可能会导致应用程序的导航状态出现不一致，这就是为什么在保存状态后，如果应用程序运行了FragmentTransaction，那么FragmentManager会抛出一个异常。
有关详细信息,请参阅[commit()](https://developer.android.google.cn/reference/android/support/v4/app/FragmentTransaction.html#commit())。

如果观察者的关联Lifecycle不是STARTED，LiveData阻止这个边界情况的发生，就避免调用它的观察者。
在幕后，它调用isAtLeast()，然后决定调用它的观察者。

不幸的是，AppCompatActivity的onStop()方法是在onSaveInstanceState()之后调用的，它在UI状态更改不被允许的情况下留下了一个空白，但是Lifecycle还没有被转移到CREATED的状态。

为了防止这一问题，beta2版本中的Lifecycle类并将状态标记为CREATED，而无需发送事件，以便任何检查当前状态的代码都能获得真正的值，即使事件直到onStop()被系统调用时才会被发送。

不幸的是，这个解决方案有两个主要问题:

* 在API级别23和更低的情况下，Android系统实际上可以节省activity的状态，即使它被另一个activity所覆写。
换句话说，Android系统调用onSaveInstanceState()，但它不一定调用onStop()。
这就产生了一个潜在的很长的时间间隔，观察者仍然认为生命周期是活动的，即使它的UI状态不能被修改。

* 任何想要向LiveData类公开类似行为的类都必须实现Lifecycle beta 2版本和更低版本提供的解决方案。

> 注意:为了使这个流更简单，并提供更好的兼容性，从版本1.0-rc1开始，生命周期对象被标记为CREATED和ON_STOP已被派发，并且在调用onSaveInstanceState()时不等待对onStop()方法的调用。
这不太可能影响您的代码，但是您需要注意的是，它不符合API级别26和更低的活动类中的调用顺序。