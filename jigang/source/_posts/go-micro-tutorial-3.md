---
title: Go微服务教程-第三部分
date: 2018-05-10 21:56:27
tags:
 - golang
 - go-micro
categories:
 - Go微服务

---

![micro-diag](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526284371762&di=4d74aecb4c8b43534361e1c232276d81&imgtype=0&src=http%3A%2F%2Fi0.wp.com%2Fwww.virtuaniz.com%2Fwp-content%2Fuploads%2F2014%2F08%2Fcloud-computing-advantages-benifits-virtuaniz.png%3Fresize%3D660%2C330)

在之前的文章中，我们介绍了[go-micro](https://github.com/micro/go-micro)和[Docker](https://www.docker.com/)的一些基础知识。我们还介绍了第二种服务。在这篇文章中，我们将讨论[docker-compose](https://docs.docker.com/compose/)，以及如何在本地更轻松地运行我们的服务。我们将介绍一些不同的数据库，最后我们将介绍第三种服务。

<!-- more -->

# 先决条件

安装docker-compose:[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

但是首先，让我们看看数据库。

# 选择数据库

到目前为止，我们的数据实际上并没有存储在任何地方，而是存储在我们的服务中，当我们的容器重新启动时，它就会丢失。当然，我们需要一种持久化、存储和查询数据的方法。

微服务的好处在于，每个服务可以使用不同的数据库。当然你不需要这么做，很多人都不需要。事实上，我很少为小团队做这样的事情，因为维护几个不同的数据库是一种精神上的飞跃，而不仅仅是一个。但是在某些情况下，一个服务数据可能不适合您为其他服务使用的数据库。所以用别的东西是有意义的。由于您的关注是完全独立的，微服务使这个简单的简单。

为您的服务选择“正确”的数据库是一篇完全不同的文章，[例如这篇文章](https://www.infoworld.com/article/3236291/database/how-to-choose-a-database-for-your-microservices.html)，所以我们不会对这个subect做太多的细节。但是，如果您有相当松散或不一致的数据集，那么NoSQL文档存储解决方案是完美的。它们更灵活，可以存储和使用json。我们将为NoSQL数据库使用MongoDB。没有什么特别的原因，除了它表现良好，它被广泛使用和支持，并且有一个伟大的在线社区。

如果您的数据更严格地定义和关系，那么使用传统的rdbms或关系数据库是有意义的。但实际上并没有硬性规定，一般情况下，任何人都可以胜任这份工作。但是一定要查看您的数据结构，考虑您的服务是否正在进行更多的阅读或更多的写作，查询的复杂程度如何，并尝试将这些作为选择数据库的起点。对于我们的关系数据库，我们将使用Postgres。再一次，没有什么特别的原因，除了它做的很好，我很熟悉它。你可以用MySQL, MariaDB，或者别的什么。

Amazon和谷歌对于这两种数据库类型的前提解决方案都有一些出色的解决方案，如果您想避免管理自己的数据库(通常是明智的)。另一个不错的选择是编写，它将使用相同的云提供程序作为您的服务，以避免连接延迟，从而实现各种数据库技术的完全管理、可伸缩的实例。

亚马逊:RDBMS: [https://aws.amazon.com/rds/](https://aws.amazon.com/rds/) NoSQL: [https://aws.amazon.com/dynamodb/](https://aws.amazon.com/dynamodb/)。

RDBMS: [https://cloud.google.com/spanner/](https://cloud.google.com/spanner/) NoSQL: https://cloud.google.com/datastore/。

现在我们已经讨论了一些数据库，让我们来做一些编码!

# docker-compose

在本系列的最后一部分中，我们研究了Docker，它让我们在轻量级容器中运行我们的服务，并使用它们自己的运行时和依赖项。但是，必须使用单独的Makefile来运行和管理每个服务，这有点麻烦。让我们来看看[docker-compose](https://docs.docker.com/compose/)。[Docker-compose ](https://docs.docker.com/compose/)允许您在yaml文件中定义docker容器的列表，并指定关于其运行时的元数据。Docker-compose服务或多或少地映射到我们已经使用的docker命令。例如:

```bash
$ docker run -p 50052:50051 -e MICRO_SERVER_ADDRESS=:50051 -e MICRO_REGISTRY=mdns vessel-service
```

就变成了:

```yaml
version: '3.1'

services:
  vessel-service:
    build: ./vessel-service
    ports:
      - 50052:50051
    environment:
      MICRO_REGISTRY: "mdns"
      MICRO_SERVER_ADDRESS: ":50051"
```

简单!

因此，让我们在我们的目录$ touch docker-compose.yml中创建一个docker- compile文件。现在加入我们的服务:

```yaml
# docker-compose.yml
version: '3.1'

services:

  consignment-cli:
    build: ./consignment-cli
    environment:
      MICRO_REGISTRY: "mdns"

  consignment-service:
    build: ./consignment-service
    ports:
      - 50051:50051
    environment:
      MICRO_ADDRESS: ":50051"
      MICRO_REGISTRY: "mdns"
      DB_HOST: "datastore:27017"

  vessel-service:
    build: ./vessel-service
    ports:
      - 50052:50051
    environment:
      MICRO_ADDRESS: ":50051"
      MICRO_REGISTRY: "mdns"
```

首先，我们定义了我们想要使用的docker-compose的版本，然后是服务列表。还有其他根级定义，如网络和卷，但我们现在只关注服务。

每个服务由它的名称定义，然后我们包括一个构建路径，它是指向一个位置的引用，该位置应该包含一个Dockerfile。这告诉docker-compose 使用这个Dockerfile来构建它的映像。您还可以在这里使用图像来使用预构建映像。我们以后会讲到。然后定义端口映射，最后定义环境变量。

要构建docker-compose栈，只需运行`$ docker-compose build`，并运行它，`$ docker-compose run`。要在后台运行栈，请使用`$ docker-compose up -d`。您还可以使用$ docker ps查看当前正在运行的容器的列表。最后，您可以通过运行`$ docker stop $(docker ps -qa)`来停止所有当前的容器。

我们来运行栈。您应该会看到大量的输出和dockerfile正在构建。您可能也会从我们的CLI工具中看到错误，但是不要担心，这很可能是因为它是在我们的其他服务之前运行的。它只是说它还找不到它们。

让我们通过运行CLI工具进行测试。要通过docker-组合运行它，只需运行`$ docker-compose run consignment-cli`，一旦所有其他容器都在运行。您应该看到它像以前一样成功运行。

# 实体和protobufs

在本系列中，我们谈到了protobufs是我们数据模型的中心。我们使用它来定义我们的服务结构和功能。由于protobuf以或多或少的所有正确的数据类型生成结构，我们也可以重用这些结构作为我们的底层数据库模型。这实际上相当令人兴奋。它与原型保持一致，成为真理的唯一来源。

然而，这种方法确实有缺点。有时，将protobuf生成的代码编组到一个有效的数据库实体中是很困难的。有时，数据库技术使用自定义类型，这些类型很难从protobuf生成的原生类型转换。我花了很多时间思考的一个问题是如何将Id字符串转换为Id bson。ObjectId Mongodb的实体。结果是bson。ObjectId，实际上只是一个字符串，所以你可以把它们组合在一起。另外，mongodb的id索引在内部存储为_id，因此您需要一种方法将其绑定到id字符串字段，因为您不能真正执行_id字符串。这意味着要找到一种方法来为您的原buf文件定义自定义标记。但我们以后会讲到。

此外，[许多人经常反对](https://www.reddit.com/r/golang/comments/77yd72/question_do_you_use_protobufs_in_place_of_structs/)使用protobuf定义作为数据库实体，因为您将通信技术与数据库代码紧密耦合。这也是一个有效的点。

一般情况下，建议在protobuf定义代码和数据库实体之间进行转换。但是，您最终会得到许多转换代码，用于转换两个几乎相同的类型，例如:

```go
func (service *Service) (ctx context.Context, req *proto.User, res *proto.Response) error {
  entity := &models.User{
    Name: req.Name.
    Email: req.Email,
    Password: req.Password,
  }
  err := service.repo.Create(entity)
  ...
}
```

表面上看起来并不是那么糟糕，但是当你有几个嵌套结构和几种类型时。它可能非常繁琐，而且可能需要大量的迭代来转换嵌套结构等等。

不过,这种方法真的到你喜欢很多东西在编程,这并不归结为对或错。所以无论感觉最合适的方法。但是,我自己的个人意见是,两个几乎相同的类型之间的转换,尤其是考虑到我们对待protobuf代码作为我们的基础数据,感觉自己像一个减损从我们获得的利益使用protobufs作为你的核心定义。所以我将使用我们的protobuf代码数据库。顺便说一句,我不是说我是对的,我想听到你的意见。

让我们开始我们的第一个服务，我们的寄售服务。我觉得我们应该先整理一下。我们把所有东西都集中到main_go文件中。我知道这些都是微服务，但这并不是混乱的借口!因此，让我们在consignationservice、`handler.go`、`datastore.go`和`repository.go`中创建两个更多的文件。我在服务的根目录中创建这些，而不是创建它们作为新的包和目录。这对于小型的微服务来说是完全足够的。对于开发人员来说，创建这样的结构是一种常见的诱惑:

```bash
main.go
models/
  user.go
handlers/
  auth.go
  user.go
services/
  auth.go
```

这又回到了MVC时代，在Golang中并没有得到真正的建议。当然不是小项目。如果你有一个更大的项目，有多个关注点，你可以按照下面的方式组织它:

```bash
main.go
users/
  services/
    auth.go
  handlers/
    auth.go
    user.go
  users/
    user.go
containers/
  services/
    manage.go
  models/
    container.go
```

在这里，您是按域对代码进行分组，而不是按其所做的任意分组代码。

然而，由于我们正在处理一个微服务，它应该只处理一个单一的问题，我们不需要采取上述任何一种方法。事实上，Go的宗旨是鼓励简单。因此，我们将从简单的开始，并将所有内容都放在服务的根目录中，并使用一些明确定义的文件名。

作为附带说明，我们需要更新Dockerfile的文件，因为我们不需要将新的分离的代码导入到包中，我们需要告诉go编译器将这些新文件拖入其中。因此，更新构建函数如下所示:

```bash
RUN CGO_ENABLED=0 GOOS=linux go build  -o consignment-service -a -installsuffix cgo main.go repository.go handler.go datastore.go
```

这将包括我们将要创建的新文件。

[MongoDB Golang lib](https://github.com/go-mgo/mgo)是这个简单的一个很好的例子，最后，这里有一篇[关于组织Go codebase的好文章](https://rakyll.org/style-packages/)。

让我们首先从main.go中删除所有的存储库代码，并重新使用它来使用mongodb库，mgo。再一次，我试着注释代码来解释每个部分的功能，所以请仔细阅读代码和注释。特别是关于mgo如何处理会话的部分:

```go
// consignment-service/repository.go
package main

import (
	pb "github.com/EwanValentine/shippy/consignment-service/proto/consignment"
	"gopkg.in/mgo.v2"
)

const (
	dbName = "shippy"
	consignmentCollection = "consignments"
)

type Repository interface {
	Create(*pb.Consignment) error
	GetAll() ([]*pb.Consignment, error)
	Close()
}

type ConsignmentRepository struct {
	session *mgo.Session
}

// Create a new consignment
func (repo *ConsignmentRepository) Create(consignment *pb.Consignment) error {
	return repo.collection().Insert(consignment)
}

// GetAll consignments
func (repo *ConsignmentRepository) GetAll() ([]*pb.Consignment, error) {
	var consignments []*pb.Consignment
	// Find normally takes a query, but as we want everything, we can nil this.
	// We then bind our consignments variable by passing it as an argument to .All().
	// That sets consignments to the result of the find query.
	// There's also a `One()` function for single results.
	err := repo.collection().Find(nil).All(&consignments)
	return consignments, err
}

// Close closes the database session after each query has ran.
// Mgo creates a 'master' session on start-up, it's then good practice
// to copy a new session for each request that's made. This means that
// each request has its own database session. This is safer and more efficient,
// as under the hood each session has its own database socket and error handling.
// Using one main database socket means requests having to wait for that session.
// I.e this approach avoids locking and allows for requests to be processed concurrently. Nice!
// But... it does mean we need to ensure each session is closed on completion. Otherwise
// you'll likely build up loads of dud connections and hit a connection limit. Not nice!
func (repo *ConsignmentRepository) Close() {
	repo.session.Close()
}

func (repo *ConsignmentRepository) collection() *mgo.Collection {
	return repo.session.DB(dbName).C(consignmentCollection)
}
```

因此，我们的代码负责与Mongodb数据库进行交互。我们需要创建创建主session/connection.的代码。更新consignment-service/datastore.go。用以下:

```go
// consignment-service/datastore.go
package main

import (
	"gopkg.in/mgo.v2"
)

// CreateSession creates the main session to our mongodb instance
func CreateSession(host string) (*mgo.Session, error) {
	session, err := mgo.Dial(host)
	if err != nil {
		return nil, err
	}

	session.SetMode(mgo.Monotonic, true)

	return session, nil
}
```

就是这样，非常直接。它以一个主机字符串作为参数，将会话返回给我们的数据存储，当然还有一个潜在的错误，因此我们可以在启动时处理它。让我们修改我们的主。将此文件连接到我们的存储库:

```go
// consignment-service/main.go
package main

import (

	// Import the generated protobuf code
	"fmt"
	"log"

	pb "github.com/EwanValentine/shippy/consignment-service/proto/consignment"
	vesselProto "github.com/EwanValentine/shippy/vessel-service/proto/vessel"
	"github.com/micro/go-micro"
	"os"
)

const (
	defaultHost = "localhost:27017"
)

func main() {

	// Database host from the environment variables
	host := os.Getenv("DB_HOST")

	if host == "" {
		host = defaultHost
	}

	session, err := CreateSession(host)

	// Mgo creates a 'master' session, we need to end that session
	// before the main function closes.
	defer session.Close()

	if err != nil {

		// We're wrapping the error returned from our CreateSession
		// here to add some context to the error.
		log.Panicf("Could not connect to datastore with host %s - %v", host, err)
	}

	// Create a new service. Optionally include some options here.
	srv := micro.NewService(

		// This name must match the package name given in your protobuf definition
		micro.Name("go.micro.srv.consignment"),
		micro.Version("latest"),
	)

	vesselClient := vesselProto.NewVesselService("go.micro.srv.vessel", srv.Client())

	// Init will parse the command line flags.
	srv.Init()

	// Register handler
	pb.RegisterShippingServiceHandler(srv.Server(), &service{session, vesselClient})

	// Run the server
	if err := srv.Run(); err != nil {
		fmt.Println(err)
	}
}
```

# 复制和克隆

您可能已经注意到，在使用mgo Mongodb库时。我们创建一个数据库会话，它被传递到我们的处理程序中，但是在每个请求中，我们调用一个方法来克隆该会话并将其传递到存储库代码中。

实际上，除了生成与数据库的第一个连接之外，我们从未接触过“主会话”，我们每次都要对数据存储进行调用时，我们调用了`session.clone()`。正如我在代码注释中简要提到的，但是我认为值得重新迭代一些细节，如果您使用主会话，您将重用相同的套接字。这意味着您的查询可能会被其他查询阻塞，并且必须等待操作在这个套接字上完成。这在支持并发性的语言中是没有意义的。

因此，为了避免阻塞请求，mgo允许您Copy() 或 Clone()一个会话，以便您对每个请求都有一个并发连接。您会注意到，我提到过复制和克隆方法，它们非常相似，但是有一个细微但重要的区别。克隆重新使用与master相同的套接字。这减少了生成一个全新套接字的开销。这是快速写性能的最佳选择。然而，更长的操作，例如更复杂的查询或大数据作业等，可能会导致其他的go例程阻塞，试图使用这个套接字。

一般来说，你最好是像我们这样的克隆。

我们需要做的最后一点整理是将gRPC处理程序代码移到新的`handler.go`文件。让我们这样做。

```go
// consignment-service.go

package main

import (
	"log"
	"golang.org/x/net/context"
	pb "github.com/EwanValentine/shippy/consignment-service/proto/consignment"
	vesselProto "github.com/EwanValentine/shippy/vessel-service/proto/vessel"
)

// Service should implement all of the methods to satisfy the service
// we defined in our protobuf definition. You can check the interface
// in the generated code itself for the exact method signatures etc
// to give you a better idea.
type service struct {
	vesselClient vesselProto.NewVesselService
}

func (s *service) GetRepo() Repository {
    return &ConsignmentRepository{s.session.Clone()}
}

// CreateConsignment - we created just one method on our service,
// which is a create method, which takes a context and a request as an
// argument, these are handled by the gRPC server.
func (s *service) CreateConsignment(ctx context.Context, req *pb.Consignment, res *pb.Response) error {
    repo := s.GetRepo()
    defer repo.Close()
	// Here we call a client instance of our vessel service with our consignment weight,
	// and the amount of containers as the capacity value
	vesselResponse, err := s.vesselClient.FindAvailable(context.Background(), &vesselProto.Specification{
		MaxWeight: req.Weight,
		Capacity: int32(len(req.Containers)),
	})
	log.Printf("Found vessel: %s \n", vesselResponse.Vessel.Name)
	if err != nil {
		return err
	}

	// We set the VesselId as the vessel we got back from our
	// vessel service
	req.VesselId = vesselResponse.Vessel.Id

	// Save our consignment
	err = repo.Create(req)
	if err != nil {
		return err
	}

	// Return matching the `Response` message we created in our
	// protobuf definition.
	res.Created = true
	res.Consignment = req
	return nil
}

func (s *service) GetConsignments(ctx context.Context, req *pb.GetRequest, res *pb.Response) error {
    repo := s.GetRepo()
    defer repo.Close()
	consignments, err := repo.GetAll()
	if err != nil {
		return err
	}
	res.Consignments = consignments
	return nil
}
```

我们已经更新了我们的repo中的一些返回参数，这些参数来自于上一个教程:Old:

```go
type Repository interface {
	Create(*pb.Consignment) (*pb.Consignment, error)
	GetAll() []*pb.Consignment
}
```

New:

```go
type Repository interface {
	Create(*pb.Consignment) error
	GetAll() ([]*pb.Consignment, error)
    Close()
}
```

这只是因为我觉得我们不需要在创建后返回相同的货物。现在，我们从mgo返回一个适当的错误，以获取我们的get查询。否则代码或多或少是相同的。当然，我们将我们的封闭方法添加到接口中。

现在让我们对您的vessel-service做同样的操作。我不打算在这篇文章中演示，你应该在这一点上有一个好的感觉。记住，您可以使用我的存储库作为参考。

不过，我们将为我们的vesselservice添加一个新方法，它将允许我们创建新的容器。与以往一样，让我们从更新我们的protobuf定义开始:

```go
syntax = "proto3";

package vessel;

service VesselService {
  rpc FindAvailable(Specification) returns (Response) {}
  rpc Create(Vessel) returns (Response) {}
}

message Vessel {
  string id = 1;
  int32 capacity = 2;
  int32 max_weight = 3;
  string name = 4;
  bool available = 5;
  string owner_id = 6;
}

message Specification {
  int32 capacity = 1;
  int32 max_weight = 2;
}

message Response {
  Vessel vessel = 1;
  repeated Vessel vessels = 2;
  bool created = 3;
}
```

我们在gRPC服务下创建了一个新创建方法，它接收一个容器并返回我们的一般响应。我们还为响应消息添加了一个新字段，仅仅是一个创建的bool。运行$ make构建以更新此服务。现在我们将在`vessel-service/handler.go`中添加一个新的处理程序和一个新的存储库方法:

```go
// vessel-service/handler.go

func (s *service) GetRepo() Repository {
    return &VesselRepository{s.session.Clone()}
}

func (s *service) Create(ctx context.Context, req *pb.Vessel, res *pb.Response) error {
    repo := s.GetRepo()
    defer repo.Close()
	if err := repo.Create(req); err != nil {
		return err
	}
	res.Vessel = req
	res.Created = true
	return nil
}
```

```go
// vessel-service/repository.go
func (repo *VesselRepository) Create(vessel *pb.Vessel) error {
	return repo.collection().Insert(vessel)
}
```

所以在这之后。我们已经更新了我们的服务以使用Mongodb。在尝试运行此操作之前，我们需要更新docker-compose文件，以包含Mongodb容器:

```yaml
services:
  ...
  datastore:
    image: mongo
    ports:
      - 27017:27017
```

在您的两个服务中更新环境变量，包括:DB_HOST:“datastore:27017”。注意，我们将数据存储称为主机名，而不是本地主机。这是因为docker-组合为我们处理了一些聪明的内部DNS。

所以你应该:

```yaml
version: '3.1'

services:

  consignment-cli:
    build: ./consignment-cli
    environment:
      MICRO_REGISTRY: "mdns"

  consignment-service:
    build: ./consignment-service
    ports:
      - 50051:50051
    environment:
      MICRO_ADDRESS: ":50051"
      MICRO_REGISTRY: "mdns"
      DB_HOST: "datastore:27017"

  vessel-service:
    build: ./vessel-service
    ports:
      - 50052:50051
    environment:
      MICRO_ADDRESS: ":50051"
      MICRO_REGISTRY: "mdns"
      DB_HOST: "datastore:27017"

  datastore:
    image: mongo
    ports:
      - 27017:27017
```

重新构建您的堆栈`$ docker-compose build`并重新运行它`$ docker-compose up`。注意，有时由于Dockers缓存，您可能需要运行一个cacheless构建来获取某些更改。要在docker-compose中执行此操作，只需在运行`$ docker-compose build`时使用-no-cache标志。

# 用户服务

现在，让我们创建第三个服务。我们将从更新docker-compose.yml文件开始。此外，为了将内容混合起来，我们将为我们的用户服务在docker堆栈中添加Postgres:

```yaml
...
  user-service:
    build: ./user-service
    ports:
      - 50053:50051
    environment:
      MICRO_ADDRESS: ":50051"
      MICRO_REGISTRY: "mdns"

  ...
  database:
    image: postgres
    ports:
      - 5432:5432
```

现在在根目录中创建一个用户服务目录。和之前的服务一样。创建下列文件:handler.go, main.go, repository.go, database.go, Dockerfile, Makefile，我们的proto文件的子目录，最后是proto文件本身:proto/user/user.proto。

将以下内容添加到user.proto:

```proto3
syntax = "proto3";

package go.micro.srv.user;

service UserService {
    rpc Create(User) returns (Response) {}
    rpc Get(User) returns (Response) {}
    rpc GetAll(Request) returns (Response) {}
    rpc Auth(User) returns (Token) {}
    rpc ValidateToken(Token) returns (Token) {}
}

message User {
    string id = 1;
    string name = 2;
    string company = 3;
    string email = 4;
    string password = 5;
}

message Request {}

message Response {
    User user = 1;
    repeated User users = 2;
    repeated Error errors = 3;
}

message Token {
    string token = 1;
    bool valid = 2;
    repeated Error errors = 3;
}

message Error {
    int32 code = 1;
    string description = 2;
}
```

现在，确保您已经创建了一个类似于我们以前的服务的Makefile，您应该能够运行$ make构建来生成我们的gRPC代码。根据我们以前的服务，我们已经创建了一些代码来接口我们的gRPC方法。我们只会让其中的一部分在这个系列的这一部分中工作。我们只是希望能够创建和获取一个用户。在本系列的下一部分中，我们将讨论身份验证和JWT。所以我们将会留下任何与现在有关的标记。您的处理程序应该是这样的:

```go
// user-service/handler.go
package main

import (
	"golang.org/x/net/context"
	pb "github.com/EwanValentine/shippy/user-service/proto/user"
)

type service struct {
	repo Repository
	tokenService Authable
}

func (srv *service) Get(ctx context.Context, req *pb.User, res *pb.Response) error {
	user, err := srv.repo.Get(req.Id)
	if err != nil {
		return err
	}
	res.User = user
	return nil
}

func (srv *service) GetAll(ctx context.Context, req *pb.Request, res *pb.Response) error {
	users, err := srv.repo.GetAll()
	if err != nil {
		return err
	}
	res.Users = users
	return nil
}

func (srv *service) Auth(ctx context.Context, req *pb.User, res *pb.Token) error {
	user, err := srv.repo.GetByEmailAndPassword(req)
	if err != nil {
		return err
	}
	res.Token = "testingabc"
	return nil
}

func (srv *service) Create(ctx context.Context, req *pb.User, res *pb.Response) error {
	if err := srv.repo.Create(req); err != nil {
		return err
	}
	res.User = req
	return nil
}

func (srv *service) ValidateToken(ctx context.Context, req *pb.Token, res *pb.Token) error {
	return nil
}
```

现在让我们添加我们的存储库代码:

```go
// user-service/repository.go
package main

import (
	pb "github.com/EwanValentine/shippy/user-service/proto/user"
	"github.com/jinzhu/gorm"
)

type Repository interface {
	GetAll() ([]*pb.User, error)
	Get(id string) (*pb.User, error)
	Create(user *pb.User) error
	GetByEmailAndPassword(user *pb.User) (*pb.User, error)
}

type UserRepository struct {
	db *gorm.DB
}

func (repo *UserRepository) GetAll() ([]*pb.User, error) {
	var users []*pb.User
	if err := repo.db.Find(&users).Error; err != nil {
		return nil, err
	}
	return users, nil
}

func (repo *UserRepository) Get(id string) (*pb.User, error) {
	var user *pb.User
	user.Id = id
	if err := repo.db.First(&user).Error; err != nil {
		return nil, err
	}
	return user, nil
}

func (repo *UserRepository) GetByEmailAndPassword(user *pb.User) (*pb.User, error) {
	if err := repo.db.First(&user).Error; err != nil {
		return nil, err
	}
	return user, nil
}

func (repo *UserRepository) Create(user *pb.User) error {
	if err := repo.db.Create(user).Error; err != nil {
		return err
	}
}
```

我们还需要改变ORM的行为来生成一个UUID，而不是试图生成一个整数ID，如果你不知道，UUID是一个随机生成的连字符字符串，用作ID或主键。这比使用自动递增的ID更安全，因为它阻止人们猜测或遍历API端点。MongoDB已经使用了这种变体，但是我们需要告诉我们的Postgres模型使用UUID。因此，在user-service/proto/user中创建一个名为extensions.go的新文件。在这个文件中，添加:

```go
package go_micro_srv_user

import (
	"github.com/jinzhu/gorm"
	"github.com/satori/go.uuid"
)

func (model *User) BeforeCreate(scope *gorm.Scope) error {
	uuid := uuid.NewV4()
	return scope.SetColumn("Id", uuid.String())
}
```

这个钩子进入GORM的事件生命周期，以便在实体被保存之前为Id列生成UUID。

您会注意到，与我们的Mongodb服务不同，我们没有进行任何连接处理。本机、SQL/postgres驱动程序的工作方式略有不同，所以这次我们不需要担心这个问题。我们正在使用一个名为“gorm”的软件包，让我们简单地讨论一下这个问题。

# Gorm - Go + ORM

[Gorm](http://jinzhu.me/gorm/)是一个合理的轻量级对象关系映射器，它与Postgres、MySQL、Sqlite等很好地工作，它很容易设置、使用和管理数据库模式自动更改。

也就是说，使用微服务，您的数据结构要小得多，包含更少的连接和整体的复杂性。所以，不要觉得你应该使用任何类型的ORM。

我们需要能够测试创建用户，所以让我们创建另一个cli工具。这一次用户cli在我们的项目根中。类似于我们的consignment-cli，但这次:

```go
package main

import (
	"log"
	"os"

	pb "github.com/EwanValentine/shippy/user-service/proto/user"
	microclient "github.com/micro/go-micro/client"
	"github.com/micro/go-micro/cmd"
	"golang.org/x/net/context"
	"github.com/micro/cli"
	"github.com/micro/go-micro"
)


func main() {

	cmd.Init()

	// Create new greeter client
	client := pb.NewUserService("go.micro.srv.user", microclient.DefaultClient)

    // Define our flags
	service := micro.NewService(
		micro.Flags(
			cli.StringFlag{
				Name:  "name",
				Usage: "You full name",
			},
			cli.StringFlag{
				Name:  "email",
				Usage: "Your email",
			},
			cli.StringFlag{
				Name:  "password",
				Usage: "Your password",
			},
			cli.StringFlag{
				Name: "company",
				Usage: "Your company",
			},
		),
	)

    // Start as service
	service.Init(

		micro.Action(func(c *cli.Context) {

			name := c.String("name")
			email := c.String("email")
			password := c.String("password")
			company := c.String("company")

            // Call our user service
			r, err := client.Create(context.TODO(), &pb.User{
				Name: name,
				Email: email,
				Password: password,
				Company: company,
			})
			if err != nil {
				log.Fatalf("Could not create: %v", err)
			}
			log.Printf("Created: %s", r.User.Id)

			getAll, err := client.GetAll(context.Background(), &pb.Request{})
			if err != nil {
				log.Fatalf("Could not list users: %v", err)
			}
			for _, v := range getAll.Users {
				log.Println(v)
			}

			os.Exit(0)
		}),
	)

	// Run the server
	if err := service.Run(); err != nil {
		log.Println(err)
	}
}
```

这里我们使用了go-micro的命令行帮助器，非常简洁。

我们可以运行这个并创建一个用户:

```bash
$ docker-compose run user-cli command \
  --name="Ewan Valentine" \
  --email="ewan.valentine89@gmail.com" \
  --password="Testing123" \
  --company="BBC"
```

您应该在列表中看到创建的用户!

这不是很安全，因为目前我们存储的是明文密码，但是在本系列的下一部分中，我们将在我们的服务中查看身份验证和JWT标记。

我们已经创建了一个额外的服务，一个额外的命令行工具，我们开始使用两种不同的数据库技术来保存数据。在这篇文章中，我们已经讨论了很多问题，如果我们过于迅速地讨论任何事情，涉及太多或假设太多的知识，我们就会道歉。
