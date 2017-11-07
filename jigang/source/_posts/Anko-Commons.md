---
title: Anko 通用模块
date: 2017-11-02 17:24:52
tags:
- Android
- Anko
- Kotlin
categories:
- Anko

---

![anko](https://github.com/Kotlin/anko/blob/master/doc/logo.png?raw=true)

## 目录 ##

1. [在您的项目中使用anko-commons助手](#anko_commons)
1. [Intents](#intents)
	1. [Intent构建器函数](#intent_builder)
	1. [有用Intent调用](#useful_intent)
1. [Dialogs](#dialogs)
	1. [Toasts](#toasts)
	1. [SnackBars](#snackbars)
	1. [Alerts](#alerts)
	1. [Selectors](#selectors)
	1. [进度dialogs](#progress)
1. [日志](#logging)
	1. [强风格](#trait_style)
	1. [日志记录器对象风格](#logger_object_style)
1. [Misc](#misc)
	1. [颜色](#colors)
	1. [维度值](#dimensions)
	1. [applyRecursively()](#applyRecursively)

<!-- more -->

## <a name="anko_commons"></a>在您的项目中使用anko-commons助手

添加`anko-commons`到对您的`build.gradle`的依赖:

```gradle
dependencies {
    compile "org.jetbrains.anko:anko-commons:$anko_version"
    compile "org.jetbrains.anko:anko-design:$anko_version" // For SnackBars
}
```

## <a name="intents"></a>Intents

Intent助手位于anko-commons工件内。

### <a name="intent_builder"></a>Intent构建器函数

通常，您必须编写多行来启动一个新的活动。它还要求你为传输额外的值写额外的行。例如，用extra ("id", 5)和一个特殊的标志来启动一个活动的代码：

```Kotlin
val intent = Intent(this, SomeOtherActivity::class.java)
intent.putExtra("id", 5)
intent.setFlag(Intent.FLAG_ACTIVITY_SINGLE_TOP)
startActivity(intent)
```

四行太过分了。Anko向你提供了一个更简单的方法:

```Kotlin
startActivity(intentFor<SomeOtherActivity>("id" to 5).singleTop())
```

如果您不需要传递任何标志，那么解决方案就更简单了:

```Kotlin
startActivity<SomeOtherActivity>("id" to 5)
```

### <a name="useful_intent"></a>有用Intent调用

Anko对一些广泛使用的意图进行了包装:

目标  | 解决方案
------------- | -------------
打电话 | `makeCall(number)`
发送短信  | `sendSMS(number, [text])`
浏览网页	| `browse(url)`
分享文本内容 | `share(text, [subject])`
发送电子邮件 | `email(email, [subject], [text])`

方括号中的参数(`[]`)是可选的。如果发送了意图，方法返回true。

## <a name="dialogs"></a>Dialogs

Dialog助手位于anko-commons工件内。

### <a name="toasts"></a>Toasts

简单地展示一个[Toast](https://developer.android.google.cn/guide/topics/ui/notifiers/toasts.html)的信息。

```Kotlin
toast("Hi there!")
toast(R.string.message)
longToast("Wow, such duration")
```

### <a name="snackbars"></a>SnackBars

简单地显示一条[SnackBar](https://developer.android.google.cn/reference/android/support/design/widget/Snackbar.html)消息。

```Kotlin
snackbar(view, "Hi there!")
snackbar(view, R.string.message)
longSnackbar(view, "Wow, such duration")
snackbar(view, "Action, reaction", "Click me!") { doStuff() }
```

### <a name="alerts"></a>Alerts

一个用于显示[alert dialogs](https://developer.android.google.cn/guide/topics/ui/dialogs.html)的简单DSL。

```Kotlin
alert("Hi, I'm Roy", "Have you tried turning it off and on again?") {
    yesButton { toast("Oh…") }
    noButton {}
}.show()
```

上面的代码将显示默认的Android警告对话框。
如果您想切换到appcompat实现，请使用`Appcompat`对话框:

```Kotlin
alert(Appcompat, "Some text message").show()
```

默认情况下包括`Android`和`Appcompat`对话框，但是您可以通过实现`AlertBuilderFactory`接口来创建您的定制工厂。

alert()功能无缝地支持Anko的布局，作为自定义视图:

```Kotlin
alert {
    customView {
        editText()
    }
}.show()
```

### <a name="selectors"></a>Selectors

selector()显示一个带有文本条目列表的警告对话框:

```Kotlin
val countries = listOf("Russia", "USA", "Japan", "Australia")
selector("Where are you from?", countries, { dialogInterface, i ->
    toast("So you're living in ${countries[i]}, right?")
})
```

### <a name="progress"></a>进度dialogs

progressDialog()创建并显示一个[进度对话框](https://developer.android.google.cn/reference/android/app/ProgressDialog.html)。

```Kotlin
val dialog = progressDialog(message = "Please wait a bit…", title = "Fetching data")
```

一个不确定的进度对话框也有(见`indeterminateProgressDialog`())。

## <a name="logging"></a>日志

`AnkoLogger`位于anko-commons工件内。

### <a name="trait_style"></a>强风格

Android SDK提供了[android.util.Log](https://developer.android.google.cn/reference/android/util/Log.html)类，并提供了一些日志记录方法。使用方法非常简单，但是方法要求您传递一个标记参数。你可以通过使用`AnkoLogger`的接口来消除这个问题:

```Kotlin
class SomeActivity : Activity(), AnkoLogger {
    private fun someMethod() {
        info("London is the capital of Great Britain")
        debug(5) // .toString() method will be executed
        warn(null) // "null" will be printed
    }
}
```

android.util.Log	| AnkoLogger
------------- | -------------
`v()`		| `verbose()`
`d()`		| `debug()`
`i()`	| `info()`
`w()`		| `warn()`
`e()`	| `error()`
`wtf()`	| `wtf()`

默认的标记名是一个类名(在本例中是SomeActivity)，但是您可以通过覆盖`loggerTag`属性来轻松地更改它。

每个方法有两个版本:普通和懒惰(内联):

```Kotlin
info("String " + "concatenation")
info { "String " + "concatenation" }
```

只有`Log.isLoggable(tag, Log.INFO)`是`true`，Lambda结果才会计算

### <a name="logger_object_style"></a>日志记录器对象风格

你也可以用`AnkoLogger`作为一个简单的对象。

```Kotlin
class SomeActivity : Activity() {
    private val log = AnkoLogger<SomeActivity>(this)
    private val logWithASpecificTag = AnkoLogger("my_tag")

    private fun someMethod() {
        log.warning("Big brother is watching you!")
    }
}
```

## <a name="misc"></a>Misc

### <a name="colors"></a>颜色

两个简单的扩展函数使代码更加可读。

函数	| 结果
------------- | -------------
0xff0000.opaque		| 不透明的红色
0x99.gray.opaque		| 灰色不透明的#999999

### <a name="dimensions"></a>维度值

您可以在**dip**(密度-独立像素)或**sp**(独立像素)中指定维度值:`dip(dipValue)`或`sp(spValue)`。
注意，textSize属性已经接受**sp**(`textSize=16f`)。使用`px2dip`和`px2sp`进行反向转换。

### <a name="applyRecursively"></a>applyRecursively()

`applyRecursively()`将lambda表达式应用到传递`View`本身，然后递归地对视图中的每个子视图进行递归，如果是`ViewGroup`：

```Kotlin
verticalLayout {
    editText {
        hint = "Name"
    }
    editText {
        hint = "Password"
    }
}.applyRecursively { view -> when(view) {
    is EditText -> view.textSize = 20f
}}
```