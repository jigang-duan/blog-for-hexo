---
title: Go微服务教程-第六部分
date: 2018-05-10 21:56:52
tags:
 - golang
 - go-micro
categories:
 - Go微服务

---

![micro-diag](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526284371762&di=4d74aecb4c8b43534361e1c232276d81&imgtype=0&src=http%3A%2F%2Fi0.wp.com%2Fwww.virtuaniz.com%2Fwp-content%2Fuploads%2F2014%2F08%2Fcloud-computing-advantages-benifits-virtuaniz.png%3Fresize%3D660%2C330)

在上一篇文章中，我们讨论了一些在go-micro和go中事件驱动架构的各种方法。这一部分，我们将深入到客户端，看看如何创建与我们的平台交互的web客户端。

我们将研究微型工具包，使您能够将内部rpc方法从外部代理到web客户端。

我们将为我们的平台创建一个用户界面，一个登录界面，以及一个创建委托界面。这将把以前的一些帖子联系在一起。

所以让我们开始吧!

<!-- more -->

# RPC复兴

REST已经在web上运行多年了，它已经迅速成为管理客户端和服务器之间资源的goto方式。REST的出现，取代了RPC和SOAP实现的狂野西部，这种方式有时会让人觉得过时和痛苦。曾经需要编写wsdl文件吗?

REST向我们承诺了一种实用的、简单的、统一的管理资源的方法。REST使用http谓词在执行的操作类型中更加显式。REST鼓励我们使用http错误代码来更好地描述服务器的响应。在很大程度上，这种方法运行良好，而且很好。但是像所有的好东西一样，休息也有很多抱怨和烦恼，我不打算在这里详细说明。但请务必阅读这篇[文章](https://medium.freecodecamp.org/rest-is-the-new-soap-97ff6c09896d)。但随着微服务的出现，RPC正在卷土重来。

Rest对于管理不同的资源非常有用，但是微服务通常只处理单个资源的本质。因此，我们不需要在微服务上下文中使用RESTful术语。相反，我们可以专注于每个服务的具体操作和交互。

# Micro

在本教程中，我们已经广泛地使用了go-micro，现在我们将讨论microcli /toolkit。microtoolkit提供了一个API网关、一个sidecar、一个web代理以及其他一些很酷的特性。但我们将在这篇文章中看到的主要部分是API网关。

API网关将允许我们将rpc调用代理到web友好的JSON rpc调用，并将公开我们在客户端应用程序中可以使用的url。

那么这是如何运作的呢?你首先要确保安装了微型工具包:

```bash
$ go get -u github.com/micro/micro
```

更好的是，既然我们使用Docker，让我们使用Docker映像:

```bash
$ docker pull microhq/micro
```

现在让我们进入我们的用户服务，我做了一些更改，主要是错误处理和命名约定:

```go
// shippy-user-service/main.go
package main

import (
	"log"

	pb "github.com/EwanValentine/shippy-user-service/proto/auth"
	"github.com/micro/go-micro"
	_ "github.com/micro/go-plugins/registry/mdns"
)

func main() {

	// Creates a database connection and handles
	// closing it again before exit.
	db, err := CreateConnection()
	defer db.Close()

	if err != nil {
		log.Fatalf("Could not connect to DB: %v", err)
	}

	// Automatically migrates the user struct
	// into database columns/types etc. This will
	// check for changes and migrate them each time
	// this service is restarted.
	db.AutoMigrate(&pb.User{})

	repo := &UserRepository{db}

	tokenService := &TokenService{repo}

	// Create a new service. Optionally include some options here.
	srv := micro.NewService(

		// This name must match the package name given in your protobuf definition
		micro.Name("shippy.auth"),
	)

	// Init will parse the command line flags.
	srv.Init()

    // Will comment this out for now to save having to run this locally...
	// publisher := micro.NewPublisher("user.created", srv.Client())

	// Register handler
	pb.RegisterAuthHandler(srv.Server(), &service{repo, tokenService, publisher})

	// Run the server
	if err := srv.Run(); err != nil {
		log.Fatal(err)
	}
}
```

```proto
// shippy-user-service/proto/auth/auth.proto
syntax = "proto3";

package auth;

service Auth {
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

```go
// shippy-user-service/handler.go
package main

import (
	"errors"
	"fmt"
	"log"

	pb "github.com/EwanValentine/shippy-user-service/proto/auth"
	micro "github.com/micro/go-micro"
	"golang.org/x/crypto/bcrypt"
	"golang.org/x/net/context"
)

const topic = "user.created"

type service struct {
	repo         Repository
	tokenService Authable
	Publisher    micro.Publisher
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
	log.Println("Logging in with:", req.Email, req.Password)
	user, err := srv.repo.GetByEmail(req.Email)
	log.Println(user, err)
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

	log.Println("Creating user: ", req)

	// Generates a hashed version of our password
	hashedPass, err := bcrypt.GenerateFromPassword([]byte(req.Password), bcrypt.DefaultCost)
	if err != nil {
		return errors.New(fmt.Sprintf("error hashing password: %v", err))
	}

	req.Password = string(hashedPass)
	if err := srv.repo.Create(req); err != nil {
		return errors.New(fmt.Sprintf("error creating user: %v", err))
	}

	res.User = req
	if err := srv.Publisher.Publish(ctx, req); err != nil {
		return errors.New(fmt.Sprintf("error publishing event: %v", err))
	}

	return nil
}

func (srv *service) ValidateToken(ctx context.Context, req *pb.Token, res *pb.Token) error {

	// Decode token
	claims, err := srv.tokenService.Decode(req.Token)

	if err != nil {
		return err
	}

	if claims.User.Id == "" {
		return errors.New("invalid user")
	}

	res.Valid = true

	return nil
}
```

现在运行`$ make build && make run`。然后前往您的shippy- emailservice服务并运行`$ make build && make run`。一旦两者都运行，运行:

```bash
$ docker run -p 8080:8080 \
    -e MICRO_REGISTRY=mdns \
    microhq/micro api \
    --handler=rpc \
    --address=:8080 \
    --namespace=shippy
```

这在Docker容器中作为端口8080的rpc处理程序运行微api网关。我们告诉它在本地使用mdns作为注册表，与其他服务相同。最后，我们告诉它使用名称空间shippy，这是我们所有服务名称的第一部分shippy.auth或shippy.email。这很重要，因为它默认为go.micro.api，在这种情况下，它将无法找到我们的服务来代理它们。

我们的用户服务方法现在可以通过以下方式调用:

创建一个用户:

```bash
curl -XPOST -H 'Content-Type: application/json' \
    -d '{ "service": "shippy.auth", "method": "Auth.Create", "request": { "user": { "email": "ewan.valentine89@gmail.com", "password": "testing123", "name": "Ewan Valentine", "company": "BBC" } } }' \
    http://localhost:8080/rpc
```

如您所见，我们在请求中包括我们希望被路由的服务、我们想要使用的服务的方法，以及我们希望使用的数据。

验证一个用户:

```bash
$ curl -XPOST -H 'Content-Type: application/json' \
    -d '{ "service": "shippy.auth", "method": "Auth.Auth", "request":  { "email": "your@email.com", "password": "SomePass" } }' \
    http://localhost:8080/rpc
```

## 寄售服务

现在是时候重新启动我们的寄售服务了，`$ make build && make run`。我们不需要在这里修改任何东西，但是，运行rpc代理，我们应该能够做到:

创建一个委托:

```bash
$ curl -XPOST -H 'Content-Type: application/json' \
    -d '{
      "service": "shippy.consignment",
      "method": "ConsignmentService.Create",
      "request": {
        "description": "This is a test",
        "weight": "500",
        "containers": []
      }
    }' --url http://localhost:8080/rpc
