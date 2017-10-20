---
title: '《App研发录》笔记:第一部分 高效App框架设计与重构'
date: 2017-10-20 16:11:59
tags:
	- App研发录
	- 笔记
	- App
	- Android
categories: 
	-《App研发录》笔记
	
---

[TOC]

## 第一部分 高效App框架设计与重构 ###

### 第1章 重构 ####

> 对于 App 来说，要么就一次性把它设计好，否则，就只能重构了。

#### 重新规划 Android 项目的目录结构

1. 建立 AndroidLib 类库，将与`业务无关`的逻辑转移到 AndroidLib。
	- com.infrastructure.activity: 业务无关的 Activity 基类
		- AndroidLib 下的基类 BaseActivity 封装的是业务无关的公用逻辑
		- AppBaseActivity 基类封装的是业务相关的公用逻辑
	
	![activity](http://oxwfu3w0v.bkt.clouddn.com/2017/10/20/app_yanfanl_1-1.png)
	
	- com.infrastructure.net: 网络底层封装
	- com.infrastructure.cache: 缓存数据和图片的相关处理
	- com.infrastructure.ui: 自定义控件
	- com.infrastructure.utils: 各种与业务无关的公用方法，比如对 SharedPreferences 的封装
2. 主项目中的类分门别类地进行划分
	* activity:我们按照模块继续拆分，将不同模块的 
	* Activity 划分到不同的包下。  
	* adapter:所有适配器都放在一起	* entity:将所有的实体都放在一起	* db:SQLLite 相关逻辑的封装	* engine:将业务相关的类都放在一起	* ui:将自定义控件都放在这个包中
	* utils:将所有的公用方法都放在这里 
	* interfaces:真正意义上的接口，命名以 I 作为开头
	* listener:基于 Listener 的接口，命名以 On 作为开头

> 每个文件只有一个单独的类，不要有嵌套类，比如在 Activity 中嵌套 Adapter、Entity
> 
> 将 Activity 按照模块拆分归类后，可以迅速定位具体的一个页面。
> 此外，将开发人员按照模块划分后，每个开发人员都只负责自己的那个包，开发边界线很清晰。

#### 为 Activity 定义新的生命周期

> 单一职责:一个类或方法，只做一件事情
 
 ![onCreate](http://oxwfu3w0v.bkt.clouddn.com/2017/10/20/app_yanfanl_1-5.png)
 
* `initVariables`: 初始化变量，包括 Intent 带的数据和 Activity 内的变量* `initViews`: 加载 layout 布局文件，初始化控件，为控件挂上事件方法
* `loadData`: 调用 MobileAPI 获取数据

AndroidLib 这个类库的 BaseActivity 中，重写 onCreate 方法

#### 统一事件编程模型

#### 实体化编程

使用 fastJSON 或 GSON

实体生成器

在页面跳转中使用实体

#### Adapter 模板

BaseAdapter

* getCount()* getItem()* getItemId()* getView()

`ViewHolder`  
内置一个 Holder 嵌套类，用于存放 ListView 中每一行中的控件。
ViewHolder 的存在，可以避免频繁创建同一个列表项，从而极大地节省内存。

``` Java 
class ViewHolder {	TextView tvCinemaName;	TextView tvCinemaId; 
}
```

#### 类型安全转换函数

> 因为类型转换不正确导致的崩溃占了很大的比例
 
* 空类型
* 区域越界

### 第2章 Android 网络底层框架设计

#### 网络低层封装

**抛弃 AsyncTask**

* 缺点，不能灵活控制其内部的线程池
* 没有暴露 出取消这些请求的方法

**ThreadPoolExecutor + Runnable + Handler**

`UrlConfigManager`

MobileAPI 接口的信息都 放在 url.xml 文件中:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
	<Node
		Key="getWeatherInfo" 
		Expires="300"		NetType="get"
		Url="http://www.weather.com.cn/data/sk/101010100.html" />
		
	<Node
		Key="login" 
		Expires="0"		NetType="post"
		Url="http://www.weather.com.cn/data/login.api" /></url> 

```

通过 UrlConfigManager 的 findURL 方法找到节点

`RemoteService` 和 `RequestCallback`、`RequestParameter`

``` Java
@Overrideprotected void loadData() {	weatherCallback = new RequestCallback() {
			@Override		public void onSuccess(String content) {			//TODO: 		}
				@Override		public void onFail(String errorMessage) {			//TODO: 
		}
	};
			ArrayList<RequestParameter> params = new ArrayList<RequestParameter>(); 
	RequestParameter rp1 = new RequestParameter("cityId", "111"); 
	RequestParameter rp2 = new RequestParameter("cityName", "Beijing"); 
	params.add(rp1);	params.add(rp2);	RemoteService.getInstance().invoke(this, "getWeatherInfo", params, weatherCallback);}
```

RemoteService 这个单例是用来发起请求的，它会创建一个 request，并将其添加到 RequestManager 中，然后放到 DefaultThreadPool 的一个线程中去执行这个 request.

`RequestManager`

RequestManager 这个集合类是用于取消请求(cancelRequest)的

`DefaultThreadPool`

对 ThreadPoolExecutor 和 ArrayBlockingQueue 的简单封装

`HttpRequest`

HttpRequest 是发起 Http 请求的地方，它实现了 Runnable

##### 网络底层的一些优化工作

**onFail 的统一处理机制**

如果访问 MobileAPI 请求失败，我们一般希望只是在 App 上简单地弹出一个提示框，告 诉用户网络有异常。

自定义类 AbstractRequestCallback

``` Java
public abstract class AppBaseActivity extends BaseActivity {
	public abstract class AbstractRequestCallback implements RequestCallback {
			public abstract void onSuccess(String content);
				public void onFail(String errorMessage) {			new AlertDialog.Builder(AppBaseActivity.this)
				.setTitle(" 出错啦 ").setMessage(errorMessage)
				.setPositiveButton(" 确定 ", null).show();		} 
	}}
```

**UrlConfigManager 的优化**

在 App 启动时，一次性将 url.xml 文件都读取到内存，把所有 的 UrlData 实体保存在一个集合中，然后每次调用 MobileAPI 接口，直接从内存的这个集 合中查找。

**不是每个请求都需要回调的**

**ProgressBar 的处理**

#### App 数据缓存设计

**数据缓存策略**

1. 对于 App 而言，它是感受不到取的是缓存数据还是调用 MobileAPI。具体工作由网络底层完成。
2. 在 url.xml 中为每一个 MobileAPI 接口配置缓存时间 Expired。对于 post，一律设置为 0，因为 post 不需要缓存。
3. 在 HttpRequest 类中的 run 方法中，改动 3 个地方:
	- 写一个排序算法 sortKeys，对 URL 中的 key 进行排序。
	- 将 newUrl 作为 key，检查缓存中是否有数据，有则直接返回;否则，继续调用MobileAPI 接口。 
	- MobileAPI 接口返回数据后，将数据存入缓存。
4. CacheManager 用于操作读写缓存数据，并判断缓存数据是否过期。缓存中存放的实 体就是 CacheItem。
5. 在 App 项目中，创建 YoungHeartApplication 这个 Application 级别的类，在程序启动 时，初始化缓存的目录，如果不存在则创建之。

**强制更新**

#### App MockService

#### 用户登录

##### 自动登录

Cookie

##### Cookie 过期的统一处理

##### 防止黑客刷库

一种安全解决方案是为登录接口增加第三个参数，验证码。每次登录 都必须输入验证码，其实就是为了防止被黑客刷库

##### 防止黑客刷库

#### HTTP 头中的奥妙

accept
accept-language
referrer
user-agentaccept-encoding
check-value: AppId ClientType MD5

##### 时间校准

> Date 永远使用UTC 时间
 
##### 开启 gzip 压缩

### 第3章 Android 经典场景设计

App 缓存分为两部分，数据缓存和图片缓存。

#### App 图片缓存设计

**ImageLoader 设计原理**

[ImageLoader](https://github.com/nostra13/Android-Universal-Image-Loader) 的目的是为了实现异步的网络图片 加载、缓存及显示，支持多线程异步加载。

ImageLoader 的工作原理 : 在显示图片的时候，先在内存中查找; 如果没有，就去本地查找;如果还没有，就开一个新的线程去下载这张图片，下载成功会把图片同时缓存到内存和本地。

每次退出一个页面的时候，把 ImageLoader 内存中的缓存全都清除，这样就节省了大量内存。
ImageLoader 对图片是软引用的形式，所以内存中的图片会在内存不足的时 候被系统回收(内存足够的时候不会对其进行垃圾回收)。 

**ImageLoader 优化**

在 AppBaseActivity 中的 onDestroy 方法中，执行 ImageLoader 的 clearMemoryCache 方法，以确保页面销毁时，把为了显示这个页面而增加的内存缓存清除。

#### 图片加载利器 Fresco

[Fresco](http://fresco-cn.org/docs/index.html) 的原理是，设计了一个 Image Pipeline 的概念，它负责先后检查内存、磁盘文件 (Disk)，如果都没有再老老实实从网络下载图片.

![fresco image pipeline](http://oxwfu3w0v.bkt.clouddn.com/2017/10/20/app_yanfanl_3-1.png)

Fresco 有3个线程池，其中 3个线程用于网络下载图片，2个线程用于磁盘文件的读写， 还有 2个线程用于 CPU相关操作，比如图片解码、转换，以及放在后台执行的一些费时操作.

Fresco 三层缓存:

`第一层:Bitmap 缓存`

* Android 5.0 系统中, Bitmap 缓存位于 Java 的堆(heap)中
* Android 4.x 和更低的系统，Bitmap 缓存位于 ashmem。创建和回收不会引发过多的 GC

当 App 切换到后台时，Bitmap 缓存会被清空。

`第二层:内存缓存`

内存缓存中存储了图片的原始压缩格式。从内存缓存中取出的图片，在显示前必须先解码。
当 App 切换到后台时，内存缓存也会被清空。

`第三层:磁盘缓存`

磁盘缓存，又名本地存储。磁盘缓存中存储的也是图片的原始压缩格式。在使用前也要 先解码。
当 App 切换到后台时，磁盘缓存不会丢失，即使关机也不会。

#### 对网络流量进行优化

API 层面进行优化:

* 要使用 gzip 进行压缩。注意:大于 1KB 才进行压缩， 否则得不偿失。经过 gzip 压缩后，返回的数据量大幅减少
* JSON 格式 或 ProtoBuffer
* 少网络 访问次数，能调用一次 MobileAPI 接口就能取到数据的，就不要调用两次
* TCP 长连接，以提高访问 的速度。缺点是一台服务器能支持的长连接个数不多，所以需要更多的服务器集成
* 要建立取消网络请求的机制
* 增加重试机制，get 请求配置重试机制，比如 get 请求失败后重试 3 次

#### 图片策略优化

1. 要确保下载的每张图，都符合 ImageView 控件的大小
	- `http://www.aaa.com/a.png?width=100&height=50`
2. 低流量模式
	- 在 2G 和 3G 网络环境下，我们应该适当降低图片的质量。降低图片质量，相应的图片 大小也会降低，我们称为低流量模式。
3. 极速模式
	- 在 2G 和 3G 网络环境下，用户大多对图片不感兴趣，他们可能就是想 快速下单并支付，我们需要额外设计一些页面，区别于正常模式下图文并茂的页面，这些只有文字的页面称为极速模式。

#### 城市列表的设计

* 本地保存 线上最新的城市列表数据(序列化后的)以及对应的版本号。每次发版前做一次城市数据同步。
* 每次进入到城市列表这个页面时，将本地城市列表数据对应的版本号 version 传入到 MobileAPI 接口，根据返回的 isMatch 值来决定是否版本号一致。如果一致，则直接从本地 文件中加载城市列表数据;否则，就解析 MobileAPI 接口返回的数据，在显示列表的同时， 记得要把最新的城市列表数据和版本号保存到本地。
* 如果 MobileAPI 接口没有调用成功，也是直接从本地文件中加载城市列表数据，以 确保主流程是畅通的。
* 每次调用 MobileAPI 时，会获取到大量的数据，一般我们会打开 gzip 对数据进行压 缩，以确保传输的数据量最小

**城市列表数据的增量更新机制**

#### App 与 HTML5 的交互

**App 操作 HTML5 页面的方法**

首先要定好通信协议，也就是 App 要调用的 HTML5 页面中 JavaScript 的方法名称。

HTML5:

``` html
<script type="text/javascript"> 
	function changeColor (color) {
		document.body.style.backgroundColor = color;
	}
</script>	
```

Android:

``` java
wvAds.getSettings().setJavaScriptEnabled(true);
wvAds.loadUrl("file:// /android_asset/104.html");

btnShowAlert.setOnClickListener(new View.OnClickListener() { 
	@Override	public void onClick(View v) { 
		String color = "#00ee00";		wvAds.loadUrl("javascript: changeColor ('" + color + "');"); 
	} 
});
```

**HTML5 页面操作 App 页面的方法**

仍然是先定义通信协议，这次定义的是 JavaScript 要调用的 Android 中方法名称。

HTML5:

``` html
<a onclick="baobao.callAndroidMethod(100,100,'ccc',true)"> 
	CallAndroidMethod</a>
```
Android:

``` java
class JSInteface1 {	public void callAndroidMethod(int a, float b, String c, boolean d) {		if (d) { 			String strMessage = "-" + (a + 1) + "-" + (b + 1) + "-" + c + "-" + d;			
			new AlertDialog.Builder(MainActivity.this)
				.setTitle("title")
				.setMessage(strMessage).show();		} 
	}}
```
注册 baobao 和 JSInterface1 的对应关系:

```
wvAds.addJavascriptInterface(new JSInteface1(), "baobao");
```

**App 和 HTML5 之间定义跳转协议**

在这个 HTML5 页面中，我们可以定义各种 JavaScript 点击事件，从而跳转回 App 的任 意 Native 页面。

**在 App 中内置 HTML5 页面**

**灵活切换 Native 和 HTML5 页面的策略**

经常改动的页面，做成HTML5页面， WebView的形式加载。避免了 Native页面每次修改，都要等一次迭代上线后才能看到。

HTML5 开发周期短

HTML5 的缺点是慢

1. 需要做一个后台，根据版本进行配置每个页面是使用 Native 页面还是 HTML5 页面。
2. 在 App 启动的时候，从 MobileAPI 获取到每个页面是 Native 还是 HTML5。
3. 在 App 的代码层面，页面之间要实现松耦合。为此，我们要设计一个导航器Navigator，由它来控制该跳转到 Native 页面还是 HTML5 页面。最大的挑战是页面间参数传递，字典是一种比较好的形式，消除了不同页面对参数类型的不同要求。

**页面分发器**

``` html
<a onclick="baobao.gotoAnyWhere(
		'com.example.youngheart.MovieDetailActivity, 
		iOS.MovieDetailViewController:movieId=(int)123')">			gotoAnyWhere</a>
```

``` java
public class BaseActivity extends Activity { 
	public void gotoAnyWhere(String url) { 
		if (url == null)			return;
					String pageName = getAndroidPageName(url);		if (pageName == null || pageName.trim() == "")			return;
					Intent intent = new Intent();
		
		...
		
		try {
			intent.setClass(this, Class.forName(pageName)); 
		} catch (ClassNotFoundException e) {			e.printStackTrace();
		}		startActivity(intent);
	} 
}		
```

#### 消灭全局变量

重现场景就是在 App切换到后台，闲置了一段时间后再继 续使用时，就会`崩溃`。尤其是一些配置很低的手机，

罪魁祸首就是`全局变量`

想彻底解决这个问题，就一定要使用序列化技术

**把数据作为 Intent 的参数传递**

**把全局变量序列化到本地**

序列化只是过渡型解决方案，有几个硬伤

类型 | 是否支持序列化
:---- | :------------
 简单类型 int, String, Boolean 等 | 支持 
 String[]  | 支持 
 Boolean[]  | 支持 
 int[]  | 支持 
 String[][]  | 支持 
 int[][]  | 支持 
 ArrayList  | 支持 
 Calendar  | 支持 
 JSONObject  | 不支持 
 JSONArray   | 不支持
  HashMap<String, Objeject> | 因为 Object 可能是不支持序列化的 JSONObject 类型，所以HashMap<String, Object> 不一定支持序列化
  ArrayList<HashMap<String, Object>> | 因为 Object 可能是不支持序列化的 JSONObject 类型，所以 ArrayList<HashMap<String, Object>> 不一定支持序列化

**如果 Activity 也被销毁了呢**

如果内存不足导致当前 Activity 也被销毁了呢?比如说旋转屏幕从竖屏到横屏。

即使 Activity 被销毁了，传递到这个 Activity 的 Intent 并不会丢失，在重新执行 Activity 的 onCreate 方法时，Intent 携带的 bundle 参数还是在的。

* onSaveInstanceState()* onRestoreInstanceState()

**如何看待 SharedPreferences**

SharedPreferences 是全局变量序列化到本地的另一种形式

**User 是唯一例外的全局变量**


### 第4章 Android 命名规范和编码规范

> 尽量简单，多写注释

#### Android 命名规范

**Java 类文件命名规范：**

* Activity: 以Activity作为后缀，比如 PersonActivity
* Adapter: 以Adapter作为后缀，比如 PersonAdapter
* Entity: 以Entity作为后缀，比如 PersonEntity 

**资源文件命名规范：**

* layout 
	- 页面布局文件： `act_`为前缀，以Activity所在的 Package 为中缀，以Activity的名称(去掉Activity)作为后缀。都是小写。比如 act_person_customer.xml
	- ListView中的item布局文件：`item_`为前缀，ListView控件名称为后缀。 比如控件名称lvUserList时item_lv_user_list.xml
	- Dialog布局文件：`dlg_` + 名称
* drawable
	- 对于只在一个页面使用的资源，以该页面的名称为前缀
	- 对于只在一个模块下多个页面使用的资源，以该模块的名称为前缀
	- 对于在各个模块，各个页面都有可能使用的资源，比如下导航，下导航，以common作为前缀

**Java 类中控件对象的命名规范：**

控件类型缩写 + 控件的逻辑名称（首字母大写）  比如登录按钮：btnLogin

控 件 | 缩写 
---- | ---
LayoutView | lv
RelativeView | rv
TextView | tv
Button | btn
ImageButton | img
ImageView | iv
CheckBox | chk
RadioButton | rb
DatePicker | dp
EditText	| et
TimePicker | tp
toggleButton | tb
ProgressBar | pb
WebView | wv
RantingBar | rb
Tab	| tab
List | lv
MapView | mv

**Layout 中控件对象的命名规范**

控件的逻辑名称（小写） + 控件类型   比如： login_button

**string.xml 中常量的命名规范**

loginActivity_btnLogin_text

common_

**Java 中的常量命名**

只能包括字母和下划线_，字母全大写，单词之间用 _ 隔空

#### Android 编程规范

> 不能为了规范而规范

* 要分门别类存放各种类
* 要怎么使用findViewById语句？
* Layout中的常量，要在资源strings.xml中定义
* Layout中的所有控件的字体大小，在dimens.xml中
* 在Acitivity中定义新的生命周期，拆分onCreate (initVariables -> initViews -> loadData)
* 使用fastJSON实体化数据
* 页面之间传值，使用Intent携带序列化实体数据
* 为控件添加事件，统一使用方法
* Activity中不要嵌套内部类，尽量独立出来
* Adapter
	* 所有Adapter，都放在 adapter 包中
	* Adapter绑定的数据，一律为 ArrayList<自定义可序列化实体>
	* 在Adapter中创建适合于列表自身的ViewHolder实体类，统一命名为 ViewHolder
* 实体不要在不同模块间共享，但可以在同一模块下的不同页面间共享
* 为节省内存，请使用ArrayList<自定义实体>，而不是HashMap
* 图片处理，请使用第三方库
* 什么时候使用SharedPreferences？ 简单配置信息
* 尽量使用ApplicationContext代替Context，防止内存泄漏
* 数据类型转换一定要进行校验，防止空指针或类型转换失败
* 使用常量代替枚举


#### 统一代码格式

Android源码中包含了一份 android-formatting.xml, 专门用于统一代码格式。 
导入 IDE后，使用快捷键，就可以调整代码格式

自动检查工具 checkstyle 


