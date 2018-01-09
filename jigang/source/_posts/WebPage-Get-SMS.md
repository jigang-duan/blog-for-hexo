---
title: Android 网页获取短信验证码 可行性预研
date: 2017-12-18 16:09:01
tags:
- Android
- 预研
categories:
- Android

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1513593742523&di=097d2af5b9b3b97a9e1cd88594ea90bb&imgtype=0&src=http%3A%2F%2Fwww.263.net%2Fr%2Fcms%2Fwww%2F263%2Fresources%2Fimg%2Fsms_banner.jpg)

**目标：** Android 网页获取短信验证码

**目前状态：** 预言阶段（`技术上未实现`）

<!-- more -->

**网页端方案：** Android WebKit插件开发 `SMS-BrowserPlugin`
- 方案描述： WebKit插件包括`PluginService`和`Plugin.so`动态库, Plugin.so提供与WebKit的交互，提供JS的接口；PluginService静态注册WebKit插件action，实现短信拦截功能。

**Android端拦截短信方案**
- 方案一： 监听SMS广播
- 方案二： `NotificationListenserServic`监听Notifications，过滤短信应用包名的通知信息

**过程记录：**
1. 研究Android WebKit插件原理及实现方式
1. 创建示例Plugin项目
1. 动态库需要Android中相关插件部分接口头文件，寻找头文件，下载后放入项目中。又需要依赖其他lib和头文件，依赖过重，依赖文件无从查起，放弃此方案，该用Android源码下编译方式
1. Android源码下编译方式
  1. 安装虚拟机Ubuntu环境，安装`SSH-Server`,`Samba`
  1. 安装`JDK1.6`及Android编译所需要的`依赖库`（JDK版本和依赖环境需要与要编译的Android版本相关）
  1. 安装`repo`
  1. 下载android-4.0_r1源码（翻墙下载速度过慢，使用中科大镜像。下载中由于网速不稳定问题，容易卡住，卡住后没有任何反应，需要停止再重新下载）
  1. 下载后仓库和工作区源码容量近80G，编译存储空间溢出，删除掉仓库.repo，进剩下源码部分
  1. 编译源码
  1. 编译`SampleBrowserPlugin`，生成`SampleBrowserPlugin.apk`和`SampleBrowserPlugin.odex`
  1. 合并`.apk`和`.odex` （SampleBrowserPlugin.apk无法直接安装，合并.apk和.odex，再打包重新签名。但安装还是报错）
1. 使用源码编译的动态库
  1. 解压缩.apk，得到动态库`armeabi-v7a/libsampleplugin.so` 文件
  1. 将libsampleplugin.so加入到示例Plugin项目中
  1. 编译项目生成APK，安装应用并启动
  1. 编写HTML文件，包含插件相关参数的对象
  1. 推入设备的SDCard中，使用浏览器打开(结果还是插件无法加载)
1. 编译项目生成APK，安装应用并启动
1. 脚本使用插件对象
  1. 编写HTML文件，包含插件相关参数的对象
  1. 推入设备的SDCard中，使用浏览器打开(结果还是插件无法加载)

**目前遇到的问题：**
- 插件无法被浏览器正常的加载

**后续方向：**

- 分析`Web API`是否有对应的接口

  `不可行`,

- 分析`WebExtension`是否可行

  `不可行`，WebExtension由HTML、CSS、JS构成，不能访问Android原生接口

- 借助服务器中转


---

参考文档：
- [Android浏览器插件开发](http://blog.csdn.net/ownerwu/article/details/6429072)
- [android 浏览器插件开发](http://blog.csdn.net/dj0379/article/details/18620595)
- [开发者需要了解的WebKit](http://www.infoq.com/cn/articles/webkit-for-developers)
- [Android Webkit插件](http://ffwmxr.blog.163.com/blog/static/6637272220134234470124/)
- [解析Android WebKit插件基本结构](http://mobile.51cto.com/widget-290455.htm)
- [深入理解Android：WebKit卷](http://product.dangdang.com/23917243.html)
- [Android WebKit插件的基本结构](http://blog.csdn.net/makefish/article/details/5947569)
- [Android短信拦截机制适配的坑(下)--4.4以上系统，主要是6.0](http://blog.csdn.net/crazy__chen/article/details/49474381)
- [Android短信拦截机制适配的坑(上)--4.4以下系统](http://blog.csdn.net/crazy__chen/article/details/49473705)
- [AndroidXRef](http://androidxref.com)
- [Android 清华大学镜像](https://mirror.tuna.tsinghua.edu.cn/help/AOSP/)
- [下载Android源代码错误汇总分析](http://blog.csdn.net/mc_hust/article/details/33304733)
- [你真的理解AccessibilityService吗](http://www.jianshu.com/p/4cd8c109cdfb)
- [深入理解SELinux SEAndroid（第一部分）](http://blog.csdn.net/Innost/article/details/19299937)
- [ Android 开源项目指南](http://wiki.jikexueyuan.com/project/android-source/developing.html)
- [【android源码】编译android M源码、刷机，开启源码学习的First Step](http://blog.csdn.net/tyyj90/article/details/53443913)
- [如何利用拦截马实现短信拦截](https://zhuanlan.zhihu.com/p/22124692)
- [微信抢红包外挂源码](https://github.com/lendylongli/qianghongbao)
- [ 6.0短信监听 GetSMSDemo](https://github.com/ddwhan0123/BlogSample/tree/master/GetSMSDemo)
- [PermissionGen](https://github.com/lovedise/PermissionGen)
- [NotificationListenerService不能监听到通知，研究了一天不知道是什么原因？](https://www.zhihu.com/question/33540416)
- [安卓通知栏管理详解及分析 NotificationListenerService](http://blog.csdn.net/cankingapp/article/details/50858229)
