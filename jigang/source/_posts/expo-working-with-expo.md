---
title: 使用Expo工作
date: 2018-02-08 12:15:06
tags:
- how-to-become-a-react-native-developer
- JavaScript
- 'React Native'
- Expo
categories:
- Expo

---

![](https://d30j33t1r58ioz.cloudfront.net/static/images/spots/docs.gif?f560592076320441267576e0dae10968)

<!-- more -->

---

# 启动和运行

## 创建一个帐户

打开XDE后，将提示输入用户名和密码。填入你想要的用户名和密码，然后点击继续-如果用户名还没有被使用，我们会自动为你创建帐户。

## 创建项目

按下`Project `并选择`New Project`，然后选择选项`Tab Navigation`选项，这将为我们提供一个良好的起点，然后在弹出的对话框中输入项目名称。我将调用我的`first-project`，并按下创建。

接下来，选择保存项目的位置。我把所有有趣的项目都保存在`~/coding`中，所以我导航到那个目录并按下打开。

XDE现在在选择的目录中初始化一个新项目:它复制一个基本的模板，并安装`react `、`react-native`和`expo `。

当项目初始化并准备好时，您将在XDE日志中看到`"React packager ready"`的消息。

“React packager”是一个简单的HTTP服务器，它使用[Babel](https://babeljs.io/)编译我们的应用JavaScript代码，并将其应用到世博会应用程序中。

> 注意:如果你在MacOS上，XDE被困在“Waiting for packager and tunnel to start”的时候，你可能需要[在你的机器上安装watchman](https://facebook.github.io/watchman/docs/install.html#build-install)。最简单的方法是使用Homebrew，`brew install watchman`。

## 在你的手机或模拟器上打开应用

你会发现XDE显示你一个URL `http://4v-9wa.notbrent.mynewproject.exp.direct:80`-可以在浏览器中打开它，你会看到它提供了一些JSON。这个JSON是Expo的清单。我们可以在我们的手机上打开应用程序，把这个URL输入地址栏。或者，按下分享，输入你的电话号码，然后按发送链接。打开你的手机上的信息，点击链接，在Expo上打开它。你可以和任何安装了Expo app的人分享这个链接，但是只要你在XDE中打开这个项目，它就会被使用。

要在iOS模拟器中打开应用，你可以按下`Device`按钮，选择`Open on iOS Simulator`(macOS)。要在Android模拟器中打开应用程序，首先要启动它，然后按下`Device`，然后在`Open on Android`。

## 第一次修改

在您的新项目中打开`screens/HomeScreen.js`，并更改`render()`函数中的任何文本。你应该看到你的应用重新载入你的改变。

### 看不出你的变化吗?

Live重载是默认启用的，但是让我们确保我们可以通过这些步骤来启用它，以防止在某种情况下无法正常工作。

- 首先，确保在XDE中[启用了开发模式]()。
- 接下来，关闭应用程序并重新打开它。
- 一旦应用再次打开，你就可以摇晃你的设备来显示开发者菜单。如果你使用的是模拟器，请按下`⌘+d`在iOS或`ctrl+m`在Android。
- 如果你看到`Enable Live Reload`，按下它，你的应用就会重新加载。如果您看到了`Disable Live Reload`，那么退出开发人员菜单并尝试进行另一个更改。

![](https://docs.expo.io/static/3dcb381ef2f1f8117424ddd3b156514b-57ef1.png)

#### 手动重新加载应用程序

- 如果您已经按照上面的步骤进行了重新加载，仍然没有工作，请按下XDE右下角的按钮来发送一个支持请求。在我们解决这个问题之前，您可以摇动设备并按下Reload，或者使用以下工具之一，它可以同时使用没有开发模式的工具。

![](https://docs.expo.io/static/2bbe2eb78d5256a2a6f26e3926d28f9a-6544b.png)

#### 祝贺你

您已经创建了一个新的Expo项目，进行了更改，并看到了它的更新。

---

# 术语表

### app.json

`app.json`是一个为每个Expo项目而存在的文件，它被用来配置你的项目，例如名字、图标和splash screen。更多信息请阅读[“app.json的配置”]()

### create-react-native-app

React Native与[create-react-app](https://github.com/facebookincubator/create-react-app)是一样的。您可以设置并创建一个无需构建配置的React Native app，并使用Expo来完成这一任务。请阅读更多的[“Expo和创造React Native App”]()。

### 分离

“分离”一词在Expo上被用于描述离开舒适的标准Expo开发环境，在这里你不必处理构建配置或本地代码。当你从Expo“分离”的时候，你会得到[ExpoKit](#expokit)的原生项目，所以你可以继续使用Expo的api来建造你的项目，但是你的工作流程和你在没有Expo的情况下建立一个反应的本地应用是一样的。详情请阅读[“分离ExpoKit”]()。

### eject

“eject”这个词是由create-react-app推广的，它被用于create-react-native-app。当你“eject”你的项目时，你会采取更极端的步骤，而不仅仅是分离——你失去了对Expo APIs的访问，完全离开了Expo的环境。阅读更多关于ejecting。

### Emulator

Emulator用于描述您的计算机上的Android设备的软件仿真器。通常，iOS模拟器被称为Simulators。

### exp

用于与Expo合作的命令行工具。阅读[更多]()。

### 体验

应用程序的同义词通常意味着更单一的用途和更小的范围，有时是艺术的和异想天开的

### Expo Client

### Expo SDK

### ExpoKit

ExpoKit是一个Objective-C和Java库，它允许你使用Expo SDK和平台以及你现有的Expo项目，作为一个更大的标准原生项目的一部分——你通常会用Xcode、Android Studio或者`react-native init`来创建这个项目。阅读更多。

### iOS

在iPhone、iPad和苹果电视上使用的操作系统。Expo目前在iOS版iPhone和iPad上运行。

### 链接

### Manifest

### 本地目录

### npm

### Over the Air updates

### 包管理器

### 发布

### React Native

### Shell app

### Simulator

### Slug

我们在[app.json]()中使用“slug”这个词，指的是它的url中应用程序的名称。例如，Native Component列表应用程序就生活在https://expo.io/@community/native-component-list和slug是native-component-list。

### Snack

[Snack](https://snack.expo.io/)是一种浏览器开发环境，你可以在你的手机或电脑上不安装任何工具，就可以在这里建立Expo[体验](#experience)。

### 独立app

一个可以提交到iOS应用商店或Android Play商店的应用程序二进制文件。[更多地阅读“构建独立应用”](https://docs.expo.io/versions/latest/guides/building-standalone-apps.html)。

### XDE

一个带有图形用户界面(GUI)的桌面工具，用于与Expo项目合作。它与[exp CLI工具]()的功能基本相同，但适用于更熟悉GUI的人。

### yarn

一个用于JavaScript的包管理器。[更多](https://yarnpkg.com/)

---

# 使用app.json配置

`app.json`是你用来配置不属于代码的部分应用程序的首选之地。它位于您的项目的根目录下，紧邻您的`package.json`。它看起来是这样的:

```json
{
  "expo": {
    "name": "My app",
    "slug": "my-app",
    "sdkVersion": "25.0.0",
    "privacy": "public"
  }
}
```

`app.json`之前被称为`exp.json`，但是为了保持与[ Create React Native App](https://github.com/react-community/create-react-native-app)的一致性，它已经被合并到一个文件中。
如果您正在将您的应用程序从使用`exp.json`转换为`app.json`，那么您所需要做的就是在`app.json`的根上添加一个“expo”键，作为所有其他键的父元素。

大多数来自app.json的配置在运行时都可以通过[Expo.Constants.manifest](https://docs.expo.io/versions/latest/sdk/constants.html#expoconstantsmanifest)的JavaScript代码访问。
诸如密匙之类的敏感信息被删除。有关如何将任意配置数据传递给应用程序的信息，请参阅下面的`"extra"`键。

下面是在`app.json`的`"extra"`关键字下面提供的属性列表:

- `name`

  **必需**。你的app的名字在Expo上和你的home屏幕上都是一个独立app。

- `description`

  简短地描述一下你的应用是什么，以及它为什么很棒。

- `slug`

  **必需**。用于发布的友好url名称。例如:expo.io/@your-username/slug。

- `privacy`

  `public `或`unlisted`。如果没有提供，则默认为`unlisted`。在未来，`private `将得到支持。`unlisted`隐藏了搜索结果的经验。有效值:`public `、`unlisted`

- `sdkVersion`

  **必需**。运行项目的Expo sdkVersion。这应该与您的`package.json`中指定的版本一致。

- `version`

  你的app版本，使用你喜欢的任何版本控制方案。

- `platforms`

  您的项目明确支持的平台。如果没有指定，它会默认为`["ios", "android"]`。

- `githubUrl`

  如果你想在Github上分享你的应用程序的源代码，那么在这里输入存储库的URL，它将链接到你的Expo项目页面。

- `orientation`

  把你的应用程序锁定在一个特定的方向上，用`portrait`或`landscape`。默认为没有锁。有效值:`‘default’`, `‘portrait’`, `‘landscape’`

- `primaryColor`

  在Android上，这将决定你的应用在多任务的颜色。目前这款应用还没有在iOS上使用，但将来可能会用于其他用途。6个字符长的十六进制颜色字符串，如:`'#000000'`

- `icon`

  本地路径或远程url到应用程序图标的图像。我们建议您使用1024x1024 png文件。这个图标将会出现在主屏和Expo应用程序中。

- `notification`

  远程(推送)通知的配置

  - `icon`
  本地路径或远程url到一个图像，用作推送通知的图标。透明的48x48 png灰度图。
  - `color`
  当它出现在通知托盘中时，为推送通知图像着色。6个字符长的十六进制颜色字符串，如:`'#000000'`
  - `androidMode`
  分别显示每个推送通知(default)或collapse为一个(collapse)。有效值:‘default’, ‘collapse’
  - `androidCollapsedTitle`
  如果`androidMode`被设置为`collapse`，这个标题将被用于崩溃的通知消息。例如:`’#{unread_notifications} new interactions’`

- `loading`

  **弃用**:使用`splash`代替。用户在打开应用程序时看到的加载屏幕的配置，同时获取和缓存捆绑包和资产。

  - `icon`
  在启动应用程序时，本地路径或远程url显示的图像。图像大小和纵横比取决于你。必须是一个.png。
  - `exponentIconColor`
  如果没有提供icon，我们将展示Expo标志。你可以在white和blue之间选择。有效值:‘white’, ‘blue’
  - `exponentIconGrayscale`
  类似于`exponentIconColor `，但表示它应该是灰度`(1)`还是不`(0)`。
  - `backgroundImage`
  本地路径或远程url到图像以填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。
  - `backgroundColor`
  颜色填充加载屏幕背景6字符长的十六进制颜色字符串， eg: `'#000000'`
  - `hideExponentText`
  在默认情况下，Expo在加载屏幕的底部显示了一些文本。设置为true，禁用

- `appKey`

  在默认情况下，世博会将作为`main `的注册地申请注册。如果您想要更改这个属性，您可以在该属性中指定名称。

- `androidStatusBarColor`

  **弃用**。使用`androidStatusBar`代替。6个字符长的十六进制颜色字符串，如:'#000000'

- `androidStatusBar`

  配置android状态栏。

  - `barStyle`
  配置状态栏图标，以有亮或暗颜色。有效值:‘light-content’, ‘dark-content’
  - `backgroundColor`
  配置android状态栏。6字符长十六进制颜色字符串，eg: '#000000'

- `androidShowExponentNotificationInShellApp`

  使用refresh按钮和调试信息向您的独立应用程序添加一个通知。

- `scheme`

  独立的应用程序。链接到你的应用程序的URL方案。例如，如果我们把它设置为'demo'，那么demo:// URLs会在点击时打开你的应用。字符串以字母开头，然后是字母、数字、”+”, ”.” or ”-” 的组合。

- `entryPoint`

  相对路径到您的主JavaScript文件。

- `extra`

  你想要传递给你的经验的任何额外的领域。值可以通过`Expo.Constants.manifest.extra`。([了解更多](https://docs.expo.io/versions/latest/sdk/constants.html#expoconstantsmanifest))

- `rnCliPath`
- `packagerOpts`
- `ignoreNodeModulesValidation`
- `nodeModulesPath`

- `ios`

  **独立的应用程序**。iOS独立应用特定配置

  - `bundleIdentifier`
  这是你的iOS独立应用的bundle id，你可以在app Store中使用它，但它必须是独一无二的。看到这个StackOverflow问题。iOS绑定标识符标记为您的应用程序的唯一名称。例如，exp.host。我们的Expo是我们的域名，而Expo就是我们的应用。
  - `buildNumber`
  为你的iOS独立应用构建数字，必须是一个匹配苹果格式的[CFBundleVersion格式](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CoreFoundationKeys.html#//apple_ref/doc/uid/20001431-102364)的字符串
  - `icon`
  本地路径或远程URL到你的应用在iOS上的图标。如果指定了，这将覆盖顶级的图标键。使用一个1024x1024的图标，遵循苹果的图标的界面指南，包括颜色配置和透明度。世博会将产生其他所需的规模。这个图标将会出现在主屏和Expo应用程序中。
  - `merchantId`
  在你的独立应用中使用Apple Pay的商家ID。
  - `appStoreUrl`
  如果你已经把它部署到苹果应用商店，你可以在苹果app Store上使用它。如果你的应用是公开的，你可以在你的世博会项目页面上链接到你的商店页面。
  - `config`
    - `branch`
    [分支](https://branch.io/)键连接分支连接服务。
      - `apiKey`
      你的分支API Key
    - `usesNonExemptEncryption`
    设置独立ipa’s Info.plist `ITSAppUsesNonExemptEncryption`,给定的布尔值。
    - `googleMapsApiKey`
    为你的独立应用提供的[Google Maps iOS SDK key](https://developers.google.com/maps/documentation/ios-sdk/start)。
    - `googleSignIn`
    为你的独立应用提供的[Google Sign-In iOS SDK](https://developers.google.com/identity/sign-in/ios/start-integrating)
    - `reservedClientId`
    保留的客户端ID URL方案。可以在`GoogeService-Info.plist`中找到
  - `isRemoteJSEnabled`
  如果设置为false，您的独立应用程序将永远不会下载任何代码，并且只会使用在设备上本地绑定的代码。在这种情况下，你的应用的所有更新都必须通过苹果审核提交。默认值为true。(请注意，这将无法通过ExpoKit项目来实现)
  - `loadJSInBackgroundExperimental`
  如果是这样的话，你的独立应用程序会立即运行它的缓存JS包，如果存在的话，并在后台请求一个新的。
  - `supportsTablet`
  你的独立iOS应用是否支持平板电脑的尺寸。默认值为false。
  - `isTabletOnly`
  如果这是true，说明你的独立iOS应用不支持手机，只支持平板电脑。
  - `infoPlist`
  任意配置的字典，添加到您的独立应用程序的本地Info.plist。适用于所有其他特定于Expo配置。没有执行其他验证，所以在应用程序商店的拒绝中使用它。
  - `associatedDomains`
  一个包含独立应用程序的相关域的数组。
  - `usesIcloudStorage`
  一个布尔值，指示该应用是否使用用于DocumentPicker的iCloud存储。有关详细信息，请参阅DocumentPicker文档。
  - `splash`
  为独立的iOS应用提供加载和启动屏幕的配置。
    - `backgroundColor`
    颜色填充屏幕背景6个字符的长十六进制颜色字符串，如:'#000000'
    - `resizeMode`
    确定图像将如何显示在弹出的加载屏幕中。必须是一个包含或包含的默认值。有效值:‘cover’, ‘contain’
    - `image`
    本地路径或远程url到一个图像来填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。
    - `tabletImage`
    本地路径或远程url到图像以填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。

  - `android`
    独立的应用程序。Android独立应用程序特定配置

    - `package`
    的Android独立应用程序的包名，你可以做，但它需要在Play Store中是独一无二的。看到这个StackOverflow问题。反向DNS标记为您的应用程序的唯一名称。例如，host.exp.exponent。我们的域名是我们的域名，而Expo就是我们的应用。
    - `versionCode`
    Google Play需要的版本号。每个版本增加一个。必须是一个整数。https://developer.android.com/studio/publish/versioning.html
    - `icon`
    本地路径或远程url到你的应用程序在Android上的图标。如果指定了，这将覆盖顶级的图标键。我们建议您使用1024x1024 png文件(Google Play商店推荐使用透明度)。这个图标将会出现在主屏和世博会应用程序中。
    - `playStoreUrl`
    如果你已经部署在Google Play Store，那么你的应用程序的URL。如果你的应用程序是公开的，这是用来链接到你的商店页面的。
    - `permissions`
    独立应用程序使用的权限列表。删除该字段以使用默认的权限列表。
    示例: `[ "CAMERA", "ACCESS_FINE_LOCATION" ]`.
    您可以根据需要指定以下权限:
      - ACCESS_COARSE_LOCATION
      - ACCESS_FINE_LOCATION
      - CAMERA
      - MANAGE_DOCUMENTS
      - READ_CONTACTS
      - READ_CALENDAR
      - WRITE_CALENDAR
      - READ_EXTERNAL_STORAGE
      - READ_INTERNAL_STORAGE
      - READ_PHONE_STATE
      - RECORD_AUDIO
      - USE_FINGERPRINT
      - VIBRATE
      - WAKE_LOCK
      - WRITE_EXTERNAL_STORAGE
      - com.anddoes.launcher.permission.UPDATE_COUNT
      - com.android.launcher.permission.INSTALL_SHORTCUT
      - com.google.android.c2dm.permission.RECEIVE
      - com.google.android.gms.permission.ACTIVITY_RECOGNITION
      - com.google.android.providers.gsf.permission.READ_GSERVICES
      - com.htc.launcher.permission.READ_SETTINGS
      - com.htc.launcher.permission.UPDATE_SHORTCUT
      - com.majeur.launcher.permission.UPDATE_BADGE
      - com.sec.android.provider.badge.permission.READ
      - com.sec.android.provider.badge.permission.WRITE
      - com.sonyericsson.home.permission.BROADCAST_BADGE
    - `config`
      - `branch`
      [分支](https://branch.io/)键连接分支链接服务。
        - `apiKey`
        你分支API key
      - `fabric`
      [Google Developers Fabric](https://get.fabric.io/) keys连接Crashlytics和其他服务。
        - `apiKey`
        你Fabric API key
        - `buildSecret`
        Fabric build secret
      - `googleMaps`
      你的独立app的[Google Maps Android SDK](https://developers.google.com/maps/documentation/android-api/signup) key。
        - `apiKey`
        Google Maps Android SDK API key
      - `googleSignIn`
      为你的独立应用提供的[Google Sign-In iOS SDK](https://developers.google.com/identity/sign-in/ios/start-integrating)
        - `apiKey`
        Android API key。可以在开发人员控制台的凭据部分或在`google-services.json`中找到。
        - `certificateHash`
        用于构建apk的签名证书的SHA-1哈希，没有任何分隔符:。可以在`google-service.json`中找到。https://developers.google.com/android/guides/client-auth
    - `splash`
    为独立的Android应用程序加载和启动屏幕的配置。
      - `backgroundColor`
      颜色填充屏幕背景6个字符的长十六进制颜色字符串，如: '#000000'
      - `resizeMode`
      确定图像将如何显示在弹出的加载屏幕中。必须是一个`cover`或`contain `,默认值`contain`。有效值:‘cover’, ‘contain’
      - `ldpi`
      本地路径或远程url到一个图像来填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。
      - `mdpi`
      本地路径或远程url到一个图像来填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。
      - `hdpi`
      本地路径或远程url到一个图像来填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。
      - `xhdpi`
      本地路径或远程url到一个图像来填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。
      - `xxhdpi`
      本地路径或远程url到一个图像来填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。
      - `xxxhdpi`
      本地路径或远程url到一个图像来填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。

- `facebookAppId`

  用于所有的Facebook库。在https://developers.facebook.com上设置你的Facebook应用程序ID。

- `facebookDisplayName`

  用于原生的Facebook登录。

- `facebookScheme`

  用于Facebook本地登录。从“fb”开始，接着是一系列的数字，比如“fb1234567890”。你可以在https://developers.facebook.com/docs/facebook-login/ios找到你的方案的配置信息。plist”部分。

- `splash`

  为独立应用程序加载和启动屏幕的配置。
  - `backgroundColor`
  颜色填充屏幕背景6个字符的长十六进制颜色字符串，如:'#000000'
  - `resizeMode`
  确定图像将如何显示在弹出的加载屏幕中。必须是一个包含或包含的默认值。有效值:‘cover’, ‘contain’
  - `image`
  本地路径或远程url到一个图像来填充加载屏幕的背景。图像大小和纵横比取决于你。必须是一个.png。

- `hooks`

  对脚本的配置，以将其连接到发布过程中

  - `postPublish`

- `assetBundlePatterns`

  一组文件glob字符串，它指向的资产将被绑定到独立的应用程序二进制文件中。在[离线支持指南](https://docs.expo.io/versions/latest/guides/offline-support.html)中阅读更多信息

---

# 开发模式

React Native包括一些非常有用的开发工具:在Chrome中进行远程JavaScript调试、实时重载、热重加载，以及一个类似于您在Chrome中使用的受欢迎的检查器的元素检查器。它还执行一些验证，当您的应用程序正在运行时，如果您使用的是不赞成的属性，或者您忘记将必需的属性传递到组件中，那么它将给您发出警告。

![](https://docs.expo.io/static/6f6446f29d456d65316879ceb11f2bc9-640d6.png)

**这是有代价的**:你的应用在开发模式下运行得比较慢。你可以在XDE上来回切换。当你打开它时，关闭并重新打开你的应用程序，让它生效。**在测试应用程序的性能时，一定要禁用开发模式**。

## 在XDE中切换开发模式

![](https://docs.expo.io/static/5a9f2406d395981fd83ca2dd1a6c2213-97f44.png)

---

# exp命令行接口

除了XDE之外，我们还有一个CLI `exp`，如果您喜欢在命令行上工作，或者想要在测试或持续集成(CI)中使用Expo。

## 安装

运行 `npm install -g exp` 安装全局`exp`.

如果您以前没有使用过exp或XDE，那么您需要做的第一件事就是使用`exp login`登录您的Expo帐户。

## 命令

查看使用`exp --help`的命令列表:

```

Usage: exp [options] [command]


Options:

  -V, --version          output the version number
  -o, --output [format]  Output format. pretty (default), raw
  -h, --help             output usage information


Commands:

  android [options] [project-dir]                 Opens your app in Expo on a connected Android device
  build:ios|bi [options] [project-dir]            Build a standalone IPA for your project, signed and ready for submission to the Apple App Store.
  build:android|ba [options] [project-dir]        Build a standalone APK for your project, signed and ready for submission to the Google Play Store.
  build:status|bs [options] [project-dir]         Gets the status of a current (or most recently finished) build for your project.
  convert|onentize [options] [project-dir]        Initialize Expo project files within an existing React Native project
  detach [options] [project-dir]                  Creates Xcode and Android Studio projects for your app. Use this if you need to add custom native functionality.
  diagnostics [options] [project-dir]             Uploads diagnostics information and returns a url to share with the Expo team.
  doctor [options] [project-dir]                  Diagnoses issues with your Expo project.
  fetch:ios:certs [options] [project-dir]         Fetch this project' iOS certificates. Writes to PROJECT_DIR/PROJECT_NAME_(dist|push).p12 and prints passwords to stdout.
  fetch:android:keystore [options] [project-dir]  Fetch this project's Android keystore. Writes keystore to PROJECT_DIR/PROJECT_NAME.jks and prints passwords to stdout.
  init|i [options] [project-dir]                  Initializes a directory with an example project. Run it without any options and you will be prompted for the name and type.
  install:ios [options]                           Install the latest version of Expo Client for iOS on the simulator
  install:android [options]                       Install the latest version of Expo Client for Android on a connected device or emulator
  ios [options] [project-dir]                     Opens your app in Expo in an iOS simulator on your computer
  login|signin [options]                          Login to Expo
  logout [options]                                Logout from exp.host
  path [options]                                  Sets PATH for XDE
  prepare-detached-build [options] [project-dir]  Prepares a detached project for building
  publish:history|ph [options] [project-dir]      View a log of your published releases.
  publish:details|pd [options] [project-dir]      View the details of a published release.
  publish:set|ps [options] [project-dir]          Set a published release to be served from a specified channel.
  publish:rollback|pr [options] [project-dir]     Rollback an update to a channel.
  publish|p [options] [project-dir]               Publishes your project to exp.host
  register [options]                              Sign up for a new Expo account
  send [options] [project-dir]                    Sends a link to your project to a phone number or e-mail address
  start|r [options] [project-dir]                 Starts or restarts a local server for your app and gives you a URL to it
  url|u [options] [project-dir]                   Displays the URL you can use to view your project in Expo
  whoami|w [options]                              Checks with the server and then says who you are logged in as
```

通过传递--help标志查看关于特定命令的其他信息。例如，`exp start --help`输出:

```
Usage: start|r [options] [project-dir]

Starts or restarts a local server for your app and gives you a URL to it


Options:

  -s, --send-to [dest]   A phone number or e-mail address to send a link to
  -c, --clear            Clear the React Native packager cache
  -a, --android          Opens your app in Expo on a connected Android device
  -i, --ios              Opens your app in Expo in a currently running iOS simulator on your computer
  -m, --host [mode]      tunnel (default), lan, localhost. Type of host to use. "tunnel" allows you to view your link on other networks
  -p, --protocol [mode]  exp (default), http, redirect. Type of protocol. "exp" is recommended right now
  --tunnel               Same as --host tunnel
  --lan                  Same as --host lan
  --localhost            Same as --host localhost
  --dev                  Turns dev flag on
  --no-dev               Turns dev flag off
  --strict               Turns strict flag on
  --no-strict            Turns strict flag off
  --minify               Turns minify flag on
  --no-minify            Turns minify flag off
  --exp                  Same as --protocol exp
  --http                 Same as --protocol http
  --redirect             Same as --protocol redirect
  --non-interactive      Fails if an interactive prompt would be required to continue.
  --offline              Allows this command to run while offline
  -h, --help             output usage information
```

另外，你可以在离线模式下运行，将离线标志传递给android、ios或启动命令。

---

# 查看日志

在Expo应用程序中写入日志就像在浏览器中一样:使用`console.log`, `console.warn`, `console.error`。
注意:我们目前不支持远程调试模式之外的`console.table`。

## 推荐:使用Expo工具查看日志

当你打开一个从XDE或exp中服务的应用程序时，该应用会将日志发送到服务器，并让它们方便地提供给你。这意味着你甚至不需要把你的设备连接到你的电脑上查看日志——事实上，如果有人打开了世界另一端的应用，你仍然可以从他们的设备上看到你的应用的日志。

### XDE日志面板

当您在XDE中打开一个项目时，日志窗口将被一分为二。您的应用程序日志显示在右边，而packager日志显示在左边。

![](https://docs.expo.io/static/6e16643410992b1e7bd3f39f6f604ef4-cca32.png)

XDE还允许您在任何打开应用程序的设备之间切换日志。

![](https://docs.expo.io/static/6c8db7841722e4eeb57c4b14fcaee869-cca32.png)

### 使用`exp`查看日志

如果您使用我们的命令行工具exp，那么当您的项目运行时，packager日志和应用日志都将自动地流。为了停止您的项目(并结束日志流)，使用`ctrl+c`终止进程。

## 可选:手动访问设备日志

虽然这通常是不必要的，但是如果你想看到你的设备上发生的所有事情，甚至是来自其他应用和操作系统本身的日志，你可以使用以下方法之一

### 查看iOS模拟器的日志

#### 选项1:使用GUI日志

- 在模拟器中，按`⌘ + /`或进入`Debug -> Open System Log`-这两个打开一个日志窗口，显示设备上的所有日志，包括Expo应用程序的日志。

#### 选项2:在终端打开它

- 运行`instruments -s devices`
- 找到你正在使用的模拟器的设备/操作系统版本，eg: `iPhone 6s (9.2) [5083E2F9-29B4-421C-BDB5-893952F2B780]`
- 括号中的部分结束时设备代码,因此您现在可以这样做: `tail -f ~/Library/Logs/CoreSimulator/DEVICE_CODE/system.log`, 如: `tail -f ~/Library/Logs/CoreSimulator/5083E2F9-29B4-421C-BDB5-893952F2B780/system.log`

### 查看iPhone的日志

- `brew install libimobiledevice`
- 代入你的手机
- `idevicepair pair`
- 在你的设备上按下 accept
- 运行 `idevicesyslog`

### 从Android设备或模拟器查看日志

- 确保安装了Android SDK
- 确保在您的设备上启用了USB调试(对于仿真器来说没有必要)。
- 运行`adb logcat`

---

# 调试

## 使用 Simulator / Emulator

在一个实际的设备上测试你的应用程序的性能和感觉是没有任何替代的，但是在调试过程中，你可能会更容易使用模拟器。

苹果将他们的模拟器称为“Simulator”，而谷歌将他们的模拟器称为“Emulator”。

### iOS

确保你有最新的Xcode(比如Mac App Store)。这包括iOS模拟器，以及其他一些工具。

### Android

在Android上，我们推荐使用标准模拟器的Genymotion模拟器——我们发现它更完整、更快、更容易使用。

[下载Genymotion](https://www.genymotion.com/fun-zone/)(免费版本)并遵循[Genymotion安装指南](https://docs.genymotion.com/Content/01_Get_Started/Installation.htm)。一旦你安装了Genymotion，创建一个虚拟设备——我们推荐一个Nexus 5, Android版本由你决定。启动虚拟设备时，它已经准备好了。如果你遇到任何问题，请遵循我们的[Genymotion指南](https://docs.expo.io/versions/latest/guides/genymotion.html#genymotion)。

## 开发者菜单

这个菜单使您能够访问几个对调试有用的函数。它也被称为调试菜单。调用它取决于您正在运行应用程序的设备。

### iOS设备上

摇一下这个装置。

### 在iOS模拟器

在模拟器上的Mac上按`Ctrl-Cmd-Z`来模拟握手动作，或者按下`Cmd+D`。

### 在Genymotion

在Genymotion的工具栏中按下“菜单”按钮，或者直接点击`Cmd-m`。

## 调试Javascript

你可以使用Chrome调试器工具来调试Expo应用程序。与其在你的手机上运行你的应用程序的JavaScript，不如将它运行在Chrome浏览器的一个webworker中。然后，您可以设置断点、检查变量、执行代码等等，就像调试web应用程序时所做的那样。

- 为了确保最佳调试体验，首先将您的主机类型更改为LAN或localhost。如果你使用了启用调试的隧道，那么你很可能会经历太多的延迟，以至于你的应用无法使用。在这里，也确保检查开发模式。

![](https://docs.expo.io/static/0af875d134c9a8835b71baaa0e1791bc-d94ef.png)

- 如果你正在使用局域网，请确保你的设备与你的开发机器在同一个wifi网络上。这可能不会在某些公共网络上奏效。除非你在模拟器中，否则本地主机将无法工作，而且只有在你的设备通过usb连接到你的机器时，它才会在Android上运行。

- 在你的设备上打开这个应用，打开开发者菜单，然后点击`Debug JS Remotely`。这将打开一个浏览器选项卡URL ` http://localhost:19001/debugger-ui`。从那里，您可以设置断点，并通过JavaScript控制台进行交互。当你完成的时候，摇动这个设备，停止Chrome的调试。

- 当使用Chrome调试时，`console.log`语句的行号在默认情况下不会起作用。要获得正确的行号，打开Chrome开发工具设置，进入“black装箱”选项卡，确保选中“Blackbox内容脚本”，并将`expo/src/Logs.js`添加为带有“Blackbox”选项的模式。

### 故障主机调试

当您在XDE中打开一个项目时，当您在Android上打开时，XDE将自动告诉您的设备转发localhost:19000和19001到您的开发机器，只要您的设备被插入或模拟器正在运行。如果您正在使用localhost进行调试，并且它没有工作，关闭应用程序并在Android上再次打开它。或者，如果您安装了Android开发工具，您可以使用以下命令手动转发端口:`adb reverse tcp:19000 tcp:19000` - `adb reverse tcp:19001 tcp:19001`


### 源码映射和异步函数

源码映射和异步函数不是100%可靠的。在每种情况下，本地的响应都不能很好地使用Chrome的源代码映射，所以如果你想要确保你在正确的地方使用了断点，你应该直接从你的代码中使用调试器调用。

## HTTP调试

要调试应用程序的HTTP请求，您应该使用代理。以下的选项将全部奏效:

- [Charles Proxy](https://www.charlesproxy.com/documentation/configuration/browser-and-system-configuration/)
- [mitmproxy](https://medium.com/@rotxed/how-to-debug-http-s-traffic-on-android-7fbe5d2a34#.hnhanhyoz)
- [Fiddler](http://www.telerik.com/fiddler)

在Android上，[代理设置](https://play.google.com/store/apps/details?id=com.lechucksoftware.proxy.proxysettings)应用程序有助于在调试和非调试模式之间切换。不幸的是，它还不能与Android M兼容。

未来的工作是在Chrome DevTools中显示网络请求。

## 热加载和重新加载

[热模块重载](http://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html)是一种快速重载更改的方法，不会在屏幕或导航堆栈中丢失状态。要启用，请调用开发人员菜单并点击“启用热重加载”项。而Live Reload将重新加载整个JS上下文，热模块重载将使您的调试周期更快。但是，确保您没有打开两个选项，因为这是不支持的行为。

## 其他调试技巧

Dotan Nahum在他的“[Debugging React Native Applications](https://medium.com/reactnativeacademy/debugging-react-native-applications-6bff3f28c375)”中概述了其他有用的工具，比如监视桥梁信息和JSEventLoopWatchdog。

---

# Genymotion

---

# 发布

当您在开发项目时，您正在计算机上编写代码，当您使用XDE或exp时，服务器和本地包装器会在您的机器上运行，并将所有的源代码打包，并使其从URL中获得。您正在处理的项目的URL可能是这样的:`exp://i3-kvb.ccheever.an-example.exp.direct:80`。

`exp.direct`是我们用来进行隧道挖掘的领域，所以即使你在VPN或防火墙后面，任何有你的URL的设备都应该能够访问你的项目。这使得在你的手机上打开你的项目或发送给你正在和谁不在同一局域网的其他人更容易。

但是由于包装器和服务器在您的计算机上运行，如果您关闭了您的笔记本或关闭XDE，您将无法从该URL加载您的项目。“发布”是我们用来部署您的项目的术语。它使您的项目可以在一个持久的URL中使用，例如`https://expo.io/@community/native-component-list`。它还会将你的所有应用图片、字体和视频上传至CDN(在这里阅读更多)。

## 如何发布

要发布项目，请单击XDE中的发布按钮。(在窗户的右上角。)如果您正在使用exp cli工具，则运行`exp publish`。不需要设置，继续创建一个新项目，并在不发生任何更改的情况下发布它，您将看到它是有效的。

当你这么做的时候，packager会将你所有的代码都缩小，并生成两个版本的代码(一个用于iOS，一个用于Android)，然后将这些代码上传到CDN上。您将获得一个链接，如`https://exp.host/@ccheever/an-example`，任何人都可以从这个项目中加载您的项目。

任何时候您想要部署一个更新，再次点击发布，新的版本将会在用户下一次打开它的时候立即可用。

## 部署到App Store和Play Store

当你准备好将你的应用分发给最终用户时，你可以创建一个独立的应用程序二进制文件(ipa或apk文件)，然后将它放到iOS应用商店和Google Play商店中。看到分发应用程序。

这个独立的应用程序知道在你的应用发布的url中寻找更新，如果你发布了一个更新，那么下次用户打开你的应用时，他们会自动下载这个新版本。这些通常被称为“空中”(OTA)更新，其功能类似于[CodePush](https://microsoft.github.io/code-push/)，但它是内置在Expo上的，所以你不需要安装任何东西。

要配置你的应用程序处理JS更新的方式，请参见[离线支持](https://docs.expo.io/versions/latest/guides/offline-support.html)。

## 限制

如果您在app.json中做出以下任何更改，您将需要重新构建应用程序的二进制文件，以便更改生效:

- 增加Expo SDK版本
- 在ios或安卓系统下修改任何东西
- 改变你的应用程序启动
- 改变你的应用程序图标
- 改变你的应用程序的名字
- 改变你的应用方案
- 改变你的facebookScheme
- 在assetBundlePatterns下改变你的捆绑资产

## 隐私

您可以将项目的隐私设置在您的`app.json`配置文件中，将关键的`“privacy”`设置为`“public”`或`“unlisted”`。

这些选项的工作方式与YouTube上类似。未列出的项目url将是秘密的，除非您告诉人们关于它们或共享它们。公共项目可能会出现在其他开发人员身上。

---

# 链接

## 介绍

每个好的网站都有`https://`，https就是所谓的URL方案。不安全的网站使用`http://`，http是URL方案。我们把它叫做短期计划。

要从一个网站导航到另一个网站，你可以在网络上使用一个锚标签(`<a>`)。您还可以使用像窗口这样的JavaScript APIs `window.history`和`window.location`。

除了`https`之外，您可能还熟悉`mailto` scheme。当您打开mailto scheme的链接时，您的操作系统将打开一个已安装的邮件应用程序。如果您安装了多个邮件应用程序，那么您的操作系统可能会提示您选择一个。类似地，也有打电话和发短信的计划。请阅读下面关于[内置URL方案](https://docs.expo.io/versions/latest/guides/linking.html#built-in-url-schemes)的更多信息。

https和http是由您的浏览器处理的，但是可以通过使用不同的url方案链接到其他应用程序。例如，当你从Slack获得一个“神奇链接”的电子邮件时，“启动Slack”按钮是一个带有href的锚标签，它看起来像是:`slack://secret/magic-login/other-secret`。就像Slack一样，你可以告诉操作系统你想要处理一个定制的方案。请阅读有关[配置方案](https://docs.expo.io/versions/latest/guides/linking.html#in-a-standalone-app)的更多信息。当Slack应用打开时，它会收到用于打开它的URL，然后可以对通过URL提供的数据进行操作——在这种情况下，是一个将用户登录到特定服务器的秘密字符串。这通常被称为深度链接。阅读更多关于如何[处理你的应用的深度链接](https://docs.expo.io/versions/latest/guides/linking.html#handling-links-into-your-app)。

深度链接与scheme并不是唯一可用的链接工具——我们正在努力为iOS上的通用链接添加支持，我们还支持与分支的延迟深度链接。我们将在未来的sdk中更新更多的信息。

## 从你的应用程序到其他应用程序

### 内置的URL方案

正如在简介中提到的，在每个平台上都存在一些核心功能的URL方案。下面是一个不完整的列表，但是涵盖了最常用的方案。

Scheme | 描述 | iOS | Android
--|--|--|--
`mailto` | 打开邮件应用，如:`mailto: support@expo.io` | ✅ | ✅
`tel`	| 打开phone app, eg: tel:+123456789	| ✅	| ✅
`sms`	| 打开SMS app, eg: sms:+123456789	| ✅	| ✅
`https` / `http`	| 打开web浏览器app, eg: https://expo.io	| ✅	| ✅

### 打开你的应用程序的链接

React Native中没有锚标记，所以我们不能写`<a href="https://expo.io">`。相反，我们必须使用`Linking.openURL`。

```js
import { Linking } from 'react-native';

Linking.openURL('https://expo.io');
```

通常，如果没有用户的请求，您就不会打开URL，这是一个简单的`Anchor`组件示例，当它被按下时，它将打开一个URL。

```js
import { Linking, Text } from 'react-native';

export default class Anchor extends React.Component {
  _handlePress = () => {
    Linking.openURL(this.props.href);
    this.props.onPress && this.props.onPress();
  };

  render() {
    return (
      <Text {...this.props} onPress={this._handlePress}>
        {this.props.children}
      </Text>
    );
  }
}

// <Anchor href="https://google.com">Go to Google</Anchor>
// <Anchor href="mailto://support@expo.io">Go to Google</Anchor>
```

### 使用`Expo.WebBrowser`，而不是链接打开web链接

下面的例子说明了打开一个web链接使用`Expo.WebBrowser.openBrowserAsync`和`React Native’s Linking.openURL`的的区别。
通常，WebBrowser是一个更好的选择，因为它是你的应用中的一个模式，用户可以很容易地关闭它并返回到你的应用。

```
import React, { Component } from 'react';
import { Button, Linking, View, StyleSheet } from 'react-native';
import { Constants, WebBrowser } from 'expo';

export default class App extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Button
          title="Open URL with ReactNative.Linking"
          onPress={this._handleOpenWithLinking}
          style={styles.button}
        />
        <Button
          title="Open URL with Expo.WebBrowser"
          onPress={this._handleOpenWithWebBrowser}
          style={styles.button}
        />
      </View>
    );
  }
```

### 打开其他应用程序的链接

如果你知道另一款应用的定制方案，你可以链接到它。一些服务提供了深度链接的文档，例如[Lyft深度链接文档](https://developer.lyft.com/v1/docs/deeplinking)描述了如何直接链接到特定的位置和目的地:

```
lyft://ridetype?id=lyft&pickup[latitude]=37.764728&pickup[longitude]=-122.422999&destination[latitude]=37.7763592&destination[longitude]=-122.4242038
```

用户可能没有安装Lyft应用，在这种情况下，你可能想打开应用/Play商店，或者让他们知道他们需要先安装。我们建议在这些情况下使用库[react-native-app-link](https://github.com/fiber-god/react-native-app-link)。

在iOS上，`Linking.canOpenUrl`需要额外的配置来查询其他应用的链接方案。你可以在你的`app.json`中使用`ios.infoPlist`键来指定应用程序需要查询的方案列表。例如:

```json
"infoPlist": {
  "LSApplicationQueriesSchemes": ["lyft"]
}
```

如果你没有指定这个列表，不管设备是否安装了这个应用，`Linking.canOpenUrl`可能会返回false。注意，这种配置只能在独立的应用程序中进行测试，因为它需要在Expo客户端测试时不应用的本地更改。

## 链接到你的应用

### 在Expo客户端

在继续之前，花点时间学习如何在Expo客户端链接到你的应用程序是值得的。Expo客户端使用`exp:// scheme`，但如果我们链接到`exp://`没有任何地址，它将会打开应用到主界面。

在开发中,应用程序将会住在一个url像`exp://wg-qka.community.app.exp.direct:80`。当它被部署时，它将会出现在一个URL中，比如`exp://exp.host/@community/with-webbrowser-redirect`。如果你创建了一个网站，其中有一个链接，比如`<a href="exp://expo.io/@community/with-webbrowser-redirect">Open my project</a>`。打开我的项目，然后在你的设备上打开这个网站，点击这个链接，它就会在Expo客户端打开你的应用程序。你可以通过使用`Linking.openURL`从另一个应用中链接到它。

### 在一个独立的应用程序

要链接到你的独立应用，你需要为你的应用程序指定一个方案，你可以在你的`app.json`中注册一个方案，通过在`scheme key`下面添加一个字符串:

```json
{
  "expo": {
    "scheme": "myapp"
  }
}
```

一旦你建立了独立的应用，并将其安装到你的设备上，你就可以用`myapp://`的链接打开它。

### `Expo.Constants.linkingUri`

为了省去插入大量条件的麻烦，基于您所在的环境和硬编码url，我们提供了`linkingUri`常量。当您想要提供一个需要重定向到应用程序的url的服务时，您可以使用它，它将解决以下问题:

- 在Expo客户端发布的应用:`exp://exp.host/@community/with-webbrowser-redirect/+`
- 独立出版的应用:`myapp://+`
- 开发: `exp://wg-qka.community.app.exp.direct:80/+`

您将注意到，在每个URL的末尾有一个`/+` — 任何在`/+`之后被应用程序用来接收数据的东西，我们将在下一节中讨论。

> 注意:目前有一个已知的问题，在Android独立版本中没有出现+。我们希望在不久的版本中解决这个问题！

### 在你的应用中处理链接

有两种方法可以处理打开应用程序的url。

1. 如果应用已经打开，应用就会被启动，链接事件就会被触发

  您可以使用Linking.addEventListener('url', callback)来处理这些事件。

1. 如果这个应用程序还没有打开，它就会打开，url会作为initialURL传递

  您可以使用`Linking.getInitialURL`来处理这些事件——如果有的话，它返回一个解析到url的Promise。

请参见下面的示例，以查看这些示例。  

### 通过URL将数据传递给应用程序

如果我想将一些数据传递到我的应用程序中，我可以在常量`Constants.linkingUri`的末尾添加一个查询字符串。然后，您可以使用类似于[qs](https://www.npmjs.com/package/qs)的东西来解析查询字符串。

当[处理用于打开/前景应用的URL](https://docs.expo.io/versions/latest/guides/linking.html#handling-urls-in-your-app)时，它看起来是这样的:

```js
_handleUrl = (url) => {
  this.setState({ url });
  let queryString = url.replace(Constants.linkingUri, '');
  if (queryString) {
    let data = qs.parse(queryString);
    alert(`Linked to app with data: ${JSON.stringify(data)}`);
  }
}
```

如果你打开一个URL，像`${Constants.linkingUri}?hello=world&goodbye=now`，这将会alert `{hello: 'world', goodbye: 'now'}`。

### 例子:从网页浏览器链接到你的应用

示例项目[examples/with-webbrowser-redirect](https://github.com/expo/examples/tree/master/with-webbrowser-redirect)演示了如何从WebBrowser重定向并从查询字符串中获取数据。在[Expo上试试](https://expo.io/@community/with-webbrowser-redirect)吧。

### 示例:使用链接进行身份验证

链接到你的应用的一个常见的用例是在打开一个网页浏览器后重定向回你的应用。例如，您可以在屏幕上打开一个web浏览器会话，当用户成功登录时，您可以使用该方案将您的网站重定向到您的应用程序，并将身份验证令牌和其他数据附加到URL。

> 注意:如果尝试使用链接。openURL打开web浏览器进行身份验证，然后你的应用可能会被苹果拒绝，原因是糟糕或令人困惑的用户体验。`WebBrowser.openBrowserAsync`打开了一个模态的浏览器窗口，外观和感觉都很好，苹果公司也批准了。

要查看使用web浏览器进行身份验证的完整示例，请参见[examples/with-facebook-auth](https://github.com/expo/examples/tree/master/with-facebook-auth)。目前，Facebook认证要求你部署一个小的webserver来重定向到你的应用程序(正如在这个例子中所描述的那样)，因为Facebook不允许你重定向到自定义的方案，而Expo正在为解决这个问题而努力。在Expo上试一试。

另一个使用`WebBrowser`进行身份验证的例子可以在[expo/auth0-example](https://github.com/expo/auth0-example)中找到。

## 当不使用深层链接时

这是在你的应用中建立深度链接的最简单方法，因为它只需要很少的配置。

主要的问题是，如果用户没有安装你的应用，并按照它的自定义模式链接到你的应用程序，那么他们的操作系统将会显示页面不能打开，但不能提供更多的信息。这不是一次伟大的经历。在浏览器中没有办法解决这个问题。

此外，许多消息应用程序不使用自定义方案自动生成url——例如，`exp://exp.host/@community/native-component-list `可能只是在浏览器中显示为纯文本，而不是作为链接([exp://exp.host/@community/native- componentlist](exp://exp.host/@community/native-component-list))。

---

# Expo & "Create React Native App"

[Create React Native App](https://facebook.github.io/react-native/blog/2017/03/13/introducing-create-react-native-app.html)允许您构建一个没有任何构建配置的React Native app。这对您来说可能听起来很熟悉，因为世博会也会这样做——当您用XDE或exp创建一个项目时，您不必处理Xcode或Android Studio配置文件，它只是工作。本指南旨在概述博览会与CRNA (create-react-native-app)之间的一些关键区别。



---

# Expo是如何工作的

---

# 升级 Expo

---
