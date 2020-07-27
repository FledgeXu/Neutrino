# ItemStackTileEntityRenderer(物品特殊渲染)

这一节中，我们将要来学习Item的特殊渲染：`ItemStackTileEntityRenderer`简称`ISTER`，它的作用和`TileEntityRenderer`非常类似，在以前`ItemStackTileEntityRenderer`甚至就是靠`TileEntityRenderer`实现的。

利用`ISTER`你可以实现一些非常酷的渲染效果，举例来说Create（机械动力）中的扳手，它会自动旋转的齿轮就是利用`ISTER`实现的。

首先我们先来看物品的代码

```java
public class ObsidianWrench extends Item {
    public ObsidianWrench() {
        super(new Properties().group(ModGroup.itemGroup).setISTER(() -> {
            return () -> {
                return new ObsidianWrenchItemStackTileEntityRenderer();
            };
        }));
    }
}
```

可以看到物品的代码其实很简单，这里唯一特别的地方就是我们调用了`setISTER`方法，为我们的物品绑定了一个`setISTER`。但是这样绑定还不能直接用，我还得替换物品原本的`IBakedModel`，并在其中启用`ISTER`。

接下来我们来看替换物品`IBakedModel`的代码

```java
@Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.MOD, value = Dist.CLIENT)
public class ModBusEventHandler {
    @SubscribeEvent
    public static void onModelBaked(ModelBakeEvent event) {
        Map<ResourceLocation, IBakedModel> modelRegistry = event.getModelRegistry();
        ModelResourceLocation location = new ModelResourceLocation(ItemRegistry.obsidianWrench.get().getRegistryName(), "inventory");
        IBakedModel existingModel = modelRegistry.get(location);
        if (existingModel == null) {
            throw new RuntimeException("Did not find Obsidian Hidden in registry");
        } else if (existingModel instanceof ObsidianWrenchBakedModel) {
            throw new RuntimeException("Tried to replaceObsidian Hidden twice");
        } else {
            ObsidianWrenchBakedModel obsidianWrenchBakedModel = new ObsidianWrenchBakedModel(existingModel);
            event.getModelRegistry().put(location, obsidianWrenchBakedModel);
        }
    }
}
```

可以看到，我们同样监听了`ModelBakeEvent`，然后通过`modelRegistry.get`获取了默认的`IBakedModel`并将它传入我们新的`IBakedModel`中，然后调用`event.getModelRegistry().put`替换了原版的`IBakedModel`，这里也别忘了`value = Dist.CLIENT`。

接下来看我们的`IBakedModel`：

```java
public class ObsidianWrenchBakedModel implements IBakedModel {
    private IBakedModel existingModel;

    public ObsidianWrenchBakedModel(IBakedModel existingModel) {
        this.existingModel = existingModel;
    }

    @Nonnull
    @Override
    public List<BakedQuad> getQuads(@Nullable BlockState state, @Nullable Direction side, @Nonnull Random rand, @Nonnull IModelData extraData) {
        throw new AssertionError("IForgeBakedModel::getQuads should never be called, only IForgeBakedModel::getQuads");
    }

    @Override
    public List<BakedQuad> getQuads(@Nullable BlockState state, @Nullable Direction side, Random rand) {
        return this.existingModel.getQuads(state, side, rand);
    }

    @Override
    public boolean isAmbientOcclusion() {
        return this.existingModel.isAmbientOcclusion();
    }

    @Override
    public boolean isGui3d() {
        return this.existingModel.isGui3d();
    }

    @Override
    public boolean func_230044_c_() {
        return this.existingModel.func_230044_c_();
    }

    @Override
    public boolean isBuiltInRenderer() {
        return true;
    }

    @Override
    public TextureAtlasSprite getParticleTexture() {
        return this.existingModel.getParticleTexture();
    }

    @Override
    public ItemOverrideList getOverrides() {
        return this.existingModel.getOverrides();
    }

    @Override
    public IBakedModel handlePerspective(ItemCameraTransforms.TransformType cameraTransformType, MatrixStack mat) {
        if (cameraTransformType == ItemCameraTransforms.TransformType.FIRST_PERSON_RIGHT_HAND || cameraTransformType == ItemCameraTransforms.TransformType.FIRST_PERSON_LEFT_HAND) {
            return this;
        }
        return this.existingModel.handlePerspective(cameraTransformType, mat);
    }
}
```

