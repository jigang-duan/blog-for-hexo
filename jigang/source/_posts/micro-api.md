---
title: micro api
date: 2018-03-20 20:57:18
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

**micro api**是微服务的api网关。使用API网关[模式](http://microservices.io/patterns/apigateway.html)为您的服务提供一个单一入口点。micro api使用服务发现服务HTTP和动态路由到适当的后端。

<!-- more -->

![](https://github.com/micro/docs/raw/master/images/api.png)

## 概述

micro api是一个HTTP api。对API的请求通过HTTP提供，并通过RPC在内部路由。它建立在[go-micro](https://github.com/micro/go-micro)的基础上，利用它进行服务发现、负载平衡、编码和基于RPC的通信。

因为micro api在内部使用了micro，这也使得它成为可插拔的。参见[go-plugins](https://github.com/micro/go-plugins)支持gRPC、kubernetes、etcd、nats、rabbitmq等。

## 入门指南

### 安装

```bash
go get -u github.com/micro/micro
```

### 运行

```bash
micro api
```

### ACME

默认使用ACME安全服务

```bash
MICRO_ENABLE_ACME=true micro api
```

可以选择指定一个主机白名单

```bash
MICRO_ENABLE_ACME=true \
MICRO_ACME_HOSTS=example.com,api.example.com \
micro api
```

### TLS证书

API支持安全地使用TLS证书

```bash
MICRO_ENABLE_TLS=true \
MICRO_TLS_CERT_FILE=/path/to/cert \
MICRO_TLS_KEY_FILE=/path/to/key \
micro api
```

### 设置名称空间

API利用名称空间来逻辑地分离后端和面向公众的服务。名称空间和http路径用于解析服务name/method。默认的名称空间是`go.micro.api`。

```bash
MICRO_NAMESPACE=com.example.api micro api
```

## 例子

这里我们有一个3层架构的例子

- `micro api`: (localhost:8080) - 作为http入口点。
- `api服务`:提供面向公众的api服务。
- `后端服务`:(go.micro.srv.greeter) - 内部范围服务。

完整的例子是[examples/greeter](https://github.com/micro/examples/tree/master/greeter)

### 运行示例

> 确保您正在运行服务发现 e.g `consul agent -dev`

得到例子

```bash
git clone https://github.com/micro/examples
```

启动该服务

```bash
go run examples/greeter/srv/main.go
```

启动API

```bash
go run examples/greeter/api/api.go
```

启动 micro api

```bash
micro api
```

### 查询

通过micro api进行HTTP调用

```bash
curl "http://localhost:8080/greeter/say/hello?name=John"
```

HTTP路径/greeter/say/hello映射到服务go.micro.api.greeter Say.Hello方法

绕过api服务，直接通过/rpc调用后

```bash
curl -d 'service=go.micro.srv.greeter' \
     -d 'method=Say.Hello' \
     -d 'request={"name": "John"}' \
     http://localhost:8080/rpc
```

完全相同地调用JSON

```bash
curl -H 'Content-Type: application/json' \
     -d '{"service": "go.micro.srv.greeter", "method": "Say.Hello", "request": {"name": "John"}}' \
     http://localhost:8080/rpc
```

## API

micro api提供了以下HTTP api

```bash
- /[service]/[method]	# HTTP路径被动态映射到服务
- /rpc			# 通过名称和方法显式地调用后端服务
```

参见下面的例子

## Handlers

Handlers是管理请求路由的HTTP处理程序。

默认handler使用注册中心的端点元数据来确定服务路由。如果没有找到匹配的路由，它将返回到API handler。您可以使用[go-api](https://github.com/micro/go-api)配置注册路由。

API具有以下可配置的请求handler

- [api]() -处理任何HTTP请求。通过RPC完全控制http请求/响应。
- [event]()-处理任何HTTP请求并发布到消息总线。
- [http]() -处理任何http请求和转发作为反向代理。
- [rpc]() -处理json和protobuf POST请求。向前 RPC。
- [web]() -包含web套接字支持的HTTP handler。

可以选择使用[/rpc]()端点绕过handlers

### API Handler

- Content-Type: Any
- Body: Any
- Forward Format: [api.Request](https://github.com/micro/go-api/blob/master/proto/api.proto#L11)/[api.Response](https://github.com/micro/go-api/blob/master/proto/api.proto#L21)
- 路径: `/[service]/[method]`
- 解析器:路径用于解析服务和方法。
- Configure: Flag `--handler=api`或`MICRO_API_HANDLER=api`。
- 没有指定handler时的默认handler。

### Event Handler

事件handler服务于HTTP，并将请求转发给使用go-micro代理的消息总线上的消息。

- Content-Type: Any
- Body: Any
- Forward Format:请求被格式化为[go-api/proto.Event](https://github.com/micro/go-api/blob/master/proto/api.proto#L28L39)
- Path: `/[topic]/[event]`
- 解析器:路径用于解析主题和事件名称。
- Configure: Flag `--handler=event`或 `MICRO_API_HANDLER=event`

### HTTP Handler

http handler是一个基于服务发现的http备用代理。

- Content-Type: Any
- Body: Any
- Forward Format: HTTP反向代理
- Path: `/[service]`
- 解析器:路径用于解析服务名称
- 配置:Flag `—handler=http`或`MICRO_API_HANDLER=http`
- REST可以作为微服务在API后面实现


### RPC Handler

RPC handler以RPC请求的形式提供json或protobuf HTTP POST请求和转发。

- Content-Type: application/json 或 application/protobuf
- Body: JSON  或 Protobuf
- 正向格式:基于内容的json-rpc或proto-rpc。
- Path: `/ [service] / [method]`
- 解析器:路径用于解析服务和方法。
- 配置:Flag `—handler=rpc`或`MICRO_API_HANDLER=rpc`

### Web Handler

web handler是一个基于服务发现和web socket支持的http反向代理。

- Content-Type: Any
- Body: Any
- 正向格式:HTTP反向代理，包括web sockets。
- Path: `/[service]`
- 解析器:路径用于解析服务名称。
- 配置:Flag `—handler=web` 或 `MICRO_API_HANDLER=web`

### RPC端点

/rpc端点让我们绕过主handler直接与任何服务对话

- 请求参数
    - service - 设置服务名称
    - method - 设置服务方法
    - request - 请求体
    - address - 可选地指定主机地址到目标

示例调用:

```bash
curl -d 'service=go.micro.srv.greeter' \
     -d 'method=Say.Hello' \
     -d 'request={"name": "Bob"}' \
     http://localhost:8080/rpc
```

在[github.com/micro/examples/api](https://github.com/micro/examples/tree/master/api)找到工作的例子

## 解析器

使用名称空间值和HTTP路径到服务的Micro动态路由。

默认的名称空间是go.micro.api。通过`--namespace`或`MICRO_NAMESPACE=`设置名称空间

使用的解析器如下所示。

### RPC解析器

RPC服务有一个名称(go.micro.api.greeter)和一个方法(Greeter.Hello)。

URLs的解析如下:

Path |	Service	| Method 
-- | -- | --
/foo/bar |	go.micro.api.foo |	Foo.Bar
/foo/bar/baz |	go.micro.api.foo |	Bar.Baz
/foo/bar/baz/cat |	go.micro.api.foo.bar |	Baz.Cat


可以很容易地将版本化的API URLs映射到服务名称:

Path	 | Service	 | Method
-- | -- | --
/foo/bar	 | go.micro.api.foo	 | Foo.Bar
/v1/foo/bar	 | go.micro.api.v1.foo	 | Foo.Bar
/v1/foo/bar/baz	 | go.micro.api.v1.foo	 | Bar.Baz
/v2/foo/bar	 | go.micro.api.v2.foo	 | Foo.Bar
/v2/foo/bar/baz	 | go.micro.api.v2.foo	 | Bar.Baz

### HTTP解析器

对于http handler，我们只需要处理服务名称的解析。所以分辨率与RPC解析器略有不同。

URLs的解析如下:

Path | 	Service	Service  | Path
-- | -- | --
/foo	 | go.micro.api.foo	 | /foo
/foo/bar	 | go.micro.api.foo	 | /foo/bar
/greeter	 | go.micro.api.greeter	 | /greeter
/greeter/:name	 | go.micro.api.greeter	 | /greeter/:name

#### Go Restful API的例子

[这里](https://github.com/micro/examples/tree/master/greeter/api/rest)是如何使用go-restful在API后面服务REST的一个示例:

```go
package main

import (
	"log"

	"github.com/emicklei/go-restful"

	hello "github.com/micro/examples/greeter/srv/proto/hello"
	"github.com/micro/go-micro/client"
	"github.com/micro/go-web"

	"context"
)

type Say struct{}

var (
	cl hello.SayClient
)

func (s *Say) Anything(req *restful.Request, rsp *restful.Response) {
	log.Print("Received Say.Anything API request")
	rsp.WriteEntity(map[string]string{
		"message": "Hi, this is the Greeter API",
	})
}

func (s *Say) Hello(req *restful.Request, rsp *restful.Response) {
	log.Print("Received Say.Hello API request")

	name := req.PathParameter("name")

	response, err := cl.Hello(context.TODO(), &hello.Request{
		Name: name,
	})

	if err != nil {
		rsp.WriteError(500, err)
	}

	rsp.WriteEntity(response)
}

func main() {
	// Create service
	service := web.NewService(
		web.Name("go.micro.api.greeter"),
	)

	service.Init()

	// setup Greeter Server Client
	cl = hello.NewSayClient("go.micro.srv.greeter", client.DefaultClient)

	// Create RESTful handler
	say := new(Say)
	ws := new(restful.WebService)
	wc := restful.NewContainer()
	ws.Consumes(restful.MIME_XML, restful.MIME_JSON)
	ws.Produces(restful.MIME_JSON, restful.MIME_XML)
	ws.Path("/greeter")
	ws.Route(ws.GET("/").To(say.Anything))
	ws.Route(ws.GET("/{name}").To(say.Hello))
	wc.Add(ws)

	// Register Handler
	service.Handle("/", wc)

	// Run server
	if err := service.Run(); err != nil {
		log.Fatal(err)
	}
}
```

##### 运行 Micro API

```bash
$ micro api --handler=proxy
```

##### 运行 Greeter Service

```bash
$ go run greeter/server/main.go
```

##### 运行 Greeter API

```bash
$ go run go-restful.go
Listening on [::]:64738
```

##### Curl API

测试index

```bash
curl http://localhost:8080/greeter
{
  "message": "Hi, this is the Greeter API"
}
```

测试资源

```bash
curl http://localhost:8080/greeter/asim
{
  "msg": "Hello asim"
}
```