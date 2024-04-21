# Sockets与Virtual Bones的区别

## 骨骼Sockets

- **定义**：Sockets是附加到骨骼网格（Skeletal Mesh）的特定骨骼上的定位点。它们不是骨骼的一部分，而是一个可以附加其他对象（如武器、道具或其他网格）的锚点。
- **用途**：Sockets常用于在角色的手中放置武器或在特定位置附加效果（如粒子系统）。通过使用Sockets，开发者可以轻松地在角色的特定部位动态添加或移除对象。
- **灵活性**：Sockets可以在编辑器中动态添加、移动和旋转，无需修改原始骨骼动画或网格，提供了很高的灵活性。
- 一个骨骼上可以添加任意个Sockets；
- 实时运行时不能修改相对位置、旋转等，但是可以修改对应的骨骼对象；
## 虚拟谷歌Virtual Bones

- **定义**：Virtual Bones是虚幻引擎中的一个概念，允许在两个现有的骨骼之间创建一个虚拟的骨骼。这个虚拟骨骼不会影响网格的几何形状，但可以用于动画和骨骼绑定。
- **用途**：Virtual Bones主要用于简化动画和骨骼设置，比如在需要在两个骨骼之间创建额外的IK（反向动力学）目标点或需要一个参考点来控制动画时。
- **实现**：虽然Virtual Bones在动画系统中表现为骨骼，但它们不会添加到网格的几何结构中，也不会影响网格的性能。它们是完全虚拟的，仅用于动画和逻辑上的参考。
- 一个骨骼上只能附加一个虚拟骨骼；
- 在动画蓝图中可以作为目标使用；

---

一般武器不需要单独的动画，而是依附于握持的骨骼本身，因此适合使用Sockets；

# 给右手骨骼添加一个Socket

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317172354.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317172430.png)

## 添加预览体来调整Socket位置

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317181532.png)

在A字/T字型调整位置是没有意义的，因此需要给骨骼体指定一个动画，此处使用`Idle`动画；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317182051.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317200241.png)

# 附加武器

将武器蓝图拖入人物蓝图，使其成为人物的Mesh组件的子组件（否则无法设置`Parent Socket`）；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317175304.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317175346.png)

给武器选择`Parent Socket`，由于之前已经调整了Socket的位置，因此这里应该是和手部基本吻合的，如果偏差过大可以继续微调：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317175431.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317180127.png)

# 切换武器的Socket

关键节点：`Attach Actor to Component`；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317215653.png)




