---
title: Android 架构组件指南
date: 2017-11-08 11:23:22
tags:
- Android
- 架构设计
categories:
- Android架构组件

---


![arch-hero.png](https://developer.android.google.cn/topic/images/arch/hero/arch-hero.png)

本指南适用于开发app的开发人员，现在希望了解`最佳实践`和推荐的`体系结构`，构建健壮、高质量的apps。

> 注意:本指南假定读者熟悉Android框架。
> 如果你是应用开发的新手，请查看[入门培训](https://developer.android.google.cn/training/index.html)系列，其中包含了本指南的必备主题。

<!-- more -->

## app开发人员面临的常见问题 ##

与传统的桌面应用不同，在大多数情况下，它们都有一个单一入口点，并作为一个单一的整体流程运行，而Android应用程序的结构要复杂得多。
一个典型的Android应用程序是由多个[应用组件](https://developer.android.google.cn/guide/components/fundamentals.html#Components)组成的，包括activities、fragments、services、content providers和broadcast receivers。

大部分应用程序组件都是在应用程序清单中声明的，Android操作系统使用这些组件来决定如何将你的应用集成到用户体验中。
正如前面提到的，桌面应用程序传统上是一个单一的过程，一个正确编写的Android应用程序需要更加灵活，因为用户可以通过设备上的不同应用程序，不断地切换流量和任务。

例如，考虑一下当你在你最喜欢的社交网络应用中分享一张照片时会发生什么。
这款应用会触发一个摄像头的intent，Android操作系统会启动一个摄像头应用来处理这个请求。
在这一点上，用户离开了社交网络应用，但他们的体验是无缝的。
反过来，摄像头应用可能会触发其他intents，比如启动文件选择器，这可能会启动另一个应用。
最终用户会回到社交网络应用，分享照片。
另外，用户可以在这个过程的任何时候被一个电话打断，然后在完成电话呼叫后返回来分享照片。

在Android中，这种应用程序的行为是很常见的，所以你的应用程序必须正确处理这些流程。
请记住，移动设备是资源受限的，因此在任何时候，操作系统可能需要杀死一些应用程序来为新的应用程序腾出空间。

这一切的关键在于，你的应用程序组件可以单独启动，也可以不按顺序启动，可以随时被用户或系统摧毁。
因为应用组件是短暂的，它们的生命周期(当它们被创建和销毁时)不在你的控制之下，**_你不应该在应用程序组件中存储任何应用数据或状态_**，而应用程序组件不应该相互依赖。

## 常见的架构原则 ##

如果你不能使用应用程序组件来存储应用数据和状态，那么应用程序应该如何构建呢?

* 你应该关注的最重要的事情是你的应用中[关注点的分离](https://en.wikipedia.org/wiki/Separation_of_concerns)。

在Activity或Fragment中编写所有代码是一个常见的错误。
任何不处理UI或操作系统交互的代码都不应该在这些类中。
让他们尽可能地精简，这样可以避免许多与生命周期相关的问题。
不要忘记你不拥有这些类，它们只是在操作系统和你的应用之间体现契约的胶类。
Android操作系统可能会在任何时候基于用户交互或低内存等其他因素破坏它们。
最好是减少对它们的依赖，以提供可靠的用户体验。

* 第二个重要原则是，应该**`模型驱动UI`**，最好是持久模型。

持久性是理想的两个原因:如果操作系统破坏了你的应用程序来释放资源，你的用户不会丢失数据，即使网络连接很脆弱或没有连接，你的应用也会继续工作。
模型是负责处理应用程序数据的组件。
它们独立于应用程序中的视图和应用程序组件，因此它们与这些组件的生命周期问题是隔离的。
保持UI代码的简单性和应用程序逻辑的自由使管理变得更加容易。
将你的应用程序建立在具有明确责任管理数据的模型类上，这将使它们具有可测试性，并且你的应用程序是一致的。

## <a name="recommended_app_architecture"></a>推荐应用架构 ##

在本节中，我们将演示如何通过使用用例来构造应用程序的[结构组件](/2017/11/08/AndroidArchitectectureComponents/)。

> 注意:要有一种编写应用程序所有场景中最好的方式是不可能的。
> 也就是说，对于大多数用例来说，推荐的体系结构应该是一个良好的起点。
> 如果你已经有了编写Android应用程序的好方法，你就不需要改变了。

假设我们正在构建一个显示用户配置文件的UI。
这个用户配置文件将从我们自己的私有后端使用REST API获取。

### 构建用户界面

UI将由一个fragment `UserProfileFragment.java` 及其相应的布局文件`user_profile_layout.xml`组成。

为了驱动UI，我们的数据模型需要保存两个数据元素。

* **用户ID**: 用户的标识符。
最好是使用fragment参数将这些信息传递到fragment中。
如果Android操作系统破坏了你的进程，那么这个信息就会被保存下来，所以当你的应用重新启动时，id还可以使用。
* **User对象**: 保存用户数据的POJO。

我们将基于**ViewModel**类创建一个`UserProfileViewModel`，保存此信息。

[ViewModel](https://developer.android.google.cn/topic/libraries/architecture/viewmodel.html)为特定的UI组件提供数据，例如fragment或activity，并处理与数据处理业务部分的通信，例如调用其他组件来加载数据或转发用户修改。
ViewModel不知道视图，也不受配置更改的影响，比如由于旋转造成的重新创建一个activity。

现在我们有3个文件：

* `user_profile.xml`: 屏幕的UI定义
* `UserProfileViewModel.java`: 为UI准备数据类
* `UserProfileFragment.java`: 在ViewModel中显示数据并对用户交互作出反应的UI控制器

下面开始我们的实现(为了简单起见，省略布局文件):

```java
public class UserProfileViewModel extends ViewModel {
    private String userId;
    private User user;

    public void init(String userId) {
        this.userId = userId;
    }
    public User getUser() {
        return user;
    }
}
```

```java
public class UserProfileFragment extends Fragment {
    private static final String UID_KEY = "uid";
    private UserProfileViewModel viewModel;

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        String userId = getArguments().getString(UID_KEY);
        viewModel = ViewModelProviders.of(this).get(UserProfileViewModel.class);
        viewModel.init(userId);
    }

    @Override
    public View onCreateView(LayoutInflater inflater,
                @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.user_profile, container, false);
    }
}
```

现在，我们有了这三个代码模块，我们如何连接它们?
毕竟，当ViewModel的user字段被设置时，我们需要一种方法来通知UI。
这就是LiveData类的用武之地。

[LiveData](https://developer.android.google.cn/topic/libraries/architecture/livedata.html)是一个可观察的数据持有者。
它让应用程序中的组件可以在不创建显式和严格的依赖路径的情况下，观察LiveData对象的变化。
LiveData还尊重应用程序组件(activities、fragments、services)的生命周期状态，并做正确的事情来防止对象泄漏，这样您的应用程序就不会消耗更多的内存

> 注意:如果您已经在使用[RxJava](https://github.com/ReactiveX/RxJava)或[Agera](https://github.com/google/agera)这样的库，您可以继续使用它们，而不是使用LiveData。
但是当您使用它们或其他方法时，请确保您正确地处理生命周期，这样当相关的生命周期所有者被停止时，您的数据流就会暂停，而当生命周期所有者被销毁时，流就会被销毁。
您还可以添加android.arch.lifecycle:reactivestreams部件以使用LiveData与另一个响应式流库(例如RxJava2)。

现在，我们在`UserProfileViewModel`中使用`LiveData<User>`字段替换`User`字段，以便在数据更新时可以通知fragment。
LiveData最重要的一点是生命周期感知，当不再需要时，它将自动清除引用。

```java
public class UserProfileViewModel extends ViewModel {
    ...
    //private User user;
    private LiveData<User> user;
    public LiveData<User> getUser() {
        return user;
    }
}
```

现在，我们修改`UserProfileFragment`来观察数据并更新UI。

```java
@Override
public void onActivityCreated(@Nullable Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    viewModel.getUser().observe(this, user -> {
      // update UI
    });
}
```

每次更新用户数据时，都会调用[onChanged](https://developer.android.google.cn/reference/android/arch/lifecycle/Observer.html#onChanged(T))的回调，并刷新UI。

如果您熟悉使用observable回调函数的其他库，您可能已经意识到，我们不必覆写片段的onStop()方法来停止观察数据。
这对于LiveData来说是不必要的，因为它是生命周期感知的，这意味着除非fragment处于活动状态(接收到onStart()，但是没有接收onStop())，否则它将不会调用回调。
当片段接收到onDestroy()时，LiveData也会自动删除观察者。

我们也没有做任何特别的事情来处理配置更改(例如，用户旋转屏幕)。
当配置发生变化时，ViewModel会自动恢复，因此当新fragment出现时，它将接收到相同的ViewModel实例，并且将立即使用当前数据调用回调。
这就是viewmodel不直接引用视图的原因;它们可以比视图的生命周期活的长。
[查看一个ViewModel的生命周期](https://developer.android.google.cn/topic/libraries/architecture/viewmodel.html#the_lifecycle_of_a_viewmodel)。

### 获取数据

现在我们已经将ViewModel连接到fragment，但是ViewModel如何获取用户数据呢?
在本例中，我们假设我们的后端提供了一个REST API。
我们将使用[Retrofit](http://square.github.io/retrofit/)库来访问我们的后端，尽管您可以使用不同的库来实现相同的目的。

这是我们的retrofit `Webservice`与我们的后端通信:

```java
public interface Webservice {
    /**
     * @GET 声明一个HTTP GET请求
     * @Path("user") 在userId参数上的注释
     * 替换 @GET路径中{user}占位符
     */
    @GET("/users/{user}")
    Call<User> getUser(@Path("user") String userId);
}
```

ViewModel的一个幼稚的实现是直接调用Webservice来获取数据并将其返回给用户对象。
即使它很好用，但你的应用也很难在成长的过程中维护。
它给ViewModel类带来了太多的责任，这违背了我们之前提到的__关注点分离原则__。
此外，ViewModel的范围与Activity或Fragment生命周期相关联，因此在其生命周期结束时丢失所有数据是一个糟糕的用户体验。
相反，我们的ViewModel将把这个工作委托给一个新的`Repository`模块。

**`Repository`**模块 负责处理数据操作。
他们为应用程序的其余部分提供了一个干净的API，他们知道从哪里获取数据，以及在数据更新时要做什么API调用。
您可以将它们视为不同数据源之间的中介(持久模型、web服务、缓存等)。

下面的`UserRepository`类使用`WebService`来获取用户数据项。

```java
public class UserRepository {
    private Webservice webservice;
    // ...
    public LiveData<User> getUser(int userId) {
        // This is not an optimal implementation, we'll fix it below
        final MutableLiveData<User> data = new MutableLiveData<>();
        webservice.getUser(userId).enqueue(new Callback<User>() {
            @Override
            public void onResponse(Call<User> call, Response<User> response) {
                // error case is left out for brevity
                data.setValue(response.body());
            }
        });
        return data;
    }
}
```

尽管Repository模块看起来没有必要，但它有一个重要的目的;
它从应用的其余部分中提取数据源。
现在，我们的ViewModel不知道数据是由`Webservice`获取的，这意味着我们可以根据需要将其交换到其他实现。

> 注意:为了简单起见，我们省略了网络错误的例子。
> 对于暴露错误和加载状态的另一种实现，请参阅[附录:暴露网络状态](#addendum)。

#### 管理组件之间的依赖关系:

上面的`UserRepository`类需要一个`Webservice`的实例来完成它的工作。
它可以简单地创建它，但是要这样做，构造它还需要知道`Webservice`类的依赖项。
这将大大复杂化并复制代码(例如，需要一个`Webservice`实例的每个类都需要知道如何使用它的依赖项来构造它)。
此外，`UserRepository`可能不是唯一需要`Webservice`的类。
如果每个类都创建一个新的`Webservice`，那么它将是非常资源非常重的。

有两类模式可以用来解决这个问题:

* [依赖注入](https://en.wikipedia.org/wiki/Dependency_injection): 依赖注入允许类在不构造它们的情况下定义它们的依赖关系。
在运行时，另一个类负责提供这些依赖项。
我们推荐Google的[Dagger 2](https://google.github.io/dagger/)库，用于在Android应用中实现依赖注入。
Dagger 2通过使用依赖树来自动构造对象，并提供对依赖项的编译时间保证

* [服务定位器](https://en.wikipedia.org/wiki/Service_locator_pattern): 服务定位器提供一个注册表，在这个注册表中，类可以获得它们的依赖，而不是构建它们。
它比依赖项注入(DI)要容易得多，所以如果您不熟悉DI，那么使用服务定位器代替。

这些模式允许您扩展您的代码，因为它们为管理依赖关系提供了清晰的模式，而不需要复制代码或增加复杂性。
它们都允许交换实现进行测试;这是使用它们的主要好处之一。

在本例中，我们将使用`Dagger 2`来管理依赖关系。

### 连接ViewModel和Repository

现在，我们来使用repository修改`UserProfileViewModel`。

```java
public class UserProfileViewModel extends ViewModel {
    private LiveData<User> user;
    private UserRepository userRepo;

    @Inject // UserRepository参数由Dagger 2提供
    public UserProfileViewModel(UserRepository userRepo) {
        this.userRepo = userRepo;
    }

    public void init(String userId) {
        if (this.user != null) {
            // ViewModel是每个片段Fragment的，因此我们知道userId不会改变
            return;
        }
        user = userRepo.getUser(userId);
    }

    public LiveData<User> getUser() {
        return this.user;
    }
}

```

### 缓存数据

上面的repository实现很好地提取了对web服务的调用，但是因为它只依赖于一个数据源，所以它不是很有用。

上面的`UserRepository`实现的问题是，在获取数据之后，它并没有将数据保存在任何地方。
如果用户离开`UserProfileFragment`并返回到它，该应用程序将重新获取数据。
这有两个原因:它浪费了宝贵的网络带宽，并迫使用户等待新的查询完成。
为了解决这个问题，我们将向我们的`UserRepository`添加一个新的数据源，该数据源将在内存中缓存`User`对象。

```java
@Singleton  // 通知Dagger，这个类应该被构造一次
public class UserRepository {
    private Webservice webservice;
    // 在内存缓存中很简单，为了简洁而省略了细节
    private UserCache userCache;
    public LiveData<User> getUser(String userId) {
        LiveData<User> cached = userCache.get(userId);
        if (cached != null) {
            return cached;
        }

        final MutableLiveData<User> data = new MutableLiveData<>();
        userCache.put(userId, data);
        // 这仍然是次优的，但比以前更好.
        // 一个完整的实现还必须处理错误案例.
        webservice.getUser(userId).enqueue(new Callback<User>() {
            @Override
            public void onResponse(Call<User> call, Response<User> response) {
                data.setValue(response.body());
            }
        });
        return data;
    }
}

```

### 持久化数据

在我们当前的实现中，如果用户旋转屏幕或离开并返回应用程序，那么现有的UI将立即可见，因为repository从内存缓存中检索数据。
但是，如果用户离开了应用，几个小时后又回来了，在Android操作系统扼杀了这个过程之后，又会发生什么呢?

使用当前的实现，我们将需要从网络获取数据。
这不仅是一种糟糕的用户体验，而且也是一种浪费，因为它将使用移动数据来重新获取相同的数据。
您可以简单地通过缓存web请求来解决这个问题，但是它会产生新的问题。
如果相同的用户数据来自另一种类型的请求(例如，获取好友列表)，会发生什么情况?
然后，你的应用程序可能会显示不一致的数据，这是最令人困惑的用户体验。
例如，相同用户的数据可能会以不同的方式出现，因为好友请求和用户请求可以在不同的时间执行。
你的应用需要将它们合并，以避免显示不一致的数据。

处理这个问题的正确方法是使用一个持久化模型。这就是[Room](https://developer.android.google.cn/training/data-storage/room/index.html)持久化库来拯救的地方。

[Room]()是一个对象映射库，它为本地数据持久化提供了最小的样板代码。
在编译时，它会根据模式验证每个查询，因此，损坏的SQL查询会导致编译时错误，而不是运行时失败。
Room抽象了处理原始SQL表和查询的一些底层实现细节。
它还允许对数据库数据进行更改(包括集合和连接查询)，通过LiveData对象公开这些更改。
此外，它还显式地定义了解决常见问题的线程约束，例如访问主线程上的存储。

> 注意:如果您的应用程序已经使用了另一个持久性解决方案，比如SQLite对象关系映射(ORM)，那么您就不需要用Room来替换现有的解决方案了。
然而，如果你正在写一款新应用或重构现有的应用，我们推荐使用房间来保存你的应用的数据。
这样，您就可以利用该库的抽象和查询验证功能。

使用Room，我们需要定义我们的本地模式。
首先，用[@Entity](https://developer.android.google.cn/reference/android/arch/persistence/room/Entity.html)注释User类，将其标记为数据库中的一个表。

```java
@Entity
class User {
  @PrimaryKey
  private int id;
  private String name;
  private String lastName;
  // getters and setters for fields
}
```

然后，通过为应用程序扩展[RoomDatabase](https://developer.android.google.cn/reference/android/arch/persistence/room/RoomDatabase.html)来创建一个数据库类:

```java
@Database(entities = {User.class}, version = 1)
public abstract class MyDatabase extends RoomDatabase {
}
```

> 注意，`MyDatabase`是抽象的。
Room自动提供它的实现。
有关详细信息，请参阅[Room](https://developer.android.google.cn/topic/libraries/architecture/room.html)文档。

现在我们需要一种方法来将用户数据插入到数据库中。
为此，我们将创建一个[数据访问对象(DAO)](https://en.wikipedia.org/wiki/Data_access_object)。

```java
@Dao
public interface UserDao {
    @Insert(onConflict = REPLACE)
    void save(User user);
    @Query("SELECT * FROM user WHERE id = :userId")
    LiveData<User> load(String userId);
}
```

然后，从我们的数据库类引用DAO。

```java
@Database(entities = {User.class}, version = 1)
public abstract class MyDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

请注意，load方法返回一个`LiveData<User>`。
Room知道数据库何时被修改，当数据发生变化时，它会自动通知所有的活动观察者。
因为它使用*LiveData*，所以这将是有效的，因为只有在至少有一个活动观察者的情况下，它才会更新数据。

> 注意:基于表修改的Room检查失效，这意味着它可能会发送错误的正向通知。

现在，我们可以修改我们的`UserRepository`来合并*Room*数据源。

```java
@Singleton
public class UserRepository {
    private final Webservice webservice;
    private final UserDao userDao;
    private final Executor executor;

    @Inject
    public UserRepository(Webservice webservice, UserDao userDao, Executor executor) {
        this.webservice = webservice;
        this.userDao = userDao;
        this.executor = executor;
    }

    public LiveData<User> getUser(String userId) {
        refreshUser(userId);
        // 从数据库直接返回一个LiveData.
        return userDao.load(userId);
    }

    private void refreshUser(final String userId) {
        executor.execute(() -> {
            // 在后台线程中运行
            // 检查用户最近是否被获取
            boolean userExists = userDao.hasUser(FRESH_TIMEOUT);
            if (!userExists) {
                // 刷新数据
                Response response = webservice.getUser(userId).execute();
                // TODO 检查错误等.
                // 更新数据库。LiveData会自动刷新
                // 除了更新数据库之外，我们不需要做其他任何事情
                userDao.save(response.body());
            }
        });
    }
}
```

请注意，即使我们更改了来自UserRepository的数据，我们也不需要更改UserProfileViewModel或UserProfileFragment。
这是抽象提供的灵活性。
这对于测试来说也很好，因为您可以在测试您的UserProfileViewModel时提供一个假的UserRepository。

现在我们的代码已经完成了。
如果用户在几天后返回相同的UI，他们将立即看到用户信息，因为我们已经持久化了它。
同时，如果数据陈旧，我们的存储库将更新背景中的数据。
当然，根据您的用例，如果它太老了，您可能不愿意显示持久的数据。

在一些用例中，例如“下拉刷新”，如果当前有网络操作正在进行中，那么UI向用户显示是很重要的。
将UI操作与实际数据分开是一种很好的做法，因为它可能会因为各种原因而被更新(例如，如果我们获取一个朋友列表，同样的用户可能会被再次获取，从而触发一个LiveData<User>更新)。
从UI的角度来看，在飞行中有请求的事实只是另一个数据点，类似于任何其他的片段数据(比如User对象)。

对于这个用例有两种常见的解决方案:

* 更改getUser以返回包含网络操作状态的LiveData。
提供了一个示例实现:[附录:暴露网络状态](#addendum) 部分。
* 在repository类中提供另一个public函数，可以返回用户的刷新状态。
如果您想在UI中显示网络状态，只在对显式的用户操作(如拉到刷新)的响应中显示网络状态，那么这个选项就更好了。

#### <a name="truth"></a>单一来源的真相

不同REST API端点返回相同的数据是很常见的。
例如，如果我们的后端有另一个返回好友列表的端点，那么相同的用户对象可能来自两个不同的API端点，可能是不同的粒度。
如果UserRepository按原样返回来自Webservice请求的响应，那么我们的UIs可能会显示不一致的数据，因为数据可能会在这些请求之间的服务器端发生变化。
这就是为什么在UserRepository实现中，Webservice回调只是将数据保存到数据库中。
然后，对数据库的更改将触发对LiveData对象回调的活动。

在这个模型中，`数据库作为唯一的真理来源`，并且应用程序的其他部分通过存储库访问它。
不管您是否使用磁盘缓存，我们建议您将存储库数据源指定为您应用程序其余部分的唯一来源。

### 测试

我们已经提到分离的好处之一是`可测试性`。
让我们看看如何测试每个代码模块。

* **用户界面和交互**: 这将是你唯一需要[Android UI Instrumentation test](https://developer.android.google.cn/training/testing/unit-testing/instrumented-unit-tests.html)的时候。
测试UI代码的最佳方法是创建一个[Espresso](https://developer.android.google.cn/training/testing/ui-testing/espresso-testing.html)测试。
您可以创建fragment并提供一个mock ViewModel。
由于fragment只与ViewModel进行对话，因此对其进行模拟将足以充分测试此UI。

* **ViewModel**: 可以使用[JUnit](https://developer.android.google.cn/training/testing/unit-testing/local-unit-tests.html)对ViewModel进行测试。
您只需要模拟`UserRepository`来测试它。

* **UserRepository**: 您还可以使用JUnit来测试UserRepository。
您需要模拟Webservice和DAO。
您可以测试它是否提供了正确的Webservice调用，将结果保存到数据库中，并且如果数据被缓存和更新，就不会产生任何不必要的请求。
由于Webservice和UserDao都是接口，所以您可以模拟它们，或者为更复杂的测试用例创建假的实现。

* **UserDao**: 测试DAO类的推荐方法是使用instrumentation测试。
由于这些instrumentation测试不需要任何UI，所以它们仍然会运行得很快。
对于每一个测试，您都可以创建一个内存中的数据库，以确保测试没有任何副作用(比如更改磁盘上的数据库文件)。

	*Room还允许指定数据库实现，这样您就可以通过提供[SupportSQLiteOpenHelper](https://developer.android.google.cn/reference/android/arch/persistence/db/SupportSQLiteOpenHelper.html)的JUnit实现来测试它。
通常不建议采用这种方法，因为在设备上运行的SQLite版本可能与主机上的SQLite版本不同。*

* **Webservice**: 重要的是要使测试独立于外部世界，因此即使您的Webservice测试也应该避免对后端进行网络调用。
有很多库可以帮助解决这个问题。
例如，[MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver)是一个很棒的库，它可以帮助您为测试创建一个假的本地服务器。

* **测试工件**: `架构组件`提供了一个maven工件来控制它的后台线程。
在`android.arch.core:core-testing`工件，有两个JUnit规则:
	* `InstantTaskExecutorRule`: 此规则可用于强制体系结构组件立即执行调用线程上的任何后台操作。
	* `CountingTaskExecutorRule`: 这个规则可以在*instrumentation*测试中使用，等待架构组件的后台操作，或者将它作为一个空闲的资源连接到*Espresso*。

### 最终的架构

下图显示了我们推荐的体系结构中的所有模块以及它们之间的交互方式:

![final-architecture.png](https://developer.android.google.cn/topic/libraries/architecture/images/final-architecture.png)

## 指导原则

编程是一个有创意的领域，开发Android应用也不是一个例外。
解决一个问题有很多方法，比如在多个activities或fragments之间通信数据，检索远程数据，并在本地进行脱机模式，或者其他一些常见的应用程序遇到的其他常见场景。

虽然下面的建议不是强制性的，但是根据我们的经验，从长远来看，遵循这些建议将使您的代码基础更加健壮、可测试和可维护。

* 在清单中定义的入口点  — activities、services、broadcast receivers等 — `不是数据的来源`。
相反，它们应该只协调与该入口点相关的数据子集。
由于每个应用程序组件都很短，取决于用户与设备的交互以及运行时的整体健康状况，所以您不希望这些入口点成为数据的来源。

* 无情地在你的应用程序的不同模块之间创建`明确的责任界限`。
例如，不要将代码从网络中跨多个类或包加载到您的代码库中。
类似地，不要把不相关的职责——比如数据缓存和数据绑定——放到同一个类中。

* `尽可能少地暴露`于每个模块。
不要试图创建一个从一个模块中暴露内部实现细节的“只有那个”的快捷方式。
你可能会在短期内获得一些时间，但随着代码库的不断发展，你将会花费很多时间来支付技术债务。

* 当您定义模块之间的交互时，请考虑如何使每个模块`独立地进行测试`。
例如，有一个定义良好的API从网络中获取数据，这将使测试在本地数据库中持久存储数据的模块变得更加容易。
相反，如果您将这两个模块的逻辑混合在一个地方，或者将您的网络代码洒在整个代码库中，那么要进行测试将会困难得多。

* 你的应用程序的核心是让它脱颖而出的原因。
`不要把时间花在重新发明轮子上`，或者一次又一次地编写相同的样板代码。
相反，把你的精力集中在让你的应用独一无二的东西上，让Android架构组件和其他推荐的库处理重复的样板文件。

* `保持尽可能多的相关和新鲜的数据`，这样当设备处于脱机状态时，你的应用就可以使用了。
虽然你可能喜欢持续的高速连接，但你的用户可能不会。

* 您的repository应该指定一个数据源作为事实的`单一来源`。
当你的应用程序需要访问这段数据时，它应该总是来自于单一的真相来源。
要了解更多信息，请参阅[单一来源的真相](#truth)。

## <a name="addendum"></a>附录:暴露网络状态 ##

在上面[推荐应用架构](#recommended_app_architecture)章节中，我们故意省略了网络错误和加载状态，以保持示例的简单性。
在本节中，我们将演示如何使用Resource类公开网络状态，以封装数据及其状态。

下面是一个示例实现:

```java
//描述具有状态的数据的泛型类
public class Resource<T> {
    @NonNull public final Status status;
    @Nullable public final T data;
    @Nullable public final String message;
    private Resource(@NonNull Status status, @Nullable T data, @Nullable String message) {
        this.status = status;
        this.data = data;
        this.message = message;
    }

    public static <T> Resource<T> success(@NonNull T data) {
        return new Resource<>(SUCCESS, data, null);
    }

    public static <T> Resource<T> error(String msg, @Nullable T data) {
        return new Resource<>(ERROR, data, msg);
    }

    public static <T> Resource<T> loading(@Nullable T data) {
        return new Resource<>(LOADING, data, null);
    }
}
```

因为从网络中加载数据是一个常见的用例，所以我们将创建一个助手类`NetworkBoundResource`，它可以在多个位置重用。
下面是`NetworkBoundResource`的决策树:

![network-bound-resource.png](https://developer.android.google.cn/topic/libraries/architecture/images/network-bound-resource.png)

它首先通过对resource的数据库进行观察。
当第一次从数据库加载条目时，NetworkBoundResource会检查结果是否足够好，是否可以从网络中获取。
请注意，这两种情况都可能同时发生，因为您可能想要在从网络中更新缓存数据同时显示缓存的数据

如果网络调用成功完成，它会将响应保存到数据库中，并重新初始化流。
如果网络请求失败，我们将直接发送失败。

> 注意:在将新数据保存到磁盘之后，我们将从数据库重新初始化流，但是通常我们不需要这样做，因为数据库将会分派更改。
另一方面，依赖于数据库来调度更改将依赖于不好的副作用，因为如果数据库能够避免在数据没有变化的情况下发送更改，则可能会中断。
我们也不希望将来自网络的结果发送出去，因为这将与单一的真相来源相违背(也许数据库中有一些触发器将改变保存的值)。
我们也不想在没有新数据的情况下分派SUCCESS，因为它会向客户发送错误的信息。

下面是NetworkBoundResource类为其孩子提供的公共API：

```java
// ResultType: Resource数据类型
// RequestType: API response类型
public abstract class NetworkBoundResource<ResultType, RequestType> {
    // 调用将API响应的结果保存到数据库中
    @WorkerThread
    protected abstract void saveCallResult(@NonNull RequestType item);

    // 调用数据库中的数据来决定是否应该从网络中获取数据。
    @MainThread
    protected abstract boolean shouldFetch(@Nullable ResultType data);

    // 调用从数据库中获取缓存的数据
    @NonNull @MainThread
    protected abstract LiveData<ResultType> loadFromDb();

    // 创建API调用.
    @NonNull @MainThread
    protected abstract LiveData<ApiResponse<RequestType>> createCall();

    // fetch失败时。子类可能希望重新设置诸如速率限制器之类的组件
    @MainThread
    protected void onFetchFailed() {
    }

    // 返回一个代表resource的LiveData，在基类中实现
    public final LiveData<Resource<ResultType>> getAsLiveData();
}
```

注意，上面的类定义了两个类型参数(`ResultType`，`RequestType`)，因为从API返回的数据类型可能与本地使用的数据类型不匹配。

还要注意，上面的代码使用了网络请求的`ApiResponse`。
ApiResponse是一个围绕`Retrofit2.Call`类的简单包装器，用于将其响应转换为LiveData。

下面是`NetworkBoundResource`类的其余实现:

```java
public abstract class NetworkBoundResource<ResultType, RequestType> {
    private final MediatorLiveData<Resource<ResultType>> result = new MediatorLiveData<>();

    @MainThread
    NetworkBoundResource() {
        result.setValue(Resource.loading(null));
        LiveData<ResultType> dbSource = loadFromDb();
        result.addSource(dbSource, data -> {
            result.removeSource(dbSource);
            if (shouldFetch(data)) {
                fetchFromNetwork(dbSource);
            } else {
                result.addSource(dbSource,
                        newData -> result.setValue(Resource.success(newData)));
            }
        });
    }

    private void fetchFromNetwork(final LiveData<ResultType> dbSource) {
        LiveData<ApiResponse<RequestType>> apiResponse = createCall();
        // 我们将dbSource作为新的源重新连接,
        // 它将很快地发布最新的价值
        result.addSource(dbSource,
                newData -> result.setValue(Resource.loading(newData)));
        result.addSource(apiResponse, response -> {
            result.removeSource(apiResponse);
            result.removeSource(dbSource);
            //noinspection ConstantConditions
            if (response.isSuccessful()) {
                saveResultAndReInit(response);
            } else {
                onFetchFailed();
                result.addSource(dbSource,
                        newData -> result.setValue(
                                Resource.error(response.errorMessage, newData)));
            }
        });
    }

    @MainThread
    private void saveResultAndReInit(ApiResponse<RequestType> response) {
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... voids) {
                saveCallResult(response.body);
                return null;
            }

            @Override
            protected void onPostExecute(Void aVoid) {
                // we specially request a new live data,
                // otherwise we will get immediately last cached value,
                // which may not be updated with latest results received from network.
                result.addSource(loadFromDb(),
                        newData -> result.setValue(Resource.success(newData)));
            }
        }.execute();
    }

    public final LiveData<Resource<ResultType>> getAsLiveData() {
        return result;
    }
}
```

现在，我们可以使用NetworkBoundResource来在存储库中编写磁盘和网络绑定用户实现。

```java
class UserRepository {
    Webservice webservice;
    UserDao userDao;

    public LiveData<Resource<User>> loadUser(final String userId) {
        return new NetworkBoundResource<User,User>() {
            @Override
            protected void saveCallResult(@NonNull User item) {
                userDao.insert(item);
            }

            @Override
            protected boolean shouldFetch(@Nullable User data) {
                return rateLimiter.canFetch(userId) && (data == null || !isFresh(data));
            }

            @NonNull @Override
            protected LiveData<User> loadFromDb() {
                return userDao.load(userId);
            }

            @NonNull @Override
            protected LiveData<ApiResponse<User>> createCall() {
                return webservice.getUser(userId);
            }
        }.getAsLiveData();
    }
}
```