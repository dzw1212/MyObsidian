使用一张图记录各个方向的光照信息（默认光照来自无限远处）；

## 存储格式

### Spherical Map

![SphericalMap|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203224946.png)


### Cube Map

![CubeMap|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203225018.png)

## IBL

*基于图片的光照 Image Based Lighting*

### 蒙特卡洛方法

需要大量采样，费时，一般用于离线计算；

### Split Sum近似


![IBLEquation|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221203232517.png)

其中的BRDF项为：
$$
BRDF=f_r(p,\omega_i,\omega_o)cos\theta_i
$$
BRDF glossy时support很小，diffuse时变化smooth，满足近似不等式的条件；

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
将渲染方程中的Light和BRDF项拆分（不考虑shadow，也就是visibility项）：
$$
L_o(p,\omega_o)\approx \frac{\int_{\Omega_{f_r}}L_i(p,\omega_i)d\omega_i}{\int_{\Omega_{f_r}}d\omega_i}\cdot \int_{\Omega^+}f_r(p,\omega_i,\omega_o)\cos\theta_i d\omega_i
$$
#### 对于Light项

对$L_i$积分后再除以一个常数值，相当于与环境贴图进行`Filter`操作；
该过程可以在计算开始前预先计算，称为**Pre-filter**；
类似Mipmap，生成数张不同Filter Size的Image，通过插值计算结果；

![prefilter|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205011734.png)

反射光形成一个波瓣（Lobe），
材质越粗糙，波瓣越大，Filter越大，所用的环境贴图越模糊；
材质越光滑，波瓣越小，Filter越小，所用的环境贴图越清晰；

![glossyLobe|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205013107.png)

#### 对于BRDF项

在微表面模型BRDF中，
`F`表示菲涅尔项，`G`表示阴影遮罩，`D`表示法线分布，则反射光：
$$
f(\vec{i},\vec{o})=\frac{F(\vec{i}, \vec{h})\cdot G(\vec{i}, \vec{o}, \vec{h})\cdot D(\vec{h})}{4(\vec{n},\vec{i})(\vec{n},\vec{o})}
$$

##### F - Schlick近似

不同材质的菲涅尔项都类似于下图，因此将菲涅尔项近似为一个指数函数；

![fresnel|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205023036.png)

$F_0$为基础反射率，$\theta$为入射角，
$$
\begin{align}
F(\theta) &= F_0+(1-F_0)(1-\cos\theta)^5\\
\end{align}
$$
有两个维度：基础反射率$F_0$与入射角$\theta$；

##### D - Beckmann分布

![beckmann|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205023441.png)

$\alpha$为粗糙度roughness，$\theta_h$为法线与半程向量的夹角，
$$
D(h)=\frac{e^{-\frac{\tan^2\theta_h}{\alpha^2}}}{\pi \alpha^2\cos^4\theta_h}
$$

有两个维度：粗糙度$\alpha$与半程角$\theta_h$（与$\theta$可以相互转换）；
##### G - SmithGGX近似

![ggx|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230827224223.png)

有两个维度：粗糙度$\alpha$与入射角/出射角$\theta$；

##### 降低维度

总结可得，BRDF项目前共有三个维度：粗糙度，基础反射率，入/出射角；

想要对BRDF项进行预计算，最好能将维度降为2维；为了这个目的，此处使用了UE中实现PBR的`Split Sum`方法，将基础反射率拆出：

![cutF0|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205025316.png)

此时BRDF被分为两个部分，而这两个部分都只有两个维度，都可以做成贴图来预计算存储数据（两个texture，或者同一个texture的两个通道），如下图：

![precomputeBRDF|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205025511.png)

#### Split Sum近似

用求和取代积分：

![splitSum|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205030534.png)

使用预计算避免了采样，不仅快速还有着与采样方法不相上下的还原度；

![splitSum|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221205030342.png)



