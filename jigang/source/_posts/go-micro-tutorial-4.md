---
title: Go微服务教程-第四部分
date: 2018-05-10 21:56:32
tags:
 - golang
 - go-micro
categories:
 - Go微服务

---

![micro-diag](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526284371762&di=4d74aecb4c8b43534361e1c232276d81&imgtype=0&src=http%3A%2F%2Fi0.wp.com%2Fwww.virtuaniz.com%2Fwp-content%2Fuploads%2F2014%2F08%2Fcloud-computing-advantages-benifits-virtuaniz.png%3Fresize%3D660%2C330)

在本系列的前一部分中，我们研究了创建用户服务并开始存储一些用户。现在我们需要考虑使我们的用户服务存储用户密码安全，并创建一些功能来验证用户，并在我们的微服务中发布安全令牌。

<!-- more -->

注意，我现在已经将我们的服务分离到单独的存储库中。我发现这更容易部署。最初，我打算尝试做一个monorepo，但是我发现用Go的dep管理来设置它太麻烦了，不会有各种冲突。我还将开始演示如何独立运行和测试微服务。

不幸的是，使用这种方法，我们将会失去docker。但现在还好。

现在需要手动运行数据库:

```bash
$ docker run -d -p 5432:5432 postgres
$ docker run -d -p 27017:27017 mongo
```

新的存储库可以在这里找到:

- https://github.com/EwanValentine/shippy-consignment-service
- https://github.com/EwanValentine/shippy-user-service
- https://github.com/EwanValentine/shippy-vessel-service
- https://github.com/EwanValentine/shippy-user-cli
- https://github.com/EwanValentine/shippy-consignment-cli

首先，让我们更新我们的用户处理程序来哈希我们的密码，这是绝对必须的。永远不要存储明文密码。你们中的很多人会认为“duh很明显”，但不幸的是，它还在继续!

```go
// shippy-user-service/handler.go
...
func (srv *service) Auth(ctx context.Context, req *pb.User, res *pb.Token) error {
	log.Println("Logging in with:", req.Email, req.Password)
	user, err := srv.repo.GetByEmail(req.Email)
	log.Println(user)
	if err != nil {
		return err
	}

	// Compares our given password against the hashed password
	// stored in the database
	if err := bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(req.Password)); err != nil {
		return err
	}

	token, err := srv.tokenService.Encode(user)
	if err != nil {
		return err
	}
	res.Token = token
	return nil
}

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
	return nil
}
```

这里并没有发生很大的变化，只是我们添加了密码散列功能，然后在保存新用户之前将其设置为密码。此外，在身份验证方面，我们还检查了哈希密码。

现在我们可以安全地对数据库进行身份验证，我们需要一种机制，在这个机制中，我们可以跨用户界面和分布式服务来实现这一点。有很多方法可以做到这一点，但是我遇到的最简单的解决方案是[JWT](https://jwt.io/)，我们可以在服务和web上使用它。

但是在我们打开之前，请检查一下我对Dockerfiles和每个服务的makefile所做的更改。我还更新了导入来匹配新的git存储库。

# JWT

JWT代表JSON web令牌，是一种分布式安全协议。OAuth相似。这个概念很简单，您可以使用一个算法为用户生成一个唯一的散列，可以对其进行比较和验证。但不仅如此，令牌本身可以包含并由我们的用户元数据组成。换句话说，它们的数据本身可以成为令牌的一部分。让我们看一个JWT的例子:

```jwt
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

这个令牌被分成三部分。每一段都有重要意义。第一部分是关于令牌本身的一些元数据。例如令牌的类型和用于创建令牌的算法。这使客户能够理解如何解码令牌。第二部分由用户定义的元数据组成。这可以是您的用户详细信息，一个过期时间，任何您想要的。最后一个部分是验证签名，它是关于如何散列标记和使用什么数据的信息。

当然，使用JWT也有缺点和风险，[本文](http://cryto.net/%7Ejoepie91/blog/2016/06/13/stop-using-jwt-for-sessions/)概述了这些优点。另外，我建议您阅读本文以了解安全最佳实践。

我建议您特别关注的是，获取用户的IP，并将其作为标记声明的一部分。这确保了某人不能窃取您的令牌，并在另一个设备上充当您的角色。确保您使用https有助于减轻这种攻击类型，因为它模糊了您在中间风格攻击中的标记。

有许多不同的散列算法，可以使用到散列JWT，通常分为两类。对称和非对称。对称就像我们使用的方法，使用共享的盐。不对称利用客户端和服务器之间的公钥和私钥。这对于跨服务的身份验证非常有用。

更多资源:

- [Auth0](https://auth0.com/blog/json-web-token-signing-algorithms-overview/)
- [RFC算法](https://tools.ietf.org/html/rfc7518#section-3)

现在我们已经讨论了JWT是什么，让我们更新token_service。执行这些操作。我们将会使用一个非常棒的Go库:github.com/dgrijalva/jwt-go，其中包含一些很好的例子。

```go
// shippy-user-service/token_service.go
package main

import (
	"time"

	pb "github.com/EwanValentine/shippy-user-service/proto/user"
	"github.com/dgrijalva/jwt-go"
)

var (

	// Define a secure key string used
	// as a salt when hashing our tokens.
	// Please make your own way more secure than this,
	// use a randomly generated md5 hash or something.
	key = []byte("mySuperSecretKeyLol")
)

// CustomClaims is our custom metadata, which will be hashed
// and sent as the second segment in our JWT
type CustomClaims struct {
	User *pb.User
	jwt.StandardClaims
}

type Authable interface {
	Decode(token string) (*CustomClaims, error)
	Encode(user *pb.User) (string, error)
}

type TokenService struct {
	repo Repository
}

// Decode a token string into a token object
func (srv *TokenService) Decode(tokenString string) (*CustomClaims, error) {

	// Parse the token
	token, err := jwt.ParseWithClaims(tokenString, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
		return key, nil
	})

	// Validate the token and return the custom claims
	if claims, ok := token.Claims.(*CustomClaims); ok && token.Valid {
		return claims, nil
	} else {
		return nil, err
	}
}

