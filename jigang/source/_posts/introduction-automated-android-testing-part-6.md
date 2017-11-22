---
title: Android 自动化测试 - 第6部分
date: 2017-11-20 18:31:58
tags:
- Android
- 自动化测试
categories:
- Android自动化测试

---

<img src="http://szimg.mukewang.com/5850bc4500015ecd05400300-360-202.jpg" width="88%" height="320" align=center/>

在之前的5篇博客文章中，我们讨论了从头开始构建Android应用的不同方面。我们专注于在过程中包括测试。以下是与前几篇文章的链接:

* [第1部分](/2017/11/17/introduction-automated-android-testing/) - 为什么我们要写测试?
* [第2部分](/2017/11/17/automated-android-testing-part-2-setup/) - 进行测试设置你的应用程序
* [第3部分](/2017/11/17/introduction-android-testing-part3/) - 创建API调用
* [第4部分](/2017/11/17/introduction-android-testing-part-4/) - 创建repositories
* [第5部分](/2017/11/20/introduction-automated-android-testing-part-5/) - 遵循MVP模式

在本系列的最后一篇文章中，我们将介绍为我们在第5部分中创建的视图创建Espresso测试。
这篇文章的Github repo可以在[这里](https://github.com/riggaroo/GithubUsersSearchApp)找到。

如果数据是动态的，那么测试一个视图包含所期望的确切信息是很棘手的。
这些数据可以随时更改，我们的测试不应该因为它而失败。
为了使测试可靠且可重复，我们不应该调用任何生产APIs。

模拟API调用的响应将使我们能够编写依赖于模拟数据的测试。
有几种方法可以模拟我们的API调用:

* **选项1** - 使用[WireMock](http://wiremock.org/)并运行一个独立的服务器，它为特定的网络调用提供相同的静态JSON。
* **选项2** - 使用OkHttp的[MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver)，它在你的设备上运行一个webserver，并提供你所请求的任何响应。
* **选项3** - 创建一个返回虚拟对象的Retrofit REST接口的自定义实现。

显然，这个选择完全取决于您如何编写UI测试。
在我的例子中，WireMock是额外的工作，因为我需要确保我有一个独立的服务器，它运行的是一个静态IP地址。

与WireMock相比，MockWebServer要容易得多，因为您不需要设置一个独立的web服务器(服务器在设备上运行)。
MockWebServer也很灵活，因为您可以给它不同的场景。
一些有用的特性，比如指定某个调用的失败率或模拟慢速网络，都是可以使用MockWebServer实现的。([阅读更多这里](https://riggaroo.co.za/retrofit-2-mocking-http-responses/))

我将使用选项3来测试UI与模拟响应数据匹配的目的。
如果我想为慢网络环境添加测试(或某种非功能测试)，我将选择选项2。
如果您不能使用OkHttp，那么我将选择选项1作为Wiremock与任何HTTP客户机一起工作。

<!-- more -->

## 用Gradle flavors来模拟数据

通过使用Gradle flavors，我们可以很容易地模拟出API的响应。
如果你读了第二篇关于Gradle flavors的文章，你应该已经有了一个“**mock**”和“**production**”的flavor。

1. 确保您已经切换到**mockDebug** flavor。

	![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/09/Screen-Shot-2016-09-02-at-11.34.01-AM.png?ssl=1)
	
1. 	在src目录中创建一个mock文件夹。
然后在**mock**文件夹中创建一个包，它模仿了主包名。
使一个类称为`MockGithubUserRestServiceImpl`。
最终的文件结构应该是这样的:

	![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/09/Screen-Shot-2016-09-02-at-11.41.31-AM.png?ssl=1)
	
1. 	创建一个**prod**目录。
将先前定义的Injection类移动到这个文件夹中。
我们将在**mock**文件夹中创建另一个Injection类。
这个类将会注入被模拟的Github服务，而不是生产API。

	![](https://i1.wp.com/riggaroo.co.za/wp-content/uploads/2016/09/Screen-Shot-2016-09-02-at-12.14.31-PM.png?resize=612%2C1024&ssl=1)
	
	在Injection类位于**mock**文件夹中,我们简单地返回MockGithubUserServiceImpl创建。
在**prod**文件夹中，我们将返回实际的更新Github服务。

	Mock Injection类:
	
	```java
	public class Injection {
 
    	private static GithubUserRestService userRestService;
 
    	public static UserRepository provideUserRepo() {
        	return new UserRepositoryImpl(provideGithubUserRestService());
    	}
 
    	static GithubUserRestService provideGithubUserRestService() {
        	if (userRestService == null) {
            	userRestService = new MockGithubUserRestServiceImpl();
        	}
        	return userRestService;
    	}
	}
	```
	Prod注射类:
	
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
1. 	从模拟服务返回的数据依赖于您的特定需求。
下面是我`MockGithubUserRestServiceImpl`的实现:

	```java
	public class MockGithubUserRestServiceImpl implements GithubUserRestService {
    
    	private final List<User> usersList = new ArrayList<>();
    	private User dummyUser1, dummyUser2;
 
    	public MockGithubUserRestServiceImpl() {
        	dummyUser1 = new User("riggaroo", "Rebecca Franks",
                "https://riggaroo.co.za/wp-content/uploads/2016/03/rebeccafranks_circle.png", "Android Dev");
        	dummyUser2 = new User("riggaroo2", "Rebecca's Alter Ego",
                "https://s-media-cache-ak0.pinimg.com/564x/e7/cf/f3/e7cff3be614f68782386bfbeecb304b1.jpg", "A unicorn");
        	usersList.add(dummyUser1);
        	usersList.add(dummyUser2);
    	}
 
    	@Override
    	public Observable<UsersList> searchGithubUsers(final String searchTerm) {
        	return Observable.just(new UsersList(usersList));
    	}
 
    	@Override
    	public Observable<User> getUser(final String username) {
        	if (username.equals("riggaroo")) {
            	return Observable.just(dummyUser1);
        	} else if (username.equals("riggaroo2")) {
            	return Observable.just(dummyUser2);
        	}
        	return Observable.just(null);
    	}
	}
	```
	在这种情况下，我只是返回一些虚构的数据。让我们运行这个应用的模拟版本，无论你搜索什么，我们都应该得到相同的结果。
	
	![](https://i1.wp.com/riggaroo.co.za/wp-content/uploads/2016/09/gif-dummydata.gif?resize=480%2C854&ssl=1)
	
	现在我们有了一个工作虚拟应用程序！我们现在可以写Espresso UI测试。
	
## 写Espresso测试的基础

在编写一个Espresso测试时，下面的公式用于在UI中执行函数:

```java
onView(withId(R.id.menu_search))      // withId(R.id.menu_search) is a ViewMatcher
  .perform(click())               // click() is a ViewAction
  .check(matches(isDisplayed())); // matches(isDisplayed()) is a ViewAssertion
```

* **ViewMatchers** - 用于在activity中找到一个视图。
有一堆不同种类的matchers。
例如:`withId(R.id.menu_search)`、`withText("Search")`、`withTag("custom_tag")`。
* **ViewActions** - 用于与视图交互。
例如:`click()`、`doubleClick()`、`swipeUp()`、`typeText()`
* **ViewAssertations** - 用于断言某些视图具有特定的属性。
例如:`doesNotExist()`、`isAbove()`、`isBelow()`。

对于不同的Espresso方法，有一个很好的备忘单，在这里可以找到pdf格式的:[android-espresso-testing.pdf](https://google.github.io/android-testing-support-library/downloads/espresso-cheat-sheet-2.1.0.pdf)。
值得一提的是，在写Espresso测试时，可以使用普通的hamcrest matchers。
方法如`not()`, `allOf()`  和 `anyOf()`都是有效的。

## 编写Espresso UI测试

如果您还记得，在第2篇文章中，我们讨论了需要添加哪些依赖项来编写Espresso测试。
现在我们来写一份Espresso测试。

1. 创建一个文件夹**androidTestMock**。
这个文件夹中的测试只运行在**mock**的变体上，而不是在production版本上。
然后创建一个与主包名匹配的目录。
在该目录中，添加一个名为`UserSearchActivityTest`的新类。
你的项目应该是这样的:

	![](https://i1.wp.com/riggaroo.co.za/wp-content/uploads/2016/09/AndroidTestMock_folder.png?resize=768%2C703&ssl=1)
	
1. 	我们将首先编写一个基本的测试，以确保当activity启动时，文本“开始键入搜索”将显示出来:

	```java
	public class UserSearchActivityTest {
 
    	@Rule
    	public ActivityTestRule<UserSearchActivity> testRule = new ActivityTestRule<>(UserSearchActivity.class);
 
    	@Test
    	public void searchActivity_onLaunch_HintTextDisplayed(){
        	//Given activity automatically launched
        	//When user doesn't interact with the view
        	//Then
        	onView(withText("Start typing to search"))
              .check(matches(isDisplayed()));
    	}
	}
	```
	`@Rule` `ActivityTestRule`指定该测试将使用哪个activity。
在这种情况下，这个测试将使用`UserSearchActivity`运行。
这将自动启动`UserSearchActivity`。
传递额外参数将指示您是否希望该activity自动启动或不启动。

	测试`searchActivity_onLaunch_HintTextDisplayed`()是非常简单的。它在视图中搜索文本，并断言文本在UI中可见
	
1. 	下一步测试稍微复杂一点:

	```java
	@Test
    public void searchText_ReturnsCorrectlyFromWebService_DisplaysResult() {
        //Given activity is automatically launched
 
        //When
        onView(allOf(withId(R.id.menu_search), withEffectiveVisibility(ViewMatchers.Visibility.VISIBLE))).perform(
                click());  // When using a SearchView, there are two views that match the id menu_search - one that represents the icon, and the other the edit text view. We want to click on the visible one.
        onView(withId(R.id.search_src_text)).perform(typeText("riggaroo"), pressKey(KeyEvent.KEYCODE_ENTER));
 
        //Then
        onView(withText("Start typing to search")).check(matches(not(isDisplayed())));
        onView(withText("riggaroo - Rebecca Franks")).check(matches(isDisplayed()));
        onView(withText("Android Dev")).check(matches(isDisplayed()));
        onView(withText("A unicorn")).check(matches(isDisplayed()));
        onView(withText("riggaroo2 - Rebecca's Alter Ego")).check(matches(isDisplayed()));
    }
	```
	输入到`SearchView`并按enter键后，我们断言假的结果会显示在UI上。
	
1. 	我们现在已经为积极的场景编写了测试，我们也应该为这个消极的情况添加一个测试。
我们需要调整`MockGithubUserRestServiceImpl`为了让它返回自定义错误可见如果需要。

	```java
	private static Observable dummyGithubSearchResult = null;
 
    public static void setDummySearchGithubCallResult(Observable result) {
        dummyGithubSearchResult = result;
    }
    
    @Override
    public Observable<UsersList> searchGithubUsers(final String searchTerm) {
        if (dummyGithubSearchResult != null) {
            return dummyGithubSearchResult;
        }
        return Observable.just(new UsersList(usersList));
    }
	```
	在上面的代码中，创建了一个方法，目的是为搜索结果设置一个可观察的虚拟数据。
当`searchGithubUsers`()被调用时，该可观察的值将被返回。

1. 现在我们可以创建一个测试来检查是否在UI上显示了错误。

	```java
	@Test
    public void searchText_ServiceCallFails_DisplayError(){
        String errorMsg = "Server Error";
        MockGithubUserRestServiceImpl.setDummySearchGithubCallResult(Observable.error(new Exception(errorMsg)));
 
        onView(allOf(withId(R.id.menu_search), withEffectiveVisibility(ViewMatchers.Visibility.VISIBLE))).perform(
                click());  // When using a SearchView, there are two views that match the id menu_search - one that represents the icon, and the other the edit text view. We want to click on the visible one.
        onView(withId(R.id.search_src_text)).perform(typeText("riggaroo"), pressKey(KeyEvent.KEYCODE_ENTER));
 
       onView(withText(errorMsg)).check(matches(isDisplayed()));
 
    }
	```
	在这个测试中，我们首先确保服务将返回一个异常。然后，我们断言错误消息将显示在UI上。
	
1. 	让我们运行测试:

	![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/09/Passing_UI_Tests.png?resize=768%2C153&ssl=1)
	
	**他们都通过了!**
	
## Android代码覆盖率

为了了解您的测试有多有效，获得代码覆盖度量是非常棒的。

1. 为了在您的UI测试中启用代码覆盖率，可以在您的build.gradle:中添加testCoverageEnabled = true。

	```gradle
	buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            testCoverageEnabled = true
        }
    }
	```
1. 运行任务createMockDebugCoverageReport。你将在这里找到HTML报告所在地:app/build/reports/coverage/mock/debug/index.html。

	![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/09/Screen-Shot-2016-09-08-at-9.22.55-PM.png?ssl=1)
	
	Yay-我们只有模拟UI测试的82%的覆盖率。
考虑到我们在第四篇和这篇文章中看到的覆盖率报告，它可以很好地说明我们整个应用程序的测试覆盖率。
现在我们可以迭代地返回并尝试覆盖更多的代码区域。

PS-代码覆盖目前与Jack编译器不兼容。
为了使代码覆盖率报告工作，我切换到使用[Retrolambda](https://github.com/evant/gradle-retrolambda)。
如果你对学习有兴趣，可以去看看这个[分支](https://github.com/riggaroo/GithubUsersSearchApp/tree/retrolamda-usage-part6)。

## 结论

我们已经完成了我们的功能。
唷!
6的博客文章。
显然，在这款应用中还可以完成更多的测试，比如测试你的应用在低内存设备上的表现，或者在网络连接不佳的情况下，也可以添加一些非功能测试。

这一系列的结论是“引入Android自动化测试”。

## 进一步的阅读

* 先进的Espresso测试 – Chiu-Ki – [https://realm.io/news/chiu-ki-chan-advanced-android-espresso-testing/](https://realm.io/news/chiu-ki-chan-advanced-android-espresso-testing/)
* Screen Robots – [https://realm.io/news/kau-jake-wharton-testing-robots/](https://realm.io/news/kau-jake-wharton-testing-robots/)
* TestButler – [https://github.com/linkedin/test-butler](https://github.com/linkedin/test-butler)