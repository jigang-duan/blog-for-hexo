---
title: Go微服务教程-第七部分
date: 2018-05-10 21:56:56
tags:
 - golang
 - go-micro
categories:
 - Go微服务

---

![micro-diag](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526284371762&di=4d74aecb4c8b43534361e1c232276d81&imgtype=0&src=http%3A%2F%2Fi0.wp.com%2Fwww.virtuaniz.com%2Fwp-content%2Fuploads%2F2014%2F08%2Fcloud-computing-advantages-benifits-virtuaniz.png%3Fresize%3D660%2C330)

在前一篇文章中，我们简要介绍了用户界面和web客户端，以及如何使用微工具包rpc代理与新创建的rpc服务进行交互。

这篇文章，我们将讨论如何创建一个云环境来承载我们的服务。我们将使用[Terraform](https://www.terraform.io/)来架构我们的云集群在谷歌云平台上。这应该是一篇相当短的文章，但它很重要。

<!-- more -->

# 为什么Terraform

