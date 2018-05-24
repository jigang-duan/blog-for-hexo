---
title: 'micro new [service]'
date: 2018-03-23 21:21:14
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

**micro new**命令是为micro服务生成样板模板的快捷方式。

<!-- more -->

## 使用

通过指定相对于$GOPATH的目录路径来创建新服务。

```bash
micro new github.com/micro/foo
```

将运行：

```bash
micro new github.com/micro/foo

creating service go.micro.srv.foo
creating /Users/asim/checkouts/src/github.com/micro/foo
creating /Users/asim/checkouts/src/github.com/micro/foo/main.go
creating /Users/asim/checkouts/src/github.com/micro/foo/handler
creating /Users/asim/checkouts/src/github.com/micro/foo/handler/example.go
creating /Users/asim/checkouts/src/github.com/micro/foo/subscriber
creating /Users/asim/checkouts/src/github.com/micro/foo/subscriber/example.go
creating /Users/asim/checkouts/src/github.com/micro/foo/proto/example
creating /Users/asim/checkouts/src/github.com/micro/foo/proto/example/example.proto
creating /Users/asim/checkouts/src/github.com/micro/foo/Dockerfile
creating /Users/asim/checkouts/src/github.com/micro/foo/README.md

download protobuf for micro:

go get github.com/micro/protobuf/{proto,protoc-gen-go}

compile the proto file example.proto:

protoc -I/Users/asim/checkouts/src \
	--go_out=plugins=micro:/Users/asim/checkouts/src \
	/Users/asim/checkouts/src/github.com/micro/foo/proto/example/example.proto
```

## 选项

指定更多的选项，如名称空间、类型、fqdn和别名

```bash
micro new --fqdn com.example.srv.foo github.com/micro/foo
```

## 帮助

```bash
NAME:
   micro new - Create a new micro service

USAGE:
   micro new [command options] [arguments...]

OPTIONS:
   --namespace "go.micro"	Namespace for the service e.g com.example
   --type "srv"			Type of service e.g api, srv, web
   --fqdn 			FQDN of service e.g com.example.srv.service (defaults to namespace.type.alias)
   --alias 			Alias is the short name used as part of combined name if specified
```