使用*有向距离场 Signed Distance Field*来绘制阴影；

## SDF 特性

### SDF Blending

使用SDF来进行Blending，能得到平滑的过渡边界；

![blending|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203203005.png)

## SDF 用途

### Ray-SDF Intersection

在Ray Marching中，使用三维的空间SDF来判断光线与物体的相交；
通过SDF，能轻松知道一个点在空间中的“安全移动距离”，在该距离内不会与物体相交；

![sdfRayMathching|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203204211.png)

### SDF Visible Angle

在生成软阴影时，使用SDF来获取不与物体相交的距离，转化为可视角度；
该可视角度即代表了“不被遮挡的程度”，转换后可以作为“阴影的程度”；

![safeSDFAngle|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203205659.png)

## 原理

使用Visible Angle来估算阴影的Visibility；
不准确，但高效；

### 如何求Min Distance

使用Ray Marching的思想，从Shading Point点出发，根据SDF给出的Safe Distance得到点P1，再根据P1的Safe Distance得到点P2......

![computeAngle|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203212737.png)

### 如何求Visible Angle

根据三角函数关系，
$$
\begin{align}
(p-o)\cdot \sin\theta &= SDF(p)\\
\theta &= \arcsin\frac{SDF(p)}{p-o}
\end{align}
$$
![arcsin|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203214001.png)

实际使用中，由于反三角函数比较耗时，又因为$\arcsin$是正相关的，因此没有必要计算反三角项，直接使用比值作为阴影的Visibility，并限制最大值；
$$
\min\{\frac{k\cdot SDF(p)}{p-o}, 1.0\} 
$$
其中$k$用来调节阴影的软硬程度，$k$越大，阴影越硬；

![kValue|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203214712.png)

## 缺点

需要大量空间来存储SDF数据；