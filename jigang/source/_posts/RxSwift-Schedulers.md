---
title: RxSwift 调度器
date: 2017-10-27 09:56:45
tags:
- rxswift
- swift
- iOS
categories:
- rxswift

---

![reactivex.io](http://reactivex.io/assets/reactivex_bg.jpg)

调度器
==========

1. [串行和并行调度器](#serial-vs-concurrent-schedulers)
1. [定制的调度器](#custom-schedulers)
1. [内置的调度器](#builtin-schedulers)

<!-- more -->

调度器抽象出执行工作的机制.

执行工作的不同机制包括当前线程、调度队列、操作队列、新线程、线程池和运行循环.

有两个主要的操作符与调度器有关，`observeOn`和`subscribeOn`.

如果您想在不同的调度器上执行工作，只需使用`observeOn(scheduler)`操作符.

你通常会比`subscribeOn`更频繁地使用`observeOn`.

如果没有显式地指定`observeOn`，那么将在生成的任何线程/调度器上执行工作.

使用`observeOn`操作符的例子:

```
sequence1
  .observeOn(backgroundScheduler)
  .map { n in
      print("This is performed on the background scheduler")
  }
  .observeOn(MainScheduler.instance)
  .map { n in
      print("This is performed on the main scheduler")
  }
```

如果您想要启动序列生成(`subscribe`方法)并在特定的调度器上调用处理，请使用`subscribeOn(scheduler)`.

如果没有显式地指定`subscribeOn`，则`subscribe`闭包(通过`Observable.create`的闭包)将在同一线程/调度器上调用，在`subscribe(onNext:)` 或 `subscribe`被调用.

如果没有显式地指定`subscribeOn`，则`dispose`方法将在启动处理的同一线程/调度器上调用。.

简而言之，如果没有选择显式调度器，这些方法将在当前线程/调度器中调用.

# 串行和并行调度器

因为调度器确实可以是任何东西，并且所有转换序列的操作符都需要保留额外的[隐式保证](GettingStarted.md#implicit-observable-guarantees)，所以您要创建什么样的调度程序是很重要的。

如果调度器是并发的，Rx的`observeOn`和`subscribeOn`操作符将确保所有的操作都是完美的.

如果您使用的调度器，Rx可以证明是串行的，它将能够执行额外的优化.

到目前为止，它只对分派队列调度器执行这些优化.

在串行调度队列调度器的情况下，`observeOn`仅对一个简单的`dispatch_async`调用进行了优化.

# 定制的调度器

除了当前的调度器之外，您还可以编写自己的调度器.

如果您只是想描述谁需要立即执行工作，您可以通过实现`ImmediateScheduler`协议来创建自己的调度器.

```swift
public protocol ImmediateScheduler {
    func schedule<StateType>(state: StateType, action: (/*ImmediateScheduler,*/ StateType) -> RxResult<Disposable>) -> RxResult<Disposable>
}
```

如果您想要创建一个支持基于时间的操作的新调度程序，那么您需要实现`Scheduler`协议:

```swift
public protocol Scheduler: ImmediateScheduler {
    associatedtype TimeInterval
    associatedtype Time

    var now : Time {
        get
    }

    func scheduleRelative<StateType>(state: StateType, dueTime: TimeInterval, action: (StateType) -> RxResult<Disposable>) -> RxResult<Disposable>
}
```

如果调度器只有周期性调度功能，那么您可以通过实现`PeriodicScheduler`协议通知Rx:

```swift
public protocol PeriodicScheduler : Scheduler {
    func schedulePeriodic<StateType>(state: StateType, startAfter: TimeInterval, period: TimeInterval, action: (StateType) -> StateType) -> RxResult<Disposable>
}
```

如果调度程序不支持`PeriodicScheduling`功能，那么Rx将会透明地模拟周期性调度.

# 内置的调度器

Rx可以使用所有类型的调度程序，但是它也可以执行一些额外的优化，如果它能够证明调度程序是串行的.

这些是当前支持的调度器:

## CurrentThreadScheduler (串行调度器)

在当前线程上调度工作单元.
这是用于生成元素的操作符的默认调度器.

这个调度器有时也被称为"trampoline scheduler".

如果`CurrentThreadScheduler.instance.schedule(state) { }`第一次在一些线程被调用,预定行为将立即执行,将会创建一个隐藏的队列,所有递归地行动将临时加入队列.

如果调用堆栈上的一些父框架已经运行的`CurrentThreadScheduler.instance.schedule(state) { }`,预定行为将入队和当前运行时执行的行动和完成所有先前入队操作.

## MainScheduler (串行调度器)

抽象工作需要在`MainThread`上执行。如果`schedule`方法是从主线程调用的，那么它将在没有调度的情况下立即执行该操作.

这个调度器通常用于执行UI工作.

## SerialDispatchQueueScheduler (串行调度器)

对需要在特定`dispatch_queue_t`上执行的抽象工作。它将确保即使一个并发调度队列被传递，它也会被转换为一个串行化队列.

串行调度程序支持对`observeOn`的某些优化.

主要的调度器是`SerialDispatchQueueScheduler`的一个实例`.

## ConcurrentDispatchQueueScheduler (并行调度器)

对需要在特定`dispatch_queue_t`上执行的抽象工作。您也可以通过一个串行调度队列，它不应该引起任何问题.

当需要在后台执行某些工作时，这个调度器是合适的.

## OperationQueueScheduler (并行调度器)

抽象需要在特定的`NSOperationQueue`上执行的工作.

此调度器适合情况有一些大的工作需要在后台执行的,并且你想要调整并发处理使用`maxConcurrentOperationCount`.