```

## 船服务

为了测试我们的用户界面，我们需要运行的最终服务是我们的船只服务，这里没有什么变化，所以只要运行`$ make build && make run`。

## 用户界面

让我们使用新的rpc端点，并创建一个UI。我们将使用React，但你可以随意使用任何你喜欢的。请求都是一样的。我将使用来自Facebook的react-create-app library。`$ npm install -g react-create-app`.一旦安装好，你就可以做`$ react-create-app shippy-ui`了。这将为您创建一个反应应用程序的框架。

所以让我们开始…

```js
// shippy-ui/src/App.js
import React, { Component } from 'react';
import './App.css';
import CreateConsignment from './CreateConsignment';
import Authenticate from './Authenticate';

class App extends Component {

  state = {
    err: null,
    authenticated: false,
  }

  onAuth = (token) => {
    this.setState({
      authenticated: true,
    });
  }

  renderLogin = () => {
    return (
      <Authenticate onAuth={this.onAuth} />
    );
  }

  renderAuthenticated = () => {
    return (
      <CreateConsignment />
    );
  }

  getToken = () => {
    return localStorage.getItem('token') || false;
  }

  isAuthenticated = () => {
    return this.state.authenticated || this.getToken() || false;
  }

  render() {
    const authenticated = this.isAuthenticated();
    return (
      <div className="App">
        <div className="App-header">
          <h2>Shippy</h2>
        </div>
        <div className='App-intro container'>
          {(authenticated ? this.renderAuthenticated() : this.renderLogin())}
        </div>
      </div>
    );
  }
}

export default App;
```

现在，让我们添加主要的两个组件，身份验证和createcon:

```js
// shippy-ui/src/Authenticate.js
import React from 'react';

class Authenticate extends React.Component {

  constructor(props) {
    super(props);
  }

  state = {
    authenticated: false,
    email: '',
    password: '',
    err: '',
  }

