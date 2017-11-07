---
title: RxSwift 入门指南
date: 2017-10-26 12:28:14
tags:
- rxswift
- swift
- iOS
categories:
- rxswift

---

![reactivex.io](http://reactivex.io/assets/reactivex_bg.jpg)

`RxSwift`试图与[ReactiveX.io](http://reactivex.io/)保持一致.在`RxSwift`的情况下，通用跨平台的文档和教程也应该是有效的.

1. [Observables又叫序列](#observables-aka-sequences)
1. [Disposing](#disposing)
1. [`Observable`的隐式保障](#implicit-observable-guarantees)
1. [创造你自己的 `Observable` (observable序列)](#creating-your-own-observable-aka-observable-sequence)
1. [创造一个 `Observable` 执行工作](#creating-an-observable-that-performs-work)
1. [共享订阅和`shareReplay`操作符](#sharing-subscription-and-sharereplay-operator)
1. [操作符](#operators)
1. [Playgrounds](#playgrounds)
1. [自定义操作符](#custom-operators)
1. [错误处理](#error-handling)
1. [调试编译错误](#debugging-compile-errors)
1. [调试](#debugging)
1. [调试内存泄漏](#debugging-memory-leaks)
1. [启用调试模式](#enabling-debug-mode)
1. [KVO](#kvo)
1. [UI层提示](#ui-layer-tips)
1. [HTTP请求](#making-http-requests)
1. [RxDataSources](#rxdatasources)
1. [Driver](Traits.md#driver)
1. [特征: Driver, Single, Maybe, Completable](Traits.md)
1. [例子](Examples.md)

<!-- more -->

# Observables又叫序列

## 基础知识
观察者模式(`Observable<Element>`序列)和普通序列(`Sequence`)的[等价性](MathBehindRx)是理解Rx最重要的东西。

**每一个`Observable`序列就像是一个序列。`Observable` 对比 Swift的 `Sequence`的关键优势在于，它还可以异步地接收元素。这是RxSwift的核心，这里的文档是关于我们如何扩展这个想法的。**

* `Observable`(`ObservableType`) 相当于 `Sequence`
* `ObservableType.subscribe` 方法是相当于 `Sequence.makeIterator` 方法.
* 观察者(callback)需要通过`ObservableType.subscribe`方法接收序列元素，而不是在返回的迭代器上调用next()。

序列是一个简单而又熟悉的概念，**很容易理解**。

人类是有着巨大视觉纹理的生物。当我们可以很容易地看到一个概念时，它就容易多了。

我们可以从试图模拟每个Rx操作符中的事件状态机到高级别操作，从而提升许多认知.

如果我们不使用Rx，但在模型异步系统，这可能意味着我们的代码中充满了状态机和临时状态，我们需要模拟而不是抽象出来.

列表和序列可能是数学家和程序员学习的第一个概念.

这是一系列的数字:


```
--1--2--3--4--5--6--| // 正常终止
```

另一个序列,字符:

```
--a--b--a--a--a---d---X // 终止在错误上
```

一些序列是有限的，而另一些则是无限的，就像一个按钮点击序列:

```
---tap-tap-------tap--->
```

这些被称为大理石图表。还有更多的大理石图表在 [rxmarbles.com](http://rxmarbles.com).

如果我们把序列语法指定为正则表达式，它看起来就像:

**next* (error | completed)?**

描述如下:

* **序列可以有0个或多个元素.**
* **一旦接收到 `error` 或 `completed` 的事件, 序列就无法生成任何其他元素.**

Rx中的序列由一个push接口(即callback)描述.

```swift
enum Event<Element>  {
    case next(Element)      // 序列的下一个元素
    case error(Swift.Error) // 序列失败的错误
    case completed          // 成功终止序列
}

class Observable<Element> {
    func subscribe(_ observer: Observer<Element>) -> Disposable
}

protocol ObserverType {
    func on(_ event: Event<Element>)
}
```

**当一个序列发送`completed`或`error`事件时，所有计算序列元素的内部资源将被释放.**

**要立即取消序列元素和释放资源的产生，请在返回的订阅中调用`dispose`.**

如果一个序列在有限的时间内终止，则不调用`dispose`或不使用`disposed(by: disposeBag)`不会造成任何永久性的资源泄漏。但是，这些资源将被使用，直到序列完成，要么完成元素的生产，要么返回一个错误。

如果一个序列没有自动终止，例如使用一个按钮点击的系列，资源将被永久地分配，除非是手动调用`dispose`，自动地在`disposeBag`内，使用`takeUntil` 操作符,或者以另一种方式进行操作。

**使用`disposeBag`或`takeUntil`操作符是一种可靠的方法，确保资源被清理干净。我们建议在生产环境中使用它们,即使序列在有限时间内终止**

如果你好奇为什么`Swift.Error`不使用一般的错误类型，你可以在[这里](DesignRationale.md#why-error-type-isnt-generic)找到解释.

## Disposing

有一种额外的方法可以被observed序列终止。当我们完成一个序列，并且我们想要释放 分配用来计算 即将到来的元素的所有资源时，我们可以在订阅中调用`dispose`。

这是一个带有`interval`操作符的例子.

```swift
let scheduler = SerialDispatchQueueScheduler(qos: .default)
let subscription = Observable<Int>.interval(0.3, scheduler: scheduler)
    .subscribe { event in
        print(event)
    }

Thread.sleep(forTimeInterval: 2.0)

subscription.dispose()
```

将打印:

```
0
1
2
3
4
5
```

注意，您通常不希望手动调用`dispose`;这只是一个教育的例子。手动调用dispose通常是一种糟糕的代码气味。有更好的方式来处理订阅，比如`DisposeBag`，或者`takeUntil`操作符，或者其他的一些机制。

那么，在执行了`dispose`调用之后，这段代码可以打印出什么东西吗?答案是:这取决于

* 如果 `scheduler`  是一个 **串行调度程序器** (ex. `MainScheduler`) 并且 `dispose`被调用**在相同的串行调度程序器**, 答案是**no**.

* 否则它是 **yes**.

你可以找到更多关于调度器的信息 [这里](Schedulers.md).

你只需要并行地进行两个过程.

* 一个是生产元素
* 另一种是处理订阅

问题是不同的调度器在这些不同过程中“什么东西可以印出来?”都没有什么意义 “在不同的调度程序中，这些过程是不合理的。”

再来几个例子 (`observeOn` 的解释在 [这里](Schedulers.md)).

如果我们有类似的东西:

```swift
let subscription = Observable<Int>.interval(0.3, scheduler: scheduler)
            .observeOn(MainScheduler.instance)
            .subscribe { event in
                print(event)
            }

// ....

subscription.dispose() // 从主线程调用

```

**在`dispose`调用返回之后，没有任何东西会被打印出来。这是保证的.**

同时,在这种情况下:

```swift
let subscription = Observable<Int>.interval(0.3, scheduler: scheduler)
            .observeOn(serialScheduler)
            .subscribe { event in
                print(event)
            }

// ...

subscription.dispose() // 执行在相同的`serialScheduler`

```

**在`dispose`调用返回之后，没有任何东西会被打印出来。这是保证的.**

### Dispose Bags

Dispose bags就像RX的ARC行为.

当一个`DisposeBag`被释放时，它将调用`dispose` 对每一个添加了disposables项的进行处理.

它没有`dispose`方法，因此不允许明确地显式调用。如果需要立即进行清理，我们可以创建一个新bag。

```swift
  self.disposeBag = DisposeBag()
```

这将清除旧的引用，并导致资源的处置.

If that explicit manual disposal is still wanted, use `CompositeDisposable`. **It has the wanted behavior but once that `dispose` method is called, it will immediately dispose any newly added disposable.**

### Take until

在dealloc中自动处理订阅的另一种方法是使用`takeUntil`操作符.

```swift
sequence
    .takeUntil(self.rx.deallocated)
    .subscribe {
        print($0)
    }
```

## `Observable`的隐式保障

还有一些额外的保证，所有序列的生产者(`Observable`s)都必须遵守.

它们在哪个线程上生成元素无关紧要，但是如果它们生成一个元素并将其发送给观察者`observer.on(.next(nextElement))`, 它们不能发送下一个元素,直到`observer.on`方法已完成.

生产者也不能发送终止`.completed或`.error`在 `.next`事件还没有结束的情况.

简而言之，考虑一下这个例子:

```swift
someObservable
  .subscribe { (e: Event<Element>) in
      print("Event processing started")
      // processing
      print("Event processing ended")
  }
```

总是会打印:

```
Event processing started
Event processing ended
Event processing started
Event processing ended
Event processing started
Event processing ended
```

它不能打印:

```
Event processing started
Event processing started
Event processing ended
Event processing ended
```

## 创造你自己的 `Observable` (observable序列)

关于observables有一件重要的事情要理解.

**当一个observable被创建时，它不会仅仅因为它已经被创建而执行任何工作.**

的确，`Observable`可以有许多方法产生元素。它们中的一些会产生副作用，其中一些会利用现有的运行过程，比如点击鼠标事件等等.

**但是，如果您只是调用一个`Observable`的方法，那么就不会执行任何序列生成，并且没有副作用。`Observable`只是定义了序列是如何生成的，以及为元素生成使用了哪些参数。当调用了订阅方法时，序列生成才开始.**

假设，你有一个类似prototype的方法:

```swift
func searchWikipedia(searchTerm: String) -> Observable<Results> {}
```

```swift
let searchForMe = searchWikipedia("me")

// 没有执行请求，没有工作，没有发送URL请求

let cancel = searchForMe
  // 序列生成现在开始，URL请求被触发
  .subscribe(onNext: { results in
      print(results)
  })

```

有很多方法可以创建你自己的`Observable` 序列。最简单的方法可能是使用create函数.

让我们编写一个函数来创建一个序列，该序列在订阅时返回一个元素。这个函数被称为'just'.

*这是实际的实现*

```swift
func myJust<E>(_ element: E) -> Observable<E> {
    return Observable.create { observer in
        observer.on(.next(element))
        observer.on(.completed)
        return Disposables.create()
    }
}

myJust(0)
    .subscribe(onNext: { n in
      print(n)
    })
```

将打印:

```
0
```

那么什么是`create`函数呢?

它只是一种方便的方法，它使您能够使用Swift闭包方便地实现`subscribe`方法。就像 `subscribe`方法一样，它需要一个参数，一个`observer`，然后返回`disposable `.

这样实现的顺序实际上是同步的。它将生成元素，并在终止之前`subscribe`调用返回disposable。因为它的返回并不重要所以生成元素的过程不会被打断.

在生成同步序列时，通常使用的disposable返回是`NopDisposable`实例.

现在让我们创建一个可以从数组中返回元素的observable.

*实际的实现*

```swift
func myFrom<E>(_ sequence: [E]) -> Observable<E> {
    return Observable.create { observer in
        for element in sequence {
            observer.on(.next(element))
        }

        observer.on(.completed)
        return Disposables.create()
    }
}

let stringCounter = myFrom(["first", "second"])

print("Started ----")

// first time
stringCounter
    .subscribe(onNext: { n in
        print(n)
    })

print("----")

// again
stringCounter
    .subscribe(onNext: { n in
        print(n)
    })

print("Ended ----")
```

将打印:

```
Started ----
first
second
----
first
second
Ended ----
```

## 创造一个 `Observable` 执行工作

好了，现在更有趣了。我们来创建之前例子中使用的`interval`操作符.

*这相当于分派队列调度程序的实际实现*

```swift
func myInterval(_ interval: TimeInterval) -> Observable<Int> {
    return Observable.create { observer in
        print("Subscribed")
        let timer = DispatchSource.makeTimerSource(queue: DispatchQueue.global())
        timer.scheduleRepeating(deadline: DispatchTime.now() + interval, interval: interval)

        let cancel = Disposables.create {
            print("Disposed")
            timer.cancel()
        }

        var next = 0
        timer.setEventHandler {
            if cancel.isDisposed {
                return
            }
            observer.on(.next(next))
            next += 1
        }
        timer.resume()

        return cancel
    }
}
```

```swift
let counter = myInterval(0.1)

print("Started ----")

let subscription = counter
    .subscribe(onNext: { n in
        print(n)
    })


Thread.sleep(forTimeInterval: 0.5)

subscription.dispose()

print("Ended ----")
```

将打印

```
Started ----
Subscribed
0
1
2
3
4
Disposed
Ended ----
```

如果你想写

```swift
let counter = myInterval(0.1)

print("Started ----")

let subscription1 = counter
    .subscribe(onNext: { n in
        print("First \(n)")
    })
let subscription2 = counter
    .subscribe(onNext: { n in
        print("Second \(n)")
    })

Thread.sleep(forTimeInterval: 0.5)

subscription1.dispose()

Thread.sleep(forTimeInterval: 0.5)

subscription2.dispose()

print("Ended ----")
```

将打印:

```
Started ----
Subscribed
Subscribed
First 0
Second 0
First 1
Second 1
First 2
Second 2
First 3
Second 3
First 4
Second 4
Disposed
Second 5
Second 6
Second 7
Second 8
Second 9
Disposed
Ended ----
```

**订阅的每个订阅者通常会生成它自己的单独的元素序列。默认情况下，操作符是无状态的。有很多无状态的操作符比有状态的多.**

## 共享订阅和`shareReplay`操作符

但是，如果您希望多个观察者只从一个订阅中共享事件(元素)，该怎么办呢?

有两件事需要定义。

* 在新用户对它们感兴趣之前,如何处理以前接收到的过去的元素(仅重放最新、重放所有、重放最后n个)
* 如何决定何时触发共享订阅(refCount、手动或其他算法)

通常的选择是`replay(1).refCount()` 也可以 `shareReplay()`.

```swift
let counter = myInterval(0.1)
    .shareReplay(1)

print("Started ----")

let subscription1 = counter
    .subscribe(onNext: { n in
        print("First \(n)")
    })
let subscription2 = counter
    .subscribe(onNext: { n in
        print("Second \(n)")
    })

Thread.sleep(forTimeInterval: 0.5)

subscription1.dispose()

Thread.sleep(forTimeInterval: 0.5)

subscription2.dispose()

print("Ended ----")
```

将打印

```
Started ----
Subscribed
First 0
Second 0
First 1
Second 1
First 2
Second 2
First 3
Second 3
First 4
Second 4
First 5
Second 5
Second 6
Second 7
Second 8
Second 9
Disposed
Ended ----
```

注意，现在只有一个`Subscribed`和`Disposed`事件.

URL observables的行为是一样的.

这就是在Rx中封装HTTP请求的方式。它和interval操作符的模式是一样的.

```swift
extension Reactive where Base: URLSession {
    public func response(_ request: URLRequest) -> Observable<(Data, HTTPURLResponse)> {
        return Observable.create { observer in
            let task = self.dataTaskWithRequest(request) { (data, response, error) in
                guard let response = response, let data = data else {
                    observer.on(.error(error ?? RxCocoaURLError.Unknown))
                    return
                }

                guard let httpResponse = response as? HTTPURLResponse else {
                    observer.on(.error(RxCocoaURLError.nonHTTPResponse(response: response)))
                    return
                }

                observer.on(.next(data, httpResponse))
                observer.on(.completed)
            }

            task.resume()

            return Disposables.create {
                task.cancel()
            }
        }
    }
}
```

## 操作符

在RxSwift中实现了许多操作符.

所有操作符的大理石图表都可以在 [ReactiveX.io](http://reactivex.io/)

几乎所有的运算符都在 [Playgrounds](#).

要使用playgrounds，请打开`Rx.xcworkspace`，构建`RxSwift-macOS` scheme，然后在`Rx.xcworkspace`上打开playgrounds的树视图.

如果你需要一个操作符，不知道怎么找到它，这有一个[操作符树](http://reactivex.io/documentation/operators.html#tree).

### 自定义操作符

如何创建自定义操作符有两种方法.

#### 简单的方法

所有的内部代码都使用了高度优化的操作符，因此它们并不是最好的教程材料。这就是为什么使用标准操作符的原因.

幸运的是，有一种更简单的方法来创建操作符。创建新的运算符实际上是关于创建observables，而之前的章节已经描述了如何做到这一点.

让我们看看如何实现一个未优化的映射操作符.

```swift
extension ObservableType {
    func myMap<R>(transform: @escaping (E) -> R) -> Observable<R> {
        return Observable.create { observer in
            let subscription = self.subscribe { e in
                    switch e {
                    case .next(let value):
                        let result = transform(value)
                        observer.on(.next(result))
                    case .error(let error):
                        observer.on(.error(error))
                    case .completed:
                        observer.on(.completed)
                    }
                }

            return subscription
        }
    }
}
```

现在你可以使用自己的map了:

```swift
let subscription = myInterval(0.1)
    .myMap { e in
        return "This is simply \(e)"
    }
    .subscribe(onNext: { n in
        print(n)
    })
```

将打印:

```
Subscribed
This is simply 0
This is simply 1
This is simply 2
This is simply 3
This is simply 4
This is simply 5
This is simply 6
This is simply 7
This is simply 8
...
```

### Life happens

那么，如果不用自定义操作符来解决某些情况会怎么样呢? 您可以退出单个Rx，在真实世界中执行活动，然后使用`Subject`s将结果再次使用到Rx.

这不是应该经常练习的东西，而且是一种不好的代码味道，但你可以这样做.

```swift
  let magicBeings: Observable<MagicBeing> = summonFromMiddleEarth()

  magicBeings
    .subscribe(onNext: { being in     // exit the Rx monad
        self.doSomeStateMagic(being)
    })
    .disposed(by: disposeBag)

  //
  //  Mess
  //
  let kitten = globalParty(   // calculate something in messy world
    being,
    UIApplication.delegate.dataSomething.attendees
  )
  kittens.on(.next(kitten))   // send result back to rx
  //
  // Another mess
  //

  let kittens = Variable(firstKitten) // again back in Rx monad

  kittens.asObservable()
    .map { kitten in
      return kitten.purr()
    }
    // ....
```

每次你这么做的时候，有人可能会把这个代码写在某个地方:

```swift
  kittens
    .subscribe(onNext: { kitten in
      // do something with kitten
    })
    .disposed(by: disposeBag)
```

所以请尽量不要这样做.

## Playgrounds

如果你不确定其中的一些操作符是如何工作的，[playgrounds](#)包含几乎所有的操作符，都已经准备好了演示他们行为的小例子.

**要使用playgrounds，请打开Rx.xcworkspace，构建RxSwift-macOS scheme，然后在Rx.xcworkspace上打开playgrounds的树视图.**

**要查看playgrounds上的示例结果，请打开`Assistant Editor`。您可以通过点击`View > Assistant Editor > Show Assistant Editor`来打开`Assistant Editor`**

## 错误处理

有两种错误机制.

### observables中的异步错误处理机制

错误处理非常简单。如果一个序列以错误结束，那么所有依赖序列都将以错误结束。这是通常的短路逻辑。

您可以通过使用`catch`操作符来恢复overloads的故障。有各种重载使您能够详细地指定恢复.

还有`retry`操作符，可以在出现错误序列时进行重试.

## 调试编译错误

在编写优雅的rxswift/rxcocoa代码时，您可能非常依赖于编译器来推断`Observable`的类型。这就是为什么Swift很棒的原因之一，但有时也会让人沮丧.

```swift
images = word
    .filter { $0.containsString("important") }
    .flatMap { word in
        return self.api.loadFlickrFeed("karate")
            .catchError { error in
                return just(JSON(1))
            }
      }
```

如果编译器报告在这个表达式的某个地方有一个错误，我建议首先注释返回类型.

```swift
images = word
    .filter { s -> Bool in s.containsString("important") }
    .flatMap { word -> Observable<JSON> in
        return self.api.loadFlickrFeed("karate")
            .catchError { error -> Observable<JSON> in
                return just(JSON(1))
            }
      }
```

如果这不起作用，您可以继续添加更多类型注释，直到将错误本地化.

```swift
images = word
    .filter { (s: String) -> Bool in s.containsString("important") }
    .flatMap { (word: String) -> Observable<JSON> in
        return self.api.loadFlickrFeed("karate")
            .catchError { (error: Error) -> Observable<JSON> in
                return just(JSON(1))
            }
      }
```

**我建议首先注释返回类型和闭包的参数.**

通常在您修复了错误之后，您可以删除类型注释来重新清理您的代码.

## 调试

单独使用调试器是很有用的，但是通常使用`debug`会更有效。`debug`操作符将把所有事件打印到标准输出中，您还可以添加标记这些事件。.

`debug`就像一个探针。这里有一个使用它的例子:

```swift
let subscription = myInterval(0.1)
    .debug("my probe")
    .map { e in
        return "This is simply \(e)"
    }
    .subscribe(onNext: { n in
        print(n)
    })

Thread.sleepForTimeInterval(0.5)

subscription.dispose()
```

将打印

```
[my probe] subscribed
Subscribed
[my probe] -> Event next(Box(0))
This is simply 0
[my probe] -> Event next(Box(1))
This is simply 1
[my probe] -> Event next(Box(2))
This is simply 2
[my probe] -> Event next(Box(3))
This is simply 3
[my probe] -> Event next(Box(4))
This is simply 4
[my probe] dispose
Disposed
```

您还可以轻松地创建`debug`操作符的版本.

```swift
extension ObservableType {
    public func myDebug(identifier: String) -> Observable<Self.E> {
        return Observable.create { observer in
            print("subscribed \(identifier)")
            let subscription = self.subscribe { e in
                print("event \(identifier)  \(e)")
                switch e {
                case .next(let value):
                    observer.on(.next(value))

                case .error(let error):
                    observer.on(.error(error))

                case .completed:
                    observer.on(.completed)
                }
            }
            return Disposables.create {
                   print("disposing \(identifier)")
                   subscription.dispose()
            }
        }
    }
 }
```

### 启用调试模式
使用`RxSwift.Resources`来调试内存泄漏或自动记录所有HTTP请求，您必须启用调试模式.

为了启用调试模式，必须将一个`TRACE_RESOURCES`标志添加到RxSwift目标构建设置中，在_Other Swift Flags_下.

有关如何用Cocoapods & Carthage设置`TRACE_RESOURCES`标志的进一步讨论和指导，请参阅 [#378](https://github.com/ReactiveX/RxSwift/issues/378)

## 调试内存泄漏

在调试模式下，Rx在全局变量`Resources.total`中跟踪所有分配的资源.

如果您想要有一些资源泄漏检测逻辑，最简单的方法就是打印`RxSwift.Resources.total`周期性的输出.

```swift
    /* add somewhere in
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey : Any]? = nil)
    */
    _ = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        .subscribe(onNext: { _ in
            print("Resource count \(RxSwift.Resources.total)")
        })
```

测试内存泄漏的最有效方法是:
* 导航到你的屏幕并使用它
* 导航返回
* 观察初始资源数
* 第二次浏览你的屏幕并使用它
* 导航返回
* 观察最终的资源数

如果在初始资源和最终资源计数之间存在差异，可能会出现内存泄漏.

之所以建议2个导航，是因为第一个导航强制加载了惰性资源.

## Variables

`Variable`s表示一些observable状态。不包含值的`Variable`不能存在，所以需要初始值.

Variable封装[`Subject`](http://reactivex.io/documentation/subject.html)。更具体地说，这是一个`BehaviorSubject`。与`BehaviorSubject`不同，它只显示值接口，因此variable永远不能以错误结束.

它还会在订阅中立即播放它的当前价.

在variable被释放后，它将完成从`.asObservable()`返回observable序列.

```swift
let variable = Variable(0)

print("Before first subscription ---")

_ = variable.asObservable()
    .subscribe(onNext: { n in
        print("First \(n)")
    }, onCompleted: {
        print("Completed 1")
    })

print("Before send 1")

variable.value = 1

print("Before second subscription ---")

_ = variable.asObservable()
    .subscribe(onNext: { n in
        print("Second \(n)")
    }, onCompleted: {
        print("Completed 2")
    })

print("Before send 2")

variable.value = 2

print("End ---")
```

将打印

```
Before first subscription ---
First 0
Before send 1
First 1
Before second subscription ---
Second 1
Before send 2
First 2
Second 2
End ---
Completed 1
Completed 2
```

## KVO

KVO是一种Objective-C机制。这意味着它并不是在考虑类型安全的情况下建立起来的。这个项目试图解决一些问题。

这个库支持KVO的方式有两种。

```swift
// KVO
extension Reactive where Base: NSObject {
    public func observe<E>(type: E.Type, _ keyPath: String, options: KeyValueObservingOptions, retainSelf: Bool = true) -> Observable<E?> {}
}

#if !DISABLE_SWIZZLING
// KVO
extension Reactive where Base: NSObject {
    public func observeWeakly<E>(type: E.Type, _ keyPath: String, options: KeyValueObservingOptions) -> Observable<E?> {}
}
#endif
```

例如如何观察`UIView`的frame.

**警告:UIKit不是KVO兼容的，但这是可行的.**

```swift
view
  .rx.observe(CGRect.self, "frame")
  .subscribe(onNext: { frame in
    ...
  })
```

或

```swift
view
  .rx.observeWeakly(CGRect.self, "frame")
  .subscribe(onNext: { frame in
    ...
  })
```

### `rx.observe`

`rx.observe`更有性能，因为它只是围绕KVO机制的一个简单包装，但它的使用场景更有限

* 它可以用于observe路径开始于来自`self`或来自所有权的祖先 (`retainSelf = false`)
* 它可以用于observe路径开始于从所有权中的后代 (`retainSelf = true`)
* 路径必须只包含 `strong`属性，否则，在dealloc之前，您可能会冒着破坏系统的风险.

如

```swift
self.rx.observe(CGRect.self, "view.frame", retainSelf: false)
```

### `rx.observeWeakly`

`rx.observeWeakly`的速度比`rx.observe`慢一些。观察因为它必须在弱引用的情况下处理对象的交易.

它可以在`rx.observe`的所有情况下使用

* 因为它不会持有observed的目标，它可以用来observe任意的对象，它的所有权关系是未知的
* 它可以用来观察`weak`属性

如

```swift
someSuspiciousViewController.rx.observeWeakly(Bool.self, "behavingOk")
```

### 观察结构

KVO是一种Objective-C机制，因此它非常依赖于`NSValue`.

**RxCocoa为KVO提供了对`CGRect`、`CGSize`和`CGPoint`结构的支持.**

当观察其他结构体时，必须手动从`NSValue`中提取这些结构。

RxCocoa/Foundation/KVORepresentable+CoreGraphics.swift)是如何扩展示例KVO观察机制和`rx.observe*`的方法，通过实现KVORepresentable协议。

## UI层提示

当绑定到UIKit控件时你的`Observable`s 的一些东西在UI层中需要满足.

### 线程

`Observable`s需要在`MainScheduler`(UIThread)上发送值。这只是普通的UIKit/Cocoa需求.

对于您的api来说，在`MainScheduler`上返回结果通常是一个好主意。如果您试图从后台线程中绑定某个东西，在调试构建RxCocoa中通常会抛出一个异常来通知您这一点

要解决这个问题，您需要添加`observeOn(MainScheduler.instance)`.

**URLSession扩展在默认情况下不会返回`MainScheduler`.**

### 错误

你不能将失败绑定到UIKit控件因为这是未定义的行为.

如果你不知道`Observable`可能失败,您可以使用`catchErrorJustReturn(valueThatIsReturnedWhenErrorHappens)`确保它不能失败,**但是，在发生错误之后，底层的序列仍然会完成**.

如果想要的行为是底层序列继续生成元素，则需要某些版本的`retry`操作符.

### 分享订阅

您通常希望在UI层中共享订阅。您不需要单独进行HTTP调用来将相同的数据绑定到多个UI元素.

假设你有这样的东西:

```swift
let searchResults = searchText
    .throttle(0.3, $.mainScheduler)
    .distinctUntilChanged
    .flatMapLatest { query in
        API.getSearchResults(query)
            .retry(3)
            .startWith([]) // clears results on new search term
            .catchErrorJustReturn([])
    }
    .shareReplay(1)              // <- notice the `shareReplay` operator
```

你通常想要的是分享搜索结果。这就是`shareReplay`的意思。

**在UI层中，在转换链的末尾添加`shareReplay`通常是一个很好的经验法则，因为您确实想要共享计算结果。当将`搜索结果`绑定到多个UI元素时，您不希望触发单独的HTTP连接。**

**再看一下`Driver`单元。它被设计为透明地包装这些共享调用，确保在主UI线程上观察到元素，并且没有错误可以绑定到UI。**

## HTTP请求

http请求是人们尝试的第一件事。

首先需要构建一个表示需要完成的工作的`URLRequest`对象.

请求确定是GET请求，还是POST请求，请求主体是什么，查询参数 ...

这就是如何创建一个简单的GET请求

```swift
let req = URLRequest(url: URL(string: "http://en.wikipedia.org/w/api.php?action=parse&page=Pizza&format=json"))
```

如果您想在组合之外执行其他observables的请求，那么这就是需要做的事情。

```swift
let responseJSON = URLSession.shared.rx.json(request: req)

// 没有任何请求会被执行到responseJSON,只是一个描述如何获取响应


let cancelRequest = responseJSON
    // this will fire the request
    .subscribe(onNext: { json in
        print(json)
    })

Thread.sleep(forTimeInterval: 3.0)

// 如果你想在3秒后取消请求
cancelRequest.dispose()

```

**URLSession扩展不会在默认情况下返回`MainScheduler`.**

如果您想要一个更低级别访问响应，您可以使用:

```swift
URLSession.shared.rx.response(myURLRequest)
    .debug("my request") // 将把信息打印到控制台
    .flatMap { (data: NSData, response: URLResponse) -> Observable<String> in
        if let response = response as? HTTPURLResponse {
            if 200 ..< 300 ~= response.statusCode {
                return just(transform(data))
            }
            else {
                return Observable.error(yourNSError)
            }
        }
        else {
            rxFatalError("response = nil")
            return Observable.error(yourNSError)
        }
    }
    .subscribe { event in
        print(event) // 如果发生错误，也会将错误输出到控制台
    }
```
### 日志记录HTTP流量

在调试模式下，RxCocoa将默认将所有HTTP请求记录到控制台。如果您想要改变这种行为，请设置`Logging.URLRequests`的过滤器。.

```swift
// 读自己的配置
public struct Logging {
    public typealias LogURLRequest = (URLRequest) -> Bool

    public static var URLRequests: LogURLRequest =  { _ in
    #if DEBUG
        return true
    #else
        return false
    #endif
    }
}
```

## RxDataSources

... 是一组为UITableView s和UICollectionView提供全功能的响应式数据源的类。

RxDataSources被绑定在 [这里](https://github.com/RxSwiftCommunity/RxDataSources).

在[RxExample](#)项目中包含了如何使用它们的功能完整的演示。
