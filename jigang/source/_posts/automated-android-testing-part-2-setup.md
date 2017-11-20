---
title: Android 自动化测试 - 第2部分 安装
date: 2017-11-17 18:34:14
tags:
- Android
- 自动化测试
categories:
- Android自动化测试

---

<img src="http://szimg.mukewang.com/5850bc4500015ecd05400300-360-202.jpg" width="88%" height="320" align=center/>

在本系列文章中，我们将介绍自动化的Android测试的介绍。
在这个自动化测试系列的第1部分中，我们讨论了为什么要编写测试、测试文件夹所在的位置以及在Android中使用的不同类型的测试。

在本文中，我们将介绍Android应用程序的典型结构和设置，以便进行测试。我将在本系列中从头开始创建一个简单的应用程序，并逐步完成我的思想过程中的每一步。
我们将要创建的应用是一个简单的应用，它可以为用户搜索Github的API。
下面是我们将要创建的一个粗略的模型:


![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/SampleApp-Android_Automated_Testing_.png?w=360&ssl=1)

<!-- more -->

## 开始使用一款新应用

1. 打开Android Studio，选择“Start a new Android Project”。
1. 把这个项目叫做“Gus”(Github User Search)和公司域名“riggaroo.co.za”。
这将创建一个包名 —— za.co.riggaroo.gus。
单击next。
![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Step1-AutomatedAndroidTestProject.png?resize=768%2C502&ssl=1)
1. 接下来，选择您希望支持的Android版本(我一般不会低于API 16。如果您选择了API 16和以上的API，您将覆盖95.2%的用户。
选择API版本16并单击“Next”。
![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Step2-AutomatedAndroid_Test_Project.png?resize=768%2C502&ssl=1)
1. 选择“Empty Activity”并单击“Next”。
![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Step3-AutomatedTestProjectAndroid.png?resize=768%2C500&ssl=1)
1. 将activity的名称更改为UserSearchActivity和布局文件到activity_user_search。
单击“Finish”。
![](https://i1.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Step4-UserActivityName.png?resize=768%2C498&ssl=1)
1. 如果你运行这个应用程序，你应该看到如下的空白activity。
![](https://i2.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Step4-BlankApp.png?ssl=1)

## 添加测试依赖

导航到您的应用程序`build.gradle`文件，并添加以下依赖项。
需要添加的测试依赖包括[Espresso](https://developer.android.com/topic/libraries/testing-support-library/packages.html)、[Mockito](http://mockito.org/)、[PowerMock](https://github.com/jayway/powermock)和[Hamcrest](http://hamcrest.org)。
[Retrofit](http://square.github.io/retrofit/)、[OkHttp](http://square.github.io/okhttp/)、[RxJava](https://github.com/ReactiveX/RxJava)和[RxAndroid](https://github.com/ReactiveX/RxAndroid)也将被添加，这样我们就可以进行有效的网络连接，并实现更简洁的代码。

```build.gradle
compile 'com.squareup.okhttp3:logging-interceptor:3.4.0-RC1'
    compile 'com.android.support:appcompat-v7:24.1.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.0-alpha4'

    //Retrofit, RxJava and OkHttp.
    compile 'com.squareup.retrofit2:retrofit:2.1.0'
    compile 'com.squareup.retrofit2:converter-gson:2.1.0'
    compile 'com.squareup.okhttp3:okhttp:3.4.0-RC1'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.1.0'
    compile 'io.reactivex:rxandroid:1.2.1'
    compile 'io.reactivex:rxjava:1.1.6'
    compile 'com.squareup.picasso:picasso:2.5.2'

    //Dependencies for JUNit and unit tests.
    testCompile "junit:junit:4.12"
    testCompile "org.mockito:mockito-all:1.10.19"
    testCompile "org.hamcrest:hamcrest-all:1.3"
    testCompile("org.powermock:powermock-module-junit4:1.6.2")
    testCompile("org.powermock:powermock-api-mockito:1.6.2")
    testCompile 'com.squareup.okhttp3:mockwebserver:3.4.0-RC1'

    //Dependencies for Espresso
    androidTestCompile 'com.android.support:appcompat-v7:24.1.1'
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    androidTestCompile("com.android.support.test:runner:0.5") {
        exclude module: 'support-annotations'
        exclude module: 'support-v4'
    }
    androidTestCompile("com.android.support.test:rules:0.5") {
        exclude module: 'support-annotations'
        exclude module: 'support-v4'
    }
    androidTestCompile("com.android.support.test.espresso:espresso-intents:2.2.2") {
        exclude module: 'recyclerview-v7'
        exclude module: 'support-annotations'
        exclude module: 'support-v4'
    }
    androidTestCompile('com.android.support.test.espresso:espresso-contrib:2.2.1') {
        exclude module: 'recyclerview-v7'
        exclude module: 'support-annotations'
        exclude module: 'support-v4'

    }
```

`testCompile`是用于单元测试的配置(位于`src/test`)，而`androidTestCompile`用于交互测试(位于`src/androidTest`)。
这两种类型的测试之间的差异可以在本系列的第1部分中找到。

## 使用Gradle Build Flavors使能Mocking

为了方便快捷地对UI进行测试，我们不会对Github提供的APIs进行访问。
我们将模拟响应并模拟不同网络的响应。
有几种不同的方法来实现这一点。
我要演示的方法是使用[Gradle product flavors](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Product-flavors)。

Flavors允许你构建不同版本的应用程序，有一些源代码差异或资源差异。
例如，如果你想要制作一个免费版本的应用程序和一个带有更多功能的付费版本，那么使用product flavors将是实现这一目标的好方法。

在这个例子中，我们将创建一个“production”和“mock”的flavor。
我们还可以添加一个“staging”的flavor，如果我们有一个舞台环境，我们想让我们的应用指向它。
这将允许我们在一个设备上安装一个production版本的应用程序，同时也可以在mock版本上安装。

1. 为了使用`productFlavors`，导航到你的app `build.gradle`文件，并将以下内容添加到`android{ }`部分。
下面的代码意味着对于`mock`变体，`applicationId`将不同，这意味着我可以同时在一个设备上安装。
对于`prod`版本，`applicationId`将从`defaultConfig`设置中获取。
```build.gradle
productFlavors {
        prod {
 
        }
        mock {
            applicationId "za.co.riggaroo.gus.mock"
        }
}
```
1. 调用Gradle Sync，然后在IDE的左边，您应该看到一个“Build Variants”选项卡。
打开它，你可以看到你的应用的不同风格。
![](https://i1.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Gradle_Build_Variants_Android_Studio.png?resize=768%2C367&ssl=1)

如果您选择了一个不同的变体，当您单击run时，将部署到您的设备上的构建将使用该变量的源码集和`applicationId`。

## 运行单元测试

为了运行项目中存在的默认单元测试(ExampleUnitTest)，您可以通过几种方式来完成:

1. 在Android Studio中，导航到app/src/test/java文件夹，右键单击。
点击“Run Tests”。
![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Running_Unit_Tests_with_JUnit_in_Android_Studio.png?resize=768%2C994&ssl=1)
1. 使用Gradle: 在终端你可以运行 `./gradlew check`

## 运行交互测试(需要设备或模拟器)

为了运行运行项目中默认交互测试(ExampleInstrumentationTest),你可以在以下方面:

1.  在Android Studio中，导航到app/src/androidTest/java文件夹，右键单击。
单击“Run All Tests”。
![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/Run_Connected_tests_android_studio.png?resize=768%2C910&ssl=1)
1. 使用Gradle: 在终端你可以运行 `./gradlew connectedAndroidTest`

## 如何构造您的代码以实现简单的测试

为了方便测试，我将使用依赖项注入。
您可以在没有任何框架的情况下实现这一点，但是很多人都提倡使用[Dagger 2](http://google.github.io/dagger/)来实现它。
这篇博客文章不会使用Dagger。

在以下文件夹结构中创建这些类:

![](https://i0.wp.com/riggaroo.co.za/wp-content/uploads/2016/07/SimpleStructureOfAndroidApp.png?resize=766%2C1024&ssl=1)

我有4个顶级文件夹用于应用程序的不同部分。
这可能会让你的应用变得更复杂。
这些是我喜欢创建的基本文件夹:

* `presentation` —— 在这个文件夹中，我通过特性创建子文件夹和组功能。
在此情况，我创建了一个名为`search`的文件夹，它将保存搜索屏幕的view和presenter。
它还将包含用于搜索屏幕的适配器和任何其他视图相关代码。
* `data` —— 这将包含从Github API获取数据的存储库(`repositories`)。
* `model` - 包含将在表示层上使用的模型，以及来自于服务调用的模型。
* `injection` —— 将用于依赖注入的类。

我们已经介绍了示例应用程序的设置、如何运行不同类型的测试以及我们将遵循的文件夹结构。
这让我们有了一条很好的途径来确保我们可以为我们的应用程序写测试，如果你想看看这篇博文的结果，就看看这篇博文对应的[完整代码](https://github.com/riggaroo/GithubUsersSearchApp/tree/testing-tutorial-part2-complete)吧。

本系列的下一篇博文将详细介绍如何通过测试实现该特性。