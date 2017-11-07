---
title: kotlinx.coroutines入门指南示例
date: 2017-11-06 19:01:27
tags:
- kotlinx.coroutines
- Android
- Kotlin
categories:
- Kotlin

---

![kotlin](http://images2015.cnblogs.com/blog/831804/201603/831804-20160323231701386-1892583268.png)

这是一个关于kotlinx.coroutines核心特性的简短指南，有一系列的例子。

## 介绍和设置 ##

作为一种语言，Kotlin在其标准库中只提供了最低限度的低级api，以使各种其他库能够利用这些库。
与其他具有类似功能的语言不同，`async`和`await`在Kotlin中不是关键字，甚至都不是它的标准库的一部分。

kotlinx.coroutines是一个如此丰富的库。
它包含了许多高级的可支持内核的原语，包括`async`和`await`。
您需要添加对`kotlinx-coroutines-core`模块的依赖，[这是](https://github.com/Kotlin/kotlinx.coroutines/blob/master/README.md#using-in-your-projects)在您的项目中使用本指南中的基础。

## 目录 ##

* [协程基础](#coroutine_basics)
	* [你的第一个协程](#your_first_coroutine)
	* [桥接阻塞和非阻塞的世界](#bridging_blocking_nonblocking_worlds)
	* [等待job](#waiting_for_job)
	* [提取函数重构](#extract_function_refactoring)
	* [协程是轻量级的](#coroutines_are_light_weight)
	* [协程就像守护线程](#coroutines_like_daemon_threads)
* [取消和超时](#cancellation_timeouts)
	* [取消协程执行](#cancelling_coroutine_execution)
	* [取消是需要协作的](#cancellation_cooperative)
	* [使计算代码可取消](#making_computation_code_cancellable)
	* [在finally关闭资源](#closing_resources_with_finally)
	* [运行不可取消块](#run_noncancellable_block)
	* [超时](#timeout)
* [组织挂起函数](#composing_suspending_functions)
	* [默认顺序](#sequential_default)
	* [使用async并发](#concurrent_using_async)
	* [懒启动async](#lazily_started_async)
	* [异步式函数](#async_style_functions)
* [协程context和分派器](#coroutine_context_dispatchers)
	* [分派器和线程](#dispatchers_threads)
	* [非限制 vs 限制 分派器](#unconfined_vs_confined_dispatcher)
	* [调试协程和线程](#debugging_coroutines_threads)
	* [线程之间跳转](#jumping_between_threads)
	* [context中的job](#job_context)
	* [子协程](#children_of_a_coroutine)
	* [结合contexts](#combining_contexts)
	* [父协程责任](#parental_responsibilities)
	* [命名协程的调试](#naming_coroutines_debugging)
	* [取消显式工作](#cancellation_via_explicit_job)
* [通道](#channels)
	* [通道基础知识](#channel_basics)
	* [通道的关闭和迭代](#closing_iteration_over_channels)
	* [构建通道生成器](#building_channel_producers)
	* [管道](#pipelines)
	* [用管道求素数](#prime_numbers_pipeline)
	* [扇出](#Fan-out)
	* [扇入](#Fan-in)
	* [缓冲通道](#buffered_channels)
	* [通道是公平的](#channels_fair)
* [共享的可变状态和并发性](#shared_mutable_state_concurrency)
	* [这个问题](#the_problem)
	* [Volatiles毫无用处](#volatiles-are-of-no-help)
	* [线程安全的数据结构](#thread_safe_data_structures)
	* [细粒度线程约束](#thread_confinement_fine_grained)
	* [粗粒度线程约束](#thread_confinement_coarse_grained)
	* [Mutex](#mutual_exclusion)
	* [参与者](#actors)
* [Select表达式](#select_expression)
	* [选择通道](#selecting_from_channels)
	* [关闭时的select](#selecting_on_close)
	* [选择发送](#selecting_to_send)
	* [选择deferred值](#selecting_deferred_values)
	* [切换到deferred值的通道](#switch_over_a_channel_of_deferred_values)
* [进一步的阅读](#further_reading)

<!-- more -->

## <a name="coroutine_basics"></a>协程基础

本节介绍基本的协程概念。

### <a name="your_first_coroutine"></a>你的第一个协程

运行下面的代码:

```kotlin
fun main(args: Array<String>) {
    launch { // 启动新协程
        delay(1000L) // 非阻塞延迟1秒 (默认时间单位是ms)
        println("World!") // 打印在延迟后
    }
    println("Hello,") // 主函数继续，而协程延迟
    Thread.sleep(2000L) // 阻塞主线程2秒，以保持JVM的生命
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-basic-01.kt)得到完整的代码

运行这段代码:

```
Hello,
World!
```

本质上，协程是轻量的线程。
它们是由`launch` 协程构建器启动的。
你可以用同样的结果取代`launch { ... }`，用`thread { ... }` 和`delay(…)`，用Thread.sleep(...)。
试一试。

如果您开始用`thread`替换`launch`，编译器会产生以下错误:

```
错误:Kotlin:挂起函数只允许从一个协程或另一个挂起函数调用
```

这是因为[delay](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/delay.html)是一种特殊的挂起函数，它不会阻塞线程，而是挂起了协程，它只能在一个协程中使用。

### <a name="bridging_blocking_nonblocking_worlds"></a>桥接阻塞和非阻塞的世界

第一个例子混合了非阻塞`delay(...)`和阻塞`Thread.sleep(...)`在相同的mian代码中。
很容易迷路。
让我们通过使用[runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/run-blocking.html)来清楚地分离阻塞和非阻塞的世界。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> { // 开始主协程
    launch { // 启动新协程
        delay(1000L)
        println("World!")
    }
    println("Hello,") // 当孩子被延迟时，主协程仍在继续
    delay(2000L) // 非阻塞延迟2秒使JVM存活
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-basic-02.kt)得到完整的代码

结果是一样的，但是这段代码只使用非阻塞的[delay](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/delay.html)。

`runBlocking { ... }` 作为一个适配器，在这里使用它来启动顶级的主协程。
runBlocking块之外的常规代码,直到`runBlocking`内部的协程活跃。

这也是为挂起函数编写单元测试的一种方法:

```kotlin
class MyTest {
    @Test
    fun testMySuspendingFunction() = runBlocking<Unit> {
        // 这里我们可以使用任何我们喜欢的断言样式来使用挂起函数
    }
}
```

### <a name="waiting_for_job"></a>等待job

拖延一段时间，而另一种协程正在工作，这不是一种好方法。让我们明确地等待(以非阻塞方式)直到我们启动的后台任务完成:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch { // 启动新的协程，并保持job的引用
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join() // 等到孩子协程完成
}
```
> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-basic-03.kt)得到完整的代码

现在的结果仍然是相同的，但是主协程的代码并没有以任何方式与后台作业的持续时间相关联。
好多了。

### <a name="extract_function_refactoring"></a>提取函数重构

让我们在`launch { ... }` 中提取代码块到一个单独的函数。
当您在这段代码中执行"Extract function"重构时，您将得到一个带有`suspend`修饰符的新函数。
这是你的第一个挂起函数。
挂起函数可以像普通函数一样在内部使用，但是它们的附加功能是，它们可以使用其他的挂起函数，比如这个例子中的`delay`，挂起执行的协程。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch { doWorld() }
    println("Hello,")
    job.join()
}

// 这是你的第一个挂起函数
suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```
> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-basic-04.kt)得到完整的代码

### <a name="coroutines_are_light_weight"></a>协程是轻量级的

运行下面的代码:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val jobs = List(100_000) { // 启动大量的协程和他们的jobs列表
        launch {
            delay(1000L)
            print(".")
        }
    }
    jobs.forEach { it.join() } // 等待所有的jobs完成
}
```
> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-basic-05.kt)得到完整的代码

它启动了100K的协程，一秒钟之后，每一个协程就会打印一个点。
现在，用线程来试一下。将会发生什么?(很可能你的代码会产生某种内存溢出错误)

### <a name="coroutines_like_daemon_threads"></a>协程就像守护线程

下面的代码启动了一个长时间运行的协程，它每秒打印两次"I'm sleeping"，然后在一些delay之后返回main函数:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    launch {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // 在延迟之后退出
}
```

您可以运行并看到它打印了三行并终止:

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
```

活跃的协程并不能使整个过程保持活跃。
它们就像守护进程线程。

----

## <a name="cancellation_timeouts"></a>取消和超时

本节介绍协程取消和超时。

### <a name="cancelling_coroutine_execution"></a>取消协程执行

在小应用程序中，"main"方法的返回听起来似乎是一个好主意，可以让所有的协程隐式终止。
在一个较大的、长期运行的应用程序中，您需要更细粒度的控制。
launch函数返回一个可以用来取消运行的协程的Job:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
    delay(1300L) // 延迟一点
    println("main: I'm tired of waiting!")
    job.cancel() // 取消job
    job.join() // 等待job的完成
    println("main: Now I can quit.")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-cancel-01.kt)得到完整的代码

它产生以下输出:

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
main: I'm tired of waiting!
main: Now I can quit.
``` 

只要main调用了`job.cancel`，我们看不到其他协程的输出，因为它被取消了。
还有一个[Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/index.html)扩展函数[cancelAndJoin](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/cancel-and-join.html)，它结合了[cancel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/cancel.html)和[join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/join.html)调用。

### <a name="cancellation_cooperative"></a>取消是需要协作的

协程取消是需要协作的。
一个取消的代码必须进行协作。
所有的挂起函数在`kotlinx.coroutines`都是可取消的。
他们检测协程的取消，取消时抛出[CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-cancellation-exception.html)。
但是，如果一个协程正在计算并且没有检查取消，那么它就不能被取消，就像下面的例子所示:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val startTime = System.currentTimeMillis()
    val job = launch {
        var nextPrintTime = startTime
        var i = 0
        while (i < 5) { // 循环计算，只是浪费CPU
            // 每秒打印消息两次
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // 延迟一点
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // 取消job并等待其完成
    println("main: Now I can quit.")
}
```
> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-cancel-02.kt)得到完整的代码
 
运行它，看它会继续打印"I'm sleeping"，即使是在取消之后，直到工作完成了五个迭代之后。

### <a name="making_computation_code_cancellable"></a>使计算代码可取消

有两种方法可以使计算代码可以被取消。
第一个是周期性地调用一个挂起函数来检查取消。
[yield](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/yield.html)函数是这个目的的一个很好的选择。
另一个是显式地检查取消状态。
让我们来试试后面的方法。

在前面的例子中用`while (isActive)`替换while (i < 5)并重新运行它。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val startTime = System.currentTimeMillis()
    val job = launch {
        var nextPrintTime = startTime
        var i = 0
        while (isActive) { // 可取消的计算循环
            // 每秒打印消息两次
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("I'm sleeping ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
    delay(1300L) // 延迟一点
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // 取消job并等待其完成
    println("main: Now I can quit.")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-cancel-03.kt)得到完整的代码

可以看到，现在这个循环被取消了。
[isActive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/is-active.html)是通过[CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/index.html)对象在协程代码中可用的属性。

### <a name="closing_resources_with_finally"></a> 在finally关闭资源

可取消的挂起函数在协程中抛出[CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-cancellation-exception.html)的地方可以用通常的方式来处理。
例如,`try {...} finally {…}` 表达式和Kotlin `use`函数执行它们的终结动作，当协程被取消时：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch {
        try {
            repeat(1000) { i ->
                println("I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            println("I'm running finally")
        }
    }
    delay(1300L) // 延迟一点
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // 取消job并等待其完成
    println("main: Now I can quit.")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-cancel-04.kt)得到完整的代码

[join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/join.html)和[cancelAndJoin](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/cancel-and-join.html)等待所有的终结操作完成，因此上面的示例产生以下输出:

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
main: I'm tired of waiting!
I'm running finally
main: Now I can quit.
```

### <a name="run_noncancellable_block"></a>运行不可取消块

在前一个示例的finally块中使用挂起函数的任何尝试都会导致CancellationException，因为运行该代码的协程被取消了。
通常，这不是问题，因为所有行为良好的关闭操作(关闭文件、取消job或关闭任何通信通道)通常是非阻塞的，并且不涉及任何挂起函数。
然而，在极少数情况下，当您需要在被取消的协程 挂起时，您可以将相应的代码打包在`run(NonCancellable) {...}`使用[run](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/run.html)函数和[NonCancellable](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-non-cancellable/index.html) context，如下面的示例所示:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = launch {
        try {
            repeat(1000) { i ->
                println("I'm sleeping $i ...")
                delay(500L)
            }
        } finally {
            run(NonCancellable) {
                println("I'm running finally")
                delay(1000L)
                println("And I've just delayed for 1 sec because I'm non-cancellable")
            }
        }
    }
    delay(1300L) // 延迟一点
    println("main: I'm tired of waiting!")
    job.cancelAndJoin() // 取消job并等待其完成
    println("main: Now I can quit.")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-cancel-05.kt)得到完整的代码

### <a name="timeout"></a>超时

在实践中取消协程执行的最明显的原因是它的执行时间超过了一些超时。
虽然您可以手动跟踪对应Job的引用，并启动一个单独的协程，以便在延迟之后取消跟踪，但是可以使用withTimeout函数来执行它。
请看下面的例子:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    withTimeout(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-cancel-06.kt)得到完整的代码
 
它产生以下输出:

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Exception in thread "main" kotlinx.coroutines.experimental.TimeoutCancellationException: Timed out waiting for 1300 MILLISECONDS
```

[withTimeout](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/with-timeout.html)抛出的`TimeoutCancellationException`是[CancellationException](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-cancellation-exception.html)的子类。
我们还没有看到它的堆栈跟踪在控制台上打印出来。
这是因为在被取消的协程里`CancellationException`被认为是一个正常的原因。
但是，在本例中，我们在`main`函数中使用了`withTimeout`。

因为取消只是一个异常，所有的资源都将以通常的方式关闭。
如果你需要在任何类型的超时上做一些额外的动作,您可以在`try {...} catch (e: TimeoutCancellationException) {...}`块中使用超时来包装代码,者使用类似于[withTimeout](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/with-timeout.html)的[withTimeoutOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/with-timeout-or-null.html)函数，但在超时时它返回null，而不是抛出异常：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val result = withTimeoutOrNull(1300L) {
        repeat(1000) { i ->
            println("I'm sleeping $i ...")
            delay(500L)
        }
        "Done" // will get cancelled before it produces this result
    }
    println("Result is $result")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-cancel-07.kt)得到完整的代码

运行这段代码时不再出现异常:

```
I'm sleeping 0 ...
I'm sleeping 1 ...
I'm sleeping 2 ...
Result is null
```

## <a name="composing_suspending_functions"></a>组成挂起函数

这一节介绍了各种挂起函数的组成。

### <a name="sequential_default"></a>默认顺序

假设我们在其他地方定义了两个挂起的函数，它们做一些有用的事情，比如远程服务调用或计算。
我们只是假装它们很有用，但实际上每一个都只是为了这个例子的目的延迟了一秒钟：

```kotlin
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L) // 假装我们在做一些有用的事情
    return 13
}

suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L) // 假装我们在做一些有用的事情
    return 29
}
```

如果需要按顺序调用它们，我们需要做什么?首先是`doSomethingUsefulOne`，然后是`doSomethingUsefulTwo`，然后计算它们的结果的总和?
在实践中，我们就会这样做，我们使用第一个函数的结果来决定是否需要调用第二个函数，或者决定如何调用它。

我们只是使用常规的顺序调用，因为在协程中的代码，就像常规代码一样，是默认的顺序。下面的例子通过测量执行两个挂起函数的总时间来演示它:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = doSomethingUsefulOne()
        val two = doSomethingUsefulTwo()
        println("The answer is ${one + two}")
    }
    println("Completed in $time ms")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-compose-01.kt)得到完整的代码

它产生了这样的东西:

```
The answer is 42
Completed in 2017 ms
```

### <a name="concurrent_using_async"></a>使用async并发

如果在调用`doSomethingUsefulOne`和`doSomethingUsefulTwo`之间没有依赖关系，并且我们希望通过同时执行这两种方法来获得更快的答案?这就是[async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/async.html)提供帮助的地方。

在概念上，[async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/async.html)就像[launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/launch.html)一样。
它启动了一个单独的协程，它是一个轻量的线程，它可以和所有其他的协同工作。
不同的是，`launch`返回一个[Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/index.html)，并且不带任何结果值，而`async`返回一个[Deferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-deferred/index.html)——一个轻量级的非阻塞future，它代表了稍后提供一个结果的承诺。
您可以使用.await()在一个`deferred`值上获得它的最终结果，但是`Deferred`也是一项`Job`，所以您可以在需要的时候取消它。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-compose-02.kt)得到完整的代码

它产生了这样的东西:

```
The answer is 42
Completed in 1017 ms
```

速度是两倍，因为我们同时执行了两个协程。
注意，与协程的并发性总是很明显的。

### <a name="lazily_started_async"></a>懒启动async

使用一个具有[CoroutineStart.LAZY](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-start/-l-a-z-y.html)值的可选`start`参数，可以使用惰性选项来使用[async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/async.html)。
只有当某个[await](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-deferred/await.html)或某个[start](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/start.html)函数被调用时，它才会启动它的协程。
运行以下示例与之前的示例不同，仅通过此选项:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val time = measureTimeMillis {
        val one = async(start = CoroutineStart.LAZY) { doSomethingUsefulOne() }
        val two = async(start = CoroutineStart.LAZY) { doSomethingUsefulTwo() }
        println("The answer is ${one.await() + two.await()}")
    }
    println("Completed in $time ms")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-compose-03.kt)得到完整的代码

它产生了这样的东西:

```
The answer is 42
Completed in 2017 ms
```

因此，我们回到了顺序执行，因为我们首先`start`和`await` `one`，然后是`two`。
它不是用于懒惰目的的用例。
当计算值涉及到挂起函数时，它被设计为替代标准的惰性函数。

### <a name="async_style_functions"></a>异步式函数

我们可以定义异步式的函数，使用async协程构建器来异步调用`doSomethingUsefulOne`和`doSomethingUsefulTwo`。
用"async"前缀来命名这样的函数是一种很好的方式，以强调它们只启动异步计算，并且需要使用产生的延迟值来获得结果。

```kotlin
// asyncSomethingUsefulOne的结果类型是Deferred<Int>
fun asyncSomethingUsefulOne() = async {
    doSomethingUsefulOne()
}

// asyncSomethingUsefulTwo的结果类型是Deferred<Int>
fun asyncSomethingUsefulTwo() = async {
    doSomethingUsefulTwo()
}
```

注意，这些`asyncXXX`函数**不是**挂起函数。它们可以在任何地方使用。
然而，它们的使用总是意味着对调用代码的异步(这里即并发)执行。

下面的例子展示了他们在协程外的使用:

```kotlin
// 注意，在本例中，我们没有将`runBlocking`放在`main`的右边
fun main(args: Array<String>) {
    val time = measureTimeMillis {
        // 我们可以在协程外启动异步操作
        val one = asyncSomethingUsefulOne()
        val two = asyncSomethingUsefulTwo()
        // 但是等待结果必须包括挂起或阻塞.
        // 这里我们在主线程使用`runBlocking { ... }`块等待结果
        runBlocking {
            println("The answer is ${one.await() + two.await()}")
        }
    }
    println("Completed in $time ms")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-compose-04.kt)得到完整的代码

## <a name="coroutine_context_dispatchers"></a>协程context和分派器

协程总是在某些context中执行，这些context由[CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/-coroutine-context/)类型的值来表示，这是在Kotlin标准库中定义的。

协程context是一组不同的元素。
主要的元素是我们之前见过的协程的[Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/index.html)，以及它的分派器，它在这一节中介绍。

### <a name="dispatchers_threads"></a>分派器和线程

协程context包括一个协程分派器(请参阅[CoroutineDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-dispatcher/index.html))，它决定了相应的协程执行的线程或线程的使用情况。
协程分派器可以将协程执行限制在特定的线程中，将其分派到线程池中，或者让它不受限制地运行。

像[launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/launch.html)和[async](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/async.html)这样的构建器都可以接受一个可选的[CoroutineContext](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/-coroutine-context/)参数，该参数可用于显式地为新的协程和其他context元素指定分派器。

试试下面的例子:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val jobs = arrayListOf<Job>()
    jobs += launch(Unconfined) { // 不受限制——将工作在主线程
        println("      'Unconfined': I'm working in thread ${Thread.currentThread().name}")
    }
    jobs += launch(coroutineContext) { // 父ccontext，runBlocking 协程
        println("'coroutineContext': I'm working in thread ${Thread.currentThread().name}")
    }
    jobs += launch(CommonPool) { // 将被分派到ForkJoinPool.commonPool(或同等的)
        println("      'CommonPool': I'm working in thread ${Thread.currentThread().name}")
    }
    jobs += launch(newSingleThreadContext("MyOwnThread")) { // 将得到它自己的新线程
        println("          'newSTC': I'm working in thread ${Thread.currentThread().name}")
    }
    jobs.forEach { it.join() }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-01.kt)得到完整的代码
 
它产生以下输出(可能顺序不同):

```
      'Unconfined': I'm working in thread main
      'CommonPool': I'm working in thread ForkJoinPool.commonPool-worker-1
          'newSTC': I'm working in thread MyOwnThread
'coroutineContext': I'm working in thread main
```

我们在前面的部分中使用的缺省分派器是[DefaultDispatcher](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-default-dispatcher.html)，它等于当前实现中的[CommonPool](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-common-pool/index.html)。因此，`launch { ... }` 与`launch(DefaultDispather) { ... }`相同，与`launch(CommonPool) { ... }`相同。

稍后将显示[coroutineContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/coroutine-context.html)父类与非约束[context](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-unconfined/index.html)之间的区别

注意，[newSingleThreadContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/new-single-thread-context.html)创建了一个新的线程，这是一个非常昂贵的资源。在实际应用程序中，它必须在不再需要时释放，使用[close](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-thread-pool-dispatcher/close.html)函数，或者存储在顶级变量中，并在整个应用程序中重用。

### <a name="unconfined_vs_confined_dispatcher"></a> 非限制 vs 限制 分派器

非限制([Unconfined](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-unconfined/index.html))的协程分派器在调用者线程中启动了协程，但直到第一个挂起点才开始。
挂起之后，它在被调用的挂起函数完全确定的线程中恢复。
当协程不消耗CPU时间或更新任何限制在特定线程的共享数据(如UI)时，非限制分派器是合适的。

另一方面，可以通过[CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/index.html)接口在任何协程的块中使用的[coroutineContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/coroutine-context.html)属性，是对这个特定的协程的context的引用。
通过这种方式，可以继承父context。
特别是，[runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/run-blocking.html) 协程的缺省调度程序被限制在调用程序线程中，因此继承它的作用是将执行限制在这个线程中，并具有可预测的FIFO调度。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val jobs = arrayListOf<Job>()
    jobs += launch(Unconfined) { // 不受限制——主要线程的工作
        println("      'Unconfined': I'm working in thread ${Thread.currentThread().name}")
        delay(500)
        println("      'Unconfined': After delay in thread ${Thread.currentThread().name}")
    }
    jobs += launch(coroutineContext) { // 父ccontext，runBlocking 协程
        println("'coroutineContext': I'm working in thread ${Thread.currentThread().name}")
        delay(1000)
        println("'coroutineContext': After delay in thread ${Thread.currentThread().name}")
    }
    jobs.forEach { it.join() }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-02.kt)得到完整的代码
 
它产生以下输出(可能顺序不同):

```
      'Unconfined': I'm working in thread main
'coroutineContext': I'm working in thread main
      'Unconfined': After delay in thread kotlinx.coroutines.DefaultExecutor
'coroutineContext': After delay in thread main
```

因此，继承了runBlocking {...} 协程的coroutineContext 继续在主线程中执行，而未限制的线程则在默认的执行程序线程中恢复，而[delay](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/delay.html)函数正在使用该线程。

### <a name="debugging_coroutines_threads"></a>调试协程和线程

协程可以在一个线程上挂起，并在另一个线程上使用非限制([Unconfined](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-unconfined/index.html))分派器或默认的多线程分派器。
即使是单线程的分派器，也很难弄清楚协程在做什么，在什么地方，什么时候做什么。
使用线程调试应用程序的常用方法是在每个日志语句的日志文件中打印线程名。这个特性得到了日志框架的广泛支持。
在使用协程时，线程名称本身并没有提供太多context，所以`kotlinx.coroutines`包括调试工具，使其更容易。

运行下面的代码使用`-Dkotlinx.coroutines.debug` JVM选项:

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main(args: Array<String>) = runBlocking<Unit> {
    val a = async(coroutineContext) {
        log("I'm computing a piece of the answer")
        6
    }
    val b = async(coroutineContext) {
        log("I'm computing another piece of the answer")
        7
    }
    log("The answer is ${a.await() * b.await()}")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-03.kt)得到完整的代码

有三个协程。
主要的协程(#1)—— 一个`runBlocking`，和两个协程计算deferred值`a`(#2)和`b`(#3)。
它们都是在`runBlocking`上下文中执行的，并且仅限于主线程。这段代码的输出是:

```
[main @coroutine#2] I'm computing a piece of the answer
[main @coroutine#3] I'm computing another piece of the answer
[main @coroutine#1] The answer is 42
```

日志函数在方括号中打印出线程的名称，您可以看到，它是主线程，但是当前执行的协程的标识符被附加。
当打开调试模式时，这个标识符被连续地分配给所有创建的协程。

您可以在[newCoroutineContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/new-coroutine-context.html)函数的文档中阅读更多关于调试工具的信息。

### <a name="jumping_between_threads"></a>线程之间跳转

运行下面的代码使用`-Dkotlinx.coroutines.debug` JVM选项:

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main(args: Array<String>) {
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                run(ctx2) {
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-04.kt)得到完整的代码

它演示了几种新技术。
一个是使用[runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/run-blocking.html)和一个显式指定的context，另一个是使用[run](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/run.html)函数来更改一个协程的上下文，同时仍然保持在相同的协程中，如下所示:

```
[Ctx1 @coroutine#1] Started in ctx1
[Ctx2 @coroutine#1] Working in ctx2
[Ctx1 @coroutine#1] Back to ctx1
```

注意，这个例子也使用了来自Kotlin标准库的`use`函数来释放那些在不需要时创建的线程，这些线程是用[newSingleThreadContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/new-single-thread-context.html)创建的。

### <a name="job_context"></a>context中的job

协程的Job是其context的一部分。协程可以使用coroutineContext[Job]表达式从自己的context中检索它：

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    println("My job is ${coroutineContext[Job]}")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-05.kt)得到完整的代码

当在[调试模式](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#debugging-coroutines-and-threads)下运行时，它会产生这样的东西:

```
My job is "coroutine#1":BlockingCoroutine{Active}@6d311334
```

因此，在[CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/index.html)中，[isActive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/is-active.html)是coroutineContext[Job]的一个便捷的快捷方式!!

### <a name="children_of_a_coroutine"></a>子协程

当一个协程的[coroutineContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/coroutine-context.html)被用来`launch`另一个协程时，新的协程job就变成了父母的子Job。
当父协程被取消时，所有的子协程也会被递归地取消。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        // it spawns two other jobs, one with its separate context
        val job1 = launch {
            println("job1: I have my own context and execute independently!")
            delay(1000)
            println("job1: I am not affected by cancellation of the request")
        }
        // and the other inherits the parent context
        val job2 = launch(coroutineContext) {
            println("job2: I am a child of the request coroutine")
            delay(1000)
            println("job2: I will not execute this line if my parent request is cancelled")
        }
        // request completes when both its sub-jobs complete:
        job1.join()
        job2.join()
    }
    delay(500)
    request.cancel() // cancel processing of the request
    delay(1000) // delay a second to see what happens
    println("main: Who has survived request cancellation?")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-06.kt)得到完整的代码

这段代码的输出是:

```
job1: I have my own context and execute independently!
job2: I am a child of the request coroutine
job1: I am not affected by cancellation of the request
main: Who has survived request cancellation?
``` 

### <a name="combining_contexts"></a>组合contexts

使用`+`操作符可以组合使用协程contexts。
右边的contexts替换了左边的contexts的相关条目。
例如，父Job可以被继承，而它的分派器替换了:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // 启动一个协程来处理某种类型的传入请求
    val request = launch(coroutineContext) { // 使用`runBlocking`context
        // 在CommonPool中生成cpu密集型的子job !!! 
        val job = launch(coroutineContext + CommonPool) {
            println("job: I am a child of the request coroutine, but with a different dispatcher")
            delay(1000)
            println("job: I will not execute this line if my parent request is cancelled")
        }
        job.join() // 当它的子job完成时，请求就完成了
    }
    delay(500)
    request.cancel() // 取消对请求的处理
    delay(1000) // 延迟一秒看看会发生什么
    println("main: Who has survived request cancellation?")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-07.kt)得到完整的代码

这段代码的预期结果是:

```
job: I am a child of the request coroutine, but with a different dispatcher
main: Who has survived request cancellation?
```

### <a name="parental_responsibilities"></a>父协程责任

一个父协程总是等待所有孩子的完成。
父协程不必显式地跟踪它启动的所有子节点，它也不必使用Job.join在最后等待它们:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // launch a coroutine to process some kind of incoming request
    val request = launch {
        repeat(3) { i -> // launch a few children jobs
            launch(coroutineContext)  {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, 600ms
                println("Coroutine $i is done")
            }
        }
        println("request: I'm done and I don't explicitly join my children that are still active")
    }
    request.join() // wait for completion of the request, including all its children
    println("Now processing of the request is complete")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-08.kt)得到完整的代码

结果将是:

```
request: I'm done and I don't explicitly join my children that are still active
Coroutine 0 is done
Coroutine 1 is done
Coroutine 2 is done
Now processing of the request is complete
```

### <a name="naming_coroutines_debugging"></a>命名协程的调试

自动分配的id通常都很好，而且您只需要将来自相同的协程的日志记录关联起来。
然而，当协程与特定的请求或特定的后台任务绑定时，为调试的目的，最好将其命名。
[CoroutineName](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-name/index.html) context元素提供了与线程名称相同的功能。
当[调试模式](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#debugging-coroutines-and-threads)打开时，它将显示在线程名称中，该名称将执行这个协程。

下面的例子演示了这个概念:

```kotlin
fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main(args: Array<String>) = runBlocking(CoroutineName("main")) {
    log("Started main coroutine")
    // run two background value computations
    val v1 = async(CoroutineName("v1coroutine")) {
        delay(500)
        log("Computing v1")
        252
    }
    val v2 = async(CoroutineName("v2coroutine")) {
        delay(1000)
        log("Computing v2")
        6
    }
    log("The answer for v1 / v2 = ${v1.await() / v2.await()}")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-09.kt)得到完整的代码

输出它产生使用`-Dkotlinx.coroutines.debug` JVM选项类似于:

```
[main @main#1] Started main coroutine
[ForkJoinPool.commonPool-worker-1 @v1coroutine#2] Computing v1
[ForkJoinPool.commonPool-worker-2 @v2coroutine#3] Computing v2
[main @main#1] The answer for v1 / v2 = 42
```

### <a name="cancellation_via_explicit_job"></a>显式取消Job

让我们把关于contexts、children和jobs的知识放在一起。
假设我们的应用程序有一个具有生命周期的对象，但是这个对象不是一个协程。
例如，我们正在编写一个Android应用程序，并在Android activity的环境中launch各种各样的协程，以执行异步操作来获取和更新数据、执行动画等等。
当活动被破坏以避免内存泄漏时，必须取消所有这些协程。

我们可以通过创建与activity生命周期相关联的[Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/index.html)实例来管理我们的协程的生命周期。
一个job实例是使用[job()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job.html)工厂函数创建的，如下面的例子所示。
我们需要确保所有的协程都是在这个job的context下启动的。然后，一次[Job.cancel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/cancel.html)调用就会终止它们。
此外,[Job.join](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/join.html)等待它们所有完成，因此我们也可以在本例中使用[cancelAndJoin](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/cancel-and-join.html):

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val job = Job() // 创建一个job对象来管理我们的生命周期
    // 现在launch 10个coroutines，每个都在不同的时间工作
    val coroutines = List(10) { i ->
        // 他们都是我们工作对象的孩子
        launch(coroutineContext + job) { // 我们使用main runBlocking线程的上下文，但是使用我们自己的job对象
            delay((i + 1) * 200L) // variable delay 200ms, 400ms, ... etc
            println("Coroutine $i is done")
        }
    }
    println("Launched ${coroutines.size} coroutines")
    delay(500L) // 延迟半秒
    println("Cancelling the job!")
    job.cancelAndJoin() // 取消所有的协程，等待它们全部完成
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-context-10.kt)得到完整的代码

这个例子的输出是:

```
Launched 10 coroutines
Coroutine 0 is done
Coroutine 1 is done
Cancelling the job!
```

正如您所看到的，只有前三个协程打印了一条消息，而其他的则被单次调用`job.cancelAndJoin()`取消了。
因此，我们在假想的Android应用程序中所需要做的就是在创建activity时创建父Job对象，将其用于子节点，并在活动destroyed时取消它。
我们不能在Android生命周期的情况下`join`它们，因为它是同步的，但是当构建后端服务以确保有界的资源使用时，这种连接能力是很有用的。

## <a name="channels"></a>通道

Deferred值提供了一种方便的方法，可以在协程之间传递单个值。`通道`提供了一种传输数据流的方法。

### <a name="channel_basics"></a>通道基础知识

一个[通道](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-channel/index.html)在概念上非常类似于BlockingQueue。一个关键的区别是，它用挂起的[send](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-send-channel/send.html)替代阻塞的put操作，用挂起的[receive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-receive-channel/receive.html) 替代阻塞的take操作。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        // this might be heavy CPU-consuming computation or async logic, we'll just send five squares
        for (x in 1..5) channel.send(x * x)
    }
    // here we print five received integers:
    repeat(5) { println(channel.receive()) }
    println("Done!")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-01.kt)得到完整的代码

这段代码的输出是:

```
1
4
9
16
25
Done!
```

### <a name="closing_iteration_over_channels"></a>通道的关闭和迭代

与队列不同的是，通道可以关闭，以指示不再有更多的元素出现。
在接收端，使用常规的`for`循环从通道接收元素是很方便的。

从概念上说，[close](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-send-channel/close.html)就像向通道发送一个特殊的关闭令牌。
当接收到这个关闭令牌时，迭代就停止了，因此可以保证所有之前发送的元素都被接收到:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<Int>()
    launch {
        for (x in 1..5) channel.send(x * x)
        channel.close() // we're done sending
    }
    // here we print received values using `for` loop (until the channel is closed)
    for (y in channel) println(y)
    println("Done!")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-02.kt)得到完整的代码

### <a name="building_channel_producers"></a>构建通道生成器

在这种情况下，一种协程生成一系列元素的模式是很常见的。
这是在并发代码中经常发现的生产者-消费者模式的一部分。
您可以将这样一个生成器抽象为一个以通道为参数的函数，但这与通常的结果相反，即必须从函数返回结果。
有一个名为[produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/produce.html)的便利的协程构建器，它使得在生产者端很容易做，并且一个扩展函数[consumeEach](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/consume-each.html)，它替代了消费者端的`for`循环:

```kotlin
fun produceSquares() = produce<Int> {
    for (x in 1..5) send(x * x)
}

fun main(args: Array<String>) = runBlocking<Unit> {
    val squares = produceSquares()
    squares.consumeEach { println(it) }
    println("Done!")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-03.kt)得到完整的代码

### <a name="pipelines"></a>管道

管道是一种模式，其中一种可能是无限的值流:

```kotlin
fun produceNumbers() = produce<Int> {
    var x = 1
    while (true) send(x++) // 从1开始的无限整数流
}
```

另一个协程或多个协程正在消耗这一流，进行一些处理，并产生一些其他的结果。在下面的例子中，数字是平方:

```kotlin
fun square(numbers: ReceiveChannel<Int>) = produce<Int> {
    for (x in numbers) send(x * x)
}
```

主程序启动并连接整个管道:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val numbers = produceNumbers() // 从1和on生成整数
    val squares = square(numbers) // 整数平方
    for (i in 1..5) println(squares.receive()) // 打印前五
    println("Done!") // we are done
    squares.cancel() // 需要在更大的应用程序中取消这些协程
    numbers.cancel()
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-04.kt)得到完整的代码

我们不必在这个示例应用程序中取消这些协程，因为[协程就像守护线程](#coroutines-are-like-daemon-threads)，但是在一个更大的应用程序中，如果我们不再需要它，我们就需要停止我们的管道。
或者，我们可以像下面的例子中演示的那样，将管道协程作为[主协程的孩子](#children-of-a-coroutine)运行。

### <a name="prime_numbers_pipeline"></a>用管道求质数

让我们用一个例子来说明管道的极端情况，这个例子可以用一个管道来生成质数。
我们从一个无限的数字序列开始。
这一次，我们引入了一个显式的`context`参数并将其传递给[produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/produce.html)建造者，以便调用者可以控制我们的协作程序的运行位置:

```kotlin
fun numbersFrom(context: CoroutineContext, start: Int) = produce<Int>(context) {
    var x = start
    while (true) send(x++) // 从start开始的无限整数流
}
```

下面的管道阶段将过滤传入的数字流，删除可以被给定的素数整除的所有数字:

```kotlin
fun filter(context: CoroutineContext, numbers: ReceiveChannel<Int>, prime: Int) = produce<Int>(context) {
    for (x in numbers) if (x % prime != 0) send(x)
}
```

现在，我们通过从2启动一个数字流来构建我们的管道，从当前通道中获取一个质数，并为每个找到的素数启动新的管道阶段:

```
numbersFrom(2) -> filter(2) -> filter(3) -> filter(5) -> filter(7) ... 
```

下面的例子输出了前10个素数，在主线程的context中运行整个管道。
因为所有的协程都是在[coroutineContext](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-coroutine-scope/coroutine-context.html)中作为主要的[runBlocking](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/run-blocking.html)协程的孩子发布的，所以我们不需要对我们已经开始的所有的协程有一个明确的列表。
我们使用[cancelChildren](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/kotlin.coroutines.experimental.-coroutine-context/cancel-children.html)扩展函数来取消所有的子协程。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    var cur = numbersFrom(coroutineContext, 2)
    for (i in 1..10) {
        val prime = cur.receive()
        println(prime)
        cur = filter(coroutineContext, cur, prime)
    }
    coroutineContext.cancelChildren() // cancel all children to let main finish
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-05.kt)得到完整的代码

这段代码的输出是:

```
2
3
5
7
11
13
17
19
23
29
```

注意，您可以使用来自标准库的构建器[buildIterator](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/build-iterator.html)构建相同的管道。
用`produce`换成`buildIterator`，`send`换成`yield`，`receive`换成`next`，`ReceiveChannel` 换成`Iterator`，并摆脱`context`。
您也不需要`runBlocking`。
但是，使用通道的管道的好处是，如果在[CommonPool](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-common-pool/index.html) context中运行它，它实际上可以使用多个CPU内核。

无论如何，这是一种非常不切实际的求质数的方法。
在实践中,管道涉及一些其他挂起调用(如异步调用远程服务),这些管道不能使用`buildSeqeunce/buildIterator`,因为他们不允许任意挂起,像`produce`,这是完全异步的。

### <a name="Fan-out"></a>扇出

多个协程可以从相同的通道接收，在它们之间分配工作。让我们从一个定期产生整数的生产者(每秒钟10个数字)开始。

```kotlin
fun produceNumbers() = produce<Int> {
    var x = 1 // start from 1
    while (true) {
        send(x++) // produce next
        delay(100) // wait 0.1s
    }
}
```

然后我们可以有几个处理器的协程。在本例中，他们只打印他们的id和接收的号码:

```kotlin
fun launchProcessor(id: Int, channel: ReceiveChannel<Int>) = launch {
    channel.consumeEach {
        println("Processor #$id received $it")
    }    
}
```

现在，让我们启动5个处理器，让它们工作大约一秒钟。看看会发生什么:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val producer = produceNumbers()
    repeat(5) { launchProcessor(it, producer) }
    delay(950)
    producer.cancel() // cancel producer coroutine and thus kill them all
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-06.kt)得到完整的代码

输出将类似于下面一个，尽管接收每个特定整数的处理器id可能不同:

```
Processor #2 received 1
Processor #4 received 2
Processor #0 received 3
Processor #1 received 4
Processor #3 received 5
Processor #2 received 6
Processor #4 received 7
Processor #0 received 8
Processor #1 received 9
Processor #3 received 10
```

请注意，取消一个生产商的协程会关闭它的通道，从而最终终止处理器协程正在进行的通道的迭代。

### <a name="Fan-in"></a>扇入

多个协程可以发送到相同的通道。例如，让我们有一个字符串的通道，以及一个挂起函数，它通过指定的延迟将指定的字符串发送到这个通道。

```kotlin
suspend fun sendString(channel: SendChannel<String>, s: String, time: Long) {
    while (true) {
        delay(time)
        channel.send(s)
    }
}
```

现在，让我们看看如果我们启动了一对发送字符串的协程(在本例中，我们在主线程中作为主协程的子协程启动它们):

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<String>()
    launch(coroutineContext) { sendString(channel, "foo", 200L) }
    launch(coroutineContext) { sendString(channel, "BAR!", 500L) }
    repeat(6) { // 接收前六
        println(channel.receive())
    }
    coroutineContext.cancelChildren() // cancel all children to let main finish
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-07.kt)得到完整的代码

它的输出是:

```
foo
foo
BAR!
foo
foo
BAR!
```

### <a name="buffered_channels"></a>缓冲通道

到目前为止，所展示的通道没有任何缓冲。
当发送方和接收方相遇时，没有缓冲的通道传输元素(也就是会合点)。
如果首先调用`send`，那么它将被挂起，直到`receive`被调用，如果首先调用`receive`，它将被挂起，直到`send`被调用。

[Channel()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-channel.html)工厂函数和[produce](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/produce.html)构建器都使用一个可选的`capacity`参数来指定缓冲区大小。
缓冲区允许发送者在挂起之前发送多个元素，类似于具有指定容量的`BlockingQueue`，在缓冲区满时阻塞。

看看以下代码的行为:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val channel = Channel<Int>(4) // create buffered channel
    val sender = launch(coroutineContext) { // launch sender coroutine
        repeat(10) {
            println("Sending $it") // print before sending each element
            channel.send(it) // will suspend when buffer is full
        }
    }
    // don't receive anything... just wait....
    delay(1000)
    sender.cancel() // cancel sender coroutine
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-08.kt)得到完整的代码

它用一个有四个容量的缓冲通道打印“发送”5次:

```
Sending 0
Sending 1
Sending 2
Sending 3
Sending 4
```

前4个元素被添加到缓冲区中，当发送第5个元素时，发送器会挂起。


### <a name="channels_fair"></a>通道是公平的

对通道的发送和接收操作是公平的，这与它们从多个协程调用的顺序是公平的。
它们以先入先出的顺序服务，例如，第一个调用接收的协程获取元素。
在下面的例子中，两个“ping”和“pong”从共享的“table”通道接收到“ball”对象。

```kotlin
data class Ball(var hits: Int)

fun main(args: Array<String>) = runBlocking<Unit> {
    val table = Channel<Ball>() // a shared table
    launch(coroutineContext) { player("ping", table) }
    launch(coroutineContext) { player("pong", table) }
    table.send(Ball(0)) // serve the ball
    delay(1000) // delay 1 second
    coroutineContext.cancelChildren() // game over, cancel them
}

suspend fun player(name: String, table: Channel<Ball>) {
    for (ball in table) { // receive the ball in a loop
        ball.hits++
        println("$name $ball")
        delay(300) // wait a bit
        table.send(ball) // send the ball back
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-channel-09.kt)得到完整的代码

"ping"协程是先开始的，所以它是第一个收到球的。
即使在把球送回到table的时候，"ping" 协程马上又开始接球，球却被"pong" 协程 所接收，因为它已经在等待它了。

```
ping Ball(hits=1)
pong Ball(hits=2)
ping Ball(hits=3)
pong Ball(hits=4)
```

请注意，有时通道可能会产生由于正在使用的执行器的性质而看起来不公平的执行。请参阅[这个问题](https://github.com/Kotlin/kotlinx.coroutines/issues/111)的详细信息。

## <a name="shared_mutable_state_concurrency"></a>共享的可变状态和并发性

可以使用多线程的分派器并发地执行协程，比如默认的CommonPool。
它展示了所有常见的并发问题。
主要问题是对共享易变状态的访问同步。
解决这一问题的方法与多线程世界中的解决方案类似，但也有一些是独特的。

### <a name="the_problem"></a>这个问题

让我们发动一千次的协程，一千次做同样的操作(总共有100万次执行)。
我们还将测量他们的完成时间来进行进一步的比较:

```kotlin
suspend fun massiveRun(context: CoroutineContext, action: suspend () -> Unit) {
    val n = 1000 // number of coroutines to launch
    val k = 1000 // times an action is repeated by each coroutine
    val time = measureTimeMillis {
        val jobs = List(n) {
            launch(context) {
                repeat(k) { action() }
            }
        }
        jobs.forEach { it.join() }
    }
    println("Completed ${n * k} actions in $time ms")    
}
```

我们从一个非常简单的动作开始，它使用多线程的CommonPool context来增加一个共享的可变变量。

```kotlin
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) {
        counter++
    }
    println("Counter = $counter")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-sync-01.kt)得到完整的代码

它最后会打印什么?
它不太可能打印“计数器=1000000”，因为1000个coroutines可以同时从多个线程并行地增加计数器，而不需要任何同步。

> 注意:如果您的旧系统中有2个或更少的cpu，那么您将始终看到1000000，因为在本例中，CommonPool只运行一个线程。为了重现这个问题，你需要做出以下的改变:

```kotlin
val mtContext = newFixedThreadPoolContext(2, "mtPool") // explicitly define context with two threads
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(mtContext) { // use it instead of CommonPool in this sample and below 
        counter++
    }
    println("Counter = $counter")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-sync-01b.kt)得到完整的代码

### <a name="volatiles-are-of-no-help"></a>Volatiles毫无用处

有一种常见的误解是，volatile变量可以解决并发性问题。让我们试一试:

```kotlin
@Volatile // in Kotlin `volatile` is an annotation 
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) {
        counter++
    }
    println("Counter = $counter")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-sync-02.kt)得到完整的代码

这段代码的工作速度较慢，但最终我们仍然不能得到“Counter=1000000”，因为volatile变量可以保证线性化(这是“原子”的技术术语)，并对相应的变量进行读写操作，但是不提供更大的操作的原子性(在我们的例子中是增加的)。

### <a name="thread_safe_data_structures"></a>线程安全的数据结构

对于线程和协程工作的通用解决方案是使用线程安全的(即同步的、线性化的或原子的)数据结构，为需要在共享状态上执行的相应操作提供所有必要的同步。
在一个简单的计数器的例子中，我们可以使用`AtomicInteger`类，它具有原子的递增操作:

```kotlin
var counter = AtomicInteger()

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) {
        counter.incrementAndGet()
    }
    println("Counter = ${counter.get()}")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-sync-03.kt)得到完整的代码

对于这个特殊的问题，这是最快的解决方案。
它适用于普通的计数器、集合、队列和其他标准数据结构和基本操作。
但是，它不容易扩展到复杂的状态，也不容易扩展到没有现成的线程安全实现的复杂操作。

### <a name="thread_confinement_fine_grained"></a>细粒度线程约束

线程约束是一种解决共享可变状态问题的方法，在这种状态下，所有对特定共享状态的访问都被限制在一个线程中。
它通常在UI应用程序中使用，所有的UI状态都被限制在单一事件分派/应用程序线程中。
使用单线程上下文可以很容易地应用于协程。

```kotlin
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) { // run each coroutine in CommonPool
        run(counterContext) { // but confine each increment to the single-threaded context
            counter++
        }
    }
    println("Counter = $counter")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-sync-04.kt)得到完整的代码

这段代码运行非常缓慢，因为它执行细粒度的线程限制。每个单独的递增从多线程的`CommonPool`上下文切换到使用`run`块的单线程上下文。

### <a name="thread_confinement_coarse_grained"></a>粗粒度线程约束

在实践中，线程约束是在大块中执行的，例如，状态更新业务逻辑的大片段被限制在单个线程中。
下面的例子是这样的，在单线程上下文中运行每个协程。

```kotlin
val counterContext = newSingleThreadContext("CounterContext")
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(counterContext) { // run each coroutine in the single-threaded context
        counter++
    }
    println("Counter = $counter")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-sync-05.kt)得到完整的代码

现在的工作速度更快，产生了正确的结果。

### <a name="mutual_exclusion"></a> Mutex

对于这个问题的互斥解决方案是保护共享状态的所有修改，而这一关键部分不会同时执行。
在一个阻塞的世界中，您通常会使用`synchronized`或`ReentrantLock`。
协程的选择被称为 [Mutex](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.sync/-mutex/index.html)。
它有[lock](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.sync/-mutex/lock.html)和[unlock](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.sync/-mutex/unlock.html)函数，以限制临界区。
关键的区别在于`Mutex.lock`是一个挂起函数。
它不会阻塞线程。

还有[withLock](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.sync/with-lock.html)扩展函数，它可以方便地表示mutex.lock(); try { ... } finally { mutex.unlock() } 模式:

```kotlin
val mutex = Mutex()
var counter = 0

fun main(args: Array<String>) = runBlocking<Unit> {
    massiveRun(CommonPool) {
        mutex.withLock {
            counter++        
        }
    }
    println("Counter = $counter")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-sync-06.kt)得到完整的代码

这个例子中的锁定是细粒度的，因此它会付出代价。
但是，在某些情况下，这是一个很好的选择，您必须定期修改某些共享状态，但是这个状态并没有被限制的自然线程。

### <a name="actors"></a>参与者

`参与者(actor)`是一种结合在一起的协程，被限制和压缩到这个协程的状态，以及一个外联的沟通渠道。
一个简单的参与者可以被编写为一个函数，但是一个具有复杂状态的参与者更适合于一个类。

有一个[参与者(actor)](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/actor.html)协程构建器，它可以方便地将参与者的邮箱通道与它的范围相结合，接收消息，并将发送通道组合到产生的job对象中，这样就可以将单个引用作为它的句柄进行传递。

使用`actor`的第一步是定义一个参与者将要处理的消息类。
Kotlin的[sealed classes](https://kotlinlang.org/docs/reference/sealed-classes.html)非常适合这个目的。
我们定义CounterMsg sealed类，并使用`IncCounter`消息增加计数器和`GetCounter`消息以获取其值。
稍后需要发送响应。
一个[CompletableDeferred](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-completable-deferred/index.html)通信原语，表示将来会被知道(通信)的一个单一值，用于此目的。

```kotlin
// 消息类型为counterActor
sealed class CounterMsg
object IncCounter : CounterMsg() // 增量计数器单向消息
class GetCounter(val response: CompletableDeferred<Int>) : CounterMsg() // 一个请求与应答
```

然后，我们定义了一个函数，该函数使用参与者协程构建器启动一个参与者:

```kotlin
// 这个函数将启动一个新的计数器参与者
fun counterActor() = actor<CounterMsg> {
    var counter = 0 // actor state
    for (msg in channel) { // 遍历传入的消息
        when (msg) {
            is IncCounter -> counter++
            is GetCounter -> msg.response.complete(counter)
        }
    }
}
```

主程序很简单:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val counter = counterActor() // create the actor
    massiveRun(CommonPool) {
        counter.send(IncCounter)
    }
    // send a message to get a counter value from an actor
    val response = CompletableDeferred<Int>()
    counter.send(GetCounter(response))
    println("Counter = ${response.await()}")
    counter.close() // shutdown the actor
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-sync-07.kt)得到完整的代码

参与者自己在什么环境中被执行,这并不重要(对于正确性)。一个参与者是一个协程，而一个协程是按顺序执行的，因此将状态限制到特定的协程是解决共享可变状态问题的一个解决方案。

Actor比在负载下锁定更有效，因为在这种情况下，它总是有工作要做，而且它不需要切换到不同的上下文。

> 值得注意的是，一个参与者的协程构建器是一个生产协程构建器的双重角色。一个参与者与它接收消息的通道相关联，而一个生产者与它发送元素的通道相关联。

## <a name="select_expression"></a>Select表达式

Select表达式使得可以同时等待多个挂起函数，并选择第一个可用的函数。

### <a name="selecting_from_channels"></a>选择通道

让我们作两个字符串的生产者: fizz和buzz。fizz生产者每300毫秒就会产生"Fizz"的字符串:

```kotlin
fun fizz(context: CoroutineContext) = produce<String>(context) {
    while (true) { // sends "Fizz" every 300 ms
        delay(300)
        send("Fizz")
    }
}
```

buzz生产者每500毫秒就会产生"Buzz"的字符串:

```kotlin
fun buzz(context: CoroutineContext) = produce<String>(context) {
    while (true) { // sends "Buzz!" every 500 ms
        delay(500)
        send("Buzz!")
    }
}
```

我们可以从一个通道或另一个通道接收挂起函数。
但select表达式允许我们同时使用它的onReceive子句:

```kotlin
suspend fun selectFizzBuzz(fizz: ReceiveChannel<String>, buzz: ReceiveChannel<String>) {
    select<Unit> { // <Unit> means that this select expression does not produce any result 
        fizz.onReceive { value ->  // this is the first select clause
            println("fizz -> '$value'")
        }
        buzz.onReceive { value ->  // this is the second select clause
            println("buzz -> '$value'")
        }
    }
}
```

让我们来运行它七次:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val fizz = fizz(coroutineContext)
    val buzz = buzz(coroutineContext)
    repeat(7) {
        selectFizzBuzz(fizz, buzz)
    }
    coroutineContext.cancelChildren() // cancel fizz & buzz coroutines    
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-select-01.kt)得到完整的代码

这段代码的结果是:

```
fizz -> 'Fizz'
buzz -> 'Buzz!'
fizz -> 'Fizz'
fizz -> 'Fizz'
buzz -> 'Buzz!'
fizz -> 'Fizz'
buzz -> 'Buzz!'
```

### <a name="selecting_on_close"></a>关闭时的select

当通道关闭时，`select`中的[onReceive](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-receive-channel/on-receive.html)子句失败，相应的select抛出一个异常。
当通道关闭时，我们可以使用[onReceiveOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-receive-channel/on-receive-or-null.html)子句执行特定的操作。
下面的示例还显示了select是返回其所选子句的结果的表达式:

```kotlin
suspend fun selectAorB(a: ReceiveChannel<String>, b: ReceiveChannel<String>): String =
    select<String> {
        a.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'a' is closed" 
            else 
                "a -> '$value'"
        }
        b.onReceiveOrNull { value -> 
            if (value == null) 
                "Channel 'b' is closed"
            else    
                "b -> '$value'"
        }
    }
```

让我们用通道a生成“Hello”字符串4次，而b通道生成“World”4次:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    // we are using the context of the main thread in this example for predictability ... 
    val a = produce<String>(coroutineContext) {
        repeat(4) { send("Hello $it") }
    }
    val b = produce<String>(coroutineContext) {
        repeat(4) { send("World $it") }
    }
    repeat(8) { // print first eight results
        println(selectAorB(a, b))
    }
    coroutineContext.cancelChildren()    
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-select-02.kt)得到完整的代码

这段代码的结果非常有趣，所以我们将在模式细节上进行分析:

```
a -> 'Hello 0'
a -> 'Hello 1'
b -> 'World 0'
a -> 'Hello 2'
a -> 'Hello 3'
b -> 'World 1'
Channel 'a' is closed
Channel 'a' is closed
```

有几条观察结果可以证明这一点。

首先，`select`对第一个子句有偏爱。
当多个子句同时可选时，其中的第一个子句被选中。
在这里，两个通道都在不断地生成字符串，因此一个通道，作为select中的第一个子句获胜。
然而，由于我们使用的是非缓冲通道，所以`a`在发送调用时被暂时挂起，并给b发送了一个机会。

第二个观察结果是，当通道已经关闭时，[onReceiveOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-receive-channel/on-receive-or-null.html)会立即被选中。

### <a name="selecting_to_send"></a>选择发送

Select表达式有[onSend](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-send-channel/on-send.html)子句，它可以用于很好的组合和选择的偏置性。

让我们来编写一个例子，当在主通道上的消费者无法跟上时，将它的值发送到一个侧通道的整数的生产者。

```kotlin
fun produceNumbers(context: CoroutineContext, side: SendChannel<Int>) = produce<Int>(context) {
    for (num in 1..10) { // 从1到10产生10个数字
        delay(100) // every 100 ms
        select<Unit> {
            onSend(num) {} // 发送到主通道
            side.onSend(num) {} // 或者是侧通道    
        }
    }
}
```

消费者将会非常缓慢，花费250毫秒来处理每一个数字:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val side = Channel<Int>() // allocate side channel
    launch(coroutineContext) { // 这是侧边通道的一个非常快的消费者
        side.consumeEach { println("Side channel has $it") }
    }
    produceNumbers(coroutineContext, side).consumeEach { 
        println("Consuming $it")
        delay(250) // 让我们适当地消化所消耗的数字，不要着急
    }
    println("Done consuming")
    coroutineContext.cancelChildren()    
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-select-03.kt)得到完整的代码

让我们看看会发生什么:

```
Consuming 1
Side channel has 2
Side channel has 3
Consuming 4
Side channel has 5
Side channel has 6
Consuming 7
Side channel has 8
Side channel has 9
Consuming 10
Done consuming
```

### <a name="selecting_deferred_values"></a>选择deferred值

deferred值可以使用[onAwait](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-deferred/on-await.html)子句来选择。让我们从一个异步函数开始，该函数在随机延迟之后返回deferred字符串值:

```kotlin
fun asyncString(time: Int) = async {
    delay(time.toLong())
    "Waited for $time ms"
}
```

让我们用随机的延迟来启动一打:

```kotlin
fun asyncStringsList(): List<Deferred<String>> {
    val random = Random(3)
    return List(12) { asyncString(random.nextInt(1000)) }
}
```

现在，主函数将等待第一个函数完成并计算仍然处于活动状态的deferred值的数量。
注意，我们已经在这里使用了`select`表达式是一个Kotlin DSL，因此我们可以使用任意代码为它提供子句。
在本例中，我们迭代了deferred值的列表，为每个延迟值提供`onAwait`子句。

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val list = asyncStringsList()
    val result = select<String> {
        list.withIndex().forEach { (index, deferred) ->
            deferred.onAwait { answer ->
                "Deferred $index produced answer '$answer'"
            }
        }
    }
    println(result)
    val countActive = list.count { it.isActive }
    println("$countActive coroutines are still active")
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-select-04.kt)得到完整的代码

输出是:

```
Deferred 4 produced answer 'Waited for 128 ms'
11 coroutines are still active
```

### <a name="switch_over_a_channel_of_deferred_values"></a>切换到deferred值的通道

让我们编写一个通道生成器函数，该函数使用一个deferred字符串值的通道，等待每个接收到的deferred值，但直到下一次deferred或通道被关闭时才会出现。
这个示例将[onReceiveOrNull](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-receive-channel/on-receive-or-null.html)和[onAwait](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-deferred/on-await.html)子句放在同一个select子句中:

```kotlin
fun switchMapDeferreds(input: ReceiveChannel<Deferred<String>>) = produce<String> {
    var current = input.receive() // start with first received deferred value
    while (isActive) { // loop while not cancelled/closed
        val next = select<Deferred<String>?> { // return next deferred value from this select or null
            input.onReceiveOrNull { update ->
                update // replaces next value to wait
            }
            current.onAwait { value ->  
                send(value) // send value that current deferred has produced
                input.receiveOrNull() // and use the next deferred from the input channel
            }
        }
        if (next == null) {
            println("Channel was closed")
            break // out of loop
        } else {
            current = next
        }
    }
}
```

为了测试它，我们将使用一个简单的async函数，它在指定的时间之后解析为指定的字符串:

```kotlin
fun asyncString(str: String, time: Long) = async {
    delay(time)
    str
}
```

主函数是启动一个协程来打印开关的结果，并发送一些测试数据给它:

```kotlin
fun main(args: Array<String>) = runBlocking<Unit> {
    val chan = Channel<Deferred<String>>() // the channel for test
    launch(coroutineContext) { // launch printing coroutine
        for (s in switchMapDeferreds(chan)) 
            println(s) // print each received string
    }
    chan.send(asyncString("BEGIN", 100))
    delay(200) // enough time for "BEGIN" to be produced
    chan.send(asyncString("Slow", 500))
    delay(100) // not enough time to produce slow
    chan.send(asyncString("Replace", 100))
    delay(500) // give it time before the last one
    chan.send(asyncString("END", 500))
    delay(1000) // give it time to process
    chan.close() // close the channel ... 
    delay(500) // and wait some time to let it finish
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/core/kotlinx-coroutines-core/src/test/kotlin/guide/example-select-05.kt)得到完整的代码

输出是:

```
BEGIN
Replace
END
Channel was closed
```

## <a name="further_reading"></a>进一步的阅读

* [使用协程的UI编程指南](/2017/11/06/GuideUIProgrammingCoroutines/)
* [使用协程响应式流的指南](https://github.com/Kotlin/kotlinx.coroutines/blob/master/reactive/coroutines-guide-reactive.md)
* [协程设计文档(保持)](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)
* [完整的kotlinx.coroutines API参考](http://kotlin.github.io/kotlinx.coroutines)