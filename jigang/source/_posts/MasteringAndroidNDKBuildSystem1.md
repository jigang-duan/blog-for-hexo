---
title: 掌握Android NDK构建系统——第1部分:使用ndk-build技巧
date: 2017-11-01 18:31:23
tags:
- Android
- NDK
- 翻译
categories:
- Android NDK

---

[翻译]掌握Android NDK构建系统——第1部分:使用ndk-build技巧
===

[原英文文章](http://web.guohuiwang.com/technical-notes/androidndk1)

![ndk](https://static.pexels.com/photos/595804/pexels-photo-595804.jpeg)

这篇文章不是一个“Hello world!”类型的NDK”教程。尽管我仍将简要介绍ndk构建的基本知识，但这并不是本文的重点。相反，我将总结一些非常有用的NDK技巧和我在项目中使用的技巧。希望这些建议对于那些想要建立一些实际项目的人来说是非常有用的，而不是一个学习NDK的玩具项目。因此，目标读者是中或高级Android开发人员。这篇文章包含两个部分:

* **[第1部分](#):ndk-build** <br />在这一部分中，我们将讨论如何灵活地使用`ndk-build`来构建您的项目，以及如何组织您的项目的文件结构。
* **[第2部分](/2017/11/01/MasteringAndroidNDKBuildSystem1/):独立的工具链** <br />在第2部分中，将讨论独立工具链的设置和使用。

所有例子的源代码可以在这里找到:[https://github.com/robertwgh/mastering-ndk](https://github.com/robertwgh/mastering-ndk)。

<!-- more -->

## 目录 ##

1. [介绍](#introduction)
1. [先决条件](#prerequisites)
1. [ndk-build的基本知识](#ndkbuild_basics)
1. [使用NDK构建本地可执行文件](#building_exe_ndk)
1. [有用的技术](#useful)
	1. [如何在`jni`目录中编译源代码](#compile_jni)
	1. [去掉`jni`文件夹](#rid_jni)
	1. [为makefile使用自定义名称](#name_makefiles)
	1. [使用`include`嵌入`.mk`文件](#include_mk)
	1. [关于`LOCAL_PATH`和`CLEAR_VARS`](#local_path_clear_vars)
	1. [如何调试`.mk` makefile](#debug_mk)
	1. [针对多个目标体系结构的构建](#multiple_target)
1. [总结](#summary)


## <a name="introduction"></a>介绍

Android NDK(本地开发工具包)是Android应用程序开发人员的一种强大工具，他们想要高效和高性能的本地代码，或者需要处理底层硬件细节(如OpenGL、OpenCL等)。

Android NDK官方文档([在线版本](http://www.kandroid.org/ndk/docs/OVERVIEW.html))是OK的，如果你已经使用NDK工作过一段时间了。不过，它并不是专门为刚开始使用Android NDK开发的人设计的。官方文件的问题是没有重点，因此重要的信息很容易被忽视。

还有许多在线教程和文章展示了NDK的基础和NDK构建工具的使用。然而，这些信息到处都是。没有一个地方可以深入地讨论这些主题和技术。希望本文能涵盖其中的一些内容。

## <a name="prerequisites"></a>先决条件

在本文中，我提出以下假设:

* 你知道NDK是什么;
* 你了解C/C++;
* 你已经在电脑上安装了Android NDK。在我的设置中，我安装在`D:\development\android-ndk-r10d`下面。在本文的后面部分，我将调用这条路径`NDK_ROOT`。
* 为了避免使用`ndk-build`时的长路径名，我将`NDK_ROOT`添加到系统`PATH`环境变量中。

## <a name="ndkbuild_basics"></a>ndk-build的基本知识

当然，使用Android NDK的第一步是从Android开发者网络下载NDK安装包。在安装NDK包之后，这些是您得到的:

* `NDK_ROOT\ndk-build.cmd`脚本;
* `NDK_ROOT\docs` 文档;
* 工具链和编译器;
* 一些本地库的源代码;
* 一些示例代码。

如果您想了解基本的设置和NDK的makefile的语法，那么示例代码可能非常有用。通过NDK提供的示例代码，您将发现大多数代码示例，如果您正在开发一个Android应用程序项目，并将使用NDK来构建Android应用程序的JNI部分。这就是为什么您注意到所有的项目都在jni文件夹下放置c/c++源代码和makefile。

以下是NDK示例的典型文件结构:

```project structure
+-- project_root
|   +-- jni
|       +-- Android.mk
|       +-- Application.mk
|       +-- main.c
|   +-- obj
|   +-- libs
```

如您所见，`jni`目录是整个NDK项目的核心，它包含c/c++源代码，两个makefile Android.mk和Application.mk。稍后将讨论，您将看到c/c++源代码并不需要放在jni文件夹中。此外，您不需要为makefile提供完全相同的名称。但作为一个起点，使用ndroid.mk和Application.mk将是最简单的方法，可以为你节省大量的精力，除非你真的不喜欢makefile的当前名称。默认情况下，`ndk-build`将尝试定位

另外两个文件夹obj和libs是由NDK构建系统生成的，它们分别包含中间文件和最终的二进制代码。

**Android.mk**和**Application.mk**是NDK项目最重要的makefile文件。

* __Android.mk__更像是一个传统的makefile，定义源代码、包含头文件的路径、链接器的路径来定位库、模块名、构建类型等等。
* __Application.mk__定义了Android应用程序相关的属性，如Android SDK版本、调试或发布模式、目标平台ABI(架构二进制接口)、标准c/c++库等。

一个典型的**Android.mk**

```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE    := <module_name>  # 命名你的模块.
LOCAL_SRC_FILES := main.c
include $(BUILD_SHARED_LIBRARY)
```

最小**Application.mk**

```makefile
APP_ABI := all
```

为了构建这样的项目，我们可以访问`project_root`，并输入`ndk-build`(假设您在系统`PATH`中有`NDK_ROOT`)，NDK构建脚本将自动找到`jni`文件夹下的本地代码。

```bash
$ cd project_root
$ ndk-build
```

如果代码中没有bug，那么编译后的共享库`lib<module_name>.so`将会在`libs/<abi>/` 目录下生成。一旦你得到了这个共享库文件，你的Android应用程序构建系统将把它打包到最终的APK安装文件中。您将能够在JAVA代码中使用JNI调用本地函数。

在本文中，我们将更多地关注如何使用NDK构建可执行的二进制文件，因为我们将更容易以这种方式测试我们的结果。但是请记住，这里所讨论的所有技术都是相同的，并且可以直接应用到共享库项目中，而不需要任何更改。

## <a name="building_exe_ndk"></a>使用NDK构建本地可执行文件

让我们先建立一个“Hello World！””测试。项目结构将是这样的:

```project structure
+-- ex1_helloworld
|   +-- jni
|       +-- Android.mk
|       +-- Application.mk
|       +-- hello.c
```

**_Hello.cpp_**

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello World!" << std::endl;
    return 0;
}
```

**_Android.mk_**

请注意，在`.mk`文件注释以`“#”`开头。

```makefile
LOCAL_PATH:= $(call my-dir) # 获取项目的本地路径.
include $(CLEAR_VARS) # 用前缀"LOCAL_"清除所有变量

LOCAL_SRC_FILES:=hello.cpp # 包含源代码.
LOCAL_MODULE:= hello # 二进制的名称.
include $(BUILD_EXECUTABLE) # 告诉ndk-build，我们想要构建一个本地可执行文件.
```

**_ Application.mk_**

```makefile
APP_OPTIM := debug    # 在调试模式下构建目标. 
APP_ABI := armeabi-v7a # 将目标架构定义为ARM.
APP_STL := stlport_static # 我们使用stlport作为标准的c/c++库.
APP_CPPFLAGS := -frtti -fexceptions    # 这是你启用异常的地方.
APP_PLATFORM := android-19 # 定义本地应用程序的Android版本.
```

您可能已经发现，共享库项目和本地项目之间的主要区别仅仅是**Android.mk**中的一行。

对于一个共享的库，我们使用:

```
include $(BUILD_SHARED_LIBRARY)
```

对于可执行的二进制文件，我们使用:
```
include $(BUILD_EXECUTABLE)
```

在这一点上，我们可以构建我们的“Hello World！”NDK项目:

```sh
$ cd project_root
$ ndk-build
[armeabi-v7a] Cygwin         : Generating dependency file converter script
[armeabi-v7a] Compile++ thumb: hello <= hello.cpp
[armeabi-v7a] Executable     : hello
[armeabi-v7a] Install        : hello => libs/armeabi-v7a/hello
```

然后，我们可以将`hello`本机程序推到Android设备上运行它(为了实现这一点，我们需要Android ADB工具。请安装Android SDK，并将`ANDROID_SDK_ROOT/platform-tools`设置为系统**PATH**。当然，如果您不想费心安装Android SDK，也可以从网上找到ADB安装包。)

```sh
$ adb root
$ adb shell "mkdir -p /data/mastering_ndk && chmod 777 /data/mastering_ndk"
$ adb push ./libs/armeabi-v7a/hello /data/mastering_ndk
$ adb shell "cd /data/mastering_ndk && chmod 777 ./hello && ./hello"
$ Hello World!
```

> 提示:上面的命令只适用于Root设备。如果你没有root权限设备，你可以使用[Android原生程序启动工具](https://jigang-duan.github.io/2017/11/01/AndroidNDKNativeProgramLauncher/)在任何Android设备上启动本机可执行文件。

我们已经回忆了`ndk-build`的基本知识。在下一节中，我们将展示一些技术，以便更好地利用NDK构建系统来完成一些更大的项目。

## <a name="useful"></a>有用的技术

### <a name="compile_jni"></a>如何在`jni`目录中编译源代码

假设您有一个大型项目，可能是一个跨平台的项目，因此很可能您无法轻松地将所有源代码移动到`jni`文件夹下。实际上需要对现有的`Android.mk` makefile进行了一些小的修改。下面的例子将展示如何实现这一点。您可以在示例2中找到完整的项目:`ex2_src_not_in_jni_folder`。

项目结构:

```project structure
+-- ex2_src_not_in_jni_folder
|   +-- jni
|       +-- Android.mk
|       +-- Application.mk
|   +-- src
|       +-- hello.c
```

**_Android.mk_**

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_SRC_FILES:= ../src/hello.cpp
LOCAL_MODULE:= hello
include $(BUILD_EXECUTABLE)
```

**Application.mk**是相同的。我们可以构建项目并使用与示例1相同的方式来执行二进制文件。

### <a name="rid_jni"></a>去掉`jni`文件夹

`jni`文件夹对于Android应用程序项目中的jni本机项目更有意义。如果我们想要对我们的项目有更有意义的东西，我们可以去掉默认的`ndk-build`所使用的那个特定文件夹。为了实现这一点，需要适当地设置makefile中的几个变量。以下步骤将实现这一目标。完整的示例项目可以在示例3中找到。

项目结构:

```project structure
+-- ex3_get_rid_of_jni_folder
|   +-- Android.mk
|   +-- Application.mk
|   +-- src
|       +-- hello.c
```

如您所见，我们删除了jni文件夹，并将**Android.mk**和**Application.mk**移到`project_root`文件夹中。

新的**Android.mk**

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_SRC_FILES:= src/hello.cpp # 这改变了!
LOCAL_MODULE:= hello
include $(BUILD_EXECUTABLE)
```

新的 **Application.mk**

```makefile
APP_OPTIM := debug
APP_ABI := armeabi-v7a
APP_STL := stlport_static 
APP_CPPFLAGS := -frtti -fexceptions
APP_PLATFORM := android-19
APP_BUILD_SCRIPT := Android.mk  # 这行是新的!
```

请注意，`APP_BUILD_SCRIPT`指示整个应用程序的主要makefile条目。在我们的例子中，它是**Android.mk**。到目前为止，一切看起来都很好，然后，我们使用下面的命令来构建项目，请注意我们在ndk-build命令中添加了`NDK_APPLICATION_MK`变量，告诉ndk-build在哪里可以找到**Application.mk**。在本例中，我们使用以下命令:

```sh
$ ndk-build NDK_APPLICATION_MK=./Application.mk
```

然而，我们将会有以下错误:

```sh
$ ndk-build NDK_APPLICATION_MK=./Application.mk
Android NDK: Could not find application project directory !
Android NDK: Please define the NDK_PROJECT_PATH variable to point to it.
/cygdrive/d/development/android-ndk-r10d/build/core/build-local.mk:148: *** Android NDK: Aborting    .  Stop.
```

NDK_PROJECT_PATH是一个系统环境变量。让我们将它定义到**Application.mk**所在的位置并重新构建。

```sh
$ export NDK_PROJECT_PATH=.
$ ndk-build NDK_APPLICATION_MK=./Application.mk
[armeabi-v7a] Cygwin         : Generating dependency file converter script
[armeabi-v7a] Compile++ thumb: hello <= hello.cpp
[armeabi-v7a] Executable     : hello
[armeabi-v7a] Install        : hello => libs/armeabi-v7a/hello
```

还有另一种方法可以修复上面的构建错误。解决方案是在与**Application.mk**相同的文件夹中创建一个空的**AndroidManifest.xml**文件

在添加这个虚拟的**AndroidManifest.xml**文件之后，我们可以在不定义`NDK_PROJECT_PATH`变量的情况下构建项目。最终的项目结构是:

```project structure
+-- ex3_get_rid_of_jni_folder
|   +-- Android.mk
|   +-- AndroidManifest.xml
|   +-- Application.mk
|   +-- src
|       +-- hello.c
```

### <a name="name_makefiles"></a>为makefile使用自定义名称

我们可以进一步推进前面的技术来定义我们自己的makefile。下面的例子可以在示例4中找到。

在下面的示例中，我们将应用程序makefile重命名为**MyApplication.mk**，并将模块makefile文件重命名为**MyAndroid.mk**。

项目结构:

```project structure
+-- ex4_custom_make_files
|   +-- MyAndroid.mk
|   +-- AndroidManifest.xml
|   +-- MyApplication.mk
|   +-- src
|       +-- hello.c
```

**MyAndroid.mk**和之前的**Android.mk**是一样的。

新**MyApplication.mk**

```makefile
APP_OPTIM := debug
APP_ABI := armeabi-v7a
APP_STL := stlport_static 
APP_CPPFLAGS := -frtti -fexceptions
APP_PLATFORM := android-19
APP_BUILD_SCRIPT := MyAndroid.mk
```

构建命令就变成:

```sh
$ ndk-build NDK_APPLICATION_MK=./MyApplication.mk
[armeabi-v7a] Compile++ thumb: hello <= hello.cpp
[armeabi-v7a] Executable     : hello
[armeabi-v7a] Install        : hello => libs/armeabi-v7a/hello
```

一切都很好，我们成功地得到了二进制的hello。

### <a name="include_mk"></a>使用`include`嵌入`.mk`文件

为了更好地处理包含多个子模块的大型项目，以静态库、共享库或预构建文件的形式，NDK构建系统允许makefile包含另一个makefile。下面是语法:

```makefile
include PATH_TO_MK_FILE/Android.mk
```

这将包括在`PATH_TO_MK_FILE`目录下的**Android.mk**文件到当前makefile。这个“`include`”特性为我们提供了巨大的灵活性，可以创建一些非常有创意的方式来利用建筑系统。

让我们看一个简单的例子(示例5)，看看它是如何工作的。请注意，这个示例包含一个非常简单的makefile。有了“include”的强大功能，我相信您可以创建更加复杂的构建脚本，无论项目多么复杂，它几乎可以为任何项目做任何事情。

在下面的示例项目中，我们在源文件`compute.cpp`中有一个`main()`函数，它调用`add()`和`mul()`函数来执行输入号上的添加和乘法。我们在两个子模块中定义`add()`和`mul()`函数，并将它们编译成两个静态库。最后，当构建可执行文件时，链接器将把所有内容链接在一起，并生成最终可执行的二进制文件。

因此，为了更好地处理子模块，并将每个子模块分开，在这个项目中，我们为每个模块创建了一个**Android.mk**。正如您将很快看到的，通过以这种方式组织makefile，项目现在有了一个非常可伸缩的结构。更具体地说，如果您想要在同一个项目中添加一个子小部件，您只需添加另一个子模块文件夹(不管它是什么，比方说，divide)，然后为新的子模块“divide”创建一个新的**Android.mk**。然后，您只需要在主模块的makefile中更改一行。所有现有的子模块都保持不变。这个项目很容易维护和扩展，以支持更多的功能。

#### 顶层

首先，让我们看一下项目结构，并有一个整体的图景:

```project structure
+-- ex5_using_include_to_embed_make_files
|   +-- makefiles
|       +-- Android.mk
|       +-- Application.mk
|   +-- src
|       +-- main
|           +-- compute.cpp
|           +-- Android.mk
|       +-- submodules
|           +-- add
|               +-- add.cpp
|               +-- Android.mk
|           +-- mul
|               +-- mul.cpp
|               +-- Android.mk   
|           +-- Android.mk          
|   +-- AndroidManifest.xml
```

`makefiles/Application.mk`

```makefile
APP_OPTIM := debug
APP_ABI := armeabi-v7a
APP_STL := stlport_static 
APP_CPPFLAGS := -frtti -fexceptions
APP_PLATFORM := android-19
APP_BUILD_SCRIPT := makefiles/Android.mk
```

`makefiles/Android.mk`

```makefile
TOP_LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

include $(TOP_LOCAL_PATH)/../src/submodules/Android.mk
include $(TOP_LOCAL_PATH)/../src/main/Android.mk
```

在这里，这个**Android.mk**作为顶级makefile，包含另外两个**Android.mk**，一个用于子模块，另一个用于主模块。

#### 主要模块: compute

我们首先看一下主模块。

`src/main/``compute.cpp`

```cpp
#include <iostream>

int add(int a, int b);
int mul(int a, int b);

int main()
{
    int a = 2;
    int b = 3;
    std::cout << "a = " << a << std::endl;
    std::cout << "b = " << b << std::endl;
    std::cout << "add(a, b) = " << add(a, b) << std::endl;
    std::cout << "mul(a, b) = " << mul(a, b) << std::endl;

    return 0;
}
```

`src/main/``Android.mk`

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_SRC_FILES:= compute.cpp
LOCAL_MODULE:= compute
LOCAL_STATIC_LIBRARIES:= add mul
include $(BUILD_EXECUTABLE)
```
我们通过定义`$include $(BUILD_EXECUTABLE)`来构建主模块作为一个可执行文件。我们还定义了`LOCAL_STATIC_LIBRARIES` `add` `mul`，这意味着这个主模块依赖于两个静态库，分别使用模块名`add`和`mul`。但是这两个模块的定义是什么，让我们继续看一下子模块。

#### 子模块：add和mul

`src/submodules/``Android.mk`

```makefile
include $(call all-subdir-makefiles)
```

这里，在`submodules`文件夹中，makefile只包含一行，它在NDK构建系统中调用一个函数。这个命令包括`include $(call all-subdir-makefiles)`，基本上等同于将所有的**Android.mk**文件手动地包含在所有子目录中。在我们的例子中,这将帮助我们包括`src/submodules/add/Android.mk`和`src/submodules/mul/Android.mk`。

让我们看一下子模块`add`。

`src/submodules/add/``add.cpp`

```cpp
int add(int a, int b)
{
    return a + b;
}
```

`src/submodules/add/``Android.mk`

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_SRC_FILES:= ./add.cpp
LOCAL_MODULE:= add
include $(BUILD_STATIC_LIBRARY)
```

我们将模块`add`定义为一个静态库。

类似地，我们还有子模块`mul`，它几乎拥有与`add`模块相同的makefile。

`src/submodules/mul/``mul.cpp`

```cpp
int mul(int a, int b)
{
    return a * b;
}
```

`src/submodules/mul/``Android.mk`

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_SRC_FILES:= ./mul.cpp
LOCAL_MODULE:= mul
include $(BUILD_STATIC_LIBRARY)
```

到目前为止，我们已经列出了项目中的所有文件。构建流程清晰地显示如下:

1. 使用`add.cpp`构建静态库`add`;
1. 使用`mul.cpp`构建静态库`mul`;
1. 使用`compute.cpp`构建主模块`compute`;
1. 将`compute`链接到静态库`libadd.a`和`libmul.a`。生成可执行的`compute`。

#### 构建和执行

我们使用前面示例中的相同命令来构建项目。

```sh
$ ndk-build NDK_APPLICATION_MK=./makefiles/Application.mk
[armeabi-v7a] Cygwin         : Generating dependency file converter script
[armeabi-v7a] Compile++ thumb: compute <= compute.cpp
[armeabi-v7a] Compile++ thumb: add <= add.cpp
[armeabi-v7a] StaticLibrary  : libadd.a
[armeabi-v7a] Compile++ thumb: mul <= mul.cpp
[armeabi-v7a] StaticLibrary  : libmul.a
[armeabi-v7a] Executable     : compute
[armeabi-v7a] Install        : compute => libs/armeabi-v7a/compute
```

我们执行的二进制。

```sh
$ adb shell "mkdir -p /data/mastering_ndk && chmod 777 /data/mastering_ndk"
$ adb push ./libs/armeabi-v7a/compute /data/mastering_ndk
$ adb shell "cd /data/mastering_ndk && chmod 777 ./compute && ./compute"
a = 2
b = 3
add(a, b) = 5
mul(a, b) = 6
```

我们可以看到结果正是我们所期望的。显然，通过使用“include”，项目变得更加结构化。所有的模块都是单独构建的，但是有能力在makefile中共享变量。
可以想象，在大型项目中，必须有更多的设置，比如`LOCAL_C_INCLUDES`, `LOCAL_CFLAGS`, `LOCAL_LDFLAGS`, `LOCAL_LDLIBS`等等。
许多子小管在这些设置中可能具有相同的值。在这些情况下，我们可以提取公共部分并将常见的部分放入一个普通makefile中，然后将其包含到每个子模块的makefile中。
这样做可以节省大量的编码工作，并且可以使makefile更容易修改和维护。
在添加新的子模块时，编写新的makefile的工作量将是最小的。

### <a name="local_path_clear_vars"></a>关于`LOCAL_PATH`和`CLEAR_VARS`

以下两个NDK内置函数非常重要:

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
```

第一个(`LOCAL_PATH:= $(call my-dir)`)检索**Android.mk**文件的当前本地路径，以便同一文件中的所有变量都可以根据本地路径生成绝对路径。命令`include $(CLEAR_VARS)`清除了以`LOCAL_`开头的所有NDK内置变量，如`LOCAL_SRC_FILES`, `LOCAL_C_INCLUDES`, `LOCAL_CFLAGS`, `LOCAL_LDFLAGS`, `LOCAL_LDLIBS`等，除了`LOCAL_PATH`。

当你在这个项目中只有一个**Android.mk**时，这个功能就非常好了。但是，如果您使用“include”来将多个makefile放在一起，那么您需要注意上面的两个命令。

原因是`LOCAL_PATH`变量可以被随后的命令调用`LOCAL_PATH:= $(call my-dir)`覆盖。例如，在以下情况下:

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

include subfolder/Android.mk
LOCAL_SRC_FILES:= $(LOCAL_PATH)/test.cpp
```

上述makefile的问题在于，在“include”之后，在subfolder/**Android.mk**中，`LOCAL_PATH`可能会被包含的**Android.mk**修改。然后，当您试图定位**test.cpp**，ndk-build将会失败，因为现在文件的路径是错误的。

如果您对示例5进行了足够的关注，您将看到一个小技巧已经在那里使用了。让我们看一下示例5的顶级makefile。

`ex5_using_include_to_embed_make_files/makefiles/``Android.mk`

```makefile
TOP_LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

include $(TOP_LOCAL_PATH)/../src/submodules/Android.mk
include $(TOP_LOCAL_PATH)/../src/main/Android.mk
```

为了避免使用错误的`LOCAL_PATH`，我定义了`TOP_LOCAL_PATH`，以保证这两种方法包括使用相同的本地路径。

有些人使用了另一个技巧，在include **Android.mk**中，第一件事是将LOCAL_PATH变量备份到一个临时变量，在退出之前，恢复LOCAL_PATH。例如:

```makefile
LOCAL_PATH_BACK_UP := $(LOCAL_PATH)
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
...
LOCAL_PATH:=$(LOCAL_PATH_BACK_UP )
```

类似地，如果您的模块共享一些公共设置，请小心使用`include $(CLEAR_VARS)`，只有当您知道它是安全且必需的时才使用它。

### <a name="debug_mk"></a>如何调试`.mk` makefile

对于一个小项目，在makefile中查找问题可能更容易。
然而，随着您的项目越来越大，尤其是当您包含了多个模块，包括共享库、静态库和可执行程序时，情况变得更加复杂。您可能会遇到诸如“无法找到构建foo的规则”或“不能链接库bar”之类的问题。
有时，这可能只是由于某个角落出现了打印错误，但可能会导致您花上几个小时来查找，因为`ndk-build`提供的信息确实非常有限，而且常常是不够的。

幸运的是，有一些非常有用的工具:

```makefile
$(error error message)
$(warning warning message)
$(info information message)
```

这些命令可以插入到您的**Android.mk**或**Application.mk**。

这三个命令之间有一些差异。

* `$(info)`命令简单地打印出一些信息，比如在c中的printf()。
* `$(warning)`命令不仅打印信息，还插入行号，指示发出警告的位置。
* `$(error)`命令将打印信息，并停止后续的构建过程。

通常，`$(info)`足以显示我们感兴趣的信息，例如，如果我们想要检查当前`LOCAL_PATH`变量，我们可以执行以下操作:

```makefile
$(info LOCAL_PATH=$(LOCAL_PATH))
```

或者，如果我们在androidmk文件中定义一个变量，我们也可以检查它的值:

```makefile
BUILD_MODE:=NATIVE_MODE
DEVICE_NAME:=NEXUS-5
$(info BUILD_MODE is $(BUILD_MODE) for device $(BUILD_MODE))
```

#### 另一种方法

另一种帮助调试makefile的有用方法是使用`ndk-build`的`V=1`选项。例如，在我们的示例5中，如果我们使用`ndk-build V=1`，以下就是我们将看到的:

```sh
$ ndk-build NDK_APPLICATION_MK=./MyApplication.mk V=1
[Robert: Here I have skipped some print information...]
[armeabi-v7a] Executable     : hello
/cygdrive/d/development/android-ndk-r10d/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-g++ -Wl,--gc-sections -Wl,-z,nocopyreloc --sysroot=D:/development/android-ndk-r10d/platforms/android-19/arch-arm -Wl,-rpath-link=D:/development/android-ndk-r10d/platforms/android-19/arch-arm/usr/lib -Wl,-rpath-link=./obj/local/armeabi-v7a ./obj/local/armeabi-v7a/objs-debug/hello/src/hello.o D:/development/android-ndk-r10d/sources/cxx-stl/stlport/libs/armeabi-v7a/thumb/libstlport_static.a -lgcc -no-canonical-prefixes -march=armv7-a -Wl,--fix-cortex-a8  -Wl,--no-undefined -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now -fPIE -pie   -lc -lm -o ./obj/local/armeabi-v7a/hello
[armeabi-v7a] Install        : hello => libs/armeabi-v7a/hello
install -p ./obj/local/armeabi-v7a/hello ./libs/armeabi-v7a/hello
/cygdrive/d/development/android-ndk-r10d/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-strip --strip-unneeded ./libs/armeabi-v7a/hello
```

最有用的部分是直接起动`/cygdrive/d/development/android-ndk-r10d/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-g++`。
这基本上显示了用于构建目标输出的命令行。
您可以在这里看到所有的编译和链接信息。
对于一个复杂的项目，尤其是带有嵌入式makefile的项目，这将允许我们检查库的路径、头文件的路径和项目依赖关系是否正确设置。

### <a name="multiple_target"></a>针对多个目标体系结构的构建

最后，我们将讨论为多个目标ABIs(AABI=Architecture Binary Interface)构建二进制文件的方法。
当您想要为多个平台发布您的本地程序时，这将非常有用，例如arm v5、arm v7、x86等等。
在为Android应用程序构建原生JNI共享库时，这将非常有用，因为在这种情况下，您需要认真考虑如何发布和发布最终的应用程序。
您构建本地库并发布它们的方法将影响您的应用程序的兼容性。

对于一个没有连接external/3rd-party共享库的典型项目，可以通过在**Application.mk**中设置`APP_ABI`变量来轻松实现这一点。
这已经在官方NDK文档中详细讨论了:[Android Native CPU ABI Management](http://www.kandroid.org/ndk/docs/CPU-ARCH-ABIS.html)。基本上，我们可以在**Application.mk**的一行中选择目标ABI:

```makefile
APP_ABI := armeabi armeabi-v7a mips x86
```

您可以简单地设置`APP_ABI := all`来为上面的目标ABIs构建本地代码。

这种方法非常适合那些不依赖于任何外部库的项目。
通常情况下，我们需要将一些现有的libs与我们的项目联系起来。
这意味着有可能以二进制格式提供外部的libs。
这将使问题变得棘手。
在这些情况下，我们需要检测当前构建的当前目标`APP_ABI`，然后根据该架构执行正确的操作。

我将用一个非常实际的问题作为例子来说明为什么这是不简单的。
我遇到的问题是在开发Android应用程序时，使用了OpenCL加速的本地代码。(对于那些不知道OpenCL的人来说:OpenCL是由Khronos组织维护的异构计算的开放规范;大多数现代桌面CPU，GPU和最新一代的移动GPU支持OpenCL，因此，你可以利用OpenCL来利用GPU并行架构的强大功能来加速你的算法。)
OpenCL的工作方式是SoC芯片供应商实现OpenCL软件堆栈，包括驱动程序和共享库中的编译器。
这里的问题是，对于不同厂商提供的应用SoC芯片组的不同设备，OpenCL支持是由驱动程序库支持的，它通常是一个共享的库，驻留在目录中，如`/system/vendor/lib`, 或`/system/lib`，或其他目录。
因此，对于在移动设备(智能手机或平板电脑)中使用的不同芯片组，共享库将会非常不同。
构建OepnCL程序的一个问题是，您需要动态链接到这些OpenCL驱动程序库，这样，您的应用程序调用的OpenCL API函数可以在链接时解决。

下表显示了一些主要的移动SoC芯片厂商的移动GPU的OpenCL库。

| SoC芯片组 | GPU | Arch | OpenCL库 |
|:------------- |:---------------:| :-------------:| :-------------|
| 三星Exynos(5420 或 5433) | ARM Mali T628 或 T760 | ARM | `/system/vendor/lib/egl/libGLES_mali.so` |
| 高通骁龙800,801,805,810 | Adreno A330, A420, A430 | ARM | `/system/vendor/lib/libOpenCL.so` |
| 英特尔原子Z3560 | PowerVR G6430 |  x86 | `/system/vendor/lib/libPVROCL.so` |

我们可以很容易地看到，共享库位于Android系统的不同位置，它们的名称不同，甚至具有不同的架构。
如果您想要构建一个应用程序来支持以上所有的平台，那么您可以使用正确的动态链接来构建本地部分。

我们可以使用`TARGET_ARCH`变量或`TARGET_ARCH_ABI`变量来实现这一目标，这些变量表示当前的目标体系结构或目标体系结构ABIs。

> `TARGET_ARCH`和`TARGET_ARCH_ABI`略有不同:`TARGET_ARCH`报告体系结构名称;`TARGET_ARCH_ABI`报告体系结构和指令集版本。
> 例如，如果当前的`APP_ABI`是“armeabi-v7a”，`TARGET_ARCH`只显示“arm”，而`TARGET_ARCH_ABI`将是“armeabi-v7a”。请记住这一点，这样您就可以充分利用这两个变量。

下面是一个示例，其中我们构建一个本地程序(或JNI使用的共享库)，目标是支持多个体系结构:

```makefile
ifeq ($(TARGET_ARCH),x86)
    GPU_FAMILY=powervr
    OPENCL_LIB := PVROCL
    OPENCL_INC_DIR    := $(OPENCL_COMMON)/include/CL12/
    OPENCL_LIB_DIR    := $(OPENCL_COMMON)/libs/$(GPU_FAMILY)_$(TARGET_ARCH)/
    LOCAL_LDLIBS    := -llog  -L$(OPENCL_LIB_DIR) -l$(OPENCL_LIB) 
    MYMODULE_NAME:=mymodule_$(GPU_FAMILY)
    include OTHER_MK_FILES.mk
endif

ifeq ($(TARGET_ARCH),arm)
    #Adreno
    GPU_FAMILY:=adreno
    OPENCL_LIB := OpenCL
    OPENCL_INC_DIR    := $(OPENCL_COMMON)/include/CL12/
    OPENCL_LIB_DIR    := $(OPENCL_COMMON)/libs/$(GPU_FAMILY)_$(TARGET_ARCH)/
    LOCAL_LDLIBS    := -llog  -L$(OPENCL_LIB_DIR) -l$(OPENCL_LIB) 
    MYMODULE_NAME:=mymodule_$(GPU_FAMILY)
    include OTHER_MK_FILES.mk

    #Mali
    GPU_FAMILY:=mali
    OPENCL_LIB := GLES_mali
    OPENCL_INC_DIR    := $(OPENCL_COMMON)/include/CL11/
    OPENCL_LIB_DIR    := $(OPENCL_COMMON)/libs/$(GPU_FAMILY)_$(TARGET_ARCH)/
    LOCAL_LDLIBS    := -llog  -L$(OPENCL_LIB_DIR) -l$(OPENCL_LIB) 
    MYMODULE_NAME:=mymodule_$(GPU_FAMILY)
    include OTHER_MK_FILES.mk
endif
```

这是完整makefile的一部分，它只显示了使用`TARGET_ARCH`变量的部分。在本节之后，我们将包含一些makefile来执行常规的构建过程。请注意，`$(OPENCL_COMMON)`文件夹包含用于不同目标架构的共享库(我们可以从实际设备中`adb pull`它们)。

通过这样做，我们基本上实现了本地代码的几个不同版本。假定本地程序的模块名是“mymodule”。然后，最终生成的二进制文件将具有以下结构。

```project structure
+-- libs
|   +-- armeabi-v7a
|       +-- libmymodule_adreno.so
|       +-- libmymodule_mali.so
|   +-- x86
|       +-- libmymodule_powervr.so
```

这些库将被打包到Android APK安装程序中。在安装过程中，正确的架构版本将基于给定设备的主ABI属性(请参阅[这里](http://www.kandroid.org/ndk/docs/CPU-ARCH-ABIS.html)的详细信息)安装。

在我们的示例中，如果我们将APK安装在带有ARM CPU的设备上，那么前两个.so，文件将被安装。
在您的程序中，您可以很容易地检测到设备芯片供应商，并加载相应的共享库。
如果APK安装在x86设备上，那么唯一的x86库将被安装，并在应用程序执行时自动加载。

> 当然，您需要在JAVA代码中调用`System.loadLibrary()`函数，或者使用`dlopen()`和`dlsym()`函数在本机代码中动态地链接共享库，但是一旦我们获得了以上的库，加载正确库的工作量就变得微不足道了。
  
## <a name="summary"></a>总结

在本文中，我们介绍了`ndk-build`的基础知识，接着讨论了`ndk-build`系统中的一些半隐藏特性或技术，我个人认为这非常有用。
我希望这篇文章对Android NDK开发者有用。

尽管每一种技术都可能简单而直接，但是当您将它们组合在一起时，我确信您将能够创建一些不仅强大而且高效的构建脚本。请下载github上的示例，并尝试一下，你需要的只是Android NDK和Android设备(或者像[Genymotion](https://www.genymotion.com/)这样的虚拟设备)。
