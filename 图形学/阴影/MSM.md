*矩阴影贴图 Moment Shadow Mapping*

相比`VSSM`，`MSM`希望更好地解决`VSSM`中描述分布不准确的缺陷；
[[VSSM]]

## 原理

使用更高阶的**矩（Moment）** 来拟合分布曲线；
`VSSM`中用到了`Shadow Map`和`Square-Depth Shadow Map`，相当于用了一阶和二阶的矩；
**使用前$m$阶的矩能够拟合$m/2$个台阶的CDF曲线**；
通常使用4阶的矩来拟合，正好使用RGBA四个通道；

![moments|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203191013.png)

对比`VSSM`的效果：

![VSSM|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203191520.png)

![MSM|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203191611.png)

# 优缺点

优点：表现比`VSSM`更好；

缺点：
1. 更大的存储需求；
2. 更大的计算量需求；

