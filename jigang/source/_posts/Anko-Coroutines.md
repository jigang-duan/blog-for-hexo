---
title: Anko 协程
date: 2017-11-03 13:14:08
tags:
- Android
- Anko
- Kotlin
categories:
- Anko

---

![anko](https://github.com/Kotlin/anko/blob/master/doc/logo.png?raw=true)

## 目录 ##

* [在你的项目中使用Anko协程](#using_anko_coroutines)
* [Listener助手](#listener_helper)
* [asReference()](#asreference)
* [bg()](#bg)

<!-- more -->

## <a name="using_anko_coroutines"></a>在你的项目中使用Anko协程

添加`anko-coroutines` 到你的`build.gradle`：

```
dependencies {
    compile "org.jetbrains.anko:anko-coroutines:$anko_version"
}
```

## <a name="listener_helper"></a>Listener助手

## <a name="asreference"></a>asReference()

如果您的异步API不支持取消，你的协程可能会被无限期挂起。
由于协程持有对被捕获对象的强引用，捕获的Activity或Fragment实例可能会导致内存泄漏。

在这种情况下使用asReference()而不是直接捕获:

```
suspend fun getData(): Data { ... }

class MyActivity : Activity() {
    fun loadAndShowData() {
	// Ref<T> uses the WeakReference under the hood
	val ref: Ref<MyActivity> = this.asReference()

	async(UI) {
	    val data = getData()
			
	    // Use ref() instead of this@MyActivity
	    ref().showData()
	}
    }

    fun showData(data: Data) { ... }
}
```

## <a name="bg"></a>bg()

您可以使用`bg()`轻松地在后台线程上执行您的代码:

```
fun getData(): Data { ... }
fun showData(data: Data) { ... }

async(UI) {
    val data: Deferred<Data> = bg {
        // 运行在后台
        getData()
    }

    // 该代码在UI线程上执行
    showData(data.await())
}
```