[[瞄准偏移 Aim Offset]]
# 动画设置为叠加动画

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260127005236.png)

以对应的向正前方看的姿态（`AO_CC`）为基准Pose：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260127005700.png)

# 创建AO

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260127011326.png)

一个自带HorizonalAxis（Yaw）和VerticalAxis（Pitch）的2D混合空间：

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260127011504.png)

Blend采样点：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260127013007.png)

- （0，0）：Center-Center
- （0，90）：Center-Up
- （0，-90）：Center-Down
- （-90，0）：Left-Center
- （-90，90）：Left-Up
- （-90，-90）：Left-Down
- （90，0）：Right-Center
- （90，90）：Right-Up
- （90，-90）：Right-Down
- （-180，0）：Left-Back-Center
- （-180，90）：Left-Back-Up
- （-180，-90）：Left-Back-Down
- （180，0）：Right-Back-Center
- （180，90）：Right-Back-Up
- （180，-90）：Right-Back-Down

# AimOffset Player节点

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260127024551.png)


![1500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260127024523.png)
