---
title: 协程响应式流指南
date: 2017-11-07 15:08:04
tags:
- kotlinx.coroutines
- Android
- Kotlin
categories:
- Kotlin

---

![kotlin](http://images2015.cnblogs.com/blog/831804/201603/831804-20160323231701386-1892583268.png)


本指南解释了Kotlin 协程和响应式流之间的关键区别，并展示了它们如何可以一起使用以获得更大的好处。
预先熟悉基本的协程概念，这些概念在[kotlinx.coroutines指南](/2017/11/06/GuideToKotlinxCoroutinesByExample/)，不需要的，但是是一个很大的优势。
如果您熟悉响应式流，您可能会发现这个指南更好地介绍了协程的世界。

在`kotlinx.coroutines`项目中有几个与响应式流相关的模块:

* [kotlinx-coroutines-reactive](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-reactive) -- [Reactive Streams](http://www.reactive-streams.org/) utilities
* [kotlinx-coroutines-reactor](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-reactor) -- [Reactor](https://projectreactor.io/) utilities
* [kotlinx-coroutines-rx1](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx1) -- [RxJava 1.x](https://github.com/ReactiveX/RxJava/tree/1.x) utilities
* [kotlinx-coroutines-rx2](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2) -- [RxJava 2.x](https://github.com/ReactiveX/RxJava) utilities

该指南主要基于[Reactive Streams](http://www.reactive-streams.org/)规范，并使用它的Publisher接口与一些基于[RxJava 2.x](https://github.com/ReactiveX/RxJava)的例子，后者实现了响应式流规范。

欢迎您从GitHub到您的工作站克隆kotlinx.coroutines项目，以运行所有提供的示例。
他们是包含在reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide项目的目录。

<!-- more -->

## 目录 ##

* [响应式流与通道之间的差异](#Differences_between_reactive_streams_and_channels)
	* [迭代的基本知识](#basics_of_iteration)
	* [订阅和取消](#subscription_and_cancellation)
	* [背压](#backpressure)
	* [Rx主题和BroadcastChannel](#Rx_Subject_vs_BroadcastChannel)
* [操作符](#Operators)
	* [Range](#Range)
	* [融合filter-map混合](#Fused_filter_map_hybrid)
	* [Take until](#Take_until)
	* [Merge](#Merge)
* [协程上下文](#Coroutine_context)
	* [Rx中的线程](#Threads_with_Rx)
	* [协程中的线程](#Threads_with_coroutines)
	* [Rx observeOn](#Rx_observeOn)
	* [协程上下文来统治它们](#Coroutine_context_to_rule_them_all)
	* [无限制的上下文](#Unconfined_context)

----

## <a name="Differences_between_reactive_streams_and_channels"></a>响应式流与通道之间的差异

本节概述了响应式流和基于协程的通道之间的关键区别。

### <a name="basics_of_iteration"></a>迭代的基本知识

与以下的响应式流类，通道是一个类似的概念:

* Reactive stream [Publisher](https://github.com/reactive-streams/reactive-streams-jvm/blob/master/api/src/main/java/org/reactivestreams/Publisher.java);
* Rx Java 1.x [Observable](http://reactivex.io/RxJava/javadoc/rx/Observable.html);
* Rx Java 2.x [Flowable](http://reactivex.io/RxJava/2.x/javadoc/), 它实现了发布者(Publisher).

它们都描述了一个异步的元素流(也就是Rx中的items)，要么是无限的，要么是有限的，它们都支持背压。

然而，使用Rx术语，通道始终代表一个items热流。
元素正被生产者的协程发送到通道，并由消费者协程接收。
每个接收调用都会从通道中消费一个元素。
让我们用下面的例子来说明一下:

```Kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // 创建一个通道，从1到3生成数字，其中有200毫秒的延迟
    val source = produce<Int>(coroutineContext) {
        println("Begin") // 这行输出标示开始
        for (x in 1..3) {
            delay(200) // 等待200毫秒
            send(x) // 发送x到通道
        }
    }
    // 从source中打印元素
    println("Elements:")
    source.consumeEach { // 使用它的元素
        println(it)
    }
    // 再次从source打印元素
    println("Again:")
    source.consumeEach { // 使用它的元素
        println(it)
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-01.kt)得到完整的代码

此代码产生以下输出:

```
Elements:
Begin
1
2
3
Again:
```

请注意，“Begin”行仅打印一次，因为当[produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/produce.html)协程构建器执行时，启动一个协程来生成一个元素流。
所有生成的元素都使用[ReceiveChannel.consumeEach](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/consume-each.html)扩展函数来接收。
无法再从这个通道接收元素。
当producer协程结束时，通道关闭，试图从它接收的尝试不能得到任何东西。

让我们重写这个代码，使用来自kotlinx-coroutines-reactive模块的publish协程构建器，而不是来自kotlinx-coroutines-core模块的produce。
代码保持不变，但source用于ReceiveChannel类型，它现在有了reactive流Publisher类型。

```Kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // create a publisher that produces numbers from 1 to 3 with 200ms delays between them
    val source = publish<Int>(coroutineContext) {
    //           ^^^^^^^  <---  不同于前面的例子
        println("Begin") // mark the beginning of this coroutine in output
        for (x in 1..3) {
            delay(200) // wait for 200ms
            send(x) // send number x to the channel
        }
    }
    // print elements from the source
    println("Elements:")
    source.consumeEach { // consume elements from it
        println(it)
    }
    // print elements from the source AGAIN
    println("Again:")
    source.consumeEach { // consume elements from it
        println(it)
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-02.kt)得到完整的代码

现在该代码的输出更改为:

```
Elements:
Begin
1
2
3
Again:
Begin
1
2
3
```

这个例子强调了响应式流和通道之间的关键区别。
响应式流是一个高阶函数概念。
虽然通道是一个元素流，但响应式流定义了一个关于元素流如何产生的菜谱。
它成为订阅元素的实际流。
每个订阅者可以接收相同或不同的元素流，这取决于发布者的相应实现方式。

在上面的示例中使用的[publish](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-reactive/kotlinx.coroutines.experimental.reactive/publish.html)协程构建器，在每个订阅上启动一个新的协程。
每个[Publisher.consumeEach](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-reactive/kotlinx.coroutines.experimental.reactive/org.reactivestreams.-publisher/consume-each.html)调用都创建一个新的订阅。
在这段代码中，我们有两个，这就是为什么我们看到“Begin”打印两次

在Rx术语中，这被称为冷发布者。
许多标准的Rx操作员也会产生冷流。
我们可以通过协程对它们进行迭代，每个订阅都产生相同的元素流。

> 注意，我们可以通过使用Rx publish操作符和connect方法来复制我们在通道中看到的相同行为。

### <a name="subscription_and_cancellation"></a>订阅和取消

上一节中的示例使用`source.consumeEach { ... }`片段打开一个订阅，并从其中接收所有元素。
如果我们需要更多的控制如何处理从通道接收的元素，我们可以使用[Publisher.openSubscription](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-reactive/kotlinx.coroutines.experimental.reactive/org.reactivestreams.-publisher/open-subscription.html)，如下例所示:

```Kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val source = Flowable.range(1, 5) // a range of five numbers
        .doOnSubscribe { println("OnSubscribe") } // provide some insight
        .doFinally { println("Finally") }         // ... into what's going on
    var cnt = 0 
    source.openSubscription().use { channel -> // open channel to the source
        for (x in channel) { // iterate over the channel to receive elements from it
            println(x)
            if (++cnt >= 3) break // break when 3 elements are printed
        }
        // `use` will close the channel when this block of code is complete
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-03.kt)得到完整的代码

它产生如下输出:

```
OnSubscribe
1
2
3
Finally
```

在显式`openSubscription`的情况下，我们应该关闭相应的订阅源以取消订阅。
然而，这段代码不是直接调用[close](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-subscription-receive-channel/close.html)，而是依赖于Kotlin的标准库中的[use](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/use.html)函数。
安装的[doFinally](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Flowable.html#doFinally(io.reactivex.functions.Action)) listener打印“Finally”来确认订阅实际上是关闭的。

如果对发布者发出的所有项进行迭代，我们不需要使用显式的关闭，因为它是由`consumeEach`自动关闭的:

```Kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val source = Flowable.range(1, 5) // a range of five numbers
        .doOnSubscribe { println("OnSubscribe") } // provide some insight
        .doFinally { println("Finally") }         // ... into what's going on
    // iterate over the source fully
    source.consumeEach { println(it) }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-04.kt)得到完整的代码

它产生如下输出:

```
OnSubscribe
1
2
3
4
Finally
5
```

注意，在最后一个元素“5”之前，为何将“Finally”打印出来。
这是因为我们在这个例子中的主函数是一个从runBlocking 协程构建器启动的协程。
我们的主协程使用source.consumeEach {…}表达式。
主协程在等待source发出一个item时挂起。
当最后一项被Flowable.range(1, 5)发送时，它会恢复主协程，它被分派到主线程上，在稍后的时间点上打印最后一个元素，而source完成和打印“Finally”。

### <a name="backpressure"></a>背压

背压是响应式流最有趣、最复杂的方面之一。
协程可以挂起，它们为处理背压提供了一个自然的答案。

在Rx Java 2.x中，一个有背压能力类称为Flowable。
在下面的示例中，我们使用来自 kotlinx-coroutines-rx2模块的rxFlowable协程构建器定义了一个可流，它从1到3发送了3个整数。
它在调用挂起send函数之前将消息打印到输出，这样我们就可以研究它是如何操作的。

这些整数是在主线程的上下文中生成的，但是，使用Rx observeOn操作符的另一个线程，使用的缓冲区大小为1。
订阅者是缓慢的。
每个项目需要500 ms来处理，这是使用Thread.sleep模拟的。

```Kotlin
fun main(args: Array<String>) = runBlocking<Unit> { 
    // coroutine -- fast producer of elements in the context of the main thread
    val source = rxFlowable(coroutineContext) {
        for (x in 1..3) {
            send(x) // this is a suspending function
            println("Sent $x") // print after successfully sent item
        }
    }
    // subscribe on another thread with a slow subscriber using Rx
    source
        .observeOn(Schedulers.io(), false, 1) // specify buffer size of 1 item
        .doOnComplete { println("Complete") }
        .subscribe { x ->
            Thread.sleep(500) // 500ms to process each item
            println("Processed $x")
        }
    delay(2000) // suspend the main thread for a few seconds
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-05.kt)得到完整的代码

这段代码的输出很好地说明了背压如何与协程一起工作:

```
Sent 1
Processed 1
Sent 2
Processed 2
Sent 3
Processed 3
Complete
```

我们在这里看到了生产者协程如何将第一个元素放在缓冲区中，并在试图发送另一个元素时挂起。
只有在消费者处理第一个项目后，生产者才会发送第二个和恢复等。

### <a name="Rx_Subject_vs_BroadcastChannel"></a>Rx主题和BroadcastChannel

RxJava有一个[Subject](https://github.com/ReactiveX/RxJava/wiki/Subject)概念，它是一个可以有效地向所有订阅者广播元素的对象。
协程世界中的匹配概念称为[BroadcastChannel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-broadcast-channel/index.html)。
在Rx中，有很多subjects使用[BehaviorSubject](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/subjects/BehaviorSubject.html)作为管理状态:

```Kotlin
fun main(args: Array<String>) {
    val subject = BehaviorSubject.create<String>()
    subject.onNext("one")
    subject.onNext("two") // updates the state of BehaviorSubject, "one" value is lost
    // now subscribe to this subject and print everything
    subject.subscribe(System.out::println)
    subject.onNext("three")
    subject.onNext("four")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-06.kt)得到完整的代码

此代码打印订阅主题的当前状态及其所有进一步更新:

```
two
three
four
``` 

你可以像任何其他的响应式流一样订阅一个协程的主题:

```Kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val subject = BehaviorSubject.create<String>()
    subject.onNext("one")
    subject.onNext("two")
    // now launch a coroutine to print everything
    launch(Unconfined) { // launch coroutine in unconfined context
        subject.consumeEach { println(it) }
    }
    subject.onNext("three")
    subject.onNext("four")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-07.kt)得到完整的代码

结果是一样的:

```
two
three
four
```

在这里，我们使用非受限的协程上下文，以与Rx的订阅相同的行为来启动消费协程。
它的基本意思是，启动的协程将会立即在相同的线程中被执行。
上下文在[单独的部分](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/coroutines-guide-reactive.md#coroutine-context)中包含了更多的细节。

协程的优势在于它很容易获得单线程UI更新的合并行为。
一个典型的UI应用程序不需要对每个状态变化做出反应。
只有最近的状态是相关的。
对于应用程序状态的连续更新序列必须在UI线程空闲时才在UI中反映一次。
下面的例子，我们将通过在主线程的上下文中启动消耗的协程来模拟它，使用yield函数来模拟更新序列中的一个中断，并释放主线程:

```Kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val subject = BehaviorSubject.create<String>()
    subject.onNext("one")
    subject.onNext("two")
    // 现在，启动一个协程来打印最新的更新
    launch(coroutineContext) { // 使用主线程的上下文来进行相关的处理
        subject.consumeEach { println(it) }
    }
    subject.onNext("three")
    subject.onNext("four")
    yield() // yield主线程带到启动的协程 <-- -这里
    subject.onComplete() // 现在完成subject的序列也可以取消消费者   
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-08.kt)得到完整的代码

现在协程流程(打印)只是最新的更新:

```
four
```

相应的行为在一个纯粹的协同程序的世界由ConflatedBroadcastChannel实现提供相同的逻辑上的协同程序直接渠道,不经过这座桥响应式流:

```Kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val broadcast = ConflatedBroadcastChannel<String>()
    broadcast.offer("one")
    broadcast.offer("two")
    // now launch a coroutine to print the most recent update
    launch(coroutineContext) { // use the context of the main thread for a coroutine
        broadcast.consumeEach { println(it) }
    }
    broadcast.offer("three")
    broadcast.offer("four")
    yield() // yield the main thread to the launched coroutine
    broadcast.close() // now close broadcast channel to cancel consumer, too    
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/kotlinx-coroutines-rx2/src/test/kotlin/guide/example-reactive-basic-09.kt)得到完整的代码

它产生与上一个基于`BehaviorSubject`的示例相同的输出:

```
four
```

BroadcastChannel的另一个实现是ArrayBroadcastChannel。
它向每个订阅服务器发送每个事件，因为相应的订阅是打开的。
它对应于Rx中的PublishSubject。
在ArrayBroadcastChannel的构造函数中，缓冲区的容量控制在发送方挂起等待接收方接收这些元素之前发送的元素的数量。

## <a name="Operators"></a>操作符
### <a name="Range"></a>Range
### <a name="Fused_filter_map_hybrid"></a>融合filter-map混合
### <a name="Take_until"></a>Take until
### <a name="Merge"></a>Merge
## <a name="Coroutine_context"></a>协程上下文
### <a name="Threads_with_Rx"></a>Rx中的线程
### <a name="Threads_with_coroutines"></a>协程中的线程
### <a name="Rx_observeOn"></a>Rx observeOn
### <a name="Coroutine_context_to_rule_them_all"></a>协程上下文来统治它们
### <a name="Unconfined_context"></a>无限制的上下文