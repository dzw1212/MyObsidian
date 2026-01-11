当角色正在向一个方向移动时，玩家突然推反向摇杆（或大幅度改变移动方向），角色为了抵消惯性而产生的“急停并转身”的过程；

需要用到`Distance Matching`，因此和停步动画一样：
1. 曲线压缩设置改为`UniformIndexable`；
2. 启用根运动并`Force Root Lock`；
3. 添加`Distance`曲线；

---

Pivot可以分为两个过程：
1. 减速直到停止；
	加速度立即变为反方向，速度还在原方向慢慢减少；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111202639.png)


2. 反方向加速；
	速度和加速度方向一直，开始慢慢增加；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111202733.png)

# 状态机

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111202242.png)

## 过渡到Cycle的条件

1. 非Pivot减速、且方向产生变化的情况下；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111203107.png)

2. Pivot动画接近结束，通过检测`AnimNotifyState`实现；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111203211.png)

所有Pivot动画在结尾处添加一个时长为0.25s的`AnimNotifyState`；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111203240.png)

## Pivot状态机

Pivot内部维护一个状态机，用来处理角色在两个方向之间反复来回急转的情况；这两个`State`和`Transition Rule`完全一样，只是为了在上一个Pivot还没结束时重新开始一段Pivot；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111203435.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111203602.png)

### OnBecomeRelevant

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111203742.png)

### OnUpdate

Step1、重新设置正确的动画；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111203829.png)

Step2、处理减速和加速两个阶段的`Distance Matching`；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111203937.png)

#### 预测Pivot距离

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111204034.png)

# 角度偏转

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260111204300.png)
