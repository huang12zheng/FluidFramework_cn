# 教程

在本演练中,我们将通过构建一个在一起的简单的[DiceRoller](https://github.com/microsoft/FluidHelloWorld)应用程序,学习使用Fluid Framework.
.~~首先开始并继续前进~~,次外,
通过我们的[快速入门](./quick-start.md)指南进行操作.

在我们的DiceRoller应用程序中,我们将向用户展示带有可按钮来滚动的骰子.骰子滚动后,我们将使用Fluid
跨客户端同步数据的框架,因此每个人都可以看到相同的结果.我们将使用以下步骤进行操作.

1. 编写视图.
1. 定义我们的模型将公开的接口.
1. 使用fluid框架编写模型.
1. 将我们的模型包含在我们的容器中.
1. 将我们的容器连接到服务以进行协作.
1. 将模型实例连接到视图以进行渲染.


## 风景

在这个应用程序中,我们将只渲染视图而无需任何UI库,例如React,Vue或Angular.我们将使用
[TypeScript](https://www.typescriptlang.org/)和HTML/DOM方法.Fluid是不关心你的视图的,因此
您可以根据需要使用自己喜欢的视图框架.

由于尚未创建模型,因此只需将代码硬编码为"1",然后在单击按钮时登录到控制台.

```ts
export function renderDiceRoller(div: HTMLDivElement) {
    const wrapperDiv = document.createElement("div");
    wrapperDiv.style.textAlign = "center";
    div.append(wrapperDiv);
    const diceCharDiv = document.createElement("div");
    diceCharDiv.style.fontSize = "200px";
    const rollButton = document.createElement("button");
    rollButton.style.fontSize = "50px";
    rollButton.textContent = "Roll";

    rollButton.addEventListener("click", () => { console.log("Roll!"); });
    wrapperDiv.append(diceCharDiv, rollButton);

    const updateDiceChar = () => {
        const diceValue = 1;
        // Unicode 0x2680-0x2685 are the sides of a die (⚀⚁⚂⚃⚄⚅).
        diceCharDiv.textContent = String.fromCodePoint(0x267F + diceValue);
        diceCharDiv.style.color = `hsl(${diceValue * 60}, 70%, 50%)`;
    };
    updateDiceChar();
}
```


## 模型界面

为了弄清楚我们的模型需要支持什么,让我们从定义其公共接口开始.

```ts
export interface IDiceRoller extends EventEmitter {
    readonly value: number;
    roll: () => void;
    on(event: "diceRolled", listener: () => void): this;
}
```

如您所料,我们可以读取其值并命令其滚动.但是,我们还需要声明一个
我们的界面中的事件`"diceRolled"`.每当骰子滚动时,我们都会触发此事件(使用
[EventEmitter](https://nodejs.org/api/events.html#events_class_eventemitter)).

该活动特别重要,因为我们正在建立协作体验.每个客户都会这样
观察到其他客户端已远程滚动骰子,因此他们知道要使用新值进行更新.

##实施模型

到目前为止,我们一直在使用TypeScript.现在,我们正在为我们的协作实施模型
DiceRoller,我们将开始使用Fluid Framework中的功能.

Fluid Framework提供了一个名为**[DataObject](https://fluidframework.com/apis/aqueduct/dataobject/)**的类,我们可以对其进行扩展以构建模型.我们将使用一些
DataObject的功能,但让我们先看一下代码.

ts
导出类DiceRoller扩展了DataObject实现IDiceRoller {
受保护的asyncinitializingFirstTime(){
this.root.set(diceValueKey,1);
}

受保护的具有hasInitialized(){
this.root.on("valueChanged",(已更改：IValueChanged)=> {
如果(changed.key === diceValueKey){
this.emit("diceRolled");
}
});
}

公开获取值(){
返回this.root.get(diceValueKey);
}

公共只读卷=(()=> {
const rollValue = Math.floor(Math.random()* 6)+ 1;
this.root.set(diceValueKey,rollValue);
};
}
```

由于您创建的模型会随着用户的加载和关闭应用程序而持续保留,因此DataObject提供了生命周期
控制首次创建和随后加载模型的方法.

-当客户端第一次创建DiceRoller时,将运行"initializingFirstTime()".它不会在以下时间运行
  其他客户端连接到该应用程序.我们将使用它来提供骰子的初始值.

-当客户端加载DiceRoller时,会运行"hasInitialized()".我们将使用它来连接事件监听器以响应
  其他客户端中所做的数据更改.

DataObject还提供了**"根"分布式数据结构(DDS)**. DDS是协作数据结构,
您将使用类似本地数据结构的方法,但是当每个客户端修改数据时,所有其他客户端都将看到更改.
这个"根目录"DDS是[SharedDirectory](https://fluidframework.com/apis/map/shareddirectory/),用于存储键/值对,并且与
[地图](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map),提供了类似的方法
set()和get().但是,它也会触发一个""valueChanged"`事件,因此我们可以观察到传入数据的变化
来自其他用户.

要实例化DataObject,Fluid Framework需要一个相应的工厂.由于我们使用的是DataObject类,
我们还将使用与之配对的[DataObjectFactory](https://fluidframework.com/apis/aqueduct/dataobjectfactory/).在这种情况下,我们只需要为其提供一个唯一的名称
(在这种情况下为"骰子滚子")和类构造函数.第三个和第四个参数提供了其他选项,
在此示例中我们将不使用.

```ts
export const DiceRollerInstantiationFactory = new DataObjectFactory(
    "dice-roller",
    DiceRoller,
    [],
    {},
);
```

就是这样-我们的DiceRoller模型完成了！


##定义容器内容

在我们的应用程序中,我们只需要单个骰子的单个模型的单个实例.但是,在更复杂的情况下
我们可能有多种模型类型和许多模型实例.您将编写以指定代码类型和数量的代码
您的应用程序使用的数据对象是"容器代码".

由于我们只需要一个模具,因此Fluid Framework提供了一个名为
[ContainerRuntimeFactoryWithDefaultDataStore](https://fluidframework.com/apis/aqueduct/containerruntimefactorywithdefaultdatastore/)可用作容器代码.我们给它两个参数：
我们想要单个实例的模型工厂的类型,以及我们的容器代码的模型类型的列表
需求(在这种情况下,仅是单个模型类型).此列表称为"容器注册表".

```
export const DiceRollerContainerRuntimeFactory = new ContainerRuntimeFactoryWithDefaultDataStore(
    DiceRollerInstantiationFactory.type,
    new Map([
        DiceRollerInstantiationFactory.registryEntry,
    ]),
);
```

现在我们已经定义了所有部分,现在是时候将它们放在一起了！


##将容器连接到服务以进行协作

为了协调协作,我们需要连接到服务以发送和接收数据更新.方式
我们这样做是为了将[Fluid container](concepts/containers-runtime/)对象连接到服务,然后将我们的容器代码加载到其中.

现在,我们将在名为Tinylicious的本地测试服务上运行,并使其更易于连接到该服务.
我们提供了一个辅助函数`getTinyliciousContainer()`.辅助函数使用唯一的ID来标识我们的
**文档**(我们的应用程序使用的数据集合),容器代码和一个标志,用于指示我们是否要
创建一个新文档或加载现有文档.您可以使用任何想要生成ID并确定ID的应用逻辑
是否创建一个新文档.在[示例
存储库](https://github.com/microsoft/FluidHelloWorld/blob/main/src/app.ts),我们将时间戳和URL哈希用作
一种方法.

```ts
const container =
    await getTinyliciousContainer(documentId, DiceRollerContainerRuntimeFactory, createNew);
```

转到生产服务时,这看起来会有些不同,但是您最终仍会获得
引用运行代码并连接到服务的"容器"对象.

连接了"Container"对象后,我们的容器代码将已经运行以创建我们的实例
模型.因为我们使用了"ContainerRuntimeFactoryWithDefaultDataStore"来构建容器代码,所以我们也可以使用
辅助函数Fluid提供了名为"getDefaultObjectFromContainer"的功能,以获取对模型实例的引用：

```ts
const diceRoller: IDiceRoller = await getDefaultObjectFromContainer<IDiceRoller>(container);
```


## 连接模型实例以进行渲染

现在我们有了一个模型实例,我们可以将其连接到我们的视图！我们将更新功能以
`IDiceRoller`,将我们的按钮连接到`roll()`方法,侦听"diceRolled"事件以检测值变化,并从模型中读取该值.

```ts
export function renderDiceRoller(diceRoller: IDiceRoller, div: HTMLDivElement) {
    const wrapperDiv = document.createElement("div");
    wrapperDiv.style.textAlign = "center";
    div.append(wrapperDiv);
    const diceCharDiv = document.createElement("div");
    diceCharDiv.style.fontSize = "200px";
    const rollButton = document.createElement("button");
    rollButton.style.fontSize = "50px";
    rollButton.textContent = "Roll";

    // Call the roll method to modify the shared data when the button is clicked.
    rollButton.addEventListener("click", diceRoller.roll);
    wrapperDiv.append(diceCharDiv, rollButton);

    // Get the current value of the shared data to update the view whenever it changes.
    const updateDiceChar = () => {
        // Unicode 0x2680-0x2685 are the sides of a die (⚀⚁⚂⚃⚄⚅).
        diceCharDiv.textContent = String.fromCodePoint(0x267F + diceRoller.value);
        diceCharDiv.style.color = `hsl(${diceRoller.value * 60}, 70%, 50%)`;
    };
    updateDiceChar();

    // Use the diceRolled event to trigger the re-render whenever the value changes.
    diceRoller.on("diceRolled", updateDiceChar);
}
```

##运行应用

此时,我们可以运行我们的应用程序了. [此应用程序的完整代码是
可用](https://github.com/microsoft/FluidHelloWorld)供您试用.尝试在多个浏览器窗口中打开它查看客户端之间反映的变化.