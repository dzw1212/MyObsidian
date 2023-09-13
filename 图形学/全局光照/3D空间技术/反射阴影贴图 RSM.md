`Reflective Shadow Map`用于实现全局光照效果，特别是间接光照和柔和阴影；
这种技术通过将光源视角的深度图（shadow map）与反射向量（reflective vector）结合，从而模拟光线在场景中的反射；

# 二次光源

将直接光照能照射到的每个`shading Point`都视作`Secondary Light Source`；

![RSM|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230905224202.png)

![RSM|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230905224411.png)


为此需要知道两点：

## 寻找二次光源

1、哪些点是能被光源直接照射到、可以视作二次光源？

使用`Shadow Map`即可（或者PCF后的），每个深度图上的像素都可以视作二次光源；

## 计算二次光源的贡献

2、这些二次光源对P点所接收光照的权重是多少？

### 重要假设

首先，为了简化计算，**假设每个二次光源都是diffuse的**，也就是其对每个方向的光照是相同的；
其次，**不考虑二次光源与shading Point之间的可见性**；

### 计算

假设二次光源为点`q`，shading Point为点`p`，相当于用一块Texel大小的Path光源来照亮点`p`：

![RSM|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910175051.png)

$$
\begin{align}
L_o(p,\omega_o)&=\int_{\Omega_{patch}}L_i(p,\omega_i)V(p,\omega_i)f_r(p,\omega_i,\omega_o)\cos{\theta_i}d\omega_i\\
&=\int_{A_{patch}}L_i(q\to p)V(p,\omega_i)f_r(p,q\to p,\omega_o)\frac{\cos{\theta_p}\cos{\theta_q}}{{||q-p||}^2}dA
\end{align}
$$

又因为二次光源是diffuse的，所以

$$
\begin{align}
f_r&=\frac{\rho}{\pi}\\
L_i(q\to p)&=f_r \space \frac{\Phi}{dA}=\frac{\rho \space \Phi}{\pi \space dA}
\end{align}
$$
带入可得

$$
E_q(x,\vec n)=\Phi_q\frac{\max\{0,\vec{n_q} \cdot (x-x_q)\} \space \max\{0,\vec{n}\cdot (x_q-x)\}}{||x-x_q||^4}
$$

可以看到，随着距离的增大，二次光源对shading Point的光照贡献是指数级衰减的，因此只需要计算离shading Point较近的几个二次光源即可；

# 寻找附近的二次光源

将shading Point投影到`Shadow Map`上，假设在阴影贴图上离shading Point近的点、在世界坐标上也在shading Point近；

![RSM|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910183904.png)

假设在`Shadow Map`取一个范围长度为20的区域，只需要将范围内的400个点视作二次光源进行计算即可；

# 效果

![RSM|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910184102.png)

# 优缺点

优点：方便计算，容易实现；
缺点：
1. 假设太多，效果不太真实；
2. 假如直接光照增多，对应的计算量也会翻倍；