  login = () => {
    fetch(`http://localhost:8080/rpc`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        request: {
          email: this.state.email,
          password: this.state.password,
        },
        service: 'shippy.auth',
        method: 'Auth.Auth',
      }),
    })
    .then(res => res.json())
    .then(res => {
      this.props.onAuth(res.token);
      this.setState({
        token: res.token,
        authenticated: true,
      });
    })
    .catch(err => this.setState({ err, authenticated: false, }));
  }

  signup = () => {
    fetch(`http://localhost:8080/rpc`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        request: {
          email: this.state.email,
          password: this.state.password,
          name: this.state.name,
        },
        method: 'Auth.Create',
        service: 'shippy.auth',
      }),
    })
    .then((res) => res.json())
    .then((res) => {
      this.props.onAuth(res.token.token);
      this.setState({
        token: res.token.token,
        authenticated: true,
      });
      localStorage.setItem('token', res.token.token);
    })
    .catch(err => this.setState({ err, authenticated: false, }));
  }

  setEmail = e => {
    this.setState({
      email: e.target.value,
    });
  }

  setPassword = e => {
    this.setState({
      password: e.target.value,
    });
  }

  setName = e => {
    this.setState({
      name: e.target.value,
    });
  }

  render() {
    return (
      <div className='Authenticate'>
        <div className='Login'>
          <div className='form-group'>
            <input
              type="email"
              onChange={this.setEmail}
              placeholder='E-Mail'
              className='form-control' />
          </div>
          <div className='form-group'>
            <input
              type="password"
              onChange={this.setPassword}
              placeholder='Password'
              className='form-control' />
          </div>
          <button className='btn btn-primary' onClick={this.login}>Login</button>
          <br /><br />
        </div>
        <div className='Sign-up'>
          <div className='form-group'>
            <input
              type='input'
              onChange={this.setName}
              placeholder='Name'
              className='form-control' />
          </div>
          <div className='form-group'>
            <input
              type='email'
              onChange={this.setEmail}
              placeholder='E-Mail'
              className='form-control' />
          </div>
          <div className='form-group'>
            <input
              type='password'
              onChange={this.setPassword}
              placeholder='Password'
              className='form-control' />
          </div>
          <button className='btn btn-primary' onClick={this.signup}>Sign-up</button>
        </div>
      </div>
    );
  }
}

export default Authenticate;
```

```js
// shippy-ui/src/CreateConsignment.js
import React from 'react';
import _ from 'lodash';

class CreateConsignment extends React.Component {

  constructor(props) {
    super(props);
  }

  state = {
    created: false,
    description: '',
    weight: 0,
    containers: [],
    consignments: [],
  }

  componentWillMount() {
    fetch(`http://localhost:8080/rpc`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        service: 'shippy.consignment',
        method: 'ConsignmentService.Get',
        request: {},
      })
    })
    .then(req => req.json())
    .then((res) => {
      this.setState({
        consignments: res.consignments,
      });
    });
  }

  create = () => {
    const consignment = this.state;
    fetch(`http://localhost:8080/rpc`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        service: 'shippy.consignment',
        method: 'ConsignmentService.Create',
        request: _.omit(consignment, 'created', 'consignments'),
      }),
    })
    .then((res) => res.json())
    .then((res) => {
      this.setState({
        created: res.created,
        consignments: [...this.state.consignments, consignment],
      });
    });
  }

  addContainer = e => {
    this.setState({
      containers: [...this.state.containers, e.target.value],
    });
  }

  setDescription = e => {
    this.setState({
      description: e.target.value,
    });
  }

  setWeight = e => {
    this.setState({
      weight: Number(e.target.value),
    });
  }

  render() {
    const { consignments, } = this.state;
    return (
      <div className='consignment-screen'>
        <div className='consignment-form container'>
          <br />
          <div className='form-group'>
            <textarea onChange={this.setDescription} className='form-control' placeholder='Description'></textarea>
          </div>
          <div className='form-group'>
            <input onChange={this.setWeight} type='number' placeholder='Weight' className='form-control' />
          </div>
          <div className='form-control'>
            Add containers...
          </div>
          <br />
          <button onClick={this.create} className='btn btn-primary'>Create</button>
          <br />
          <hr />
        </div>
        {(consignments && consignments.length > 0
          ? <div className='consignment-list'>
              <h2>Consignments</h2>
              {consignments.map((item) => (
                <div>
                  <p>Vessel id: {item.vessel_id}</p>
                  <p>Consignment id: {item.id}</p>
                  <p>Description: {item.description}</p>
                  <p>Weight: {item.weight}</p>
                  <hr />
                </div>
              ))}
            </div>
          : false)}
      </div>
    );
  }
}

export default CreateConsignment;
```

注意:我还添加了Twitter Bootstrap 到/public/index.html和修改了一些css。

现在运行用户界面`$ npm start`。这应该会在您的浏览器中自动打开。现在，您应该能够注册和登录，并查看托运表单，您可以在其中创建新的托运。在您的dev工具中查看网络选项卡，并查看从不同的微服务中触发和获取数据的rpc方法。

