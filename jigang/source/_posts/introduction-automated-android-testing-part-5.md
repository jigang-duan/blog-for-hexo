---
title: Android 自动化测试 - 第5部分
date: 2017-11-20 17:15:26
tags:
- Android
- 自动化测试
- Android自动化测试
categories:
- Android

---

<img src="http://szimg.mukewang.com/5850bc4500015ecd05400300-360-202.jpg" width="88%" height="320" align=center/>

在这一系列的博客文章中，我们正在研究一个叫做[Github用户搜索](https://github.com/riggaroo/GithubUsersSearchApp)的样本应用。
第1-4部分介绍了为什么我们应该进行测试、设置测试、创建API调用并创建一个演示程序。
看一下前面的文章，第5部分是这个系列的延续。

在第5部分中，我们将查看与第4部分中创建的Presenter的交互，我们将创建UI来显示搜索结果的列表。

<!-- more -->

## 创建UI

对于用户界面，我们需要一个简单的列表来显示列表中的avatar、name和其他用户信息。

在第4部分中，我们定义了一个Activity应该实现的视图契约。
这是Android专用代码所在的位置(诸如可见性变化或任何UI更改都将位于此处)。
为了更新你的记忆，这是我们在上一篇文章中创建的视图契约:

```java
interface UserSearchContract {

    interface View extends MvpView {
        void showSearchResults(List<User> githubUserList);

        void showError(String message);

        void showLoading();

        void hideLoading();
    }

    interface Presenter extends MvpPresenter<View> {
        void search(String term);
    }
}
```

让我们实现视图!

1. 创建一个名为`UserSearchActivity`的类。
这个类将实现`UserSearchContract.View`契约并扩展`AppCompatActivity`。
定义一个变量`userSearchPresenter` `UserSearchContract.Presenter`类型。
这是我们将与之交互的对象，以执行我们的网络调用。

	```java
	public class UserSearchActivity extends AppCompatActivity implements UserSearchContract.View {

    private UserSearchContract.Presenter userSearchPresenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_user_search);
         userSearchPresenter = new UserSearchPresenter(Injection.provideUserRepo(), Schedulers.io(),
                AndroidSchedulers.mainThread());
        userSearchPresenter.attachView(this);

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        userSearchPresenter.detachView();
    }

    @Override
    public void showSearchResults(List<User> githubUserList) {

    }

    @Override
    public void showError(String message) {

    }

    @Override
    public void showLoading() {

    }

    @Override
    public void hideLoading() {
    }
	}
	```
	在`onCreate()`中，创建presenter对象。
提供在注入类中定义的User repo。
通过`io()`调度器和`AndroidSchedulers.mainThread()`调度器,RxJava订阅他们应该知道哪个线程上执行他们的工作。

	下一行,你可以看到我调用`userSearchPresenter.attachView(this)`。
它将视图附加到presenter，以便presenter可以通知视图任何更改。
因为presenter不知道activity的生命周期,在`onDestroy()`我们需要通知presenter不复存在的观点是,我们应该叫`userSearchPresenter.detachView()`。
这将注销任何RxJava订阅，防止内存泄漏。

1. 创建`activity_user_search.xml`文件夹中的xml。
这将包含一个`RecyclerView`、一个`ProgressBar`、一个错误`TextView`和一个`Toolbar`。
我使用了`ConstraintLayout`来设计我的屏幕，所以我不会过多地讲细节，因为它主要是拖放。
(如果你想读更多关于ConstraintLayout的文章，请参阅[我的博客文章](https://riggaroo.co.za/constraintlayout-101-new-layout-builder-android-studio/))

	![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/08/UserSearchActivity.png?resize=768%2C632&ssl=1)

	```xml
	<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_user_search"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="za.co.riggaroo.gus.presentation.search.UserSearchActivity">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:background="?attr/colorPrimary"
        android:minHeight="?attr/actionBarSize"
        android:theme="?attr/actionBarTheme"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintLeft_toLeftOf="@+id/activity_user_search"
        app:layout_constraintRight_toRightOf="@+id/activity_user_search"
        app:layout_constraintTop_toTopOf="@+id/activity_user_search">

    </android.support.v7.widget.Toolbar>

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view_users"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginBottom="16dp"
        android:clipToPadding="false"
        android:scrollbars="vertical"
        app:layoutManager="android.support.v7.widget.LinearLayoutManager"
        app:layout_constraintBottom_toBottomOf="@+id/activity_user_search"
        app:layout_constraintLeft_toLeftOf="@+id/activity_user_search"
        app:layout_constraintRight_toRightOf="@+id/activity_user_search"
        app:layout_constraintTop_toBottomOf="@+id/toolbar"
        tools:listitem="@layout/list_item_user">

    </android.support.v7.widget.RecyclerView>

    <TextView
        android:id="@+id/text_view_error_msg"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="@string/search_for_some_users"
        android:visibility="visible"
        app:layout_constraintBottom_toBottomOf="@+id/recycler_view_users"
        app:layout_constraintLeft_toLeftOf="@+id/toolbar"
        app:layout_constraintRight_toRightOf="@+id/recycler_view_users"
        app:layout_constraintTop_toBottomOf="@+id/toolbar"
        tools:text="No Data has loaded"/>

    <ProgressBar
        android:id="@+id/progress_bar"
        style="@style/Widget.AppCompat.ProgressBar"
        android:layout_width="60dp"
        android:layout_height="60dp"
        android:layout_marginBottom="16dp"
        android:layout_marginTop="16dp"
        android:visibility="gone"
        app:layout_constraintBottom_toBottomOf="@+id/activity_user_search"
        app:layout_constraintLeft_toLeftOf="@+id/recycler_view_users"
        app:layout_constraintRight_toRightOf="@+id/recycler_view_users"
        app:layout_constraintTop_toBottomOf="@+id/toolbar"
        tools:visibility="visible"/>

	</android.support.constraint.ConstraintLayout>
	```
1. 	我们还需要向工具栏添加一个SearchView，这样我们就可以在某个地方输入。
添加一个menu_user_search.xml文件到菜单资源文件夹。
在它里面，你需要添加一个SearchView:

	```xml
	<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/menu_search"
        android:icon="@drawable/ic_search"
        app:showAsAction="always|collapseActionView"
        android:title="@string/search_icon_title"
        app:actionViewClass="android.support.v7.widget.SearchView"/>
</menu>
	```
1. 	我们需要创建一个布局，该布局将用于RecyclerView中的每个条目。
布局文件夹中创建一个名为list_item_user.xml的文件。
我用ConstraintLayout包含一个ImageView和两个textview。

	![](https://i1.wp.com/riggaroo.co.za/wp-content/uploads/2016/08/List_item_user_designmode.png?resize=768%2C647&ssl=1)

	```xml
	<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
                                             xmlns:app="http://schemas.android.com/apk/res-auto"
                                             xmlns:tools="http://schemas.android.com/tools"
                                             android:id="@+id/constraintLayout"
                                             android:layout_width="match_parent"
                                             android:layout_height="wrap_content"
                                             android:orientation="vertical">

    <ImageView
        android:id="@+id/imageview_userprofilepic"
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_marginLeft="16dp"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        app:layout_constraintLeft_toLeftOf="@+id/constraintLayout"
        app:layout_constraintTop_toTopOf="@+id/constraintLayout"
        app:srcCompat="@mipmap/ic_launcher"/>

    <TextView
        android:id="@+id/textview_username"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="16dp"
        android:layout_marginStart="16dp"
        android:layout_marginTop="16dp"
        android:textAppearance="@style/TextAppearance.AppCompat.Medium"
        app:layout_constraintLeft_toRightOf="@+id/imageview_userprofilepic"
        app:layout_constraintTop_toTopOf="@+id/constraintLayout"
        tools:text="Rebecca Franks"/>

    <TextView
        android:id="@+id/textview_user_profile_info"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:layout_marginLeft="16dp"
        android:layout_marginRight="16dp"
        android:layout_marginStart="16dp"
        android:textAppearance="@style/TextAppearance.AppCompat.Caption"
        app:layout_constraintBottom_toBottomOf="@+id/constraintLayout"
        app:layout_constraintLeft_toRightOf="@+id/imageview_userprofilepic"
        app:layout_constraintRight_toRightOf="@+id/constraintLayout"
        app:layout_constraintTop_toBottomOf="@+id/textview_username"
        tools:text="JHB, South Africa. Lots of code, lots and lots and lots of code."/>
</android.support.constraint.ConstraintLayout>
	```
1. 	现在，我们已经有了所需的所有布局，让我们将XML与Activity联系起来。
首先，在onCreate()中，我们将获得对我们需要的视图的引用。

	```java
	private UsersAdapter usersAdapter;
    private SearchView searchView;
    private Toolbar toolbar;
    private ProgressBar progressBar;
    private RecyclerView recyclerViewUsers;
    private TextView textViewErrorMessage;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...

        toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        progressBar = (ProgressBar) findViewById(R.id.progress_bar);
        textViewErrorMessage = (TextView) findViewById(R.id.text_view_error_msg);
        recyclerViewUsers = (RecyclerView) findViewById(R.id.recycler_view_users);
        usersAdapter = new UsersAdapter(null, this);
        recyclerViewUsers.setAdapter(usersAdapter);

    }
	```
1. 我们需要将SearchView连接到我们的activity中，以使它触发presenters `search()`方法。
在`onCreateOptionsMenu()`中，添加以下代码:

	```java
	@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        super.onCreateOptionsMenu(menu);
        getMenuInflater().inflate(R.menu.menu_user_search, menu);
        final MenuItem searchActionMenuItem = menu.findItem(R.id.menu_search);
        searchView = (SearchView) searchActionMenuItem.getActionView();
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                if (!searchView.isIconified()) {
                    searchView.setIconified(true);
                }
                userSearchPresenter.search(query);
                toolbar.setTitle(query);
                searchActionMenuItem.collapseActionView();
                return false;
            }

            @Override
            public boolean onQueryTextChange(String s) {
                return false;
            }
        });
        searchActionMenuItem.expandActionView();
        return true;
    }
	```
	这将使正确的菜单inflate，找到搜索视图并设置一个查询文本侦听器。
在这种情况下，只有当有人按下键盘上的提交时，我们才会以查询的方式调用搜索提供器。
我们也可以在`onQueryTextChange`中完成，但是由于在Github API上的限制，我将坚持`onQueryTextSubmit`。
默认情况下，项目将被扩展。

1. 接下来，我们将实现presenter在完成加载时将调用的回调。

	```java
	@Override
    public void showSearchResults(List<User> githubUserList) {
        recyclerViewUsers.setVisibility(View.VISIBLE);
        textViewErrorMessage.setVisibility(View.GONE);
        usersAdapter.setItems(githubUserList);
    }

    @Override
    public void showError(String message) {
        textViewErrorMessage.setVisibility(View.VISIBLE);
        recyclerViewUsers.setVisibility(View.GONE);
        textViewErrorMessage.setText(message);
    }

    @Override
    public void showLoading() {
        progressBar.setVisibility(View.VISIBLE);
        recyclerViewUsers.setVisibility(View.GONE);
        textViewErrorMessage.setVisibility(View.GONE);
    }

    @Override
    public void hideLoading() {
        progressBar.setVisibility(View.GONE);
        recyclerViewUsers.setVisibility(View.VISIBLE);
        textViewErrorMessage.setVisibility(View.GONE);

    }
	```
	我们基本上只是在这里切换视图的可见性，并将usersAdapter设置为服务返回的新items。

1. 	为了完整起见，这里是`UserSearchAdapter`类，它用于`activity`的`RecyclerView`:

	```java
	class UsersAdapter extends RecyclerView.Adapter<UserViewHolder> {
    	private final Context context;
    	private List<User> items;

    	UsersAdapter(List<User> items, Context context) {
        	this.items = items;
        	this.context = context;
    	}

    	@Override
    	public UserViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        	View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.list_item_user, parent, false);
        	return new UserViewHolder(v);
    	}

    	@Override
    	public void onBindViewHolder(UserViewHolder holder, int position) {
       	User item = items.get(position);

        	holder.textViewBio.setText(item.getBio());
        	if (item.getName() != null) {
           		holder.textViewName.setText(item.getLogin() + " - " + item.getName());
        	} else {
           	holder.textViewName.setText(item.getLogin());
        	}
        	Picasso.with(context).load(item.getAvatarUrl()).into(holder.imageViewAvatar);
    	}

    	@Override
    	public int getItemCount() {
        	if (items == null) {
            	return 0;
        	}
        	return items.size();
    	}

    	void setItems(List<User> githubUserList) {
        	this.items = githubUserList;
        	notifyDataSetChanged();
    	}
	}


		class UserViewHolder extends RecyclerView.ViewHolder {
    	final TextView textViewBio;
    	final TextView textViewName;
    	final ImageView imageViewAvatar;

    	UserViewHolder(View v) {
        	super(v);
        	imageViewAvatar = (ImageView) v.findViewById(R.id.imageview_userprofilepic);
        	textViewName = (TextView) v.findViewById(R.id.textview_username);
        	textViewBio = (TextView) v.findViewById(R.id.textview_user_profile_info);
    	}
	}
	```
	注射类

	```java
	public class Injection {

    	private static final String BASE_URL = "https://api.github.com";
    	private static OkHttpClient okHttpClient;
    	private static GithubUserRestService userRestService;
    	private static Retrofit retrofitInstance;


    	public static UserRepository provideUserRepo() {
        	return new UserRepositoryImpl(provideGithubUserRestService());
    	}

    	static GithubUserRestService provideGithubUserRestService() {
        	if (userRestService == null) {
            	userRestService = getRetrofitInstance().create(GithubUserRestService.class);
        	}
        	return userRestService;
    	}

    	static OkHttpClient getOkHttpClient() {
        	if (okHttpClient == null) {
            	HttpLoggingInterceptor logging = new HttpLoggingInterceptor();
            	logging.setLevel(HttpLoggingInterceptor.Level.BASIC);
            	okHttpClient = new OkHttpClient.Builder().addInterceptor(logging).build();
        	}

        	return okHttpClient;
    	}

    	static Retrofit getRetrofitInstance() {
        	if (retrofitInstance == null) {
            	Retrofit.Builder retrofit = new Retrofit.Builder().client(Injection.getOkHttpClient()).baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create())
                    .addCallAdapterFactory(RxJavaCallAdapterFactory.create());
            retrofitInstance = retrofit.build();

        }
        return retrofitInstance;
    }
	}
	```
1. 	现在，如果你运行这款应用，你应该能够在Github上搜索用户名，并查看结果。

	![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/08/github_user_search.gif?resize=480%2C854&ssl=1)


耶!我们有一个工作应用程序。这篇文章的代码可以在这里找到。在接下来的部分中，我们将考虑为应用程序编写UI测试！
