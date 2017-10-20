---
title: PlantUML
date: 2017-10-17 14:34:26
tags:
- PlantUML
- Atom
- idea
- uml
---

> PlantUML是一个开源项目，支持快速绘制：
>
> * `时序图` Sequence diagram
> * `用例图` Usecase diagram
> * `类图` Class diagram
> * `活动图` Activity diagram
> * `组件图` Component diagram
> * `状态图` State diagram
> * `部署图` Deployment diagram
> * `对象图` Object diagram
> * `线框图形界面` wireframe graphical interface
> * `时序图` Timing Diagram

___

_使用方式请参考如下文档：_

[___官网___](http://plantuml.com)

[___使用 PlantUML 绘制的 UML___](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/PlantUML.pdf)


### 安装 ###

PlantUML 显示依赖 `Graphviz` , 需要先安装 Graphviz

Mac 安装 Graphviz

```
 brew install Graphviz
```

Ubuntu 安装 Graphviz

```
apt-get install graphviz
```

Windows 下使用choco来安装Graphviz

* 在cmd中安装chocolatey

```
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "iex ((new-object net.webclient).DownloadString('http://bit.ly/psChocInstall'))" && SET PATH=%PATH%;%systemdrive%\chocolatey\bin
```

* 在cmd中安装Graphviz

```
choco install Graphviz
```

#### 在Atom中安装

安装两个Package : `language-plantuml`、 `plantuml-preview`

![language-plantuml](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/atom_plantuml.png)

![plantuml-preview](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/atom_plantuml_2.png)

#### 在IDE中安装

安装插件 `PlantUML integration`

![InteIIij_PlantUML](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/InteIIij_PlantUML.png)

### 使用 ###

#### 在Atom中使用

* 新建文件，保存为.pu文件

```
@startuml

abstract class AbstractList
abstract AbstractCollection
interface List
interface Collection

List <|-- AbstractList
Collection <|-- AbstractCollection

Collection <|- List
AbstractCollection <|- AbstractList
AbstractList <|-- ArrayList

class ArrayList {
  Object[] elementData
  size()
}

enum TimeUnit {
  DAYS
  HOURS
  MINUTES
}

@enduml
```

* 打开 plantuml-preview

![plantuml-preview](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/atom_plantuml_open_preview.png)

__如果出现下列错误情况，是需要指定`plantuml.jar`__

![error](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/atom_plantuml_open_preview_errow.png)

plantuml.jar 可以通过[PlantUML官网下载](http://zh.plantuml.com/download.html)

* 设置 plantuml-preview 的 plantuml.jar

	![plantuml.jar](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/atom_plantuml_open_preview_jar.png)

* 对应的UML图如下所示

	![ok](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/atom_plantuml_open_preview_ok.png)

#### 在IDE中使用

通过File菜单打开：

![ide_plantuml](http://oxwfu3w0v.bkt.clouddn.com/2017/10/17/ide_plantuml_open.png)


----

### 示例效果展示

**效果如下：**

![TCLMOVE框架图](http://oxwfu3w0v.bkt.clouddn.com/2017/10/18/mt30_tclmove_ios.svg)

以下为一个框架图 PlantUML code:

```
@startuml

title TCLMOVE 架构图

skinparam backgroundColor #DBDBD8
skinparam handwritten true

skinparam package {
  backgroundColor<<Scenes>> #7BD2F7
  borderColor<<Scenes>> #90D2F7
  fontSize<<Scenes>> 13

  backgroundColor<<Servers>> #DBEAD0
  borderColor<<Servers>> #DBEAD0

  backgroundColor<<Models>> #FEFECE
  borderColor<<Models>> #FEFEAE
}

skinparam folder {
  backgroundColor<<Managers>> #BBDAC0
  borderColor<<Managers>> #DBEAD0

  backgroundColor<<Workers>> #DBEAA0
  borderColor<<Workers>> #DBEAD0
}

skinparam interface {
  backgroundColor RosyBrown
  borderColor orange
}

skinparam entity {
  backgroundColor RosyBrown
  borderColor orange
}

skinparam cloud {
  backgroundColor #00
  borderColor #00
}

skinparam database {
  backgroundColor #B1BFC3
  borderColor #B1BFE3
}

skinparam component {
  backgroundColor #5ABAA7
  borderColor #7ABAA7
}

skinparam node {
  backgroundColor #E68274
  borderColor #E68274
}

package "UI Layer" <<Scenes>> {

  [View1] as v1
  [ViewModel1] as vm1
  v1 -(0)- vm1
  [View2] as v2
  [ViewModel2] as vm2
  v2 -(0)- vm2
  [View3] as v3
  [ViewModel3] as vm3
  v3 -(0)- vm3
  component "View" as v
  component "ViewModel" as vm
  v -(0)- vm
}

node "State" {

  component "App State" as state
  component "Reducer" as reducer
  component "Action" as action
  component "Action Creator" as creator

  state <- reducer
  reducer <-- action
  creator -> action

}

vm <... state
vm1 <.. state
vm2 <.. state
vm3 <.. state

package "Logic Layer" <<Servers>> {

folder Managers <<Managers>> {

  [Account Manager] as am
  [Device Manager] as dm
  [Location Manager] as lm
  [Message Manager] as mm
  [Notification Manager] as nm
  [... Manager] as om
}

creator <-- am
creator <-- dm
creator <-- lm
creator <-- mm
creator <- nm
creator <- om

folder Workers <<Workers>> as workers {
  interface "Account Worker Protocl" as awp
  [Account Worker] as aw
  awp ^-- aw
  interface "Device Worker Protocl" as dwp
  [Device Worker] as dw
  dwp ^-- dw
  interface "Location Worker Protocl" as lwp
  [Location Worker] as lw
  lwp ^-- lw
  interface "Message Worker Protocl" as mwp
  [Message Worker] as mw
  mwp ^-- mw
  interface "Notification Worker Protocl" as nwp
  [Notification Worker] as nw
  nwp ^-- nw
  interface "... Worker Protocl" as owp
  [... Worker] as ow
  owp ^-- ow
}

am --> awp
dm --> dwp
lm --> lwp
mm --> mwp
nm --> nwp
om --> owp

}

node "Repository" as repository

skinparam rectangle {
    roundCorner<<API>> 25
    roundCorner<<BLE>> 8
    roundCorner<<DAO>> 12
    roundCorner<<...>> 35

    backgroundColor<<API>> #C4DADF
    borderColor<<API>> #E0DADF

    backgroundColor<<BLE>> #C4DFDA
    borderColor<<BLE>> #E4DFDA

    backgroundColor<<DAO>> #DFC4DA
    borderColor<<DAO>> #DFC4EA

    backgroundColor #AFC4DA
    borderColor #AFC4EA
}

package "Persistence Layer" <<Models>> as models {

  rectangle "... Managements" <<...>> {
	   rectangle "..." <<...>>
  }

  rectangle "BLE Managements" <<BLE>> as blem {
	   rectangle "BLE attributes" <<BLE>> as ble
     rectangle "Local Storage"
     rectangle "Events"
  }

  rectangle "DAO Managements" <<DAO>> as daom {
	   rectangle "DAO" <<DAO>> as dao
     rectangle "Change Events"
  }

  rectangle "API Managements" <<API>> as apim {
     rectangle "Resource APIs" <<API>> as rapis
     rectangle "Local Cache"
  }


  [Http Client] as httpclient
  [database orm] as orm
  [BLE 外设] as blew

  rapis <-- httpclient
  dao <-- orm
  ble <-- blew
}

aw --> repository
dw --> repository
lw --> repository
mw --> repository
nw --> repository
ow --> repository

repository --> apim
repository --> daom
repository --> blem

cloud Cloud as cloud
database "database" as database
entity "BLE" as ble_entity

httpclient --> cloud
orm --> database
blew --> ble_entity

@enduml

```
