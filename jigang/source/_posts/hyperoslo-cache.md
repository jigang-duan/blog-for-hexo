---
title: Hyperoslo Cache介绍
date: 2018-01-26 09:20:34
tags:
- iOS
- swift
- iOS第三方库
categories:
- iOS

---

![](https://github.com/hyperoslo/Cache/raw/master/Resources/CachePresentation.png)

## 描述

在这个领域里，`Cache`并不是独一无二的，但它并不是另一个让你拥有上帝力量的怪物库。
除了缓存，它什么都不做，但它做得很好。
它提供了一个良好的公共API，它有开箱即用的实现和巨大的自定义可能性。
高速缓存在Swift 4中使用`Codable`来执行序列化。

<!-- more -->

## 关键特性

- [x] 与Swift 4 `Codable`合作。任何符合`Codable`的东西都将被`Storage`轻松地保存和加载。
- [X] 默认磁盘存储。可选地使用`内存存储`来启用混合.
- [X] 选项通过`DiskConfig`和`MemoryConfig`.
- [x] 支持`过期`和清除过期的对象.
- [x] 线程安全的。可以从任何队列访问操作.
- [x] 默认情况下同步。还支持异步APIs.
- [X] 存储图像通过`ImageWrapper`.
- [x] 广泛的单元测试覆盖面和文档.
- [x] 支持iOS, tvOS 和 macOS.

## 使用

### Storage

`Cache`是建立基于[责任链模式](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern),在其中有许多处理对象，每个处理对象都知道如何执行1个任务，并委托给下一个任务。
但这只是实现细节。您需要知道的是`Storage`，它保存并加载`Codable`对象。

`Storage`有磁盘存储和可选的内存存储。
内存存储应该更少时间和内存消耗，而磁盘存储用于满足应用程序生命周期的内容，更像一种存储用户信息的方便方式，这些信息应该在应用程序启动时持久化。

`DiskConfig`是必需的，设置磁盘存储。您可以选择通过`MemoryConfig`来将内存作为前端存储。

```swift
let diskConfig = DiskConfig(name: "Floppy")
let memoryConfig = MemoryConfig(expiry: .never, countLimit: 10, totalCostLimit: 10)

let storage = try? Storage(diskConfig: diskConfig, memoryConfig: memoryConfig)
```

#### Codable 类型

`Storage`支持任何符合[Codable](https://developer.apple.com/documentation/swift/codable)协议对象。
你可以[做你自己的事情符合Codable](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types),这样可以保存和加载`Storage`。

支持的类型:

- 原语如 `Int`, `Float`, `String`, `Bool`, ...
- 数组的元素如 `[Int]`, `[Float]`, `[Double]`, ...
- 基本类型的Set如 `Set<String>`, `Set<Int>`, ...
- 简单的字典 `[String: Int]`, `[String: String]`, ...
- `Date`
- `URL`
- `Data`

#### 错误处理

错误处理是通过`try catch`完成的。`Storage`在`StorageError`方面抛出错误。

```swift
public enum StorageError: Error {
  /// 不能找到对象
  case notFound
  /// 对象被找到，但是被请求的类型失败了
  case typeNotMatch
  /// 文件属性是畸形的
  case malformedFileAttributes
  /// 不能执行解码
  case decodingFailed
  /// 不能执行编码
  case encodingFailed
  /// 存储已经被释放了
  case deallocated
}
```

在从存储中加载时，可能会出现磁盘问题或类型不匹配的错误，因此，如果要处理错误，则需要`try catch`

```swift
do {
  let storage = try Storage(diskConfig: diskConfig, memoryConfig: memoryConfig)
} catch {
  print(error)
}
```

### 配置

下面是如何使用多种配置选项

```swift
let diskConfig = DiskConfig(
  //磁盘存储的名称，这将用作目录中的文件夹名称
  name: "Floppy",
  //在默认情况下，每个添加的对象都将使用该过期日期
  //如果它没有在`setObject(forKey:expiry:)`方法中被覆盖
  expiry: .date(Date().addingTimeInterval(2*3600)),
  maxSize: 10000,
  // 存储磁盘缓存的位置。如果nil，它被放置在cachesDirectory目录中.
  directory: try! FileManager.default.url(for: .documentDirectory, in: .userDomainMask,
                                          appropriateFor: nil, create: true).appendingPathComponent("MyPreferences"),
  // 数据保护用于在磁盘上以加密格式存储文件，并根据需要对其进行解密
  protectionType: .complete
)
```

```swift
let memoryConfig = MemoryConfig(
  //在默认情况下，每个添加的对象都将使用该过期日期
  //如果它没有在`setObject(forKey:expiry:)`方法中被覆盖
  expiry: .date(Date().addingTimeInterval(2*60)),
  /// 内存中的对象的最大数量应该保持在内存中
  countLimit: 50,
  /// 缓存在开始清除对象之前所能容纳的最大总成本
  totalCostLimit: 0
)
```

在iOS平台上，tvOS还可以在`DiskConfig`上指定`protectionType`，在应用程序的容器中为存储在磁盘上的文件添加安全级别。
有关更多信息,请参见[FileProtectionType](https://developer.apple.com/documentation/foundation/fileprotectiontype)

### 同步 APIs

缺省情况下，`Storage`是同步的，是`线程安全`的，您可以从任何队列访问它。
所有同步功能都受到`StorageAware`协议的约束。

```swift
// 保存到storage
try? storage.setObject(10, forKey: "score")
try? storage.setObject("Oslo", forKey: "my favorite city", expiry: .never)
try? storage.setObject(["alert", "sounds", "badge"], forKey: "notifications")
try? storage.setObject(data, forKey: "a bunch of bytes")
try? storage.setObject(authorizeURL, forKey: "authorization URL")

// 从storage加载
let score = try? storage.object(ofType: Int.self, forKey: "score")
let favoriteCharacter = try? storage.object(ofType: String.self, forKey: "my favorite city")

// 检查对象是否存在
let hasFavoriteCharacter = try? storage.existsObject(ofType: String.self, forKey: "my favorite city")

// 删除存储中的对象
try? storage.removeObject(forKey: "my favorite city")

// 删除所有对象
try? storage.removeAll()

// 删除过期的对象
try? storage.removeExpiredObjects()
```

#### Entry

您可以使用它的过期信息和元数据来获取对象。您可以使用 `Entry`

```swift
let entry = try? storage.entry(ofType: String.self, forKey: "my favorite city")
print(entry?.object)
print(entry?.expiry)
print(entry?.meta)
```

如果从磁盘存储器中取出对象，`meta`可能包含文件信息。


#### 自定义 Codable

`Codable`适用于像`[String: Int]`, `[String: String]`, ... 这样的简单字典
它并不适用于`[String: Any]`，因为`Any`不符合`Codable`的一致性，它将在运行时引发致命错误。因此，当您从后端响应中获得json时，您需要将其转换为您的自定义`Codable`对象，并将其保存为存储。

```swift
struct User: Codable {
  let firstName: String
  let lastName: String
}

let user = User(fistName: "John", lastName: "Snow")
try? storage.setObject(user, forKey: "character")
```

### 异步 APIs

在`async`方式中，您处理`Result`而不是`try catch`，因为result是在稍后的时间交付的，以避免阻塞当前的调用队列。在完成块中，您要么有`value`，要么有`error`。

您可以通过`storage.async`访问异步APIs，它也是线程安全的，您可以按照您想要的任何顺序使用同步和异步APIs。所有的异步函数都受到`AsyncStorageAware`协议的约束。

```swift
storage.async.setObject("Oslo", forKey: "my favorite city") { result in
  switch result {
    case .value:
      print("saved successfully")
    case .error(let error):
      print(error)
    }
  }
}

storage.async.object(ofType: String.self, forKey: "my favorite city") { result in
  switch result {
    case .value(let city):
      print("my favorite city is \(city)")
    case .error(let error):
      print(error)
    }
  }
}

storage.async.existsObject(ofType: String.self, forKey: "my favorite city") { result in
  if case .value(let exists) = result, exists {
    print("I have a favorite city")
  }
}

storage.async.removeAll() { result in
  print("removal completes")
}

storage.async.removeExpiredObjects() { result in
  print("removal completes")
}
```

### 到期日期

默认情况下，所有保存的对象与您在`DiskConfig`或`MemoryConfig`中指定的到期时间具有相同的过期时间。可以通过指定`setObject`的`expiry`来覆盖特定对象

```swift
// 配置的默认过期日期将被应用到项目中
try? storage.setObject("This is a string", forKey: "string")

// 给定的过期日期
try? storage.setObject(
  "This is a string",
  forKey: "string"
  expiry: .date(Date().addingTimeInterval(2 * 3600))
)

// 清除过期的对象
storage.removeExpiredObjects()
```

## 关于图片？

如您所知，`NSImage`和`UIImage`在默认情况下不符合`Codable`。
为了使它在可编程序中运行良好，我们引入了`ImageWrapper`，这样您就可以保存和加载图像

```swift
let wrapper = ImageWrapper(image: starIconImage)
try? storage.setObject(wrapper, forKey: "star")

let icon = try? storage.object(ofType: ImageWrapper.self, forKey: "star").image
```

如果你想把图像载入UIImageView或NSImageView，那么我们也有一个很好的礼物给你。叫做[Imaginary](https://github.com/hyperoslo/Imaginary),并使用`Cache`下罩时,让你的生活更容易处理远程图像。

## 处理JSON响应

大多数情况下，我们的用例是从后端获取json，并将json存储到存储中，以便将来使用。如果你使用库[Alamofire](https://github.com/Alamofire/Alamofire)或[Malibu](https://github.com/hyperoslo/Malibu),你是得到json形式的字典,字符串,或数据。

`Storage`可以持久存储字符串或数据。您甚至可以使用`JSONArrayWrapper`和`JSONDictionaryWrapper`将json保存到`Storage`中，但是我们更喜欢持久化强类型的对象，因为这些对象是您将在UI中显示的对象。此外，如果json数据不能转换为强类型的对象，那么保存它的意义是什么呢?😉

您可以在`JSONDecoder`上使用这些扩展来解码json字典、字符串或数据到对象。

```swift
let user = JSONDecoder.decode(jsonString, to: User.self)
let cities = JSONDecoder.decode(jsonDictionary, to: [City].self)
let dragons = JSONDecoder.decode(jsonData, to: [Dragon].self)
```

这就是如何使用`Alamofire`执行对象转换和保存

```swift
Alamofire.request("https://gameofthrones.org/mostFavoriteCharacter").responseString { response in
  do {
    let user = try JSONDecoder.decode(response.result.value, to: User.self)
    try storage.setObject(user, forKey: "most favorite character")
  } catch {
    print(error)
  }
}
```

## 安装

### Cocoapods

**Cache** 是通过[CocoaPods](http://cocoapods.org)提供的。
安装它简单地将以下行添加到您的Podfile:

```ruby
pod 'Cache'
```

### Carthage

**Cache** 是通过 [Carthage](https://github.com/Carthage/Carthage)提供的.
安装它简单地将以下行添加到您的Cartfile:

```ruby
github "hyperoslo/Cache"
```

你还需要添加 `SwiftHash.framework` 在你的 [copy-frameworks](https://github.com/Carthage/Carthage#if-youre-building-for-ios-tvos-or-watchos) 脚本.
