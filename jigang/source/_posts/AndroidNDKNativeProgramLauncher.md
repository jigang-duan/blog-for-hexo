---
title: Android原生程序启动工具
date: 2017-11-01 16:38:27
tags:
- Android
- NDK
- 翻译
categories:
- Android NDK

---

[翻译]Android原生程序启动工具
====

![android.ndk](https://static.pexels.com/photos/595804/pexels-photo-595804.jpeg)

* [介绍](#introduction)
* [主要功能](#major_features)
* [入门指南](#getting_started)
	* [下载地址](#download_address)
	* [提供了什么?](#what_is_provided)
	* [使用说明](#instructions)
	* [一些细节](#some_details)
* [示例](#examples)
	* [示例1](#example1)
	* [示例2](#example2)

<!-- more -->

## <a name="introduction"></a>介绍 ##

在使用原生代码开发Android应用程序时，我们通常需要使用纯原生模式来调试它(使用NDK编译代码，生成可执行文件，并通过`adb shell`执行它)。有时，我们甚至不想花时间开发Android应用，因为我们只是想测试一些原生函数。

以处理OpenCL和GPU为例。编写的程序中超过**80%**将是性能和基准代码。由于它们直接操作系统驱动程序或硬件，所以通过ADB以纯原生模式运行它们非常自然。我享受着原生程序带给我的效率和速度。

但是，纯原生代码可能无法在Android设备上运行。比如以下遇到的一些情况:

> 1. 设备没有被ROOT。因此，由于缺少权限，您不能将原生程序推到/data/文件夹下。您不能通过使用`chmod+x`文件来给原生可执行文件提供执行权限，因此，在设备上运行它是不可能的。
> 2. 你的设备已经ROOT了。您可以通过ADB来运行代码。然而，当使用ADB运行代码时，您必须使用USB电缆将您的设备连接到计算机。如果你用一些外部功率计来测量系统功耗呢?由于USB电缆为设备提供了额外的电源输入，所以连接到USB电缆使功率测量变得不可能。
> 3. 对于某些设备，您甚至没有正确的ADB驱动程序。
> 4. 一些未知的情况下…

在所有这些情况下，我们需要特定的工具来帮助我们在`没有ROOT`、`独立的`、`没有usb连接`的设备上运行本机程序。

## <a name="major_features"></a>主要功能 ##

* 不需要root权限。
* 你可以在没有ADB和USB连接到电脑的情况下运行原生代码。
* 支持从sdcard中加载本机程序。
* 支持命令行参数。
* 允许用户设置`LD_LIBRARY_PATH`。
* 支持子目录。原生程序装入的输入文件和配置可以放在子目录中，在原生程序中，使用相对路径来访问它们。
* 输出文件的支持。可以从工作目录中检索输出文件。

## <a name="getting_started"></a>入门指南 ##

### <a name="download_address"></a>下载地址

[AndroidNativeLauncher@github.](https://github.com/robertwgh/AndroidNativeLauncher)

![NativeLauncher.png](https://raw.githubusercontent.com/robertwgh/AndroidNativeLauncher/master/screenshot/test1.png?raw=true)

### <a name="what_is_provided"></a>提供了什么?

* 一个APK安装程序。
* 一些示例NDK项目，以及构建脚本。

### <a name="instructions"></a>使用说明

* 将您的原生程序(由ndk-build编译)复制到您的sdcard上的一个文件夹中，例如 `/sdcard/test/`。
* 将所有依赖的共享库(.so)复制到该文件夹。
* 将输入文件复制到同一个文件夹。
* 按下“载入原生程序”按钮。
* 如果有的话，设置命令行参数。
* 按下“运行”按钮来执行程序。
* 检查你文件夹下的结果，例如`/sdcard/test/`。

### <a name="some_details"></a>一些细节

1. **问**:工作目录在哪里? <br />**答**:工作目录是可执行文件的目录。
2. **问**:如何设置`LD_LIBRARY_PATH`?库的装载顺序是什么? <br /> **答**:在textedit字段中，键入您的自定义库路径。请注意，您的工作目录将自动添加到`LD_LIBRARY_PATH`。例如，如果您在字段中输入“`mylib`”，那么`LD_LIBRARY_PATH`将被设置为`LD_LIBRARY_PATH=mypath:workingdirectory:$LD_LIBRARY_PATH`。
3. **问**:如何使用子目录? <br /> **答**: 子文件夹可以通过相对路径访问。例如，如果我们在工作目录中有一个名为“`image`”的文件夹，其中包含一个输入图片“`lena.png`”。在你的代码中，你可以通过`imag/lena.png`来访问它。您的输出文件也将在工作目录下。

## <a name="examples"></a>示例 ##

提供了两个示例来演示原生程序启动程序的功能。您可以在`bin`目录中找到预编译的二进制文件。

如果需要，还可以使用NDK编译代码。我提供一个`build_examples.sh`脚本简化这个过程，请安装NDK并确保在系统路径中进行NDK构建，然后您就可以使用构建下面的示例:

``` bash
$ cd examples
$ ./build_examples.sh
```

编译完代码后，让我们将二进制文件推入到sdcard中

``` bash
$ adb shell "mkdir -p /sdcard/examples"
$ adb push bin /sdcard/example
```

然后，您可以使用NativeLauncher应用程序来加载本地程序并执行它。

--

让我们来看看两个例子的细节。

### <a name="example1"></a>示例1

**_test1.cpp_**

```cpp
#include <iostream>
#include <fstream>

int main(int argc, char ** argv)
{
    if(argc > 1)
    {
        std::cout << "argc: " << argc << std::endl;
        for(int i = 0; i < argc; i ++)
        {
            std::cout << "argv[" << i << "]: " << argv[i] << std::endl;
        }
        std::cout << std::endl;
    }

    std::cout << "cout: Hello world!" << std::endl;
    std::cerr << "cerr: Hello world!" << std::endl;


    std::cout << "\nTest:\n" << std::endl;
    for(int i = 0; i < 20; i ++)
    {
        for(int j = 0; j < i; j ++)
            std::cout << "*";

        std::cout << std::endl;
    }

    std::ofstream ofs("results.txt");
    if(ofs.is_open()){
        ofs << "This is just a test." << std::endl;

        ofs.close();

        std::cout << "results.txt is generated." << std::endl;
    }

    return 0;
}
```

正如您所看到的，这个演示基本上打印出命令行参数、一些测试消息、一个使用的简单图形，最后保存一个`results.txt`文件到sdcard。

我们使用以下makefile编译test1.cpp:

**_Android.mk_**

```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := test1
LOCAL_LDLIBS   = -llog
LOCAL_CFLAGS    += -fPIE
LOCAL_LDFLAGS     += -fPIE -pie 
LOCAL_SRC_FILES := test1.cpp
include $(BUILD_EXECUTABLE)
```

### <a name="example2"></a>示例2

这个例子展示了对共享库和`LD_LIBRARY_PATH`的支持。我们定义一个`add()`函数并将其编译成一个共享库。然后在`main()`函数中，我们从共享库调用`add()`函数来执行计算。

**_addfunc.cpp_**

```cpp
int add(int a, int b)
{
    return a + b;
}
```

**_test2.cpp_**

```cpp
#include <iostream>

extern int add(int, int);
int main(int argc, char ** argv)
{
    std::cout << "Test shared library." << std::endl;
    int a = 1;
    int b = 2;
    std::cout << "a=" << a << std::endl
        <<"b=" << b << std::endl
        << "add(a, b)=" << add(a, b) << std::endl;

    return 0;
}
```

**_Android.mk_**

```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := addfunc
LOCAL_SRC_FILES := addfunc.cpp
include $(BUILD_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE := test2
LOCAL_SRC_FILES := test2.cpp
LOCAL_CFLAGS    := -fPIE
LOCAL_LDFLAGS     := -fPIE -pie 
LOCAL_LDLIBS    := -L$(LOCAL_PATH)/../libs/$(TARGET_ARCH_ABI)/ -laddfunc 
include $(BUILD_EXECUTABLE)
```

因为`libaddfunc.so`和`test2`在同一个文件夹中，当您加载test2可执行文件时，共享库`libaddfunc.so`自动加载。因此，当您运行程序时，您将看到正确的输出:

```print
Test shared library.
a=1
b=2
add(a,b)=3
```

但是，如果我们移动`libaddfunc.so`到一个名为lib的子目录：

```bash
$ adb shell
$ cd sdcard/example/example2
$ mkdir lib
$ mv libaddfunc.so lib/
```

然后，如果我们再次执行本机程序，因为这一次共享库不在搜索路径中，我们将看到以下消息:

```print
CANNOT LINK EXECUTABLE: could not load library "libaddfunc.so" needed by ".test2"; caused by library "libaddfunc.so" not found.
```

为了解决这个问题，我们可以在`LD_LIBRARY_PATH`文本字段中输入`lib`。也就是说，当前的图书馆搜索路径变成了:

```
LD_LIBRARY_PATH=lib:working_directory:$LD_LIBRARY_PATH
```

在这里，工作目录是您的本机程序所在的文件夹。

通过这样做，我们可以再次按下“Run”按钮运行原生程序，我们应该能够看到正确的输出消息。这意味着已经找到并加载了共享库。