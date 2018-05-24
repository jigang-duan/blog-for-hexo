---
title: micro web
date: 2018-03-22 07:09:09
tags:
 - golang
 - go-micro
categories:
 - go-micro

---

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1521376593279&di=576a492a87f7c5608921cec413e24295&imgtype=0&src=http%3A%2F%2Fstatic.open-open.com%2Fnews%2FuploadImg%2F20151218%2F20151218212318_391.jpg)

**micro web**提供了查看和查询服务的仪表板。

<!-- more -->

## 入门指南

### 安装

```bash
go get -u github.com/micro/micro
```

### 运行

```bash
micro web
```

### ACME

默认使用ACME安全服务

```bash
MICRO_ENABLE_ACME=true micro web
```

可以选择指定一个主机白名单

```bash
MICRO_ENABLE_ACME=true \
MICRO_ACME_HOSTS=example.com,api.example.com \
micro web
```

### TLS证书

API支持安全地使用TLS证书

```bash
MICRO_ENABLE_TLS=true \
MICRO_TLS_CERT_FILE=/path/to/cert \
MICRO_TLS_KEY_FILE=/path/to/key \
micro web
```

## 截图

![](https://github.com/micro/docs/raw/master/images/web1.png)

![](https://github.com/micro/docs/raw/master/images/web2.png)

![](https://github.com/micro/docs/raw/master/images/web3.png)

![](https://github.com/micro/docs/raw/master/images/web4.png)