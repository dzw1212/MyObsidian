`Control Rig` 是虚幻引擎5中新引入的一个功能强大的工具，它允许动画师和技术美术师通过更高级的程序化控制来创建和编辑骨骼动画，在此之前如果要修改动画需要退回到DCC中修改，非常麻烦。`Control Rig` 提供了一种更灵活、更直观的方式来创建复杂的动画和动态调整骨骼动作，而不仅仅依赖于传统的关键帧动画技术。

*https://docs.unrealengine.com/5.1/zh-CN/control-rig-in-unreal-engine/*
# 用途

1. **程序化动画:** `Control Rig` 使得动画师可以使用图形化的节点和逻辑来定义动画行为，这种方法比传统的关键帧动画更加灵活和强大。例如，可以创建一个控制器来自动调整角色的脚部位置，使其始终贴地面，而不需要为每一帧手动设置关键帧；

2. **动画调整和修正**: 在动画制作过程中，经常需要对动画进行微调和修正。使用`Control Rig`，动画师可以快速调整骨骼的位置和旋转，实现精确的动画效果，而不需要重新制作或大幅修改现有的关键帧动画；

3. **复用和变体**: `Control Rig` 使得创建动画变体变得更加容易。通过调整控制器的参数，可以从一个基础动画中生成多个不同的动作变体，这对于需要大量类似但略有不同动画的游戏开发尤其有用；

# 示例动画的脚步IK

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310225512.png)

在Control Rig出现之前，是在动画蓝图中写一大堆逻辑，维护起来很麻烦，现在则可以针对不同的需求，对骨骼进行编程，统一放到`Control Rig`中。

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310225633.png)

# 创建

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310230724.png)

![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310230652.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310230814.png)

## 为骨骼添加控制点

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310231052.png)

默认名称是骨骼名称加上后缀`_ctrl`，默认与骨骼同层级，通常建议将其取出并且在骨架旁边构建一个`Control Rig`层级。选中控制点并且按下 `Shift+P` 来解除控制点和骨架的嵌套关系。

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310231237.png)

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310231308.png)

设置控制点的一些属性：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310232818.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310232834.png)



将`Control Rig`拖入场景后，会自动进入`Animation Mode`；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310232208.png)

选中对应的轨道，按下`S`可添加关键帧；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310233652.png)

拖动滑块，可以看到骨骼在两个关键帧之间运动；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310233926.png)

## 控制点、骨骼与Null

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310234252.png)

### 控制点

控制点（`Controls`） 是实现`Control Rig`交互的主要元素。它们用于驱动骨骼链，在 `Sequencer` 中制作动画，并提供更多自定义属性。你还可以让控制点以使用自定义的形状和颜色、属性类型和变换限值。 

控制点类型：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310234509.png)

- `Animation Control`：默认控制点类型，提供普通的可视动画控制；
- `Animation Channel`：用于提供动画通道或者自定义属性的控制点。如果启用了 通道组，那么该属性可以在父级控制点上访问，和其它属性一起加入动画；
- `Proxy Control`：代理控制点能够以 驱动/被驱动 的关系链接到其它控制点。这通过在 驱动的控制点（`Driven Controls`） 数组中添加要驱动的控制点来完成。代理控制点不能直接添加动画，但是你可以通过 `Get Driven` 和 `Set Driven` 节点来整体驱动其它控制点；
- 

### 骨骼

除了骨架中已有的骨骼，你还可以在`Control Rig`中新建骨骼。这些骨骼可用于辅助用途，或为特定操作提供额外的骨骼（例如在IK链中创建结束执行器），或以合并方式使"虚拟"骨骼控制点成为"真实"的骨骼。 

### Null

`Null` 是一种容器元素，能以任意方式收集、分组和变换其他Rig元素。在经典的人体模型`Control Rig`设置中，它们能将对称控制点（例如腿部和手臂）分组，以便镜像这些控制点。`Null`这个概念类似于`Autodesk` `Maya`中的 组（`Groups`） ，旨在管理骨骼绑定的结构。 