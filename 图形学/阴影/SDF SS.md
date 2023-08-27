
*有向距离场软阴影 Signed Distance Field Soft Shadow*

使用*有向距离场 Signed Distance Field*来绘制阴影；


# SDF的特性

**使用SDF可以方便地描述运动的混合**；

对于常规的`lerp`，A与B的结果是三色的图案；
倘如把A与B看作黑色的运动过程，则理想中的`blend`的结果是黑色部分正好运动到中间；
`SDF blend`恰好符合我们希望的结果；

![blending|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203203005.png)

# SDF的用途

## SDF Ray-Matching

构建一个场景的`SDF`，场景中一个点的`SDF`值，**表示该点到场景中其他物体的最小距离**；
在`Ray Marching`中，使用三维的空间`SDF`来判断光线与物体的相交；
通过`SDF`，能轻松知道一个点在空间中的“安全移动距离”，在这个距离内不会与其他物体相交；

![sdfRayMathching|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203204211.png)

## SDF Shadow

在生成软阴影时，使用`SDF`来获取不与物体相交的距离（称为安全角度`Safe Angle`），转化为可视角度；
该可视角度即代表了该点处“不被遮挡的程度”，转换后可以作为“阴影的程度”；

![safeSDFAngle|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203205659.png)


### 求Safe Angle

从o点出发，前进SDF(o)的距离，到达P1点，再前进SDF(P1)的距离到达P2，再前进SDF(P2)的距离到达P3......
途中依次记录角度为$\theta_1$，$\theta_2$，$\theta_3$ ......
其中最小的即为`Safe Angle`；

![computeAngle|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203212737.png)

根据三角函数关系，
$$
\begin{align}
(p-o)\cdot \sin\theta &= SDF(p)\\
\theta &= \arcsin\frac{SDF(p)}{p-o}
\end{align}
$$
![arcsin|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203214001.png)

实际使用中，由于反三角函数比较耗时，又因为`arcsin`是正相关的，因此没有必要计算反三角项，直接使用比值作为阴影的Visibility，并限制最大值；
$$
\min\{\frac{k\cdot SDF(p)}{p-o}, 1.0\} 
$$

其中`k`用来调节阴影的软硬程度，`k`越大，阴影越硬；

![kValue|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203214712.png)

# 优缺点

优点：比shadowMap更快；

缺点：
1. 需要大量的预计算得到场景的`SDF`数据（有形变后需要再重新计算）；
2. 需要大量空间来存储`SDF`数据；

