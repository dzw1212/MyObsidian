# 状态机

当角色的Acceleration为0，即开始减速时，进入Stop；当角色的Velocity为0时，即停步完成，离开Stop；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104014617.png)

# 距离匹配

玩家停止按键输入时，正在运动中的角色会受到**摩擦力Friction**与**制动阻力Braking Deceleration**等参数的影响，其位移距离是不确定的，但是停步动画的根位移长度是确定的，如果直接使用停步动画就会产生实际位移与动画不匹配的问题，因此需要**距离匹配Distance Matching**功能；

1.  角色预测出距离停止点还有 50 厘米。
2.  引擎去 `Distance` 曲线里找：哪一帧对应的距离正好是 50 厘米？
3.  引擎强制将动画跳转（或缩放播放速率）到那一帧。

## 创建编解码器为UniformIndexable的CCS

### 曲线压缩设置 Curve Compression Settings

*   **传统压缩（Compressed Rich Curve）**：为了节省内存，会删掉曲线上的冗余点。比如一段直线，它只保留起点和终点。在播放时，引擎通过插值计算中间值。
*   **距离匹配的需求**：Distance Matching 的核心逻辑是“**反向查询**”。
    *   普通动画是：时间 $\to$ 姿态。
    *   距离匹配是：**剩余距离 $\to$ 应该播放哪一帧**。
    *   这就要求引擎在每一帧都要极快地在曲线中找到“距离为 X 时对应的 Time 是多少”。

`Uniform Indexable` 是一种特殊的编码方式，它不使用关键帧插值，而是将曲线存储为**均匀采样的数组**（类似于 Lookup Table）。

*   **常量时间查询 (O(1) Access)**：
    由于数据是均匀分布的（比如每 0.01 秒一个点），引擎可以通过简单的数学计算直接定位到内存地址，而不需要像传统压缩曲线那样去进行二分查找或复杂的插值运算。
*   **避免“反向查找”的性能瓶颈**：
    距离匹配需要频繁调用 `GetTimeAtDistance` 这种操作。如果曲线是压缩过的，这种反转查询开销很大；如果是 `Uniform Indexable`，查询速度极快，支撑起多个角色同时进行距离匹配。
*   **极高的精度（防止滑步）**：
    `Stop` 动画（停止动画）对脚位要求极高。如果曲线被过度压缩，距离和时间的关系会产生微小偏差，导致脚部在停止的一瞬间产生微小的“滑步”或“抖动”。`Uniform Indexable` 保留了极高密度的采样点，确保了脚位锁定的精准度。

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260103191640.png)

**Curve Compression Settings（动画曲线压缩设置）** 是一个用于控制**非骨骼位移数据**（即“动画曲线”）如何被压缩和优化的资源对象。

简单来说，它的作用是：**在尽量不丢失精度的情况下，减少动画文件中曲线数据占用的内存大小。**

压缩设置的作用是：**删掉那些冗余的关键帧**。例如，如果曲线在一段时间内是直线，或者变化非常微小，压缩算法就会去掉中间的点，只保留起止点，运行时通过插值计算回来。

当你创建一个 `Curve Compression Settings` 资源时，你会看到不同的**编解码器（Codec）**，通常默认使用的是 `AnimCurveCompressionCodec_CompressedRichCurve`。其核心参数包括：

*   **Max Curve Error (最大曲线误差)**：
    *   这是最重要的参数。它决定了压缩的“狠度”。
    *   数值越小（如 0.001），动画越还原，但文件越大。
    *   数值越大（如 0.1），压缩率越高，内存占用越小，但可能会导致动作“走样”或产生抖动。
*   **Sample Rate (采样率)**：决定了在计算压缩时，引擎以多高的频率去检查曲线。

---

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260103192749.png)

`Uniform Indexable` 的缺点是：
*   **内存占用更高**：因为它不再删除冗余点，而是硬性地按采样率存储所有数据，文件体积会比普通压缩设置大。
*   **仅针对关键曲线**：通常我们只对涉及距离匹配的关键曲线（如 `Distance`、`DistanceRemaining`）应用此设置，而不是给所有动画曲线都用。


所有Stop动画均切换为使用该CCS；

## 生成Distance Curve曲线

需要`Animation Modifier Library`插件；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104004346.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104004857.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104004906.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104004931.png)

---

批量生成Distance曲线：

动画序列全选后：
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104005133.png)


# 预测停止距离

需要 `Animation Locmotion Library` 插件；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104005307.png)


![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104005232.png)


![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104005408.png)


# 侧向停步

==需要先取消勾选`Teleport to Explicit Time`，该节点会导致动画不连续，`Orientation Warping`无法正确计算旋转偏移==：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104012513.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260104012755.png)
