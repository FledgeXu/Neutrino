# 事件系统

在开始我们接下来的教程之前，我们得先讲讲事件系统。在之前的章节中，我一直用`DeferredRegister`系统掩盖了直接的事件系统，可惜的是这已经无法再隐藏下去了，在这一节中，我们将学习事件系统，为了读者方便理解，如果你已经忘了总线，事件和事件处理器是什么的话，请先翻阅之前的内容。

首先要说明的是这个事件系统并不是Minecraft自带的，这个事件系统是Forge实现的。

你可以用两种方式来使用事件系统，一种是实例方式，一种是静态的方式。

```java
public class MyForgeEventHandler {
    @SubscribeEvent
    public void pickupItem(EntityItemPickupEvent event) {
        System.out.println("Item picked up!");
    }
}
```

我们先从实例的方法开始说起。可以看到这里最为特殊的就是`@SubscribeEvent`注解，这个注解的作用就是标记下方的`pickupItem` 方法是一个事件处理器，至于它具体监听的事件是由它的参数类型决定的，在这里它的参数类型是`EntityItemPickupEvent`，说明它监听的是实体捡起物品这个事件。

当然，对于实例方式的事件处理这样还不够，我们还得手动在某个地方实例化它并把它注入到事件总线里，我们之前说过Minecraft里有两条事件总线「Forge总线」和「Mod总线」，Mod总线主要负责游戏的生命周期事件，也就是初始化过程的事件，而Forge总线负责的就是除了生命周期事件外的所有事件。你可以用`MinecraftForge.EVENT_BUS.register()`方法将你的事件实例注册到Forge总线中，也可用`FMLJavaModLoadingContext.get().getModEventBus().register()`方法将其注册到Mod总线中，一般情况下你应该在你的Mod主类的初始化方法里注册这些事件。

在我们的例子里就是如下:

```java
MinecraftForge.EVENT_BUS.register(new MyForgeEventHandler());
```

当然所有的事件处理器都要手动注册非常的麻烦，Forge同样提供了一个静态的注册事件的方法。内容如下:

```java
@Mod.EventBusSubscriber()
public class MyStaticClientOnlyEventHandler {
    @SubscribeEvent
    public static void drawLast(RenderWorldLastEvent event) {
        System.out.println("Drawing!");
    }
}
```

可以看到相比之前的代码这里有两点不同，首先多了一个`@Mod.EventBusSubscriber()`注解，这个注解就是用来标注这个类里面的所有打了` @SubscribeEvent`静态方法都是事件处理器，这个注解里有好几个参数，其中最为常用的就是用来指定你所用注入的总线是什么，在默认情况下下面的事件处理注入是Forge总线，你可以用`@Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.MOD)`来指定你要注入到Mod总线中。当然这里的参数不止一个，大家可以自行查看`@Mod.EventBusSubscriber`的具体内容，关于可以使用的变量都要详细的注释。

这里第二点不同就是我们的事件处理函数也变成了静态的，这是在你还不熟悉事件系统使用时非常容易出错的地方，请务必注意。

作为我个人的偏好，我更喜欢后一种静态的事件处理方式，之后的教程也会使用这个方式。

当然事件系统还有很多功能，比如取消，设置结果以及设置优先级，限于篇幅大家可以自行阅读Forge的[文档](https://mcforge.readthedocs.io/en/latest/events/intro/)

