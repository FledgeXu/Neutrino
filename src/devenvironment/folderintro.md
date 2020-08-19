# 开发环境的介绍

在这一节中，我会介绍一下Mod开发产生的一系列文件和文件夹，以及它们的作用。

首先最为重要的一个文件便是：`build.gradle`，这个文件是Gradle的配置文件，它规定了Mod的项目是如何构建，有哪些依赖，如何配置等。

其中的`minecraft `闭包下的内容就是关于Forge Gradle的配置。

其中`mappings channel: 'snapshot', version: '20190719-1.14.3'`配置项规定了本项目使用的mapping文件版本，这里我强烈建议你经常更新mapping文件，你可以在[这里](http://export.mcpbot.bspk.rs/)找到所有的mapping文件。那么什么是mapping文件呢？还记得我们之间提及的`srg名`和`mcp名`吗？mapping文件的作用就是提供`srg名`和`mcp名`之间的翻译。

`channel`的意思是mapping文件的分类，在大部分情况下，你都应该使用`snapshot`（快照版本）来确保你的mcp名字是最新的。而之后的`version`就是具体的版本了，大部分情况下是高版本游戏兼容低版本mapping的，当然游戏版本号不能相差太远。其中还有两个被注释起来的参数，这里我们暂且不提。

另外一个你可能会用的就是`dependencies`配置，如果你的mod需要依赖别的java库或者别的mod，你需要在这里添加内容，具体添加的方式，注释已经给出了详细的例子，这里就不多说了。其中`minecraft 'net.minecraftforge:forge:1.15.2-31.1.0'`规定了你需要用到的Forge版本，如果你想升级Forge版本可以修改这一行的内容，版本的格式是`net.minecraftforge:forge:游戏版本号-Forge版本号`。

这个文件剩余的部分就和一个普通的`build.gradle`没什么差别了，如果想知道更详细的知识建议去学习Gradle。

接下去的就是`src`文件夹，这里是放我们代码和资源文件的地方，其中`main`文件夹是具体运行代码和文件的地方，`test`文件夹是放测试代码的地方。`main`文件夹下的`java`就是放我们写的java代码的地方，而`resources`文件夹里放的则是我们的材质模型等一些除了代码之外的属于mod的内容。

接下去是`run`文件夹，这个基本上就是一个标准的`.minecraft`文件夹，值得注意的是，因为开发环境是同时有Minecraft客户端和服务器代码的，它们两个是共用`run`目录的。

剩下值得一提的就是`build`目录，当你在Gradle面板里运行`build`任务，你的mod就会被打包好放在`build=>libs`下。

剩下所有带`gradle`相关的文件夹和文件都是Gradle所需要的运行和配置文件，请不要随便删除。

