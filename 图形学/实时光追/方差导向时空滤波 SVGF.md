`SVGF`（`Spatio-Temporal Variance-Guided Filtering`）是一种基于图像的滤波器，用于改善渲染质量，常用于处理光线追踪渲染时的噪声问题，利用图像的空间和时间信息来引导滤波过程，通过计算像素的方差来确定滤波的强度，从而在保持图像细节的同时，有效地减少噪声；
# 权重

## 对于深度

![SVGF|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928192118.png)

对于上图中的A、B，两点在屏幕空间相距较近，两者应该有类似的颜色，也就是说两点对对方的权重应该较大，但是由于两点位于一个斜面上，它们之间的深度差较大，使用高斯滤波等方式得到的权重非常小；

`SVGF`中**引入了梯度来修正斜率对深度差判断的影响**，$\sigma_z$ 控制衰减的速度，$\epsilon$ 避免分母项为0；

![SVGF|320](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928192033.png)

## 对于法线

![SVGF|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928195245.png)

使用法线点乘的结果作为判断依据，并引入 $\sigma_n$ 控制衰减；

![SVGF|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928195413.png)

## 对于颜色

![SVGF|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928195528.png)

在取颜色计算差异时，可能会取到恰好是噪声的点，得到错误的颜色，因此引入一个 $3\times 3$ 的采样区域以及方差，避免噪声对颜色的干扰；

$\sigma_l$ 控制衰减，$\epsilon$ 避免分母为0；

![SVGF|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928200052.png)
