# 如何原地转身

当玩家没有运动时，角色不会跟着鼠标视角旋转；
当玩家开始运动时，角色始终面朝鼠标视角；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117213847.png)

## 静止时如何不旋转

一般情况下由于勾选了`Use Controller Rotation Yaw`，角色的`Yaw`会始终跟随鼠标视角；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117214337.png)

使用一个`Rotate Root Yaw`节点来旋转骨骼，当这个值与鼠标视角的`Yaw`相反时，就能抵消鼠标视角的旋转；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117215013.png)

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117215334.png)


## 开始运动时平滑转身

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117220807.png)


## 转身的判定条件

转身动画分为
- 左转90度
- 左转180度
- 右转90度
- 右转180度

使用50度和130度作为分界点，角度差50-130之间，旋转90度；角度差130~180之间，旋转180度；

![2000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117221125.png)




# 提取并调整动画的Yaw数据曲线

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260113021519.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260113021625.png)

**之所以乘以-1，是希望通过之后的整体平移，得到最终值为0.0的曲线，方便使用**；

比如说，左转90度的动画，其原本的Yaw值是从0 ~ -90度的，经过-1后，变成：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260113022055.png)

此时再整体下移90度，变成：

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260113024536.png)


![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260113022714.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260113023033.png)

对所有Turn In Place动画执行相同的操作；

| 动画     |  Yaw变化   | 调整后的Root_Rotation_Z值 |
| ------ | :------: | :------------------: |
| 左转90度  | 0 ~ -90  |       -90 ~ 0        |
| 左转180度 | 0 ~ -180 |       -180 ~ 0       |
| 右转90度  |  0 ~ 90  |        90 ~ 0        |
| 右转180度 |  0 ~180  |       180 ~ 0        |
# 添加Turning曲线

需要有一条曲线，用来区分 **正在转身`Turning`** 和 **转身后恢复`Recovery`** 阶段；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260113025338.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260113030243.png)

# 状态机

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117222225.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117222356.png)

## Turning状态

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117222435.png)

### 设置动画与重置Time

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117222544.png)

### 手动叠加DeltaTime播放动画

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117223129.png)

### 检测IsTurning曲线

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117223217.png)

## Recovery状态

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117223255.png)

### 设置动画并跳转到Turning结束时间点

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117223408.png)

### 接近结束时回到Idle状态

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117223444.png)


# 根据Yaw曲线调整RootYawOffset

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117225502.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117225529.png)

---

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117223556.png)

Turning阶段，计算两帧之间`Yaw`曲线的插值，叠加给`RootYawOffset`；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117225224.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260117225405.png)
