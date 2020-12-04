# 宿主
Fluid加载器是Fluid Framework的关键部分之一.开发人员在应用程序内部使用fluid加载器来 
加载fluid容器并 启动与fluid服务之间的通信.

"fluid宿主"是 使用fluid加载器加载fluid容器的 任何应用程序.

fluid加载器使用插件模型.


## 谁需要fluid装载机？

如果您的应用程序或网站将加载Fluid容器,那么您则需要使用
fluid装载机来创建Fluid宿主,！

如果您要构建Fluid容器,并且不会使用Fluid来构建独立的应用程序,则可能你~~有兴趣~~(需要)了解fluid加载器.fluid加载器包括容器使用使用的~~功能~~(能力),例如宿主示波器(host scopes).

您可能还希望将Fluid容器作为 一个独立的网站 的宿主
(host your Fluid container on a standalone website)


## 摘要

fluid加载器加载fluid容器,是通过连接到fluid服务来 获取fluid容器代码(来实现的).从一个从系统架构的角度来看,fluid加载器位于fluid服务和fluid容器之间.

！[fluid框架系统架构图](https://fluidframework.com/docs/concepts/images/architecture.png)

fluid加载器的用途非常广泛.为了保持通用性,加载器使用插件模型.随着
正确的插件(驱动程序,处理程序,解析器),Fluid loader将适用于任何有线协议和任何服务的实现.

加载器模仿现有的Web协议.类似于浏览器从网站请求状态和应用逻辑(网站)的方式
Web服务器上,Fluid宿主使用加载器从Fluid服务请求[Fluid容器](./containers-runtime.md).

## 宿主职责

Fluid宿主使用
URL解析器,Fluid服务驱动程序和代码加载器 来 创建Fluid加载器.宿主从加载器上请求fluid容器.最后,宿主对Fluid容器进行了某些操作.宿主可以从加载器中请求多个容器.

！[加载器的架构和请求流程](images/load-flow.png)

在下一部分中,我们将讨论这些部分中的每一个,从请求和加载器的依赖关系开始.

## 加载容器:按类分类

让我们解决fluid加载器各部分的作用,并深入了解一些细节.

### 请求

一个请求包括 fluid容器URL 和 可选的标头信息 .该URL包含协议和其他
URL解析器将解析的信息,以识别容器的位置.

这不是实例化加载器的一部分.该请求将从装载容器的过程开始(The request kicks of the process of loading a container).

### URL解析器

URL解析器解析一个请求并返回一个"IFluidResolvedUrl".该对象包括所有端点 和 fluid服务驱动访问容器需要的 令牌.

IFluidResolvedUrl示例包括以下信息.

```ts
const resolvedUrl: IFluidResolvedUrl = {
    endpoints: {
        deltaStorageUrl: "www.ContosoFluidService.com/deltaStorage",
        ordererUrl: "www.ContosoFluidService.com/orderer",
        storageUrl: "www.ContosoFluidService.com/storage",
    },
    tokens: { jwt: "token"},
    type: "fluid",
    url: "fluid://www.ContosoFluidService.com/ContosoTenant/documentIdentifier",
}
```

您可能会注意到,我们正在模仿浏览器在加载网页时执行的DNS和协议查找.那是因为
装载机可以访问 存储在多个Fluid服务上的 容器.此外,每个fluid服务都可以处理 不同的API和协议.

### fluid服务驱动程序工厂(DocumentServiceFactory)

加载器使用Fluid Service驱动程序连接到Fluid Service.

尽管许多开发人员一次只加载一个容器,但是考虑如何加载 存储在不同Fluid服务上 的 两个容器 会很有趣. 为了跟踪服务,加载器 使用URL解析器中的协议,以识别用于Fluid服务的正确Fluid服务驱动.

### 代码加载器

加载器使用 代码加载器 来获取容器代码.因为Fluid容器是应用的逻辑和分布式的状态
我们需要所有连接的客户就相同的容器代码达成共识.

### 范围(Scopes)

Scopes允许容器 通过宿主上 访问资源.例如,宿主可以访问 容器代码 无法访问的 授权上下文(因为容器代码无法被信任).宿主可以为联合容器提供Scopes访问安全资源.

## 处理回应

fluid加载器将从请求中返回响应对象.这是我们网络协议~~隐喻的延续~~隐含的,
您将收到一个带有mimeType(例如""fluid/object"),响应状态(例如200)和值(例如Fluid object).

宿主负责检查此响应是否有效.装载机返回了200吗？ mimeType是否正确？
随着Fluid Framework的扩展,我们打算进一步利用这些响应.