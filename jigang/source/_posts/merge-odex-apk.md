---
title: Android中odex和apk文件的合并方法
date: 2017-12-14 11:48:01
tags:
- Android
- 逆向
- 反编译
categories:
- Android逆向

---


![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1513233147241&di=4c1fd65a6c356db0169cc8f273fc521d&imgtype=0&src=http%3A%2F%2F7u2thd.com1.z0.glb.clouddn.com%2Fbuck_dex.png)

Android Rom中的APK是被分开的，分为`apk`和`odex`。APK中的`dex`被分离出来成了单独的文件`odex`,包括了Java编译的文件，而`apk`包含了资源文件和`.so`。

这样做可以使其厂商保证一定的反盗版，因为没有dex文件的apk是无法正常安装的，而厂商直接将`odex`和不完整的`apk文件`放到手机rom固化到`/system/bin`中可以让一般用户无法正常导出使用。

可能想到的是`合并odex和apk`变成`apk中包含dex文件`的，这样合并后最终apk文件安装在/data/中，而rom存放在 `/system/app`中，所以最终导致了用户可装在Android手机中的软件会变少，占用系统空间。

下面介绍如何重新打包和签名的方法：

<!-- more -->

## 重新打包

### 下载编译和反编译工具: `smali`和`baksmali`：

下载地址：
https://bitbucket.org/JesusFreke/smali/downloads/


### 将ROM中的framework中的文件全复制出来

```bash
$ cp /home/jigangduan/android/out/target/product/generic/system/framework/*.* boot/
```

### 通过odex生成class文件

```bash
java -jar ../smail/baksmali-2.0.jar -x -d boot/ 111.odex
```

反编译生成：

```bash
out/
└── com
    └── android
        └── sampleplugin
            ├── AnimationSurface.smali
            ├── BackgroundSurface.smali
            ├── BackgroundTest.smali
            ├── graphics
            │   ├── CubeRenderer.smali
            │   └── Cube.smali
            ├── PaintSurface$1.smali
            ├── PaintSurface.smali
            ├── R$attr.smali
            ├── R$drawable.smali
            ├── R.smali
            ├── R$string.smali
            ├── SamplePlugin.smali
            └── VideoSurface.smali
```


### 通过class生成classes.dex 文件

```bash
$ java -Xmx512M -jar ../smail/smali-2.0.jar out -o classes.dex
```

### 将classes.dex放到apk文件

```bash
$ zip -m 111.apk classes.dex
zip I/O error: Permission denied
zip error: Could not create output file (111.apk)

$ sudo chmod +w 111.apk

$ zip -m 111.apk classes.dex
  adding: classes.dex (deflated 52%)
```


## 打包重新签名

### 生成证书  *.keystore

```bash
$ keytool -genkeypair -alias mydemo.keystore -keyalg RSA -validity 100 -keystore mydemo.keystore
Enter keystore password:  
Keystore password is too short - must be at least 6 characters
Enter keystore password:  
Re-enter new password:
What is your first and last name?
  [Unknown]:  111
What is the name of your organizational unit?
  [Unknown]:  logit
What is the name of your organization?
  [Unknown]:  logit
What is the name of your City or Locality?
  [Unknown]:  changsha
What is the name of your State or Province?
  [Unknown]:  hn
What is the two-letter country code for this unit?
  [Unknown]:  ch
Is CN=111, OU=logit, O=logit, L=changsha, ST=hn, C=ch correct?
  [no]:  y

Enter key password for <mydemo.keystore>
(RETURN if same as keystore password):  

```

### 给apk签名

```bash
$ jarsigner -verbose -keystore mydemo.keystore -signedjar 111_ca.apk 111.apk mydemo.keystore
Enter Passphrase for keystore:
 updating: META-INF/MANIFEST.MF
   adding: META-INF/MYDEMO_K.SF
   adding: META-INF/MYDEMO_K.RSA
  signing: AndroidManifest.xml
  signing: lib/armeabi-v7a/libsampleplugin.so
  signing: res/drawable-mdpi/sample_browser_plugin.png
  signing: resources.arsc
  signing: classes.dex

Warning:
The signer certificate will expire within six months.
```
