---
title: 猫的JavaScript
date: 2018-02-06 12:24:23
tags:
- how-to-become-a-react-native-developer
- JavaScript
- 'React Native'
categories:
- 'React Native'

---

![](http://jsforcats.com/images/substack-cats.png)

## 新程序员简介

**你的人类伙伴也可以轻易的做到！**

JavaScript是一种编程语言，或者，换句话说，是一种计算机被指令去做事情的方式。
就像一个人用“口哨”和“猫叫”控制人类一样，一个人控制着用编程语言编写的语句。
所有的web浏览器都能理解JavaScript，您可以利用它来让web页面做一些疯狂的事情！

JavaScript是一种使web页面更具交互性的方法。
如今，JavaScript在很多地方运行，而不仅仅是web浏览器——它运行在web服务器、手机甚至是机器人上。
这篇文章将会教你一些JavaScript基础知识，这样你就可以随时启动和运行。

> 实际时间:没有的很多。大概一两个小时吧。因为你是一只猫你不太可能跑，更有可能在太阳下躺着

<!-- more -->

## 不要做一只胆小的猫

即使是编程，你也会永远地站在你的脚上。与在笔记本电脑上的一杯水不同，这些教程中的任何东西都不会对你的电脑造成任何损害，即使你键入命令或者点击错误的按钮。
像猫一样，计算机程序员总是犯错误:拼写错误，忘记引号或括弧，忘记基本的功能(和yarn，lasers)的工作。
程序员更关心的是让它最终工作，而不是让它第一次工作。
学习的最好方法就是犯错误！

所以，不要成为一只胆小的猫！最糟糕的情况是，如果您陷入困境，您可能不得不在web浏览器中刷新这一页面。不过，不要担心，这种情况很少发生。

## 基础知识

现在这个页面上有JavaScript运行。让我们稍微玩一下。
为了简单起见，我假设您使用Google Chrome来阅读这一页(如果您不使用Chrome的话，这对我们两个人来说可能更容易些)。

首先，右键单击屏幕上的任何地方，然后单击检查元素，然后单击控制台选项卡。你应该看到这样的东西:

![](http://jsforcats.com/images/console.gif)

这是一个控制台，也称为“命令行”或“终端”。
基本上，这是一种将一件事一件一件地输入到电脑里的方法，然后立即让电脑回复。
它们作为一种学习工具非常有用(我几乎每天都在使用控制台)。

控制台有一些很酷的东西。
在这里，我开始输入一些东西，控制台正在帮助我，给我一个列表，让我可以继续输入所有可能的东西！
另一件你可以做的事情是在控制台中输入`1+1`，然后点击`Enter`键，看看会发生什么。

使用控制台是学习JavaScript的一个非常重要的部分。
如果你不知道某件事是否有效，或者你的命令是什么，那就去控制台，把它弄清楚！
这里有一个例子:

## Strings

因为我是一只猫，所以我想用那些该死的狗,把狗狗的每一个单词都替换掉。
首先进入你的控制台，输入几句包含`狗`至少一次的句子。
在JavaScript中，一串字母、数字、单词或其他任何东西都被称为字符串(如字符串中的字符串)。
字符串必须以引号开始和结束。
单`'`或双`"`，只是确保你在开始时使用的是一样的。

![](http://jsforcats.com/images/console-strings.gif)

看到令人讨厌的错误消息了吗?别担心，你没有违反任何法律。
当机器人告诉你你的程序有问题时，同步错误就是它听起来的样子。
头两个句子在开头和结尾都有匹配的引号，但是当我把单引号和双引号混合在一起时，我就感到害怕了

好，为了修复其中的一个句子(用我们的增强版本替换`dog`)，我们必须先保存原来的句子，这样当我们替换魔法的时候，我们可以把它叫起来。
注意，当我们将字符串输入到控制台时，该字符串是如何以红色重复的?这是因为我们没有告诉它要在任何地方保存这个句子，所以它才会返回(或者如果我们把事情搞砸了，它会给我们一个错误)。

## 值和变量

`值`是JavaScript中最简单的组件。
`1`是一个值，`true`是一个值，`”hello”`是一个值，`function() {}`是一个值，`list`会继续下去！
JavaScript中有几种不同`类型`的值，但我们不需要马上处理它们——您将会自然地了解到它们的代码！

为了存储值，我们使用`变量`。“变量”这个词的意思是“可以改变”，因为变量可以存储许多不同类型的值，并且可以多次更改它们的值。
它们很像邮箱。我们把一些东西放在一个变量里，比如我们的句子，然后给这个变量一个地址，我们可以用它来查找这个句子。
在现实生活中，邮箱必须有PO Box号码，但在JavaScript中，通常只使用小写字母或数字，而不需要任何空格。

![](http://jsforcats.com/images/console-variables.gif)

`var`是变量的简写，而等号的意思是把右边的东西放在左边的东西上。正如你所看到的，既然我们将句子存储在一个变量中，控制台不会立即返回我们的句子，而是给我们`未定义`的意思，这意味着没有任何东西可以返回。

如果您简单地将一个变量名输入到控制台，它将输出该变量中存储的值。
关于变量的一个注释是，默认情况下，当切换到另一个页面时，它们会消失。
例如，如果我在Chrome中点击Refresh按钮，我的`dogSentence`变量就会被抹去，就像它从来没有存在过一样。
但现在不要太担心这一点——你可以在键盘上敲击键盘上的上或下箭头，来完成你最近输入的所有东西。

## 函数

既然我们已经将语句存储在一个变量中，那么让我们更改一个存储在其中的单词！我们可以通过执行一个函数来做到这一点。函数是一种类型的值，它为我们提供一个特定的功能(也就是目的或行为)。我认为，把它们称为“行动”听起来很奇怪，所以他们用“函数”一词来代替。

JavaScript有一个名为`replace`的函数，它完全符合我们的要求！函数在括号中(0、1或多个)中接受任意数量的值，并返回任何值(`未定义的`)或更改的字符串。替换函数可用于任何字符串，并具有两个值:要取出的字符和要交换的字符。描述这些东西会让你感到困惑所以这里是一个视觉上的例子:

![](http://jsforcats.com/images/console-replace.gif)

请注意，即使在我们运行替换之后，它的值是一样的吗?这是因为`replace`函数(以及大多数JavaScript函数)接受我们赋予它的值，并返回一个新的值，而不需要修改传入的值。因为我们没有存储结果(在替换函数的左边没有`=`)，它只是在我们的控制台中打印了返回值。

## 标准库

您可能想知道在JavaScript中还有哪些函数可用。答案:一吨。在MDN(一个由Mozilla运行的站点拥有许多关于web技术的漂亮信息)，您可以在MDN中了解到许多 **构建的标准库** 。例如，这里是关于[JavaScript的数学对象的MDN页面](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Math)。

## 第三方JavaScript

还有很多JavaScript代码都 __没有内置__。来自第三方的JavaScript通常被称为“库”或“插件”。我最喜欢的一个叫做 **Underscore.js** 。让我们去抓取它并载入到我们的页面！第一次去Underscore网站,[http://underscorejs.org/](http://underscorejs.org/),点击下载链接(我通常使用开发版本,因为他们更容易读但都将给你相同的基本功能),然后复制到您的剪贴板的所有代码(你可以使用从编辑菜单中选择所有选择的一切)。然后把它粘贴到你的控制台，点击回车。现在，您的浏览器中有一个新的变量:`_`。`Underscore`给你提供了很多有用的功能。稍后我们将学习更多关于如何使用它们的知识。

![](http://jsforcats.com/images/underscore.gif)

## 制作新函数

你并不局限于使用他人的函数——你也可以自己写。这很简单!让我们创建一个名为`makeMoreExciting`的函数，它在字符串的末尾添加了一些感叹号。

```js
function makeMoreExciting(string) {
  return string + '!!!!'
}
```

在我的脑海里，我这样大声地读出来:“有一个函数叫做 `make more exciting`，它接收一个字符串，然后返回一个新的字符串，这个字符串的末尾有很多感叹号。如果我们不使用函数，我们将如何在控制台中手动写入。

![](http://jsforcats.com/images/custom-function-manually.gif)

表达式`string + '!!!!'`返回一个新的字符串，我们的变量名为string的变量保持不变(因为我们从未将它更新为`=`)。

让我们使用我们的函数，而不是手动操作。首先，将该函数粘贴到控制台，然后通过传入字符串来调用函数:

![](http://jsforcats.com/images/custom-function-call.gif)

您也可以通过传递指向一个字符串的变量来调用同一个函数(在上面的例子中，我们只是将字符串直接输入为一个值，而不是先将其保存到一个变量中):

![](http://jsforcats.com/images/custom-function-call-variable.gif)

`MakeMoreExciting (sentence)`的相当于`sentence + ‘!!!!’`。如果我们想要修改in-place(更新)`sentence `的值呢?简单地将函数的返回值保存回我们的`sentence `变量:

```js
var sentence = "time for a nap"
sentence = makeMoreExciting(sentence)
```

现在`sentence`里有了感叹号！注意，当您初始化一个变量时，您只需要使用var，这是您第一次使用它。在此之后，您不应该使用var，除非您想要重新初始化(重新设置/清除/清空)变量。

如果我们在函数中取出`return`  statement会发生什么?

![](http://jsforcats.com/images/custom-function-no-return.gif)

为什么`sentence`是空的吗?因为函数在默认情况下返回`undefined`！您可以通过返回某个值来返回一个值。函数应该具有一个值，如果它们改变了值或者创建了一个新的值，这个值应该在以后被使用，返回一个值(有趣的事实:这个样式的一个奇特的术语是函数式编程)。这是另一个函数，它不返回任何东西，而是使用另一种方法来显示输出:

```js
function yellIt(string) {
  string = string.toUpperCase()
  string = makeMoreExciting(string)
  console.log(string)
}
```

这个函数，`yellIt`，使用我们之前的函数`makeMoreExciting`，以及内置的字符串方法[toUpperCase](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/String/toUpperCase)。方法只是一个函数的名字，当它属于某个东西的时候——在这个例子中，toUpperCase是一个属于字符串的函数，所以我们可以把它称为一个方法或者一个函数。另一方面，`makeMoreExciting`不属于任何人，因此将其称为一种方法是不正确的(我知道这是一种混淆)。

该函数的最后一行是另一个内置函数，它只接受您提供的任何值，并将其输出到控制台

![](http://jsforcats.com/images/custom-function-console-log.gif)

那么上面的yellIt函数有什么问题吗?视情况而定!以下是两种主要的功能:
- 修改或创建值并返回它们的函数
- 函数接受值并执行一些无法返回的操作

`console.log `是第二种功能的例子:它把东西打印到你的控制台——你可以用眼睛看到的动作，但这不能用JavaScript的值来表示。我自己的经验法则是，把这两种函数分开，这是我重写yellIt函数的方式:

```js
function yellIt(string) {
  string = string.toUpperCase()
  return makeMoreExciting(string)
}

console.log(yellIt("i fear no human"))
```

`yellIt`变得更通用，这意味着它只做一两个简单的小事情，不知道如何将自己打印到控制台——在函数定义之外，该部分总是可以编程的。

## 循环

现在我们已经有了一些基本的技能我们可以开始变得懒惰。什么? !是的，这是对的:编程就是懒惰。Perl编程语言的发明者拉里沃尔称，懒惰是优秀程序员[最重要的美德](http://c2.com/cgi/wiki?LazinessImpatienceHubris)。如果电脑不存在，你将不得不手工做各种繁琐的工作，但如果你学会编程，你就可以整天躺在太阳底下，而电脑则可以为你运行程序。这是一种充满了放松的光荣的生活方式！

循环是利用计算机的力量的最重要的方法之一。还记得早些时候`Underscore.js`吗?
确保你在页面上加载了它(记住:你可以在你的键盘上敲上几下箭头，然后按下回车键，如果你需要的话)，然后在你的控制台中复制/粘贴这个。

```js
function logANumber(someNumber) {
  console.log(someNumber)
}
_.times(10, logANumber)
```

这段代码使用的是Underscore的[times](http://underscorejs.org/#times)方法，它包含1个数字和1个函数，然后从0开始，10个步骤是1，用每个步骤的数字来调用这个函数。

![](http://jsforcats.com/images/times-loop.png)

如果我们要手动写出上面的代码，它会是这样的:

```js
logANumber(0)
logANumber(1)
logANumber(2)
logANumber(3)
logANumber(4)
logANumber(5)
logANumber(6)
logANumber(7)
logANumber(8)
logANumber(9)
```

但是猫拒绝做这样的不必要的手工工作，所以我们必须总是问自己:“我是不是用最懒的方式做这件事?”

那么为什么这个叫做循环呢?可以这样想:如果我们要用JavaScript数组写出10个数字(从0到9)，它会是这样的:

```js
var zeroThroughTen = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

真正的`times `是访问每个数字并重复一个任务:在上面的例子中，任务是用当前的数字调用`logANumber `函数。以这种方式重复任务被称为循环遍历数组。

## Arrays

我已经提过几次了，但让我们花一分钟时间来了解它们。想象一下，你需要跟踪所有的朋友。好吧，数组会做得很好。想象一个数组，就像一个排序列表，你可以保存大量的东西。

这就是你的制作方法:

```js
var myCatFriends = ["bill", "tabby", "ceiling"]
```

甜蜜的!现在你有了一份你的猫友的名单。

元素(也就是数组中的单一项)存储在数组中，从0开始，从那里开始计数。`myCatFriends[0]`返回`bill`和`myCatFriends[1]`返回`tabby`。等等。

为了让你的好友离开你的全新的数组，你可以直接访问一个元素:

```js
console.log(myCatFriends[0])
```

![](http://jsforcats.com/images/array-access.png)

如果你在另一个晚上在最时髦的猫咪俱乐部里做了一个全新的猫咪朋友，你想把它们加到你的名单上，那就太简单了:`myCatFriends.push("super hip cat")`。

为了检查新猫是否能进入你的数组，你可以使用`.length`:

![](http://jsforcats.com/images/array-push-length.png)

注意到`push`是如何返回长度的?方便的!还需要注意的是，数组将始终保持顺序，这意味着它们将记住您添加或定义的顺序的顺序。并不是所有的JavaScript都保存了排序，所以要记住数组的特殊属性！

## Objects

数组对列表很好，但是对于其他任务，它们可能很难处理。想想我们的猫朋友圈吧。如果您还想存储更多的名称，又该如何呢?

```js
var myCatFriends = ["bill", "tabby", "ceiling"]
var lastNames = ["the cat", "cat", "cat"]
var addresses = ["The Alley", "Grandmas House", "Attic"]
```

有时在一个变量中包含所有的地址或名称是很好的。但有时你有一只猫，让我们说bill，你只是想查一下那只猫的地址。使用数组需要大量的工作，因为你不能只说“嗨，数组，给我Bill的地址”因为“Bill”在一个数组中，他的地址是完全不同的数组。

![](http://jsforcats.com/images/array-lookup.png)

这可能是很脆弱的，因为如果我们的数组改变了，并且我们在开始的时候添加了一个新的cat，那么我们就必须更新我们的账单变量，以指向在数组中新的Bill信息的位置！这里有一个更容易维护的方式来存储这样的信息:使用对象:

```js
var firstCat = { name: "bill", lastName: "the cat", address: "The Alley" }
var secondCat = { name: "tabby", lastName: "cat", address: "Grandmas House" }
var thirdCat = { name: "ceiling", lastName: "cat", address: "Attic" }
```

我们为什么要这样做呢?因为现在我们为每只猫都有一个变量，我们可以用它来让猫的值更方便、更容易读懂。

![](http://jsforcats.com/images/object-lookup.png)

你可以把钥匙放在钥匙圈上。每一个都是针对特定的门，如果你的钥匙上有漂亮的标签，你就能快速打开门。事实上，左边的东西被称为键(也称为属性)，右边的东西是值。

你可以把`对象`想象为钥匙环上的钥匙。每一个都是针对特定的门，如果你的钥匙上有漂亮的标签，你就能快速打开门。事实上，左边的东西被称为`键`(也称为`属性`)，右边的东西是`值`。

```js
// an object with a single key 'name' and single value 'bill'
{ name: 'bill' }
```

如果你能把数据放到对象中，你为什么要使用数组呢?因为对象不记得你设置的键的顺序，你可能会进入这样一个对象:

```js
{ date: "10/20/2012", diary: "slept a bit today", name: "Charles" }
```

但是电脑可以把它还给你:

```js
{ diary: "slept a bit today", name: "Charles", date: "10/20/2012" }
```

还是这样!

```js
{ name: "Charles", diary: "slept a bit today", date: "10/20/2012" }
```

所以你不能相信对象中键的顺序。如果你想要变得非常漂亮，你可以做一个充满了对象的数组，或者一个充满数组的对象！

```js
var moodLog = [
  {
    date: "10/20/2012",
    mood: "catnipped"
  },
  {
    date: "10/21/2012",
    mood: "nonplussed"
  },
  {
    date: "10/22/2012",
    mood: "purring"
  }
]

// ordered from least to most favorite
var favorites = {
  treats: ["bird sighting", "belly rub", "catnip"],
  napSpots: ["couch", "planter box", "human face"]
}
```

当你把不同的东西组合在一起时，你就在制造`数据结构体`，就像乐高积木一样！

## Callbacks

Callbacks并不是JavaScript像对象或数组那样的特性，而是使用函数的一种特定方式。要理解Callbacks为什么是有用的，您首先必须了解异步(通常简称为异步)编程。根据定义，异步代码是以一种非同步的方式编写的。同步代码易于理解和编写。这里有一个例子来说明:

```js
var photo = download('http://foo-chan.com/images/sp.jpg')
uploadPhotoTweet(photo, '@maxogden')
```

这段同步的伪代码下载了一张可爱的猫照片，然后将照片上传到twitter，并在@maxogden上推文。很简单的!

这段代码是同步的，因为为了将照片上传至推文，照片下载必须完成。这意味着第2行不能运行，直到第1行中的任务完全完成。如果我们要实现这个伪代码下载我们想要确保“阻塞”执行,直到下载完成了,这意味着它将防止其他JavaScript执行,直到完成,当下载完成后,将un-block JavaScript执行和2号线将执行。

同步代码对于快速发生的事情很好，但是对于那些需要保存、加载、下载或上传的东西来说是很可怕的。如果你正在下载照片的服务器很慢，或者你正在使用的网络连接很慢，或者你正在运行的电脑上运行的电脑上有太多的youtube视频标签，运行速度很慢，怎么办?这意味着，在第2行开始运行之前，它可能需要等待几分钟的等待。同时，由于页面上的所有JavaScript都在下载过程中被阻止运行，网页会完全冻结，直到下载完成后才会出现响应。

应该不惜一切代价避免阻塞的执行，尤其是这样做会使程序冻结或变得无响应。让我们假设上面的照片需要一秒钟的时间下载。为了说明一秒钟对一台现代计算机有多长的时间，这里有一个程序来测试JavaScript在一秒内可以处理多少任务。

```js
function measureLoopSpeed() {
  var count = 0
  function addOne() { count = count + 1 }

  // Date.now() returns a big number representing the number of
  // milliseconds that have elapsed since Jan 01 1970
  var now = Date.now()

  // Loop until Date.now() is 1000 milliseconds (1 second) or more into
  // the future from when we started looping. On each loop, call addOne
  while (Date.now() - now < 1000) addOne()

  // Finally it has been >= 1000ms, so let's print out our total count
  console.log(count)
}

measureLoopSpeed()
```

将上面的代码复制到你的JavaScript控制台，一秒钟后它就会打印出一个数字。在我的电脑上，我得到了8527360，大约850万。在一秒内，JavaScript可以调用addOne函数850万次！因此，如果您有同步代码下载照片，而照片下载需要一秒，这意味着您可能会阻止850万操作的发生，而JavaScript执行被阻塞。

有些语言有一个名为`sleep`的函数，它可以阻塞执行数秒。例如，在使用`sleep`的Mac OS上运行Terminal.app的bash代码。当您运行命令`sleep 3 && echo 'done sleeping now`时。在打印出来之前，它会阻塞3秒。

![](http://jsforcats.com/images/bash-sleep.png)

JavaScript没有`sleep`函数。既然你是一只猫，你可能会问自己:“我为什么要学习一门不涉及睡眠的编程语言?”但留在我身边。与其依赖睡眠等待事情的发生，JavaScript的设计还鼓励使用函数。如果在执行任务B之前必须等待任务A完成，则将任务B的所有代码都放到一个函数中，在完成任务时只调用该函数。

例如，这是一种封闭风格的代码:

```js
a()
b()
```

这是一种非阻塞的风格:

```js
a(b)
```

在非阻塞版本中`b`是一个callback。在阻塞版本中a和b都是called/invoked(它们都有()后立即执行函数)。在非阻塞版本中，您将注意到只有`a`被调用，而`b`只是作为一个参数传递给`a`。

在阻塞版本中，`a`和`b`之间没有明确的关系，在非阻塞版本中，它变成了一项工作，做它需要做的事情，然后在完成时调用`b`。以这种方式使用函数称为回调函数，因为在这种情况下，回调函数在`a`完成后会被调用。

下面是一个伪代码的实现，它可能是这样的:

```js
function a(done) {
  download('https://pbs.twimg.com/media/B4DDWBrCEAA8u4O.jpg:large', function doneDownloading(error, png) {
    // handle error if there was one
    if (err) console.log('uh-oh!', error)

    // call done when you are all done
    done()
  })
}
```

回想一下我们的非阻塞例子a(b)，我们把a和b作为第一个参数。在上面的函数定义中，`done`参数是我们传入的b函数。这种行为一开始很难让你的头脑有个头绪。当你调用一个函数时，传入的参数在函数中不会有相同的变量名。在这种情况下，我们称之为b的函数是在函数内部完成的。但b和`done`只是变量名，指向相同的底层函数。通常回调函数被标记为`done`或回调，以明确它们是在当前函数为`done`时应该调用的函数。

所以，只要`a`和`b`在完成时，`a`和`b`都被调用在非阻塞和阻塞的两个版本中。不同之处在于，在非阻塞版本中，我们不需要停止JavaScript的执行。一般来说，非阻塞样式是指您编写每个函数的地方，以便它能够尽快返回，而不会阻塞。

为了进一步说明这个问题，如果你需要一秒钟的时间来完成，并且你使用了阻塞版本，那意味着你只能做一件事。如果你使用非阻塞版本(又称使用回调)，你可以在同一秒内完成数百万件其他事情，这意味着你可以更快地完成你的工作，并在剩下的时间里睡觉。

> 记住:编程完全是关于懒惰的，你应该是一个睡觉的人，而不是你的电脑。

希望您现在可以看到，回调函数只是在一些异步任务之后调用其他函数的函数。异步任务的常见例子有:读取照片、下载歌曲、上传图片、与数据库对话、等待用户按下键或点击某人等，任何需要时间的事情。JavaScript非常擅长处理像这样的异步任务，只要您花时间学习如何使用回调，并防止JavaScript被阻塞。

____

## 结束!

这只是您与JavaScript的关系的开始！你不可能一下子就学会所有的东西，但是你应该找到适合你的东西，并尝试在这里学习所有的概念。

我建议明天再来一遍，从一开始就把整件事再做一遍！在你得到所有东西之前，可能需要几次(编程是困难的)。试着避免在任何包含闪亮对象的房间里阅读这一页。它们会让人难以置信的分心。

你还想知道另一个话题吗?在[github](http://github.com/maxogden/javascript-for-cats)上为它打开一个问题。

## 推荐阅读

对于猫来说，JavaScript跳过了很多不重要的细节，这些细节并不重要(猫的注意力不集中)，但如果你觉得你需要深入研究一下，就可以查看以下内容:

- [NodeSchool.io](http://nodeschool.io/)是一种社区驱动的、开源的教育软件，它以一种交互的、自导向的方式教授各种web开发技能。我帮助NodeSchool !可悲的是，它比这一页更少的猫。
- [Eloquent Javascript](http://eloquentjavascript.net/)是一本免费的书，教你Javascript！很好!特别是关于值、变量和控制流的章节
- [Mozilla的JavaScript指南](https://developer.mozilla.org/en-US/docs/JavaScript/Guide)也有一个很可爱的入门章节叫做值，变量和文字
- [标准JS风格指南](https://github.com/feross/standard)是我使用的JS风格的“零配置”linter
- [让我们来写@shama的代码](https://github.com/shama/letswritecode)这是我的一个朋友制作的很棒的YouTube编码教程
