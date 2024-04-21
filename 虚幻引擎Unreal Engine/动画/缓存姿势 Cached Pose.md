`Cached Pose`节点用于存储一个动画姿势（`Pose`），以便可以在动画蓝图的多个地方重用这个姿势，而不需要重新计算这个姿势；

当相同的姿势需要在动画蓝图的多个分支中使用时，使用Cached Pose可以避免重复计算同一个姿势，从而提高性能；

存储`Cached Pose`：
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310210444.png)

使用之前存储的`Cached Pose`：
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240310210615.png)

