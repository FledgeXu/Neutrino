# WorldSavedData(世界数据保存)

在这节中，我们将来学习另外一种可以保存数据的方法，这就是World Saved Data。World Saved Data 是Minecraft 提供的一种可以让你保存和共享数据的类。World Saved Data 是在维度级别的，你只能指定某个维度拥有一个WorldSavedData，里面保存的数据是所有玩家通用的，并且用World Saved Data保存的数据与存档数据是分开存放的。

在这节中，我们创建一个可以远程传输和储存物品的机器。

首先我们来看`ObsidianWorldSavedData`:

```java
public class ObsidianWorldSavedData extends WorldSavedData {
    private static final String NAME = "ObsidianWorldSavedData";
    private Stack<ItemStack> itemStacks = new Stack<>();

    public ObsidianWorldSavedData() {
        super(NAME);
    }

    public boolean putItem(ItemStack item) {
        itemStacks.push(item);
        markDirty();
        return true;
    }

    public ItemStack getItem() {
        if (itemStacks.isEmpty()) {
            return new ItemStack(Items.AIR);
        }
        markDirty();
        return itemStacks.pop();
    }

    public ObsidianWorldSavedData(String name) {
        super(name);
    }

    public static ObsidianWorldSavedData get(World worldIn) {
        if (!(worldIn instanceof ServerWorld)) {
            throw new RuntimeException("Attempted to get the data from a client world. This is wrong.");
        }

        ServerWorld world = worldIn.getServer().getWorld(DimensionType.OVERWORLD);
        /***
         *   如果你需要每个纬度都有一个自己的World Saved Data。
         *  用 ServerWorld world = (ServerWorld)world; 代替上面那句。
         */
        DimensionSavedDataManager storage = world.getSavedData();
        return storage.getOrCreate(() -> {
            return new ObsidianWorldSavedData();
        }, NAME);
    }

    @Override
    public void read(CompoundNBT nbt) {
        ListNBT listNBT = (ListNBT) nbt.get("list");
        if (listNBT != null) {
            for (INBT value : listNBT) {
                CompoundNBT tag = (CompoundNBT) value;
                ItemStack itemStack = ItemStack.read(tag.getCompound("itemstack"));
                itemStacks.push(itemStack);
            }
        }
    }

    @Override
    public CompoundNBT write(CompoundNBT compound) {
        ListNBT listNBT = new ListNBT();
        itemStacks.stream().forEach((stack) -> {
            CompoundNBT compoundNBT = new CompoundNBT();
            compoundNBT.put("itemstack", stack.serializeNBT());
            listNBT.add(compoundNBT);
        });
        compound.put("list", listNBT);
        return compound;
    }
}
```

首先你需要给你的WorldSavedData命名，请注意这个名字是必须是唯一的，然后在构建函数的`Super`方法里填入这个名字。

```java
public ObsidianWorldSavedData() {
  super(NAME);
}
public ObsidianWorldSavedData(String name) {
  super(name);
}
```

类似如上。

接下来就是你要创建这个WorldSavedData，我们来看`get`方法

```java
public static ObsidianWorldSavedData get(World worldIn) {
  if (!(worldIn instanceof ServerWorld)) {
    throw new RuntimeException("Attempted to get the data from a client world. This is wrong.");
  }
  ServerWorld world = worldIn.getServer().getWorld(DimensionType.OVERWORLD);
  DimensionSavedDataManager storage = world.getSavedData();
  return storage.getOrCreate(() -> {
    return new ObsidianWorldSavedData();
  }, NAME);
}
```

首先，所有的存档保存的操作都应该在服务端执行，所以我们要先判断我们执行`get`方法时，是不是在服务端。

```java
ServerWorld world = worldIn.getServer().getWorld(DimensionType.OVERWORLD);
```

接下来，我们通过这句话获取了主世界维度相对应的`ServerWorld`类，之所以强制获取主世界相对应的`ServerWorld`原因是，我们得有要实现跨世界传送功能，所以我们把数据统一保存在主世界下的WorldSavedData里。

如果你希望每个维度都有自己的WorldSavedData，可以使用下面这条命令代替即可。

```java
ServerWorld world = (ServerWorld)world;
```

然后就是调用从`ServerWorld`中获取`DimensionSavedDataManager`，并调用`getOrCreate`来获取或者创建我们的WorldSavedData.

