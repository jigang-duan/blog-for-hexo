---
title: 使用协程的UI编程指南
date: 2017-11-06 19:08:01
tags:
- kotlinx.coroutines
- Android
- Kotlin
categories:
- Kotlin

---

![kotlin](http://images2015.cnblogs.com/blog/831804/201603/831804-20160323231701386-1892583268.png)

本指南假定读者熟悉基本的协程概念，并给出了在UI应用程序中如何使用kotlinx.coroutines的具体示例。

所有的UI应用程序库都有一个共同点。
它们有一个线程，所有UI的状态都被限制，所有的UI更新都必须在这个特定的线程中发生。
对于协程，这意味着您需要一个适当的协程分派器上下文，它将协程执行限制在这个UI线程中。

特别是，kotlinx.coroutines 有三个模块为不同的UI应用程序库提供了协程上下文:

* [kotlinx-coroutines-android](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-android) -- Android应用程序的UI上下文
* [kotlinx-coroutines-javafx](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx) -- JavaFx UI应用程序的JavaFx上下文
* [kotlinx-coroutines-swing ](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-swing)-- Swing UI应用程序的Swing上下文

这个指南同时涵盖了所有的UI库，因为每个模块只包含一个对象定义，而这个对象定义只有几页长。
您可以使用它们中的任何一个作为示例，为您最喜欢的UI库编写相应的上下文对象，即使它没有包含在这个框中

<!-- more -->

## 目录 ##

* [设置](#setup)
	* [JavaFx](#javafx)
	* [Android](#android)
* [基本的UI协程](#basic_ui_coroutiones)
	* [启动UI协程](#launch_ui_coroutine)
	* [取消UI协程](#cancel_ui_coroutine)
* [在UI上下文中使用参与者](#using_actors_within_ui_context)
	* [扩展协程](#extensions_for_coroutines)
	* [最多协程job](#at_most_one_concurrent_job)
	* [事件合并](#event_conflation)
* [阻塞操作](#blocking_operations)
	* [UI冻结的问题](#the_problem_of_ui_freezes)
	* [阻塞操作](#blocking_operations)
* [高级的主题](#advanced_topics)
	* [生命周期和协程父子层次结构](#lifecycle_and_coroutine_parent_child_hierarchy)
	* [在没有分派的情况下，在UI事件处理程序中启动协程](#starting_coroutine_in_ui_event_handlers_withoutdispatch)

## <a name="setup"></a>设置

本指南中的可运行示例是为JavaFx提供的。
这样做的好处是，所有的示例都可以直接在任何OS上启动，而不需要模拟器或类似的操作，而且它们是完全自包含的(每个示例都在一个文件中)。
对于需要在Android上进行复制(如果有的话)需要做什么改变，有一些单独的注释。

### <a name="javafx"></a>JavaFx

JavaFx的基本示例应用程序包含一个窗口，其中包含一个名为hello的文本标签，它最初包含“hello World！”在右下角的字符串和一个粉红色的圆圈命名为fab(浮动操作按钮)

![ui-example-javafx.png](https://github.com/Kotlin/kotlinx.coroutines/raw/master/ui/ui-example-javafx.png)

JavaFX应用程序的`start`函数调用`setup`函数，将其引用传递给`hello`和`fab`节点。这就是在本指南的其他部分中放置各种代码的地方:

```kotlin
fun setup(hello: Text, fab: Circle) {
    // placeholder
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-basic-01.kt)得到完整的代码

您可以从GitHub上复制[kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines)项目到您的工作站，并在IDE中打开项目。
本指南的所有示例都在[ui/kotlinx-coroutinjavafx-javafx](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx)模块的测试文件夹中。
通过这种方式，您将能够运行并查看每个示例的工作方式，并通过进行更改来进行试验。

### <a name="android"></a>Android

按照[Android和Kotlin的入门指导](https://kotlinlang.org/docs/tutorials/kotlin-android.html)，在Android Studio中创建Kotlin项目。您还可以在应用程序中添加[Kotlin Android扩展](https://kotlinlang.org/docs/tutorials/android-plugin.html)。

在Android Studio 2.3中，你会得到一个与下图相似的应用程序:

![ui-example-android.png](https://github.com/Kotlin/kotlinx.coroutines/raw/master/ui/ui-example-android.png)

转到应用程序的context_main.xml，并将“hello”的ID分配给text view，“hello World！”string，因此它可以在你的应用中作为hello，与Kotlin Android扩展一起使用。
在创建的项目模板中，粉红色的浮动操作按钮已经被命名为fab。

在您的应用程序的`MainActivity.kt`中，删除了块`fab.setOnClickListener { ... }`，并将`setup(hello，fab)`调用添加为`onCreate`函数的最后一行。
在文件的末尾创建一个占位符设置函数。
这就是在本指南的其他部分中放置各种代码的地方:

```kotlin
fun setup(hello: TextView, fab: FloatingActionButton) {
    // placeholder
}
```

将对kotlinx-coroutines-android模块的依赖关系添加到app/build.gradle文件的dependencies { ... } 部分:

```
compile "org.jetbrains.kotlinx:kotlinx-coroutines-android:0.19.3"
```

协程是Kotlin的实验特征。您需要在Kotlin编译器中启用协程，将以下行添加到gradle.properties文件:

```
kotlin.coroutines=enable
```

您可以从GitHub上克隆kotlinx.coroutines项目到您的工作站。
Android的最终模板项目是在ui/kotlinx-coroutines-android/exam-app目录中。
你可以在Android Studio中加载它，以遵循Android的这个指南。

## <a name="basic_ui_coroutiones"></a>基本的UI协程

本节展示了在UI应用程序中使用协程的基本用法。

### <a name="launch_ui_coroutine"></a>启动UI协程

kotlinx-coroutines- JavaFx模块包含[JavaFx](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-javafx/kotlinx.coroutines.experimental.javafx/-java-fx/index.html)上下文，它将协程执行分派给JavaFx应用程序线程。
我们将其导入为UI，以使所有示例易于移植到Android:

```kotlin
import kotlinx.coroutines.experimental.javafx.JavaFx as UI
```

被限制在UI线程的协程可以自由地在UI中更新任何东西，并在不阻塞UI线程的情况下挂起。
例如，我们可以用命令式的方式编写动画。
以下代码以每秒10到1秒的时间更新文本，使用[launch](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/launch.html) 协程构建器:

```kotlin
fun setup(hello: Text, fab: Circle) {
    launch(UI) { // launch coroutine in UI context
        for (i in 10 downTo 1) { // countdown from 10 to 1 
            hello.text = "Countdown $i ..." // update text
            delay(500) // wait half a second
        }
        hello.text = "Done!"
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-basic-02.kt)得到完整的代码

所以,这里发生了什么?
因为我们在UI上下文中启动了协程，我们可以从这个协程中自由地更新UI，并调用延迟的函数，比如延迟。
当延迟等待时，UI不会被冻结，因为它不会阻塞UI线程——它只是挂起了协程。

> Android应用程序的相应代码是相同的。
> 您只需要将setup函数的主体复制到Android项目的相应函数中。

### <a name="cancel_ui_coroutine"></a>取消UI协程

我们可以对`launch`函数返回的[Job](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/index.html)对象进行引用，当我们想要停止时，可以使用它来取消协程。
当点击桃色的圆圈时，让我们取消协程:


```kotlin
fun setup(hello: Text, fab: Circle) {
    val job = launch(UI) { // launch coroutine in UI context
        for (i in 10 downTo 1) { // countdown from 10 to 1 
            hello.text = "Countdown $i ..." // update text
            delay(500) // wait half a second
        }
        hello.text = "Done!"
    }
    fab.onMouseClicked = EventHandler { job.cancel() } // cancel coroutine on click
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-basic-03.kt)得到完整的代码

现在，如果在倒计时还在运行的时候，圆圈被单击，倒计时就停止了。
注意,[Job.cancel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-job/cancel.html)是完全线程安全的和非阻塞的。
它只是向协程发出信号，以取消它的工作，而不等待它真正终止。
它可以从任何地方调用。
在已经取消或已经完成的协程上调用它，什么也不做。

> Android对应的线如下图所示:

```kotlin
fab.setOnClickListener { job.cancel() }  // 在点击取消协程
```

## <a name="using_actors_within_ui_context"></a>在UI上下文中使用参与者

在本节中，我们将展示UI应用程序如何在其UI上下文中使用参与者(actors)，确保在启动的协程s的数量上没有无限制的增长。

### <a name="extensions_for_coroutines"></a>扩展协程

我们的目标是编写一个名为`onClick`的扩展*协程构建器*函数，这样每当单击时，我们就可以执行这个简单代码倒计时动画:

```kotlin
fun setup(hello: Text, fab: Circle) {
    fab.onClick { // start coroutine when the circle is clicked
        for (i in 10 downTo 1) { // countdown from 10 to 1 
            hello.text = "Countdown $i ..." // update text
            delay(500) // wait half a second
        }
        hello.text = "Done!"
    }
}
```

`onClick`的第一个实现是在每个鼠标事件上启动一个新的协程，并将相应的鼠标事件传递给所提供的操作(以防我们需要它):

```kotlin
fun Node.onClick(action: suspend (MouseEvent) -> Unit) {
    onMouseClicked = EventHandler { event ->
        launch(UI) {
            action(event)
        }
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-actor-01.kt)得到完整的代码

注意，每次单击圆圈时，它启动一个新的协程，它们都在竞争更新文本。试一试。看起来不太好。以后我们会修复它。

> 在Android上，可以为视图类编写相应的扩展，这样在上面所示的setup函数中的代码可以不用修改就可以使用。
> Android上没有MouseEvent，所以省略了。

```kotlin
fun View.onClick(action: suspend () -> Unit) {
    setOnClickListener { 
        launch(UI) {
            action()
        }
    }
}
```

### <a name="at_most_one_concurrent_job"></a>最多协程job

在开始新的Job之前，我们可以取消一份积极的Job，以确保在大多数情况下，协程正在对倒计时进行动画。
然而，这通常不是最好的主意。
`cancel`函数只是作为一个信号来中止一个协程。
取消是合作的，而协程此时可能正在做一些不可撤销的事情，或者忽略了一个取消信号。
更好的解决方案是使用参与者([actor](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/actor.html))来执行不应该同时执行的任务。
让我们改变onClick扩展实现:

```kotlin
fun Node.onClick(action: suspend (MouseEvent) -> Unit) {
    // launch one actor to handle all events on this node
    val eventActor = actor<MouseEvent>(UI) {
        for (event in channel) action(event) // pass event to action
    }
    // install a listener to offer events to this actor
    onMouseClicked = EventHandler { event ->
        eventActor.offer(event)
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-actor-02.kt)得到完整的代码

下面的关键思想是一个参与者(actor)协程和一个常规事件处理程序的集成，那就是在[SendChannel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-send-channel/index.html)上有一个不等待的[offer](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-send-channel/offer.html)函数。
如果可能的话，它会立即向参与者(actor)发送一个元素，否则就会丢弃一个元素。
一个`offer`实际上返回一个我们忽略的`布尔`结果。

> 在Android上，没有MouseEvent，所以我们只是将一个Unit发送给actor作为一个信号。
> 视图类的相应扩展如下:

```kotlin
fun View.onClick(action: suspend () -> Unit) {
    // launch one actor
    val eventActor = actor<Unit>(UI) {
        for (event in channel) action()
    }
    // install a listener to activate this actor
    setOnClickListener { 
        eventActor.offer(Unit)
    }
}
```

### <a name="event_conflation"></a>事件合并

有时处理最近的事件更合适，而不是在我们忙于处理前一个事件时忽略事件。
参与者协程构建器接受一个可选的容量参数，该参数控制该参与者在其邮箱中使用的通道的实现。
在 [Channel()](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-channel/index.html)工厂函数的文档中给出了所有可用选项的描述。

让我们修改代码以通过Channel.CONFLATED容量值来使用ConflatedChannel。
这种变化只会产生一个参与者:

```kotlin
fun Node.onClick(action: suspend (MouseEvent) -> Unit) {
    // launch one actor to handle all events on this node
    val eventActor = actor<MouseEvent>(UI, capacity = Channel.CONFLATED) { // <--- Changed here
        for (event in channel) action(event) // pass event to action
    }
    // install a listener to offer events to this actor
    onMouseClicked = EventHandler { event ->
        eventActor.offer(event)
    }
}
```

您可以在这里得到完整的JavaFx代码。
在Android上，你需要更新val eventActor =…从上一个例子中得到的行。

现在，如果在动画运行时单击圆，它将在结束后重新启动动画。
只有一次。
当动画运行时，重复的点击量会被合并，只有最近的事件才会被处理。

这也是UI应用程序需要的一种行为，这些应用程序必须根据最近收到的更新来更新它们的UI，从而对传入的高频事件流做出反应。
使用ConflatedChannel的协程可以避免由于缓存事件而导致的延迟。

您可以在上面一行中进行`capacity`参数的实验，以了解它如何影响代码的行为。
设置`capacity = Channel.UNLIMITED`创建一个与[LinkedListChannel](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental.channels/-linked-list-channel/index.html)邮箱关联的协程，它可以缓冲所有事件。
在这种情况下，动画会在单击圆圈时多次运行。

## <a name="blocking_operations"></a>阻塞操作

本节解释如何使用带有线程阻塞操作的UI协程。

### <a name="the_problem_of_ui_freezes"></a>UI冻结的问题

如果所有的api都被写为挂起执行线程的函数，那就太好了。
然而，事实并非如此。
有时您需要进行cpu消耗计算，或者仅仅需要调用一些第三方api来访问网络，例如，这样可以阻止调用线程。
您不能直接从UI线程或UI限制的协程完成，因为这会阻塞UI线程并导致UI的冻结。

下面的例子说明了这个问题。
我们将使用onClick扩展和上一节中使用UI限制 事件合并 actor来处理UI线程中的最后一次点击。
对于这个例子，我们将对[斐波那契数](https://en.wikipedia.org/wiki/Fibonacci_number)进行简单的计算:

```kotlin
fun fib(x: Int): Int =
    if (x <= 1) 1 else fib(x - 1) + fib(x - 2)
```

每次点击圆圈时，我们都会计算更大的斐波那契数。
为了使UI更加明显，还有一个快速计数动画，它总是在运行，并且不断地更新UI上下文中的文本:

```kotlin
fun setup(hello: Text, fab: Circle) {
    var result = "none" // the last result
    // counting animation 
    launch(UI) {
        var counter = 0
        while (true) {
            hello.text = "${++counter}: $result"
            delay(100) // update the text every 100ms
        }
    }
    // compute the next fibonacci number of each click
    var x = 1
    fab.onClick {
        result = "fib($x) = ${fib(x)}"
        x++
    }
}
```

> 您可以在这里得到完整的JavaFx代码。
> 你可以将fib函数和setup函数的主体复制到Android项目中。

试着在这个例子中点击圆圈。
在大约30 - 40次点击之后，我们的天真计算将会变得非常缓慢，你会立刻看到UI线程是如何冻结的，因为动画在UI冻结期间停止运行。

### <a name="blocking_operations"></a>阻塞操作

在UI线程上的阻塞操作的修复非常简单，用协程。
我们将把我们的“阻塞”fib函数转换为一个非阻塞的挂起函数，通过使用run函数将它的执行上下文更改到后台线程的CommonPool，从而在后台线程中运行计算。
注意，fib函数现在被标记为`suspend`修饰符。
它不会阻塞已被调用的协程，但在后台线程的计算工作时暂停执行。

```kotlin
suspend fun fib(x: Int): Int = run(CommonPool) {
    if (x <= 1) 1 else fib(x - 1) + fib(x - 2)
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-blocking-02.kt)得到完整的代码

您可以运行此代码，并验证UI不是冻结的，而大量的斐波那契数字正在计算中。
然而，这段代码计算fib稍微慢一些，因为对fib的每次递归调用都经过了`run`。
这在实践中并不是一个大问题，因为`运行`足够智能，可以检查协程已经在所需的上下文中运行，并且避免了将协程分派到另一个线程上的开销。
尽管如此，它仍然是一个开销，在这个原始代码中可以看到，它只在调用之间增加了`run`的整数。
对于一些更重要的代码，额外`run`调用的开销不会很大。

但是，这个特殊的`fib`实现可以像以前一样快速地运行，但是在后台线程中，通过将原来的`fib`函数重命名为`fibBlocking`，并在`fibBlocking`上定义`fib`和`run`包装:

```kotlin
suspend fun fib(x: Int): Int = run(CommonPool) {
    fibBlocking(x)
}

fun fibBlocking(x: Int): Int = 
    if (x <= 1) 1 else fibBlocking(x - 1) + fibBlocking(x - 2)
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-blocking-03.kt)得到完整的代码

您现在可以在不阻塞UI线程的情况下享受全速天真的斐波那契运算。我们只需要运行(`CommonPool`)。

注意，由于fib函数是由代码中的单个参与者调用的，因此在任何给定的时间内都有一个并发计算，因此这段代码对资源的利用率有一个自然的限制。
它可以在大多数CPU核心中饱和。

## <a name="advanced_topics"></a>高级的主题

本节介绍各种高级主题。

### <a name="lifecycle_and_coroutine_parent_child_hierarchy"></a>生命周期和协程父子层次结构

一个典型的UI应用程序有许多具有生命周期的元素。
Windows、UI控件、activities、views、fragments和其他可视化元素被创建和销毁。
一个长时间运行的协程，执行一些IO或后台计算，可以保留对相应UI元素的引用，时间超过所需的时间，
防止所有已被破坏且不再显示的UI对象树的垃圾回收。

这个问题的自然解决方案是将一个Job对象与每个具有生命周期的UI对象相关联，并在该Job的上下文中创建所有的协程。

例如，在Android应用程序中，一个Activity最初被创建，当它不再需要时被销毁，当它的内存必须被释放时被销毁。
一个自然的解决方案是将Job的实例附加到Activity的实例上。
我们可以通过定义以下JobHolder接口来创建一个小型框架:

```kotlin
interface JobHolder {
    val job: Job
}
```

现在，一个与job相关的activity需要实现这个JobHolder接口，并定义其onDestroy函数来取消对应的作业:

```kotlin
class MainActivity : AppCompatActivity(), JobHolder {
    override val job: Job = Job() // 该活动的job实例

    override fun onDestroy() {
        super.onDestroy()
        job.cancel() // 当activity被破坏时取消job
    }
 
    // the rest of code
}
```

我们还需要一个方便的方法来检索应用程序中的任何视图的Job。
这很简单，因为一个Job是它的视图的一个Android上下文，所以我们可以定义如下的View.contextJob扩展属性:

```kotlin
val View.contextJob: Job
    get() = (context as? JobHolder)?.job ?: NonCancellable
```

在这里，我们使用[NonCancellable](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.experimental/-non-cancellable/index.html)的`Job`实现作为一个空对象，因为我们的`contextJob`扩展属性是在没有附加`Job`的上下文中调用的。

有一个可以使用的`contextJob`的便利是，我们可以简单地使用它来启动所有的协程，而不用担心显式地维护我们已经开始的协程的列表。
所有的生命周期管理将由Job之间的亲子关系的机制来处理。

例如，上一节的View.onClick扩展现在可以使用contextJob来定义:

```kotlin
fun View.onClick(action: suspend () -> Unit) {
    // launch one actor as a parent of the context job
    val eventActor = actor<Unit>(contextJob + UI, capacity = Channel.CONFLATED) {
        for (event in channel) action()
    }
    // install a listener to activate this actor
    setOnClickListener {
        eventActor.offer(Unit)
    }
}
```

注意，在上面的代码中如何使用contextJob + UI表达式来启动一个参与者。
它为我们的新参与者定义了一个协程上下文，包括job和UI分派器。
由这个actor(contextJob + UI)表达式启动的协程将会成为相应上下文作业的一个孩子。
当活动被破坏，它的工作被取消时，所有的孩子都被取消了。

jobs之间的父子关系形成了一个层次结构。
代表视图和在其上下文中执行某些后台Job的协程可以创建更多的子节点。
当父job被取消时，整棵树都被取消了。
其中一个例子是在《协程指南》的["子协程"]()部分。

### <a name="starting_coroutine_in_ui_event_handlers_withoutdispatch"></a>在没有分派的情况下，在UI事件处理程序中启动协程

让我们在设置中编写下面的代码，以可视化从UI线程启动协程时执行的顺序:

```kotlin
fun setup(hello: Text, fab: Circle) {
    fab.onMouseClicked = EventHandler {
        println("Before launch")
        launch(UI) { 
            println("Inside coroutine")
            delay(100)
            println("After delay")
        } 
        println("After launch")
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-advanced-01.kt)得到完整的代码

当我们启动这段代码并点击一个粉红色的圆圈时，以下消息被打印到控制台:

```
Before launch
After launch
Inside coroutine
After delay
```

您可以看到，`launch`之后立即执行，而coroutine则被发布到UI线程上，以便稍后执行。
`kotlinx.coroutines`的所有UI调度程序都是这样实现的。
为什么如此?

基本上，这里的选择是“js风格”的异步方法(异步操作总是延迟到稍后在调度线程中执行)和“c#风格”的方法(在invoker线程中执行异步操作，直到第一个挂起点)。
c#方法似乎更有效率,它最终建议像“使用收益率如果需要....”。
这是容易出错的。
js风格的方法更一致，不需要程序员考虑是否需要让步。

然而，在这个特殊的情况下，当协程从一个事件处理程序开始，并且没有其他的代码时，这个额外的分派确实增加了额外的开销而不带来任何额外的值。
在这种情况下，可以使用一个可选的CoroutineStart参数启动，async和actor协程构建器可以用于性能优化。
将其设置为CoroutineStart.UNDISPATCHED的值，将立即开始执行协程，直到其第一个挂起点，如下示例所示:

```kotlin
fun setup(hello: Text, fab: Circle) {
    fab.onMouseClicked = EventHandler {
        println("Before launch")
        launch(UI, CoroutineStart.UNDISPATCHED) { // <--- Notice this change
            println("Inside coroutine")
            delay(100)                            // <--- And this is where coroutine suspends      
            println("After delay")
        }
        println("After launch")
    }
}
```

> 你可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/ui/kotlinx-coroutines-javafx/src/test/kotlin/guide/example-ui-advanced-02.kt)得到完整的代码

当我们启动这段代码并点击一个粉红色的圆圈时，以下消息被打印到控制台:

```
Before launch
Inside coroutine
After launch
After delay
```