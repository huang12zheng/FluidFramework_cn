# 介绍分布式数据结构
Fluid Framework为开发人员提供了_分布式数据结构_(DDSes),它们可以自动确保每个客户端可以访问相同的状态.我们称它们为"分布式数据结构",因为它们与编程时常用的数据结构相似,例如strings, maps/dictionaries, and sequences/lists.API提供的DDS旨在使使用过此类数据结构的程序员熟悉.例如,
[SharedMap]就是DDS用于键/值对存储,例如典型的map 和 dictionary数据结构,并提供`get`
和`set`方法来存储和检索地图中的数据.

使用DDS时,您可以在很大程度上将其视为本地对象.您可以向其中添加数据,删除数据,更新数据等.
但是,DDS并非只是本地对象.其他正在编辑的用户也可以更改DDS.

>按照惯例,大多数分布式数据结构都以"Shared"为前缀. _SharedMap _,_SharedMatrix _,_SharedString_,
此前缀表示对象在多个客户端之间共享.

任何客户端更改DDS时,都会在本地引发一个[event](#events).您的代码可以监听这些事件,以便
您知道何时更改数据并可以做出适当的反应.例如,在以下情况下,您可能需要重新计算派生值
DDS中的某些数据发生了变化.

## 合并行为

了解如何合并来自多个客户端的更改,在类似Fluid的分布式系统中是至关重要的.
了解合并逻辑可以使您在用户协作数据时"保留用户意图".这意味着
合并行为应符合用户在编辑数据时的意图或期望.

在Fluid中,合并行为由DDS定义.合并策略被像SharedMap这样分布式键值所使用,最简单的合并策略是_last writer wins_(LWW)
的结构.使用这种合并策略,当多个客户端编写不同的内容时相同键的值,最后写入的值将覆盖其他键.请参阅
有关每个DDS的`API文档`,以获取有关其使用的合并策略的更多详细信息.

## 性能特点

FluidDDS根据其与Fluid服务的交互方式表现出不同的性能特征. DDS
通常分为两大类:_optimistic_和_consensus-based_.

> * [Fluid框架架构](./architecture.md)
> * [Fluid服务](./service.md)

### 乐观数据结构

乐观的DDS是能够 在Fluid服务进行排序之前 就应用Fluid运算的.本地的更改被称为_optimistically_,因此名称为_optimistic DDSes_. DDS以一致的方式进行时,也适用于远程运算(remote operations).

许多最常用的DDS都很乐观,包括SharedMap,SharedSequence,SharedMatrix和SharedString.

### 基于共识的数据结构

基于共识的DDS与乐观的DDS不同,因为它们(基于共识的DDS)在应用运算之前,需要等待Fluid服务的确认(甚至包括本地运算).
~~在你需要使用原子性时,这些数据结构提供了额外的保证行为.或者,这些数据结构提供了同步行为.~~
这些数据结构提供了同步行为.或者,在你需要使用原子性时,这些数据结构提供了额外的保证行为.

这些行为保证不能以乐观的方式实施.这样的成本就是性能；乐观的DDS是
使Fluid如此快速的部分原因,因此几乎总是首选使用乐观的DDS,但是您可以使用性能(performance) 来进行保证行为的 交换(trade).(but you can trade performance for behavioral guarantees
但是您可以为了保证行为来牺牲性能)

#### 为什么基于共识的DDS有用

> 如果您刚开始使用Fluid,则无需阅读本节.随时跳过它,稍后再返回.

要了解为什么基于共识的DDS有用,请考虑实现堆栈DDS.据我所知,作为一种乐观的方法来实现堆栈DDS这是不可能的.例如,在基于ops的Fluid架构中,可以定义一种运算类似于"pop",这个运算是当客户端在op流中看到该运算符时,它将从其本地堆栈对象中pop一个值.

使用乐观的DDS的条件下,在服务器知道本地运算符之前,客户端甚至可以~~没有等待的~~直接完成本地运算.
想象一下,客户端A 进行pop运算,客户端B在它看到客户端A的远程pop运算完成之前,就也进行了一个pop运算. 这导致了分歧--
客户端A从本地堆栈pop一个值,而客户端B本应弹出(pop)第二个值,却弹出(pop)与A相同的值.我们期望分布式堆栈能确保`pop`运算运行正确(以及与此相关的任何其他运算),以使客户端达到最终一致的状态.乐观实现方式的结果与我们刚刚描述的期望相违背.

基于共识的DDS不会乐观地进行本地运算.而是,等到服务器完成了之前运算,才进行本地运算(these DDSes wait for the server to apply a sequence number to the operation before applying it locally).
通过这种方法,当两个客户端pop时,两个都不会进行
本地更改,直到它们从服务器取回顺序操作.完成后,他们会依次应用操作
导致所有远程客户端的行为一致.

## 创建和存储分布式数据结构

分布式数据结构对象是使用其类型的静态"create"方法创建的.

```ts
const myMap = SharedMap.create(this.runtime);
```

您必须传入由DDS管理的`IFluidDataStoreRuntime`.我们将在之后,在[使用DataObject封装数据](./dataobject-aqueduct.md)部分详细介绍运行时.

### 将DDS存储在另一个DDS中

分布式数据结构可以存储数字和字符串之类的原始值,以及_JSON serializable_对象.对于例如DDS这样,
无法JSON序列化的对象,Fluid提供了一种称为_handles_的机制,该机制是可序列化.

在其他DDS中存储DDS时,必须存储其句柄,而不是DDS本身.例如,考虑以下代码:

```ts
//Create a new map for our Fluid data
const myMap = SharedMap.create(this.runtime);

//Create a new counter
const myCounter = SharedCounter.create(this.runtime);

//Store the handle in the map
myMap.set("counter", myCounter.handle);
```

这就是您需要了解的有关 有效地使用DDS句柄的所有信息.如果您想了解更多有关手柄的信息,请参阅"高级"部分中的[Fluid句柄](../advanced/handles.md).

## 事件

当Fluid运行时要 更改分布式数据结构时,它将发出事件.你可以监听这些事件,以便您可以知道何时远程客户端更改了数据,并且可以做出适当的反应.例如,您可能需要
当DDS中的某些数据更改时,重新计算派生值.

```ts
myMap.on("valueChanged", () => {
  recalculate();
});
```

关于事件(每个DDS发出的)的更多的详细信息,请参阅后面的部分.

## 选择正确的数据结构

由于分布式数据结构可以相互存储,因此可以
将DDS与创建 协作数据模型 相结合.以下两个问题可以帮助确定用于协作数据模型的最佳数据结构.

* 我的方案(场景)需要的_粒度_是什么？
* 分布式数据结构的合并行为对此(this)有何影响？

如果在您的场景中,用户需要分别进行哪些编辑？例如,假设您正在存储有关几何的数据,
因为您正在构建协作式编辑工具.
您可以存储形状的坐标,长度,宽度等

用户编辑此数据时,可以同时编辑哪些数据？这是一个重要的问题,
因为它会影响您在DDS中构造的数据结构.

让我们暂时假设,关于形状的所有数据都存储为一个看起来像这样的对象:

```json
{
  "x": 0,
  "y": 0,
  "height": 60,
  "width": 40
}
```

如果我们想使用Fluid来使这些数据协作,最直接的方法(但最终有缺陷)是将shape对象存储在SharedMap中.我们的SharedMap看起来像这样:

```json
{
  "aShape": {
    "x": 0,
    "y": 0,
    "height": 60,
    "width": 40
  }
}
```

回想一下[SharedMap使用的是最后胜出合并策略](#merge-behavior).这意味着如果两个用户是
同时编辑数据,那么进行最新更改的用户将覆盖其他用户.

想象一下,您正在与同事协作,并且在您更改同事的形状时更改形状的宽度
形状的高度.这将生成两个操作:为您执行一次set操作,为您的同事进行另一次set操作.这两个操作将由Fluid服务排序,但是只有一个操作会"获胜",因为SharedMap的合并行为是LWW.由于我们将shape存储为对象,因此两个"设置"操作都设置了整个对象.

从用户的角度来看,这导致(最后的结果是)某人的更改"丢失".这可能完全符合您的需求.
但是,如果您的场景要求用户能 编辑shape的各个属性,则使用SharedMap LWW合并策略可能会给您带来不想要的结果.

但是,您可以通过将单个(形状)属性存储在SharedMap键中来解决此问题.~~而不是~~相比存储在一个包含所有数据的JSON对象中,您可以将其进行分解.比如,将长度存储在一个SharedMap键中,将宽度存储在另一个SharedMap键中,使用此数据模型,用户可以更改形状的各个属性,而不会覆盖其他用户的更改.

您的数据模型中可能有多个形状,因此您可以创建一个SharedMap来存储所有形状,然后将表示每个形状的SharedMap存储在该SharedMap中.

要了解更多有关如何使用DDS创建更复杂的Fluid对象的信息,请参见[使用DataObject封装数据](./dataobject-aqueduct.md)部分.

### 键值对数据

这些DDS用于存储键值数据. 他们都是乐观模式的,并采用了最后作者胜出的合并政策.

- SharedMap-基本的键值分布式数据结构.
- SharedDirectory-具有API的SharedMap更适合分层数据.
- SharedCell-一个"单对象SharedMap"； 用于包装对象.

### 序列

这些DDS用于存储顺序数据. 他们都很乐观.
- SharedNumberSequence –分布式数字序列.
- SharedObjectSequence –对象的分布式序列.
- SharedMatrix –一种可有效使用二维表格数据的分布式数据结构,.
### 专用数据结构
- SharedCounter –分布式计数器.
- SharedString –用于处理协作文本的专用数据结构.
- ink–墨水数据的专用数据结构