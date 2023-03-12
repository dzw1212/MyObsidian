使用一张图记录各个方向的光照信息（默认光照来自无限远处）；

## 存储格式

### Spherical Map

![SphericalMap|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203224946.png)


### Cube Map

![CubeMap|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203225018.png)

## IBL

*基于图片的光照 Image Based Lighting*

### 蒙特卡洛方法

需要大量采样，费时；

### 拆分积分 Split Sum


![IBLEquation|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203232517.png)

其中的BRDF项为：
$$
BRDF=f_r(p,\omega_i,\omega_o)cos\theta_i
$$
BRDF的性质为，高亮时支持集（定义域）很小，漫反射时平滑，满足近似不等式的条件；

![BRDF|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203235034.png)

$$
\int_\Omega f(x)g(x)dx \approx \frac{\int_{\Omega_G}f(x)dx}{\int_{\Omega_G}dx} \cdot \int_\Omega g(x)dx
$$
假设：
$$
\begin{align}
f(x)&=L_i(p,\omega_i)\\
g(x)&=BRDF
\end{align}
$$
将渲染方程中的Light和BRDF项拆分：
$$
L_o(p,\omega_o)\approx \frac{\int_{\Omega_{f_r}}L_i(p,\omega_i)d\omega_i}{\int_{\Omega_{f_r}}d\omega_i}\cdot \int_{\Omega^+}f_r(p,\omega_i,\omega_o)\cos\theta_i d\omega_i
$$
#### 对于Light项

对$L_i$积分后再除以一个常数值，相当于Filter操作；
该过程可以在计算开始前预先完成，类似Mipmap的生成，称为**Pre-filter**；
生成数张不同Filter Size的Image，通过三线性插值（Trilinear Lerp）计算结果；

![prefilter|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205011734.png)

材质glossy时，反射光形成一个波瓣（Lobe）；材质越粗糙，波瓣越大；
求这个波瓣的Light项，近似相当于在r方向对Pre-filter Image查询取值；

![glossyLobe|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205013107.png)

#### 对于BRDF项

在微表面模型BRDF中，
假设F表示菲涅尔项，G表示阴影遮罩，D表示法线分布，则反射光：
$$
f(\vec{i},\vec{o})=\frac{F(\vec{i}, \vec{h})\cdot G(\vec{i}, \vec{o}, \vec{h})\cdot D(\vec{h})}{4(\vec{n},\vec{i})(\vec{n},\vec{o})}
$$
主要思想还是实现预计算BRDF项，但是BRDF项相关的变量有很多，需要尽量降低BRDF项的维度，以减少预计算的难度；

##### 菲涅尔项的Schlick近似

不同材质的菲涅尔项都类似于下图，因此将菲涅尔项近似为一个指数函数；

![fresnel|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205023036.png)

$F_0$为基础反射率（基础颜色），$\theta$为入射角，
$$
\begin{align}
F(\theta) &= F_0+(1-F_0)(1-\cos\theta)^5\\
\end{align}
$$
##### Beckmann法线分布

![beckmann|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205023441.png)

$\alpha$为粗糙度roughness，$\theta_h$为法线与半程向量的夹角，
$$
D(h)=\frac{e^{-\frac{\tan^2\theta_h}{\alpha^2}}}{\pi \alpha^2\cos^4\theta_h}
$$
##### 剔除$F_0$维度

![cutF0|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205025316.png)

剔除$F_0$维度后，BRDF项的值只与$\alpha$和$\theta$有关，从而实现预计算BRDF项；

![precomputeBRDF|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205025511.png)

#### Split Sum再近似

用求和取代积分：

![splitSum|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205030534.png)

#### 评价

完全避免了采样，快速并且有着不错的还原度；

![splitSum|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205030342.png)


## 环境光的Shadow

### 取最强光源

选择环境光中一个或者几个最强的光源（如日光）来绘制Shadow；

### PRT

*预计算辐射传输 Precomputed Radiance Transfer*

