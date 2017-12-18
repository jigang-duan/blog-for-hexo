---
title: Android 架构组件 - 分页库
date: 2017-11-09 14:45:09
tags:
- 架构设计
- Android架构组件
categories:
- Android

---

![arch-hero.png](https://developer.android.google.cn/topic/images/arch/hero/arch-hero.png)

分页库使您的应用程序可以更容易地根据需要从数据源加载信息，而不会使设备超载，或者等待一个大的数据库查询等待太长的时间。

<!-- more -->

## 概述

许多应用程序都使用大量的数据，但只需要在任何时候加载和显示一小部分数据。
一个应用程序可能会有数千个可能显示的项目，但它可能只需要一次访问其中的几十个。
如果应用程序不小心，它可能最终会请求它实际上不需要的数据，从而给设备和网络带来性能负担。
如果数据被存储或与远程数据库同步，这也会降低应用程序的速度，并浪费用户的数据计划。

尽管现有的Android api允许在内容中进行分页，但它们带来了巨大的限制和缺点:

* [CursorAdapter](https://developer.android.google.cn/reference/android/widget/CursorAdapter.html)使将数据库查询结果映射到ListView项变得更加容易，但是它在UI线程上运行数据库查询，并且在页面内容中使用[Cursor](https://developer.android.google.cn/reference/android/database/Cursor.html)效率不高。
有关使用CursorAdapter的缺点的更多细节，请参见该博客在[Android上发布大型数据库查询](https://medium.com/google-developers/large-database-queries-on-android-cb043ae626e8)。

* AsyncListUtil允许将基于位置的数据转换为RecyclerView，但不允许非位置分页，它在可数的数据集中强制null作为占位符。

新的分页库解决了这些问题。
这个库包含几个类，可以简化请求数据的过程。
这些类还可以与现有的体系结构组件无缝协作，比如Room。

## 类

分页库提供了以下类，以及附加的支持类:

**[DataSource](https://developer.android.google.cn/reference/android/arch/paging/DataSource.html)**

使用这个类来定义一个数据源，您需要从它获取分页数据。
根据您需要访问数据的方式，您将扩展它的两个子类中的一个:

* 使用[KeyedDataSource](https://developer.android.google.cn/reference/android/arch/paging/KeyedDataSource.html)，如果您需要使用来自item N的数据来获取item N+1。
例如，如果您正在为一个讨论应用程序获取线程评论，您可能需要传递一个评论的ID来获取下一个评论的内容。

* 使用[TiledDataSource](https://developer.android.google.cn/reference/android/arch/paging/TiledDataSource.html)，如果您需要从您在数据存储中选择的任何位置获取数据页。
这个类支持从您选择的任何位置开始请求一组数据项，比如“返回从位置1200开始的20个数据项”。

如果您使用[Room持久性库](/2017/11/09/architecture-room/)来管理您的数据，它可以自动为您创建一个TiledDataSource，例如:

```java
@Query("select * from users WHERE age > :age order by name DESC, id ASC")
TiledDataSource<User> usersOlderThan(int age);
```

**[PagedList](https://developer.android.google.cn/reference/android/arch/paging/PagedList.html)**

这个类从一个数据源加载数据。
您可以配置一次加载了多少数据，以及应该预先获取多少数据，从而最小化了用户等待数据加载的时间。
这个类可以向其他类提供更新信号，比如RecyclerView.Adapter，允许您在数据加载到页面时更新RecyclerView的内容。

**[PagedListAdapter](https://developer.android.google.cn/reference/android/arch/paging/PagedListAdapter.html)**

这个类是`RecyclerView.Adapte`r的实现，它显示了`PagedList`的数据。例如，当加载一个新页面时，`PagedListAdapter`表示数据已经到达的`RecyclerView`;这样，`RecyclerView`就可以用实际的条目替换任何占位符，执行适当的动画。

PagedListAdapter还使用一个后台线程来计算PagedList下一次的更改(例如，当数据库更改生成一个带有更新数据的新PagedList时)，并根据需要调用[notifyItem…()](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView.Adapter.html#notifyItemChanged(int)) 方法来更新列表的内容。
然后，RecyclerView执行必要的更改。
例如，如果一个项目改变了PagedList版本之间的位置，那么RecyclerView将使该项目移动到列表中的新位置。

**[LivePagedListProvider](https://developer.android.google.cn/reference/android/arch/paging/LivePagedListProvider.html)**

这个类从您提供的[DataSource](https://developer.android.google.cn/reference/android/arch/paging/DataSource.html)生成`LiveData<PagedList>`。
此外，如果您使用[Room持久性库](/2017/11/09/architecture-room/)来管理数据库，那么DAO可以使用`TiledDataSource`为您生成`LivePagedListProvider`，例如:

```java
@Query("SELECT * from users order WHERE age > :age order by name DESC, id ASC")
public abstract LivePagedListProvider<Integer, User> usersOlderThan(int age);
```

整数参数告诉Room使用TiledDataSource，以基于位置的负载为基础。

紧挨着，分页库的组件组织了来自后台线程的数据流，以及UI线程的表示。
例如，当在数据库中插入一个新项时，数据源就会失效，而LivePagedListProvider会在后台线程中生成一个新的PagedList。

![paging-threading.gif](https://developer.android.google.cn/images/topic/libraries/architecture/paging-threading.gif)
	图1所示 分页库组件在后台线程中完成了大部分工作，因此它们不会负担UI线程

这个新创建的PagedList被发送到UI线程上的PagedListAdapter。
然后，PagedListAdapter会在后台线程上使用[DiffUtil](https://developer.android.google.cn/reference/android/support/v7/util/DiffUtil.html)来计算当前列表和新列表之间的差异。
当完成比较,PagedListAdapter使用差异列表信息做出适当的调用[RecyclerView.Adapter.notifyItemInserted()](https://developer.android.google.cn/reference/android/support/v7/widget/RecyclerView.Adapter.html#notifyItemInserted(int))信号,插入一个新项。

在UI线程上的RecyclerView知道它只需要绑定一个新条目，并将它显示在屏幕上。

下面的代码示例显示了所有的片段一起工作。当用户在数据库中添加、删除或更改时，RecyclerView的内容会自动且有效地更新:

```java
@Dao
interface UserDao {
    @Query("SELECT * FROM user ORDER BY lastName ASC")
    public abstract LivePagedListProvider<Integer, User> usersByLastName();
}

class MyViewModel extends ViewModel {
    public final LiveData<PagedList<User>> usersList;
    public MyViewModel(UserDao userDao) {
        usersList = userDao.usersByLastName().create(
                /* initial load position */ 0,
                new PagedList.Config.Builder()
                        .setPageSize(50)
                        .setPrefetchDistance(50)
                        .build());
    }
}

class MyActivity extends AppCompatActivity {
    @Override
    public void onCreate(Bundle savedState) {
        super.onCreate(savedState);
        MyViewModel viewModel = ViewModelProviders.of(this).get(MyViewModel.class);
        RecyclerView recyclerView = findViewById(R.id.user_list);
        UserAdapter<User> adapter = new UserAdapter();
        viewModel.usersList.observe(this, pagedList -> adapter.setList(pagedList));
        recyclerView.setAdapter(adapter);
    }
}

class UserAdapter extends PagedListAdapter<User, UserViewHolder> {
    public UserAdapter() {
        super(DIFF_CALLBACK);
    }
    @Override
    public void onBindViewHolder(UserViewHolder holder, int position) {
        User user = getItem(position);
        if (user != null) {
            holder.bindTo(user);
        } else {
            // Null defines a placeholder item - PagedListAdapter will automatically invalidate
            // this row when the actual object is loaded from the database
            holder.clear();
        }
    }
    public static final DiffCallback<User> DIFF_CALLBACK = new DiffCallback<User>() {
        @Override
        public boolean areItemsTheSame(@NonNull User oldUser, @NonNull User newUser) {
            // User properties may have changed if reloaded from the DB, but ID is fixed
            return oldUser.getId() == newUser.getId();
        }
        @Override
        public boolean areContentsTheSame(@NonNull User oldUser, @NonNull User newUser) {
            // NOTE: if you use equals, your object must properly override Object#equals()
            // Incorrectly returning false here will result in too many animations.
            return oldUser.equals(newUser);
        }
    }
}
```
