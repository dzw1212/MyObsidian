
`Precomputed Radiance Transfer` (`PRT`) 是一种计算机图形学中的技术，用于预计算复杂光照模型的影响，以便在实时渲染中快速应用；这种技术特别适用于处理全局光照效果，如阴影、反射和折射等；

`PRT` 的主要思想是预先计算和存储场景中每个点的光照响应，然后在实时渲染时，通过查找和插值这些预计算的数据来得到最终的光照效果，这样可以大大减少实时渲染时的计算量，从而提高渲染速度；

`PRT` 的一个主要挑战是如何有效地存储和查找预计算的数据，为了解决这个问题，研究者们提出了许多不同的方法，如球谐函数、波前编码和压缩纹理等；

# 基础知识

## 基函数 Basis Functions

在一个向量空间中，基函数是**一组线性无关的函数**，任何在该空间中的函数都可以表示为这组基函数的线性组合；

$$
f(x)=\sum_i c_i \cdot B_i(x)
$$

## 球谐函数 Spherical Harmonics

一组定义在球面上的2D的基函数；

球谐函数是拉普拉斯方程在球坐标系下的解，可以看作是圆谐函数（即复指数函数）在三维空间的推广；它们在球面上形成了一组完备的正交基，**任何定义在球面上的函数都可以表示为球谐函数的线性组合**；

球谐函数的一个重要应用是在计算机图形学中模拟光照和阴影，通过将环境光照表示为球谐函数的线性组合，可以有效地计算物体在复杂光照环境下的渲染效果；

![SH|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230902041615.png)

球谐函数的一些特性：
1. 第 $l$ 阶具有 $2l+1$ 种形态，并且编号从左到右分别是 $-l$  到 $l$ ；
2. 每个球谐函数由勒让德多项式（`Legendre Polynomials`）表示；
3. 阶越高，对应的频率越高；
4. 颜色表示值，越偏白的地方绝对值越大，黄色和蓝色的中心处值为0；
5. 正交性，一个基投影到另一个基上结果为0，一个基投影到自身结果为1；
6. 对旋转的支持非常好，要计算一个函数旋转后的系数，只需要旋转基函数即可；而旋转后的基函数可以由与其同阶的基函数的线性组合得到；


使用球谐函数，可以**只使用前n阶的系数（低频部分）来大致描述一个球面上的函数**；

$$
c_i=\int_{\Omega}f(\omega)B_i(\omega)d\omega
$$

# Diffuse物体的环境光照

![diffuse|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903005922.png)

对Diffuse物体来说，对模糊后的环境贴图的一次采样，等价于，对原图的一个区域内的采样；
也就是说，**Diffuse物体剔除了环境贴图的高频部分，相当于一个低通滤波器**；

一个复杂的环境光照，打在Diffuse物体上，保留的只有环境光照的低频部分；

因此，可以对环境光照进行简化，只使用它的低频部分来代替它（反正打在Diffuse物体上的结果也没差）；

用球谐函数来表示`Diffuse BRDF`，如图，可以看到当到第3阶的时候，系数就基本为0了；

![SH|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903181539.png)

那么对应的，用前3阶的球谐函数来表示环境光照也就足够了；

![SH|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903182625.png)

**对于任何的环境光照，只要被照射物体是Diffuse的，都可以用球谐函数的前3阶来表示，平均误差小于3%**；

# 环境贴图的shadow

![environmentMapShadow|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903183947.png)

假如使用球面函数来表示Visibility（也就是阴影），每个shading Point都需要存储一张不同的cubeMap，存储代价非常高昂；

## PRT的思想

假设场景中只有光照会发生变化，其他部分都不变；
将`Rendering Equation`看作`Light`和`Light transport`两部分，对两部分分别进行预计算；

![PRT|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903185253.png)

`light transport`可以写成一个球面函数，并使用球谐函数来表示；

- 对于BRDF Diffuse的情况下，

BRDF项就是一个常值，
$$
L(o)=\rho \int_{\Omega} L(i)V(i)\max(0,n\cdot i)di
$$

将 $L(i)$ 写作球谐函数的表示形式，发现后半部分变成了求`Visibility`部分用球谐函数表示的系数，也就是一个值 $T_i$ ：

![PRT|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903190126.png)

这么做的一大前提就是一旦渲染开始`Visibility`是不能变化的，也就是场景需要是静态的；

如此，每个shading Point的着色只需要一个点乘即可， 大大提升了渲染效率；

![PRT shadow|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903193514.png)



- 对于BRDF Glossy的情况下，

相比较于BRDF Diffuse的情况，此时摄像机所处的角度会影响BRDF的值，因此**每个不同的角度都对应一组球谐函数的系数**；

$$
L(o) \approx \sum l_iT_i(o)
$$

此时 $T_i(o)$ 是一个关于角度 $o$ 的二维的函数（极坐标$\theta,\phi$），可以使用球谐函数表示；

![PRT|550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903202252.png)

可见对于Glossy的情况，假如球谐函数取前5阶，则`light transport`的预计算结果存储需要25 * 25 * 25的空间；

![PRT|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903203325.png)


## 空间复杂度

假如球谐函数取前5阶，即需要25个系数，则：
- 表示光照 $L_i$ 需要一个*25 * 1*的向量；
- Diffuse时，表示`light transport`需要一个*25 * 1*的向量；
- Glossy时，表示`light transport`需要一个*25 * 25 * 25*的矩阵；

# 预计算方法

以Diffuse时的`light transport`举例，可以把积分中的 $B_i(i)$ 看作某种非常规的光照，此时积分方程就变成了渲染方程，要计算的就是这种非常规的光照的着色结果，可以使用`path tracing`等方法，最后将结果累加起来即可；

![PRT|550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903204459.png)

可以说，无论一个物体的`BRDF`有多复杂（各向异性、各点异性等），只要可以通过离线渲染得到预计算的着色结果，就可以使用`PRT`进行着色；

![PRT|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230903205544.png)

# PRT的缺陷

1. 只适合描述低频的环境光照，太多高频的信息会极大的增加计算量；
2. 只使用于动态光照、静态场景、静态BRDF的情况；

# 改进

## 使用其他基函数

### 哈尔小波  Haar Wavelet

小波基函数是一种数学函数，它在小波变换中起到关键作用。小波变换是一种信号处理技术，可以用于分析信号的不同频率成分，并且可以确定这些频率成分在信号中的位置。

![wavelet|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230904202734.png)


小波基函数具有这样一种性质 —— **其系数大多数为0，因此用有限多个系数就能大致还原函数的低频和高频部分**；

在小波变换的过程中，将图像的高频信息留在右上、右下、左下的格子中，低频信息留在左上的格子中，如此往复（类似于四叉树）；

![wavelet|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230904204939.png)


与球谐函数的对比：

![wvelet|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230904205136.png)

小波函数相比球谐函数的缺陷 —— 不支持快速的旋转变换；


