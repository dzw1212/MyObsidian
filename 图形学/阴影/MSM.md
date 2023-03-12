*矩阴影贴图 Moment Shadow Mapping*

相比VSSM，MSM希望更好地解决VSSM描述分布不准确的缺陷；[[VSSM]]

## 原理

使用更高阶的**矩（Moment）** 来拟合分布曲线；
VSSM中用到了Shadow Map和Square-Depth Shadow Map，相当于用了一阶和二阶的矩；
**使用前$m$阶的矩能够拟合$m/2$个台阶的CDF曲线**；
通常使用4阶的矩来拟合，正好使用RGBA四个通道；

![moments|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203191013.png)

对比VSSM的效果：

![VSSM|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203191520.png)

![MSM|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203191611.png)

