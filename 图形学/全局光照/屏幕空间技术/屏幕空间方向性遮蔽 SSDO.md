屏幕空间方向性遮蔽（`Screen Space Directional Occlusion`，简称`SSDO`）用于模拟全局照明效果。它是屏幕空间环境光遮蔽（SSAO）的一个扩展，增加了对光源方向的考虑；

在`SSAO`中，所有的光源都被假设为均匀分布在环境中，因此只需要考虑物体表面被其他物体遮蔽的程度。但在现实世界中，光源通常是有方向的，`SSDO`就是为了模拟这种有方向的光照效果；

`SSDO`的基本步骤与`SSAO`类似，都是对屏幕上的每一个像素进行采样，然后根据采样点的深度和法线信息来计算遮蔽效果。不同的是，`SSDO`还会考虑光源的方向，只有当采样点的法线方向与光源方向相近时，才会认为这个采样点对当前像素产生了遮蔽效果；

并且不同于`SSAO`，`SSDO`可以实现不同颜色反射光的混合，即*Color Blending*的效果；

# 思想

从shading Point发出许多道射线，如果射线没有打到障碍物，则认为该方向收到直接光照的照射；如果射向打到了障碍物，则认为该方向收到间接光照的照射；

![SSDO|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230911205245.png)

# 步骤

Step1、在shading Point法线所在半球内采样多个点，判断这些点是否被光源照亮；

![SSDO|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230911211255.png)

Step2、如果这些点被遮蔽，则取深度图中对应的点作为二次光源，加权计算贡献；

![SSDO|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230911211658.png)


# 缺陷

1. 采样在一个小范围半球内，即认为间接光照仅来自于近处，与现实不符；

![SSDO|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230911213356.png)


2. 某些情况下对是否遮蔽的判断不准确；

![SSDO|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230911212758.png)


# 效果

![SSDO|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230911204334.png)

