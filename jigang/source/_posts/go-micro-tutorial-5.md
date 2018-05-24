---
title: Go微服务教程-第五部分
date: 2018-05-10 21:56:48
tags:
 - golang
 - go-micro
categories:
 - Go微服务

---

![micro-diag](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526284371762&di=4d74aecb4c8b43534361e1c232276d81&imgtype=0&src=http%3A%2F%2Fi0.wp.com%2Fwww.virtuaniz.com%2Fwp-content%2Fuploads%2F2014%2F08%2Fcloud-computing-advantages-benifits-virtuaniz.png%3Fresize%3D660%2C330)

在本系列的前一部分中，我们讨论了用户身份验证和JWT。在这节课中，我们将快速浏览一下go-micro的代理功能，甚至是代理。

正如前文所提到的，go-micro是一个可插拔的框架，它接口许多不同的常用技术。如果您查看一下plugins repo，您会看到它支持了多少个插件。

在我们的例子中，我们将使用NATS代理插件。

<!-- more -->

# 事件驱动架构

事件驱动的体系结构是一个非常简单的概念。我们通常认为好的体系结构是可以解耦的;这些服务不应该与其他服务相耦合。当我们使用gRPC这样的协议时，这在某些情况下是正确的，因为我们说我想将这个请求发布到go.srv。例如用户服务。使用服务发现来查找该服务的实际位置。虽然这并不能直接将我们与实现结合起来，但它确实将服务连接到另一个名为go.srv的东西上。用户服务，所以它不是完全解耦的，因为它直接与其他东西对话。

那么，是什么使事件驱动架构真正地解耦了呢?为了理解这一点，让我们首先看一下发布和订阅事件的过程。服务a完成了任务x，然后对生态系统“x刚刚发生”表示。它不需要知道，或者关心是谁在听那个事件，或者是什么受到了那个事件的影响。这种责任留给了倾听的客户。

如果你期望在某一事件上有n个服务，这也会更容易。例如，如果您需要12个不同的服务来处理使用gRPC创建的新用户，那么您必须在您的用户服务中实例化12个客户端。使用pubsub或事件驱动架构，您的服务不需要关心这些。

现在，客户端服务将简单地侦听事件。这意味着您需要某种中间的中介来接受这些事件，并将其发布通知客户。

在本文中，我们将在每次创建用户时创建一个事件，我们将创建一个新服务来发送电子邮件。我们实际上不会写电子邮件的实现，只是暂时把它模拟出来。

## 代码

