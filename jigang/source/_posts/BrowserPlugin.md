---
title: SampleBrowserPlugin介绍
date: 2017-12-04 22:56:21
tags:
- Android
- Java
- 'Browser Plugin'
categories:
- 'Android Browser Plugin'

---

![](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=018b3271a76eddc432eabca958b2dd98/730e0cf3d7ca7bcb0539dbbbb4096b63f724a8e7.jpg)

对于Android浏览器插件的开发可以参考源码的实例,development/samples/browseplugin实例。本文对此实例进行介绍。

<!-- more -->

## 介绍

这个示例插件是为了给插件开发者一个参考点，看看android浏览器插件是如何创建的，以及如何使用可用的APIs。
一个插件就像标准的apk一样被打包，可以通过市场或adb安装。
示例插件尝试尽可能多地使用APIs，但不幸的是，并不是所有的APIs都被覆盖。

试图让一个插件演示一个屏幕上所有可能的API交互是不现实的。
另一方面，我们也不希望每一个功能都有一个单独的插件，因为那会导致许多独立的apk需要维护。
为了解决这个问题，我们开发了使用“子插件”的想法。
在一个插件中，插件只有一个特定的功能，但是它们都可以共享尽可能多的通用代码。
每个子插件及其功能的详细描述可以在子插件部分中找到。

## 插件结构

样例插件被打包为一个插件，但是包含许多独特的子插件(如音频和paint)。

这个包由两部分组成:

1. 包含配置的Java代码;
2. C++共享库，包含浏览器/插件绑定和子插件类。

### JAVA

- `Android.mk`: 指定APK(SampleBrowserPlugin)的名称，以及包含哪些共享库。

- `AndroidManifest.xml`: 类似于标准的android清单文件，除了它必须包含特定于插件的`"uses-permission"`和`"service"` 元素。
`"service"元素`包含描述service的java组件的子元素。

- `src/*`: java源文件的位置。
这包含了`SamplePlugin`。
类是我们的插件的java组件。
组件必须存在并实现所需的接口，尽管简单地返回null是有效的。

- `res/*`: 静态资源的位置(例如插件的图标)

### C++

- `jni/Android.mk`: 指定与apk一起包含的共享库的构建参数。
该库包含插件和浏览器之间的所有绑定。

- `jni/main.*`: 这段代码是插件和浏览器之间的绑定点。
它支持标准的netscape风格插件以及所有android特定APIs所需的所有功能。
插件的起始点是`NP_Initialize`函数。
`NPP_New`函数负责读取输入的args，并选择适当的子插件实例化。
大多数其他函数要么返回固定值，要么将它们的输入传递给子插件进行处理。

- `jni/PluginObject.*`: `pluginObject`提供了一个方便的容器来存储变量(插件的状态)。
这个对象的两个主要职责是
  1. 构造和存储NPClass对象(使用苹果提供的代码完成)
  2. 为子插件对象提供抽象类，并在实例化后存储子插件。


- `jni/*/*`: 每个子插件都有一个包含其类定义和逻辑的文件夹。
子插件从浏览器接收事件，它也可以使用netscape插件功能和专用的android界面来与浏览器进行通信。

## 如何部署

只需简单地在设备/模拟器上编译和安装插件即…

1. 运行`"make SampleBrowserPlugin"`(编译`libsampleplugin.so`，构建apk)
2. 前面的命令生成一个apk文件，以便记录它的位置
3. 运行`"adb install [apk_file]"`，将其安装在设备/模拟器上
4. 浏览器会自动识别插件的可用性

现在插件已经安装好了，你可以像通过`设置 -> 应用 -> 管理应用程序`一样管理它。
要执行插件，您需要在一个文档中包含一个html代码片段，该文档可由浏览器访问。
`mime-type`不能改变，但您可以更改宽度、高度和参数。
这些参数用于通知插件要执行的插件，以及使用哪个绘图模型。

```javascript
<object type="application/x-testbrowserplugin" height=50 width=250>
    <param name="DrawingModel" value="Surface" />
    <param name="PluginType" value="Background" />
</object>
```

## 子插件

每个子插件对应一个插件类型，可以支持一个或多个绘图模型。
在下面的小节中，有关于每个子插件的描述以及创建html代码片段所需的信息。

### 动画

- 插件类型: 动画
- 绘制模型: 位图

这个插件在屏幕上画了一个小球。
如果这个插件不是完全在屏幕上，并且它被触摸了，那么它就会试图在屏幕上显示它自己。

### 音频

- 插件类型:音频
- 绘制模型:位图

这个插件播放一个位于`/sdcard/sdample.raw`的原始音频文件(需要提供你自己的)。
它使用touch来触发播放、暂停和停止按钮。

### 后台

- 插件类型:后台
- 绘制模型:Surface

这个插件的可视组件很少，但主要是在后台运行API测试。
这个插件处理缩放它自己的位图，在这个例子中是一个简单的文本字符串。
这个插件的输出在日志中被发现，因为如果它检测到任何API失败，它会打印错误。
测试的一些API是定时器、javascript访问和位图格式。

### 表单

- 插件类型:表单
- 绘制模型:位图

这个插件模仿了一个简单的用户名/密码表单。
您可以通过触摸或使用导航键来选择文本框。
一旦选中，这个框就会突出显示，键盘就会出现。
如果选择的文本框不是完全的视图，那么插件将确保它以屏幕为中心。

### PAINT

- 插件类型: Paint
- 绘制模型: Surface

这个插件提供了一个用户可以“Paint”的界面。
输入方法可以在鼠标(点)和触摸(线)之间进行切换。
这个插件有一个固定的表面，可以让浏览器在缩放时缩放。

---

相关知识可以参考https://developer.mozilla.org/en/Gecko_Plugin_API_Reference
