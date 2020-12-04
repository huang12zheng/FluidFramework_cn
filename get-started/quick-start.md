## 快速入门
在本快速入门中,我们将安装一个骰子辊fluid应用程序,并在计算机的
本地宿主我们已经在下面嵌入了具有两个客户端的应用程序实例.点击**滚动**
按钮,以查看两个客户端之间如何共享骰子状态.
[quick-start example](https://fluidframework.com/docs/get-started/quick-start/)
## 设置您的开发环境

首先,您需要安装以下软件.

- [Node.js](https://nodejs.org/en/download)
- 代码编辑器-我们建议使用[Visual Studio Code](https://code.visualstudio.com/).

我们还建议您安装以下内容:

- [Git](https://git-scm.com/downloads)

### 安装fluid包装

对于此示例,我们的package.json中已经有必要的Fluid包.因此,当您运行`npm
install`,它们将为您安装.但是,如果您想自己启动一个新项目,
可以在文档的[Fluid API部分](https://fluidframework.com/apis/)获取标记的可用的软件包.

要安装软件包,您可以采用以下格式:`npm i package-name`,如果您使用[npm](https://docs.npmjs.com/)或
如果使用[yarn](https://yarnpkg.com/),则使用`yarn add package-name`.

在本教程中,我们使用以下Fluid软件包:

-**@fluid框架/aqueduct**
-**@fluidframework/get-tinylicious-container**
-**@fluidframework/map**
-**tinylicious**
  
>注意:Tinylicious只是开发依赖项,因为它是
开发您的Fluid应用程序时使用的[service](/concepts/service.md).您可以将其安装为开发依赖项,使用`npm i tinylicious --save-dev`或`yarn addtinylicious --dev'.

## 入门

打开一个新的命令窗口,导航到要在其中安装项目的文件夹,然后克隆
[FluidHelloWorld repo](https://github.com/microsoft/FluidHelloWorld)使用以下命令.克隆过程
将创建一个名为FluidHelloWorld的子文件夹,其中包含项目文件.

```
git clone https://github.com/microsoft/FluidHelloWorld.git
```

>如果您尚未安装git,则可以[单击此处](https://github.com/microsoft/FluidHelloWorld/archive/main.zip)
下载FluidHelloWorld存储库的zip文件.下载文件后,解压缩.zip文件的内容并
运行以下步骤.

导航到新创建的文件夹并安装所需的依赖项.

```
cd FluidHelloWorld
```

```
npm安装
```

启动客户端和服务器.

```
npm开始
```

一个新的浏览器选项卡将打开<http://localhost:8080>,您将看到骰子滚轮出现！要查看协作操作,可以将浏览器中的完整URL(包括ID)复制到新窗口或其他浏览器中.这会打开一个
骰子滚子应用的第二个客户.在两个窗口都打开的情况下,单击任一窗口中的"滚动"按钮并记下骰子的状态在两个客户端中都发生变化.

🥳**恭喜**🎉您已成功迈出了通往Fluid协作世界的第一步.

## 下一步

请参阅[tutorial](./tutorial.md)中骰子滚子应用程序的代码.