---
title: Android Theme 和 Style
date: 2017-10-09 17:02:11
tags:
- Android
- 'Android UI'
categories:
- Android

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1513857484893&di=f44d6937a4f10732b39e203b321a4816&imgtype=0&src=http%3A%2F%2Fpic.58pic.com%2F58pic%2F15%2F24%2F14%2F87658PICnfX_1024.jpg)

Android 5.0 带来了新的功能，允许你为视图(以及任何后代)指定一个覆盖主题。下面介绍如何使用它以及为什么要使用它。

<!-- more -->

## 为什么?
您可能已经在不知道的情况下使用了这个功能:`Theme.Holo.Light.DarkActionBar`。

考虑一下Light.DarkActionBar主题。内容是亮系的主题(背景是亮的，前景是暗的)，但是动作条使用了一个暗的主题(暗背景和亮前景色)。

如果不能提供一个单独的主题，您需要手动将文本颜色和其他前景颜色设置为某种相反色。

这里就是旧的actionBarWidgetTheme属性出现的地方，它允许你指定只用于你的操作栏的主题。以下是一个该平台的DarkActionBar主题:

```xml
<style name="Theme.Holo.Light.DarkActionBar">
    <item name="android:actionBarWidgetTheme">@android:style/Theme.Holo</item>
</style>
```

因此，你可以在action bar上使用暗主题。

## 基本功能

如何工作的呢?简单地说，它是API v1的一个可用的[ContextThemeWrapper](https://developer.android.com/reference/android/view/ContextThemeWrapper.html)类。这是一个很简单的类，它的作用是:

它包装在现有的Context (比如Activity)，然后将一个新主题覆盖到该Context的主题之上。这一点很重要，因为这将导致……

## ThemeOverlay

您可能已经在Lollipop SDK中看到了这些主题。有两种主要的覆盖:

- ThemeOverlay.Material.Light
- ThemeOverlay.Material.Dark

那么这些是什么呢?同样，线索在名称中，它们与ContextThemeWrapper的工作方式是直接匹配的。

它们是特殊的主题，覆盖了正常的`Theme.Material`主题，覆盖了相关的属性，使它们变得亮/暗。

## ThemeOverlay + ActionBar

你敏锐的眼睛也会看到ActionBar ThemeOverlay衍生品:

- ThemeOverlay.Material.Light.ActionBar
- ThemeOverlay.Material.Dark.ActionBar

这些只应该通过新的`actionBarTheme`属性与Action Bar一起使用，或者直接设置在您的Toolbar上(见下面)。

目前对他们的父母来说，唯一不同的是，他们将`colorControlNormal`更改为`android:textColorPrimary`，从而使任何文本和图标都不透明。

## android:theme

让我们回到新的Lollipop功能。正如前面提到的，现在可以直接在布局中指定一个主题。最常见的使用(可能)是使用Toolbar:

```xml
<Toolbar  
    android:layout_height="?android:attr/actionBarSize"
    android:layout_width="match_parent"
    android:background="?android:attr/colorPrimaryDark"
    android:theme="@android:style/ThemeOverlay.Material.Dark.ActionBar" />
```

希望你们现在能看到它们是如何结合在一起的。我们现在已经让Toolbar有了一个黑色的主题，确保它的内容是浅色的，并且与黑暗背景形成对比。

有一点需要注意的是，在Lollipop上的`android:theme`有效于所有在布局中声明的孩子们:

```xml
<LinearLayout
    android:theme="@android:style/ThemeOverlay.Material.Dark">

    <!-- Anything here will also have a dark theme -->

</LinearLayout>
```

如果需要，你的孩子可以设定自己的主题。

## 例子

让我们来总结一下我最近被问到的一个问题:

> 我如何设置android:colorEdgeEffect，使它只在一个视图上生效?

`colorEdgeEffect`属性是Android 5.0中的一个新主题属性，用于自定义列表的超滚动效果的颜色。

因为这是一个主题属性，您不能直接在视图上设置它。相反，我们需要使用一个定制的theme叠加来使用`android:theme`。我们的自定义覆盖只是将`android:colorEdgeEffect`设为红色。然后我们将这个主题设置为视图，这样它就会生效。

**res/values/themes.xml**

```xml
<style name="RedThemeOverlay" parent="android:ThemeOverlay.Material">
    <item name="android:colorEdgeEffect">#FF0000</item>
</style>
```

**res/layout/fragment_list.xml**

```xml
<ListView
    ...
    android:theme="RedThemeOverlay" />
```

需要注意的是，colorEdgeEffect只是一个例子，这个技术可以与所有的主题属性一起使用。

## Theme 和 Style

那么区别是什么呢?它们都是以完全相同的方式声明的(你已经知道了)，它们的区别在于它们是如何被使用的。

主题是为你的应用程序设计的全局样式，新功能并没有改变，它只是允许你根据每个视图进行调整。

样式是在一个视图级别上应用的。在内部，当您在视图上设置样式时，LayoutInflater将读取该样式并在任何显式属性之前将其应用到AttributeSet(这允许您在视图上覆盖样式值)。

属性集中的值可以引用视图的主题中的值。

> 主题是全局的，风格是局部的。

## AppCompat

那么AppCompat是如何适应这一情况的呢?很明显，它会返回一些新的颜色主题属性。

它还支持某些widgets的`android:theme`功能，目前只有`android.support.v7.widget.Toolbar`。
