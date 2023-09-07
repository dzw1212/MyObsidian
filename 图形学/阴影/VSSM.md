*方差软阴影贴图 Variance Soft Shadow Mapping*

针对PCSS中计算消耗大的Step 1 `Blocker Search`与Step 3 `Filter` 进行优化；
[[PCF-PCSS]]

## 对于Filter

假如取Filter的大小为3，目标像素点处的深度值为3.5，周围一圈范围内的深度值为：
$$
\left[
\begin{aligned}
1&&2&&3\\
4&&5&&6\\
7&&8&&9
\end{aligned}
\right]
$$
则$Percentage=6/9=66.7\%$，其实我们要求的只是一个比例，不需要知道每个像素点的确切深度值；
假设深度的分布随机均匀，则可以通过**正态分布**来估算当前所处的比例；

![normalDistribute|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202010700.png)

为了得知正态分布的情况，需要得知Filter区域的**方差** + **均值**；（方差决定宽度，均值决定中心线偏移）

### 均值

#### 借助Mipmap

`Mipmap`是硬件支持的，速度比较快；
但是只支持固定位置的正方形区域，并且需要进行插值，因此结果不准确；
如果是非正方形的区域就无法使用`Mipmap`计算了；

![mipmap|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202033819.png)


#### Summed Area Tables(SAT)

`SAT`是一种使用了**前缀和算法**的数据结构，用来快速求和，得到的结果是准确的；

对于一维的情况，

![SAT1|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202034258.png)

若想知道第$n$到第$m$个数的和，只需要计算$SAT[m]-SAT[n-1]$即可，均值则为：
$$
avg=\frac{SAT[m]-SAT[n-1]}{m-n}
$$
对于二维的情况，

![SAT2|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202034904.png)


$$
avg=SAT(right-bottom)-SAT(left-bottom)-SAT(right-top)+SAT(left-top)
$$

### 方差

$$
Var(X) = E(X^2)-E(X)^2
$$
在生成ShaodwMap的同时，额外再生成一个"Square-Depth"的ShadowMap；
或者使用shadowMap的其他通道存储深度的平方值；

### 计算比例

#### PDF与CDF

`PDF`（`Probability Density Function`，概率密度函数）

![PDF|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202012339.png)

`CDF`（`Cumulative Distribution Function`，累计分布函数）

![CDF|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202012554.png)

所要求的比例，即阴影部分的面积，即`CDF`的值；

![CDF2|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202015259.png)

计算`CDF`，需要先标准化，再查表得到`erf(Error Function)`的结果（只有数值解，没有解析解）；
其中$\mu$为均值，$\sigma^2$为方差：
$$
CDF(x)=\frac{1}{2}(1+erf\frac{x-\mu}{\sigma\sqrt{2}})
$$

但是计算`Error Function`还是太麻烦了，因此使用切比雪夫不等式进行近似求解；

#### 切比雪夫不等式 Chebychev Inequality

以下为单侧版本的切比雪夫不等式（不强制要求分布符合正态/高斯分布）：
$$
P(x>t)\le\frac{\sigma^2}{\sigma^2+(t-\mu)^2}\space\space(t>\mu)
$$

![chebychev|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230827214549.png)

右侧阴影的面积不会大于$P$，可以使用$1-P$来近似代替$CDF$，从而避免了计算$erf$；

# 对于Blocker Search

假设Blocker Search的范围是5，目标像素点的深度为7，周围一圈像素点的深度为：

![blockerSearch|150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202031629.png)

其中红色的部分深度更大，不是遮挡物，要求的是蓝色部分遮挡物深度的平均值；
定义所有像素点的平均深度为$Z_{avg}$，遮挡物的平均深度为$Z_{occ}$，非遮挡物的平均深度为$Z_{unocc}$，
非遮挡物的数量为$N_1$，遮挡物的数量为$N_2$，总共像素点数量为$N$，
则有：
$$
\frac{N_1}{N}Z_{unocc}+\frac{N_2}{N}Z_{occ}=Z_{avg}
$$

套用切比雪夫不等式，
$$
\begin{align}
\frac{N_1}{N}&=P(x>t)\\
\frac{N_2}{N}&=1-P(x>t)
\end{align}
$$
$$
P\cdot Z_{unocc}+(1-P)\cdot Z_{occ}=Z_{avg}
$$

其中$Z_{avg}$可以通过上面求均值的方法求得，$P$通过切比雪夫不等式计算得到，因此要想得到$Z_{occ}$的值，只需要$Z_{unocc}$的值即可；


此时==大胆假设==$Z_{unocc}$==等同于shading Point的深度==（==大多数阴影投射在平面上==），从而求得$Z_{occ}$；
# 缺点

1. 如果深度的实际情况不符合正态分布；

![lightLeak|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202040307.png)

![overDark|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202040412.png)

2. 如果阴影接收面不是平面；

![notFlat|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202040707.png)

3. 如果切比雪夫不等式中的$t<\mu$时；

那么计算出的Shadow的程度就会偏大或者偏下，从而导致**阴影过暗**（影响不大）或者**漏光**（对观感影响较大）；

