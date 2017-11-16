---
title: 自定义视图(1) —— 创建自定义视图类
date: 2017-08-10 20:02:30
tags:
- Android
- UI
categories:
- 'Android 用户界面的最佳实践'

---

![android](http://oxwfu3w0v.bkt.clouddn.com/qiniu.jpg)


一个设计良好的自定义视图与其他设计良好的类非常相似。
它封装了一个特定的功能集，它使用了一个简单的接口，它有效地使用CPU和内存，等等。
除了是一个设计良好的类之外，自定义视图应该是:

* 符合Android标准
* 提供与Android XML布局兼容的定制样式属性
* 发送访问事件
* 与多个Android平台兼容

Android框架提供了一组基类和XML标记，以帮助您创建满足所有这些需求的视图。
本课程将讨论如何使用Android框架来创建视图类的核心功能。

<!-- more -->

## 子类化View

在Android框架中定义的所有视图类都扩展[View](https://developer.android.google.cn/reference/android/view/View.html)。
您的自定义视图也可以直接扩展`View`，或者您可以通过扩展现有的视图子类，例如[Button](https://developer.android.google.cn/reference/android/widget/Button.html)来节省时间。

为了让Android Studio能够与视图交互，至少必须提供一个构造函数，该构造函数将[Context](https://developer.android.google.cn/reference/android/content/Context.html)和[AttributeSet](https://developer.android.google.cn/reference/android/util/AttributeSet.html)对象作为参数。
这个构造函数允许布局编辑器创建和编辑视图的实例。

```java
class PieChart extends View {
    public PieChart(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

## 定义自定义属性

要向用户界面添加内置视图，需要在XML元素中指定它，并使用元素属性控制其外观和行为。
编写良好的自定义视图也可以通过*XML*添加和样式化。
要在您的自定义视图中启用该行为，您必须:

* 在`<declare-styleable>`资源元素中为视图定义定制属性
* 为XML布局中的属性指定值
* 在运行时检索属性值
* 将检索到的属性值应用到您的视图中

本节讨论如何定义自定义属性并指定它们的值。
下一节将讨论如何在运行时检索和应用这些值。

要定义自定义属性，请将`<declare-styleable>`资源添加到项目中。
按照惯例将这些资源放入一个`res/values/attrs.xml`文件中。
下面是一个`attrs.xml`文件的例子:

```xml
<resources>
   <declare-styleable name="PieChart">
       <attr name="showText" format="boolean" />
       <attr name="labelPosition" format="enum">
           <enum name="left" value="0"/>
           <enum name="right" value="1"/>
       </attr>
   </declare-styleable>
</resources>
```

这段代码声明了两个自定义属性，`showText`和`labelPosition`，它们属于一个名为`PieChart`的样式实体。
根据惯例，样式化实体的名称与定义自定义视图的类的名称相同。
虽然没有必要遵循这个惯例，但是许多流行的代码编辑器都依赖于这个命名约定来提供语句完成。

一旦定义了定制属性，就可以在布局XML文件中使用它们，就像内置的属性一样。
惟一的区别是，您的自定义属性属于不同的名称空间。
而不是属于http://schemas.android.com/apk/res/android名称空间,他们属于http://schemas.android.com/apk/res/(你的包名称)。
例如，下面是如何使用为`PieChart`定义的属性:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:custom="http://schemas.android.com/apk/res/com.example.customviews">
 <com.example.customviews.charting.PieChart
     custom:showText="true"
     custom:labelPosition="left" />
</LinearLayout>
```

为了避免重复冗长的名称空间URI，该示例使用了`xmlns`指令。
这个指令分配别名定义命名空间`http://schemas.android.com/apk/res/com.example.customviews`。
您可以为您的命名空间选择任何别名。

请注意将自定义视图添加到布局的XML标记的名称。
它是自定义视图类的完全限定名。
如果您的视图类是内部类，则必须使用视图的外层类的名称进一步限定它。
进一步。例如，`PieChart`类有一个名为`PieView`的内部类。
使用这个类的自定义属性,可以使用标签`com.example.customviews.charting.PieChart$PieView`。

## 应用自定义属性

当从XML布局创建视图时，XML标记中的所有属性都是从资源包中读取的，并作为AttributeSet传递到视图的构造函数中。
虽然可以直接从[AttributeSet](https://developer.android.google.cn/reference/android/util/AttributeSet.html)中读取值，但是这样做有一些缺点:

* 属性值中的资源引用没有被解析
* 不能应用风格

相反，将[AttributeSet](https://developer.android.google.cn/reference/android/util/AttributeSet.html)传递到[obtainStyledAttributes()](https://developer.android.google.cn/reference/android/content/res/Resources.Theme.html#obtainStyledAttributes(android.util.AttributeSet,%20int[],%20int,%20int))。
这个方法传递了一个已经被取消和样式化的值的TypedArray数组。

Android资源编译器为您做了很多工作，使调用obtainStyledAttributes()更容易。
对于res目录中的每个<declare-styleable>资源，生成的R.java定义了一个属性id数组和一组常量，这些常量为数组中的每个属性定义索引。
您可以使用预定义的常量来读取TypedArray的属性。
下面是压电图类如何读取它的属性:

```java
public PieChart(Context context, AttributeSet attrs) {
   super(context, attrs);
   TypedArray a = context.getTheme().obtainStyledAttributes(
        attrs,
        R.styleable.PieChart,
        0, 0);

   try {
       mShowText = a.getBoolean(R.styleable.PieChart_showText, false);
       mTextPos = a.getInteger(R.styleable.PieChart_labelPosition, 0);
   } finally {
       a.recycle();
   }
}
```

注意，[TypedArray](https://developer.android.google.cn/reference/android/content/res/TypedArray.html)对象是一个共享资源，必须在使用后回收。

## 添加属性和事件

属性是控制视图的行为和外观的一种强大的方式，但是只有在视图初始化时才能读取它们。
为了提供动态行为，为每个定制属性公开一个属性对 getter和setter。
下面的代码片段展示了PieChart如何公开一个名为showText的属性:

```java
public boolean isShowText() {
   return mShowText;
}

public void setShowText(boolean showText) {
   mShowText = showText;
   invalidate();
   requestLayout();
}
```

注意，`setShowText`调用[invalidate()](https://developer.android.google.cn/reference/android/view/View.html#invalidate())和[requestLayout()](https://developer.android.google.cn/reference/android/view/View.html#requestLayout())。
这些调用对于确保视图的行为可靠是至关重要的。
您必须在对可能**更改其外观的属性**进行任何更改之后，使视图`invalidate`，以便系统知道需要重新绘制它。
同样地，如果属性更改可能**影响视图的大小或形状**，则需要**请求一个新的布局**。
忘记这些方法调用会导致难以找到的bug。

自定义视图还应该支持事件侦听器来通信重要事件。
例如，PieChart公开一个名为OnCurrentItemChanged的自定义事件，以通知侦听器告知，用户已经将饼图转到一个新的饼图上。

很容易忘记暴露属性和事件，尤其是当您是自定义视图的唯一用户时。
花一些时间仔细地定义视图的接口可以减少将来的维护成本。
一个好的规则是始终公开任何影响您的自定义视图的可见外观或行为的属性。

## 可访问性的设计

您的自定义视图应该支持最广泛的用户。
这包括有残疾的用户阻止他们看到或使用触摸屏。
为了支持残疾用户，您应该:

* 标签你输入字段使用`android:contentDescription`属性
* 在适当的时候，通过调用[sendAccessibilityEvent()](https://developer.android.google.cn/reference/android/view/accessibility/AccessibilityEventSource.html#sendAccessibilityEvent(int)) 来发送可访问性事件。
* 支持备用控制器，如D-pad和轨迹球

有关创建可访问视图的更多信息，请参见在Android开发人员指南中[可访问的应用程序](https://developer.android.google.cn/guide/topics/ui/accessibility/apps.html#custom-views)。
