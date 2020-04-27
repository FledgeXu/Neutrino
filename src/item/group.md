# 自定义创造模式物品栏

在这一节中，我们将研究如何创建一个属于自己的创造模式物品栏，非常简单。

首先创建一个类，让它继承`ItemGroup`，内容如下:

```java
public class ObsidianGroup extends ItemGroup {
    public ObsidianGroup() {
        super("obsidian_group");
    }

    @Override
    public ItemStack createIcon() {
        return new ItemStack(ItemRegistry.obsidianIngot.get());
    }
}
```

第一个方法用于设置创造模式物品栏的标题名，第二个提供了创造模式物品栏的图标，这里我们用了黑曜石碇作为图标。这里是一个ItemStack，可能有的读者还没有阅读关于ItemStack的内容，在这里我稍微讲解一下。

ItemStack故名思义相当于是「一堆」物品，当然这个一堆物品的数量也可以是一个，ItemStack是一个具体的东西，你物品栏中每个槽位的物品都是一个ItemStack，ItemStack比起Item类来说它提供了数量，是否为空等信息，其中最为重要的是ItemStack还能附加NBT标签。

然后我们需要在实例化这个类，创建`ModGroup`

```java
public class ModGroup {
    public static ItemGroup itemGroup = new ObsidianGroup();
}
```

内容如上，应该不用过多解释。

创建完成以后想要调用这个物品栏也非常简单，我们以黑曜石碇举例。

```java
public class ObsidianIngot extends Item {
    public ObsidianIngot() {
        super(new Properties().group(ModGroup.itemGroup));
    }
}
```

此时打开游戏我们的黑曜石碇应该就在指定的物品栏里了。

![image-20200427211358242](group.assets/image-20200427211358242.png)

[源代码](https://github.com/FledgeXu/NeutrinoSourceCode/tree/master/src/main/java/com/tutorial/neutrino/group)