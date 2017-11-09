---
title: Android架构组件 添加到你的项目
date: 2017-11-08 16:24:19
tags:
- Android
- 架构设计
categories:
- Android架构组件

---


![arch-hero.png](https://developer.android.google.cn/topic/images/arch/hero/arch-hero.png)


架构组件可以从Google的Maven存储库中获得。
要使用这些步骤，请遵循以下步骤:

### 添加Google Maven存储库

在默认情况下，Android Studio项目不会被配置为访问这个存储库。

要将它添加到您的项目中，为`您的项目`打开 `build.gradle `文件(而不是您的应用程序或模块)，如下所示:

```gradle
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
```

### 添加架构组件

打开您的应用程序或模块build.gradle文件，并添加依赖项:

* Lifecycles:
	* `implementation "android.arch.lifecycle:runtime:1.0.3"` //使用着lifecycle:extensions or lifecycle:common-java8 就不需
	* `annotationProcessor "android.arch.lifecycle:compiler:1.0.0"` //如果您使用common-java8中的DefaultLifecycleObserver，则不需要
	* 对于Lifecycles的Java8 Lanaguage支持，添加:
		* `implementation "android.arch.lifecycle:common-java8:1.0.0"`

* LiveData和ViewModel：
	* `implementation "android.arch.lifecycle:extensions:1.0.0"`
	* 要在测试中控制LiveData后台线程，请添加:
		* `testImplementation "android.arch.core:core-testing:1.0.0"`
	* 要使用ReactiveStreams  API的LiveData，请添加:
		* `implementation "android.arch.lifecycle:reactivestreams:1.0.0"`

*  Room
	*  `implementation "android.arch.persistence.room:runtime:1.0.0"`
	*  `annotationProcessor "android.arch.persistence.room:compiler:1.0.0"`
	*  对于测试Room迁移，添加:
		*  `testImplementation "android.arch.persistence.room:testing:1.0.0"`
	*  对Room  RxJava的支持，添加:
		*  `implementation "android.arch.persistence.room:rxjava2:1.0.0"`

*  Paging
	*  `implementation "android.arch.paging:runtime:1.0.0-alpha3"`
	

要了解更多信息，请参阅[添加构建依赖项](https://developer.android.google.cn/studio/build/dependencies.html)。