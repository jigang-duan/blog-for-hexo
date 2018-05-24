---
title: Go微服务教程-第一部分
date: 2018-05-10 21:56:15
tags:
 - golang
 - go-micro
categories:
 - Go微服务

---

![micro-diag](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526284371762&di=4d74aecb4c8b43534361e1c232276d81&imgtype=0&src=http%3A%2F%2Fi0.wp.com%2Fwww.virtuaniz.com%2Fwp-content%2Fuploads%2F2014%2F08%2Fcloud-computing-advantages-benifits-virtuaniz.png%3Fresize%3D660%2C330)

# 介绍

这是一个十节Golang的微服务的系列。利用protobuf和gRPC作为底层传输协议。为什么?因为我花了很长时间才弄明白并解决了一个清晰而简洁的解决方案，我想要分享我在创建、测试和部署微服务的过程中所了解到的东西，这些服务是与其他新出现的用户端到一起的。

在本教程中，我们将介绍一些基本概念、术语，并以最原始的形式创建我们的第一个微服务。

<!-- more -->

我们将在整个系列中创建以下服务:

- 货物
- 库存
- 用户
- 身份验证
- 角色
- 船只

我们最终将会用到:golang, mongodb, grpc, docker，谷歌云，Kubernetes, NATS, CircleCI, Terraform和go-micro。

