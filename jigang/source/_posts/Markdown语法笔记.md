---
title: Markdown语法笔记
date: 2017-03-20 12:26:28
tags:
- Markdown
---

## 关于Markdown
Markdown是一个 Web 上使用的文本到HTML的转换工具，可以通过简单、易读易写的文本格式生成结构化的HTML文档。

<!-- more -->

## Markdown简单语法
主要分为区块元素和区段元素。
### 区块元素
#### 1.段落和换行
一个 Markdown 段落是由一个或多个连续的文本行组成，它的前后要有一个以上的空行。

#### 2.标题
用#标识符表示
``` bash
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
```
显示如下：
># 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题

#### 3.区块引用
* 在段落的第一行最前面加">"
* 区块引用可以嵌套（例如：引用内的引用），只要根据层次加上不同数量的 >
* 区块内也可以套用其他的 Markdown 语法，包括加粗、列表、代码区块等

``` bash
>如人以手，指月示人。彼人因指，当因看月。若复观指以为月体。此人岂唯亡失月轮，亦亡其指。
何以故，以所标指为明目故，岂唯亡指，亦复不识明之与暗。
何以故，即以指体，为月明性，明暗二性，无所了故。
>>                                                            《楞严经》
```
显示如下：
>如人以手，指月示人。彼人因指，当因看月。若复观指以为月体。此人岂唯亡失月轮，亦亡其指。
何以故，以所标指为明目故，岂唯亡指，亦复不识明之与暗。
何以故，即以指体，为月明性，明暗二性，无所了故。
>>                                                            《楞严经》

#### 4.列表
有序列表和无序列表
* 无序列表使用星号、加号或是减号作为列表标记

``` bash
* 星号
+ 加号
- 减号
```
显示如下：
* 星号
+ 加号
- 减号

* 有序列表则使用数字接着一个英文句点

``` bash
1. 第一
2. 第二
```
显示如下：
1. 第一
2. 第二

#### 5.代码区块
要在 Markdown 中建立代码区块很简单，只要简单地缩进 4 个空格或是 1 个制表符就可以
``` bash
    代码区块
```
显示如下：

    代码区块

#### 6.分隔线
你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。
``` bash
***
```
***
``` bash
---
```
---
``` bash
* * *
```
* * *


### 区段元素

#### 1.链接
方块括号后面紧接着圆括号并插入网址链接即可
``` bash
[我的博客地址](https://jigangduan.github.io)
```
[我的博客地址](https://jigangduan.github.io)

#### 2.强调
``` bash
*强调*
_强调_
```
*强调*
_强调_
``` bash
**粗体**
```
**粗体**
``` bash
~~删除线~~
```
~~删除线~~

空格

Markdown语法会忽略首行开头的空格，如果要体现出首行开头空两个的效果，可以使用 全角符号下的空格 ，windows下使用 shift+空格 切换。

#### 3.行内标记
行内标记用反引号把它包起来' '

#### 4.插入图片

``` bash
![天黑黑](URL)
```

![天黑黑](http://cdn.duitang.com/uploads/blog/201409/05/20140905182021_vH2wL.thumb.700_0.jpeg)

#### 反斜杠
Markdown可以利用反斜杠来插入一些在语法中有其它意义的符号，例如：如果你想要用星号加在文字旁边的方式来做出强调效果，你可以在星号的前面加上反斜杠

#### 自动邮箱链接

Markdown支持以比较简短的自动链接形式来处理电子邮件信箱
``` bash
<jigang.duan@tcl.com>
```
<jigang.duan@tcl.com>

## 正文
### 1. 内容目录
使用[TOC]引用目录

**备注：** 有一些编辑器不支持[TOC]标记,比如我使用的Atom

### 2. 加强代码块

使用\`\`\`  + 语言名称 进行标记
``` bash
``` swift
// MARK: - Date component
extension Date {

    var year: Int {
        return Calendar.current.component(.year, from: self)
    }
}
```
```
显示如下：

``` swift
// MARK: - Date component
extension Date {

    var year: Int {
        return Calendar.current.component(.year, from: self)
    }
}
```


### 3. 脚注

备注：关于注脚好像每个编辑器表示方式会有所不用

### 4.标签和分类

一般在文首输入tags添加标签，categories添加分类
``` bash
tags:
- Markdown
- 语言

categories:
- 技术
```


### 5. 待办事宜 Todo 列表
使用带有 [ ] 或 [x] （未完成或已完成）项的列表语法撰写一个待办事宜列表

### 6.表格
``` bash
|表头| 1 | 2 |
| -- | --: | :--: |
| a | a1 | a2 |
| b | b1 | b2 |
```
|表头| 1 | 2 |
| -- | --: | :--: |
| a | a1 | a2 |
| b | b1 | b2 |

### 7.流程图和时序图
* 流程图
* 时序图

### 8. LaTeX 公式


## 参考
[wowubuntu.com/markdown](http://wowubuntu.com/markdown/)
