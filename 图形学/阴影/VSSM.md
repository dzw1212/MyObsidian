*方差软阴影贴图 Variance Soft Shadow Mapping*
针对PCSS中计算消耗大的Step 1与Step 3进行优化；[[PCF-PCSS]]

## 原理

### 优化 PCSS Step3
对于PCSS Step3中计算$Percentage$的过程，
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
则$Percentage=\frac{6}{9}=66.7\%$，我们要求的只是一个比例，不需要知道每个像素点的深度值；
假设深度的分布符合正态分布，则可以通过**正态分布**来估算当前所处的比例；

![normalDistribute|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202010700.png)

为了得知正态分布的情况，需要**方差** + **均值**；（方差决定宽度，均值决定中心线偏移）

#### 如何得知均值？

##### 借助Mipmap

由于经过了多次插值，通过Mipmap得到的均值并非准确的；

![mipmap|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202033819.png)


##### Summed Area Tables(SAT)

SAT是一种使用了前缀和算法的数据结构，用来快速求和，得到的结果是准确的；
对于一维的情况，

![SAT1|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202034258.png)

若想知道第$n$到第$m$个数的和，只需要计算$SAT[m]-SAT[n-1]$即可，均值则为：
$$
avg=\frac{SAT[m]-SAT[n-1]}{m-n}
$$
对于二维的情况，

![SAT2|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202034904.png)




#### 如何得知方差？
$Var(X) = E(X^2)-E(X)^2$
在生成ShaodwMap的同时，额外再生成一个"Square-Depth"的ShadowMap；

#### 如何计算比例？

##### PDF vs CDF
PDF（Probability Density Function，概率密度函数）

![PDF|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202012339.png)

CDF（Cumulative Distribution Function，累计分布函数）

![CDF|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202012554.png)

所要求的比例，即CDF的值；

![CDF2|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202015259.png)

CDF需要查表借助erf计算，只有数值解，没有解析解；
其中$\mu$为均值，$\sigma^2$为方差：
$$
CDF(x)=\frac{1}{2}(1+erf\frac{x-\mu}{\sigma\sqrt{2}})
$$

##### 切比雪夫不等式

实际计算中没有必要求出CDF的准确值，使用切比雪夫不等式计算近似值即可（不仅限于正态分布）；
$$
P(x>t)=\frac{\sigma^2}{\sigma^2+(t-\mu)^2}\space\space(t>\mu)
$$

### 优化 PCSS Step1

对于PCSS Step1中计算遮挡物平均深度的过程，
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
并且大胆假设$Z_{unocc}=t$（大多数阴影投射在平面上），从而求得$Z_{occ}$；


## 缺点

1. 如果深度的实际情况不符合正态分布，则对阴影比例估算不准确，可能会偏暗或者漏光；

![lightLeak|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202040307.png)

![overDark|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202040412.png)

2. 如果阴影投射面不是平面，则阴影不真实；

![notFlat|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221202040707.png)

