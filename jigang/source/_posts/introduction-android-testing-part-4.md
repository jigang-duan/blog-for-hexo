---
title: Android 自动化测试 - 第4部分
date: 2017-11-17 20:28:15
tags:
- Android
- 自动化测试
categories:
- Android自动化测试

---

<img src="http://szimg.mukewang.com/5850bc4500015ecd05400300-360-202.jpg" width="88%" height="320" align=center/>

在这篇博文中，我们正在开发一个名为Github用户搜索的Android应用程序。在之前的博客文章中，我们研究了为测试创建应用程序、创建API调用和编写API转换的第一个基本测试。阅读第1部分、第2部分和第3部分。

这篇文章将着眼于创建一个与repository通信并向视图传递信息的presenter。这还包括为presenter编写单元测试。这篇博文的示例github repo将会在[这里](https://github.com/riggaroo/GithubUsersSearchApp)找到。

<!-- more -->

## 创建Presenter

1. 为了开始创建，创建称为MvpView和MvpPresenter的基本接口。所有的MVP功能都将扩展这两个接口。

	```java
	public interface MvpView {
	}
	```

	```java
	public interface MvpPresenter<V extends MvpView> {
 
    	void attachView(V mvpView);
 
    	void detachView();
	}
	```

1. 创建一个BasePresenter。
这将提供功能来检查一个视图是否附加到presenter上，以及管理RxJava订阅的方法。

	```java
	public class BasePresenter<T extends MvpView> implements MvpPresenter<T> {
 
    	private T view;
 
    	private CompositeSubscription compositeSubscription = new CompositeSubscription();
 
    	@Override
    	public void attachView(T mvpView) {
        	view = mvpView;
    	}
 
    	@Override
    	public void detachView() {
        	compositeSubscription.clear();
        	view = null;
    	}
 
    	public T getView() {
        	return view;
    	}
 
    	public void checkViewAttached() {
        	if (!isViewAttached()) {
            	throw new MvpViewNotAttachedException();
        	}
    	}
 
    	private boolean isViewAttached() {
        	return view != null;
    	}
 
    	protected void addSubscription(Subscription subscription) {
        	this.compositeSubscription.add(subscription);
    	}
 
    	protected static class MvpViewNotAttachedException extends RuntimeException {
        	public MvpViewNotAttachedException() {
            	super("Please call Presenter.attachView(MvpView) before" + " requesting data to the Presenter");
        	}
    	}
}
```

	正如您可以看到的，在presenter中有一个`CompositeSubscription`。
该对象将持有一组RxJava订阅。
`detachView()`方法调用`compositeSubscription.clear()`将从所有订阅退订,防止内存泄漏和视图崩溃(代码将不会运行时视图被摧毁,因为它是unsubscribed)。
当一个presenter在一个子类中创建一个订阅器时，我们将调用`addSubscription()`

1. 在一个名为`UserSearchContract`的类中创建视图和presenter之间的契约。
在这个类中，创建一个用于视图和presenter的两个接口。

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
	在view中，有4种方法，`showSearchResults()`、`showLoading()`、`hideLoading()`、`showError()`。
在presenter中，有一个名为`search()`的方法。

	**presenter**不关心**view**是如何显示结果的，也不关心它如何显示错误。类似地，一个**view**并不关心一个**presenter**如何搜索，只要它使用这些回调来通知，实现就无关紧要了。

	分离视图和presenter之间的逻辑很简单。
考虑为另一种类型的UI重用演示程序，这将使您意识到代码应该在哪里。
例如，如果您必须使用Java Swing，那么在这种情况下，您的演示器可以保持不变，只有您的视图实现会有所不同。
这可以帮助您通过简单地问自己一个问题来放置逻辑:**如果我有不同类型的UI，那么presenter中的逻辑会有意义吗?**

1. 现在，view和presenter之间的契约被定义了。
创建/导航到UserSearchPresenter。
这里将创建UserRepository的订阅，它将调用Github API。

	```java
	class UserSearchPresenter extends BasePresenter<UserSearchContract.View> implements UserSearchContract.Presenter {
    	private final Scheduler mainScheduler, ioScheduler;
    	private UserRepository userRepository;
 
    	UserSearchPresenter(UserRepository userRepository, Scheduler ioScheduler, Scheduler mainScheduler) {
        	this.userRepository = userRepository;
        	this.ioScheduler = ioScheduler;
        	this.mainScheduler = mainScheduler;
    	}
 
	}
	```
	在这里，presenter扩展了`BasePresenter`，并实现了第3步中定义的`UserSearchContract.Presenter`契约。该类将实现`search()`方法。
	
	在尝试进行单元测试时，使用构造函数注入可以轻松地模拟`UserRepository`。schedulers也被注入到构造函数中，因为单元测试总是使用`Schedulers.immediate()`，但是在视图中，我们将使用不同的线程。
	
1. 现在，为了实现search():

	```java
	@Override
    public void search(String term) {
        checkViewAttached();
        getView().showLoading();
        addSubscription(userRepository.searchUsers(term).subscribeOn(ioScheduler).observeOn(mainScheduler).subscribe(new Subscriber<List<User>>() {
            @Override
            public void onCompleted() {
 
            }
 
            @Override
            public void onError(Throwable e) {
                getView().hideLoading();
                getView().showError(e.getMessage()); //TODO You probably don't want this error to show to users - Might want to show a friendlier message :)
            }
 
            @Override
            public void onNext(List<User> users) {
                getView().hideLoading();
                getView().showSearchResults(users);
            }
        }));
    }
	```
	首先，运行checkViewAttached()，如果在方法开始运行时没有附加视图，它将抛出一个异常。
然后告诉视图，它应该通过调用showLoading()开始加载。
创建`userRepository.searchUsers()`的订阅。
将`subscribeOn()`设置为`ioScheduler`变量，因为我们希望在IO线程上发生这些网络调用。
设置observeOn()  `mainScheduler`，因为我们希望在主线程上观察到这个订阅的结果。
然后通过调用`addSubscription`，将订阅添加到复合订阅中。

在`onNext()`方法中，通过使用API返回的用户列表调用`hideLoading()`和`showSearchResults()`来处理结果。
在`onError()`中，停止加载，并使用异常的消息调用`showError()`。

下面是`UserSearchPresenter`的完整代码:

```java
package za.co.riggaroo.gus.presentation.search;
 
 
import java.util.List;
 
import rx.Scheduler;
import rx.Subscriber;
import za.co.riggaroo.gus.data.UserRepository;
import za.co.riggaroo.gus.data.remote.model.User;
import za.co.riggaroo.gus.presentation.base.BasePresenter;
 
class UserSearchPresenter extends BasePresenter<UserSearchContract.View> implements UserSearchContract.Presenter {
    private final Scheduler mainScheduler, ioScheduler;
    private UserRepository userRepository;
 
    UserSearchPresenter(UserRepository userRepository, Scheduler ioScheduler, Scheduler mainScheduler) {
        this.userRepository = userRepository;
        this.ioScheduler = ioScheduler;
        this.mainScheduler = mainScheduler;
    }
 
    @Override
    public void search(String term) {
        checkViewAttached();
        getView().showLoading();
        addSubscription(userRepository.searchUsers(term).subscribeOn(ioScheduler).observeOn(mainScheduler).subscribe(new Subscriber<List<User>>() {
            @Override
            public void onCompleted() {
 
            }
 
            @Override
            public void onError(Throwable e) {
                getView().hideLoading();
                getView().showError(e.getMessage()); //TODO You probably don't want this error to show to users - Might want to show a friendlier message :)
            }
 
            @Override
            public void onNext(List<User> users) {
                getView().hideLoading();
                getView().showSearchResults(users);
            }
        }));
    }
}
```

## 为UserSearchPresenter编写单元测试

现在已经定义了presenter，让我们为它创建一些单元测试。

1. 选择`UserSearchPresenter`类名。按“*ALT+Enter*”并选择“*Create Test*”。选择“app/src/**test**/java”文件夹，因为这是一个不需要Android依赖的单元测试。结果测试的位置如下:`app/src/test/java/za/co/riggaroo/gus/presentation`
1. 在`UserSearchPresenterTest`中，创建setup方法，并定义测试所需的变量。

	```java
	public class UserSearchPresenterTest {

    	@Mock
    	UserRepository userRepository;
    	@Mock
    	UserSearchContract.View view;

    	UserSearchPresenter userSearchPresenter;

    	@Before
    	public void setUp() throws Exception {
        	MockitoAnnotations.initMocks(this);
        	userSearchPresenter = new UserSearchPresenter(userRepository, Schedulers.immediate(), Schedulers.immediate());
        	userSearchPresenter.attachView(view);
    	}
	}
	```
	通过创建`UserRepository`和`UserSearchContract.View`的模拟实例，我们将确保我们只是在测试`UserSearchPresenter`。
在`setUp()`方法中,我们称之为`MockitoAnnotations.initMocks()`。
然后使用模拟对象和即时调度程序创建搜索演示程序。
使用模拟视图对象调用`attachView()`，因为只有当一个视图被连接时，它才会起作用。

1. 第一个测试将测试一个有效的搜索词是否有正确的回调:

	```java
	private static final String USER_LOGIN_RIGGAROO = "riggaroo";
    private static final String USER_LOGIN_2_REBECCA = "rebecca";
   
    @Test
    public void search_ValidSearchTerm_ReturnsResults() {
        UsersList userList = getDummyUserList();
        when(userRepository.searchUsers(anyString())).thenReturn(Observable.<List<User>>just(userList.getItems()));
 
        userSearchPresenter.search("riggaroo");
 
        verify(view).showLoading();
        verify(view).hideLoading();
        verify(view).showSearchResults(userList.getItems());
        verify(view, never()).showError(anyString());
    }
 
    UsersList getDummyUserList() {
        List<User> githubUsers = new ArrayList<>();
        githubUsers.add(user1FullDetails());
        githubUsers.add(user2FullDetails());
        return new UsersList(githubUsers);
    }
 
    User user1FullDetails() {
        return new User(USER_LOGIN_RIGGAROO, "Rigs Franks", "avatar_url", "Bio1");
    }
 
    User user2FullDetails() {
        return new User(USER_LOGIN_2_REBECCA, "Rebecca Franks", "avatar_url2", "Bio2");
    }
	```
	这个测试断言:假定(**Given**)用户repository返回一组用户，当(**when**)在演示程序中调用search()时，将(**then**)调用`showLoading()`和`showSearchResults()`。这个测试还断言`showError()`方法永远不会被调用。
	
1. 下一个测试是在UserRepository抛出错误时测试负面场景的测试。

	```java
	 @Test
    public void search_UserRepositoryError_ErrorMsg() {
        String errorMsg = "No internet";
        when(userRepository.searchUsers(anyString())).thenReturn(Observable.error(new IOException(errorMsg)));

        userSearchPresenter.search("bookdash");

        verify(view).showLoading();
        verify(view).hideLoading();
        verify(view, never()).showSearchResults(anyList());
        verify(view).showError(errorMsg);
    }
	```
	这个测试正在测试以下内容:假定(**Given**) `userRepository`返回一个异常，在调用`search()`时，应该调用`showError()`。
	
1. 我们将添加的最后一个测试将断言，如果视图没有附加，将抛出一个异常。

	```java
	@Test(expected = BasePresenter.MvpViewNotAttachedException.class)
    public void search_NotAttached_ThrowsMvpException() {
        userSearchPresenter.detachView();
 
        userSearchPresenter.search("test");
 
        verify(view, never()).showLoading();
        verify(view, never()).showSearchResults(anyList());
    }
	```
1. 让我们运行测试，看看我们有多少测试覆盖率。右键单击测试名称并单击“*Run tests with coverage*”。

	![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/08/Screen-Shot-2016-08-08-at-8.00.29-PM.png?resize=768%2C109&ssl=1)
	
	我们对UserSearchPresenter有100%的覆盖率！耶!

下一篇博客文章将讨论创建视图和为视图编写测试。