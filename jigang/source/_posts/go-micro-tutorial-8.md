---
title: Go微服务教程-第八部分
date: 2018-05-10 21:57:01
tags:
 - golang
 - go-micro
categories:
 - Go微服务

---

![micro-diag](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1526284371762&di=4d74aecb4c8b43534361e1c232276d81&imgtype=0&src=http%3A%2F%2Fi0.wp.com%2Fwww.virtuaniz.com%2Fwp-content%2Fuploads%2F2014%2F08%2Fcloud-computing-advantages-benifits-virtuaniz.png%3Fresize%3D660%2C330)

在之前的文章中，我们研究了用[Terraform](https://terraform.io/)创建一个容器引擎集群。在本文中，我们将使用容器引擎和[Kubernetes](https://kubernetes.io/)来研究如何将容器部署到我们的集群中。

# Kubernetes

首先,Kubernetes是什么?Kubernetes是一个开源的容器管理框架。它是平台无关的，这意味着您可以在本地机器、AWS、谷歌云或其他任何地方运行它。它允许您使用解密配置控制容器组和它们的网络规则。

您只需编写描述哪些容器应该运行的yaml/json文件，以及在何处运行。您可以定义您的网络规则，例如任何端口转发。它还为您处理服务发现。

Kubernetes是云场景中最重要的添加物之一，它正迅速成为云容器管理的实际选择。这是一个很好的理解。

所以让我们开始吧!

首先，确保您在本地安装了kubectl cli:

```bash
$ gcloud components install kubectl
```

现在让我们确保您已经连接到您的集群并正确地进行了身份验证。首先，我们会登录并确认我们的身份。其次，我们将设置项目设置，以确保我们使用正确的项目ID和可用性区域。

```bash
$ echo "This command will open a web browser, and will ask you to login
$ gcloud auth application-default login

$ gcloud config set project shippy-freight
$ gcloud config set compute/zone eu-west2-a

$ echo "Now generate a security token and access to your KB cluster"
$ gcloud container clusters get-credentials shippy-freight-cluster
```

在上面的命令中，您可能需要用所选择的区域替换计算/区域，您的项目id和集群名称也可能与我的不同。

这里有一个大纲……