可以看到，这里和之前创建的`IBakedModel`并无太大差别。同样是保存了一个原本的`IBakedModel`。

这里我们来说说他们区别：

第一，对于物品的`IBakedModel`来说，只会调用`IBakedModel#getQuads`而不会调用`IForgeBakedModel::getQuads`，你可以和之前方块的`IBakedModel`做对比，可以发现刚好是相反的。

第二，对于物品，你可以通过` handlePerspective`这个方块来选择不同`TransformType`下的`IBakedModel`，具体的`TransformType`请自行翻阅模型相关的Wiki，这里我们希望在第一人称的视角下用`ISTER`渲染我们的模型，所以在`if`语句中返回了`this`，注意只有当你这里返回了`this`的`TransformType`，才会启用`ISTER`渲染。

第三，你可以在`getOverrides`处理物品`json`文件中的`Override`，什么是`Overrides`请执行翻阅wiki。

第四，也是最重要的，如果你希望你的`IBakedModel`被`ISTER`渲染，你必须在`isBuiltInRenderer`返回`true`，如果你没有给你的物品中调用`setISTER`指定自定义的`ISTER`，默认会使用`ItemStackTileEntityRenderer.instance`渲染。

接下就是最为关键的部分，我们的`ISTER`

```java
public class ObsidianWrenchItemStackTileEntityRenderer extends ItemStackTileEntityRenderer {
    private int degree = 0;

    @Override
    public void render(ItemStack itemStackIn, MatrixStack matrixStackIn, IRenderTypeBuffer bufferIn, int combinedLightIn, int combinedOverlayIn) {
        if (degree == 360) {
            degree = 0;
        }
        degree++;
        ItemRenderer itemRenderer = Minecraft.getInstance().getItemRenderer();
        IBakedModel ibakedmodel = itemRenderer.getItemModelWithOverrides(itemStackIn, null, null);
        matrixStackIn.push();
        matrixStackIn.translate(0.5F, 0.5F, 0.5F);
        float xOffset = -1 / 32f;
        float zOffset = 0;
        matrixStackIn.translate(-xOffset, 0, -zOffset);
        matrixStackIn.rotate(Vector3f.YP.rotationDegrees(degree));
        matrixStackIn.translate(xOffset, 0, zOffset);
        itemRenderer.renderItem(itemStackIn, ItemCameraTransforms.TransformType.NONE, false, matrixStackIn, bufferIn, combinedLightIn, combinedOverlayIn, ibakedmodel.getBakedModel());
        matrixStackIn.pop();
    }
}
```

可以看到我们的`ISTER`类直接继承了`ItemStackTileEntityRenderer`，`ItemStackTileEntityRenderer`只有一个方法，即`render`方法，这里的`render`方法和之前我们学习过的`TER`里的`render`方法作用类似。你可以在这里渲染，你想要渲染的东西。

我只在这里渲染了，我们的物品模型然后在调整完位置之后。让模型旋转起来。

```java
float xOffset = -1 / 32f;
float zOffset = 0;
matrixStackIn.translate(-xOffset, 0, -zOffset);
matrixStackIn.rotate(Vector3f.YP.rotationDegrees(degree));
matrixStackIn.translate(xOffset, 0, zOffset);
```

这段就是让模型能按其中轴线旋转的代码，来源是Create Mod 的`WrenchItemRenderer`。

因为`render`每一帧都会被调用一次。

```java
if (degree == 360) {
  degree = 0;
}
degree++;
```

所以你可以利用这样写出平滑的旋转动画。

到此，我们的物品特殊渲染就完成了。

打开游戏看看，你应该就能看到一个会自动旋转的物品了。

[源代码](https://github.com/FledgeXu/NeutrinoSourceCode/tree/master/src/main/java/com/tutorial/neutrino/ister)