// Encode a claim into a JWT
func (srv *TokenService) Encode(user *pb.User) (string, error) {

	expireToken := time.Now().Add(time.Hour * 72).Unix()

	// Create the Claims
	claims := CustomClaims{
		user,
		jwt.StandardClaims{
			ExpiresAt: expireToken,
			Issuer:    "go.micro.srv.user",
		},
	}

	// Create token
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

	// Sign token and return
	return token.SignedString(key)
}
```

根据我的观点，我留下了一些注释，解释了一些更详细的细节，但是这里的前提非常简单。Decode接受一个字符串令牌，将其解析为一个令牌对象，并验证它，并在有效的情况下返回断言。这将允许我们从索赔中获取用户元数据以验证该用户。

编码方法做相反的事情，它将您的自定义元数据散入一个新的JWT并返回它。

注意，我们还在顶部设置了一个“关键”变量，这是一个安全的盐，请使用比这个更安全的产品。

现在我们有了一个验证令牌服务。让我们更新我们的用户cli，我已经简化了这只是一个脚本，因为我在之前的cli代码中有问题，我会回到这个，但是这个工具只是用于测试:

```go
// shippy-user-cli/cli.go
package main

import (
	"log"
	"os"

	pb "github.com/EwanValentine/shippy-user-service/proto/user"
	micro "github.com/micro/go-micro"
	microclient "github.com/micro/go-micro/client"
	"golang.org/x/net/context"
)

