
## Aqueduct

@fluidframework/aqueduct软件包是一个用于在Fluid中构建Fluid对象和Fluid容器的库
框架.其目标是在现有的Fluid Framework接口上提供薄的基础层,从而使开发人员
快速入门. 

[了解更多](https://fluidframework.com/apis/aqueduct/)

##代码加载器(Code loader)

如果你的应用与你的容器是分开的,则Fluid可以使用代码加载器下载
并动态加载容器代码包.

## 容器

容器是您的应用程序到Fluid Framework的入口点.它运行您的容器代码,并且您将通过其检索数据对象.

## 容器代码

您将编写容器代码来定义你设想的Fluid对象以及如何访问它们.

## DataObject

aqueduct对Fluid对象的实现.旨在从你的设想出发,为DDS进行有意义的语义
分组,以及为数据提供API表面.

##分布式数据结构(Distributed data structures (DDSes))

DDS是Fluid Framework提供的用于存储协作数据的数据结构.当协作者修改数据时,
更改将反映给所有其他协作者.

##fluid装载机(Fluid loader)

负责连接到fluid服务并加载fluid容器.

##fluid对象(Fluid object)

任何实现Fluid特征接口的JavaScript对象.

##fluid服务(Fluid service)

一个服务端点,负责接收,处理,存储和广播操作.

##fluid服务驱动程序(Fluid service driver)

负责连接到Fluid服务的客户代码.

## URL解析器

Fluid的API~~表面~~接口~~利用~~使用URLs,例如在"Loader"的"resolve()"方法和"Container"的"request()"中
方法. URL解析器用于解释这些URL,以便与Fluid服务一起使用.