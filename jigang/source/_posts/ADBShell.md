---
title: ADB Shell
date: 2017-11-02 13:09:20
tags:
- Android
- ADB
categories:
- Android

---


![adb](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509609605843&di=88105c30fe37995440f33d837967ebe5&imgtype=0&src=http%3A%2F%2Fcdn.droidviews.com%2Fwp-content%2Fuploads%2F2013%2F04%2FADB-Android-Debug-Bridge.jpg)

Android调试桥(`adb`)是一种命令行工具，可以让您与仿真器或连接的Android设备进行通信。您可以在android `sdk/platform-tools`中找到adb工具，或者下载[adb工具包](http://adbshell.com/downloads)。

## 目录 ##

1. [ADB调试](#adb_debugging)
1. [无线](#wireless)
1. [包管理器](#package_manager)
1. [文件管理器](#file_manager)
1. [网络](#network)
1. [Logcat](#logcat)
1. [截图](#screenshot)
1. [系统](#system)

<!-- more -->

---

## <a name="adb_debugging"></a>ADB调试

### adb devices

打印所有附加的模拟器/设备的列表

```sh
adb devices
```

打印所有附加的模拟器/设备的列表

```print
e4b25377        device
emulator-5554  device
```

### adb forward

转发socket

`adb forward <local> <remote>`

```sh
adb forward tcp:8000 tcp:9000 ##把PC端8000端口的数据, 转发到Android端的9000端口上
```

执行命令后, PC端的8000端口会被 adb监听, 这个时候我们只需要往8000端口写数据, 这个数据就会发送到手机端的9000端口上.

> 先决条件:在设备上启用USB调试。

### adb kill-server

终止adb服务器进程

```sh
adb kill-server
```

> 注释:如果服务器正在运行，就将其杀死。(终端adb.exe进程)

---

## <a name="wireless"></a>无线

### adb connect

使用ADB Wi-Fi

`adb connect <host>[:<port>]`


_步骤1_

通过USB连接到设备。

_步骤2_

```
adb devices
```
附加的设备列表

```
######## device
```

_步骤3_

```
adb tcpip 5555
```

在TCP模式端口555重新启动

_步骤4_

找出Android设备的IP地址: Settings -> About -> Status -> IP address。记住IP地址 #.#.#.#

_步骤5_

```
adb connect #.#.#.#
```

连接到#.#.#.#:5555

_步骤6_

从设备中移除USB电缆，并确认你仍然可以访问设备:

```
adb devices
```

附加的设备列表

```
#.#.#.#:5555 device
```

> 注意:确保你的主机仍然连接到你的Android设备上的相同的wifi网络。

### adb usb

以USB模式重新启动ADB。

```
adb usb
```

> 参见:adb connect

---

## <a name="package_manager"></a>包管理器

### adb install

安装一个Android应用程序(指定一个完整路径的.apk文件)到一个模拟器/设备。


`adb install [option] <path>`


```
adb install test.apk

adb install -l test.apk ##安装锁定应用程序

adb install -r test.apk ##替换现有的应用程序

adb install -t test.apk ##允许test包

adb install -s test.apk ##sdcard上安装应用程序

adb install -d test.apk ##允许降级版本代码

adb install -p test.apk ##部分应用程序安装
```

### adb uninstall

从模拟器/设备中删除一个包。

`adb uninstall [options] <PACKAGE>`


```
adb uninstall com.test.app
adb uninstall -k com.test.app ##在包删除后保留数据和缓存目录.
```


### adb shell pm list packages

打印所有包，可选的只有那些包名包含<FILTER>文本的包。

`adb shell pm list packages [options] <FILTER>`

```
adb shell pm list packages

adb shell pm list packages -f ##看到他们的相关文件.

adb shell pm list packages -d ##只显示禁用的包.

adb shell pm list packages -e ##过滤仅显示已启用的包.

adb shell pm list packages -s ##只显示系统包.

adb shell pm list packages -3 ##筛选只显示第三方包.

adb shell pm list packages -i ##查看包的安装程序.

adb shell pm list packages -u ##还包括卸载包.

adb shell pm list packages --user <USER_ID> ##查询用户空间.
```

### adb shell pm path

打印给定的<PACKAGE>APK的路径。

`adb shell pm path <PACKAGE>`

```
adb shell pm path com.android.phone
```

package:/system/priv-app/TeleService/TeleService.apk

### adb shell pm clear

删除一个包相关的所有数据。

`adb shell pm clear <PACKAGE>`

```
adb shell pm clear com.test.abc
```

> 注意:清除应用程序数据，缓存

---

## <a name="file_manager"></a>文件管理器

### adb pull

从模拟器/设备下载指定的文件到您的计算机

`adb pull <remote> [local]`

```
adb pull /sdcard/demo.mp4
```

下载/sdcard/demo.mp4 到<android-sdk-path>/platform-tools目录。

```
adb pull /sdcard/demo.mp4 e:\
```

下载/sdcard/demo.mp4 到E盘。

### adb push

从你的电脑上上传一个指定的文件到一个模拟器/设备。

`adb push <local> <remote>`

```
adb push test.apk /sdcard
```

复制 <android-sdk-path>/platform-tools/test.apk 到 /sdcard 目录

```
adb push d:\test.apk /sdcard
```

复制 d:\test.apk 到 /sdcard 目录

### adb shell ls

列出目录的内容

`ls [options] <directory>`

_步骤1_

```
adb shell
```

_步骤2_

```
ls
ls -a ##不隐藏条目
ls -i ##每个文件的打印索引号
ls -s ##每个文件的打印大小，在块中
ls -n ##列出数字uid和gid
ls -R ##递归地列出子目录
```

> 注意:按 Ctrl-C停止

### adb shell cd

改变当前工作目录

_步骤1_

```
adb shell
```

_步骤2_

```
cd /system
```

### adb shell rm

删除文件或目录

_步骤1_

```
adb shell
```

_步骤2_

```
rm /sdcard/test.txt
rm -f /sdcard/test.txt 	## 强制删除没有提示
rm -r /sdcard/tmp 		## 递归地删除目录的内容
rm -d /sdcard/tmp 		## 删除目录，即使它是一个非空目录
rm -i /sdcard/test.txt 	## 提示符之前删除
```

> ⚠️:rm -d相当于rmdir命令

### adb shell mkdir

创建目录

`mkdir [options] <directory name>`

```
mkdir /sdcard/tmp
mkdir -m 777 /sdcard/tmp 			## 设置权限模式
mkdir -p /sdcard/tmp/sub1/sub2 	## 根据需要创建父目录
```

### adb shell touch

创建空文件或更改文件时间戳

`touch [options] <file>`

_步骤1_

```
adb shell
```

_步骤2_

```
touch /sdcard/tmp/test.txt
```

### adb shell pwd

打印当前工作目录位置。

```
pwd
```

### adb shell cp

复制文件和目录

`cp [options] <source> <dest>`

_步骤1_

```
adb shell
```

_步骤2_

```
cp /sdcard/test.txt /sdcard/demo.txt
```

### adb shell mv

移动或重命名文件

`mv [options] <source> <dest>`

_步骤1_

```
adb shell
```

_步骤2_

```
mv /sdcard/tmp /system/tmp 	## 移动
mv /sdcard/tmp /sdcard/test 	## 重命名
```

---

## <a name="network"></a>网络

### adb shell netstat

网络统计信息

`netstat`

_步骤1_

```
adb shell
```

_步骤2_

```
netstat
```

### adb shell ping

测试两个网络连接之间的连接和延迟。

`ping [-aAbBdDfhLnOqrRUvV] [-c count] [-i interval] [-I interface]
[-m mark] [-M pmtudisc_option] [-l preload] [-p pattern] [-Q tos]
[-s packetsize] [-S sndbuf] [-t ttl] [-T timestamp_option]
[-w deadline] [-W timeout] [hop1 ...] destination`

_步骤1_

```
adb shell
```

_步骤2_

```
ping www.google.com
ping www.google.com -c 4
```

### adb shell netcfg

通过配置文件配置和管理网络连接

`netcfg [<interface> {dhcp|up|down}]`

_步骤1_

```
adb shell
```

_步骤2_

```
netcfg
```

### adb shell ip

显示、操作路由、设备、策略路由和隧道

`ip [ OPTIONS ] OBJECT`

OBJECT := { link | addr | addrlabel | route | rule | neigh | ntable |tunnel | tuntap | maddr | mroute | mrule | monitor | xfrm |netns | l2tp }

OPTIONS := { -V[ersion] | -s[tatistics] | -d[etails] | -r[esolve] |-f[amily] { inet | inet6 | ipx | dnet | link } |-l[oops] { maximum-addr-flush-attempts } |-o[neline] | -t[imestamp] | -b[atch] [filename] |-rc[vbuf] [size]}

_步骤1_

```
adb shell
```

_步骤2_

```
ip -f inet addr show wlan0 ## 显示WiFi IP地址
```

---

## <a name="logcat"></a>Logcat

### adb logcat

将日志数据打印到屏幕上。

`adb logcat [option] [filter-specs]`

```
adb logcat
```

```
adb logcat *:V ## 最低优先级，过滤只显示详细级别
adb logcat *:D ## 只显示调试级别
adb logcat *:I ## 只显示信息级别的过滤器
adb logcat *:W ## 只显示警告级别
adb logcat *:E ## 只显示错误级别
adb logcat *:F ## 仅显示致命级别的过滤器
adb logcat *:S ## 无声的，最高的优先级，没有任何东西被打印出来
```

`adb logcat -b <Buffer>`

```
adb logcat -b radio ## 查看包含无线/电话相关消息的缓冲区.
adb logcat -b event ## 查看包含与事件相关的消息的缓冲区.
adb logcat -b main  ## 默认的
```

```
adb logcat -c ## 清除整个日志和退出。
adb logcat -d ## 将日志转储到屏幕并退出。
adb logcat -f ## test.logs记录日志消息输出以进行test.logs。
adb logcat -g ## 打印指定的日志缓冲区和退出的大小。
adb logcat -n <count> ## 将旋转日志的最大数量设置为可数。
adb logcat -r <kbytes> ## 将日志文件旋转到输出的每一个字节。
adb logcat -s ## 设置默认的过滤器规范为静默。
```

`adb logcat -v <format>`

```
adb logcat -v brief ## D显示发送消息(默认格式)的进程的优先级/标记和PID.
adb logcat -v process ## 只显示PID)
adb logcat -v tag ## 只显示优先级/标记。
adb logcat -v raw ## 显示原始的日志消息，没有其他的元数据字段。
adb logcat -v time ## 显示发出消息的日期、调用时间、优先级/标记和PID。
adb logcat -v threadtime ## 显示发出消息的线程的日期、调用时间、优先级、标记和PID和TID。
adb logcat -v long ## 显示所有元数据字段和用空行分隔的消息。
```

### adb shell dumpsys

转储文件系统数据

`adb shell dumpsys [options]`

```
adb shell dumpsys
adb shell dumpsys meminfo
adb shell dumpsys battery
adb shell dumpsys batterystats ## 从你的设备收集电池数据
adb shell dumpsys batterystats --reset ## 消除旧的收集数据
adb shell dumpsys activity
adb shell dumpsys gfxinfo com.android.phone ## 测量com.android.phone 用户界面性能
```

### adb shell dumpstate

转储状态

```
adb shell dumpstate
adb shell dumpstate > state.logs ## 将状态转储到文件中
```

---

## <a name="screenshot"></a>截图

### adb shell screencap

拿一个设备显示屏的截图。

`adb shell screencap <filename>`

```
adb shell screencap /sdcard/screen.png
```
从设备中下载文件

```
adb pull /sdcard/screen.png
```

### adb shell screenrecord [4.4+]

记录运行Android 4.4(API级别19)的设备的显示和更高的显示。

`adb shell screenrecord [options] <filename>`

```
adb shell screenrecord /sdcard/demo.mp4
```

从设备中下载文件

```
adb pull /sdcard/demo.mp4
```

> 注意:按下Ctrl-C停止屏幕记录，否则记录在三分钟内自动停止，或按时间限制设定的时间限制。

```
adb shell screenrecord --size <WIDTHxHEIGHT>
```

设置视频大小:1280x720。默认值是设备的本地显示分辨率(如果支持的话)，如果不是的话，则是1280x720。为了得到最好的结果，使用你的设备的高级视频编码(AVC)编码器所支持的尺寸。

```
adb shell screenrecord --bit-rate <RATE>
```

设置视频的视频比特率，以每秒兆比特的速度。默认值是4Mbps。你可以增加比特率来提高视频质量，但是这样做会导致更大的电影文件。下面的示例将记录比特率设置为5Mbps:adb shell screenrecord --bit-rate 5000000 /sdcard/demo.mp4

```
adb shell screenrecord --time-limit <TIME>
```

设置最大的记录时间，以秒计。默认值和最大值是180(3分钟)。

```
adb shell screenrecord --rotate
```

将输出旋转90度。这个特性是实验性的。

```
adb shell screenrecord --verbose
```

在命令行屏幕上显示日志信息。如果您不设置此选项，则该实用程序在运行时不会显示任何信息。

---

## <a name="system"></a>系统

### adb root

使用根权限重新启动adbd守护进程

```
adb root
```

> 注意:adbd不能在生产构建中作为root运行(在模拟器中进行测试)

### adb sideload

flashing/恢复 Android update.zip包。

```
adb sideload <update.zip>
```

> 注意:adb重新启动Android M+

### adb shell ps

打印进程状态

`ps [options]`

_步骤1_

```
adb shell
```

_步骤2_

```
ps
ps -p
```

### adb shell top

显示最高CPU进程

`top [options]`

_步骤1_

```
adb shell
```

_步骤2_

```
top
top -t ## 显示线程而不是进程.
```

### adb shell getprop

通过安卓property服务获得property

`getprop [options]`

_步骤1_

```
adb shell
```

_步骤2_

```
getprop
getprop ro.build.version.sdk
getprop ro.chipname
getprop | grep adb
```

### adb shell setprop

设置property服务

`setprop <key> <value>`

_步骤1_

```
adb shell
```

_步骤2_

```
setprop service.adb.tcp.port 5555
```
