---
title: Micro架构和微服务的设计模式
date: 2018-03-18 21:58:35
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

在过去的几个月里，我们对微服务的micro架构和设计模式有很多疑问。所以今天我们将尝试把两者都包括进去。

<!-- more -->

## 关于Micro

Micro是一个微型服务工具包。它被构建为在它的特性和接口上的观点，同时提供一个强大的可插入架构，允许将底层的依赖项交换出去。

Micro专注于满足构建微服务的基本需求，并通过采用深思熟虑和慎重的方法来实现它的设计。

如果你想在microtoolkit上阅读，看看之前的博客文章，或者你想了解更多关于微服务的概念。

在深入讨论进一步的架构讨论之前，我们将快速回顾一下Micro的特性。

## Toolkit

[Go Micro](https://github.com/micro/go-micro)是一种可插入的RPC框架，用于在Go中编写微服务。它为服务发现、客户端负载平衡、编码、同步和异步通信提供了库。

[Micro API](https://github.com/micro/micro/tree/master/api)是一个API网关，它服务于HTTP，并将请求路由到适当的微服务。它作为一个入口点，可以作为反向代理，也可以将HTTP请求转换为RPC。

[Micro Web](https://github.com/micro/micro/tree/master/web)是一个Web仪表盘，是Micro Web应用程序的反向代理。我们认为，网络应用应该作为微服务构建，因此在微服务领域被视为一等公民。它的行为类似于API反向代理，但也包括对web sockets的支持。

[Micro Sidecar](https://github.com/micro/micro/tree/master/car)提供了go-micro作为HTTP服务的所有功能。虽然我们热爱Go，相信它是构建微服务的伟大语言，但您可能也想使用其他语言，因此Sidecar提供了一种将其他应用集成到Micro世界的方法。

[Micro CLI](https://github.com/micro/micro/tree/master/cli)是一个直接与您的微服务交互的命令行界面。它还允许您利用Sidecar作为代理，您可能不希望直接连接到服务注册中心。

这是快速的回顾。现在让我们更深。

## RPC, REST, Proto…

首先你可能会想到为什么RPC，为什么不休息?我们的信念是，RPC是跨服务通信的一个更合适的选择。或者使用protobuf IDL定义的protobuf编码和api更具体地说RPC。这种组合允许创建强定义的API接口和有效的消息编码格式。RPC是一种直接的、无附加的、用于通信的协议。

我们并不孤单。

谷歌是创建protobuf，在内部使用RPC，以及最近开源的gRPC，一个RPC框架。Hailo同时也是RPC/Protobuf的有力倡导者，并且在跨团队开发中比系统性能更有优势。优步选择了自己的路径，开发了一个名为TChannel的RPC框架协议。

就个人而言，我们认为未来的api将会使用RPC来构建，因为它们的结构格式很好，使用高效的编码协议(如protobuf)，并提供了强定义的api和性能通信。

## HTTP 到 RPC, API…

但实际上，我们在网络上的RPC还有很长的路要走。虽然它在数据中心内是完美的，但却为公众服务如网站和移动api，是另一个交互。让我们面对它，在我们离开HTTP之前会有一段时间。这是micro包含API网关、服务和传输 HTTP请求的原因之一。

API网关是用于微服务体系结构的模式。它充当外部世界的单一入口点，并根据请求提供适当的服务。这使得HTTP API本身可以由不同的微服务组成。

这是一个强大的架构模式。在过去的日子里，你的API的某个部分可能会毁掉整个整体。

micro API使用路径到服务的解析，这样每个独特的请求路径都可以由不同的API微服务来服务，例如:/user => user api, /order => order api。

这是一个例子。对**/customer/orders**的请求将通过**Customer.Orders**方法发送到API服务**go.micro.api.customer**。

![](https://micro.mu/blog/assets/images/api.png)

您可能想知道什么是API服务。现在是讨论不同类型服务的合适时机。

## 服务类型

微服务的概念是关于关注点分离的，并且从unix的哲学中借鉴了很多，并且做得很好。部分出于这个原因，我们认为需要在不同职责的服务之间进行逻辑和架构的分离。

我现在要承认，这些概念并不是什么新鲜事物，但它们是令人信服的，因为它们已经在非常成功的科技公司中得到证明。我们的目标是通过工具传播这些开发理念和指导设计决策。

这是我们目前定义的服务类型。

- **API** - 由**micro api**服务，API服务位于您的基础设施的边缘，最有可能服务于公共交通和您的移动或web应用程序。您可以使用HTTP处理程序构建它，并以反向代理模式运行micro api，或者默认地处理在这里可以找到的特定RPC api请求响应格式。

- **Web** - 由***micro web*服务，Web服务侧重于服务html内容和指示板。micro web反向代理HTTP和WebSockets。这是目前支持的唯一协议，但将来可能会扩展。如前所述，我们认为web应用程序是微服务。

- **SRV** - 这些是基于后端RPC的服务。他们主要集中于为您的系统提供核心功能，而且很可能不是面向公众的。如果您愿意，您仍然可以通过micro api或web使用/rpc端点访问它们，但更有可能的是，api、web和其他SRV服务使用go-micro客户机直接调用它们。

![](https://micro.mu/blog/assets/images/arch.png)

根据以往的经验，我们发现这种类型的体系结构模式非常强大，它可以扩展到数百种服务。通过将其构建到微架构中，我们觉得它为微服务开发提供了一个良好的基础。

## 命名空间

因此，你可能会想，是什么阻止了micro api与web服务或micro web对话的api服务。我们使用逻辑命名空间来分隔这些。通过将名称空间预先设置为服务名称，我们清楚地确定了它在系统中的用途和位置。这是一种简单而有效的模式，对我们有好处。

micro api和web将为请求路径的命名空间和第一个路径组成一个服务名称，例如请求api `/customer`成为`go.micro.api.customer`。

默认名称空间:

- **API** - go.micro.api
- **Web** - go.micro.web
- **SRV** - go.micro.srv

你应该把这些设置成你的域名，比如com.example.{api, web, srv}。micro api和micro web可以在运行时配置，以路由到您的名称空间。

## 同步/异步

你会经常听到与响应模式相同的微服务。对于许多人来说，微服务是关于创建事件驱动的架构和设计服务，这些服务主要通过异步通信进行交互。

Micro将异步通信作为一个一流的公民，是微服务的基本构建块。通过异步消息传递传递事件允许任何人消费并对其进行操作。可以构建新的独立服务，而不需要对系统的其他方面进行任何修改。这是一个强大的设计模式，因此我们已经将[Broker](https://godoc.org/github.com/micro/go-micro/broker#Broker)接口包含在了go-micro中。

![](https://micro.mu/blog/assets/images/pub-sub.png)

同步和异步通信是在Micro中作为单独的需求处理的。传输接口用于创建服务之间的点连接。go-micro客户机和服务器构建在传输上，以执行请求-响应RPC，并提供双向流的功能。

![](https://micro.mu/blog/assets/images/request-response.png)

在构建系统时，应该使用这两种通信模式，但关键是要了解何时何地都合适。在很多情况下，没有对错之分，而是会做出某些权衡。

在跟踪客户事件历史的审计跟踪系统中，可能会使用代理和异步通信的示例。

![](https://micro.mu/blog/assets/images/audit.png)

在这个示例中，每个API或服务都发布一个事件，当某些操作发生时，比如客户登录、更新他们的配置文件或下订单。审计服务将订阅这些事件并将它们存储在某个时间序列数据库中。管理员或其他人可以查看系统中为任何用户所发生的事件的历史。

如果这是一个同步调用，那么当出现高流量或定制服务的数量增加时，我们可以很容易地淹没审计服务。如果由于某些原因而关闭了审计服务，或者调用失败，我们将失去这段历史。通过将这些事件发布给代理，我们可以异步地持久化它们。这是事件驱动架构和微服务的常见模式。

## 好的，稍等一下，但什么定义了微服务呢?

我们涵盖了许多微型工具提供的微服务，我们已经定义了服务的类型(API, WEB, SRV)，但实际上并没有什么真正的微服务。

它与其他类型的应用有何不同?是什么赋予了它一个“微服务”的特殊名称。

这里有不同的定义和解释，但这里有一对在微观世界中最适合的说明。

> 松散耦合的面向服务的体系结构，具有有界的上下文。
Adrian Cockcroft

> 将单个应用程序开发为一套小型服务的方法，每个应用程序都在自己的进程中运行，并与轻量级机制进行通信。
马丁

因为我们热爱unix哲学，并且觉得它与微服务理念完美契合。

> 做一件事，做得好吗?
Doug McIlroy

我们的信念和我们构建的理念是，一个微服务是一个应用程序，专注于单一类型的实体或领域，它通过一个强大的定义的API提供访问。

让我们使用一个真实的例子，比如社交网络。

![](https://micro.mu/blog/assets/images/facebook.png)

随着Ruby on Rails的兴起，一个定义良好的软件架构模式是MVC -模型-视图-控制器。

在MVC世界中，每个实体或域将被表示为一个模型，该模型依次抽象出数据库。该模型可能与其他模型有关系，例如对许多或多个模型。控制器处理即将到来的请求，从模型中检索数据并将其传递给用户。

现在，以一个微型服务架构为例。每一个模型都是一个服务，通过一个API来传递数据。用户请求、数据收集和呈现由许多不同的web服务处理。

每个服务都有一个焦点。当我们想要添加新的特性或实体时，我们可以简单地改变一个与该特性相关的服务或编写一个新服务。这种关注点分离为可伸缩的软件开发提供了一种模式。

现在回到我们的定期计划。

## 版本控制

![](https://micro.mu/blog/assets/images/versioning.png)

版本控制是开发现实世界软件的一个重要部分。在微服务领域，考虑到API和业务逻辑在许多不同的服务中被分割，这是至关重要的。出于这个原因，服务版本控制是核心工具的一部分，允许对更新和流量形成进行更细粒度的控制。

在go-micro服务中，定义了一个名称和版本。注册表将服务作为列表返回，将节点按其注册的版本进行拆分。

这是基于版本的路由的构建块。

```go
type Service struct {
	Name      string
	Version   string
	Metadata  map[string]string
	Endpoints []*Endpoint
	Nodes     []*Node
}
```

这与选择器(客户端负载均衡器)相结合，在go-micro中确保请求在不同版本之间分布。

选择器是一个功能强大的接口，我们正在构建它来提供不同类型的路由算法;随机(默认)、轮询、基于标签、延迟等。

通过使用默认的随机散列负载平衡算法，并逐步添加新服务版本的实例，您可以执行蓝绿色部署和做canary测试。

![](https://micro.mu/blog/assets/images/selector.png)

在将来，我们将考虑实现一个全局服务负载均衡器，它连接到选择器，允许基于运行系统中的历史趋势进行路由决策。它还能够在运行时调整发送到服务的每个版本的流量的百分比，并动态地将元数据或标签添加到服务中，基于标签的路由决策可以在此基础上进行。

## 扩展

以上对版本控制的评论开始暗示了扩展服务的基本模式。虽然注册表用作存储有关服务的信息的机制，但我们使用选择器来分离路由和负载平衡的关注。

把关注点分离和做好一件事的概念。扩展基础结构和代码非常简单，定义了api和分层架构。通过创建这些构建块，我们允许自己构建更可伸缩的软件，并解决其他地方更高层次的关注。

这是Micro写作的基础，也是我们希望在微服务领域指导软件开发的方式。

我们在之前的文章中简要讨论了云架构模式，并将在这里重新讨论一些想法。

当在生产环境中部署服务时，您将考虑构建可伸缩、容错和性能的东西。云计算现在让我们获得了几乎无限的规模，但没有什么是不受失败影响的。事实上，在构建分布式系统时，我们希望解决的关键问题之一是失败，在构建基础设施时应该考虑到这一点。

在云的世界中，我们希望能够容忍可用性区域(datacenter)故障，甚至是整个区域(多DC)宕机。在过去的日子里，我们常常谈论温暖和冷备用系统或灾难恢复计划。如今，最先进的技术公司以一种全球化的方式运作，每个应用程序的多个副本都在世界各地的数字数据中心运行。

我们需要向谷歌、Facebook、Netflix和Twitter等公司学习。我们必须构建能够容忍AZ失败的系统，而不影响用户，并且在大多数情况下，在几分钟或更短时间内处理区域故障。

Micro使您能够构建这种架构。通过提供可插入的接口，我们可以利用最合适的分布式系统来满足微型工具包的每个需求。

服务发现和注册表是Micro的构建块。它可以用于分离和发现在AZ或区域内的一组服务或您所选择的任何配置。然后，Micro API可以用于路由和平衡多个服务及其在该拓扑中的实例。

![](https://micro.mu/blog/assets/images/regions.png)

## 总结

希望这篇博文能够清晰地描述微处理器的架构，以及它如何为微服务提供可伸缩的设计模式。

微服务首先是关于软件设计模式的。我们可以通过工具启用某些基本模式，同时为其他模式的出现或使用提供灵活性。

因为Micro是可插拔的架构，它是多种设计模式的强大推动者，可以在许多场景中得到适当的应用。例如，如果您正在构建视频流基础设施，您可以选择HTTP传输以点到点通信。如果您没有延迟敏感，那么您可以选择一个传输插件，比如NATS或RabbitMQ。

未来的软件开发工具如Micro，是非常令人兴奋的。