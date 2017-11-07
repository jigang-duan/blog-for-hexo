---
title: RxSwift
date: 2017-10-24 14:36:35
tags:
- rxswift
- swift
- iOS
categories:
- rxswift

---

![reactivex.io](http://reactivex.io/assets/reactivex_bg.jpg)

1. [介绍](#介绍)
	1. [为什么使用RxSwift](#为什么使用RxSwift)
	1. [概念](#概念)
		- [Observables和observers(观察者)(aka subscribers 订阅者)](#Observables和observers(观察者)(aka subscribers 订阅者))
1. [创建和订阅Observables](#创建和订阅Observables)
	- [never](#never)
	- [empty](#empty)
	- [just](#just)
	- [of](#of)
	- [from](#from)
	- [create](#create)
	- [range](#range)
	- [repeatElement](#repeatElement)
	- [generate](#generate)
	- [deferred](#deferred)
	- [error](#error)
	- [doOn](#doOn)
1. [Subjects](#Subjects)
	- [PublishSubject](#PublishSubject)
	- [ReplaySubject](#ReplaySubject)
	- [BehaviorSubject](#BehaviorSubject)
	- [Variable](#Variable)
1. [合并操作符](#合并操作符)
	* [switchLatest](#switchLatest)
	* [combineLatest](#combineLatest)
	* [zip](#zip)
	* [merge](#merge)
	* [startWith](#startWith)
1. [转换操作符](#转换操作符)
	* [map](#map)
	* [flatMap 和 flatMapLatest](#flatMap)
	* [scan](#scan)
1. [过滤和条件运算符](#)
	* [filter](#)
	* [distinctUntilChanged](#)
	* [elementAt](#)
	* [single](#)
	* [take](#)
	* [takeLast](#)
	* [takeWhile](#)
	* [takeUntil](#)
	* [skip](#)
	* [skipWhile](#)
	* [skipWhileWithIndex](#)
	* [skipUntil](#)
1. [数学运算及聚合操作符](#)
	* [toArray](#)
	* [reduce](#)
	* [concat](#)
1. [连接操作符](#)
1. [错误处理操作符](#)
	* [catchErrorJustReturn](#)
	* [catchError](#)
	* [retry](#)
	* [retry(_:)](#)
1. [调试操作符](#)
	* [debug](#)
	* [RxSwift.Resources.total](#)
1. [使能RxSwift.Resources.total](#)

<!-- more -->

## 介绍 ##

### 为什么使用RxSwift ?

我们编写的绝大多数代码都会涉及到对外部事件的响应。

* 当用户操作一个control时，我们需要编写一个@IBAction方法处理响应。
* 当键盘何时改变位置时，我们需要观察通知来检测。
* 当URL会话响应网络数据时，我们必须提供闭包来执行。
* 我们还可以使用KVO来检测变量的变化。

所有这些不同的方法使我们的代码变得**不必要地复杂**。
如果有一个**一致性的方式**来处理我们的所有的调用/响应，难道不是更好吗?Rx就是这样一个机制。

RxSwift是 [Reactive](http://reactivex.io)（Rx）的正式实现, [大多数主要语言和平台](http://reactivex.io/languages.html)都有。

### 概念

> 每个可观察(Observable)实例都是一个序列(Sequence)。

可观察(Observable)序列与Swift序列的关键优势在于，它还可以异步地接收元素。这就是RxSwift的精髓所在。其他的一切都扩展了这个概念。

* 一个Observable(ObservableType)相当于一个序列(Sequence)。
* ObservableType.subscribe(_:)方法相当于Sequence.makeIterator()。
* ObservableType.subscribe(_:)接收一个observer(观察者)(ObserverType)参数，它将被订阅自动接收由Observable发射的序列事件和元素，而不是在返回的生成器上手动调用next()。

如果一个Observable发出了`next`事件(Event.next(Element))，它可以继续发出更多的事件。
但是，如果Observable发出了一个`error`事件(Event.error(ErrorType))或一个`completed`事件(Event.completed)，Observable序列就不能再向订阅者发出其它的事件了。

顺序语法更简明地解释了这一点:

> next* (error | completed)?

这也可以用图表来更直观地解释:

```
--1--2--3--4--5--6--|----> // "|" = 正常终止

--a--b--c--d--e--f--X----> // "X" = 终止与一个错误

--tap--tap----------tap--> // "|" = 无限期地继续下去，比如按钮按下的序列
```

> 这些图表被称为大理石图表。你可以在[RxMarbles.com](http://rxmarbles.com)上了解更多关于他们的信息。

#### Observables和observers(观察者)(aka subscribers 订阅者)

除非有订阅者，否则Observables不会执行他们的订阅闭包。在下面的示例中，Observable的闭包将永远不会被执行，因为没有订阅者:

``` swift
example("Observable with no subscribers") {
    _ = Observable<String>.create { observerOfString -> Disposable in
        print("This will never be printed")
        observerOfString.on(.next("😬"))
        observerOfString.on(.completed)
        return Disposables.create()
    }
}
```

在下面的例子中，当subscribe(_:)被调用时，闭包将被执行。

``` swift
example("Observable with subscriber") {
  _ = Observable<String>.create { observerOfString in
            print("Observable created")
            observerOfString.on(.next("😉"))
            observerOfString.on(.completed)
            return Disposables.create()
        }
        .subscribe { event in
            print(event)
    }
}
```

subscribe(_:)返回一个`Disposable`实例，该实例表示一个可使用的资源，如订阅。在前面的简单示例中，它被忽略了，但是应该正常地处理它。这通常意味着将它添加到一个DisposeBag实例中。所有的例子都将包括正确的处理，因为，实践是永久的🙂。您可以在[入门指南](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md)的[处理部分](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md#disposing)中了解更多关于此的信息。


## 创建和订阅Observables ##

有几种方法可以创建和订阅Observable序列。

### never

创建一个永不终止且从不发出任何事件的序列。[更多信息](http://reactivex.io/documentation/operators/empty-never-throw.html)

``` swift
example("never") {
    let disposeBag = DisposeBag()
    let neverSequence = Observable<String>.never()

    let neverSequenceSubscription = neverSequence
        .subscribe { _ in
            print("This will never be printed")
    }

    neverSequenceSubscription.disposed(by: disposeBag)
}
```

![never](http://reactivex.io/documentation/operators/images/never.c.png)

### empty

创建一个空的可观察序列，它只会发出一个已完成的事件。[更多信息](http://reactivex.io/documentation/operators/empty-never-throw.html)

``` swift
example("empty") {
    let disposeBag = DisposeBag()

    Observable<Int>.empty()
        .subscribe { event in
            print(event)
        }
        .disposed(by: disposeBag)
}
```

![empty](http://reactivex.io/documentation/operators/images/empty.c.png)

> 这个例子还介绍了连接创建和订阅可观察序列的链接。

### just

用一个元素创建一个可观察序列。[更多信息](http://reactivex.io/documentation/operators/just.html)

``` swift
example("just") {
    let disposeBag = DisposeBag()

    Observable.just("🔴")
        .subscribe { event in
            print(event)
        }
        .disposed(by: disposeBag)
}
```

![just](http://reactivex.io/documentation/operators/images/just.c.png)

### of

用固定数量的元素创建一个可观察序列。

``` swift
example("of") {
    let disposeBag = DisposeBag()

    Observable.of("🐶", "🐱", "🐭", "🐹")
        .subscribe(onNext: { element in
            print(element)
        })
        .disposed(by: disposeBag)
}
```

> 这个例子还介绍了使用subscribe(onNext:)便利方法。不同于subscribe(:)，它订阅了所有事件类型的事件(Next, Error, Completed)，subscribe(onNext:)订阅一种元素，它将忽略Error和Completed的事件，并且只生成Next事件元素。还有subscribe(onError:)和subscribe(on:)的便利方法。和有一个subscribe(onNext:onError:oncomplete:onDisposed:)的方法,你可以对一个或多个事件类型和订阅时由于任何原因终止,或处理:
>
> *Example*
>
> ```
> someObservable.subscribe(
    onNext: { print("Element:", $0) },
    onError: { print("Error:", $0) },
    onCompleted: { print("Completed") },
    onDisposed: { print("Disposed") }
)
> ```
>

### from

从集合中创建一个可观察的序列，如Array、Dictionary或Set。

``` swift
example("from") {
    let disposeBag = DisposeBag()

    Observable.from(["🐶", "🐱", "🐭", "🐹"])
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

> 这个例子还演示了使用默认参数$0而不是显式地命名参数。

![from](http://reactivex.io/documentation/operators/images/from.c.png)

### create

创建一个自定义的可观察序列。[更多信息](http://reactivex.io/documentation/operators/create.html)

``` swift
example("create") {
    let disposeBag = DisposeBag()

    let myJust = { (element: String) -> Observable<String> in
        return Observable.create { observer in
            observer.on(.next(element))
            observer.on(.completed)
            return Disposables.create()
        }
    }

    myJust("🔴")
        .subscribe { print($0) }
        .disposed(by: disposeBag)
}
```

![create](http://reactivex.io/documentation/operators/images/create.c.png)

### range

创建一个可观察序列，它会释放一系列顺序的整数，然后终止。[更多信息](http://reactivex.io/documentation/operators/range.html)

``` swift
example("range") {
    let disposeBag = DisposeBag()

    Observable.range(start: 1, count: 10)
        .subscribe { print($0) }
        .disposed(by: disposeBag)
}
```

![range](http://reactivex.io/documentation/operators/images/range.c.png)

### repeatElement

创建一个可观察的序列，它可以无限地释放给定的元素。[更多信息](http://reactivex.io/documentation/operators/repeat.html)

``` swift
example("repeatElement") {
    let disposeBag = DisposeBag()

    Observable.repeatElement("🔴")
        .take(3)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

> 这个例子还介绍了使用take操作符从一个序列开始返回指定数量的元素。

![repeat](http://reactivex.io/documentation/operators/images/repeat.c.png)

### generate

创建一个可观察序列，只要提供的条件值为true，就可以生成值。

``` swift
example("generate") {
    let disposeBag = DisposeBag()

    Observable.generate(
            initialState: 0,
            condition: { $0 < 3 },
            iterate: { $0 + 1 }
        )
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

### deferred

为每个订阅者创建一个新的可观察序列。[更多信息](http://reactivex.io/documentation/operators/defer.html)

``` swift
example("deferred") {
    let disposeBag = DisposeBag()
    var count = 1

    let deferredSequence = Observable<String>.deferred {
        print("Creating \(count)")
        count += 1

        return Observable.create { observer in
            print("Emitting...")
            observer.onNext("🐶")
            observer.onNext("🐱")
            observer.onNext("🐵")
            return Disposables.create()
        }
    }

    deferredSequence
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

    deferredSequence
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

![deferred](http://reactivex.io/documentation/operators/images/defer.c.png)

### error

创建一个可观察序列，该序列不会发送任何条目，并立即终止错误。

``` swift
example("error") {
    let disposeBag = DisposeBag()

    Observable<Int>.error(TestError.test)
        .subscribe { print($0) }
        .disposed(by: disposeBag)
}
```

![throw](http://reactivex.io/documentation/operators/images/throw.c.png)

### doOn

为每个发出的事件调用一个副操作，并返回(通过)原始事件。[更多信息](http://reactivex.io/documentation/operators/do.html)

``` swift
example("doOn") {
    let disposeBag = DisposeBag()

    Observable.of("🍎", "🍐", "🍊", "🍋")
        .do(onNext: { print("Intercepted:", $0) }, onError: { print("Intercepted error:", $0) }, onCompleted: { print("Completed")  })
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

> 还有 doOnNext(_:), doOnError(_:), doOnCompleted(_:)便利方法拦截这些特定事件,和doOn(onNext:onError:oncomplete)拦截一个或多个事件。


## Subjects ##

`主题`

主题是一种桥梁或代理，它在Rx的某些实现中可用，它既是观察者(observer)，也是可观察对象(Observable)。因为它是一个观察者，它可以订阅一个或多个可观察对象，因为它是一个可观察对象，它可以通过它所观察到的事件重新发射它们，它也可以发射新的事件。[更多信息](http://reactivex.io/documentation/subject.html)

``` swift
extension ObservableType {

    /**
     Add observer with `id` and print each emitted event.
     - parameter id: an identifier for the subscription.
     */
    func addObserver(_ id: String) -> Disposable {
        return subscribe { print("Subscription:", id, "Event:", $0) }
    }

}

func writeSequenceToConsole<O: ObservableType>(name: String, sequence: O) -> Disposable {
    return sequence.subscribe { event in
        print("Subscription: \(name), event: \(event)")
    }
}
```

### PublishSubject

向所有的观察者广播新的事件，从他们订阅时开始。

``` swift
example("PublishSubject") {
    let disposeBag = DisposeBag()
    let subject = PublishSubject<String>()

    subject.addObserver("1").disposed(by: disposeBag)
    subject.onNext("🐶")
    subject.onNext("🐱")

    subject.addObserver("2").disposed(by: disposeBag)
    subject.onNext("🅰️")
    subject.onNext("🅱️")
}
```

![PublishSubject](http://reactivex.io/documentation/operators/images/S.PublishSubject.png)

> 这个例子还介绍了使用onNext(:)便利方法，它等价于on(.next(:)，它会将一个新事件发送给带有所提供元素的订阅者。还有onError(:)和onCompleted()便利方法，它们分别对应于on(.error(:))和on(.completed)。

### ReplaySubject

向所有订阅者广播新事件，并向新订户指定特定缓冲区大小的先前事件。

``` swift
example("ReplaySubject") {
    let disposeBag = DisposeBag()
    let subject = ReplaySubject<String>.create(bufferSize: 1)

    subject.addObserver("1").disposed(by: disposeBag)
    subject.onNext("🐶")
    subject.onNext("🐱")

    subject.addObserver("2").disposed(by: disposeBag)
    subject.onNext("🅰️")
    subject.onNext("🅱️")
}
```

![ReplaySubject](http://reactivex.io/documentation/operators/images/S.ReplaySubject.png)

### BehaviorSubject

向所有订阅者广播新事件，以及最近(或最初)对新订户的事件。

``` swift
example("BehaviorSubject") {
    let disposeBag = DisposeBag()
    let subject = BehaviorSubject(value: "🔴")

    subject.addObserver("1").disposed(by: disposeBag)
    subject.onNext("🐶")
    subject.onNext("🐱")

    subject.addObserver("2").disposed(by: disposeBag)
    subject.onNext("🅰️")
    subject.onNext("🅱️")

    subject.addObserver("3").disposed(by: disposeBag)
    subject.onNext("🍐")
    subject.onNext("🍊")
}
```

![BehaviorSubject](http://reactivex.io/documentation/operators/images/S.BehaviorSubject.png)

注意前面的例子中遗漏了什么?一个完整的事件。PublishSubject、ReplaySubject和BehaviorSubject在即将disposed时不会自动发出Completed事件。

### Variable

包装一个BehaviorSubject，因此它将向新订户发送最新(或初始)值。变量也保持当前的值状态。变量永远不会发出错误事件。但是，它将自动发出一个已完成的事件，并在deinit上终止。

``` swift
example("Variable") {
    let disposeBag = DisposeBag()
    let variable = Variable("🔴")

    variable.asObservable().addObserver("1").disposed(by: disposeBag)
    variable.value = "🐶"
    variable.value = "🐱"

    variable.asObservable().addObserver("2").disposed(by: disposeBag)
    variable.value = "🅰️"
    variable.value = "🅱️"
}
```

> 在一个Variable实例上调用asObservable()可以访问其基础的BehaviorSubject序列。变量没有实现on运算符(或onNext(:))，而是公开一个可用于获取当前值的value属性，并设置一个新值。设置一个新值也会将该值添加到其基础BehaviorSubject序列中。


## 合并操作符 ##

将多个源Observables组合成一个Observable的操作符。

### startWith

在可观察的源开始发射元素之前，发射指定的元素序列。[更多信息](http://reactivex.io/documentation/operators/startwith.html)

``` swift
example("startWith") {
    let disposeBag = DisposeBag()

    Observable.of("🐶", "🐱", "🐭", "🐹")
        .startWith("1️⃣")
        .startWith("2️⃣")
        .startWith("3️⃣", "🅰️", "🅱️")
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

![startWith](http://reactivex.io/documentation/operators/images/startWith.png)

### merge

将源可观察序列中的元素组合成一个新的可观察序列，并在每个源可观察序列中释放出每个元素。[更多信息](http://reactivex.io/documentation/operators/merge.html)

``` swift
example("merge") {
    let disposeBag = DisposeBag()

    let subject1 = PublishSubject<String>()
    let subject2 = PublishSubject<String>()

    Observable.of(subject1, subject2)
        .merge()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

    subject1.onNext("🅰️")

    subject1.onNext("🅱️")

    subject2.onNext("①")

    subject2.onNext("②")

    subject1.onNext("🆎")

    subject2.onNext("③")
}
```

![merge](http://reactivex.io/documentation/operators/images/merge.png)

### zip

将多达8个源可观察序列组合成一个新的可观察序列，并将从组合的可观察序列中发射出相应的可观察序列的各个元素。[更多信息](http://reactivex.io/documentation/operators/zip.html)


``` swift
example("zip") {
    let disposeBag = DisposeBag()

    let stringSubject = PublishSubject<String>()
    let intSubject = PublishSubject<Int>()

    Observable.zip(stringSubject, intSubject) { stringElement, intElement in
        "\(stringElement) \(intElement)"
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

    stringSubject.onNext("🅰️")
    stringSubject.onNext("🅱️")

    intSubject.onNext(1)

    intSubject.onNext(2)

    stringSubject.onNext("🆎")
    intSubject.onNext(3)
}
```

![zip](http://reactivex.io/documentation/operators/images/zip.o.png)

### combineLatest

将最多8个源可观察序列组合成一个新的可观察序列序列,并将开始发出联合可观察序列的每个源的最新元素可观测序列一旦所有排放源序列至少有一个元素,并且当源可观测序列发出的任何一个新元素。[更多信息](http://reactivex.io/documentation/operators/combinelatest.html)

``` swift
example("combineLatest") {
    let disposeBag = DisposeBag()

    let stringSubject = PublishSubject<String>()
    let intSubject = PublishSubject<Int>()

    Observable.combineLatest(stringSubject, intSubject) { stringElement, intElement in
            "\(stringElement) \(intElement)"
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

    stringSubject.onNext("🅰️")

    stringSubject.onNext("🅱️")
    intSubject.onNext(1)

    intSubject.onNext(2)

    stringSubject.onNext("🆎")
}
```

![combineLatest](http://reactivex.io/documentation/operators/images/combineLatest.png)

还有一个combineLatest的变体，它使用一个数组(或任何其他可观察序列的集合):

``` swift
example("Array.combineLatest") {
    let disposeBag = DisposeBag()

    let stringObservable = Observable.just("❤️")
    let fruitObservable = Observable.from(["🍎", "🍐", "🍊"])
    let animalObservable = Observable.of("🐶", "🐱", "🐭", "🐹")

    Observable.combineLatest([stringObservable, fruitObservable, animalObservable]) {
            "\($0[0]) \($0[1]) \($0[2])"
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

> 由于combineLatest变体采用将数组传递给选择器函数，所以它要求所有的源可观察序列都是相同类型的。

### switchLatest

将可观察序列发射的元素转换成可观察的序列，并从最近的内部可观察序列中释放出元素。[更多信息](http://reactivex.io/documentation/operators/switch.html)

``` swift
example("switchLatest") {
    let disposeBag = DisposeBag()

    let subject1 = BehaviorSubject(value: "⚽️")
    let subject2 = BehaviorSubject(value: "🍎")

    let variable = Variable(subject1)

    variable.asObservable()
        .switchLatest()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

    subject1.onNext("🏈")
    subject1.onNext("🏀")

    variable.value = subject2

    subject1.onNext("⚾️")

    subject2.onNext("🍐")
}
```

> 在本例中，在设置variable.value为subject2后添加⚾️到subject1没有效果，因为只有最近的内可观察序列(subject2)才会发射元素。

 ![switch](http://reactivex.io/documentation/operators/images/switch.c.png)

## 转换操作符 ##

通过可观察的序列对下一个事件元素进行转换的操作符。

``` swift
example("map") {
    let disposeBag = DisposeBag()
    Observable.of(1, 2, 3)
        .map { $0 * $0 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

### map

将一个转换闭包应用到可观察序列所发射的元素上，并返回转换后的元素的一个新的可观察序列。[更多信息](http://reactivex.io/documentation/operators/map.html)

![map](http://reactivex.io/documentation/operators/images/map.png)

### flatMap 和 flatMapLatest

将可观察序列发射的元素转换成可观察的序列，并将两个可观测序列的排放量合并成一个可观察的序列。当你有一个可观察的序列，它本身会发出可观测的序列，你想要能够对任何可观察序列的新排放做出反应，这也是很有用的。flatMap和flatMapLatest的区别在于，flatMapLatest只会从最近的内部可观察序列中发射出元素。[更多信息](http://reactivex.io/documentation/operators/flatmap.html)

``` swift
example("flatMap and flatMapLatest") {
    let disposeBag = DisposeBag()

    struct Player {
        var score: Variable<Int>
    }

    let 👦🏻 = Player(score: Variable(80))
    let 👧🏼 = Player(score: Variable(90))

    let player = Variable(👦🏻)

    player.asObservable()
        .flatMap { $0.score.asObservable() } // Change flatMap to flatMapLatest and observe change in printed output
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

    👦🏻.score.value = 85

    player.value = 👧🏼

    👦🏻.score.value = 95 // Will be printed when using flatMap, but will not be printed when using flatMapLatest

    👧🏼.score.value = 100
}
```

flatMapLatest实际上是map和switchLatest操作符的结合。

![flatmap](http://reactivex.io/documentation/operators/images/flatMap.c.png)

### scan

从一个初始值开始，然后对一个可观察序列发射的每个元素应用一个累加器闭包，并将每个中间结果作为一个单元素可观察的序列返回。[更多信息](http://reactivex.io/documentation/operators/scan.html)

``` swift
example("scan") {
    let disposeBag = DisposeBag()

    Observable.of(10, 100, 1000)
        .scan(1) { aggregateValue, newValue in
            aggregateValue + newValue
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

![scan](http://reactivex.io/documentation/operators/images/scan.png)

## 过滤和条件运算符 ##

有选择性地从一个可观察到的序列中发射元素的操作者。

### filter

只从可观察的序列中发射出符合指定条件的元素。[更多信息](http://reactivex.io/documentation/operators/filter.html)

```swift
example("filter") {
    let disposeBag = DisposeBag()

    Observable.of(
        "🐱", "🐰", "🐶",
        "🐸", "🐱", "🐰",
        "🐹", "🐸", "🐱")
        .filter {
            $0 == "🐱"
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

![filter](http://reactivex.io/documentation/operators/images/filter.png)

### distinctUntilChanged

抑制可观察序列发出的序列重复元素。[更多信息](http://reactivex.io/documentation/operators/distinct.html)

```swift
example("distinctUntilChanged") {
    let disposeBag = DisposeBag()

    Observable.of("🐱", "🐷", "🐱", "🐱", "🐱", "🐵", "🐱")
        .distinctUntilChanged()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

```
"🐱" "🐷" "🐱" "🐵" "🐱"
```

![distinctUntilChanged.png](http://reactivex.io/documentation/operators/images/distinctUntilChanged.png)

### elementAt

只在一个可观察序列所发射指定索引中的元素。[更多信息](http://reactivex.io/documentation/operators/elementat.html)

```swift
example("elementAt") {
    let disposeBag = DisposeBag()

    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .elementAt(3)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}

```

```
"🐸"
```

![elementAt](http://reactivex.io/documentation/operators/images/elementAt.png)

### single

只发出由可观察序列发出的第一个元素(或第一个符合条件的元素)。如果可观测的序列不发射一个元素，就会抛出一个错误。

```swift
example("single") {
    let disposeBag = DisposeBag()

    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .single()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}

example("single with conditions") {
    let disposeBag = DisposeBag()

    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .single { $0 == "🐸" }
        .subscribe { print($0) }
        .disposed(by: disposeBag)

    Observable.of("🐱", "🐰", "🐶", "🐱", "🐰", "🐶")
        .single { $0 == "🐰" }
        .subscribe { print($0) }
        .disposed(by: disposeBag)

    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .single { $0 == "🔵" }
        .subscribe { print($0) }
        .disposed(by: disposeBag)
}
```

![first](http://reactivex.io/documentation/operators/images/first.png)

### take

只从可观察序列的开头发出指定数量的元素。[更多信息](http://reactivex.io/documentation/operators/take.html)

```swift
example("take") {
    let disposeBag = DisposeBag()

    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .take(3)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}

```

```
"🐱" "🐰" "🐶"
```

![take](http://reactivex.io/documentation/operators/images/take.png)

### takeLast

只从可观察序列的末尾发出指定数量的元素。[更多信息](http://reactivex.io/documentation/operators/takelast.html)

```swift
example("takeLast") {
    let disposeBag = DisposeBag()

    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .takeLast(3)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}

```

```
"🐸" "🐷" "🐵"
```

![takeLast](http://reactivex.io/documentation/operators/images/takeLast.n.png)

### takeWhile

只从可观察序列的开头发出,要指定条件的值为true 元素。[更多信息](http://reactivex.io/documentation/operators/takewhile.html)

```swift
example("takeWhile") {
    let disposeBag = DisposeBag()

    Observable.of(1, 2, 3, 4, 5, 6)
        .takeWhile { $0 < 4 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}

```

![takeWhile](http://reactivex.io/documentation/operators/images/takeWhile.c.png)

### takeUntil

从一个源可观察序列中发射元素，直到一个可观察的序列释放出一个元素。[更多信息](http://reactivex.io/documentation/operators/takeuntil.html)

```swift
example("takeUntil") {
    let disposeBag = DisposeBag()

    let sourceSequence = PublishSubject<String>()
    let referenceSequence = PublishSubject<String>()

    sourceSequence
        .takeUntil(referenceSequence)
        .subscribe { print($0) }
        .disposed(by: disposeBag)

    sourceSequence.onNext("🐱")
    sourceSequence.onNext("🐰")
    sourceSequence.onNext("🐶")

    referenceSequence.onNext("🔴")

    sourceSequence.onNext("🐸")
    sourceSequence.onNext("🐷")
    sourceSequence.onNext("🐵")
}
```

```
"🐱" "🐰" "🐶"
```

![takeUntil](http://reactivex.io/documentation/operators/images/takeUntil.png)

### skip

从一个可观察序列开始，抑制发射指定数量的元素。[更多信息](http://reactivex.io/documentation/operators/skip.html)

```swift
example("skip") {
    let disposeBag = DisposeBag()

    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .skip(2)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}

```

![skip](http://reactivex.io/documentation/operators/images/skip.png)

### skipWhile

从一个可观察到的序列的开始,抑制满足指定条件的元素。[更多信息](http://reactivex.io/documentation/operators/skipwhile.html)

```swift
example("skipWhile") {
    let disposeBag = DisposeBag()

    Observable.of(1, 2, 3, 4, 5, 6)
        .skipWhile { $0 < 4 }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

![skipWhile](http://reactivex.io/documentation/operators/images/skipWhile.c.png)

### skipWhileWithIndex

抑制从一个可观察到的序列的开始，满足指定条件的元素，并发射剩下的元素。闭包参数包含每个元素的索引。

```swift
example("skipWhileWithIndex") {
    let disposeBag = DisposeBag()

    Observable.of("🐱", "🐰", "🐶", "🐸", "🐷", "🐵")
        .skipWhileWithIndex { element, index in
            index < 3
        }
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

### skipUntil

抑制元素从一个可观察序列中发射出来，直到一个可观察序列发射出一个元素。[更多信息](http://reactivex.io/documentation/operators/skipuntil.html)

```swift
example("skipUntil") {
    let disposeBag = DisposeBag()

    let sourceSequence = PublishSubject<String>()
    let referenceSequence = PublishSubject<String>()

    sourceSequence
        .skipUntil(referenceSequence)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)

    sourceSequence.onNext("🐱")
    sourceSequence.onNext("🐰")
    sourceSequence.onNext("🐶")

    referenceSequence.onNext("🔴")

    sourceSequence.onNext("🐸")
    sourceSequence.onNext("🐷")
    sourceSequence.onNext("🐵")
}
```

![skipUntil](http://reactivex.io/documentation/operators/images/skipUntil.png)

## 数学运算及聚合操作符 ##

转换每个可观察元素成单个Observable的操作。

### toArray

将可观察序列转换为数组，将该数组作为一个新的单元素可观察序列发射，然后终止。[更多信息](http://reactivex.io/documentation/operators/to.html)

```swift
example("toArray") {
    let disposeBag = DisposeBag()

    Observable.range(start: 1, count: 10)
        .toArray()
        .subscribe { print($0) }
        .disposed(by: disposeBag)
}
```

![to](http://reactivex.io/documentation/operators/images/to.c.png)

### reduce

从一个初始值开始，然后将一个累加闭包应用到一个可观察序列所发射的所有元素上，并将总结果作为一个单元素可观察的序列返回。[更多信息](http://reactivex.io/documentation/operators/reduce.html)

```swift
example("reduce") {
    let disposeBag = DisposeBag()

    Observable.of(10, 100, 1000)
        .reduce(1, accumulator: +)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

![reduce](http://reactivex.io/documentation/operators/images/reduce.png)

### concat

从序列的可观察序列的内可观察序列中加入元素，等待每个序列在从下一个序列中释放元素之前成功终止。[更多信息](http://reactivex.io/documentation/operators/concat.html)

```swift
example("concat") {
    let disposeBag = DisposeBag()

    let subject1 = BehaviorSubject(value: "🍎")
    let subject2 = BehaviorSubject(value: "🐶")

    let variable = Variable(subject1)

    variable.asObservable()
        .concat()
        .subscribe { print($0) }
        .disposed(by: disposeBag)

    subject1.onNext("🍐")
    subject1.onNext("🍊")

    variable.value = subject2

    subject2.onNext("I would be ignored")
    subject2.onNext("🐱")

    subject1.onCompleted()

    subject2.onNext("🐭")
}
```

![concat](http://reactivex.io/documentation/operators/images/concat.png)

## 连接操作符 ##

可连接的可观察序列与普通观察序列类似，只不过它们不是在订阅时开始发射元素，而是在调用connect()方法时才开始发射元素。通过这种方式，您可以等待所有预定的订阅者在开始发出元素之前订阅一个可连接的可观察序列。

在学习连接操作符之前，我们先来看一个非连接操作符的示例:

``` swift
func sampleWithoutConnectableOperators() {
    printExampleHeader(#function)

    let interval = Observable<Int>.interval(1, scheduler: MainScheduler.instance)

    _ = interval
        .subscribe(onNext: { print("Subscription: 1, Event: \($0)") })

    delay(5) {
        _ = interval
            .subscribe(onNext: { print("Subscription: 2, Event: \($0)") })
    }
}

//sampleWithoutConnectableOperators() // ⚠️ Uncomment to run this example; comment to stop running
```

### publish

将源可观察序列转换为连接序列。[更多信息](http://reactivex.io/documentation/operators/publish.html)

``` swift
func sampleWithPublish() {
    printExampleHeader(#function)

    let intSequence = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        .publish()

    _ = intSequence
        .subscribe(onNext: { print("Subscription 1:, Event: \($0)") })

    delay(2) { _ = intSequence.connect() }

    delay(4) {
        _ = intSequence
            .subscribe(onNext: { print("Subscription 2:, Event: \($0)") })
    }

    delay(6) {
        _ = intSequence
            .subscribe(onNext: { print("Subscription 3:, Event: \($0)") })
    }
}

//sampleWithPublish() // ⚠️ Uncomment to run this example; comment to stop running
```

![publish](http://reactivex.io/documentation/operators/images/publishConnect.c.png)

### replay

将源可观察序列转换为连接序列，并将先前排放的缓冲区大小重放给每个新订户。[更多信息](http://reactivex.io/documentation/operators/replay.html)

``` swift
func sampleWithReplayBuffer() {
    printExampleHeader(#function)

    let intSequence = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        .replay(5)

    _ = intSequence
        .subscribe(onNext: { print("Subscription 1:, Event: \($0)") })

    delay(2) { _ = intSequence.connect() }

    delay(4) {
        _ = intSequence
            .subscribe(onNext: { print("Subscription 2:, Event: \($0)") })
    }

    delay(8) {
        _ = intSequence
            .subscribe(onNext: { print("Subscription 3:, Event: \($0)") })
    }
}

// sampleWithReplayBuffer() // ⚠️ Uncomment to run this example; comment to stop running
```

![replay](http://reactivex.io/documentation/operators/images/replay.c.png)

### multicast

将可观察的序列转换成可连接的序列，并通过指定的主题广播其发射。

``` swift
func sampleWithMulticast() {
    printExampleHeader(#function)

    let subject = PublishSubject<Int>()

    _ = subject
        .subscribe(onNext: { print("Subject: \($0)") })

    let intSequence = Observable<Int>.interval(1, scheduler: MainScheduler.instance)
        .multicast(subject)

    _ = intSequence
        .subscribe(onNext: { print("\tSubscription 1:, Event: \($0)") })

    delay(2) { _ = intSequence.connect() }

    delay(4) {
        _ = intSequence
            .subscribe(onNext: { print("\tSubscription 2:, Event: \($0)") })
    }

    delay(6) {
        _ = intSequence
            .subscribe(onNext: { print("\tSubscription 3:, Event: \($0)") })
    }
}

//sampleWithMulticast() // ⚠️ Uncomment to run this example; comment to stop running
```

## 错误处理操作符 ##

帮助从Observable的错误通知中恢复的操作符。

### catchErrorJustReturn

从错误事件中恢复，返回一个可观察序列，该序列释放一个元素，然后终止。[更多信息](http://reactivex.io/documentation/operators/catch.html)

```swift
example("catchErrorJustReturn") {
    let disposeBag = DisposeBag()

    let sequenceThatFails = PublishSubject<String>()

    sequenceThatFails
        .catchErrorJustReturn("😊")
        .subscribe { print($0) }
        .disposed(by: disposeBag)

    sequenceThatFails.onNext("😬")
    sequenceThatFails.onNext("😨")
    sequenceThatFails.onNext("😡")
    sequenceThatFails.onNext("🔴")
    sequenceThatFails.onError(TestError.test)
}
```

![catchErrorReturn](http://reactivex.io/documentation/operators/images/onErrorReturn.png)

### catchError

通过切换到所提供的恢复可观察序列来从错误事件中恢复。[更多信息](http://reactivex.io/documentation/operators/catch.html)

```swift
example("catchError") {
    let disposeBag = DisposeBag()

    let sequenceThatFails = PublishSubject<String>()
    let recoverySequence = PublishSubject<String>()

    sequenceThatFails
        .catchError {
            print("Error:", $0)
            return recoverySequence
        }
        .subscribe { print($0) }
        .disposed(by: disposeBag)

    sequenceThatFails.onNext("😬")
    sequenceThatFails.onNext("😨")
    sequenceThatFails.onNext("😡")
    sequenceThatFails.onNext("🔴")
    sequenceThatFails.onError(TestError.test)

    recoverySequence.onNext("😊")
}
```

![catchError](http://reactivex.io/documentation/operators/images/Catch.png)

### retry

通过重新订阅可观察的序列，可以无限地恢复重复的错误事件。[更多信息](http://reactivex.io/documentation/operators/retry.html)

```swift
example("retry") {
    let disposeBag = DisposeBag()
    var count = 1

    let sequenceThatErrors = Observable<String>.create { observer in
        observer.onNext("🍎")
        observer.onNext("🍐")
        observer.onNext("🍊")

        if count == 1 {
            observer.onError(TestError.test)
            print("Error encountered")
            count += 1
        }

        observer.onNext("🐶")
        observer.onNext("🐱")
        observer.onNext("🐭")
        observer.onCompleted()

        return Disposables.create()
    }

    sequenceThatErrors
        .retry()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

![retry](http://reactivex.io/documentation/operators/images/retry.C.png)

### retry(_:)

通过重新订阅可观察到的序列，再从错误事件中恢复，直到max重试次数的重试次数。[更多信息](http://reactivex.io/documentation/operators/retry.html)

```swift
example("retry maxAttemptCount") {
    let disposeBag = DisposeBag()
    var count = 1

    let sequenceThatErrors = Observable<String>.create { observer in
        observer.onNext("🍎")
        observer.onNext("🍐")
        observer.onNext("🍊")

        if count < 5 {
            observer.onError(TestError.test)
            print("Error encountered")
            count += 1
        }

        observer.onNext("🐶")
        observer.onNext("🐱")
        observer.onNext("🐭")
        observer.onCompleted()

        return Disposables.create()
    }

    sequenceThatErrors
        .retry(3)
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

## 调试操作符 ##

帮助调试Rx代码的操作符。

### debug

打印出所有订阅、事件和disposals。

```swift
example("debug") {
    let disposeBag = DisposeBag()
    var count = 1

    let sequenceThatErrors = Observable<String>.create { observer in
        observer.onNext("🍎")
        observer.onNext("🍐")
        observer.onNext("🍊")

        if count < 5 {
            observer.onError(TestError.test)
            print("Error encountered")
            count += 1
        }

        observer.onNext("🐶")
        observer.onNext("🐱")
        observer.onNext("🐭")
        observer.onCompleted()

        return Disposables.create()
    }

    sequenceThatErrors
        .retry(3)
        .debug()
        .subscribe(onNext: { print($0) })
        .disposed(by: disposeBag)
}
```

### RxSwift.Resources.total

提供所有Rx资源分配的计数，这对于开发期间检测泄漏非常有用。

```swift
#if NOT_IN_PLAYGROUND
#else
example("RxSwift.Resources.total") {
    print(RxSwift.Resources.total)

    let disposeBag = DisposeBag()

    print(RxSwift.Resources.total)

    let variable = Variable("🍎")

    let subscription1 = variable.asObservable().subscribe(onNext: { print($0) })

    print(RxSwift.Resources.total)

    let subscription2 = variable.asObservable().subscribe(onNext: { print($0) })

    print(RxSwift.Resources.total)

    subscription1.dispose()

    print(RxSwift.Resources.total)

    subscription2.dispose()

    print(RxSwift.Resources.total)
}

print(RxSwift.Resources.total)
#endif
```

## 使能RxSwift.Resources.total

在您的项目中，按照下列指示来启用RxSwift.Resources.total:

**CocoaPods**

1. 在你的Podfile中添加一个post_install, 如下所示:
1. 运行 `pod update`
1. 构建项目(Product -> Build)

`Podfile：`

```
target 'AppTarget' do
pod 'RxSwift'
end

post_install do |installer|
    installer.pods_project.targets.each do |target|
        if target.name == 'RxSwift'
            target.build_configurations.each do |config|
                if config.name == 'Debug'
                    config.build_settings['OTHER_SWIFT_FLAGS'] ||= ['-D', 'TRACE_RESOURCES']
                end
            end
        end
    end
end
```

**Carthage**

1. 运行 carthage build --configuration Debug
1. 构建工程(Product -> Build)
