# fluid容器运行时
fluid容器是使用Fluid Framework创建任何东西的基本概念。所有样品液
应用程序使用Fluid容器来管理用户体验，应用程序逻辑和应用程序状态。

但是，Fluid容器不是独立的应用程序。fluid容器是_code-plus-data package_。一种
容器必须由fluid加载器加载并连接到fluid服务。

因为容器是这样一个核心概念，所以我们将从几个不同的角度来研究它们。

##容器与运行时

Fluid容器是实例化的容器JavaScript对象，但它也是容器的定义。我们
可互换地使用“容器”来引用可以创建新对象的类以及实例化的对象本身。

“ ContainerRuntime”是指fluid容器的内部机制。作为开发人员，您将与
通过运行时方法公开运行时，这些方法公开了实例化容器对象的有用属性。

##什么是fluid容器？

fluid容器是代码加数据包。一个容器至少包含一个用于应用逻辑的Fluid对象，但是
通常，将多个Fluid对象组合在一起以创建整体体验。

从fluid服务的角度来看，容器是fluid的原子单位。该服务什么都不知道
fluid容器内部。

话虽如此，应用逻辑是由Fluid对象处理的，而状态是由内部的分布式数据结构处理的
fluid对象。

##fluid容器有什么作用？

fluid容器与[过程和分配操作]（./主机）交互，管理[fluid的生命周期
对象]（./ dataobject-aqueduct），并提供用于访问Fluid对象的请求API。

###处理和分发操作

当fluid加载器解析fluid容器时，它将一组服务驱动程序传递给该容器。这些司机
是** DeltaConnection **，** DeltaStorageService **和** DocumentStorageService **。

Fluid容器包含用于处理DeltaConnection中的操作的代码，以弥补错过的操作
使用DeltaStorageService，并从DocumentStorageService创建或获取摘要。这些都是
很重要，但最关键的是运算处理。

fluid容器负责将操作传递给相关的分布式数据结构和fluid对象。

###管理Fluid对象生命周期

容器提供了一个createDataStore方法来创建新的数据存储。容器负责
实例化Fluid对象并创建使其他连接的客户端知道新Fluid的操作
目的。

###使用Fluid容器：Request API

fluid容器通过请求范例进行交互。当渡槽创建默认请求处理程序时
返回默认的Fluid对象，请求范例是一种强大的模式，可让开发人员创建自定义
逻辑。

若要检索默认数据存储，可以对容器执行请求。类似于[loaders API]（./ hosts.md）
这将返回状态码和默认数据存储。