`getOrCreate`第一个参数返回值是你的WorldSavedData实例，第二个参数就是你指定的名字。

这样写完以后，你可以在你需要的地方调用`ObsidianWorldSavedData.get`即可获或者自动创建取我们写好的WorldSavedData了。

接下来是，我们自己创建的用来读取和存放数据的接口

```java
public boolean putItem(ItemStack item) {
  itemStacks.push(item);
  markDirty();
  return true;
}

public ItemStack getItem() {
  if (itemStacks.isEmpty()) {
    return new ItemStack(Items.AIR);
  }
  markDirty();
  return itemStacks.pop();
}
```

我们在这里用了` private Stack<ItemStack> itemStacks`来保存我们的数据，当然当你修改完数据之后记得要`markDirty()`不然你修改后的数据不会被保存。

```java
@Override
public void read(CompoundNBT nbt) {
  ListNBT listNBT = (ListNBT) nbt.get("list");
  if (listNBT != null) {
    for (INBT value : listNBT) {
      CompoundNBT tag = (CompoundNBT) value;
      ItemStack itemStack = ItemStack.read(tag.getCompound("itemstack"));
      itemStacks.push(itemStack);
    }
  }
}

@Override
public CompoundNBT write(CompoundNBT compound) {
  ListNBT listNBT = new ListNBT();
  itemStacks.stream().forEach((stack) -> {
    CompoundNBT compoundNBT = new CompoundNBT();
    compoundNBT.put("itemstack", stack.serializeNBT());
    listNBT.add(compoundNBT);
  });
  compound.put("list", listNBT);
  return compound;
}
```

然后就是数据的保存和恢复，这里没什么好说的，只是用的了`ListNBT`来保存我们的数据而已。

然后我们来看看如何使用这个WorldSavedData吧。

```java
public class ObsidianItemSaveBlock extends Block {
    public ObsidianItemSaveBlock() {
        super(Properties.create(Material.ROCK).hardnessAndResistance(5));
    }

    @Override
    public ActionResultType onBlockActivated(BlockState state, World worldIn, BlockPos pos, PlayerEntity player, Hand handIn, BlockRayTraceResult hit) {
        if (!worldIn.isRemote && handIn == Hand.MAIN_HAND) {
            ObsidianWorldSavedData worldSavedData = ObsidianWorldSavedData.get(worldIn);
            ItemStack mainHandItemStack = player.getItemStackFromSlot(EquipmentSlotType.MAINHAND);
            if (!mainHandItemStack.isEmpty()) {
                worldSavedData.putItem(mainHandItemStack.copy());
                mainHandItemStack.shrink(mainHandItemStack.getCount());
            } else {
                ItemStack stack = worldSavedData.getItem();
                player.setItemStackToSlot(EquipmentSlotType.MAINHAND, stack);
            }
        }
        return ActionResultType.SUCCESS;
    }
}
```

这里最重要的就是`onBlockActivated`方法，同样的我们需要判断代码是不是在服务端执行并且传入的手是不是主手。

```java
ObsidianWorldSavedData worldSavedData = ObsidianWorldSavedData.get(worldIn);
ItemStack mainHandItemStack = player.getItemStackFromSlot(EquipmentSlotType.MAINHAND);
```

我们首先获取了之前创建好的WorldSavedData以及主手的ItemStack.

然后判断ItemStack是否为空，不为空代表我们需要放东西，为空代表我们需要取东西。

```java
worldSavedData.putItem(mainHandItemStack.copy());
mainHandItemStack.shrink(mainHandItemStack.getCount());
```

这里就是我们放东西的代码，调用了我们自定义的`putItem`，来存放数据。请注意，这里传入的必须是`mainHandItemStack.copy()`，不然我们之后减少物品时，我们存入的物品也会变空。

而第二句话就是`shrink`「收缩」物品，收缩的数量就是物品的数量。这样我们就把物品变空了。

```java
ItemStack stack = worldSavedData.getItem();
player.setItemStackToSlot(EquipmentSlotType.MAINHAND, stack);
```

这句话也很简单，就是取出物品然后还给玩家而已。

打开游戏右键你的方块试试吧，现在应该可以不限距离，不限维度传输物品了。

[源代码](https://github.com/FledgeXu/NeutrinoSourceCode/tree/master/src/main/java/com/tutorial/neutrino/wordsaveddata)