---
title: RxSwift 特征
date: 2017-10-27 11:26:51
tags:
- rxswift
- swift
- iOS
categories:
- rxswift

---

![reactivex.io](http://reactivex.io/assets/reactivex_bg.jpg)

特征(以前的单位)
=====

本文将尝试描述什么是特征，为什么它们是一个有用的概念，以及如何使用和创建它们。

* [概括](#general)
  * [为什么](#why)
  * [它们是如何工作的](#how-they-work)
* [RxSwift 特征](#rxswift-traits)
  * [Single](#single)
    * [创建Single](#creating-a-single)
  * [Completable](#completable)
    * [创建Completable](#creating-a-completable)
  * [Maybe](#maybe)
    * [创建Maybe](#creating-a-maybe)
* [RxCocoa 特征](#rxcocoa-traits)
  * [Driver](#driver)
      * [为什么它被命名为 Driver](#why-is-it-named-driver)
      * [实际使用的例子](#practical-usage-example)
  * [ControlProperty / ControlEvent](#controlproperty--controlevent)

<!-- more -->

## 概括
### 为什么

Swift有一个强大的类型系统，可以用来提高应用程序的正确性和稳定性，并使使用Rx更加直观和直观.

特征有助于沟通和确保observable序列属性的接口边界，以及提供上下文意义、语法糖和更具体的用例目标。 **出于这个原因，特征完全是可选的。在您的程序中，您可以自由地使用原始的可观察序列，因为所有的核心rxswift/rxcocoa api都支持它们**。

_**注意:**本文档中描述的一些特征(如`Driver`)是特定只对[RxCocoa](https://github.com/ReactiveX/RxSwift/tree/master/RxCocoa)项目,有些是通用的一部分项目[RxSwift](https://github.com/ReactiveX/RxSwift)。但是，如果有必要，同样的原则也可以很容易地在其他Rx实现中实现。不需要私有API_

### 它们是如何工作的

特征仅仅是一个包装器结构，带有一个只读的Observable序列属性。

```swift
struct Single<Element> {
    let source: Observable<Element>
}

struct Driver<Element> {
    let source: Observable<Element>
}
...
```

您可以将它们看作是Observable序列的构建器模式实现。当一个特征被建立起来时，调用`.asObservable()`将把它转换成一个普通的可观察的序列。

---

## RxSwift traits

### Single

Single是Observable的变体，不是发射一系列的元素，它总是保证会发射_单个元素_或_一个错误_。

* 只释放一个元素，或者一个错误
* 不共享副作用

使用Single的一个常见的用例是执行HTTP请求，这些请求只能返回响应或错误, 但是，Single可以用于对任何只关心单个元素的情况进行建模，而不是对无限的元素流进行建模.

#### 创建Single
创建一个Single类似于创建一个Observable。
一个简单的例子就是这样的:

```swift
func getRepo(_ repo: String) -> Single<[String: Any]> {
    return Single<[String: Any]>.create { single in
        let task = URLSession.shared.dataTask(with: URL(string: "https://api.github.com/repos/\(repo)")!) { data, _, error in
            if let error = error {
                single(.error(error))
                return
            }

            guard let data = data,
                  let json = try? JSONSerialization.jsonObject(with: data, options: .mutableLeaves),
                  let result = json as? [String: Any] else {
                single(.error(DataError.cantParseJSON))
                return
            }

            single(.success(result))
        }

        task.resume()

        return Disposables.create { task.cancel() }
    }
}
```

之后你可以用以下方法来使用它:

```swift
getRepo("ReactiveX/RxSwift")
    .subscribe { event in
        switch event {
            case .success(let json):
                print("JSON: ", json)
            case .error(let error):
                print("Error: ", error)
        }
    }
    .disposed(by: disposeBag)
```

或者通过使用`subscribe(onSuccess:onError:)`如下:

```swift
getRepo("ReactiveX/RxSwift")
    .subscribe(onSuccess: { json in
                   print("JSON: ", json)
               },
               onError: { error in
                   print("Error: ", error)
               })
    .disposed(by: disposeBag)
```

订阅提供了`SingleEvent`枚举，它可以是`.success`包含Single类型的元素，也可以是`.error`。在第一个事件之后，不会再发生进一步的事件。

在原始的Observable序列中使用`.asSingle()`也可以将其转换为Single.

### Completable

Completable是Observable的变体，它只能_complete_或_发射一个error_。它保证不会发射任何元素.

* 发射零元素
* 发射一个completion事件或一个error
* 不共享副作用

Completable的一个有用的事例，是对任何我们只关心操作已经完成的事实的情况建模，但是不关心由该完成所导致的一个元素.
你可以将它与不能发射元素的`Observable<Void>`相比较.

#### 创建Completable

创建一个Completable类似于创建一个Observable。
一个简单的例子就是这样的:

```swift
func cacheLocally() -> Completable {
    return Completable.create { completable in
       // Store some data locally
       ...
       ...

       guard success else {
           completable(.error(CacheError.failedCaching))
           return Disposables.create {}
       }

       completable(.completed)
       return Disposables.create {}
    }
}
```

之后你可以用以下方法来使用它:

```swift
cacheLocally()
    .subscribe { completable in
        switch completable {
            case .completed:
                print("Completed with no error")
            case .error(let error):
                print("Completed with an error: \(error.localizedDescription)")
        }
    }
    .disposed(by: disposeBag)
```

或者通过使用subscribe(onSuccess:onError:)如下:

```swift
cacheLocally()
    .subscribe(onCompleted: {
                   print("Completed with no error")
               },
               onError: { error in
                   print("Completed with an error: \(error.localizedDescription)")
               })
    .disposed(by: disposeBag)
```

订阅提供了一个`CompletableEvent`枚举，可以是`.completed`表示没有错误的操作，也可以是`.error`。在第一个事件之后，不会再发生进一步的事件

### Maybe

Maybe可能是一种Observable的变体，在Single和Completable的之间。它既可以释放单个元素，也可以释放completed元素，也可以释放一个错误.

* 发出一个completed事件，一个元素或一个错误
* 不共享副作用

您可以使用Maybe来模拟任何**可以**释放元素的操作，但是**不一定**要释放一个元素。

#### 创建 Maybe

创建Maybe类似于创建一个Observable。一个简单的例子就是这样的:

```swift
func generateString() -> Maybe<String> {
    return Maybe<String>.create { maybe in
        maybe(.success("RxSwift"))

        // OR

        maybe(.completed)

        // OR

        maybe(.error(error))

        return Disposables.create {}
    }
}
```

之后你可以用以下方法来使用它:

```swift
generateString()
    .subscribe { maybe in
        switch maybe {
            case .success(let element):
                print("Completed with element \(element)")
            case .completed:
                print("Completed with no element")
            case .error(let error):
                print("Completed with an error \(error.localizedDescription)")
        }
    }
    .disposed(by: disposeBag)
```

或者通过使用subscribe(onSuccess:onError:)如下:

```swift
generateString()
    .subscribe(onSuccess: { element in
                   print("Completed with element \(element)")
               },
               onError: { error in
                   print("Completed with an error \(error.localizedDescription)")
               },
               onCompleted: {
                   print("Completed with no element")
               })
    .disposed(by: disposeBag)
```
在原始的Observable序列中使用`. asMaybe()`也可以将其转换为Maybe.

---

## RxCocoa 特征

### Driver

这是最复杂的特征。它的目的是提供一种直观的方式来在UI层中编写响应性代码，或者在任何情况下，您想要为您的应用程序建模 _Driving_的数据流。

* 不能错误
* 主调度器上的观察
* 共享副作用 (`shareReplayLatestWhileConnected`)

#### 为什么它被命名为 Driver

它的目标用例是对驱动应用程序的序列进行建模。

如

* 驱动 UI 从CoreData model
* 使用来自其他UI元素(绑定的)的值来驱动UI
...


与正常的操作系统驱动一样，如果出现序列错误，您的应用程序将停止响应用户输入。

同样重要的是，这些元素在主线程上被观察到，因为UI元素和应用程序逻辑通常不是线程安全的。

另外，一个`Driver`构建一个observable序列，并共享副作用。

如

#### 实际使用的例子

这是一个典型的初学者例子.

```swift
let results = query.rx.text
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
    }

results
    .map { "\($0.count)" }
    .bind(to: resultCount.rx.text)
    .disposed(by: disposeBag)

results
    .bind(to: resultsTableView.rx.items(cellIdentifier: "Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```

这段代码的意图是:

* 节流用户输入
* 连接服务器，获取用户结果列表(每次查询一次)
* 将结果绑定到两个UI元素:结果表视图和一个显示结果数量的标签

那么，这段代码有什么问题呢?:

* 如果`fetchAutoCompleteItems`observable序列错误(连接失败或解析错误)，这个错误将解绑所有的东西，UI将不会对新的查询作出任何响应.
* 如果`fetchAutoCompleteItems`返回一些后台线程的结果，结果将被绑定到后台线程的UI元素，这些元素可能导致非确定性崩溃.
* 结果被绑定到两个UI元素，这意味着对于每个用户查询，将创建两个HTTP请求，用于每个UI元素，这不是预期的行为.

更合适的代码版本应该是这样的:

```swift
let results = query.rx.text
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .observeOn(MainScheduler.instance)  // 在MainScheduler返回结果
            .catchErrorJustReturn([])           // 在最坏的情况下，会处理错误
    }
    .shareReplay(1)                             // HTTP请求被共享，结果被重新播放给所有UI元素
                                                //

results
    .map { "\($0.count)" }
    .bind(to: resultCount.rx.text)
    .disposed(by: disposeBag)

results
    .bind(to: resultsTableView.rx.items(cellIdentifier: "Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```

确保在大型系统中正确地处理所有这些需求是很有挑战性的，但是使用编译器和特性来证明这些需求是可以实现的，这是一种更简单的方法.

下面的代码看起来几乎是一样的:

```swift
let results = query.rx.text.asDriver()        // 这将一个正常的序列转换为一个`Driver`序列.
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .asDriver(onErrorJustReturn: [])  // 构建器只需要知道在发生错误时需要返回什么信息.
    }

results
    .map { "\($0.count)" }
    .drive(resultCount.rx.text)               // 如果有一个`drive`方法可用，而不是bindTo,
    .disposed(by: disposeBag)              // 这意味着编译器已经证明了所有属性都是满意的
                                              //
results
    .drive(resultsTableView.rx.items(cellIdentifier: "Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .disposed(by: disposeBag)
```

那么这里发生了什么?

第一个`asDriver`方法将`ControlProperty`特征转换为`Driver`特征。

```swift
query.rx.text.asDriver()
```

请注意，没有什么特别的事情需要做。`Driver`拥有`ControlProperty`特性的所有属性，再加上一些属性。潜在的可观察序列被包装成一个`Driver`特征，就是这样。

第二个变化是:

```swift
.asDriver(onErrorJustReturn: [])
```

任何observable序列都可以转换为`Driver`特性，只要它满足3个属性:

* 不能错误
* 在主调度器上观察
* 共享副作用(`shareReplayLatestWhileConnected`)

那么如何确保这些属性得到满足呢?只要使用普通的Rx操作符。`asDriver(onErrorJustReturn: [])`相当于下面的代码。

```swift
let safeSequence = xs
  .observeOn(MainScheduler.instance)       // 在主调度器上观察事件
  .catchErrorJustReturn(onErrorJustReturn) // 不能错误
  .shareReplayLatestWhileConnected()       // 副作用分享
return Driver(raw: safeSequence)           // 包起来
```

最后一个部分是使用`drive`，而不是使用`bindTo`。

`drive`只在`Driver`特性上定义。这意味着，如果您在代码的某个地方看到`drive`，observable序列永远不会出错，并且它在主线程上观察到，这对于绑定到UI元素是安全的。

但是请注意，从理论上讲，有人仍然可以定义一个`drive`方法来使用`ObservableType`或其他接口，因此为了获得额外的安全，可以创建一个临时的定义`let results: Driver<[Results]> = ...`。在绑定到UI元素之前，对于完整的证明是必要的。但是，我们将留给读者来决定这是否是一个现实的场景。

### ControlProperty / ControlEvent

* 不能错误
* 在主调度器上进行订阅
* 主调度器上的观察
* 共享副作用
