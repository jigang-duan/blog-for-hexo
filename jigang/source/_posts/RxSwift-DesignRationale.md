---
title: RxSwift 设计原理
date: 2017-10-26 10:13:53
tags:
- rxswift
- swift
- iOS
categories:
- rxswift

---

![reactivex.io](http://reactivex.io/assets/reactivex_bg.jpg)

## 为什么不使用普通的错误类型

<!-- more -->

```Swift
enum Event<Element>  {
    case next(Element)      // next element of a sequence
    case error(Error)   // sequence failed with error
    case completed          // sequence terminated successfully
}
```

让我们讨论一下`Error`的优点和缺点。

如果您有一个通用的错误类型，您将在两个observables之间创建额外的阻抗失配。

假设你有:

`Observable<String, E1>` 和 `Observable<String, E2>`

如果不知道最终的错误类型将会是什么，那么就没有多少可以做的了。

是E1，E2还是一些新的E3呢?你需要一套新的运算符来解决阻抗失配。

这损害了合成属性，Rx不关心为什么一个序列失败了，它通常会在observable链上进一步向前推进。

还有一个额外的问题，在某些情况下，由于某些内部错误，操作符可能会失败，在这种情况下，您将无法构造一个典型错误并报告失败。

但是，我们先不管它，假设我们可以用它来对不出错的序列进行建模。它对这个目的有用吗?

是的，它有可能是，但是让我们考虑一下为什么你想要使用不会出错的序列。

一个典型的应用是在UI层中保持流，驱动整个UI。当您考虑这种情况时，仅仅使用编译器来证明序列不会出错，这还不够，您还需要证明其他属性。例如，在`MainScheduler`上观察到这些元素。

你真正需要的是一种通用的方法来证明observable序列的特征。有很多你可能感兴趣的属性。例如:

* 序列在有限时间内终止(服务器端)
* 序列只包含一个元素(如果您正在运行一些计算)
* 序列不会出错，也不会终止，并且元素在主调度器(UI)
* 序列不会出错，永远不会终止，并且元素在主调度器上传递，并且重新计算共享(UI)
* 序列不会出错，也不会终止，并且元素在特定的后台调度程序中被传送(音频引擎)

你真正想要的是一个用于observable序列的一般的编译器强制系统，以及一组用于那些想要属性的不变量运算符。

一个比较好的类比:

```
1, 3.14, e, 2.79, 1 + 1i      <->    Observable<E>
1m/s, 1T, 5kg, 1.3 pounds     <->    无错误observable, UI observable, 有限 observable ...
```

在Swift中有很多方法可以通过使用组合或继承observables来完成这样的事情。

使用单元系统的另一个好处是，您可以证明UI代码在相同的调度器上执行，因此在所有转换中使用无锁的操作符。

因为RxSwift已经没有了单个序列操作的锁，并且所有存在的锁都是有状态的组件(例如UI)，所以RxSwift代码中几乎没有锁，这使得编译器可以强制执行这些细节。

在保留Rx组合语义的情况下，使用`Error`错误不能以其他更干净的方式实现，这是没有好处的。
