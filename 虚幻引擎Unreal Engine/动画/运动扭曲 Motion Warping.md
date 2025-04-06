`Motion Warping `是一种动画技术，用于动态调整角色动画以适应游戏环境的变化。这种技术可以在不改变动画原始内容的情况下，灵活地调整动画的空间和时间属性，以实现更自然和流畅的角色动作。

- **跳跃和攀爬**：角色在跳跃或攀爬时，目标位置可能会有所不同。`Motion Warping `可以调整动画，使角色准确到达目标位置。
- **攻击和躲避**：在近战攻击或躲避动作中，角色需要根据敌人的位置调整动作，`Motion Warping` 可以帮助实现这一点。

- **平滑过渡**：在不同动画之间进行平滑过渡，避免生硬的动画切换。
- **动态调整**：根据游戏状态动态调整动画，例如在不同速度下的奔跑动画。

# 使用步骤
## 启用插件

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250301181208.png)

## 给角色添加运动扭曲组件

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250301181327.png)

## 动画启用根骨骼运动

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250301182119.png)

## 轨道添加运动扭曲段

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250301182218.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250301182243.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250315200625.png)

## 设置运动扭曲的Target

角色蓝图中，

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250301183010.png)


在播放动画之前设置运动扭曲的Target：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250301183552.png)


