---
title: SDKMAN! 软件开发工具包管理器
date: 2017-11-30 13:11:25
tags:
- 工具
- JVM
categories:
- 包管理器

---


![](http://sdkman.io/img/sdk-man-small-pattern.svg)

`SDKMAN!`是在大多数Unix系统上管理多个软件开发套件的并行版本的工具。它提供了一个方便的命令行接口(CLI)和用于安装、切换、删除和列出候选对象的API。以前被称为GVM的Groovy环境管理器，它是由非常有用的RVM和rbenv工具所启发的，这些工具在Ruby社区中广泛使用。

<!-- more -->

## 安装

在类unix平台上安装SDKMAN!，就像以前一样容易。SDKMAN!在Mac OSX、Linux、Cygwin、Solaris和FreeBSD上可以顺利安装。我们还支持Bash和ZSH shell。

只需打开一个新的终端并输入:

```Bash
$ curl -s "https://get.sdkman.io" | bash
```

按照屏幕上的说明完成安装。

接下来，打开一个新的终端或输入:

```Bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
```

最后，运行以下代码片段，以确保安装成功:

```Bash
$ sdk version
```

如果一切顺利，则应该显示该版本:

```Bash
sdkman 5.0.0+51
```

### Beta通道

对于更喜欢冒险的人来说，我们有一个测试频道。所有新的CLI特性都将首先针对这一组用户进行试用。Beta版本在大多数情况下可以被认为是稳定的，但偶尔也会中断。要加入测试程序，只需更新`~/.sdkman/etc/config`文件如下:

```
sdkman_beta_channel=true
```

接下来，打开一个新的终端并执行一个强制更新:

```Bash
$ sdk selfupdate force
```

要离开测试通道，只需将上面的配置设置为`false`，并遵循相同的步骤。

### 卸载

在不太可能的情况下，您希望卸载SDKMAN！我们还没有自动化的方法来做这件事。如果你真的想把它从你的系统中删除，那就很容易做到。下面的内容将指导您进行备份，然后从系统中删除整个安装。

```Bash
tar zcvf ~/sdkman-backup_$(date +%F-%kh%M).tar.gz -C ~/ .sdkman
$ rm -rf ~/.sdkman
```

最后一步是编辑和删除您的`.bashrc`, `.bash_profile` 和/或 `.profile`文件中的初始化代码片段。
如果你使用ZSH，从`.zshrc`中删除它。
要删除的代码片段如下:

```Bash
#THIS MUST BE AT THE END OF THE FILE FOR SDKMAN TO WORK!!!
[[ -s "/home/dudette/.sdkman/bin/sdkman-init.sh" ]] && source "/home/dudette/.sdkman/bin/sdkman-init.sh"
```

一旦删除，您就成功地从你的机器卸载了SDKMAN！。

### 安装到自定义位置

安装SDKMAN！到一个除`$HOME/.sdkman`以外的自定义位置是可能的。
这可以通过在安装之前将您的定制位置导出为`SDKMAN_DIR`来实现。
只需打开一个新的终端并输入:

```Bash
$ export SDKMAN_DIR="/usr/local/sdkman" && curl -s "https://get.sdkman.io" | bash
```

为此，您的用户对该文件夹拥有完全访问权限是至关重要的。同样重要的是，如果该文件夹不存在，SDKMAN!将尝试创建它。

## 使用

### 安装SDK

#### 最新稳定版本

通过运行下面的命令，安装您的SDK的最新稳定版本(比如Java JDK)。

```Bash
$ sdk install java
```

您将看到如下输出:

```Bash
Downloading: java 8u111

In progress...

######################################################################## 100.0%

Installing: java 8u111
Done installing!
```

如果您希望将此版本设置为默认值，则会提示您。

```Bash
Do you want java 8u111 to be set as default? (Y/n):
```

回答yes(或回车)将确保所有后续的shell都将在默认情况下使用这个版本的SDK。

```Bash
Setting java 8u111 as default.
```

#### 特定版本

需要一个SDK的特定版本吗?简单地限定你需要的版本:

```Bash
$ sdk install scala 2.12.1
```

所有后续步骤都与上面相同。

#### 安装本地版本

需要一个快照吗?已经有本地安装了吗?设置一个本地版本:

```Bash
$ sdk install groovy 3.0.0-SNAPSHOT /path/to/groovy-3.0.0-SNAPSHOT
```

### 删除版本

删除一个安装版本。

```Bash
$ sdk uninstall scala 2.11.6
```

### 候选列表

获得一份可用的候选列表:

```Bash
$ sdk list
```

这将提供一个可搜索的字母列表，其中有名称、当前稳定的默认版本、网站URL、描述以及每个候选人的简单安装命令。输出被用管道连接到更少的标准键盘快捷键，可以使用`q`退出。

```Bash

================================================================================
Available Candidates
================================================================================
q-quit                                  /-search down
j-down                                  ?-search up
k-up                                    h-help
--------------------------------------------------------------------------------
Groovy (2.4.5)                                       http://www.groovy-lang.org/

Groovy is a powerful, optionally typed and dynamic language, with static-typing
and static compilation capabilities, for the Java platform aimed at multiplying
developers’ productivity thanks to a concise, familiar and easy to learn syntax.
It integrates smoothly with any Java program, and immediately delivers to your
application powerful features, including scripting capabilities, Domain-Specific
Language authoring, runtime and compile-time meta-programming and functional
programming.

                                                            $ sdk install groovy
--------------------------------------------------------------------------------
Scala (2.11.7)                                        http://www.scala-lang.org/
...


```

### 版本列表

要获得候选版本的列表:

```Bash
sdk list groovy
```

这将产生一个列表视图，显示SDK的可用、本地、安装和当前版本。

```Bash

================================================================================
Available Groovy Versions
================================================================================
 > * 2.4.4                2.3.1                2.0.8                1.8.3
     2.4.3                2.3.0                2.0.7                1.8.2
     2.4.2                2.2.2                2.0.6                1.8.1
     2.4.1                2.2.1                2.0.5                1.8.0
     2.4.0                2.2.0                2.0.4                1.7.9
     2.3.9                2.1.9                2.0.3                1.7.8
     2.3.8                2.1.8                2.0.2                1.7.7
     2.3.7                2.1.7                2.0.1                1.7.6
     2.3.6                2.1.6                2.0.0                1.7.5
     2.3.5                2.1.5                1.8.9                1.7.4
     2.3.4                2.1.4                1.8.8                1.7.3
     2.3.3                2.1.3                1.8.7                1.7.2
     2.3.2                2.1.2                1.8.6                1.7.11
     2.3.11               2.1.1                1.8.5                1.7.10
     2.3.10               2.1.0                1.8.4                1.7.1

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================


```

### 使用版本

在当前终端选择使用一个给定版本:

```Bash
$ sdk use scala 2.12.1
```

重要的是要认识到，这将只改变当前shell的候选版本。要使此更改持久，请使用`default`命令。

### 默认的版本

选择一个给定版本的默认值:

```Bash
$ sdk default scala 2.11.6
```

这将确保所有后续的shell都将以2.11.6的版本开始使用。

### 当前版本(s)

要了解目前正在使用的候选:

```Bash
$ sdk current java
  Using java version 8u111
```

要了解目前所有候选的使用情况:

```Bash
$ sdk current
  Using:
  groovy: 2.4.7
  java: 8u111
  scala: 2.12.1
```

### 过时的版本(s)

看看你的系统上的候选人目前的情况是怎样的:

```Bash
$ sdk outdated springboot
  Outdated:
  springboot (1.2.4.RELEASE, 1.2.3.RELEASE < 1.2.5.RELEASE)
```

看看所有候选都过时了:

```Bash
$ sdk outdated
  Outdated:
  gradle (2.3, 1.11, 2.4, 2.5 < 2.6)
  grails (2.5.1 < 3.0.4)
  springboot (1.2.4.RELEASE, 1.2.3.RELEASE < 1.2.5.RELEASE)
```

### SDKMAN！版本

显示当前的SDKMAN！版本

```Bash
$ sdk version
```

### 广播消息

在命令行上获取最新的SDK发布通知:

```Bash

$ sdk broadcast
==== BROADCAST =================================================================
* 06/12/16: Scala 2.12.1 released on SDKMAN! #scala
* 23/11/16: Gradle 3.2.1 released on SDKMAN! #gradle
* 22/11/16: Ceylon 1.3.1 released on SDKMAN! #ceylonlang
================================================================================
```

值得一提的是，每当在SDKMAN!上发布一个SDK版本时，在使用CLI时将会出现一个通知。每一次新的广播都被推到推特上。

### 离线模式

最初被称为“飞机模式”，它允许SDKMAN!在离线工作时发挥作用。它有一个参数，可以通过它来启用或禁用离线模式。

```Bash

$ sdk offline enable
  Forced offline mode enabled.

$ sdk offline disable
  Online mode re-enabled!
```

当在离线模式下运行时，大多数命令仍然可以工作，即使它们的操作是缩小的。一个例子是list命令，它只显示当前安装的和活动的版本(s):

```Bash
$ sdk list
  ------------------------------------------------------------
  Offline Mode: only showing installed groovy versions
  ------------------------------------------------------------
   > 2.4.4
   * 2.4.3
  ------------------------------------------------------------
  * - installed
  > - currently in use
  ------------------------------------------------------------
```

当因特网变得可用/不可用时，离线模式也将自动禁用/启用。当然，需要网络连接的命令不会起作用，但会发出警告。

### 自升级

安装一个新版本的SDKMAN！如果可用。

```Bash
$ sdk selfupdate
```

如果没有新版本可用，将显示适当的消息。重新安装可能是通过将force参数传递给命令:

```Bash
$ sdk selfupdate force
```

自动检查新版本的SDKMAN！也将代表用户执行。

### 冲洗

有时可能需要冲洗 SDKMAN!的本地状态。flush命令可以帮助执行以下操作，并允许执行以下操作:

#### Candidates

```Bash
$ sdk flush candidates
```

清除候选列表。打开一个新的终端将获取并存储最新的列表。当一个新的候选在SDKMAN上可用时，通常需要这样做

#### Broadcast

```Bash
$ sdk flush broadcast
```

清除广播缓存，下载下一个命令调用的最新可用新闻。

#### Archives

```Bash
$ sdk flush archives
```
清除包含所有下载的SDK二进制文件的缓存。这可以占用很多空间，所以值得不时地清理一下！

#### 临时文件夹

```Bash
$ sdk flush temp
```

在安装新版本的候选版本和SDKMAN的时候，清除准备工作文件夹。

### 帮助

您可以通过运行以下命令获得基本的帮助:

```Bash
$ sdk help
```

这应该会产生一些类似的东西:

```Bash
Usage: sdk   [version]
       sdk offline

   commands:
       install   or i     [version]
       uninstall or rm    
       list      or ls   
       use       or u     [version]
       default   or d     [version]
       current   or c    [candidate]
       outdated  or o    [candidate]
       version   or v
       broadcast or b
       help      or h
       offline           
       selfupdate        [force]
       flush             

   candidate  :  ...
   version    :  where optional, defaults to latest stable if not provided

eg: sdk install groovy

```

### 配置
尽管配置是有限的，但是可配置项的列表会随着需要而增长。
可以在`~/.sdkman/etc/config`文件中找到配置。
目前，以下是可配置的:

```Bash
# make sdkman non-interactive, preferred for CI environments
sdkman_auto_answer=true|false

# perform automatic selfupdates
sdkman_auto_selfupdate=true|false

# disables SSL certificate verification
# https://github.com/sdkman/sdkman-cli/issues/327
# HERE BE DRAGONS....
sdkman_insecure_ssl=true|false

# disable GVM alias, for users of the Go Version Manager
sdkman_disable_gvm_alias=true|false

# configure curl timeouts
sdkman_curl_connect_timeout=5
sdkman_curl_max_time=4

# subscribe to the beta channel
sdkman_beta_channel=true

```

---

[Windows PowerShell 使用SDKMAN](http://blog.csdn.net/soslinken/article/details/76678577)
