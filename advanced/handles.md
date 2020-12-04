# 句柄(Handle)
Fluid句柄是持协作对象(例如DataObject或dds)的引用的对象

在Fluid Framework中,句柄的主要用例是将DataObject或DDS存储到另一个DDS中.
本节介绍如何使用和使用Fluid句柄.

### 为什么使用fluid句柄？

- 例如fluid对象或DDS这样的协作对象,不能直接存储在另一个DDS中.主要有两个原因:

  1. DDS中存储的内容需要可序列化.复杂的对象和类不应直接存储在DDS.
  2. 通常,相同的协作对象(不仅是副本)必须在不同的DDS中可用.唯一的
     实现此目的的方法是将_引用(references)_(这其实就是句柄)存储到DDS中的协作对象.

- 句柄会封装在Fluid运行时的底层对象中,同时会封装如何得到它(references).
存在的位置以及如何检索它.
  通过抽象的(必要性the need to )方式来 向fluid运行 发出"请求"来检索对象,从而降低了调用的复杂性.

- 句柄使Fluid运行时底层 能够构建依赖关系层次结构.这将使我们能够在将来的版本中添加垃圾

### 基本方案

给定一个SharedMap DDS--myMap和一个SharedString DDS--myText,我们想将myText作为值存储在myMap中.因为
现在我们知道我们不能将一个DDS对象直接存储在另一个DDS中,我们需要将一个句柄存储到`myMap`(myText)然后使用该句柄检索"myText"SharedString.

实际上,这类似于以下内容.请注意,您不必构造句柄.DDS的`create`方法
为您执行此操作,并将其分配给DDS的`handle`属性.

```ts
const myMap = SharedMap.create(this.runtime);
const myText = SharedString.create(this.runtime);
myMap.set("my-text", myText.handle);
```

句柄对象本身具有一个异步函数"get()",该函数返回底层对象.在这种情况下,`myText`SharedString实例从句柄中检索对象,会如下所示:
```ts
const textHandle = myMap.get("my-text");
const text = await textHandle.get();
```

因为我们将句柄存储到协作对象中,而不是对象本身,所以可以在
其中传递
该句柄可以在系统中直接传递,并且任何人都可以通过简单地调用`get()`来轻松获得底层对象.这意味着如果我们有
第二个SharedMap称为myMap2,它也可以存储相同的myText SharedString实例.

```ts
const myMap = SharedMap.create(this.runtime);
const myMap2 = SharedMap.create(this.runtime);
const myText = SharedString.create(this.runtime);

myMap.set("my-text", myText.handle);
myMap2.set("my-text", myText.handle);

const text = await myMap.get("my-text").get();
const text2 = await myMap2.get("my-text").get();

console.log(text === text2)//true
```

### 实际方案

以下示例概述了,在不同情况下,使用句柄检索基础对象的用法.

#### 将DDS存储在DataObject的"root"上

从"DataObject"开发Fluid对象时,您经常会发现自己想要创建和存储新的DDS.
在下面的方案中,我们要创建一个新的"SharedMap",所有用户都可以访问,并且我们还想确保它仅创建一次.
我们可以通过在我们的"initializingFirstTime"生命周期方法中创建一个新的SharedMap来做到这一点,
并将其存储在我们的"root"共享目录中. DataObject中的initializingFirstTime函数
仅运行 当`MyFluidObject`第一次被创建时.
每次有MyFluidObject实例时,都会运行
hasInitialized 生命周期方法会 当 MyFluidObject初始化时 运行.我们可以使用它(hasInitialized)来在类中 本地获取和存储 我们的SharedMap.

```ts
export class MyFluidObject extends DataObject {
  public myMap;

  protected async initializingFirstTime() {
      const map = await SharedMap.create(this.runtime)
      this.root.set("map-id", map.handle);
  }

  protected async hasInitialized() {
      this.myMap = await this.root.get<IFluidHandle<SharedMap>>("map-id").get();
  }
}
```

#### 存储其他数据对象

一个Fluid句柄的高级用法是在DataObject`root`中创建和存储其他DataObject.我们
可以像DDS一样
通过将句柄存储到Fluid对象,然后使用它来检索句柄和获取对象.

以下是来自[Pond DataObject demonstrates](https://github.com/microsoft/FluidFramework/blob/main/examples/data-objects/pond/src/index.tsx))的代码段.
[池塘](https://github.com/microsoft/FluidFramework/blob/main/examples/data-objects/pond/src/index.tsx)DataObject
证明了这一点.

示例 在 首次初始化时 创建了一个Clicker对象(它是一个DataObject),

并将其存储在根SharedDirectory,
通过使用Fluid对象的名称作为键来转换成句柄,您可以在任何远程客户端从根目录检索句柄,并通过在 句柄调用`get()`来获得Clicker.

```ts
//...

protected async initializingFirstTime() {
   //The first client creates `Clicker` and stores the handle in the `root` DDS.
    const clickerObject = await Clicker.getFactory().createChildInstance(this.context);
    this.root.set(Clicker.Name, clickerObject.handle);
}

protected async hasInitialized() {
   //The remote clients retrieve the handle from the `root` DDS and get the `Clicker`.
    const clicker = await this.root.get<IFluidHandle>(Clicker.Name).get();
    this.clickerView = new HTMLViewAdapter(clicker);
}

//...
```