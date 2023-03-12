# 硬阴影 vs 软阴影

![softShadow|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221126214956.png)

![penumbra|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221126222741.png)

硬阴影可以看作是本影（Umbra）；
软阴影可以看作是半影（Penumbra）；

# 百分比邻近滤波 PCF
*Percentage Closer Filtering*

## 原理

之前判断一个点是否在阴影内，回答要么是"是(1)"，要么是“否(0)”；
PCF对结果进行Filter，判断目标像素点周围一圈像素的阴影值，求和后计算平均值；
从而得到一个介于0~1之间的值，作为“**阴影的程度(Visibility)**”；

![pcf|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221126220048.png)
$$
\left[
\begin{align}
1&&0&&1\\
1&&0&&1\\
1&&1&&0
\end{align}
\right]
=
\frac{6}{9}
\approx 0.667
$$
==PCF不是对已经有锯齿的阴影进行Filter，那样只会得到模糊的阴影==

## 缺点

计算量增多；
原本只用对一个Shading Point计算阴影值，使用PCF需要计算$n^2$次（假设Filter大小为$n$）；

# 百分比邻近软阴影 PCSS
*Percentage Closer Soft Shadow*

## 原理

观察PCF可以得出，Filter的大小决定阴影的模糊程度；
Filter越小，阴影越锐利；
Filter越大，阴影越模糊；

如果对于阴影的不同位置应用不同大小的Filter，则可以得到软阴影的效果；
那么，决定Filter大小的是什么呢？ --遮挡物距离投影平面的距离（Blocker Distance）；

![pcss|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221126223233.png)

根据相似三角形，可以得到：
$$
W_{Penumbra}=\frac{(d_{Receiver}-d_{Blocker}) \cdot W_{Light}}{d_{Blocker}}
$$
$W_{Penumbra}$的大小即软阴影的大小；

## 步骤

1. Blocker Search 得到范围内遮挡物的平均距离；

范围大小可以是固定值（比如5x5）,也可以使用如下方法动态修改：

![blockerSearch|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221126231501.png)

Blocker Search的范围（即锥体区域）的大小取决于：
	  1> 光源的大小；
	  2> 视点与光源之间的距离；

2. Penumbra Estimation 根据遮挡物平均距离得到Filter大小；
3. 应用PCF得到PCSS；

## 缺点
计算量增多；
相比PCF，还多了一个求平均遮挡物距离的步骤；


# PCF背后的数学原理

Filter，其实等同于卷积(Convolution)；
对范围内的每个值乘以对应的权重，再求和；
$$
[w*f](p)=\sum_{q\in N(p)}w(p,q)f(q)
$$
在PCF/PCSS中，
$$
Visibility(x)=\sum_{q\in N(p)}w(p,q)\cdot \chi^+[D_{ShadowMap}(q)-D_{scene}(x)]
$$
*其中，$\chi^+$表示要么是1要么是0*
 