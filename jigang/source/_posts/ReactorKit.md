---
title: ReactorKit 介绍
date: 2017-11-09 18:40:18
tags:
- iOS
- swift
- iOS第三方库
categories:
- iOS

---

![reactorkit](https://cloud.githubusercontent.com/assets/931655/25277625/6aa05998-26da-11e7-9b85-e48bec938a6e.png)

[ReactorKit](https://github.com/ReactorKit/ReactorKit)是一个响应式和单向的Swift应用程序架构的框架。
这个文章介绍了ReactorKit的基本概念，并描述了如何使用ReactorKit构建应用程序。

如果您希望看到实际的代码，您可能希望首先看到[示例](#examples)部分。访问[API参考](http://reactorkit.io/docs/latest/)的代码级文档。

<!-- more -->

## 基本概念

ReactorKit是[Flux](https://facebook.github.io/flux/)和[响应式编程](https://en.wikipedia.org/wiki/Reactive_programming)的结合。
user actions和view states通过可观察的流传递到每一层。
这些流是单向的: view只能发射actions，reactor只能发射states。

![flux](https://cloud.githubusercontent.com/assets/931655/25073432/a91c1688-2321-11e7-8f04-bf91031a09dd.png)

### 设计目标

* **可测试性**：ReactorKit的第一个目的是将业务逻辑从视图中分离出来。
这可以使代码可测试。
一个reactor对视图没有任何依赖性。
只需要测试reactors和测试视图绑定。
有关详细信息，请参见[测试部分](#testing)。

* **从小处开始**: ReactorKit并不要求整个应用程序遵循单一的架构。
对于一个或多个特定的视图，可以部分地采用ReactorKit。
你不需要重写所有的东西来在你现有的项目中使用ReactorKit。

* **更少的成本**: ReactorKit专注于避免复杂的代码为简单的事情。
与其他架构相比，ReactorKit的代码需要更少的代码。
开始简单，扩大规模

### View

*View*显示数据。
view controller和cell被视为view。
view将用户输入绑定到action流，并将view states绑定到每个UI组件。
view层中没有业务逻辑。
view定义了如何映射action流和state流。

要定义一个view，只需让现有的类遵循一个名为`View`的协议。
然后，你的类会自动地有一个名为`reactor`的属性。
这个属性通常是在view之外设置的。

```swift
class ProfileViewController: UIViewController, View {
  var disposeBag = DisposeBag()
}

profileViewController.reactor = UserViewReactor() // inject reactor
```

当`reactor`属性发生变化时，就会调用`bind(reactor:)`。
实现这个方法来定义action流和state流的绑定。

```swift
func bind(reactor: ProfileViewReactor) {
  // action (View -> Reactor)
  refreshButton.rx.tap.map { Reactor.Action.refresh }
    .bind(to: reactor.action)
    .disposed(by: self.disposeBag)

  // state (Reactor -> View)
  reactor.state.map { $0.isFollowing }
    .bind(to: followButton.rx.isSelected)
    .disposed(by: self.disposeBag)
}
```

#### 支持 Storyboard

如果使用storyboard来初始化view controllers，使用`StoryboardView`协议。
所有的东西都是一样的，唯一不同的是，在视图加载之后，`StoryboardView`执行绑定。

```swift
let viewController = MyViewController()
viewController.reactor = MyViewReactor() // 不会立即执行`bind(reactor:)`

class MyViewController: UIViewController, StoryboardView {
  func bind(reactor: MyViewReactor) {
    // 在视图加载后(viewDidLoad)调用
  }
}
```

### Reactor

Reactor是一个独立于UI的层，它负责管理视图的状态。
reactor最重要的作用是将控制流从一个视图中分离出来。
每个视图都有对应的reactor，并把所有的逻辑都委托给它的reactor。
reactor不依赖于视图，因此可以很容易地测试它。

符合Reactor协议来定义一个reactor。
该协议需要定义三种类型: `Action`、`Mutation`和`State`。
它还需要一个名为`initialState`的属性。

```swift
class ProfileViewReactor: Reactor {
  // represent user actions
  enum Action {
    case refreshFollowingStatus(Int)
    case follow(Int)
  }

  // represent state changes
  enum Mutation {
    case setFollowing(Bool)
  }

  // represents the current view state
  struct State {
    var isFollowing: Bool = false
  }

  let initialState: State = State()
}
```

一个`Action`代表一个*用户交互*，而`State`代表一个*视图状态*。
`Mutation`是`Action`与`State`之间的*_桥梁_*。
一个reactor将action流转换成state流的两个步骤:`mutate()`和`reduce()`。

![reactor](https://cloud.githubusercontent.com/assets/931655/25098066/2de21a28-23e2-11e7-8a41-d33d199dd951.png)

**`mutate()`**

`mutate()`接收到一个`Action`并产生一个`Observable<Mutation>`。

```swift
func mutate(action: Action) -> Observable<Mutation>
```

每个副作用，比如异步操作或API调用，都是在这个方法中执行的。

```swift
func mutate(action: Action) -> Observable<Mutation> {
  switch action {
  case let .refreshFollowingStatus(userID): // receive an action
    return UserAPI.isFollowing(userID) // create an API stream
      .map { (isFollowing: Bool) -> Mutation in
        return Mutation.setFollowing(isFollowing) // convert to Mutation stream
      }

  case let .follow(userID):
    return UserAPI.follow()
      .map { _ -> Mutation in
        return Mutation.setFollowing(true)
      }
  }
}
```

**`reduce()`**

`reduce()`从以前的State和Mutation中生成一个新的State。

```swift
func reduce(state: State, mutation: Mutation) -> State
```

这个方法是一个纯函数。
它应该以同步方式返回一个新State。
不要在这个函数中执行任何副作用。

```swift
func reduce(state: State, mutation: Mutation) -> State {
  var state = state // create a copy of the old state
  switch mutation {
  case let .setFollowing(isFollowing):
    state.isFollowing = isFollowing // manipulate the state, creating a new state
    return state // return the new state
  }
}
```

**`transform()`**

`transform()`变换每个流。
有三个transform()函数:

```swift
func transform(action: Observable<Action>) -> Observable<Action>
func transform(mutation: Observable<Mutation>) -> Observable<Mutation>
func transform(state: Observable<State>) -> Observable<State>
```

实现这些方法来转换和合并其他可观察的流。
例如，transform(mutation:)是将全局事件流与mutation流相结合的最佳位置。
有关详细信息，请参见[全局状态](#global-states)部分。

这些方法也可以用于调试目的:

```swift
func transform(action: Observable<Action>) -> Observable<Action> {
  return action.debug("action") // Use RxSwift's debug() operator
}
```

## 高级

### Service

ReactorKit有一个特殊的层命名为Service。
service层执行实际的业务逻辑。
reactor是视图和管理事件流的服务之间的中间层。
当一个reactor从视图接收到用户操作时，reactor将调用service逻辑。
该service发出一个网络请求，并将响应发送回reactor。
然后，reactor创建一个带有服务响应的mutation流。

这里有一个服务的例子:

```swift
protocol UserServiceType {
  func user(id: Int) -> Observable<User>
  func follow(id: Int) -> Observable<Void>
}

final class UserService: Service, UserServiceType {
  func user(id: Int) -> Observable<User> {
    return foo()
  }

  func follow(id: Int) -> Observable<Void> {
    return bar()
  }
}
```

### 全局状态

不像Redux，ReactorKit并没有定义一个全局应用程序状态。
这意味着您可以使用任何东西来管理一个全局状态。
您可以使用一个Variable，一个PublishSubject，甚至一个reactor。
ReactorKit不需要有一个全局状态，所以你可以在你的应用中使用ReactorKit在一个特定的特性。

在`Action → Mutation → State`流中没有全局状态。
您应该使用`transform(mutation:)`来将全局状态转换为一个mutation。
让我们假设我们有一个全局Variable，该Variable存储当前经过身份验证的用户。
如果您想发出Mutation.setUser(User?)当`currentUser`被更改时，您可以如下所做:

```swift
var currentUser: Variable<User> // global state

func transform(mutation: Observable<Mutation>) -> Observable<Mutation> {
  return Observable.merge(mutation, currentUser.map(Mutation.setUser))
}
```

然后，每当视图向reactor发送一个action，而currentUser被改变时，这个mutation就会被释放出来。

### View交流

在多个视图之间进行通信，您必须熟悉回调闭包或委托模式。
反应堆建议你使用[reactive的扩展](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Reactive.swift)。
最常见的控制实例是UIButton.rx.tap。
关键的概念是将您的自定义视图视为UIButton或UILabel。

![view-view](https://user-images.githubusercontent.com/931655/27789114-393e2eea-6026-11e7-9b32-bae314e672ee.png)

让我们假设我们有一个显示消息的`ChatViewController`。
ChatViewController拥有一个`MessageInputView`。
当用户在MessageInputView上点击发送按钮时，文本将被发送到ChatViewController，而ChatViewController将绑定到反应器的操作。
这是一个示例:`MessageInputView`的Rx扩展:

```swift
extension Reactive where Base: MessageInputView {
  var sendButtonTap: ControlEvent<String> {
    let source = base.sendButton.rx.tap.withLatestFrom(...)
    return ControlEvent(events: source)
  }
}
```

你可以在ChatViewController中使用那个扩展。例如:

```swift
messageInputView.rx.sendButtonTap
  .map(Reactor.Action.send)
  .bind(to: reactor.action)
```

### 测试

ReactorKit有一个内置的测试功能。
您将能够轻松地测试一个view和一个reactor，并使用下面的指令。

#### 什么测试

首先，你必须决定要测试什么。
有两件事要测试:一个view和一个reactor。

* View
	* Action:一个适当的action被发送到一个指定用户交互的reactor上吗?
	* State: view属性是否正确地按state设置呢?
* Reactor
	* State:一个state随着action而改变吗?

#### View测试

一个view可以用一个stub  reactor进行测试。
reactor有一个stub属性，它可以记录actions并强制改变states。
如果启用了一个reactor的stub，mutate() 和reduce()都不会被执行。
一个stub有这些属性:

```swift
var isEnabled: Bool { get set }
var state: Variable<Reactor.State> { get }
var action: ActionSubject<Reactor.Action> { get }
var actions: [Reactor.Action] { get } // recorded actions
```

下面是一些示例测试案例:

```swift
func testAction_refresh() {
  // 1. prepare a stub reactor
  let reactor = MyReactor()
  reactor.stub.isEnabled = true

  // 2. prepare a view with a stub reactor
  let view = MyView()
  view.reactor = reactor

  // 3. send an user interaction programatically
  view.refreshControl.sendActions(for: .valueChanged)

  // 4. assert actions
  XCTAssertEqual(reactor.stub.actions.last, .refresh)
}

func testState_isLoading() {
  // 1. prepare a stub reactor
  let reactor = MyReactor()
  reactor.stub.isEnabled = true

  // 2. prepare a view with a stub reactor
  let view = MyView()
  view.reactor = reactor

  // 3. set a stub state
  reactor.stub.state.value = MyReactor.State(isLoading: true)

  // 4. assert view properties
  XCTAssertEqual(view.activityIndicator.isAnimating, true)
}
```

#### Reactor测试

一个reactor可以独立测试。

```swift
func testIsBookmarked() {
  let reactor = MyReactor()
  reactor.action.onNext(.toggleBookmarked)
  XCTAssertEqual(reactor.currentState.isBookmarked, true)
  reactor.action.onNext(.toggleBookmarked)
  XCTAssertEqual(reactor.currentState.isBookmarked, false)
}
```

有时，一个state会被多个action改变。
例如，一个.refresh的动作将第state.isLoading个设置为true，并在刷新后设置为false。
在这种情况下，很难用currentState测试state.isLoading，所以您可能需要使用RxTest或RxExpect。
下面是一个使用RxExpect的示例测试用例:

```swift
func testIsLoading() {
  RxExpect("it should change isLoading") { test in
    let reactor = test.retain(MyReactor())
    test.input(reactor.action, [
      next(100, .refresh) // send .refresh at 100 scheduler time
    ])
    test.assert(reactor.state.map { $0.isLoading })
      .since(100) // values since 100 scheduler time
      .assert([
        true,  // just after .refresh
        false, // after refreshing
      ])
  }
}
```

## 约定

ReactorKit提供了一些约定来编写干净简洁的代码。

您必须在view之外创建reactor，并将其传递给view的reactor属性。

**Good**

```swift
let view = MyView()
view.reactor = MyViewReactor(provider: provider)
```

**Bad**

```swift
class MyView: UIView, View {
  init() {
    self.reactor = MyViewReactor()
  }
}
```

## 示例

* [Counter](https://github.com/ReactorKit/ReactorKit/tree/master/Examples/Counter): 最简单和最基本的ReactorKit例子
* [GitHub Search](https://github.com/ReactorKit/ReactorKit/tree/master/Examples/GitHubSearch):一个提供GitHub库搜索的简单应用程序
* [RxTodo](https://github.com/devxoul/RxTodo): 使用ReactorKit的iOS待办事项
* [Cleverbot](https://github.com/devxoul/Cleverbot): 使用Cleverbot和ReactorKit的iOS消息应用
* [Drrrible](https://github.com/devxoul/Drrrible): Dribbble iOS使用ReactorKit(App Store)对
* [Passcode](https://github.com/cruisediary/Passcode):iOS RxSwift、ReactorKit和IGListKit示例的Passcode
