---
title: RxSwift 警告
date: 2017-10-27 14:02:45
tags:
- rxswift
- swift
- iOS
categories:
- rxswift

---

![reactivex.io](http://reactivex.io/assets/reactivex_bg.jpg)

警告
========

1. [未使用 disposable](#unused-disposable)
1. [未使用 observable 序列](#unused-observable)

<!-- more -->

### <a name="unused-disposable"></a>未使用 disposable (unused-disposable)

下面的内容对于`subscribe*`、`bind*`和`drive*`家族函数 返回`Disposable`。

你会收到这样的警告:

```Swift
let xs: Observable<E> ....

xs
  .filter { ... }
  .map { ... }
  .switchLatest()
  .subscribe(onNext: {
    ...
  }, onError: {
    ...
  })
```

`subscribe`函数返回一个`Disposable`，可以用来取消计算和释放资源。但是，不使用它将导致一个错误。

终止这些流畅调用的首选方法是使用`DisposeBag`，或者通过链接调用`.disposed(by: disposeBag)`或者直接把disposable加到bag。

```Swift
let xs: Observable<E> ....
let disposeBag = DisposeBag()

xs
  .filter { ... }
  .map { ... }
  .switchLatest()
  .subscribe(onNext: {
    ...
  }, onError: {
    ...
  })
  .disposed(by: disposeBag) // <--- note `.disposed(by:)`
```

当`disposeBag`被释放时，它里面包含的disposables也会被自动处理掉。

如果`xs`以一种可预测的方式以`Completed` 或 `Error`的方式终止，不处理订阅`Disposable`不会泄漏任何资源. 然而，即使在这种情况下，使用一个disposeBag仍然是处理订阅处理问题的首选方法. 它确保了元素计算总是在可预见的时刻终止，并且使您的代码健壮和将来的证明，因为即使`xs`的实现改变，资源也会得到适当的处理.

另一种确保订阅和资源与某个对象的生命周期相关联的方法是使用`takeUntil`操作符

```Swift
let xs: Observable<E> ....
let someObject: NSObject  ...

_ = xs
  .filter { ... }
  .map { ... }
  .switchLatest()
  .takeUntil(someObject.deallocated) // <-- note the `takeUntil` operator
  .subscribe(onNext: {
    ...
  }, onError: {
    ...
  })
```

如果忽略订阅`Disposable`是期望的行为，这就是如何使编译器警告保持沉默。

```Swift
let xs: Observable<E> ....

_ = xs // <-- note the underscore
  .filter { ... }
  .map { ... }
  .switchLatest()
  .subscribe(onNext: {
    ...
  }, onError: {
    ...
  })
```

### <a name="unused-observable"></a>未使用 observable 序列 (unused-observable)

你会收到这样的警告:

```Swift
let xs: Observable<E> ....

xs
  .filter { ... }
  .map { ... }
```

这段代码定义了一个observable序列，该序列从`xs`序列中过滤并映射，但随后忽略了结果。

因为这段代码只定义了一个observable序列，然后忽略它，它实际上什么也不做。

您的意图可能是存储observable序列定义并随后使用它 ...

```Swift
let xs: Observable<E> ....

let ys = xs // <--- names definition as `ys`
  .filter { ... }
  .map { ... }
```

... 或者根据这个定义开始计算

```Swift
let xs: Observable<E> ....
let disposeBag = DisposeBag()

xs
  .filter { ... }
  .map { ... }
  .subscribe(onNext: { nextElement in  // <-- note the `subscribe*` method
    // use the element
    print(nextElement)
  })
  .disposed(by: disposeBag)
```
