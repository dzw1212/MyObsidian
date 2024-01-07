
建议在`AIController`中添加`AIPerception`组件，因为AIController可以控制多个NPC；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240106172206.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240106172416.png)

# 感知事件

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240106172815.png)


一般最常用的是`OnTargetPerceptionUpdate`，下面这个蓝图判断了感知类型是否为视觉感知，还判断了是否有成功感知；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240106233915.png)

# 感知类型

## 视觉感知

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240106172701.png)


按下单引号`'`后，可以看到一些调试信息，再按下小键盘的`4`，会在NPC周围绘制可视范围的线，其中绿线内为视野感知范围，紫线外为不可感知范围，绿色三条线为视角范围；

下图中，绿球为最后一次视觉感知到的位置；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240106233546.png)

## 预感感知

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240106235702.png)

通过`Request Character/Pawn Prediction Event`来触发预感感知；
根据`Predicted Actor`当前的位置、朝向和速度，以及`Prediction Time`来预测未来的位置，给`Requestor`触发预感感知事件；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107195427.png)

在接收到类型为`Prediction`的感知事件后，通过`Stimulus Location`来获取到预判的未来的位置；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107195850.png)


灰色的球即表示预判出的位置；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107200446.png)

## 听觉感知

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107202414.png)

发出噪音：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107203614.png)


黄圈标识听力范围，黄球标识上一次听觉感知到的位置；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107203404.png)

## 伤害感知

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107203721.png)

通过`Report Damage Event`触发伤害感知事件：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107205025.png)


## 触摸感知

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107205351.png)

通过`Report Touch Event`触发触摸感知事件：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107205510.png)
