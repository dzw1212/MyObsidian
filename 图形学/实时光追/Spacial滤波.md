此处主要是低通滤波；

定义输入的噪声图像为 $\tilde{C}$ ，滤波核（Filter Kernel）为 $K$ ，输出的图像为 $\overline{C}$ ；
# 高斯滤波 Gaussian Filter

![RTRT|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928133641.png)


一般取滤波核的大小为 $3\sigma$ （覆盖大部分分布范围），但为了严谨仍然需要归一化；

伪代码：
```cpp
for each pixel i:
{
	float sum_of_weights = 0.0;
	float sum_of_weighted_values = 0.0;
	for each pixel j around i:
	{
		float weight = CalcGaussian(dis(i, j), sigma);
		sum_of_weights += weight;
		sum_of_weighted_values += weight * C_Input(j);
	}
	C_Ouptut(i) = sum_of_weighted_values / sum_of_weights;
}
```

![RTRT|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928142437.png)

# 双边滤波 Bilateral Filter

希望在模糊画面的同时，能够保持边缘的锐利；

双边滤波在高斯滤波的基础上还会判断一个像素是否是边界（根据颜色变化的剧烈程度），如果是边界则会减少其他像素的权重；

权重公式中不仅考虑了两个像素点的距离还考虑了颜色差异（或者称为“颜色距离”）：

![RTRT|550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928143557.png)

其中空间距离和颜色距离都使用高斯分布进行计算；


![RTRT|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928143734.png)

# 联合双边滤波 Joint Bilateral Filter

高斯滤波使用了像素的空间距离作为权重的系数，双边滤波使用了像素的空间距离和颜色距离作为权重的参数，联合双边滤波则是用了更多的参数来得到更合理的权重；

**联合双边滤波非常适合用于低`SPP`的降噪**；

1. `G-Buffer`中存储的大量信息，可以用来更准确地判断边界；

![RTRT|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928145750.png)

比如上图中，A和B之间使用深度判断，B和C之间使用法线判断，D和E之间使用颜色判断；

2. 不一定要选择高斯分布，只要满足中心值大、离中心越远值越衰减的特点即可；

![RTRT|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928145932.png)


# 大Filter Kernel优化

当滤波核非常大时，滤波操作会变得非常慢，需要用一些方法来优化滤波速度；
## 步骤分解 Separate Passes

假设对于一个高斯滤波，滤波核的大小为 $N\times N$；

PASS 1：先横向滤波一个 $1\times N$ 的区域；
PASS 2：再纵向滤波一个 $N\times 1$ 的区域；

![RTRT|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928154358.png)

相当于将复杂度为 $N^2$ 的操作简化为了 $2N$ ；

原理：
	2维的高斯函数具有这样的一种性质：
$$
G_{2D}(x,y)=G_{1D}(x)\cdot G_{1D}(y)
$$
	因此对于滤波操作：
$$
\begin{align}
\iint F(x_0,y_0) G_{2D}(x_0-x,y_0-y)\mathrm{d}x\mathrm{d}y&=\iint F(x_0,y_0)G_{1D}(x_0-x)G_{1D}(y_0-y)\mathrm{d}x\mathrm{d}y\\
&=\int(\int F(x_0,y_0)G_{1D}(x_0-x)\mathrm{d}x)G_{1D}(y_0-y)\mathrm{d}y

\end{align}
$$
	由公式可得，可以先对x滤波再对y滤波，也可以先对y滤波再对x滤波；

PS：只有高斯滤波核支持该性质，但实际操作时往往对于任何滤波核都这么做，只要滤波核的大小不要太大，误差就可以接收；

## 尺寸递增 Progressively Growing Size

此处介绍一种名为`A-Trous Wavelet`（又名空洞卷积）的方法：

![RTRT|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928161727.png)

![RTRT|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928161811.png)

假如滤波核的大小为 $n$ ，
经过1次扩张，两点之间间距为 $m=2^1-1=1$ ，大小变为了 $n+(n+1)m=2n+1$ ；
经过2次扩张，两点之间间距为 $m=2^2-1=3$，大小变为了 $n+(n+1)m=4n+3$；
经过 $i$ 次扩张，两点之间间距为 $m=2^i-1$ ，大小变为了 $n+(n+1)(2^i-1)=2^i(n+1)-1$ ；

假设此时需要的滤波核大小为64，使用大小为5的滤波核来替代，只需要扩张4次即可，复杂度从 $64\times 64$ 下降为 $5\times 5 \times 5$ ；

