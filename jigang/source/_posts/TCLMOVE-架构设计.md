---
title: TCLMOVE 架构设计
date: 2017-10-19 19:49:52
tags:
- iOS
- 架构设计
- TCLMOVE
categories:
- iOS

---

![TCLMOVE](http://oxwfu3w0v.bkt.clouddn.com/2017/10/20/tclmove1.png)

`架构`，就是对`软件复杂度`的`管理`。

<!-- more -->

**层次划分**

**三层结构框架**

* *UI Layer*
* *Logic Layer*
* *Persistence Layer*

![TCLMOVE框架图](http://oxwfu3w0v.bkt.clouddn.com/2017/10/18/mt30_tclmove_ios.svg)

### UI Layer 展现层

UI Layer 采用MVVM框架。

***View***

V对应View，责任只负责显示，不保存任何状态，保证View的`无状态`性，`states` 只能来源于外部，同样的`states`设置，View 显示显示内容一样。

View最主要的是实现组件化，与其它组件和模块保持最大化的隔离，对内的接口只有`states`,对外的接口只有`events`.

**比如** 基于Rx的方式 :

```
view.rx.states
view.rx.events.click
```

View的复杂度在于自身的实现(复杂的绘制／动画)，以及Views之间的组合，组件之间的最大化的隔离是防止复杂度的膨胀。

***ViewModel***

VM对应ViewModel, 负责绑定数据和Views。MVVM的机制是VM实现V和M的数据绑定，双向绑定或单向绑定，由于要实现View的无状态，在此只作单向的数据绑定，数据来源于M，单向流于view的states。

ViewModel对应一个ViewController,此Scene中所有的views状态设置都集中在其中，`统一处理`，避免代码分散在ViewController各处，造成逻辑混乱；views的enents也是经由此地，产生Actions.

ViewModel会分担大部分ViewController上的逻辑代码，但也只是UI层面的，主要工作是states和events的分发和中间状态的转化，一些简单的限制判断和states的组合／分解，仅此而已。

***Model***

M对应Model，UI Layer以下对应MVVM的Model.

V和VM负责展现层上的部分，作到无状态化。而所有的状态都在Model之中，状态改变UI呈现改变。


### Logic Layer 业务层

Logic Layer会根据业务划分出许多功能模块，业务层应该最先进入详细设计，对比需求分析有效的分离出独立的功能模块。

业务层主要有三个角色：

* **Manager** `管理者`
* **Worker Protocl** `工作协议`
* **Worker** `工作者`

以下我们将以公司组织结构类比的方式介绍业务层和三个角色。

业务层会根据业务需求划分出相应的功能模块，就像公司的各个不同的职能部门，各自有自己的分工与协作。

各部门组织成员都有`Manager`和`Worker`，有自己的职责。

`Manager` 主要职责是 管理Workers, 分发任务， 协调任务， 与上层交互。`Manager` 不会处理具体的工作，具体工作由`Worker`来实现。

`Worker`只关心一个单独的任务，其它的事情不涉及，而且只对他自己的`Manager`负责。`Worker`所需要的资源和工具来源于下游。

工作任务由`Manager`下派，通过工作清单即`Worker Protocl`。`Worker Protocl`是工作规范（即接口也叫协议），指明了需要实现的操作。而能够完成这些规范的`Workers`，不只一个，比如经理(`Manager`)需要存储一份文档，一个小秘(`Worker`)说她可以存储在本地，一个码农(`Worker`)说他可以上传存储到服务器，至于使用那个`Worker`由`Manager`决定。

部门之间的协作也是由`Manager`完成，部门经理会向其它部门获取结果。如果一项工作需要几个部门协作完成，可以产生一个更高级的`Manager`，管理这几个部门的`Managers`。

### Persistence Layer 持久层

持久层是对接基础设施，资源的出入口，也可以叫资源层。

其主要包括：

* HTTP Client
* 数据库
* BLE
* 本地文件
* 资源文件
* ...

**HTTP Client**

服务器端采用HTTP/RESTful，移动端对应采用API层和Client层。

* `Client层` 对应HTTP客户端功能，完成HTTP网络请求功能。
* `API层` 对应Rest，完成接口定义，让上层通过接口直接操作资源。

还需要完成本地缓存功能，配合服务端的HTTP缓存机制，和自己的离线缓存。

**数据库**

数据库实现DAO（数据访问对象），统一数据库的数据操作标准。

数据库采用ORM(对象关系映射)框架，屏蔽SQL的操作步骤。

Change Events,当数据库数据改变时，产生相应的通知事件。

**BLE**

**File Manage**

**Resource Manage**

### 三层之间的桥梁

三层结构`UI Layer`，`Logic Layer`，`Persistence Layer`如上所述，各自有各自的职责，彼此之间也是隔离的，这样就可以分配给不同的开发人员，独立的单元测试。而它们之间需求建立沟通的桥梁。

### App States

`App States`的思想来源与Flux架构，实现它的有前端的Redux。【[可参考 Redux](http://www.redux.org.cn)】

`App States`是一个状态管理器，如上所述UI层是无状态的，而它的状态来源是`App States`，并且作状态的统一管理。

`App States`保存了所有的状态，通过View Model绑定View，当状态发生变化时相应的View会发生改变。

这里最大的难度是`states`。归纳`states`需要即要`全面`也要`精准`,那些值可以是`state`那些不是请慎重考虑。比如业务数据中的小孩子信息是一定在`states`中，还有像 _正在获取销毁信息的状态_ 也是一个布尔的`state`，但像_小孩子的数目_这种值，已经可以从组织size中知道，就不应该在`states`，这些值会照成`states`冗余的，使得管理复杂化。

然后，记住以下几点：
> `State` 是只读的
>
> 惟一改变 `State` 的方法就是触发 `action`，通过 `reducers` 改变 `states`
>
> `states`的结构尽量扁平化，不要有太多的嵌套。
>

### Repository

`Repository`采用外观模式。

`Repository`对接Persistence Layer，提供一组一致的接口，定义一个高层接口，上层不关心数据来源于那里（那个网络服务器或蓝牙或数据库），它只关心它要的数据。
