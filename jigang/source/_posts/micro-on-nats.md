---
title: "Micro在NATS - 微服务与消息"
date: 2018-03-18 20:41:09
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

在这篇文章中，我们将讨论在NATS上使用Micro。它包括关于服务发现的讨论、对微服务的同步和异步通信。

让我们开始谈正事吧。

<!-- more -->

## NATS是什么?

[NATS](http://nats.io/)是一个开源的云本地消息传递系统或更简单的消息总线。NATS是由Apcera的创始人Derek Collison创立的。它起源于VMWare，并开始以ruby为基础的系统。它早就被重写了，并且在那些寻找高度可伸缩和高性能的消息传递系统的用户中稳步获得采用。

如果你想更多地了解NATS本身，请访问[nats.io](http://nats.io/)或加入[这里](http://nats.io/community/)的社区。

## 为什么NATS ?

为什么不NATS呢?在过去处理过许多消息总线后，很明显，NATS是分开的。多年来，信息传递被认为是企业的救星，导致系统试图成为所有人的一切。它导致了很多错误的承诺，显著的特征膨胀和高成本技术，造成了比他们解决的更多的问题。

相比之下，NATS则采取了非常专注的方式，解决了性能和可用性方面的问题，同时保持了不可思议的精益。它说的是“随时待命”，并使用“fire and forget”消息模式。它的简单性、焦点和轻量级特性使它成为微服务生态系统的主要候选对象。我们相信，它将很快成为主要的候选服务，作为消息传递的服务之间的通信传输。

NATS提供:

- 较高的性能和可伸缩性
- 高可用性
- 非常轻量级的
- 最多一次交付

NATS所不提供的:

- 持久性
- 传输
- 增强的交付模式
- 企业排队

简单介绍一下什么是NATS，为什么是NATS。那么它是如何与Micro相适应的呢?让我们讨论一下。

## Micro在NATS

[Micro](https://github.com/micro/micro)是一个使用可插入架构构建的微服务工具箱，它允许将底层的依赖项以最小的变化交换出来。[Go-Micro](https://github.com/micro/go-micro)框架的每个接口都为微服务提供了一个构建块;用于服务发现的注册、同步通信的[传输](https://godoc.org/github.com/micro/go-micro/transport#Transport)、异步消息传递[代理](https://godoc.org/github.com/micro/go-micro/broker#Broker)等。

为每个组件创建一个插件就像实现接口一样简单。我们将花更多的时间详细介绍如何在以后的博客文章中编写插件。如果您想要查看NATS或其他系统(如etcd发现、kafka broker、rabbitmq传输)的插件，您可以在这里找到[github.com/micro/go-plugins](https://github.com/micro/go-plugins)。

Micro on NATS本质上是一组可用于与NATS消息系统集成的微型插件。通过为go-micro的各种接口提供插件，我们创建了许多集成点，允许选择架构模式。

在我们的经验中，一种尺寸并不适合所有的，而微操作系统的灵活性允许您定义为您和您的团队工作的模型。

下面我们将讨论用于传输、代理和注册的NATS插件的实现。

## 传输

![](https://micro.mu/blog/assets/images/request-response.png)

传输是同步通信的微型接口。它使用相当通用的Socket语义，类似于其他的Go代码，使用`Listen`、`Dial`和`Accept`。这些概念和模式对于使用tcp、http等的同步通信非常了解，但是适应消息总线可能会有些困难。与消息总线建立连接，而不是使用服务本身。为了解决这个问题，我们使用了与主题和通道有关的伪连接的概念。

这是它是如何工作的。

服务使用`transport.Listen`侦听消息。这将创建与NATS的连接。当`transport.Accept`被调用时，一个独特的主题被创建并订阅。这个独特的主题将被用作go-micro注册表中的服务地址。每个接收到的消息将被用作pseudo套接字/连接的基础。如果现有的连接存在相同的应答地址，我们将简单地将该消息放入到该连接的backlog中。

希望与此服务通信的客户端将使用`transport.Dial `来创建与服务的连接。这将连接到NATS，创建它自己独特的主题并订阅它。主题用于服务的响应。当客户端向服务发送消息时，它将为这个主题设置应答地址。

当任何一方想要关闭连接时，他们简单地调用`transport.Close`来终止与NATS的连接。

![](https://micro.mu/blog/assets/images/nats-transport.png)

### 使用传输插件

导入传输插件

```go
import _ "github.com/micro/go-plugins/transport/nats"
```

从transport标志开始

```bash
go run main.go --transport=nats --transport_address=127.0.0.1:4222
```

或者直接使用transport

```go
transport := nats.NewTransport()
```

go-micro传输接口:

```go
type Transport interface {
    Dial(addr string, opts ...DialOption) (Client, error)
    Listen(addr string, opts ...ListenOption) (Listener, error)
    String() string
}
```

## 代理

![](https://micro.mu/blog/assets/images/pub-sub.png)

代理是异步消息传递的go-micro接口。它提供了适用于大多数消息代理的高级通用实现。NATS本质上是一个异步消息传递系统，它被用作消息代理。只有一个警告，NATS不保存消息。虽然这对一些人来说可能不是理想的，但我们仍然相信NATS可以并且应该作为一个中间人使用。在不需要持久性的情况下，它允许具有高度可伸缩的pub子架构。

NATS提供了一个非常直接的发布和订阅机制，其中包含了主题、通道等概念。消息可以以异步的方式发布并忘记方式。使用相同通道名称的订阅者在NATS中形成一个队列组，然后允许消息自动均匀地分布在订阅者之间。

![](https://micro.mu/blog/assets/images/nats-broker.png)

### 使用代理插件

导入代理插件

```go
import _ "github.com/micro/go-plugins/broker/nats"
```

从broker标志开始

```bash
go run main.go --broker=nats --broker_address=127.0.0.1:4222
```

或者直接使用broker

```go
broker := nats.NewBroker()
```

go-micro代理接口:

```go
type Broker interface {
    Options() Options
    Address() string
    Connect() error
    Disconnect() error
    Init(...Option) error
    Publish(string, *Message, ...PublishOption) error
    Subscribe(string, Handler, ...SubscribeOption) (Subscriber, error)
    String() string
}
```

## 注册

![](https://micro.mu/blog/assets/images/service-discovery.png)

注册是服务发现的go-micro接口。你可能会思考。使用消息总线的服务发现?,即使是工作吗?确实如此，而且相当不错。许多使用消息总线的人将避免使用任何类型的独立发现机制。这是因为消息总线本身可以通过主题和通道来处理路由。定义为服务名称的主题可以用作路由键，自动在订阅该主题的服务实例之间进行负载平衡。

Go-micro将服务发现和传输机制视为两个不同的关注点。当客户端向另一个服务发出请求时，在覆盖层之下，它会根据名称查找注册表中的服务，选择一个节点的地址，然后通过传输与它通信。

通常，存储服务发现信息的最常用方法是通过一个分布式的键值存储库，如zookeeper, etcd或类似的东西。您可能已经意识到,NATS分布式键值存储不是一个,所以我们要做一些有点不同…

### 广播查询!

广播查询就像你想象的那样。服务侦听我们认为用于广播查询的特定主题。任何想要服务发现信息的人首先会创建一个它订阅的回复主题，然后用他们的回复地址对广播主题进行查询。

因为我们实际上不知道一个服务运行的实例有多少，或者返回的响应有多少，所以我们在等待响应的时候设置了一个上限。这是一种分散收集的粗糙机制，但由于NATs的可扩展性和性能，它实际上工作得非常好。它还间接提供了一种非常简单的过滤服务的方式，响应时间也更高。在将来，我们将考虑改进底层实现。

总结一下它是如何工作的:

1. 创建应答主题并订阅
1. 以回复地址发送广播主题查询
1. 在一个时间限制后，倾听回应和取消订阅
1. 聚合响应和返回结果

![](https://micro.mu/blog/assets/images/nats-registry.png)

### 使用注册中心插件

导入registry插件

```go
import _ "github.com/micro/go-plugins/registry/nats"
```

从registry标志开始

```bash
go run main.go --registry=nats --registry_address=127.0.0.1:4222
```

或者直接使用registry

```go
registry := nats.NewRegistry()
```

go-micro registry接口:

```go
type Registry interface {
    Register(*Service, ...RegisterOption) error
    Deregister(*Service) error
    GetService(string) ([]*Service, error)
    ListServices() ([]*Service, error)
    Watch() (Watcher, error)
    String() string
}
```

## NATS上加强Micro

在上面的示例中，我们只在localhost上指定一个单独的NATS服务器，但是我们推荐的实际应用程序是为高可用性和容错性设置一个NATS集群。要了解更多关于NATs集群检查的[NATs文档](http://nats.io/documentation/server/gnatsd-cluster/)。

Micro接受一个逗号分隔的地址列表，就像上面提到的标记或者可以选择使用环境变量。如果您直接使用客户端库，那么它也允许作为初始化注册、传输和代理的一种可变的主机集。

在一个云原生世界的架构方面，我们过去的经验表明，每个AZ或每个区域的集群是理想的。大多数云提供商在AZs之间有相对较低的(3-5ms)延迟，这使得区域集群没有问题。当运行一个高度可用的配置时，重要的是确保您的系统能够容忍AZ的失败，并且在更成熟的配置中能够容忍整个区域的失败。我们不建议跨区域进行集群。理想情况下，应该使用更高级别的工具来管理多集群和多区域系统。

Micro是一个非常灵活的运行时不可知的微服务系统。它可以在任何地方和任何配置中运行。它的世界观是由服务注册中心指导的。服务集群可以在一组机器、AZs或区域中进行本地化和命名空间，这完全基于您提供服务访问的注册中心。结合NATS集群，它允许您构建一个高度可用的体系结构来满足您的需求。

![](https://micro.mu/blog/assets/images/region.png)

## 总结

NATS是一个可扩展和性能的消息传递系统，我们认为它非常适合于微服务生态系统。它与Micro的关系非常好，正如我们所演示的，它可以作为[注册表](https://godoc.org/github.com/micro/go-plugins/registry/nats)、[传输](https://godoc.org/github.com/micro/go-plugins/transport/nats)或[代理](https://godoc.org/github.com/micro/go-plugins/broker/nats)的插件。我们已经实现了这三种功能，以强调NATS的灵活性。

微观上的NATS是微观强大的可插入架构的一个例子。每一个微型软件包都可以实现并以最小的变化交换出去。在未来，我们将会看到更多关于微观的例子。

希望这能激励你在NATS上试用Micro，甚至为其他系统编写一些插件，并回馈社区。

在[github.com/micro/go-plugins](https://github.com/micro/go-plugins)中找到NATS插件的源代码。


