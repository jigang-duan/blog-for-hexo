---
title: "Micro - 微服务工具包"
date: 2018-03-18 18:15:47
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

到现在为止，你可能已经听说过这种新的现象，微服务。
如果你还不熟悉，而且对学习感兴趣，请在查看[这里](/2018/03/18/micro-introduction/)。

在这篇博文中，我们将讨论[Micro](https://github.com/micro/micro)，一个开源的微服务工具包。Micro提供了构建和管理微服务的核心需求。它由一系列的库和工具组成，这些库和工具主要面向[Go](https://golang.org/)语言的编程开发，但是通过使用[Sidecar](https://github.com/micro/micro/tree/master/car)来解决其他语言的问题。

在我们了解微观的细节之前，让我们来谈谈为什么我们决定把时间花在这上面。

<!-- more -->

## 开发或部署

从我们过去的经验和我们在业界所看到的情况来看，我们需要关注开发而不是部署。PaaS解决方案是现成的。AWS、谷歌和微软等公司都提供了功能丰富的平台，而且如果不这样做的话，它们也会迅速转向支持容器编排。所有这一切使我们可以通过点击几个按钮来访问大规模的计算。

这个新世界听起来不错。你可能会说这解决了你所有的问题，对吧?虽然我们现在可以使用大规模的计算能力，但是我们仍然缺乏能够使我们编写能够利用它的软件的工具。不仅如此，在这个新的世界中，容器可能会更加短暂，随着运行时重新安排它们或它们正在运行的机器的失败，容器会变得更加短暂。

## 扩展的挑战

我们经常看到的另一个问题是，组织是如何成为其整体架构的牺牲品的。由于需要以极快的速度增长，有一种趋势是将功能塞进现有系统，并导致技术债务，使雪球陷入无法控制的局面。除此之外，当组织试图增加工程团队的时候，开发人员在单个代码基础上进行协作，或者并行地进行特性开发，而不在发布时被阻塞，这将变得更加困难。

对于SOA或基于微服务的架构来说，重构和最终路径是不可避免的。公司最终会在房子里进行研发工作，通过尝试和错误来学习。如果有一些工具可以帮助创建可伸缩的系统，减少研发的可能性，并从过去的经验中提供专业知识。

## 进入Micro

在Micro方面，我们正在构建一个微服务生态系统，其中包括用于微服务开发的工具、服务和解决方案。我们用一个同名的工具来构建这个生态系统的基础。Micro是一个微型服务工具包，它可以创建可伸缩的体系结构，并提高执行速度。

让我们深入了解一下Micro的特征。

## <a name="go-micro"></a>Go Micro

[Go Micro](https://github.com/micro/go-micro)是一个可插入的RPC框架，用于在Go中构建微服务。它提供了创建、发现和与服务通信所需的基本特性。任何优秀的微服务体系结构的核心都始于服务发现、同步和异步通信。

包括包装和特点:

- Registry - 客户端服务发现
- Transport - 同步通信
- Broker - 异步通信
- Selector - 节点过滤和负载平衡
- Codec - 消息编码/解码
- Server - 构建在上面的RPC服务器
- Client - 构建在上面的RPC客户端

与大多数库不同的是，它是可插拔的体系结构。这允许将每个包的实现和后端系统交换出去。例如;注册表的默认服务发现机制是[Consul](https://www.consul.io/)，但这可以很容易地与etcd、zookeeper或其他您选择实现的插件交换。我们正在实现的插件可以在[github.com/micro/go-plugins](https://github.com/micro/go-plugins)上找到。

可插拔系统的价值在于能够选择用于支持微服务的平台，而无需重写任何代码。Go Micro要求零代码更改，只需导入一个插件就可以了。

Go Micro是写微服务的起点。readme提供了如何编写、运行和查询服务的概述。这里有一个类似的例子，在[repo github.com/micro](https://github.com/micro)中提供了[micro/example/greeter](https://github.com/micro/examples/tree/master/greeter)和更多的示例服务。

## Sidecar

Go Micro提供了一种方式Go来编写服务，但是其他语言呢?我们如何创建一个polygot生态系统，让任何人都能利用Micro的优势?虽然Micro是写在Go上的，但是我们想让一个快速简单的方法来集成用任何语言编写的应用程序。

进入[Sidecar](https://github.com/micro/micro/tree/master/car)，这是一种轻量级的伙伴服务，它在概念上“附加”到主(即父级)应用程序，并通过提供Micro系统的特性来补充它。sidecar是一个与您的应用程序并行运行的进程，它通过一个HTTP接口实现了Go Micro的特性。

sidecar的特点:

- 注册与发现系统
- 主机发现其他服务
- 主应用程序的健康检查
- 用于发出RPC请求的代理
- 通过WebSockets的PubSub

![](https://micro.mu/blog/assets/images/sidecar.png)

可以在[这里](https://github.com/micro/examples/tree/master/greeter)找到使用带有ruby或python的Sidecar的例子。我们将在不久的将来添加更多的示例代码，以帮助理解如何集成sidecar。

## API

将RPC请求从一个服务发送到另一个服务非常简单，但对于外部访问并不理想。服务的实例可能会失败，它们可能被重新安排到其他地方，或者最终绑定到任意的端口。该[API](https://github.com/micro/micro/tree/master/api)为查询微服务提供了一个单一入口点，并应作为外部访问的网关。

该API提供了一些不同类型的请求处理程序。

### /rpc

可以使用/rpc端点通过RPC查询单个服务。例子:

```bash
curl \
	-d "service=go.micro.srv.greeter" \
	-d "method=Say.Hello" \
	-d "request={\"name\": \"John\"}" \
	http://localhost:8080/rpc

{"msg":"Hello John"}
```

### api.Request

该API可用于分解由单个微服务提供的url。这是一种强大的API组合方法。在这里，API使用请求路径的第一部分以及名称空间组件来确定要路由请求的服务。然后，HTTP请求被转换为一个api.Request，并适当地转发。

在Micro上，我们使用创建API微服务的模式来服务于边缘的请求。分离后端和前端服务的职责。

API请求处理的一个例子:

请求

```
GET /greeter/say/hello?name=John
```

变成

```
service: go.micro.api.greeter (default namespace go.micro.api is applied)
method: Say.Hello
request {
	"method": "GET",
	"path": "/greeter/say/hello",
	"get": {
		"name": "John"
	}
}
```

api.Request 和 api.Response的结构:

```
syntax = "proto3";

message Pair {
	optional string key = 1;
	repeated string values = 2;
}

message Request {
	optional string method = 1;   // GET, POST, etc
	optional string path = 2;     // e.g /greeter/say/hello
	map<string, Pair> header = 3; 
	map<string, Pair> get = 4;    // The URI query params
	map<string, Pair> post = 5;   // The post body params
	optional string body = 6;     // raw request body; if not application/x-www-form-urlencoded
}

message Response {
	optional int32 statusCode = 1;
	map<string, Pair> header = 2;
	optional string body = 3;
}
```

可以在这里找到如何创建API服务的示例。 [Greeter API](https://github.com/micro/micro/blob/master/examples/greeter/api/api.go)

### proxy

API的请求处理的最终方法是反向代理。正如上面所述，API使用请求路径和名称空间组件来确定要路由请求的服务。通过提供反向代理和微服务请求路由，我们能够支持REST，这是一种广泛追求的需求。

通过传递--api_handler=proxy标志，可以启用代理。

如何构建一个RESTful API的一个例子可以在这里找到[micro/examples/greeter/api](https://github.com/micro/examples/tree/master/greeter/api/rest)。

## Web UI

[web UI](https://github.com/micro/micro/tree/master/web)提供了一个简单的指示板，用于观察和与运行的系统交互。不仅如此，它还提供了与API非常相似的反向代理。我们的“web代理”的目标是使web应用程序的开发成为微服务。同样，就像API一样，请求路径与名称空间一起使用，以确定要路由请求的服务。web代理还支持web sockets，因为我们认为实时是交付web应用程序的核心部分。

![web ui](https://micro.mu/blog/assets/images/web.png)

## CLI

CLI是一个命令行工具，它提供了在运行环境中观察、交互和管理服务的方法。当前的特性集允许您检查注册表、检查服务的基本健康状况并对服务本身执行查询。

![cli](https://micro.mu/blog/assets/images/cli.png)

另一个很好的特性是使用Sidecar作为CLI的代理的能力。这就像在执行CLI时指定sidecar的地址作为标志一样简单--proxy_address=example.proxy.com。

## 把它放在一起

我们已经通过使用简单的greeter服务编写了一个完整的端到端流程示例。

流程如下:

1. HTTP GET请求name=John被发送到/greeter/say/hello的micro API
1. API将这个使用默认名称空间转换为API服务go.micro.api.greeter和方法Say.Hello。请求的结构是一个api.Request。
1. 使用Go Micro的API，查询注册表以查找服务go.micro.api的所有节点。将请求转发给节点。
1. greeter api解析请求，生成hello。请求并请求rpc服务go.micro.srv.greeter。同样，使用相同的注册表/发现机制来查找服务的节点。
1. greeter rpc服务使用hello.Response响应。
1. greeter api将响应转换为api.Response并将其传回API。
1. micro API解析响应并响应客户机的HTTP请求。

![](https://micro.mu/blog/assets/images/greeter.png)

在一个更复杂的示例中，API服务可以调用许多其他RPC服务，聚合和转换数据，然后将最终的汇总结果传递给客户机。这允许您在不了解客户机的情况下，在后台维护一致的外部入口点和更改服务。

## Dome

如果你想在运行的系统上，请在[web.micro.pm](http://web.micro.pm/)处查看我们的演示。

我们正在使用谷歌集装箱引擎在Kubernetes上运行Micro。演示是开源的，如果您想自己运行它。你可以在[这里github.com/micro/kubernetes](https://github.com/micro/kubernetes)找到k8s配置。

## 总结

Micro为编写和管理微服务提供了基本的构建模块。微观包括核心需求;发现、客户机/服务器和发布/订阅。CLI让您与环境和服务交互。sidecar可以集成任何非微应用程序。该API是rpc请求的单一入口点，可以创建REST端点。有了可插入的接口，您可以选择并选择您想要利用的系统来构建您的体系结构。

我们在Micro上的目标是在规模上实现开发，提高执行速度，并从开发人员生命周期的最初阶段开始提供价值。我们觉得Micro是做所有这些事情的最好方法。随着时间的推移，工具的生态系统将会发展到包含更多的功能丰富的服务，用于发现、路由和可转换性。

如果你想了解更多关于我们提供的服务或微服务，请检查[micro.mu](https://micro.mu)。或者是[github repo](https://github.com/micro/micro)。
