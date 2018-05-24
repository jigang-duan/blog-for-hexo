---
title: micro-heartbeat
date: 2018-03-24 21:36:35
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

演示服务发现使用心跳。

<!-- more -->

## 依据

服务发现在启动和注销时提供服务发现。有时，这些服务可能会意外地死亡或被强行杀死或面临暂时的网络问题。在这些情况下，将在服务发现中保留陈旧的节点。如果自动删除服务，那将是理想的。

## 解决方案

Micro支持一个注册器TTL的选项，并为这个确切的原因注册间隔。TTL指定一个注册在发现之后应该存在多长时间，它会过期并被删除。间隔是服务重新注册以保存其在服务发现中的注册的时间。

这些选项都可以在go-micro和micro Toolkit中使用。

## Toolkit

使用像这样的标志运行工具箱中的任何组件

```bash
micro --register_ttl=30 --register_interval=15 api
```

这个示例显示，我们设置一个30秒的ttl，重新注册间隔为15秒。

## Go Micro

在宣布micro服务时，您可以将选项作为time.Duration传递

```go
service := micro.NewService(
	micro.Name("com.example.srv.foo"),
	micro.RegisterTTL(time.Second*30),
	micro.RegisterInterval(time.Second*15),
)
```