---
title: 用Go Micro编写微服务
date: 2018-03-18 19:30:53
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

这是一个高水平的指导，用go-micro来编写微服务。

如果你想了解更多有关微服务的内容，请点击[这里](/2018/03/18/micro-introduction/)的入门博客文章，如果你想了解更多关于Micro的信息，请点击[这里](/2018/03/18/micro/)。

让我们开始吧。

<!-- more -->

## Go Micro是什么?

[Go Micro](https://github.com/micro/go-micro)是一个可插入的RPC库，它提供了用于编写微服务的基本构件。微哲学是用可插拔的体系结构设计的“电池”。从这个框中，它实现了使用代理的服务发现，通过http和使用proto-rpc或json-rpc进行编码

Go Micro是:

1. Go写的库
1. 一组可插入的接口
1. 基于RPC

Go Micro提供了接口:

1. 服务发现
1. 编码
1. 客户端/服务端
1. 发布/订阅

[这里](/2018/03/18/micro#go-micro)可以找到更详细的分类。

## 为什么要Go Micro?

一年前就开始做Go Micro，最初是为个人服务。不久之后，很明显，它对更广泛的受众来说也很有价值，他们也希望编写微服务。这是基于在谷歌和Hailo等规模经营微服务平台的各种科技公司的经验。

正如前面提到的，Go Micro是一个可插入的体系结构，它侧重于提供基于Go的接口，这些接口在一起使用时提供了用于编写微服务的构件。这些接口可以通过具体的实现来满足。例如，服务发现的[注册](https://godoc.org/github.com/micro/go-micro/registry)接口有一个默认的Consul实现，但是可以用etcd、zookeeper或其他任何能够满足接口的功能交换。

如果您想要交换底层技术，可插拔的体系结构意味着零代码重写。

让我们开始写一个服务吧。

## 写一个服务

如果您想直接阅读代码，请查看[examples/service](https://github.com/micro/examples/tree/master/service)。

顶层[服务](https://godoc.org/github.com/micro/go-micro#Service)接口是构建服务的主要组件。它将所有的底层Go Micro包封装到一个方便的接口中。

```
type Service interface {
    Init(...Option)
    Options() Options
    Client() client.Client
    Server() server.Server
    Run() error
    String() string
}
```

### 1. 初始化

服务是用`micro.NewService`创建的。

```go
import "github.com/micro/go-micro"

service := micro.NewService() 
```

选项可以在创建期间传入。

```go
service := micro.NewService(
	micro.Name("greeter"),
	micro.Version("latest"),
)
```

[这里](https://godoc.org/github.com/micro/go-micro#Option)可以找到所有可用的选项。

Go Micro还提供了一种使用micro.Flags设置命令行标志的方法。

```go
import (
	"github.com/micro/cli"
	"github.com/micro/go-micro"
)

service := micro.NewService(
	micro.Flags(
		cli.StringFlag{
			Name:  "environment",
			Usage: "The environment",
		},
	)
)
```

要解析标记使用service.Init。另外，访问标志使用micro.Action 选项。

```go
service.Init(
	micro.Action(func(c *cli.Context) {
		env := c.StringFlag("environment")
		if len(env) > 0 {
			fmt.Println("Environment set to", env)
		}
	}),
)
```

Go Micro提供预定义的标志，如果调用了service.Init，就会设置和解析这些标志。看[这里](https://godoc.org/github.com/micro/go-micro/cmd#pkg-variables)所有的flags。

### 2. 定义API

我们使用protobuf文件来定义服务API接口。这是严格定义API并为服务器和客户机提供具体类型的非常方便的方法。

这里有一个例子定义。

greeter.proto

```
syntax = "proto3";

service Greeter {
	rpc Hello(HelloRequest) returns (HelloResponse) {}
}

message HelloRequest {
	string name = 1;
}

message HelloResponse {
	string greeting = 2;
}
```

在这里，我们定义了一个服务处理程序，叫做Greeter，方法是Hello，它使用参数HelloRequest类型并返回HelloResponse。

### 3. 生成的API接口

我们使用protoc和protoc-gen-go为这个定义生成具体的go实现。

Go-micro使用代码生成来提供客户端存根方法来减少板代码，就像gRPC一样。它是通过一个protobuf插件需要golang/protobuf,可以在这里找到github.com/micro/protobuf。

```bash
go get github.com/micro/protobuf/{proto,protoc-gen-go}
protoc --go_out=plugins=micro:. greeter.proto
```

生成的类型现在可以导入，并在请求时在服务器或客户机的处理程序中使用。

这是生成代码的一部分。

```go
type HelloRequest struct {
	Name string `protobuf:"bytes,1,opt,name=name" json:"name,omitempty"`
}

type HelloResponse struct {
	Greeting string `protobuf:"bytes,2,opt,name=greeting" json:"greeting,omitempty"`
}

// Client API for Greeter service

type GreeterClient interface {
	Hello(ctx context.Context, in *HelloRequest, opts ...client.CallOption) (*HelloResponse, error)
}

type greeterClient struct {
	c           client.Client
	serviceName string
}

func NewGreeterClient(serviceName string, c client.Client) GreeterClient {
	if c == nil {
		c = client.NewClient()
	}
	if len(serviceName) == 0 {
		serviceName = "greeter"
	}
	return &greeterClient{
		c:           c,
		serviceName: serviceName,
	}
}

func (c *greeterClient) Hello(ctx context.Context, in *HelloRequest, opts ...client.CallOption) (*HelloResponse, error) {
	req := c.c.NewRequest(c.serviceName, "Greeter.Hello", in)
	out := new(HelloResponse)
	err := c.c.Call(ctx, req, out, opts...)
	if err != nil {
		return nil, err
	}
	return out, nil
}

// Server API for Greeter service

type GreeterHandler interface {
	Hello(context.Context, *HelloRequest, *HelloResponse) error
}

func RegisterGreeterHandler(s server.Server, hdlr GreeterHandler) {
	s.Handle(s.NewHandler(&Greeter{hdlr}))
}
```

### 4. 实现处理程序

服务器需要为服务请求注册*** handlers*。handler是一种public类型，其public方法符合签名`func(ctx context.Context, req interface{}, rsp interface{}) error`。

正如您在上面看到的，一个用于Greeter接口的处理程序签名看起来是这样的。

```go
type GreeterHandler interface {
        Hello(context.Context, *HelloRequest, *HelloResponse) error
}
```

下面是Greeter handler的一个实现

```go
import proto "github.com/micro/examples/service/proto"

type Greeter struct{}

func (g *Greeter) Hello(ctx context.Context, req *proto.HelloRequest, rsp *proto.HelloResponse) error {
	rsp.Greeting = "Hello " + req.Name
	return nil
}
```

handler在您的服务中注册的很像http.Handler。

```go
service := micro.NewService(
	micro.Name("greeter"),
)

proto.RegisterGreeterHandler(service.Server(), new(Greeter))
```

您还可以创建双向流handler，但我们将把它留到以后的一天。

### 5. 运行服务

greeter.go

该服务可以通过调用`server.Run`来运行。这会导致服务绑定到config中的地址(默认设置为找到的第一个RFC1918接口和一个随机端口)并侦听请求。

这将在发出一个kill信号时，在开始时和注销时对服务进行注册。

```go
if err := service.Run(); err != nil {
	log.Fatal(err)
}
```

### 6. 完整的服务

greeter.go

```go
package main

import (
        "log"

        "github.com/micro/go-micro"
        proto "github.com/micro/examples/service/proto"

        "golang.org/x/net/context"
)

type Greeter struct{}

func (g *Greeter) Hello(ctx context.Context, req *proto.HelloRequest, rsp *proto.HelloResponse) error {
        rsp.Greeting = "Hello " + req.Name
        return nil
}

func main() {
        service := micro.NewService(
                micro.Name("greeter"),
                micro.Version("latest"),
        )

        service.Init()

        proto.RegisterGreeterHandler(service.Server(), new(Greeter))

        if err := service.Run(); err != nil {
                log.Fatal(err)
        }
}
```

请注意。服务发现机制需要运行，因此服务可以注册为客户机和其他服务发现。这里有一个[快速入门](https://github.com/micro/go-micro#getting-started)。

## 编写一个客户端

客户端包用于查询服务。当您创建服务时，将包含一个与服务器使用的初始包相匹配的客户端。

查询上述服务如下所示。

```go
// create the greeter client using the service name and client
greeter := proto.NewGreeterClient("greeter", service.Client())

// request the Hello method on the Greeter handler
rsp, err := greeter.Hello(context.TODO(), &proto.HelloRequest{
	Name: "John",
})
if err != nil {
	fmt.Println(err)
	return
}

fmt.Println(rsp.Greeter)
```

`proto.NewGreeterClient`使用服务名称和用于发出请求的客户端。

完整的例子可以在[go-micro/example/service](https://github.com/micro/examples/tree/master/service)中找到。

## 总结

希望这篇博文是一个有用的高级指南，可以用[Go Micro](https://github.com/micro/go-micro)来编写微服务。您可以在[repo github.com/micro](https://github.com/micro)中找到更多的示例服务，以帮助您进一步了解更真实的世界解决方案。

如果你想了解更多关于我们提供的服务或微服务，请检查[micro.mu](https://micro.mu)。或者是[github repo](https://github.com/micro/micro)。
