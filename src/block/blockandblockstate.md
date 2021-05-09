# Block和BlockState

在开始我们接下来的讲解之前，我想先来讲一下`BlockState`这个概念，相信已经对`ItemStack`有所了解到你，应该也能很快理解这个概念。

就和在游戏中的所有物品其实是`Itemstack`一样，在游戏中的所有方块其实是`BlockState`，而`BlockState`相比起`Block`，还包括最为重要的`state`信息，也就是状态信息。状态信息乍一看很难以理解，但是其实非常好懂。我们以原版的栅栏举例，当你把原版的栅栏放在地上，栅栏会根据周围方块的不同自动地改变形状，这其实是栅栏自动地改变了状态，我们可以在F3调试模式下查看方块的状态。

![A31FDCB0-F8AC-43BC-BD70-801476111C97](blockandblockstate.assets/A31FDCB0-F8AC-43BC-BD70-801476111C97.jpeg)

这是在默认方块状态下的栅栏模型。

![EDC215E2-DF26-4764-AE2D-2640C7184482](blockandblockstate.assets/EDC215E2-DF26-4764-AE2D-2640C7184482.jpeg)

可以看到我们使用Debug Stick修改了栅栏的方块状态之后，方块的模型也相应地发生了改变。

对于一些简单方块来说，它们可能只有一种可能的状态，比如石头。
