---
title: Anko 布局
date: 2017-11-02 18:56:23
tags:
- Android
- Anko
- Kotlin
categories:
- Anko

---

![anko](https://github.com/Kotlin/anko/blob/master/doc/logo.png?raw=true)

* [为什么Anko布局?](#why_anko_layouts)
	* [DSL的原因?](#why_dsl)
	* [支持现有的代码](#existing_code)
	* [它是如何工作的](#how_works)
	* [它是可扩展的吗?](#extensible)
* [在你的项目中使用Anko Layouts](#using_anko_layouts)
* [理解Anko](#understanding_anko)
	* [基础知识](#basics)
	* [AnkoComponent](#ankocomponent)
	* [辅助块](#helper_blocks)
	* [主题块](#themed_blocks)
	* [布局和LayoutParams](#layouts_params)
	* [Listeners](#listeners)
	* [定制的协程上下文](#custom_coroutine_context)
	* [使用资源标识符](#using_resource_id)
	* [实例简化符号](#instance_shorthand_notation)
	* [UI包装](#ui_wrapper)
	* [Include标签](#include_tag)
* [Anko支持插件](#anko_support_plugin)
	* [安装Anko支持插件](#installing_anko_support_plugin)
	* [使用插件](#using_anko_support_plugin)
	* [XML DSL转换器](#xml_dsl_converter)

<!-- more -->

## <a name="why_anko_layouts"></a>为什么Anko布局?

### <a name="why_dsl"></a>DSL的原因?

默认情况下，Android中的UI是用XML编写的。这在以下方面是不方便的:

* 不是类型安全;
* 不是空值安全;
* 它迫使您为每一个布局编写几乎相同的代码;
* 在设备上解析XML，浪费CPU时间和电池;
* 最重要的是，它不允许代码重用。

虽然您可以通过编程方式创建UI，但这几乎是不可能完成的，因为它有点难看，很难维护。这是一个普通的Kotlin版本(Java中的一个更长的版本):

```Kotlin
val act = this
val layout = LinearLayout(act)
layout.orientation = LinearLayout.VERTICAL
val name = EditText(act)
val button = Button(act)
button.text = "Say Hello"
button.setOnClickListener {
    Toast.makeText(act, "Hello, ${name.text}!", Toast.LENGTH_SHORT).show()
}
layout.addView(name)
layout.addView(button)
```

DSL使相同的逻辑易于阅读，易于编写，并且没有运行时开销。这里又有了:

```Kotlin
verticalLayout {
    val name = editText()
    button("Say Hello") {
        onClick { toast("Hello, ${name.text}!") }
    }
}
```

注意，`onClick()`支持协程(接受suspending lambda)，这样您就可以在没有显式的`async(UI)`调用的情况下编写异步代码。

### <a name="existing_code"></a>支持现有的代码

你不需要重写Anko的所有UI。您可以用Java编写旧的类。此外，如果您仍然希望(或有)编写一个Kotlin activity类，并出于某种原因使XML布局膨胀，您可以使用视图属性，这将使事情变得更简单:

```Kotlin
// 与findViewById()相同，但使用起来更简单
val name = find<TextView>(R.id.name)
name.hint = "Enter your name"
name.onClick { /*do something*/ }
```

通过使用[Kotlin的Android扩展](https://kotlinlang.org/docs/tutorials/android-plugin.html)，您可以使您的代码更加紧凑。

### <a name="how_works"></a>它是如何工作的

没有🎩。Anko由一些Kotlin的[扩展函数和属性](http://kotlinlang.org/docs/reference/extensions.html)组成了类型安全的构造器，就像[类型安全的构建器](http://kotlinlang.org/docs/reference/type-safe-builders.html)所描述的那样。

由于手工编写所有这些扩展有点乏味，所以它们是由Android SDK中的android.jar文件自动生成的。

### <a name="extensible"></a>它是可扩展的吗?

简短的回答:**是的**。

例如，您可能想要在DSL中使用MapView。然后把它写在任何可以导入的Kotlin文件中:

```Kotlin
inline fun ViewManager.mapView() = mapView(theme = 0) {}

inline fun ViewManager.mapView(init: MapView.() -> Unit): MapView {
    return ankoView({ MapView(it) }, theme = 0, init = init)
}
```

`{ MapView(it) }`是您自定义`View`的工厂函数。它接受一个`Context`实例

现在你可以这样写了:

```Kotlin
frameLayout {
    val mapView = mapView().lparams(width = matchParent)
}
```

如果你想让你的用户能够应用一个定制的主题，你也可以这样写:

```Kotlin
inline fun ViewManager.mapView(theme: Int = 0) = mapView(theme) {}

inline fun ViewManager.mapView(theme: Int = 0, init: MapView.() -> Unit): MapView {
    return ankoView({ MapView(it) }, theme, init)
}
```

## <a name="using_anko_layouts"></a>在你的项目中使用Anko Layouts

这些库依赖关系包括:

```Gradle
dependencies {
    // Anko Layouts
    compile "org.jetbrains.anko:anko-sdk25:$anko_version" // sdk15, sdk19, sdk21, sdk23 are also available
    compile "org.jetbrains.anko:anko-appcompat-v7:$anko_version"

    // Coroutine listeners for Anko Layouts
    compile "org.jetbrains.anko:anko-sdk25-coroutines:$anko_version"
    compile "org.jetbrains.anko:anko-appcompat-v7-coroutines:$anko_version"
}
```

请阅读基于[Gradle的项目](https://github.com/Kotlin/anko#gradle-based-project)部分，以获得详细信息。

## <a name="understanding_anko"></a>理解Anko
### <a name="basics"></a>基础知识

在Anko，你不需要从任何特殊的类继承:仅仅使用标准的`Activity`、`Fragment`、`FragmentActivity`或任何你想要的东西。

首先，导入org.jetbrains.anko.*。在您的类中使用Anko布局DSL。

在`onCreate()`中可以使用DSL:

```Kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    verticalLayout {
        padding = dip(30)
        editText {
            hint = "Name"
            textSize = 24f
        }
        editText {
            hint = "Password"
            textSize = 24f
        }
        button("Login") {
            textSize = 26f
        }
    }
}
```

> 🐧 没有明确的调用`setContentView(R.layout.something)`: Anko为`Activities`自动设置内容视图(但只对他们)。

`hint`和`textSize`是与java风格的getter和setter绑定的[合成扩展属性](https://kotlinlang.org/docs/reference/java-interop.html#getters-and-setters)，`padding`是Anko的[扩展属性](http://kotlinlang.org/docs/reference/extensions.html#extension-properties)。这些都存在于几乎所有View属性中，允许您编写`text = "Some text"`而不是`setText("Some text")`。

`verticalLayout`(一个`线性布局`，但已经有了`LinearLayout.VERTICAL`方向)、`editText`和`button`是构造新Viwe实例并将它们添加到父类的[扩展函数](http://kotlinlang.org/docs/reference/extensions.html)。我们将引用这些函数作为块。

在Android框架中几乎所有的视图都存在块，它们在Activities、Fragments(默认情况下和android.support package)甚至是Context。例如，如果您有一个AnkoContext实例，您可以编写这样的块:

```Kotlin
val name: EditText = with(ankoContext) {
    editText {
        hint = "Name"
    }
}
```

### <a name="ankocomponent"></a>AnkoComponent

尽管您可以直接使用DSL(在onCreate()或其他任何地方)，但不需要创建任何额外的类，在单独的类中使用UI通常是很方便的。如果您使用了提供的AnkoComponent接口，那么您还可以免费获得[DSL布局预览](https://github.com/Kotlin/anko/blob/master/doc/preview.png)功能。

```Kotlin
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?, persistentState: PersistableBundle?) {
        super.onCreate(savedInstanceState, persistentState)
        MyActivityUI().setContentView(this)
    }
}

class MyActivityUI : AnkoComponent<MyActivity> {
    override fun createView(ui: AnkoContext<MyActivity>) = with(ui) {
        verticalLayout {
            val name = editText()
            button("Say Hello") {
                onClick { ctx.toast("Hello, ${name.text}!") }
            }
        }
    }
}
```

### <a name="helper_blocks"></a>辅助块

正如您可能已经注意到的，前一节中的`button()`函数接受一个字符串参数。
对于常用的View，例如`TextView`、`EditText`、`Button`或`ImageView`，都存在这样的辅助块。

如果您不需要为某个特定View设置任何属性，您可以省略 {}，编写button("Ok")，甚至只是button():

```Kotlin
verticalLayout {
    button("Ok")
    button(R.string.cancel)
}
```

### <a name="themed_blocks"></a>主题块

Anko提供了“themeable”版本，包括辅助块:

```Kotlin
verticalLayout {
    themedButton("Ok", theme = R.style.myTheme)
}
```

### <a name="layouts_params"></a>布局和`LayoutParams`

可以使用`LayoutParams`对父容器中的小部件进行定位。在XML中，它是这样的:

```xml
<ImageView 
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginLeft="5dip"
    android:layout_marginTop="10dip"
    android:src="@drawable/something" />
```

在Anko中，使用lparams()在View描述后，指定了LayoutParams。

```Kotlin
linearLayout {
    button("Login") {
        textSize = 26f
    }.lparams(width = wrapContent) {
        horizontalMargin = dip(5)
        topMargin = dip(10)
    }
}
```

如果您指定了`lparams()`，但是省略了`width`和/或`height`，它们的默认值都是`wrapContent`。但是您总是可以显式地传递它们:使用[命名参数](http://kotlinlang.org/docs/reference/functions.html#named-arguments)。

一些方便的辅助属性可以注意到:

* `horizontalMargin` 设置左和右的边距
* `verticalMargin` 设置顶部和底部的边距
* `margin` 同时设置了所有四个边距

注意，对于不同的布局，`lparams()`是不同的，例如，在`相对布局`的情况下:

```Kotlin
val ID_OK = 1

relativeLayout {
    button("Ok") {
        id = ID_OK
    }.lparams { alignParentTop() }
  
    button("Cancel").lparams { below(ID_OK) }
}
```

### <a name="listeners"></a>Listeners

Anko有一名监听器的助手，他们可以无缝地支持协程。您可以在listener内编写异步代码！

```Kotlin
button("Login") {
    onClick {
    	val user = myRetrofitService.getUser().await()
        showUser(user)
    }
}
```

几乎和这个是一样的:

```Kotlin
button.setOnClickListener(object : OnClickListener {
    override fun onClick(v: View) {
    	launch(UI) {
    	    val user = myRetrofitService.getUser().await()
            showUser(user)
    	}
    }
})
```

当你有很多方法的时候，Anko是很有帮助的。考虑下面的代码，没有使用Anko:

```Kotlin
seekBar.setOnSeekBarChangeListener(object : OnSeekBarChangeListener {
    override fun onProgressChanged(seekBar: SeekBar, progress: Int, fromUser: Boolean) {
        // Something
    }
    override fun onStartTrackingTouch(seekBar: SeekBar?) {
        // Just an empty method
    }
    override fun onStopTrackingTouch(seekBar: SeekBar) {
        // Another empty method
    }
})
```

现在,Anko:

```Kotlin
seekBar {
    onSeekBarChangeListener {
        onProgressChanged { seekBar, progress, fromUser ->
            // Something
        }
    }
}
```

如果您为相同的`View`设置`onProgressChanged()`和`onStartTrackingTouch()`，那么这两个“部分定义”的listener将被合并。对于相同的listener方法，最后一个获胜。

### <a name="custom_coroutine_context"></a>定制的协程上下文

您可以将一个定制的协程context传递给listener助手:

```Kotlin
button("Login") {
    onClick(yourContext) {
    	val user = myRetrofitService.getUser().await()
        showUser(user)
    }
}
```

### <a name="using_resource_id"></a>使用资源标识符

前面几章中的所有例子都使用了原始Java字符串，但这并不是一个好的实践。通常，您将所有的字符串数据放入`res/values/`目录中，并在运行时调用它，例如`getString(R.string.login)`。

幸运的是，在Anko中，您可以将资源标识符传递给助手块(`button(R.string.login)`)和扩展属性(`button { textResource = R.string.login }`)。

注意，属性名不是相同的: `text`、`hint`、`image`，而是我们现在使用的是`textResource`、`hintResource`和`imageResource`

> 🐧 资源属性读时总是抛出AnkoException。

### <a name="instance_shorthand_notation"></a>实例简化符号

有时候，您需要从活动代码中传递一个`Context`实例到某个Android SDK方法。
通常，你可以使用`this`，但是如果你在一个内部类里面呢?
如果您在Kotlin编写，您可能会在Java和this@SomeActivity用SomeActivity.this。

对Anko，你可以写ctx。它是一个扩展属性，既可以在Activity和Service中工作，也可以从Fragment中访问(它的外壳是使用getActivity()方法)。
您还可以使用act扩展属性获得一个Activity实例。

### <a name="ui_wrapper"></a>UI包装

在开始之前，Anko总是使用UI标签作为顶级的DSL元素:

```Kotlin
UI {
    editText {
        hint = "Name"
    }
}
```

如果你想的话，你仍然可以使用这个标签。而且，扩展DSL要容易得多，因为您必须只声明一个ViewManager.customView函数。请参见[扩展Anko](#extensible)获取更多信息。

### <a name="include_tag"></a>Include标签

可以很容易将XML布局插入到DSL中。
使用include()函数:

```Kotlin
include<View>(R.layout.something) {
    backgroundColor = Color.RED
}.lparams(width = matchParent) { margin = dip(12) }
```

您可以像往常一样使用lparams()，如果您提供了一个特定的类型而不是View，您也可以使用这个类型:

```Kotlin
include<TextView>(R.layout.textfield) {
    text = "Hello, world!"
}
```

## <a name="anko_support_plugin"></a>Anko支持插件

Anko支持插件可用于IntelliJ IDEA和Android Studio。它允许你在IDE工具窗口中直接预览与Anko一起编写的AnkoComponent类。

> ⚠️ Anko支持插件目前只支持Android Studio 2.4+。

### <a name="installing_anko_support_plugin"></a>安装Anko支持插件

你可以在[这里](https://plugins.jetbrains.com/update/index?pr=&updateId=19242)下载Anko的支持插件。

### <a name="using_anko_support_plugin"></a>使用插件

假设你用Anko写了这些classes：

```Kotlin
class MyActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?, persistentState: PersistableBundle?) {
        super.onCreate(savedInstanceState, persistentState)
        MyActivityUI().setContentView(this)
    }
}

class MyActivityUI : AnkoComponent<MyActivity> {
    override fun createView(ui: AnkoContext<MyActivity>) = ui.apply {
        verticalLayout {
            val name = editText()
            button("Say Hello") {
                onClick { ctx.toast("Hello, ${name.text}!") }
            }
        }
    }.view
}
```

将光标放在`MyActivityUI`声明的地方，打开Anko布局预览工具窗口("View" → "Tool Windows" → "Anko Layout Preview")，并按下刷新。

这需要构建项目，因此在实际显示图像之前需要花费一些时间。

### <a name="xml_dsl_converter"></a>XML DSL转换器

该插件还支持将XML格式的布局转换为Anko布局代码。
打开一个XML文件并选择"Code" → "Convert to Anko Layouts DSL"。
您可以同时转换多个XML布局文件。