---
title: 掌握Android NDK构建系统——第2部分:独立的工具链
date: 2017-11-02 11:31:23
tags:
- Android
- NDK
- 翻译
categories:
- Android NDK

---

[翻译]掌握Android NDK构建系统——第2部分:独立的工具链
===

[原英文文章](http://web.guohuiwang.com/technical-notes/androidndk2)

![ndk](https://static.pexels.com/photos/595804/pexels-photo-595804.jpeg)

这是“掌握NDK”的第2部分。在前面的部分([第1部分](/2017/11/01/MasteringAndroidNDKBuildSystem1/))中，我们介绍了如何使用`ndk-build`构建Android本地项目，我们还介绍了一些高级技术来管理和定制大型项目的构建脚本。

尽管对于大多数Android本地项目来说，`ndk-build`可能已经足够了，但是在某些情况下仍然可能需要独立的工具链。
例如，如果您已经有一个c/c++项目，它可能非常复杂，并且有一个复杂的makefile。
在这种情况下，你可能不想把所有东西都转换成`Android.mk`和`Application.mk`。
在这种情况下，使用独立的工具链更有意义，它允许您保留最初的makefile，或者重用大多数原始makefile。
因此，在本部分中，我将简要介绍独立工具链的用法，并给出一些代码示例。

所有例子的源代码可以在这里找到:[https://github.com/robertwgh/mastering-ndk](https://github.com/robertwgh/mastering-ndk)。

<!-- more -->

## 目录 ##

1. [在我们开始之前](#before_start)
1. [ndk-build工作流](#workflow)
1. [为您的项目使用定制的工具链](#customized)
1. [总结](#summary)

## <a name="before_start"></a>在我们开始之前

Android NDK官方文档包含一章[“标准工具链”](http://www.kandroid.org/ndk/docs/STANDALONE-TOOLCHAIN.html)，它提供了关于独立工具链的有用信息。但是，该文档缺少详细信息，并且没有示例来演示使用情况。因此，可能很难遵循官方文件。但无论如何，这仍然是一个很好的参考。

## <a name="workflow"></a>`ndk-build`工作流

实际上，当我们使用ndk-build时，如果我们启用调试选项V=1，如下:

```sh
ndk-build V=1
```

我们将看到ndk-build实际做了什么。下面的控制台打印来自于示例1的编译。

```sh
$ ndk-build V=1
rm -f ./libs/arm64-v8a/lib*.so ./libs/armeabi/lib*.so ./libs/armeabi-v7a/lib*.so ./libs/armeabi-v7a-hard/lib*.so ./libs/mips/lib*.so ./libs/mips64/lib*.so ./libs/x86/lib*.so ./libs/x86_64/lib*.so

rm -f ./libs/arm64-v8a/gdbserver ./libs/armeabi/gdbserver ./libs/armeabi-v7a/gdbserver ./libs/armeabi-v7a-hard/gdbserver ./libs/mips/gdbserver ./libs/mips64/gdbserver ./libs/x86/gdbserver ./libs/x86_64/gdbserver

rm -f ./libs/arm64-v8a/gdb.setup ./libs/armeabi/gdb.setup ./libs/armeabi-v7a/gdb.setup ./libs/armeabi-v7a-hard/gdb.setup ./libs/mips/gdb.setup ./libs/mips64/gdb.setup ./libs/x86/gdb.setup ./libs/x86_64/gdb.setup

[armeabi-v7a] Compile++ thumb: hello <= hello.cpp
/cygdrive/d/development/android-ndk-r10d/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-g++ -MMD -MP -MF ./obj/local/armeabi-v7a/objs-debug/hello/hello.o.d.org -fpic -ffunction-sections -funwind-tables -fstack-protector -no-canonical-prefixes -march=armv7-a -mfpu=vfpv3-d16 -mfloat-abi=softfp -fno-exceptions -fno-rtti -mthumb -Os -g -DNDEBUG -fomit-frame-pointer -fno-strict-aliasing -finline-limit=64 -O0 -UNDEBUG -marm -fno-omit-frame-pointer -ID:/development/android-ndk-r10d/sources/cxx-stl/stlport/stlport -ID:/development/android-ndk-r10d/sources/cxx-stl//gabi++/include -Ijni -DANDROID  -Wa,--noexecstack -Wformat -Werror=format-security -fPIE  -frtti   -frtti -fexceptions  -ID:/development/android-ndk-r10d/platforms/android-19/arch-arm/usr/include -c  jni/hello.cpp -o ./obj/local/armeabi-v7a/objs-debug/hello/hello.o && ./obj/convert-dependencies.sh ./obj/local/armeabi-v7a/objs-debug/hello/hello.o.d

[armeabi-v7a] Executable     : hello
/cygdrive/d/development/android-ndk-r10d/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-g++ -Wl,--gc-sections -Wl,-z,nocopyreloc --sysroot=D:/development/android-ndk-r10d/platforms/android-19/arch-arm -Wl,-rpath-link=D:/development/android-ndk-r10d/platforms/android-19/arch-arm/usr/lib -Wl,-rpath-link=./obj/local/armeabi-v7a ./obj/local/armeabi-v7a/objs-debug/hello/hello.o D:/development/android-ndk-r10d/sources/cxx-stl/stlport/libs/armeabi-v7a/thumb/libstlport_static.a -lgcc -no-canonical-prefixes -march=armv7-a -Wl,--fix-cortex-a8  -Wl,--no-undefined -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now -fPIE -pie   -lc -lm -o ./obj/local/armeabi-v7a/hello

[armeabi-v7a] Install        : hello => libs/armeabi-v7a/hello
install -p ./obj/local/armeabi-v7a/hello ./libs/armeabi-v7a/hello
/cygdrive/d/development/android-ndk-r10d/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-strip --strip-unneeded ./libs/armeabi-v7a/hello

```

我们可以看到，`ndk-build`实际上做了如下的事情:

1. 在输出文件夹libs中删除以前构建的文件。
1. 从源代码构建hello.o。
1. 从hello.o构建可执行。
1. 根据ABI，将可执行文件复制到子文件夹里的文件夹。

上面的构建信息显示了`ndk-build`如何调用工具链来构建项目。基本上`/cygdrive/d/development/android-ndk-r10d/toolchains/arm-linux-androideabi-4.8/prebuilt/windows-x86_64/bin/arm-linux-androideabi-g++`是一个独立的工具链在NDK安装中。
如果我们读取整个命令，我们会发现该命令只调用了`cross-compilation`版本的g++，并带有诸如include路径、库路径等构建参数。
使用独立的工具链最直接的方法是模仿`ndk-build`的工作方式，并直接根据目标体系结构和平台调用正确的编译器。

在`NDK_ROOT/toolchains`目录下，我们可以找到ARM、X86、x8664、MIPS等不同的工具链。还有不同编译器的版本，比如g++和clang。因此，我们基本上可以选择适合我们项目的任何东西。

## <a name="customized"></a>为您的项目使用定制的工具链

显然，上面的方法是有效的，但是它非常冗长，不适合较大的项目。我们所能做的是为特定的平台和ABI创建一个“customized”工具链，借助NDK安装中提供的$NDK/build/tools/make-standalone-toolchain.sh工具的帮助。

让我们来看一个例子。假设我们有一个具有以下配置的项目:

* 支持android-19平台。
* 主机系统是Windows 7 64位。
* 目标架构是ARM。

我们可以创建一个脚本`generate_standalone_toolchain.sh`帮助我们输出我们需要的工具链:

```sh
NDK=/cygdrive/d/development/android-ndk-r10d
SYSROOT=$NDK/platforms/android-19/arch-arm/
mkdir -p /cygdrive/d/development/standalone_toolchain/
$NDK/build/tools/make-standalone-toolchain.sh --arch=arm --platform=android-19 --system=windows-x86_64 --install-dir=/cygdrive/d/development/standalone_toolchain/
chmod -R 755 /cygdrive/d/development/standalone_toolchain
```

辅助脚本`$NDK/build/tools/make-standalone-toolchain.sh`在`/tmp`下创建一个临时目录，将文件复制到该目录，并最终将文件复制到指定的文件夹。请确保您有足够的权限，否则在访问`/tmp`目录时将遇到“权限拒绝”错误。

如果正确配置了路径，我们应该看到以下控制台信息:

```sh
$ ./generate_standalone_toolchain.sh
Auto-config: --toolchain=arm-linux-androideabi-4.8
Copying prebuilt binaries...
Copying sysroot headers and libraries...
Copying c++ runtime headers and libraries...
Copying files to: /cygdrive/d/development/standalone_toolchain/
Cleaning up...
Done.
```

一旦这样做,我们可以看到整个工具链复制从`NDK_ROOT/toolchains`到/`cygdrive/d/development/standalone_toolchain/`。

### 示例: helloworld

为了测试定制的工具链，我们创建了一个示例项目**helloworld**。项目结构非常简单:

```
+-- helloworld
|   +-- hello.cpp
|   +-- Makefile
```

**_hello.cpp_**

```cpp
#include <iostream>
int main()
{
    std::cout << "Hello World!" << std::endl;
}
```

**_Makefile_**

```makefile
STANDALONE_TOOLCHAIN=/cygdrive/d/development/standalone_toolchain/bin/
CC=$(STANDALONE_TOOLCHAIN)/arm-linux-androideabi-gcc
CXX=$(STANDALONE_TOOLCHAIN)/arm-linux-androideabi-g++
CFLAGS=-march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -c -Wall
LDFLAGS=-march=armv7-a -Wl,--fix-cortex-a8
SOURCES=hello.cpp
OBJECTS=$(SOURCES:.cpp=.o)
EXECUTABLE=hello

all: $(SOURCES) $(EXECUTABLE)
$(EXECUTABLE): $(OBJECTS) 
    $(CXX) $(LDFLAGS) $(OBJECTS) -o $@
.cpp.o:
    $(CXX) $(CFLAGS) $< -o $@

clean: 
    rm *.o hello;
```

我们可以简单地按照下面的方式编译代码，我们将得到可执行文件:

```sh
$ cd helloworld
$ make
/cygdrive/d/development/standalone_toolchain/bin//arm-linux-androideabi-g++ -march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -c -Wall hello.cpp -o hello.o
/cygdrive/d/development/standalone_toolchain/bin//arm-linux-androideabi-g++ -march=armv7-a -Wl,--fix-cortex-a8 hello.o -o hello
```

### 另一个示例:clcompute

这是一个比较复杂的例子，使用OpenCL进行平行矢量加法。假设我们有CL和库文件可用:

```
+-- D:\opencl_lib
|   +-- include
|       +-- CL
|   +-- libs
|       +-- libOpenCL.so
```

CL包含头文件可以从Khronos组网站上下载。而libopenclso库文件可以从一个具有opencl功能的手机中检索到。你可以使用`adb pull`从你的手机(或平板电脑)中拉出它。您可以参考本文的第1部分中的表，以获得ibOpenCL.so的详细位置。对于不同的SoC芯片组。

项目结构:

```
+-- clcompute
|   +-- clcompute.cpp
|   +-- Makefile
```

**_Makefile_**

```makefile
STANDALONE_TOOLCHAIN=/cygdrive/d/development/standalone_toolchain/bin/
OPENCL=/cygdrive/d/opencl_lib/
CC=$(STANDALONE_TOOLCHAIN)/arm-linux-androideabi-gcc
CXX=$(STANDALONE_TOOLCHAIN)/arm-linux-androideabi-g++
CFLAGS=-march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -c -I $(OPENCL)/inc/
LDFLAGS=-march=armv7-a -Wl,--fix-cortex-a8 -L$(OPENCL)/libs/ -lOpenCL

SOURCES=clcompute.cpp
OBJECTS=$(SOURCES:.cpp=.o)
EXECUTABLE=clcompute

all: $(SOURCES) $(EXECUTABLE)
$(EXECUTABLE): $(OBJECTS) 
    $(CXX) $(LDFLAGS) $(OBJECTS) -o $@
.cpp.o:
    $(CXX) $(CFLAGS) $< -o $@

clean: 
    rm *.o $(EXECUTABLE);
```

正如您所看到的，一旦您导出了独立的工具链，当您指定了通往工具链的路径时，makefile基本上与普通的makefile相同。我们可以想象，通过使用独立的工具链，我们可以构建一些复杂的项目，这些项目可能需要大量的工作，这是使用`ndk-build`的

## <a name="summary"></a>总结

在这个技术说明(第1部分和第2部分)中，我们介绍了与Android本地项目编译相关的使用和技术。