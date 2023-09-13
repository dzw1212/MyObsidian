辐射度量学（`Radiometry`）是物理学的一个分支，主要研究光的测量，包括光的强度、方向、分布和极化状态等；在计算机图形学中，辐射度量学是非常重要的一部分，因为它提供了一种量化描述和计算光照的方法；

# 重要概念

- 辐射能量 `Radiant Energy`：通过电磁辐射传递的能量，这种能量可以通过波或粒子的形式传播，包括光、射电、紫外线、X射线等各种形式的电磁辐射，其单位是焦耳`Joule`；

$$
Q
$$

- 辐射通量 `Radiant Flux`：单位时间内通过某一面积的辐射能量，其单位是瓦特`Watt`或流明`Lumen`；

$$
\Phi=\frac{dQ}{dt}
$$

- 辐射强度 `Radiant Intensity`：描述单位[[立体角]]（单位方向）上的辐射通量，它是描述辐射源在特定方向上辐射能力的度量，其单位是瓦特/球面度`Watt/Steradian`或坎德拉`candela`；

$$
I(\omega)=\frac{d\Phi}{d\omega}
$$

- 辐照度 `Irradiance`：描述单位面积（与光线垂直的区域）上的辐射通量，它是描述辐射源在特定面积上辐射能力的度量，其单位是瓦特/平方米`Watt/Square Meter`或拉克丝`Lux`；

$$
E(x)=\frac{d\Phi(x)}{dA}
$$

- 辐亮度 `Radiance`：描述单位面积、单位立体角（单位方向）上的辐射通量，它是描述辐射场或辐射源在特定方向上的亮度的度量，其单位是瓦特/(平方米·球面度)`Watt/(Square Meter·Steradian)`；

![radiance|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910173205.png)

$$
L(p,\omega)=\frac{d^2\Phi(p,\omega)}{d\omega \space dA \space \cos{\theta}}
$$


## Radiance与Irradiance

在理想情况下，如果一个表面从所有方向接收到的辐射都是均匀的，那么这个表面的辐照度就等于所有方向上的辐亮度之和；
但在实际情况中，由于光源的方向、距离、遮挡等因素的影响，辐照度和辐亮度之和通常会有所不同；

![Irradiance|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910174059.png)

$$
E(p)=\int_{H^2} L_i(p,\omega) \space \cos{\theta} \space d\omega
$$

其中 $H^2$ 为单位半圆区域；

