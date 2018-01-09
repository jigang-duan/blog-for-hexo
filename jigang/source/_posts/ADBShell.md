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
1. [调试](#logcat)
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

设备的端口转发

`adb forward <local> <remote>`

```sh
adb forward tcp:8000 tcp:9000 ##把PC端8000端口的数据, 转发到Android端的9000端口上
```

执行命令后, PC端的8000端口会被 adb监听, 这个时候我们只需要往8000端口写数据, 这个数据就会发送到手机端的9000端口上.

> 这个命令对于我们在调试的时候非常有用，特别在 IDA 调试中。

> 先决条件:在设备上启用USB调试。

### adb forward

查看设备中可以被调试的应用的进程号

`adb jdwp`

> 这个命令或许用途不是很多，但是对于调试的时候还是有点用途。可以忽略这个命令。

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

> 注意：如果应用已经安装了，需要使用 adb install –r [ apk 文件] 相当于升级安装

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

## <a name="logcat"></a>调试

### adb logcat

将日志数据打印到屏幕上。

`adb logcat [option] [filter-specs]`

```bash
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

把当前系统中所有应用运行的四大组件都会打印出来

`adb shell dumpsys [options]`

```bash
adb shell dumpsys
adb shell dumpsys meminfo
adb shell dumpsys battery
adb shell dumpsys batterystats ## 从你的设备收集电池数据
adb shell dumpsys batterystats --reset ## 消除旧的收集数据
adb shell dumpsys activity
adb shell dumpsys gfxinfo com.android.phone ## 测量com.android.phone 用户界面性能
```

#### adb shell dumpsys activity top

可以查看当前应用的 activity 信息

用法：
- 运行需要查看的应用，然后运行此命令即可

```bash
$ adb shell dumpsys activity top
TASK com.android.settings id=513
  ACTIVITY com.android.settings/.MainSettings f08d06d pid=7492
    Local Activity d9b221 State:
      mResumed=true mStopped=false mFinished=false
      mChangingConfigurations=false
      mCurrentConfig={1.0 ?mcc?mnc zh_CN ldltr sw360dp w360dp h616dp 320dpi nrml long port finger -keyb/v/h -nav/h s.4 themeChanged=0 themeChangedFlags=0}
      mLoadersStarted=true
      Active Fragments in c1729ea:
        #0: SettingsFragment{99e54db #0 id=0x1020002}
          mFragmentId=#1020002 mContainerId=#1020002 mTag=null
          mState=5 mIndex=0 mWho=android:fragment:0 mBackStackNesting=0
          mAdded=true mRemoving=false mResumed=true mFromLayout=false mInLayout=false
          mHidden=false mDetached=false mMenuVisible=true mHasMenu=false
          mRetainInstance=false mRetaining=false mUserVisibleHint=true
          mFragmentManager=FragmentManager{c1729ea in HostCallbacks{2a70b78}}
          mHost=android.app.Activity$HostCallbacks@2a70b78
          mContainer=android.widget.FrameLayout{3f44a4f V.E...... ........ 0,128-720,1280 #1020002 android:id/content}
          mView=com.miui.internal.widget.ActionBarOverlayLayout{aa116cc V.E...... ........ 0,0-720,1152 #100b004e miui:id/action_bar_overlay_layout}
          Child FragmentManager{d6ea251 in SettingsFragment{99e54db}}:
            FragmentManager misc state:
              mHost=android.app.Activity$HostCallbacks@2a70b78
              mContainer=android.app.Fragment$1@935fab6
              mParent=SettingsFragment{99e54db #0 id=0x1020002}
              mCurState=5 mStateSaved=false mDestroyed=false
      Added Fragments:
        #0: SettingsFragment{99e54db #0 id=0x1020002}
      FragmentManager misc state:
        mHost=android.app.Activity$HostCallbacks@2a70b78
        mContainer=android.app.Activity$HostCallbacks@2a70b78
        mCurState=5 mStateSaved=false mDestroyed=false
    ViewRoot:
      mAdded=true mRemoved=false
      mConsumeBatchedInputScheduled=false
      mConsumeBatchedInputImmediatelyScheduled=false
      mPendingInputEventCount=0
      mProcessInputEventsScheduled=false
      mTraversalScheduled=false
      android.view.ViewRootImpl$NativePreImeInputStage: mQueueLength=0
      android.view.ViewRootImpl$ImeInputStage: mQueueLength=0
      android.view.ViewRootImpl$NativePostImeInputStage: mQueueLength=0
    Choreographer:
      mFrameScheduled=false
      mLastFrameTime=113407 (34516 ms ago)
    View Hierarchy:
      com.android.internal.policy.PhoneWindow$DecorView{430fc62 V.E...... R....... 0,0-720,1280}
        com.miui.internal.widget.ActionBarOverlayLayout{1267cae V.E...... ........ 0,0-720,1280 #100b004e miui:id/action_bar_overlay_layout}
          android.widget.FrameLayout{3f44a4f V.E...... ........ 0,128-720,1280 #1020002 android:id/content}
            android.widget.FrameLayout{9dc5534 V.E...... ........ 0,0-720,1152}
              miui.view.ViewPager{efe4f5d VFED..... ........ 0,0-720,1152 #100b0000 miui:id/view_pager}
              android.widget.ListView{8c295a3 G.ED.VC.. ......I. 0,0-0,0 #102000a android:id/list}
            com.miui.internal.widget.ActionBarOverlayLayout{aa116cc V.E...... ........ 0,0-720,1152 #100b004e miui:id/action_bar_overlay_layout}
              android.widget.FrameLayout{9510d15 V.E...... ........ 0,0-720,1152 #1020002 android:id/content}
                android.widget.FrameLayout{16c61d0 V.E...... ........ 0,0-720,1152}
                  android.widget.ListView{7d335c9 VFED.VC.. .F...... 0,0-720,1152 #102000a android:id/list}
                    ... ...

    Looper (main, tid 1) {d2363b7}
      Message 0: { when=+23s220ms what=1 target=com.xiaomi.mistatistic.sdk.controller.p$1 }
      (Total messages: 1, polling=false, quitting=false)
```

#### adb shell dumpsys package

可以查看指定包名应用的详细信息(相当于应用的 AndroidManifest.xml 中的内容)

`adb shell dumpsys package [pkgname]`

```bash
$ adb shell dumpsys package com.android.settings
Activity Resolver Table:
  Full MIME Types:
      vnd.android.cursor.item/telephony-carrier:
        94d08b7 com.android.settings/.MiuiApnEditor
      vnd.android.document/root:
        2fea824 com.android.settings/.Settings$PublicVolumeSettingsActivity
      vnd.android.cursor.dir/telephony-carrier:
        94d08b7 com.android.settings/.MiuiApnEditor

  Base MIME Types:
      vnd.android.document:
        2fea824 com.android.settings/.Settings$PublicVolumeSettingsActivity

  ... ...

```

>这里看到就是相当于把应用的清单文件打印出来而已。

#### adb shell dumpsys meminfo

可以查看指定进程名或者是进程 id 的内存信息

`adb shell dumpsys meminfo [pname/pid]`

```bash
$ adb shell dumpsys meminfo 7492
Applications Memory Usage (kB):
Uptime: 1360979 Realtime: 1360979

** MEMINFO in pid 7492 [com.android.settings] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap     4526     4440        0        0     8384     7508      875
  Dalvik Heap     6077     6016       36     3872    15479    10500     4979
 Dalvik Other      994      992        0        4                           
        Stack      280      280        0        0                           
       Ashmem       12       12        0        0                           
      Gfx dev     3228     3180        0        0                           
    Other dev        4        0        4        0                           
     .so mmap      733      208        0     2516                           
    .apk mmap       13        0        0        0                           
    .dex mmap      260       32      116       20                           
    .oat mmap     1101      200      200        4                           
    .art mmap     1809     1564       28      104                           
   Other mmap       11        8        0        0                           
      Unknown      261      260        0        0                           
        TOTAL    19309    17192      384     6520    23863    18008     5854

 App Summary
                       Pss(KB)
                        ------
           Java Heap:     7608
         Native Heap:     4440
                Code:      756
               Stack:      280
            Graphics:     3180
       Private Other:     1312
              System:     1733

               TOTAL:    19309      TOTAL SWAP (KB):     6520

 Objects
               Views:      122         ViewRootImpl:        1
         AppContexts:        4           Activities:        1
              Assets:        5        AssetManagers:        2
       Local Binders:       49        Proxy Binders:       32
       Parcel memory:        6         Parcel count:       27
    Death Recipients:        2      OpenSSL Sockets:        0

 SQL
         MEMORY_USED:        0
  PAGECACHE_OVERFLOW:        0          MALLOC_SIZE:       62

```

> 利用这个命令可以查看进程当前的内存情况

#### adb shell dumpsys dbinfo

可以查看指定包名应用的数据库存储信息(包括存储的sql语句)

`adb shell dumpsys dbinfo [packagename]`

```bash
$ adb shell dumpsys dbinfo com.android.mms
Applications Database Info:

** Database info for pid 11635 [com.android.mms] **

Connection pool for /data/user/0/com.android.mms/databases/cache.db:
  Open: true
  Max connections: 1
  Available primary connection:
    Connection #0:
      isPrimaryConnection: true
      onlyAllowReadOnlyOperations: true
      Most recently executed operations:
        0: [2017-12-25 10:37:27.000] executeForCursorWindow took 1ms - succeeded, sql="SELECT action_id, action, timestamp FROM ad_cache"
        1: [2017-12-25 10:37:27.000] prepare took 0ms - succeeded, sql="SELECT action_id, action, timestamp FROM ad_cache"
        2: [2017-12-25 10:37:26.999] executeForLong took 1ms - succeeded, sql="PRAGMA user_version;"
        3: [2017-12-25 10:37:26.999] prepare took 0ms - succeeded, sql="PRAGMA user_version;"
        4: [2017-12-25 10:37:26.998] executeForString took 1ms - succeeded, sql="SELECT locale FROM android_metadata UNION SELECT NULL ORDER BY locale DESC LIMIT 1"
        5: [2017-12-25 10:37:26.995] execute took 3ms - succeeded, sql="CREATE TABLE IF NOT EXISTS android_metadata (locale TEXT)"
        6: [2017-12-25 10:37:26.995] executeForLong took 0ms - succeeded, sql="PRAGMA wal_autocheckpoint=100"
        7: [2017-12-25 10:37:26.995] executeForLong took 0ms - succeeded, sql="PRAGMA wal_autocheckpoint"
        8: [2017-12-25 10:37:26.995] executeForLong took 0ms - succeeded, sql="PRAGMA journal_size_limit=524288"
        9: [2017-12-25 10:37:26.995] executeForLong took 0ms - succeeded, sql="PRAGMA journal_size_limit"
        10: [2017-12-25 10:37:26.995] executeForString took 0ms - succeeded, sql="PRAGMA synchronous"
        11: [2017-12-25 10:37:26.995] executeForString took 0ms - succeeded, sql="PRAGMA journal_mode=PERSIST"
        12: [2017-12-25 10:37:26.994] executeForString took 1ms - succeeded, sql="PRAGMA journal_mode"
        13: [2017-12-25 10:37:26.994] executeForLong took 0ms - succeeded, sql="PRAGMA foreign_keys"
        14: [2017-12-25 10:37:26.993] executeForLong took 1ms - succeeded, sql="PRAGMA page_size"
  Available non-primary connections:
    <none>
  Acquired connections:
    <none>
  Connection waiters:
    <none>

```

> 这里可以清晰的看到应用执行过的 sql 语句信息。在对应用逆向的时候具有一定用途。毕竟可以查看应用操作数据库信息了。

### adb shell dumpstate

转储状态

```bash
adb shell dumpstate
adb shell dumpstate > state.logs ## 将状态转储到文件中
```

---

### adb shell input text

输入文本内容

`adb shell input text [需要输入文本框内容]`

案例：
-  让需要输入内容的文本框获取焦点，adb shell input text 'HelloWorld'

> 注意：这个命令也可以模拟物理按键，虚拟键盘，滑动，滚动等事件

> 延伸：这个命令对于我们在需要输入一大堆信息到文本框中的时候非常有用，比如现在你在 PC 端有一段内容，想输入到手机的某个搜索框中，那么你可以通过把这段内容发送到手机，然后在复制操作。但是有了这个命令就非常简单，先让你想要输入的文本框获取焦点，然后运行这个命令即可。

## <a name="screenshot"></a>截图

### adb shell screencap

截屏操作。

`adb shell screencap <filename>`

```
adb shell screencap /sdcard/screen.png
```
从设备中下载文件

```
adb pull /sdcard/screen.png
```

### adb shell screenrecord [4.4+]

录屏操作

录屏运行Android 4.4(API级别19)或更高的设备。

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
