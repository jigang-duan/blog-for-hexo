---
title: Android 架构组件 - LiveData
date: 2017-11-08 18:47:44
tags:
- Android
- 架构设计
categories:
- Android架构组件

---


<img src="https://developer.android.google.cn/topic/images/arch/icons/icons_cards_livedata.svg" width="240" height="240" align=center/>

[LiveData](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)是一个`可观察的数据持有者`类。
与常规的可观察到的不同，LiveData是`生命周期感知的`，这意味着它尊重其他应用组件的生命周期，比如activities、fragments或services。
这一认识确保了LiveData只更新了处于活动生命周期状态的应用程序组件观察者。

<!-- more -->

> 注意:要将LiveData组件导入到您的Android项目中，请参见[向您的项目添加组件](/2017/11/08/adding-components/)。

LiveData考虑一个观察者，它由[Observer](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html)类表示，如果它的生命周期处于[STARTED](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#STARTED)或[RESUMED](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#RESUMED)状态，则处于`活跃状态`。
LiveData只会通知活动的观察者关于更新的信息。
不活跃的观察者在观察[LiveData](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)对象的情况下不会被通知更改。

您可以向一个实现[LifecycleOwner](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html)接口的对象注册一个观察者。
当对应的生命周期对象的状态更改为[DESTROYED](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#DESTROYED)时，该关系允许观察者被删除。
这对于活动和片段尤其有用，因为它们可以安全地观察LiveData对象，而不用担心泄漏活动，当它们的生命周期被销毁时，它们会立即被取消订阅。

有关如何使用LiveData的更多信息，请参阅[使用LiveData对象](#work_livedata)

## 使用LiveData的优点

使用LiveData提供以下优点:

* **确保UI与数据状态匹配**

	LiveData遵循观察者模式。
当生命周期状态发生变化时，LiveData会通知观察者对象。
您可以通过合并代码来更新这些观察者对象中的UI。
每次当应用数据发生变化时，你的观察者都可以更新UI，而不是每次更新时都更新UI。

* **没有内存泄漏**

	观察者被绑定到Lifecycle对象，当他们的相关生命周期被破坏时，他们将自己清理干净。

* **不会因为停止activities而崩溃**

	如果观察者的生命周期是不活跃的，例如在后台堆栈中的activity，那么它就不会接收任何LiveData事件。

* **不用手动处理生命周期**
	
	UI组件只是观察相关数据，不用停止或恢复观察。
LiveData自动管理所有这些，因为它在观察时知道相关的生命周期状态变化。

* **一直到最新数据**
	
	如果一个生命周期变得不活跃，再次活跃它就会接收到最新的数据。例如，一个在后台的activity在它返回到前台后接收到最新的数据

* **适当的配置更改**
	
	如果一个activity或fragment由于配置更改而被重新创建，比如设备旋转，它会立即接收到最新可用的数据。

* **共享资源**
	
	您可以使用singleton模式扩展一个LiveData对象，以包装系统服务，这样它们就可以在您的应用程序中共享。
LiveData对象连接到系统服务一次，然后任何需要资源的观察者都可以看到LiveData对象。
要了解更多信息，请参阅[扩展LiveData](#extend_livedata)。

## <a name="work_livedata"></a>使用LiveData对象

按照以下步骤使用LiveData对象:

1. 创建一个`LiveData`实例来保存某种类型的数据。
这通常在您的ViewModel类中完成
1. 创建一个[Observer](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html)对象，该对象定义onChanged()方法，该方法控制在LiveData对象的数据更改时发生的情况。
您通常在UI控制器中创建一个`Observer`对象，例如一个activity或fragment。
1. 使用[observe()](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#observe(android.arch.lifecycle.LifecycleOwner,%20%20%20android.arch.lifecycle.Observer<T>))方法将观察者对象连接到LiveData对象。
`observe()`方法接受一个LifecycleOwner对象。
它将`Observer`对象订阅到LiveData对象，以便通知更改。
您通常将`Observer`对象附加在UI控制器中，例如activity或fragment。

> 注意:您可以使用observeForever(Observer)方法注册一个没有关联的LifecycleOwner对象的观察者。
在这种情况下，观察者被认为总是处于活动状态，因此总是被告知修改。
您可以调用removeObserver(Observer)方法删除观察器。

当您更新存储在`LiveData`对象中的值时，只要附加的`LifecycleOwner`处于活跃状态，它就会触发所有已注册的观察者。

LiveData允许用户界面控制器的观察者订阅更新。
当`LiveData`对象所持有的数据发生变化时，UI会自动更新响应。

### 创建LiveData对象

LiveData是一个可以用于任何数据的包装器，包括实现[Collections](https://developer.android.google.cn/reference/java/util/Collections.html)的对象，如[List](https://developer.android.google.cn/reference/java/util/List.html)。
LiveData对象通常存储在[ViewModel](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html)对象中，并通过getter方法访问，如下面的示例所示:

```java
public class NameViewModel extends ViewModel {

// Create a LiveData with a String
private MutableLiveData<String> mCurrentName;

    public MutableLiveData<String> getCurrentName() {
        if (mCurrentName == null) {
            mCurrentName = new MutableLiveData<String>();
        }
        return mCurrentName;
    }

// Rest of the ViewModel...
}
```

最初，`LiveData`对象中的数据没有设置。

> 注意:确保存储LiveData对象在ViewModel对象中更新UI，而不是activity或fragment，原因如下:
> 
* 避免膨胀的活动和碎片。现在，这些UI控制器负责显示数据，而不是保存数据状态。
* 将LiveData实例与特定活动或片段实例分离，并允许LiveData对象在配置更改中存活。

您可以在[ViewModel指南](https://developer.android.google.cn/topic/libraries/architecture/viewmodel.html)中更多地了解`ViewModel`类的好处和使用情况

### 观察LiveData对象

在大多数情况下，应用程序组件的[onCreate()](https://developer.android.google.cn/reference/android/app/Activity.html#onCreate(android.os.Bundle))方法是开始观察LiveData对象的正确位置，原因如下:

* 确保系统不会从activity或fragment的onResume()方法中发出冗余的调用。

* 为了确保activity或fragment拥有数据，它可以在活动开始时显示。
一旦应用程序组件处于[STARTED](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#STARTED)状态，它就会从它所观察到的LiveData对象中接收到最新的值。
只有在已经设置了`LiveData`对象时才会出现这种情况。

一般来说，LiveData只在数据发生变化时才提供更新，而且只对活动的观察者进行更新。
这种行为的一个例外是，当观察者从一个非活动状态转变为一个活动状态时，也会收到一个更新。
此外，如果观察者第二次从非活动状态变为活动状态，它只会收到一个更新，如果该值在上一次激活时发生了变化。

下面的示例代码演示了如何开始观察一个LiveData对象:

```java
public class NameActivity extends AppCompatActivity {

    private NameViewModel mModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Other code to setup the activity...

        // Get the ViewModel.
        mModel = ViewModelProviders.of(this).get(NameViewModel.class);

        // Create the observer which updates the UI.
        final Observer<String> nameObserver = new Observer<String>() {
            @Override
            public void onChanged(@Nullable final String newName) {
                // Update the UI, in this case, a TextView.
                mNameTextView.setText(newName);
            }
        };

        // Observe the LiveData, passing in this activity as the LifecycleOwner and the observer.
        mModel.getCurrentName().observe(this, nameObserver);
    }
}
```

在调用了[observe()](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#observe(android.arch.lifecycle.LifecycleOwner,android.arch.lifecycle.Observer<T>))之后，`nameObserver`被作为参数传递，[onChanged()](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html#onChanged(T))立即被调用，提供了当前在`mCurrentName`中存储的最新值。
如果`LiveData`对象没有在`mCurrentName`中设置值，则不会调用onChanged()。

### 更新LiveData对象

LiveData没有公开可用的方法来更新存储的数据。
[MutableLiveData](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html)类公开了[setValue(T)](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html#setValue(T))和[postValue(T)](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html#postValue(T))方法，如果需要编辑存储在LiveData对象中的值，则必须使用这些方法。
通常在[ViewModel](https://developer.android.google.cn/reference/android/arch/lifecycle/ViewModel.html)中使用`MutableLiveData`，然后`ViewModel`只向观察者公开不可变的`LiveData`对象。

在设置了观察者关系之后，您可以更新LiveData对象的值，如下面的例子所示，当用户点击一个按钮时，它会触发所有观察者:

```java
mButton.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        String anotherName = "John Doe";
        mModel.getCurrentName().setValue(anotherName);
    }
});
```

在示例中调用`setValue(T)`的结果是观察者 被调用了有`John Doe`值的`onChanged()`方法 。
示例显示了按钮按下，但是`setValue()`或`postValue()`的被调用可以是各种更新`mName`的原因，包括响应网络请求或数据库加载完成;在所有情况下，对setValue()或postValue()的调用会`触发观察者`并更新UI。

> 注意:您必须调用`setValue(T)`方法来从`主线程`更新LiveData对象。
如果代码是在一个`工作线程`中执行的，那么您可以使用`postValue(T)`方法来更新LiveData对象。

### 使用LiveData Room

[Room](https://developer.android.google.cn/training/data-storage/room/index.html)持久化库支持可观察的查询，这些查询返回了LiveData对象。
可观察的查询是作为数据库访问对象(DAO)的一部分编写的。

当数据库被更新时，Room会生成所有必要的代码来更新LiveData对象。
当需要时，生成的代码在后台线程上异步地运行查询。
此模式有助于将数据显示在UI中，与存储在数据库中的数据保持同步。
您可以在[Room持久化库指南]()中了解更多关于Room和DAO的内容。

## <a name="extend_livedata"></a>扩展LiveData

LiveData认为，如果观察者的生命周期处于[STARTED](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#STARTED)或[RESUMED](https://developer.android.google.cn/reference/android/arch/lifecycle/Lifecycle.State.html#RESUMED)状态，那么观察者处于活跃状态，以下示例代码说明了如何扩展[LiveData](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html)类:

```java
public class StockLiveData extends LiveData<BigDecimal> {
    private StockManager mStockManager;

    private SimplePriceListener mListener = new SimplePriceListener() {
        @Override
        public void onPriceChanged(BigDecimal price) {
            setValue(price);
        }
    };

    public StockLiveData(String symbol) {
        mStockManager = new StockManager(symbol);
    }

    @Override
    protected void onActive() {
        mStockManager.requestPriceUpdates(mListener);
    }

    @Override
    protected void onInactive() {
        mStockManager.removeUpdates(mListener);
    }
}
```

在本例中，价格listener的实现包括以下重要方法:

* [onActive()](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#onActive())方法调用，当`LiveData`对象有一个活跃的观察者时。这意味着您需要从这个方法开始观察股票价格的更新。
* [onInactive()](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#onInactive())方法调用，当`LiveData`对象没有任何活跃的观察者时。由于没有观察人员在监听，因此没有理由与`StockManager`服务保持联系。
* [setValue(T)](https://developer.android.google.cn/reference/android/arch/lifecycle/MutableLiveData.html#setValue(T))方法更新`LiveData`实例的值，并通知任何活跃的观察者关于这个变化。

您可以使用`StockLiveData`类如下:

```java
public class MyFragment extends Fragment {
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        LiveData<BigDecimal> myPriceListener = ...;
        myPriceListener.observe(this, price -> {
            // Update the UI.
        });
    }
}
```

[observe()](https://developer.android.google.cn/reference/android/arch/lifecycle/LiveData.html#observe(android.arch.lifecycle.LifecycleOwner,android.arch.lifecycle.Observer<T>))方法将片断作为第一个参数传递，它是[LifecycleOwner](https://developer.android.google.cn/reference/android/arch/lifecycle/LifecycleOwner.html)的一个实例。
这样做意味着这个观察者被绑定到与所有者相关的`Lifecycle对象`，这意味着:

* 如果`Lifecycle对象`不是处于活跃状态，那么即使值发生变化，也不会调用观察者。
* 在`Lifecycle对象`被销毁之后，观察者会被自动删除。

LiveData对象是生命周期感知的事实，这意味着您可以在多个activities、fragments和services之间共享它们。
为了保持这个示例的简单性，您可以将`LiveData`类作为一个单例对象来实现:

```java
public class StockLiveData extends LiveData<BigDecimal> {
    private static StockLiveData sInstance;
    private StockManager mStockManager;

    private SimplePriceListener mListener = new SimplePriceListener() {
        @Override
        public void onPriceChanged(BigDecimal price) {
            setValue(price);
        }
    };

    @MainThread
    public static StockLiveData get(String symbol) {
        if (sInstance == null) {
            sInstance = new StockLiveData(symbol);
        }
        return sInstance;
    }

    private StockLiveData(String symbol) {
        mStockManager = new StockManager(symbol);
    }

    @Override
    protected void onActive() {
        mStockManager.requestPriceUpdates(mListener);
    }

    @Override
    protected void onInactive() {
        mStockManager.removeUpdates(mListener);
    }
}
```

你可以在fragment中使用它，如下:

```java
public class MyFragment extends Fragment {
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        StockLiveData.get(getActivity()).observe(this, price -> {
            // Update the UI.
        });
    }
}
```

多个fragments和activities可以观察到`MyPriceListener`实例。
如果其中一个或多个具有可见性和活跃性，LiveData只连接到系统服务。

## 变换LiveData

您可能想要对LiveData对象中存储的值进行更改，然后将其发送给观察者，或者您可能需要根据另一个对象的值返回一个不同的LiveData实例。
[Lifecycle](https://developer.android.google.cn/reference/android/arch/lifecycle/package-summary.html)包提供了[Transformations](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html)类，其中包括支持这些场景的辅助方法。

* [Transformations.map()](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html#map(android.arch.lifecycle.LiveData<X>,%20android.arch.core.util.Function<X,%20Y>))

	对存储在LiveData对象中的值应用一个函数，并将结果传播到下游。
	
	```java
	LiveData<User> userLiveData = ...;
	LiveData<String> userName = Transformations.map(userLiveData, user -> {
    	user.name + " " + user.lastName
	});
	```
	
* [Transformations.switchMap()](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html#switchMap(android.arch.lifecycle.LiveData<X>,%20android.arch.core.util.Function<X,%20android.arch.lifecycle.LiveData<Y>>))

	与map()类似，将一个函数应用到LiveData对象中存储的值，并将结果发送到下游。
传递给switchMap()的函数必须返回一个LiveData对象，如下面的例子所示:

	```java
	private LiveData<User> getUser(String id) {
  		...;
	}

	LiveData<String> userId = ...;
	LiveData<User> user = Transformations.switchMap(userId, id -> getUser(id) );
	```
	
您可以使用转换方法在观察者的生命周期中携带信息。
除非观察者在观察返回的LiveData对象，否则这些转换不会被计算。
因为`转换是惰性地计算的`，所以与生命周期相关的行为在不需要额外的显式调用或依赖项的情况下被隐式地传递下去。

如果您认为在ViewModel对象中需要一个Lifecycle对象，那么转换可能是更好的解决方案。
例如，假设您有一个UI组件，该组件接受一个地址，并返回该地址的邮政编码。
您可以为该组件实现简单的ViewModel，如下示例代码所示:

```java
class MyViewModel extends ViewModel {
    private final PostalCodeRepository repository;
    public MyViewModel(PostalCodeRepository repository) {
       this.repository = repository;
    }

    private LiveData<String> getPostalCode(String address) {
       // DON'T DO THIS
       return repository.getPostCode(address);
    }
}
```

然后，UI组件需要从先前的LiveData对象中*取消注册*，并在每次调用getPostalCode()时*注册新实例*。
此外，如果UI组件被重新创建，它将触发另一个`repository.getPostCode()`方法的调用，而不是使用以前的调用的结果

相反，您可以将邮政编码查询作为地址输入的转换实现，如下面的示例所示:

```java
class MyViewModel extends ViewModel {
    private final PostalCodeRepository repository;
    private final MutableLiveData<String> addressInput = new MutableLiveData();
    public final LiveData<String> postalCode =
            Transformations.switchMap(addressInput, (address) -> {
                return repository.getPostCode(address);
             });

  public MyViewModel(PostalCodeRepository repository) {
      this.repository = repository
  }

  private void setInput(String address) {
      addressInput.setValue(address);
  }
}
```

在本例中，`postalCode`字段是`public`的和`final`，因为字段永远不会更改。
`postalCode`字段定义为`addressInput`的转换，这意味着在`addressInput`更改时调用`repository.getPostCode()`方法。
如果有一个活动的观察者，如果在`repository.getPostCode()`被调用时没有活动的观察者，那么在观察者被添加之前没有计算。

该机制允许应用程序在较低级别创建基于需求的*延迟计算的LiveData对象*。
ViewModel对象可以很容易地获得对LiveData对象的引用，然后在它们上面定义转换规则。

### 创建新的转换

在你的应用中有十几个不同的具体的转换，但它们不是默认提供的。
为了实现您自己的转换，您可以使用[MediatorLiveData](https://developer.android.google.cn/reference/android/arch/lifecycle/MediatorLiveData.html)类，它侦听其他的LiveData对象，并处理它们发出的事件。
MediatorLiveData正确地将其状态传播到源LiveData对象。
要了解更多关于此模式的信息，请参阅[Transformations](https://developer.android.google.cn/reference/android/arch/lifecycle/Transformations.html)类的参考文档。

## 合并多个LiveData源

[MediatorLiveData](https://developer.android.google.cn/reference/android/arch/lifecycle/MediatorLiveData.html)是LiveData的一个子类，它允许您合并多个live数据源。
当任何原始的LiveData源对象发生变化时，MediatorLiveData对象的观察者就会被触发。

例如，如果您的UI中有一个LiveData对象，可以从本地数据库或网络更新，那么您可以将以下源添加到MediatorLiveData对象:

* 与存储在数据库中的数据相关联的LiveData对象。
* 与从网络访问的数据相关联的LiveData对象。

您的activity只需要观察MediatorLiveData对象来接收来自两个源的更新。
对于一个详细的示例，请参阅 [应用程序架构指南](/2017/11/08/GuidetoAppArchitecture/)的[附录:网络状态](/2017/11/08/GuidetoAppArchitecture/#addendum)部分。