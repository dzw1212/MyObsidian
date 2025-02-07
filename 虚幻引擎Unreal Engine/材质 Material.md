# 拆分向量 ComponentMask

拆分U与V；

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250112214027.png)

# UV偏移 Panner

由于UV偏移需求非常常见，有一个节点`Panner`专门处理这个需求；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250112214759.png)

# 边缘平滑化

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250118223207.png)

将一个贴图的输出作为`LowResBlurredNoise`的`uv`，从而实现平滑化；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250118223513.png)

# 主材质 Master Material

`Master Material`与普通的`Material`没有本质的区别，只是前者通常设计为一个通用的材质模板，包含所有可能的材质属性和功能，可以通过参数化的方式来调整不同的材质效果，使得可以通过一个 `Master Material` 创建多个不同的材质实例；

## UV控制

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250119201830.png)

## BaseColor控制

亮度 `Brightness`
饱和度 `Saturation`
对比度 `Contrast`
色调 `Tint`

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250119202349.png)