首先，我们需要将[NATS](https://nats.io/)代理插件集成到我们的用户服务中:

```go
// shippy-user-service/main.go
func main() {
    ...
    // Init will parse the command line flags.
	srv.Init()

	// Get instance of the broker using our defaults
	pubsub := srv.Server().Options().Broker

	// Register handler
	pb.RegisterUserServiceHandler(srv.Server(), &service{repo, tokenService, pubsub})
    ...
}
```

现在，让我们在创建新用户时发布事件(请参阅[此处](https://github.com/EwanValentine/shippy-user-service/tree/tutorial-5)的完整更改):

```go
// shippy-user-service/handler.go
const topic = "user.created"

type service struct {
	repo         Repository
	tokenService Authable
	PubSub       broker.Broker
}
...
func (srv *service) Create(ctx context.Context, req *pb.User, res *pb.Response) error {

	// Generates a hashed version of our password
	hashedPass, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
	if err != nil {
		return err
	}
	req.Password = string(hashedPass)
	if err := srv.repo.Create(req); err != nil {
		return err
	}
	res.User = req
	if err := srv.publishEvent(req); err != nil {
		return err
	}
	return nil
}

func (srv *service) publishEvent(user *pb.User) error {
	// Marshal to JSON string
	body, err := json.Marshal(user)
	if err != nil {
		return err
	}

	// Create a broker message
	msg := &broker.Message{
		Header: map[string]string{
			"id": user.Id,
		},
		Body: body,
	}

	// Publish message to broker
	if err := srv.PubSub.Publish(topic, msg); err != nil {
		log.Printf("[pub] failed: %v", err)
	}

	return nil
}
...
```

确保你在运行Postgres，然后让我们运行这个服务:

```bash
$ docker run -d -p 5432:5432 postgres
$ make build
$ make run
```

现在让我们创建我们的电子邮件服务。我创建了一个新的repo:

```go
// shippy-email-service
package main

import (
	"encoding/json"
	"log"

	pb "github.com/EwanValentine/shippy-user-service/proto/user"
	micro "github.com/micro/go-micro"
	"github.com/micro/go-micro/broker"
	_ "github.com/micro/go-plugins/broker/nats"
)

const topic = "user.created"

func main() {
	srv := micro.NewService(
		micro.Name("go.micro.srv.email"),
		micro.Version("latest"),
	)

	srv.Init()

	// Get the broker instance using our environment variables
	pubsub := srv.Server().Options().Broker
	if err := pubsub.Connect(); err != nil {
		log.Fatal(err)
	}

	// Subscribe to messages on the broker
	_, err := pubsub.Subscribe(topic, func(p broker.Publication) error {
		var user *pb.User
		if err := json.Unmarshal(p.Message().Body, &user); err != nil {
			return err
		}
		log.Println(user)
		go sendEmail(user)
		return nil
	})

	if err != nil {
		log.Println(err)
	}

	// Run the server
	if err := srv.Run(); err != nil {
		log.Println(err)
	}
}

func sendEmail(user *pb.User) error {
	log.Println("Sending email to:", user.Name)
	return nil
}
```

在运行这个之前，我们需要运行[NATS…](https://nats.io/)

```bash
$ docker run -d -p 4222:4222 nats
```

另外，我想快速地解释一下我觉得在理解它是如何作为一个框架的过程中，我觉得很重要的一部分。你会注意到:

```go
srv.Init()
pubsub := srv.Server().Options().Broker
```

让我们快速看一看。当我们在go-micro中创建服务时，后台的`srv.Init()`将查找任何配置，如任何插件和环境变量，或命令行选项。然后它将实例化这些集成作为服务的一部分。为了使用这些实例，我们需要将它们从服务实例中取出。在`srv.Server().options()`中，您还将找到传输和注册表。

在这里，它将找到我们的`GO_MICRO_BROKER`环境变量，它将找到NATS代理插件，并创建一个实例。准备好让我们连接和使用。

如果您正在创建一个命令行工具，那么您将使用`cmd.Init()`，确保您导入的是`github.com/micro/go-micro/cmd`。这将产生同样的影响。

现在构建并运行这个服务:`$ make build && make run`，确保您也运行了用户服务。然后转到我们的shippy-user-cli repo，运行`$ make run`，查看我们的电子邮件服务输出。你应该看看`……2017/12/26 23:57:23发邮件至:Ewan Valentine`。

这是它!这是一个有点牵强的例子，因为我们的电子邮件服务隐式地监听单个用户。创建的事件。但是希望您能看到这种方法是如何允许您编写解耦的服务的。

值得一提的是，如果我们回到序列化JSON字符串的领域，在NATS上使用JSON将会有性能开销vs gRPC。但是，对于某些用例来说，这是完全可以接受的。NATS是非常高效的，对火灾和遗忘事件来说是非常棒的。

Go-micro还支持一些最广泛使用的queue/pubsub技术，供您使用。您可以在这里看到[它们的列表](https://github.com/micro/go-plugins/tree/master/broker)。你不需要改变你的实现，因为你可以把它抽象出来。您只需要将环境变量从`MICRO_BROKER=nats`更改为`MICRO_BROKER=googlepubsub`，然后更改main中的导入。例如，从 `_ "github.com/micro/go-plugins/broker/nats"`到`_ "github.com/micro/go-plugins/broker/googlepubsub"`。

如果你不使用go-micro，那就会有一个NATS go库(NATS本身是写在go上的，所以自然地支持go是相当可靠的)。

发布一个事件:

```go
nc, _ := nats.Connect(nats.DefaultURL)

// Simple Publisher
nc.Publish("user.created", userJsonString)
```

订阅一个事件:

```go
// Simple Async Subscriber
nc.Subscribe("user.created", func(m *nats.Msg) {
    user := convertUserString(m.Data)
    go sendEmail(user)
})
```

我之前提到过，当使用第三方message broker(比如NATS)时，您就会失去对protobuf的使用。这是一个遗憾，因为我们失去了使用二进制流进行通信的能力，当然这比序列化的JSON字符串的开销要低得多。但与大多数担忧一样，go-micro也能解决这个问题。

内置到go-micro是一个pubsub层，它位于代理层之上，但不需要第三方代理，如NATS。但是这个特性的可怕之处在于，它使用了你的protobuf定义。我们回到了低延迟的二进制流领域。因此，让我们更新我们的用户服务以替换现有的NATS代理，使用go-micro的pubsub:

```go
// shippy-user-service/main.go
func main() {
    ...
    publisher := micro.NewPublisher("user.created", srv.Client())

	// Register handler
	pb.RegisterUserServiceHandler(srv.Server(), &service{repo, tokenService, publisher})
    ...
}
```

```go
// shippy-user-service/handler.go
func (srv *service) Create(ctx context.Context, req *pb.User, res *pb.Response) error {

	// Generates a hashed version of our password
	hashedPass, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
	if err != nil {
		return err
	}
	req.Password = string(hashedPass)

    // Here's our new publisher code, much simpler
	if err := srv.repo.Create(req); err != nil {
		return err
	}
	res.User = req
	if err := srv.Publisher.Publish(ctx, req); err != nil {
		return err
	}
	return nil
}
```

现在我们的电子邮件服务:

```go
// shippy-email-service
const topic = "user.created"

type Subscriber struct{}

func (sub *Subscriber) Process(ctx context.Context, user *pb.User) error {
	log.Println("Picked up a new message")
	log.Println("Sending email to:", user.Name)
	return nil
}

func main() {
    ...
    micro.RegisterSubscriber(topic, srv.Server(), new(Subscriber))
    ...
}
```

现在，我们在服务中使用底层用户原型，而不是使用gRPC，也不使用第三方代理。太棒了!

这是一个包装!下一个教程，我们将介绍如何为我们的服务创建一个用户界面，并查看web客户端如何开始与我们的服务交互。