`UCharacterMovementComponent`（角色移动组件）是引擎提供给 `ACharacter` 类的核心模块。如果说胶囊体（Capsule Component）是角色的肉体，那么移动组件就是角色的**小脑**。它专门为双足/拟人化角色设计，处理了极其复杂的物理运算、状态切换和网络同步。

为了让你清晰地理解它的能力，我们可以将它的功能拆解为以下几个核心模块：

# 核心功能
## 1. 内置丰富的移动模式 (Movement Modes)

这是该组件最直观的功能。它内部维护了一个状态机，你可以通过 `SetMovementMode()` 随时切换，或者由引擎根据物理环境自动切换：

* **Walking（行走）：** 贴地移动。处理重力、摩擦力、加速度、刹车减速。
* **Falling（掉落/跳跃）：** 悬空状态。处理跳跃初速度（`JumpZVelocity`）、空中控制力（`AirControl`）和重力加速度。
* **Swimming（游泳）：** 在水体（Water Volume）中移动。处理浮力、水下摩擦力和流体阻力。
* **Flying（飞行）：** 无视重力的自由移动（常用于观战视角或飞行角色）。
* **Custom（自定义）：** 允许你自己编写特殊的移动逻辑（比如：爬墙、滑铲、钩爪荡秋千）。

## 2. 精密的物理与地形交互 (World Interaction)

你不需要自己写射线检测（Raycast）去处理复杂的楼梯和斜坡，它全都帮你做好了：

* **上楼梯与越障（Step Up）：** 通过 `MaxStepHeight` 属性，角色可以自动“跨”过一定高度的台阶或石头，而不是被卡住。
* **斜坡处理（Walkable Floor Angle）：** 定义角色能爬上的最大坡度。如果坡度太陡，角色会滑下来，并且无法跳跃。
* **边缘防掉落（Ledge Fall Off）：** 配合 AI 使用时，可以防止角色走下悬崖。
* **推动物理对象：** 角色走动时可以挤开开启了物理模拟的箱子或球体（通过调整 `PushForce` 相关参数）。

## 3. 世界级的网络同步 (Network Replication & Prediction)

**这是 `UCharacterMovementComponent` 最具价值、也最难自己实现的功能。** 如果你在做多人联机游戏，这个组件简直是救命恩人：

* **客户端预测（Client Prediction）：** 玩家按下移动键时，客户端立刻移动（无延迟感），同时把移动指令发给服务器。
* **服务器校验与纠回（Server Correction / Rubber-banding）：** 服务器收到指令后进行合法性验证。如果发现客户端作弊或因为网络延迟导致位置偏差过大，服务器会把客户端“拽”回到正确的位置。
* **平滑修正：** 当发生位置纠回时，它会在客户端进行平滑插值，避免画面出现极其突兀的瞬移闪烁。

## 4. 根运动支持 (Root Motion)

传统的游戏移动是“胶囊体带着模型走”，但有时我们需要“动画决定移动”（比如：翻滚、处决、受击击退）。

* 它可以接管动画资源中的 Root Motion 数据。
* 在 Root Motion 播放期间，组件会自动暂停基于摇杆的物理移动，完全由动画的位移来驱动胶囊体，确保角色的脚不会出现“滑步（Foot Sliding）”。

## 5. AI 与导航支持 (Pathfinding Integration)

* **NavMesh 寻路：** 完美适配虚幻引擎的导航网格体。AI 控制器可以直接调用 `MoveTo()`，组件会自动计算路径并沿着路径移动。
* **RVO 避让（Reciprocal Velocity Obstacles）：** 简单的动态避障功能，允许多个由 AI 控制的角色在挤在一起时互相推开，避免重叠卡死。


# 常用参数

| 参数名称 | 作用描述 |
| --- | --- |
| **Max Walk Speed** | 角色贴地行走的最大速度（默认通常是 600）。 |
| **Jump Z Velocity** | 起跳时的瞬间垂直速度，决定跳跃高度。 |
| **Gravity Scale** | 个体受重力影响的倍率（例如设为 0.5 可以在月球漫步）。 |
| **Ground Friction** | 地面摩擦力。数值越低，角色感觉越像在冰面上溜冰。 |
| **Braking Deceleration** | 松开移动键后的刹车减速度。数值越高，停得越干脆。 |
| **Air Control** | 玩家在空中能改变方向的控制力（0 为无法控制，1 为像在地面一样灵活）。 |
# 拓展功能

## 1. 攀爬

![image.png](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260303000132.png)

当切换`MovementMode`为`MOVE_Custom`时，需要重写`PhysCustom`来实现移动逻辑；

### 定义/进入/退出攀爬状态

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260303001045.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260303004141.png)

重写`OnMovementModeChanged`以修改胶囊体大小等属性；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260303004111.png)



## 2. 滑铲

基本流程：

1. **触发进入：** 满足条件时，调用 `SetMovementMode(MOVE_Custom, Custom_Sliding)`。
2. **接管运算：** 重写组件的 `PhysCustom()` 函数。只要你处于 `MOVE_Custom` 模式，引擎每帧都会调用这个函数。你所有的速度、重力和位置修改都在这里写。
3. **安全退出：** 满足结束条件时，调用 `SetMovementMode(MOVE_Walking)` 或 `MOVE_Falling` 恢复正常。

滑铲的核心在于**给予一个初始爆发速度，并顺着地面的坡度逐渐衰减**。

* **进入条件：** 角色在地面上行走（`MOVE_Walking`）、速度大于某个阈值（正在奔跑），且玩家按下了下蹲键。
* **物理逻辑 (`PhysCustom` 内)：**
* **修改碰撞体：** 瞬间将胶囊体高度减半（借用下蹲逻辑）。
* **速度控制：** 进入时，给当前速度方向加上一个巨大的冲量（Impulse）。
* **坡度影响：** 每帧向下打射线检测地面法线（Floor Normal）。如果是在下坡，就给速度叠加上一个顺着斜坡向下的重力加速度；如果是平地或上坡，则施加极大的自定义摩擦力让角色减速。


* **退出条件：** 速度衰减到低于某个阈值（停下了），或者玩家按下了跳跃键（此时要恢复胶囊体高度，并切换到 `MOVE_Falling`）。