接下来，您可以按照我所包含的步骤进行操作，但是一定要使用[git repo](https://github.com/EwanValentine/shippy)(每一篇文章都有自己的分支)作为参考，将GOPATH更改为您自己的步骤。

同时注意，我正在使用Macbook，所以您可能需要修改makefile以使用$GOPATH而不是$(GOPATH)，操作系统之间可能还有其他一些不一致的地方。

# 先决条件

- 对Golang及其生态系统的理解
- 安装gRPC / protobuf - 看这里
- 安装Golang - 看这里
- 安装以下的go库:

```bash
go get -u google.golang.org/grpc
go get -u github.com/golang/protobuf/protoc-gen-go
```

# 我们构建什么

我们将构建一个您能想到的最通用的微服务示例，一个集装箱管理平台!一个博客对微服务的用例太过简单，我想要一些能真正展示复杂性分离的东西。所以这是一个很好的挑战!

让我们从基础开始:

# 什么是微服务

在传统的庞大应用程序中，所有的组织特性都被写入到一个应用程序中。有时，它们被它们的类型分组，比如控制器、实体、工厂等等。其他时候，可能在更大的应用程序中，特性会被关注点或特性分隔开来。因此，您可能会有一个auth包、一个朋友包和一个文章包。它们可能包含自己的工厂、服务、存储库、模型等，但最终它们都被组合在一个代码库中。

微服务是将第二种方法稍作进一步的概念，并将这些关注点分离为独立的可运行的代码库。

# 为什么微服务

复杂性——将特性拆分为微服务，可以将代码分割成更小的块。这让人想起了古老的unix格言:“做好一件事”。有一种趋势是，单分子允许域彼此紧密耦合，而关注点变得模糊。这将导致更危险、更复杂的更新、潜在的更多bug和更复杂的集成。

规模——在一个整体中，某些代码领域可能比其他领域更频繁地使用。用一个整体，你只能缩放整个代码库。因此，如果您的auth服务经常受到攻击，您需要扩展整个代码库来处理您的第auth服务的负载。

有了微服务，这种分离就可以让您单独地扩展单个服务。这意味着更有效的水平扩展。它与多核、多区域的云计算非常有效。

Nginx写了一个关于微服务的各种概念的奇妙系列，[请阅读](https://www.nginx.com/blog/introduction-to-microservices/)。

# 为什么Golang

毕竟，微服务是由所有语言支持的，毕竟，微服务是一个概念，而不是一个特定的框架或工具。也就是说，有些语言更适合，或者比其他语言更好地支持微服务。有一种非常支持的语言是Golang。

Golang非常轻，非常快，并且对并发有非常好的支持，在运行多个机器和内核时，它是一个强大的功能。

Go还包含一个非常强大的用于编写web服务的标准库。

最后，还有一个很棒的微服务框架，叫做Go-micro。我们将在本系列中使用。

# 引入protobuf/gRPC

由于微服务被拆分为单独的代码库，而微服务的一个重要问题就是通信。在一个单一的通信中不是一个问题，因为你直接从代码库的其他地方调用代码。然而，微服务没有这种能力，因为它们生活在不同的地方。因此，你需要一种方式，让这些独立的服务能够以尽可能少的延迟进行对话。

在这里，您可以使用传统的REST，例如http上的JSON或XML。但是，这种方法的问题在于，服务A必须将其数据编码成JSON/XML，将一个大字符串发送到网络上，然后服务于B，然后将该消息从JSON解码为代码。这在规模上有潜在的开销问题。当您被迫采用这种形式的web浏览器通信时，服务可以以任何他们希望的格式进行对话。

[gRPC](https://grpc.io/)是由谷歌提出的基于轻量级二进制文件的RPC通信协议。这是很多单词，我们来仔细分析一下。gRPC使用二进制作为其核心数据格式。在我们的RESTful示例中，使用JSON，您将通过http发送一个字符串。字符串包含关于其编码格式的大量元数据;关于它的长度，它的内容格式和各种其他零零碎碎。这是为了让服务器能够通知传统的基于浏览器的客户端。在两个服务之间进行通信时，我们并不需要所有这些。所以我们可以使用冷硬二进制，它的重量要轻得多。gRPC使用了新的HTTP 2.0规范，它允许使用二进制数据。它甚至允许双向流，这很酷!HTTP 2对于gRPC的工作方式非常重要。有关HTTP 2的更多信息，请查看[谷歌的这篇精彩文章](https://developers.google.com/web/fundamentals/performance/http2/)。

但是对于二进制数据我们怎么做呢?gRPC有一个叫做protobuf的互换DSL。Protobuf允许您使用开发人员友好的格式定义服务的接口。

让我们从创建第一个服务定义开始。从我们的repo的根目录创建以下文件:**consignment-service/proto/consignment/consignment.proto**。就目前而言，我把我们所有的服务都安排在一个单一的回购协议里。这就是所谓的mono-repo。这主要是为了让本教程保持简单。有很多反对使用单引号的理由，我不会在这里讨论。您可以将所有这些服务和组件放在单独的repos中，也有许多很好的理由支持这种方法。

这里有一篇[很棒的关于gRPC的文章](https://blog.gopheracademy.com/advent-2017/go-grpc-beyond-basics/)，我强烈推荐你读一读。

在您刚刚创建的consignment.proto文件中，添加以下内容:

```proto
// consignment-service/proto/consignment/consignment.proto
syntax = "proto3";

package go.micro.srv.consignment;

service ShippingService {
  rpc CreateConsignment(Consignment) returns (Response) {}
}

message Consignment {
  string id = 1;
  string description = 2;
  int32 weight = 3;
  repeated Container containers = 4;
  string vessel_id = 5;
}

message Container {
  string id = 1;
  string customer_id = 2;
  string origin = 3;
  string user_id = 4;
}

message Response {
  bool created = 1;
  Consignment consignment = 2;
}
```

这是一个非常基本的例子，但是这里有一些事情。首先，定义您的服务，这应该包含您希望向其他服务公开的方法。然后定义消息类型，这些是有效的数据结构。Protobuf是静态类型的，您可以定义自定义类型，就像我们在容器中所做的那样。消息本身就是自定义类型。

这里有两个库，消息由protobuf处理，我们定义的服务由gRPC protobuf插件处理，它编译代码来与这些类型交互。我们的proto文件的服务部分。

然后，通过一个CLI来生成这个protobuf定义，生成代码来接口这个二进制数据和您的功能。

说到这里，让我们为我们的第一个服务`$ touch consignment-service/Makefile`创建一个Makefile。

注意:在复制Makefile代码时要注意格式，它们必须是制表符间距，否则就会中断。请确保您的编辑器已经对makefile进行了linting或适当的设置。

```makefile
build:
    protoc -I. --go_out=plugins=grpc:$(GOPATH)/src/github.com/ewanvalentine/shipper/consignment-service \
      proto/consignment/consignment.proto
```

这将调用protoc库，该库负责将您的protobuf定义编译成代码。我们还指定了grpc插件的使用，以及构建上下文和输出路径。

现在，当您从这个服务目录运行`$ make build`时，查看`proto/consignment/`您应该看到一个新的Go文件，叫做consignment.pb.go。这是gRPC/protobuf库自动生成的代码，允许您将protobuf定义与您自己的代码进行接口。

我们现在来创建main.go文件在项目根`$ touch consignment-service/main.go`。

```go
// consignment-service/main.go
package main

import (
	"log"
	"net"

        // Import the generated protobuf code
	pb "github.com/ewanvalentine/shipper/consignment-service/proto/consignment"
	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
)

const (
	port = ":50051"
)

type IRepository interface {
	Create(*pb.Consignment) (*pb.Consignment, error)
}

// Repository - Dummy repository, this simulates the use of a datastore
// of some kind. We'll replace this with a real implementation later on.
type Repository struct {
	consignments []*pb.Consignment
}

func (repo *Repository) Create(consignment *pb.Consignment) (*pb.Consignment, error) {
	updated := append(repo.consignments, consignment)
	repo.consignments = updated
	return consignment, nil
}

// Service should implement all of the methods to satisfy the service
// we defined in our protobuf definition. You can check the interface
// in the generated code itself for the exact method signatures etc
// to give you a better idea.
type service struct {
	repo IRepository
}

// CreateConsignment - we created just one method on our service,
// which is a create method, which takes a context and a request as an
// argument, these are handled by the gRPC server.
func (s *service) CreateConsignment(ctx context.Context, req *pb.Consignment) (*pb.Response, error) {

        // Save our consignment
	consignment, err := s.repo.Create(req)
	if err != nil {
		return nil, err
	}

        // Return matching the `Response` message we created in our
        // protobuf definition.
	return &pb.Response{Created: true, Consignment: consignment}, nil
}

func main() {

	repo := &Repository{}

        // Set-up our gRPC server.
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()

        // Register our service with the gRPC server, this will tie our
        // implementation into the auto-generated interface code for our
        // protobuf definition.
	pb.RegisterShippingServiceServer(s, &service{repo})

	// Register reflection service on gRPC server.
	reflection.Register(s)
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

请仔细阅读代码中的注释。但总的来说，我们在这里创建了实现逻辑，我们的gRPC方法接口使用生成的格式，在端口50051上创建一个新的gRPC服务器。有你有它!一个功能齐全的gRPC服务。你可以用$ go main来运行这个。去吧，但是你什么都看不到，你还不能用它……因此，让我们创建一个客户端来查看它的实际操作。

让我们创建一个命令行接口，它将使用一个JSON委托文件，并与我们的gRPC服务交互。

在根目录中，创建一个新的子目录`$ mkdir consignment-cli`。在该目录中，创建一个名为`cli.go`的文件。以下内容:

```go
// consignment-cli/cli.go
package main

import (
	"encoding/json"
	"io/ioutil"
	"log"
	"os"

	pb "github.com/ewanvalentine/shipper/consignment-service/proto/consignment"
	"golang.org/x/net/context"
	"google.golang.org/grpc"
)

const (
	address         = "localhost:50051"
	defaultFilename = "consignment.json"
)

func parseFile(file string) (*pb.Consignment, error) {
	var consignment *pb.Consignment
	data, err := ioutil.ReadFile(file)
	if err != nil {
		return nil, err
	}
	json.Unmarshal(data, &consignment)
	return consignment, err
}

func main() {
	// Set up a connection to the server.
	conn, err := grpc.Dial(address, grpc.WithInsecure())
	if err != nil {
		log.Fatalf("Did not connect: %v", err)
	}
	defer conn.Close()
	client := pb.NewShippingService(conn)

	// Contact the server and print out its response.
	file := defaultFilename
	if len(os.Args) > 1 {
		file = os.Args[1]
	}

	consignment, err := parseFile(file)

	if err != nil {
		log.Fatalf("Could not parse file: %v", err)
	}

	r, err := client.CreateConsignment(context.Background(), consignment)
	if err != nil {
		log.Fatalf("Could not greet: %v", err)
	}
	log.Printf("Created: %t", r.Created)
}
```

现在创建一个委托(consignment-cli/consignment.json):

```json
{
  "description": "This is a test consignment",
  "weight": 550,
  "containers": [
    { "customer_id": "cust001", "user_id": "user001", "origin": "Manchester, United Kingdom" }
  ],
  "vessel_id": "vessel001"
}
```

现在，如果你在consignment-service运行`$ go run main.go`，然后在一个单独的终端面板中运行`$ go run cli.go`。您应该看到一条消息说Created: true。但是，我们如何才能真正检验它创造了什么呢?让我们用GetConsignments方法来更新我们的服务，这样我们就可以查看所有创建的委托。

首先让我们更新我们的proto定义(我留下注释来表示所做的更改):

```go
// consignment-service/proto/consignment/consignment.proto
syntax = "proto3";

package go.micro.srv.consignment;

service ShippingService {
  rpc CreateConsignment(Consignment) returns (Response) {}

  // Created a new method
  rpc GetConsignments(GetRequest) returns (Response) {}
}

message Consignment {
  string id = 1;
  string description = 2;
  int32 weight = 3;
  repeated Container containers = 4;
  string vessel_id = 5;
}

message Container {
  string id = 1;
  string customer_id = 2;
  string origin = 3;
  string user_id = 4;
}

// Created a blank get request
message GetRequest {}

message Response {
  bool created = 1;
  Consignment consignment = 2;

  // Added a pluralised consignment to our generic response message
  repeated Consignment consignments = 3;
}
```

因此，我们在我们的服务上创建了一个名为GetConsignments的新方法，我们还创建了一个新的GetRequest，它暂时不包含任何东西。我们还在响应消息中添加了一个consignments字段。您将注意到这里的类型在实际类型之前重复了关键字。正如您可能已经猜到的那样，这意味着将这个字段作为这些类型的数组来处理。

现在再次运行`$ make build`。现在，再次尝试运行您的服务，您应该会看到类似于:*. service没有实现go_micro_srv_consignment.ShippingServiceServer(丢失GetConsignments方法)的错误。

由于我们的gRPC方法的实现，是基于与protobuf库生成的接口相匹配的，所以我们需要确保我们的实现符合我们的proto定义。

所以，让我们更新一下我们的`consignment-service/main.go`文件:

```go
package main

import (
	"log"
	"net"

	// Import the generated protobuf code
	pb "github.com/ewanvalentine/shipper/consignment-service/proto/consignment"
	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
)

const (
	port = ":50051"
)

type IRepository interface {
	Create(*pb.Consignment) (*pb.Consignment, error)
	GetAll() []*pb.Consignment
}

// Repository - Dummy repository, this simulates the use of a datastore
// of some kind. We'll replace this with a real implementation later on.
type Repository struct {
	consignments []*pb.Consignment
}

func (repo *Repository) Create(consignment *pb.Consignment) (*pb.Consignment, error) {
	updated := append(repo.consignments, consignment)
	repo.consignments = updated
	return consignment, nil
}

func (repo *Repository) GetAll() []*pb.Consignment {
	return repo.consignments
}

// Service should implement all of the methods to satisfy the service
// we defined in our protobuf definition. You can check the interface
// in the generated code itself for the exact method signatures etc
// to give you a better idea.
type service struct {
	repo IRepository
}

// CreateConsignment - we created just one method on our service,
// which is a create method, which takes a context and a request as an
// argument, these are handled by the gRPC server.
func (s *service) CreateConsignment(ctx context.Context, req *pb.Consignment) (*pb.Response, error) {

	// Save our consignment
	consignment, err := s.repo.Create(req)
	if err != nil {
		return nil, err
	}

	// Return matching the `Response` message we created in our
	// protobuf definition.
	return &pb.Response{Created: true, Consignment: consignment}, nil
}

func (s *service) GetConsignments(ctx context.Context, req *pb.GetRequest) (*pb.Response, error) {
	consignments := s.repo.GetAll()
	return &pb.Response{Consignments: consignments}, nil
}

func main() {

	repo := &Repository{}

	// Set-up our gRPC server.
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()

	// Register our service with the gRPC server, this will tie our
	// implementation into the auto-generated interface code for our
	// protobuf definition.
	pb.RegisterShippingServiceServer(s, &service{repo})

	// Register reflection service on gRPC server.
	reflection.Register(s)
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

这里，我们已经包含了新的GetConsignments方法，更新了我们的存储库和接口，并满足了proto定义生成的接口。如果你运行$ go run main.go。再来一遍，这又管用了。

让我们更新cli工具，以包括调用此方法和列出我们的委托的能力:

```go
func main() {
    ...

	getAll, err := client.GetConsignments(context.Background(), &pb.GetRequest{})
	if err != nil {
		log.Fatalf("Could not list consignments: %v", err)
	}
	for _, v := range getAll.Consignments {
		log.Println(v)
	}
}
```

在我们的main函数的最下面，在下面我们记录我们的"Created: success"消息，附加上面的代码，并重新运行 $ go run cli.go。这将创建一个委托，然后调用GetConsignments。您应该看到这个列表在运行的次数越多。

注意:为了简洁起见，我有时可能会修订之前用一个…表示没有对前面的代码做任何更改，但是添加了附加的行或附加的行。

因此，我们已经成功地创建了一个微服务和一个客户端来与它交互，使用protobuf和gRPC。

本系列的下一篇文章将围绕如何集成go-micro，这是一个强大的框架，用于创建基于gRPC的微服务。我们还将创建第二个服务，即容器服务。说到容器，只是为了混淆问题，我们还将在本系列的下一部分中查看Docker容器中的服务。

# 进一步的阅读

- 文章和通讯
 https://www.nginx.com/blog/introduction-to-microservices/ 
 https://martinfowler.com/articles/microservices.html 
 https://www.microservices.com/talks/ https://medium.facilelogin.com/ten-talks-on-microservices-you-cannot-miss-at-any-cost-7bbe5ab7f43f#.ui0748oat https://microserviceweekly.com/
- 书
 https://www.amazon.co.uk/Building-Microservices-Sam-Newman/dp/1491950358 https://www.amazon.co.uk/Devops-Handbook-World-Class-Reliability-Organizations/dp/1942788002 https://www.amazon.co.uk/Phoenix-Project-DevOps-Helping-Business/dp/0988262509
- 播客
 https://softwareengineeringdaily.com/tag/microservices/ 
 https://martinfowler.com/tags/podcast.html 
 https://www.infoq.com/microservices/podcasts/