需要用到`Distance Matching`，因此和停步动画一样：
1. 曲线压缩设置改为`UniformIndexable`；
2. 启用根运动并`Force Root Lock`；
3. 添加`Distance`曲线；

---
# 状态机

状态机逻辑：`Start`对应存在加速度后、速度从0变为Max的过程；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260108221237.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260110194132.png)

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260110194223.png)

# 距离匹配

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260110191334.png)

# 与Cycle动画的衔接

## 使用Inertialization混合



## 同步组 Sync Group



# 方向扭曲与步幅调整

记得取消勾选`Teleport to Explicit Time`，否则读取不到根位移信息：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260110204336.png)

如果要使用`Stride Warping`，需要勾选`Stride Scale Modifier`，平滑步幅以避免动画卡顿抽搐；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260110210150.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260110210244.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260110210306.png)
