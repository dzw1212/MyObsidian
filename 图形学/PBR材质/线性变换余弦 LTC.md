线性变换余弦（`Linearly Transformed Cosines`，`LTC`）是一种在计算机图形学中用于高效模拟复杂光照效果的技术，它由Eric Heitz等人在2016年提出；

线性变换余弦的主要思想是**通过线性变换将标准的余弦分布变换到一个更复杂的分布，从而能够更准确地模拟物体表面的反射特性**；这种方法计算效率高，可以在实时渲染中使用，而且可以模拟出各种复杂的光照效果，如金属、塑料、皮肤等；

`LTC`主要用于解决**多边形光源**下的着色，不考虑阴影，将复杂的BRDF和多边形光源的积分问题转化为简单的余弦函数和矩形光源的积分问题；
# 2D BRDF lobe

`2D BRDF lobe`也叫做`2D Slice of BRDF`，即BRDF的2D切片；一般来说，BRDF具有入射方向和出射方向共4个变量（$\theta_i,\phi_i,\theta_o,\phi_o$），2D BRDF波瓣即固定了入射或者出射方向后，只剩下两个维度的BRDF的图像；

![lobe|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230917192115.png)

# Cosine BRDF lobe

`Cosine BRDF lobe`是一种在计算机图形学中常用的模型，用于描述物体表面的反射特性；BRDF定义了光线从一个方向到另一个方向的反射强度，一个BRDF可以由一个或多个lobe组成，每个lobe代表一种特定的反射特性；

`Cosine BRDF lobe`是一种特殊的BRDF lobe，它的反射强度与入射光线和表面法线之间的角度的余弦成正比；这种模型通常用于模拟`Lambertian`反射，也就是理想的漫反射；

在`Lambertian`反射中，光线在所有方向上的反射强度都是一样的，只取决于入射光线和表面法线之间的角度，因此可以用一个`Cosine BRDF lobe`来表示这种反射特性；

![lobe|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230917194840.png)



# 思想

1. **任何一个2D BRDF lobe都可以通过一个矩阵线性转换为一个Cosine BRDF lobe**；

![LTC|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230917200855.png)


![LTC|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230917195827.png)


![](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/ltc_isotropic.gif)

![](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/ltc_anisotropic.gif)


![](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/ltc_skewness.gif)

![LTC|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230918110325.png)


2. 对多边形光源应用`LTC`；

对多边形光源的每个顶点应用和BRDF lobe一样的变换；

![LTC|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230924194818.png)

3. 此时BRDF与多边形光源的积分**存在解析解**；

# 计算

变换后，BRDF lobe、入射光方向、积分域都发生了变化；

![LTC|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230924201601.png)

![LTC|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230924201629.png)

# 效果

![LTC|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230924202931.png)

