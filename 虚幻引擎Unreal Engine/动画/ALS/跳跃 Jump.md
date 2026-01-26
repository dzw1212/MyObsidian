# 跳跃过程分解

从地面开始跳跃：
1. 开始跳跃 **Start**；
2. 跳跃后尚未达到最高点 **StartLoop**；
3. 到达最高点 **Apex**；
4. 下坠中 **FallingLoop**；
5. 落地 **FallingLand**；
6. 落地后恢复 **LandRecovery**；

从高处落下：
4. 下坠中 **FallingLoop**；
5. 落地 **FallingLand**；
6. 落地后恢复 **LandRecovery**；

# 创建Distance曲线

## Distance Z

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260119225142.png)

## 曲线Translation

将终点的值调整为0；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260119225408.png)

# 处理LandRecovery Additive动画

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121011525.png)


# 状态机

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012021.png)

## JumpStart

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121011642.png)

## JumpStartLoop

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121011720.png)

## JumpApex

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121011751.png)

## JumpFallingLoop

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121011831.png)

## JumpFallingEnd

### 进入条件

通过射线检测测量人物与地面之间的高度差；

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012134.png)

当高度差小于2m时开始播放Land动画；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012224.png)

### 高度Distance Matching

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012419.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012511.png)

## JumpFallingLandRecovery

落地后身下对抗惯性的恢复动作，此处使用一个状态机+Additive动画实现；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012824.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012852.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012951.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121013033.png)

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121013131.png)

# 处理异常落地的情况

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260121012729.png)
