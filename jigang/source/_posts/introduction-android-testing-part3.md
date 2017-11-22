---
title: Android 自动化测试 - 第3部分
date: 2017-11-17 19:30:11
tags:
- Android
- 自动化测试
categories:
- Android自动化测试

---

<img src="http://szimg.mukewang.com/5850bc4500015ecd05400300-360-202.jpg" width="88%" height="320" align=center/>

在之前的两篇博文中，我介绍了如何在Android中进行测试，我们创建了一个样本应用，我们将继续在这篇博文中继续开发。如果您错过了这两个帖子，我建议您阅读第1部分和第2部分。

在本文中，我们将从Github API中获取用户列表，并为其编写单元测试。
我们将从这个[检查点的下一个repo](https://github.com/riggaroo/GithubUsersSearchApp/tree/testing-tutorial-part2-complete)开始。

<!-- more -->

## 创建Web服务调用

要使用Github API，我们将使用翻新和RxJava。
我不打算在本系列中解释RxJava或Retrofit。
如果您不熟悉RxJava，我建议您阅读[这些文章](http://blog.danlew.net/2014/09/15/grokking-rxjava-part-1/)。
如果你没有使用过Retrofit，我建议阅读[这里](http://square.github.io/retrofit/)。

为了获得一个搜索词的用户列表，我们需要使用下面的端点:

```
https://api.github.com/search/users?per_page=2&q=rebecca
```

要获得更多的用户信息(例如用户的个人信息和位置)，我们需要进行后续调用:

```
https://api.github.com/users/riggaroo
```

要开始使用这些端点，我们应该创建它们正在返回的JSON对象，并将它们包含在我们的项目中。
我通常在在线生成它们。
让我们创建以下类:`User`类和`UsersList`类:

```Java
package za.co.riggaroo.gus.data.remote.model;
 
import com.google.gson.annotations.Expose;
import com.google.gson.annotations.SerializedName;
 
public class User {
 
    @SerializedName("login")
    @Expose
    private String login;
    @SerializedName("id")
    @Expose
    private Integer id;
    @SerializedName("avatar_url")
    @Expose
    private String avatarUrl;
    @SerializedName("gravatar_id")
    @Expose
    private String gravatarId;
    @SerializedName("url")
    @Expose
    @SerializedName("type")
    @Expose
    private String type;
    @SerializedName("name")
    @Expose
    private String name;
    @SerializedName("location")
    @Expose
    private String location;
    @SerializedName("email")
    @SerializedName("bio")
    @Expose
    private String bio;
    @SerializedName("followers")
    @Expose
    private Integer followers;
    @SerializedName("following")
    @Expose
    private Integer following;
    @SerializedName("created_at")
    @Expose
    private String createdAt;
    @SerializedName("updated_at")
    @Expose
    private String updatedAt;
 
   ... //see more at https://github.com/riggaroo/GithubUsersSearchApp/blob/testing-tutorial-part3-complete/app/src/main/java/za/co/riggaroo/gus/data/remote/model/User.java
 
}
```

```Java
package za.co.riggaroo.gus.data.remote.model;
 
import com.google.gson.annotations.Expose;
import com.google.gson.annotations.SerializedName;
 
import java.util.ArrayList;
import java.util.List;
 
public class UsersList {
 
    @SerializedName("total_count")
    @Expose
    private Integer totalCount;
    @SerializedName("items")
    @Expose
    private List<User> items = new ArrayList<User>();
 
    public Integer getTotalCount() {
        return totalCount;
    }
 
    public void setTotalCount(Integer totalCount) {
        this.totalCount = totalCount;
    }
 
    public List<User> getItems() {
        return items;
    }
 
    public void setItems(List<User> items) {
        this.items = items;
    }
 
}
```

创建模型后，导航到GithubUserRestService。这是我们将创建我们的Retrofit调用的地方。

```Java
import retrofit2.http.GET;
import retrofit2.http.Path;
import retrofit2.http.Query;
import rx.Observable;
import za.co.riggaroo.gus.data.remote.model.User;
import za.co.riggaroo.gus.data.remote.model.UsersList;
 
public interface GithubUserRestService {
 
    @GET("/search/users?per_page=2")
    Observable<UsersList> searchGithubUsers(@Query("q") String searchTerm);
 
    @GET("/users/{username}")
    Observable<User> getUser(@Path("username") String username);
}
```

第一个网络调用将执行搜索以获得用户列表，第二个网络调用将获得关于用户的更多信息。

导航到`UserRepositoryImpl`。
在这里，我们将把两个网络调用合并起来，并将数据转换为一个将在前端使用的视图。
这是使用RxJava首先获取搜索词的用户列表，然后为每个用户提供另一个网络调用，以查找更多的用户信息。(如果您自己实现了这个API，我将尝试让一个网络调用返回所有必需的信息——正如我在[减少移动数据使用谈话](https://www.youtube.com/watch?v=tnj2uoTwevg)中所讨论的那样)。

```java
import java.io.IOException;
import java.util.List;

import rx.Observable;
import za.co.riggaroo.gus.data.remote.GithubUserRestService;
import za.co.riggaroo.gus.data.remote.model.User;

public class UserRepositoryImpl implements UserRepository {

   private GithubUserRestService githubUserRestService;

    public UserRepositoryImpl(GithubUserRestService githubUserRestService) {
        this.githubUserRestService = githubUserRestService;
    }

    @Override
    public Observable<List<User>> searchUsers(final String searchTerm) {
        return Observable.defer(() -> githubUserRestService.searchGithubUsers(searchTerm).concatMap(
                usersList -> Observable.from(usersList.getItems())
                        .concatMap(user -> githubUserRestService.getUser(user.getLogin())).toList()))
                .retryWhen(observable -> observable.flatMap(o -> {
                    if (o instanceof IOException) {
                        return Observable.just(null);
                    }
                    return Observable.error(o);
                }));
    }

}
```

在上面的代码中，我使用`Observable.defer()`来创建一个observable对象，这意味着只有当它有一个订阅者(不像Observable.create()在创建时运行)时，才会运行observables代码。
正如下面的注释所纠正的那样，Observable.create()是一个不安全的RxJava API，它不应该被使用。

当有订阅者时，将调用`githubUserRestService`来搜索提供的`searchTerm`。
从那里,我使用`concatMap`用户列表,发出他们一个接一个地进入一个新的观察,然后调用`githubUserRestService.getUser()`为每个用户列表。
然后，这个可观察到的用户就变成了一个用户列表。

在这些网络调用上也定义了一个`重试机制`。
`retryWhen()`将在抛出IOException时重新尝试可观察的对象。
当用户***没有网络***时(您可能想要添加一个终止条件，例如仅重新尝试某些次数)，就会抛出`IOException`。

您可能会注意到，我在代码中使用`lambda`表达式，您可以通过使用新的Jack工具链来构建应用程序。
请阅读有关[如何在Android上启用Java 8的功能](https://developer.android.com/preview/j8-jack.html)。

现在我们有了一个存储库和两个网络调用来获得一个用户列表！我们应该为刚刚编写的代码编写测试

## 单元测试 — Mockito是什么?

为了对repository对象进行单元测试，我们将使用**Mockito**。
Mockito是什么?
根据MIT许可，它是一个开源的Java开源测试框架。
该框架允许在自动化单元测试中创建测试双对象(模拟对象)。([维基百科](https://en.wikipedia.org/wiki/Mockito))

Mockito允许你stub方法调用，并验证与对象的交互。

当我们编写单元测试时，我们需要考虑单独测试某个组件。
我们不应该测试任何超出这类职责的东西。
Mockito帮助我们实现这一分离

好了，让我们写一些测试吧！

## 为UserRepositoryImpl编写单元测试

1. 选择`UserRepositoryImpl`的类名，并按“ALT+ENTER”键。
将弹出一个对话框，选择“Create Test”选项。
选择该选项，将出现一个新的对话框:
![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Create-Test-Dialog.png?resize=768%2C837&ssl=1)
1. 您可以选择生成方法，但我通常会选择未选择的选项。
然后，它将要求您选择应该放置测试的目录。
在编写一个不需要Android上下文的JUnit测试时，选择`app/src/test`目录。
![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Select-Test-Directory-Automated-Testing-Android.png?resize=768%2C805&ssl=1)
1. 现在我们已经准备好设置单元测试了。
要做到这一点，需要创建一个`UserRepository`对象。
我们还需要创建一个`GithubUserRestService`的模拟实例，因为我们不会直接在这个测试中对API进行攻击。
这个测试将确认在UserRepository中正确地完成了转换。
下面是设置我们的单元测试的代码:

	```java
	@Mock
    GithubUserRestService githubUserRestService;
 
    private UserRepository userRepository;
 
    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
        userRepository = new UserRepositoryImpl(githubUserRestService);
    }
	```
1. 我们将编写的第一个测试将测试GithubUserRestService是否具有正确的参数。
它还将测试它是否返回预期的结果。
下面是我所写的示例测试:

	```java
	@Test
    public void searchUsers_200OkResponse_InvokesCorrectApiCalls() {
        //Given
        when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(Observable.just(githubUserList()));
        when(githubUserRestService.getUser(anyString()))
                .thenReturn(Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
 
        //When
        TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
        userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
 
        //Then
        subscriber.awaitTerminalEvent();
        subscriber.assertNoErrors();
 
        List<List<User>> onNextEvents = subscriber.getOnNextEvents();
        List<User> users = onNextEvents.get(0);
        Assert.assertEquals(USER_LOGIN_RIGGAROO, users.get(0).getLogin());
        Assert.assertEquals(USER_LOGIN_2_REBECCA, users.get(1).getLogin());
        verify(githubUserRestService).searchGithubUsers(USER_LOGIN_RIGGAROO);
        verify(githubUserRestService).getUser(USER_LOGIN_RIGGAROO);
        verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
    }
 
    private UsersList githubUserList() {
        User user = new User();
        user.setLogin(USER_LOGIN_RIGGAROO);
 
        User user2 = new User();
        user2.setLogin(USER_LOGIN_2_REBECCA);
 
        List<User> githubUsers = new ArrayList<>();
        githubUsers.add(user);
        githubUsers.add(user2);
        UsersList usersList = new UsersList();
        usersList.setItems(githubUsers);
        return usersList;
    }
 
    private User user1FullDetails() {
        User user = new User();
        user.setLogin(USER_LOGIN_RIGGAROO);
        user.setName("Rigs Franks");
        user.setAvatarUrl("avatar_url");
        user.setBio("Bio1");
        return user;
    }
 
    private User user2FullDetails() {
        User user = new User();
        user.setLogin(USER_LOGIN_2_REBECCA);
        user.setName("Rebecca Franks");
        user.setAvatarUrl("avatar_url2");
        user.setBio("Bio2");
        return user;
    }
	```
	这个测试分为三个部分: 给定(given)，什么时候(when)，什么时候(then)。
	我将我的测试分离开来，因为它确保您的测试是结构化的，并让您思考您正在测试的特定功能。
	在这个测试中，我正在测试以下内容:给定Github服务返回某些用户时，当我搜索用户时，结果应该返回并正确地转换。
	
	我发现测试的命名也很重要。我喜欢遵循的命名结构如下:
	
	`[Name of method under test]_[Conditions of test case]_[Expected Result]`
	
	在这个例子中，方法的名字是
	
	在本例中，该方法的名称是`searchUsers_200OkResponse_InvokesCorrectApiCalls()`。
	在这个测试用例中，一个`TestSubscriber`订阅了可观察到的搜索查询。
	然后在`TestSubscriber`上进行断言，以确保它具有预期的结果。
1. 下一个单元测试将测试如果一个IOException被搜索服务调用抛出，那么网络调用将被重试。
	
	```java
 	@Test
    public void searchUsers_IOExceptionThenSuccess_SearchUsersRetried() {
        //Given
        when(githubUserRestService.searchGithubUsers(anyString()))
                .thenReturn(getIOExceptionError(), Observable.just(githubUserList()));
        when(githubUserRestService.getUser(anyString()))
                .thenReturn(Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
 
        //When
        TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
        userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
 
        //Then
        subscriber.awaitTerminalEvent();
        subscriber.assertNoErrors();
 
        verify(githubUserRestService, times(2)).searchGithubUsers(USER_LOGIN_RIGGAROO);
 
        verify(githubUserRestService).getUser(USER_LOGIN_RIGGAROO);
        verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
    }
	```
	在这个测试中，我们断言githubUserRestService被调用两次，而其他的网络调用一次被调用一次。我们还断言订阅者没有终止错误。
	
## UserRepositoryImpl的最终单元测试代码

我已经添加了比上面描述的更多的测试。
它们测试不同的情况，但是它们遵循上一节中描述的相同的概念。
下面是`UserRepositoryImpl`的完整测试类:

```java
public class UserRepositoryImplTest {
 
    private static final String USER_LOGIN_RIGGAROO = "riggaroo";
    private static final String USER_LOGIN_2_REBECCA = "rebecca";
    @Mock
    GithubUserRestService githubUserRestService;
 
    private UserRepository userRepository;
 
    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        userRepository = new UserRepositoryImpl(githubUserRestService);
    }
 
    @Test
    public void searchUsers_200OkResponse_InvokesCorrectApiCalls() {
        //Given
        when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(Observable.just(githubUserList()));
        when(githubUserRestService.getUser(anyString()))
                .thenReturn(Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
 
        //When
        TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
        userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
 
        //Then
        subscriber.awaitTerminalEvent();
        subscriber.assertNoErrors();
 
        List<List<User>> onNextEvents = subscriber.getOnNextEvents();
        List<User> users = onNextEvents.get(0);
        Assert.assertEquals(USER_LOGIN_RIGGAROO, users.get(0).getLogin());
        Assert.assertEquals(USER_LOGIN_2_REBECCA, users.get(1).getLogin());
        verify(githubUserRestService).searchGithubUsers(USER_LOGIN_RIGGAROO);
        verify(githubUserRestService).getUser(USER_LOGIN_RIGGAROO);
        verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
    }
 
    private UsersList githubUserList() {
        User user = new User();
        user.setLogin(USER_LOGIN_RIGGAROO);
 
        User user2 = new User();
        user2.setLogin(USER_LOGIN_2_REBECCA);
 
        List<User> githubUsers = new ArrayList<>();
        githubUsers.add(user);
        githubUsers.add(user2);
        UsersList usersList = new UsersList();
        usersList.setItems(githubUsers);
        return usersList;
    }
 
    private User user1FullDetails() {
        User user = new User();
        user.setLogin(USER_LOGIN_RIGGAROO);
        user.setName("Rigs Franks");
        user.setAvatarUrl("avatar_url");
        user.setBio("Bio1");
        return user;
    }
 
    private User user2FullDetails() {
        User user = new User();
        user.setLogin(USER_LOGIN_2_REBECCA);
        user.setName("Rebecca Franks");
        user.setAvatarUrl("avatar_url2");
        user.setBio("Bio2");
        return user;
    }
 
    @Test
    public void searchUsers_IOExceptionThenSuccess_SearchUsersRetried() {
        //Given
        when(githubUserRestService.searchGithubUsers(anyString()))
                .thenReturn(getIOExceptionError(), Observable.just(githubUserList()));
        when(githubUserRestService.getUser(anyString()))
                .thenReturn(Observable.just(user1FullDetails()), Observable.just(user2FullDetails()));
 
        //When
        TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
        userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
 
        //Then
        subscriber.awaitTerminalEvent();
        subscriber.assertNoErrors();
 
        verify(githubUserRestService, times(2)).searchGithubUsers(USER_LOGIN_RIGGAROO);
 
        verify(githubUserRestService).getUser(USER_LOGIN_RIGGAROO);
        verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
    }
 
    @Test
    public void searchUsers_GetUserIOExceptionThenSuccess_SearchUsersRetried() {
        //Given
        when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(Observable.just(githubUserList()));
        when(githubUserRestService.getUser(anyString()))
                .thenReturn(getIOExceptionError(), Observable.just(user1FullDetails()),
                        Observable.just(user2FullDetails()));
 
        //When
        TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
        userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
 
        //Then
        subscriber.awaitTerminalEvent();
        subscriber.assertNoErrors();
 
        verify(githubUserRestService, times(2)).searchGithubUsers(USER_LOGIN_RIGGAROO);
 
        verify(githubUserRestService, times(2)).getUser(USER_LOGIN_RIGGAROO);
        verify(githubUserRestService).getUser(USER_LOGIN_2_REBECCA);
    }
 
    @Test
    public void searchUsers_OtherHttpError_SearchTerminatedWithError() {
        //Given
        when(githubUserRestService.searchGithubUsers(anyString())).thenReturn(get403ForbiddenError());
 
        //When
        TestSubscriber<List<User>> subscriber = new TestSubscriber<>();
        userRepository.searchUsers(USER_LOGIN_RIGGAROO).subscribe(subscriber);
 
        //Then
        subscriber.awaitTerminalEvent();
        subscriber.assertError(HttpException.class);
 
        verify(githubUserRestService).searchGithubUsers(USER_LOGIN_RIGGAROO);
 
        verify(githubUserRestService, never()).getUser(USER_LOGIN_RIGGAROO);
        verify(githubUserRestService, never()).getUser(USER_LOGIN_2_REBECCA);
    }
 
 
    private Observable getIOExceptionError() {
        return Observable.error(new IOException());
    }
 
    private Observable<UsersList> get403ForbiddenError() {
        return Observable.error(new HttpException(
                Response.error(403, ResponseBody.create(MediaType.parse("application/json"), "Forbidden"))));
 
    }
}
```

## 运行单元测试

在编写完这些测试之后，我们需要运行它们，看看它们是否通过了测试，看看有多少代码被测试覆盖了。

1. 要运行测试，您可以右键单击测试类名，并选择“Run UserRepositoryImplTest  with Coverage”
![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Run-unit-tests-with-coverage.png?resize=768%2C556&ssl=1)
1. 然后你会看到结果出现在Android Studio的右边。
![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Code-Coverage-Report-Unit-test-Android-Studio.png?resize=768%2C214&ssl=1)

	我们在UserRepositoryImpl上有100%的单元测试覆盖率。耶!

在下一篇博客文章中，我们将讨论实现UI以显示搜索结果集并为其编写更多的测试。