func main() {

	srv := micro.NewService(

		micro.Name("go.micro.srv.user-cli"),
		micro.Version("latest"),
	)

	// Init will parse the command line flags.
	srv.Init()

	client := pb.NewUserService("go.micro.srv.user", microclient.DefaultClient)

	name := "Ewan Valentine"
	email := "ewan.valentine89@gmail.com"
	password := "test123"
	company := "BBC"

	r, err := client.Create(context.TODO(), &pb.User{
		Name:     name,
		Email:    email,
		Password: password,
		Company:  company,
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

	authResponse, err := client.Auth(context.TODO(), &pb.User{
		Email:    email,
		Password: password,
	})

	if err != nil {
		log.Fatalf("Could not authenticate user: %s error: %v\n", email, err)
	}

	log.Printf("Your access token is: %s \n", authResponse.Token)

	// let's just exit because
	os.Exit(0)
}
```

我们现在只需要一些硬编码的值，替换那些并使用$ make build && make run运行脚本。您应该看到返回一个令牌。复制并粘贴这个长标记字符串，您很快就需要它!

现在我们需要更新我们的consign-cli以获取一个令牌字符串，并将其传递到我们的consignment-service:

```go
// shippy-consignment-cli/cli.go
...
func main() {

	cmd.Init()

	// Create new greeter client
	client := pb.NewShippingService("go.micro.srv.consignment", microclient.DefaultClient)

	// Contact the server and print out its response.
	file := defaultFilename
	var token string
	log.Println(os.Args)

	if len(os.Args) < 3 {
		log.Fatal(errors.New("Not enough arguments, expecing file and token."))
	}

	file = os.Args[1]
	token = os.Args[2]

	consignment, err := parseFile(file)

	if err != nil {
		log.Fatalf("Could not parse file: %v", err)
	}

	// Create a new context which contains our given token.
	// This same context will be passed into both the calls we make
	// to our consignment-service.
	ctx := metadata.NewContext(context.Background(), map[string]string{
		"token": token,
	})

	// First call using our tokenised context
	r, err := client.CreateConsignment(ctx, consignment)
	if err != nil {
		log.Fatalf("Could not create: %v", err)
	}
	log.Printf("Created: %t", r.Created)

	// Second call
	getAll, err := client.GetConsignments(ctx, &pb.GetRequest{})
	if err != nil {
		log.Fatalf("Could not list consignments: %v", err)
	}
	for _, v := range getAll.Consignments {
		log.Println(v)
	}
}
```

现在我们需要更新我们的委托服务来检查令牌的请求，并将其传递给我们的用户服务:

```go
// shippy-consignment-service/main.go
func main() {
    ...
    // Create a new service. Optionally include some options here.
	srv := micro.NewService(

		// This name must match the package name given in your protobuf definition
		micro.Name("go.micro.srv.consignment"),
		micro.Version("latest"),
        // Our auth middleware
		micro.WrapHandler(AuthWrapper),
	)
    ...
}

...

// AuthWrapper is a high-order function which takes a HandlerFunc
// and returns a function, which takes a context, request and response interface.
// The token is extracted from the context set in our consignment-cli, that
// token is then sent over to the user service to be validated.
// If valid, the call is passed along to the handler. If not,
// an error is returned.
func AuthWrapper(fn server.HandlerFunc) server.HandlerFunc {
	return func(ctx context.Context, req server.Request, resp interface{}) error {
		meta, ok := metadata.FromContext(ctx)
		if !ok {
			return errors.New("no auth meta-data found in request")
		}

		// Note this is now uppercase (not entirely sure why this is...)
		token := meta["Token"]
		log.Println("Authenticating with token: ", token)

		// Auth here
		authClient := userService.NewUserService("go.micro.srv.user", client.DefaultClient)
		_, err := authClient.ValidateToken(context.Background(), &userService.Token{
			Token: token,
		})
		if err != nil {
			return err
		}
		err = fn(ctx, req, resp)
		return err
	}
}
```

现在，让我们运行我们的consign-cli工具，cd进入我们新的shippy-consignment-cli repo并运行$ make build来构建我们的新docker映像，现在运行:

```bash
$ make build
$ docker run --net="host" \
      -e MICRO_REGISTRY=mdns \
      consignment-cli consignment.json \
      <TOKEN_HERE>
```

注意，在运行docker容器时，我们使用的是-net="host"标志。这告诉Docker在我们的主机网络上运行我们的容器。e 127.0.0.1或localhost，而不是内部Docker网络。注意，您不需要使用此方法进行任何端口转发。而不是-p 8080:8080你可以只做- p8080。阅读更多关于Docker网络。

现在，当您运行这个程序时，您应该看到已经创建了一个新的委托。尝试从令牌中删除几个字符，这样它就无效了。您应该会看到一个错误。

因此，我们已经创建了JWT令牌服务和一个用于验证JWT令牌的中间件来验证用户。

如果你不想使用go-micro，而你使用的是vanilla grpc，你会希望你的中间件看起来像:

```go
func main() {
    ...
    myServer := grpc.NewServer(
        grpc.UnaryInterceptor(grpc_middleware.ChainUnaryServer(AuthInterceptor),
    )
    ...
}

func AuthInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {

    // Set up a connection to the server.
    conn, err := grpc.Dial(authAddress, grpc.WithInsecure())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := pb.NewAuthClient(conn)
    r, err := c.ValidateToken(ctx, &pb.ValidateToken{Token: token})

    if err != nil {
	    log.Fatalf("could not authenticate: %v", err)
    }

    return handler(ctx, req)
}
```

这种设置在本地运行有点笨拙。但我们并不总是需要在本地运行每个服务。我们应该能够创建独立的、可以独立测试的服务。在我们的例子中，如果我们想测试我们的委托服务，我们可能并不一定要运行我们的authservice。我使用的一个技巧是切换到其他服务的调用。

我已经更新了我们的寄售服务auth包装:

```go
// shippy-user-service/main.go
...
func AuthWrapper(fn server.HandlerFunc) server.HandlerFunc {
	return func(ctx context.Context, req server.Request, resp interface{}) error {
        // This skips our auth check if DISABLE_AUTH is set to true
		if os.Getenv("DISABLE_AUTH") == "true" {
			return fn(ctx, req, resp)
		}
		...
	}
}
```

然后在Makefile中添加新的toggle:

```makefile
// shippy-user-service/Makefile
...
run:
	docker run -d --net="host" \
		-p 50052 \
		-e MICRO_SERVER_ADDRESS=:50052 \
		-e MICRO_REGISTRY=mdns \
		-e DISABLE_AUTH=true \
		consignment-service
```

这种方法使得在本地运行您的微服务的某些子部分变得更容易，对于这个问题有几种不同的方法，但我发现这是最简单的方法。我希望你已经发现这个有用，尽管方向有微小的改变。而且，任何关于运行微服务的建议都是非常受欢迎的，因为它将使这个系列变得更容易!
