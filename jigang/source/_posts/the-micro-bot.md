---
title: "Micro Bot - 微服务聊天工具"
date: 2018-03-18 23:27:02
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

今天我想和你们谈谈机器人。

<!-- more -->

## 机器人吗?真的…

现在我知道你在想什么了。现在有很多关于机器人的炒作。如果你对聊天机器人很熟悉，你就会知道这不是一个新的概念，事实上，它可以追溯到伊丽莎时代。对机器人的重新迷恋真的出现了，因为我们发现了更多有用的功能，而不仅仅是娱乐。他们还对可能成为下一个超越应用的交互界面，即会话用户界面进行了研究。

在工程领域，机器人不仅是用于会话目的，它们对于操作任务来说也非常有用。所以，我们大多数的技术人员都已经熟悉了“ChatOps”这个词。GitHub从这个词的起源开始就被认为是这个术语的起源，因为它是一个用于管理技术和业务操作任务的聊天机器人[Hubot](https://hubot.github.com/)的创建和使用。

请看看杰西·纽兰[在GitHub上对ChatOpts](https://www.youtube.com/watch?v=NST3u-GjjFw)的演讲，以了解更多。

像它一样的Hubot和机器人已经被证明在技术组织中非常有用，并且成为ops和自动化领域的主要工具。通过HipChat或Slack来指导机器人执行任务的能力是相当强大的。它对整个团队的可见性有直接的价值。每个人都能看到你在做什么，影响是什么。

## 这与micro服务有什么关系?

Micro ,microservice toolkit包括许多提供进入运行系统的入口点的服务。API、Web仪表板、CLI等都是用于交互和观察您的微服务环境的所有固定的入口点。在过去的几个月里，很明显，Bot是另一种形式的入口点，以互动和观察，并且它应该是在Micro观世界的一等公民。

所以,…

![](https://micro.mu/blog/assets/images/micro_bot.png)

让我先说一下，Micro Bot是一个非常早期的原型，目前专注于提供与CLI相同的特性。我们不是拥有人工智能基于ChatOps…但也许有一天…

Micro Bot包括hubot类似于脚本的语义，以及一种实现新输入的方式，如Slack和Hipchat。这是一个粗略的版本1，但我相信，只要有什么东西能起作用，我就会相信，通过这样做，我们就能更快地提高机器人的速度。希望与社区的贡献!

该Bot包括所有的CLI命令和对Slack和HipChat的输入。我们的社区机器人目前在演示环境中运行，现在处于微松弛状态!如果你想看的话，请加入我们。

在短期内，我们将考虑添加更多的输入插件，如IRC、XMPP和命令，这些插件可以简化管理运行环境中的微服务。如果你对其他的输入或命令有想法，或者想要提交一份关于某件事的公关，那么你的贡献就不受欢迎了。更多的插件可以在github.com/micro/go-plugins/bot中找到。

这是Micro生态系统可编程机器人的基本框架。考虑到整个工具包的可插入性，在bot形式中提供类似的东西是很有意义的。

让我们继续讨论输入和命令是如何工作的。

## Inputs

Inputs是micro bot如何连接到hipchat, slack, irc, xmpp等等。我们现在已经有了hipchat和slack的实现，这似乎涵盖了大量的用户。

这是Input接口。

```go
type Input interface {
	// Provide cli flags
	Flags() []cli.Flag
	// Initialise input using cli context
	Init(*cli.Context) error
	// Stream events from the input
	Stream() (Conn, error)
	// Start the input
	Start() error
	// Stop the input
	Stop() error
	// name of the input
	String() string
}
```

