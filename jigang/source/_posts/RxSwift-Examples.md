---
title: RxSwift 例子
date: 2017-10-26 10:21:15
tags:
- rxswift
- swift
- iOS
categories:
- rxswift

---

![reactivex.io](http://reactivex.io/assets/reactivex_bg.jpg)

1. [计算变量](#计算变量)
1. [简单的UI绑定](#简单的UI绑定)
1. [自动输入验证](#自动输入验证)

<!-- more -->

## 计算变量

首先，让我们从一些命令式代码开始。
本例的目的是将标识符`c`绑定到从`a`和`b`计算的值，在满足某些条件下。

下面是计算`c`值的命令式代码:

```swift
// 标准的命令式代码
var c: String
var a = 1       // 只是赋值`1`到`a`一次
var b = 2       // 只是赋值`2`到`b`一次

if a + b >= 0 {
    c = "\(a + b) is positive" // 只是赋值`c`一次
}
```

`c`新的值是`3 is positive`. 然而，如果我们将`a`的值更改为`4`，那么`c`仍然会包含旧的值。

```swift
a = 4           // `c` 仍然等于 "3 is positive", 这是不好的
                // 我们想让 `c` 等于 "6 is positive" 因为 4 + 2 = 6
```

这不是我们想要的行为.

使用RxSwift改进的逻辑:

```swift
let a /*: Observable<Int>*/ = Variable(1)   // a = 1
let b /*: Observable<Int>*/ = Variable(2)   // b = 2

// 使用`+`组合最新`a`和`b`的变量值
let c = Observable.combineLatest(a.asObservable(), b.asObservable()) { $0 + $1 }
	.filter { $0 >= 0 }               // 如果 `a + b >= 0` 是 `true`, `a + b` 被传递给 `map` 操作符
	.map { "\($0) is positive" }      // maps `a + b` 后为 "\(a + b) is positive"

// 因为初始值是a=1和b=2
// 1 + 2 = 3 是 >= 0, 所以 `c` 一开始等于 "3 is positive"

// 要从Rx `Observable`中提取值，请从`c`的值订阅
// `subscribe(onNext:)` 意味着订阅`c`的下一个(新)值
// 这也包括初始值 "3 is positive".
c.subscribe(onNext: { print($0) })          // 打印: "3 is positive"

// 现在，让我们改变`a`的值
a.value = 4                                   // 打印: 6 is positive
// 最新值的总和, `4` + `2`, 是 `6`.
// 因为 `>= 0`, `map` 产生 "6 is positive"
// 这个结果被赋值给 `c`.
// 由于`c`的值改变了, `{ print($0) }` 将被执行,
// "6 is positive" 将被打印出来.

// 现在，让我们改变`b`的值
b.value = -8                                 // 没有任何打印
// 最新值的总和, `4 + (-8)`, 是 `-4`.
// 因为这不是 `>= 0`, `map` 没有被执行.
// 这意味着 `c` 仍然包含 "6 is positive"
// 由于`c`没有被更新，所以没有产生新的“next”值
// `{ print($0) }` 不会被调用.
```

## 简单的UI绑定

* 不再是绑定变量, 让我们绑定`UITextField`的值使用`rx.text`属性
* 然后, `map` 类型 `String` 到类型 `Int` 使用异步API确定这个数字是否为质数
* 如果在异步调用完成之前修改了文本，那么一个新的async调用将通过`concat`来替代它
* 将结果绑定到 `UILabel`

```swift
let subscription/*: Disposable */ = primeTextField.rx.text      // 类型是 Observable<String>
            .map { WolframAlphaIsPrime(Int($0) ?? 0) }          // 类型是 Observable<Observable<Prime>>
            .concat()                                           // 类型是 Observable<Prime>
            .map { "number \($0.n) is prime? \($0.isPrime)" }   // 类型是 Observable<String>
            .bind(to: resultLabel.rx.text)                      // 返回 Disposable 可用于解除所有内容

// 这将设置 `resultLabel.text` 为 "number 43 is prime? true" 在服务器调用完成后
primeTextField.text = "43"

// ...

// 要解绑所有，只需调用它
subscription.dispose()
```

本例中使用的所有操作符都是和第一个示例中使用的操作符一样的。没有什么特别的.

## 自动输入验证

如果您是Rx的新手，那么下一个例子可能会有点让人难以抗拒。但是，这里是为了演示RxSwift代码在现实世界中的样子.

这个示例包含复杂的async UI验证逻辑，其中包含了进度通知。
所有的操作都被`disposeBag`取消了。

让我们来试一试.

```swift
enum Availability {
    case available(message: String)
    case taken(message: String)
    case invalid(message: String)
    case pending(message: String)

    var message: String {
        switch self {
        case .available(message: let message),
             .taken(message: let message),
             .invalid(message: let message),
             .pending(message: let message):

             return message
        }
    }
}

// 直接绑定UI控制值
// 使用`usernameOutlet`的`username`作为username的值源
self.usernameOutlet.rx.text
    .map { username in

        // 同步验证，这里没有什么特别的
        if username.isEmpty {
            // 构建同步结果的方便性.
            // 如果在同一个方法中有混合的同步和异步代码,
            // 那么这将构造一个立即解析的异步结果.
            return Observable.just(Availability.invalid(message: "Username can't be empty."))
        }

        // ...

        // 当异步操作执行时，用户界面应该显示一些状态
        let loadingValue = Availability.pending(message: "Checking availability ...")

        // 这将触发一个服务器调用来检查用户名是否已经存在.
        // 它的类型是 `Observable<ValidationResult>`
        return API.usernameAvailable(username)
          .map { available in
              if available {
                  return Availability.available(message: "Username available")
              }
              else {
                  return Availability.unavailable(message: "Username already taken")
              }
          }
          // 使用 `loadingValue` 直到服务器响应
          .startWith(loadingValue)
    }
// 因为我们现在有 `Observable<Observable<ValidationResult>>`
// 我们需要返回一个简单的 `Observable<ValidationResult>`.
// 我们可以使用第二个示例中的concat操作符，但是如果提供了一个新的用户名，我们确实希望取消正在等待的异步操作。
// 这就为什么使用 `switchLatest` 了.
    .switchLatest()
// 现在我们需要将它绑定到用户界面.
// 好的旧方法 `subscribe(onNext:)` 能够实现.
// 这就是`Observable`链的末端.
    .subscribe(onNext: { validity in
        errorLabel.textColor = validationColor(validity)
        errorLabel.text = validity.message
    })
// 这将生成一个`Disposable`对象，能解绑所有事并取消正在等待的异步操作。
// 而不是手动操作，这很麻烦,
// 让我们在view controller dealloc时自动处理一切.
    .disposed(by: disposeBag)
```
