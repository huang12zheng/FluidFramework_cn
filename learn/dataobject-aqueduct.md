# 用DataObject封装数据
在上一节中,我们介绍了分布式数据结构并演示了如何使用它们.现在我们来讨论如何将那些分布式数据结构与自定义代码(业务逻辑)结合起来以创建模块化的可重用片段.


## @fluidframework/aqueduct软件包

Aqueduct库在核心Fluid Framework接口上提供了一个薄层,旨在帮助开发人员快速入门fluid开发.

您不必使用Aqueduct库.它是在Fluid框架基础之上构建的抽象层的示例,它专注于简化Fluid开发,因此,您可以选择直接使用Fluid,而不是它.

话虽如此,如果您是Fluid的新手,我们认为使用它会比不使用它更有效.


## DataObject

`DataObject`是包含`SharedDirectory`和任务管理器的基类.它确保两者都被创建并准备好在您的DataObject子类中进行访问.
```ts
import { DataObject, DataObjectFactory } from "@fluidframework/aqueduct";

class MyDataObject extends DataObject implements IFluidHTMLView { }
```


### `root` SharedDirectory

DataObject具实质是`SharedDirectory`的`root`属性.通常,在DataObject初始化期间构造期间,您可以创建任何其他分布式数据.并且,
正如所说的,在根目录的共享目录(SharedDirectory)中,
将句柄存储到其他分布式数据.

### DataObject生命周期

DataObject定义了三个_lifecycle方法_,您可以重写它们以创建和初始化分布式数据
结构:

```
/**
 * Called the first time, and *only* the first time, that the DataObject
 * is opened on a client. It is _not_ called on any subsequent clients that
 * open it.
 */
protected async initializingFirstTime(): Promise<void> { }

/**
  * Called every time the DataObject is initialized _from an existing
  * instance_. * Not called the first time the DataObject is initialized.
  */
protected async initializingFromExisting(): Promise<void> { }

/**
  * Called after the DataObject is initialized, regardless of whether
  * it was a first time initialization or an initialization from loading
  * an existing object.
  */
protected async hasInitialized(): Promise<void> { }
```

#### initializingFirstTime

"initializingFirstTime"仅被调用一次.它仅由第一个客户端打开DataObject时执行,并且 在数据对象加载之前,执行完成.您应该实现此方法来实现 执行设置,其中可以包括创建分布式数据结构并使用初始数据填充它们.根SharedDirectory可用于
这种方法.

以下是Badge DataObject的示例:
```ts
protected async initializingFirstTime() {
   //Create a cell to represent the Badge's current state
    const current = SharedCell.create(this.runtime);
    current.set(this.defaultOptions[0]);
    this.root.set(this.currentId, current.handle);

   //Create a map to represent the options for the Badge
    const options = SharedMap.create(this.runtime);
    this.defaultOptions.forEach((v) => options.set(v.key, v));
    this.root.set(this.optionsId, options.handle);

   //Create a sequence to store the badge's history
    const badgeHistory =
        SharedObjectSequence.create<IHistory<IBadgeType>>(this.runtime);
    badgeHistory.insert(0, [{
        value: current.get(),
        timestamp: new Date(),
    }]);
    this.root.set(this.historyId, badgeHistory.handle);
}
```

注意,创建了三个分布式数据结构并填充了初始数据,然后将其存储在root共享目录.

>**请参考:**
> - [创建和存储分布式数据结构](./dds#creating-and-storing-distributed-data-structures)


#### initializingFromExisting

每次首次加载DataObject时都会调用`initializingFromExisting`方法.
创建.请注意,您不需要实现此方法,以便将数据加载到分布式数据结构中.
(因为)在初始化过程中,已经存储在DDS中的数据会自动加载到本地客户端的DDS中.(您)不需要(编写代码来)实现的单独的加载事件.

在简单的情况下,您可能不需要实现此方法,因为会自动加载数据,并且您将使用"initializingFirstTime"来初始创建数据模型.但是,随着数据模型的更改,此方法可以提供一个入口点来让您可以运行升级或放置你需要的模式迁移代码.

#### hasInitialized

"hasInitialized"方法称为"每个时间",即DataObject被加载.
此方法的一种常见用法是隐藏对分布式数据结构的本地引用,以便可以在同步代码中使用它们(本地引用).请回想一下,DDS检索值始终是异步操作,因此只能在异步功能中检索它们. hasInitialized在下面的示例中实现此目的.

```ts
protected async hasInitialized() {
  this.currentCell = await this.root.get<IFluidHandle<SharedCell>>("myCell").get();
}
```

现在,任何同步代码都可以使用"this.currentCell"访问SharedCell.


## DataObjectFactory

像分布式数据结构一样,DataObjects是使用工厂模式异步创建的. (构造函数在TypeScript不能是异步的,因此需要工厂模式)因此,您必须为
DataObject公开一个(如下面的代码示例所示的)工厂类.

DataObjectFactory构造函数采用以下参数.

1. 第一个参数是DataObject的字符串名称.这用于日志记录中.
1.  DataObject子类本身.
1. 工厂数组,每个数组用于DataObject使用的每个DDS.
1. 此参数用在称为_Providers_的更高级的方案中,该方案不在本文档的讨论范围之内.当不使用Providers时,必须传递一个空对象.

```ts
export const BadgeInstantiationFactory = new DataObjectFactory(
    BadgeName,
    Badge,
    [
      SharedMap.getFactory(),
      SharedCell.getFactory(),
      SharedObjectSequence.getFactory(),
    ],
    {},
);
```

## 学到更多

Aqueduct库不仅包含DataObject和DataObjectFactory.要深入了解细节,请参见
[Aqueduct包自述文件](https://github.com/microsoft/FluidFramework/blob/main/packages/framework/aqueduct/README.md).