# 开始使用预定义能力

在这一节中我们来学习如何使用Forge已经提供好的Capability。

Forge为我们已经预定义好了一下几种Capability，你可通过查看`@CapabilityInject`注释在原版内容中的使用来找到预定于好的Capability。

- `ANIMATION_CAPABILITY`，接口`IAnimationStateMachine`，这个能力与动画有关
- `ENERGY`，接口`IEnergyStorage`，这个能力就是大家可能都听说过的Forge Energy
- `FLUID_HANDLER_CAPABILITY` ，接口`IFluidHandler` ，这个能力和方块形式的流体有关
-  `FLUID_HANDLER_ITEM_CAPABILITY`，接口`IFluidHandlerItem`，这个能力和物品形式的流体有关
- `ITEM_HANDLER_CAPABILITY`，接口`IItemHandler`，这个能力和物品的输入输出有关

在这节中，我们将会以`ITEM_HANDLER_CAPABILITY`，做一个简单的演示。如果想要更详细的例子，可以看看ustc-zzzz中关于Forge 能力系统介绍的[博客](https://blog.ustc-zzzz.net/forge-energy-demo-1/)。

在这里我们要实现一个只能从上方倒入并且可以帮我们自动删除圆石的垃圾桶。

```java
public class ObsidianTrashTileEntity extends TileEntity {
    public ObsidianTrashTileEntity() {
        super(TileEntityTypeRegistry.obsidianTrash.get());
    }

    @Nonnull
    @Override
    public <T> LazyOptional<T> getCapability(@Nonnull Capability<T> cap, @Nullable Direction side) {
        if (side == Direction.UP && cap == CapabilityItemHandler.ITEM_HANDLER_CAPABILITY) {
            return LazyOptional.of(() -> {
                return new ItemStackHandler() {
                    @Override
                    public boolean isItemValid(int slot, @Nonnull ItemStack stack) {
                        return stack.getItem() == Items.COBBLESTONE;
                    }
                };
            }).cast();
        }
        return super.getCapability(cap, side);
    }
}
```

```java
if (side == Direction.UP && cap == CapabilityItemHandler.ITEM_HANDLER_CAPABILITY) 
```

在这里我们规定了我们的垃圾桶只有从上方导入时才有效，并且声明了有`ITEM_HANDLER_CAPABILITY`这个能力。

```java
 return LazyOptional.of(() -> {
   return new ItemStackHandler() {
     @Override
     public boolean isItemValid(int slot, @Nonnull ItemStack stack) {
       return stack.getItem() == Items.COBBLESTONE;
     }
   };
}).cast();
```

然后在这里创建了一个`ItemStackHandler`实例，这个是Forge内置的已经实现了`IItemHandler`接口的类，大家可以通过查看`IItemHandler`的基础树来找到这个类。

并且我们在创建这个实例的时候复写了`isItemValid`方法用来过滤物品，至于这些方法都有什么作用，请大家自行阅读`IItemHandler`接口的注释。

因为这里的能力是内置了，所以我们就不需要注册了。

至于方块，物品等注册，请读者自行完成。

打开游戏试试吧。

[源代码](https://github.com/FledgeXu/NeutrinoSourceCode/tree/master/src/main/java/com/tutorial/neutrino/use_cap)