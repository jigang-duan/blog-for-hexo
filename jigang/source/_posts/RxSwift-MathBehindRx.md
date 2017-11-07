---
title: RxSwift 背后的数学
date: 2017-10-27 09:39:57
tags:
- rxswift
- swift
- iOS
categories:
- rxswift

---

![reactivex.io](http://reactivex.io/assets/reactivex_bg.jpg)

Rx背后的数学
==============

## 观察者和迭代器/迭代器/生成器/序列之间的二元性

<!-- more -->

在观察者和生成器模式之间有一个对偶性。这使我们能够从异步回调世界过渡到序列转换的同步世界.

简而言之，迭代器和观察者模式都描述了序列。为什么迭代器定义一个序列是很明显的，但是观察者稍微复杂一点.

然而，有一个非常简单的例子，不需要太多的数学知识。假设在给定的时间，您正在观察鼠标光标在屏幕上的位置。随着时间的推移，这些鼠标位置形成了一个序列。这是一个可观察的序列.

可以访问一个序列的两种基本方法:

* 推(Push)接口 - 观察者(观察到的元素随着时间的推移形成一个序列)
* 拉(Pull)接口-迭代器/枚举器/生成器

你们也可以在这个视频中看到更正式的解释:

* [专家:Erik Meijer和Erik Meijer——.NET响应框架(Rx)的内部(视频)](https://www.youtube.com/watch?v=looJcaeboBY)
* [响应式编程概述(来自Netflix的Jafar Husain)](https://www.youtube.com/watch?v=dwP1TNXE6fc)
