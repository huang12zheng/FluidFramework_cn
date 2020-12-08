# 容器 与 容器运行时
对于用Fluid Framework的创建的任何东西,fluid容器都是一个基本概念.
应用程序使用Fluid容器来管理用户体验,应用程序逻辑和应用程序状态.

但是,Fluid容器不是独立的应用程序.fluid容器是_code-plus-data package_.一种
容器必须由fluid加载器加载并连接到fluid服务.

因为容器是这样重要的一个核心概念,所以我们将从几个不同的角度来研究它们.

## 容器与 运行时 比较
`
译者注:
Fluid容器 既是容器,又是实例化的JsO
`
Fluid容器是实例化的容器(的)JavaScript对象,同时它也是容器的定义.我们可交替使用(这两种方式),无论是 以类的方式使用容器(类可以实例化的对象本身),还是实例化的对象(??)本身.

"ContainerRuntime"是指fluid容器的内部机制.作为开发人员,您通过运行时方法来与运行时交互,这些方法公开了实例化容器对象的有用属性.

## 什么是fluid容器？

fluid容器是代码加数据的包.一个容器至少包含一个用于应用逻辑的Fluid对象,但是
通常,将多个Fluid对象组合在一起以创建整体体验.

从fluid服务的角度来看,容器是fluid的原子单位.该服务对fluid容器内部,什么都不知道.

因为如此,应用逻辑是由Fluid对象处理的,而状态是由fluid对象内部的分布式数据结构处理的.

## fluid容器有什么作用？

fluid容器与[进程和分配操作](./hosts.md)交互,来管理[fluid的生命周期
对象](./dataobject-aqueduct.md),并提供用于访问Fluid对象的API请求.

### 处理和分发操作

当fluid加载器解析fluid容器时,它将一组服务驱动程序传递给该容器.这些驱动
是** DeltaConnection**,** DeltaStorageService**和** DocumentStorageService**.

Fluid容器包含代码来用DeltaConnection处理操作,来使用DeltaStorageService弥补错过的操作
,并用DocumentStorageService创建或获取摘要.这些都是
很重要,但最关键的是**运算处理**.

fluid容器负责将操作传递给相关的分布式数据结构和fluid对象.

### 管理Fluid对象生命周期

容器提供了一个createDataStore方法来创建新的数据存储.容器负责
实例化Fluid对象并
创建一个操作,来使其他连接的客户端 知道有新的fluid对象.

### 使用Fluid容器:Request API

fluid容器通过请求范例 进行交互.当aqueduct创建默认请求处理程序时,
返回默认的Fluid对象,请求范例是一种强大的模式,可让开发人员创建自定义
逻辑.

若要检索默认数据存储,可以对容器执行请求.类似于[loaders API](./hosts.md),它将返回状态码和默认数据存储(data store).
```ts
container.request({url: "/"})
```