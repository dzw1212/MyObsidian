# 导数

![slope|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231005110941.png)

以下推导 $y=x^2$ 的导数：
	取一点处 $x$ ，该点处有一变化量 $\Delta x$ ，对应的 $y$ 的变化量为：
$$
\begin{align}
\Delta y&=(x+\Delta x)^2-x^2\\
&=x^2+2x\Delta x+(\Delta x)^2-x^2\\
&=2x\Delta x + (\Delta x)^2
\end{align}
$$
	则点 $x$ 处的斜率（或着说导数）为：
$$
\begin{align}
\frac{dy}{dx}&=\frac{\Delta y}{\Delta x}\\
&=\frac{2x\Delta x+(\Delta x)^2}{\Delta x}\\
&=2x+\Delta x\\
\lim_{\Delta x\to 0} \frac{dy}{dx} &=2x
\end{align}
$$

## 二阶导数 

一阶导数揭示了函数曲线的极值点，
	当一阶导数为0时，函数取得极大值或者极小值；
二阶导数揭示了函数曲线的弯曲性，
	当二阶导数小于0时，函数曲线向下弯曲（`Bending Down`）；
	当二阶导数大于0时，函数曲线向上弯曲（`Bending Up`）；
	当二阶导数等于0时，函数曲线弯曲方向发生变化，该点称为**拐点**（`Inflection Point`）；

![inflection|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231005125839.png)


## 三角函数导数

试证明
1. $\sin\theta < \theta \space (0<\theta)$ ；
2. $\tan\theta > \theta \space (0<\theta<\pi/2)$ ；
3. $\lim_{\theta\to 0} \sin\theta/\theta=1$ ；
4. $(\sin\theta)\prime=\cos\theta,(\cos\theta)\prime=-\sin\theta$ ；


1. 由下图易得 $\sin\theta<\theta$ ：
![sin|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231007232016.png)

2. 在下图中，扇形的面积为 $\theta/2$ ，延长线围成的三角形的面积为 $\tan\theta/2$ ，易得扇形的面积小于三角形的面积，得证；

![tan|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231007233155.png)

3. 结合以上两个不等式，当 $0 < \theta < \pi/2$ 时，
$$
\begin{align}
\sin\theta < \theta &\implies \frac{\sin\theta}{\theta}<1\\
\tan\theta > \theta &\implies \frac{\sin\theta}{\theta}>\cos\theta\\
\end{align}
$$
当 $\lim \theta\to 0$ 时，根据夹逼定理可得 $\sin\theta/\theta=1$ ；

4. 根据导数的定义和[[三角函数公式]]进行计算：
$$
\begin{align}
\frac{d}{dx}\sin x&=\lim_{\Delta x\to 0}\frac{\sin(x+\Delta x)-\sin x}{\Delta x}\\
&=\lim_{\Delta x\to 0}\frac{\sin x\cos\Delta x+\cos x\sin\Delta x-\sin x}{\Delta x}\\
&=\lim_{\Delta x\to 0}(\frac{\sin x(\cos\Delta x-1)}{\Delta x}+\frac{\cos x\sin\Delta x}{\Delta x})\\
&=\cos x
\end{align}
$$

$$
\begin{align}
\frac{d}{dx}\cos x&=\lim_{\Delta x\to0}\frac{\cos(x+\Delta x)-\cos x}{\Delta x}\\
&=\lim_{\Delta x\to0}\frac{\cos x\cos\Delta x-\sin x\sin\Delta x-\cos x}{\Delta x}\\
&=\lim_{\Delta x\to0}(\frac{\cos x(\cos\Delta x-1)}{\Delta x}-\frac{\sin x\sin\Delta x}{\Delta x})\\
&=-\sin x
\end{align}
$$

## 求导法则

试推导：
1. 若 $p(x)=f(x)g(x)$ ，则 $p\prime(x)=f\prime(x)g(x)+f(x)g\prime(x)$ ；
2. 若 $q(x)=f(x)/g(x)$ ，则 $q\prime(x)=$ ；


1. $\Delta p(x)$ 为新增的三个矩形的面积之和；

![chainRule|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231011131231.png)

$$
\begin{align}
\Delta p(x)&=\Delta f(x)g(x)+f(x)\Delta g(x)+\Delta f(x)\Delta g(x)\\
p\prime(x)&=\lim_{\Delta x\to 0}\frac{\Delta p(x)}{\Delta x}\\
&=f\prime(x)g(x)+f(x)g\prime(x)+0
\end{align}
$$


## 自然指数函数

试证明 $e^x\times e^X=e^{x+X}$ ：
	自然指数函数的一个性质就是导数为其自身，当 $x=0$ 时，$e^x=(e^x)\prime=1$，
	假设有一个函数 $y=f(x)$ ，其具有自然指数函数的性质，则
$$
\begin{align}
y\prime=1 &\implies y=1+x\\
y\prime=1+x &\implies y=1+x+\frac{1}{2}x^2\\
y\prime=1+x+\frac{1}{2}x^2 &\implies y=1+x+\frac{1}{3\times 2}x^3\\
&......\\
y\prime=y&=1+x+\frac{1}{2}x^2+...+\frac{x^n}{n!}
\end{align} 
$$
	如此得到的，即是 $e^x$ 在 $x=0$ 处的泰勒展开；
$$
\begin{align}
e^x&=1+x+\frac{1}{2}x^2+...+\frac{x^n}{n!}\\
e^X&=1+X+\frac{1}{2}X^2+...+\frac{X^n}{n!}\\
e^{x+X}&=1+(x+X)+\frac{1}{2}(x+X)^2+...+\frac{(x+X)^n}{n!}\\
(1+x+\frac{1}{2}x^2)(1+X+\frac{1}{2}X^2)&=1+x+\frac{1}{2}x^2+X+xX+\frac{1}{2}x^2X+\frac{1}{2}X^2+\frac{1}{2}X^2x+\frac{1}{4}x^2X^2\\
&=1+x+X+\frac{1}{2}x^2+xX+\frac{1}{2}X^2+...\\
\therefore e^xe^X&=e^{x+X}
\end{align}
$$
	当 $x=1$ 时，可以计算得到
$$
e=1+1+\frac{1}{2}+\frac{1}{6}+\frac{1}{24}+...\approx 2.71828
$$


## 积分的推导

![calculus|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231005161955.png)

假设将一段曲线分为多段，每段的间隔为 $\Delta x$ ，则所有`Slope`的总和为：
$$
\begin{align}
\sum \frac{dy}{dx}&=\frac{y_0}{\Delta x}+\frac{y_1-y_0}{\Delta x}+\frac{y_2-y_1}{\Delta x}+...\frac{y_n-y_{n-1}}{\Delta x}\\
&=\frac{y_n-y_0}{\Delta x}
\end{align}
$$
当 $\lim_{n\to\infty}$ 时，$\Delta x\to dx$，$\sum \to \int$，
$$
\int y\prime \space dx = y\lvert_0^n
$$
即——**函数的积分等于其原函数的差值**；

![calculus|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231005165348.png)

随着 $\Delta x\to 0$ ，积分的值越来越趋近于该曲线所围区域的面积；
即——**函数的积分等于其图像所围区域的面积**；


## 可导与连续

### 初等函数

初等函数包括：
1. 常数函数，如 $f(x)=c$，其中 $c$ 为常数；
2. 幂函数，如 $f(x)=x^n$ ；
3. 指数函数，如 $f(x)=a^x$ ；
4. 对数函数，如 $f(x)=\log_a x$ ；
5. 三角函数与反三角函数，如 $\sin(x),\cos(x),\arcsin(x),\arccos(x)$ ；
6. 双曲函数与反双曲函数，如 $\sinh(x),arcsinh(x)$ ；
7. 以上这些函数的加、减、乘、除、复合运算组合；

### 连续

如果函数在某一个领域内有定义，并且当 $x\to x_0$ 时，有 $lim f(x)=f(x_0)$ ，则称函数 $f(x)$ 在 $x_0$ 处连续；
即需要三个条件：
1. 函数 $f(x)$ 在 $x_0$ 处有定义；
2. 当 $x\to x_0$ 时，$lim f(x)$ 存在；
3. 当 $x\to x_0$ 时，$lim f(x)$ 的值等于 $f(x_0)$ ；

**初等函数在其定义域内都是连续的**；

#### epsilon-delta描述

假如函数 $f(x)$ 在点$x_0$ 处连续，则对于任意窄带 $\epsilon$，总存在竖条 $\delta$ ，使得该竖条范围内的函数值位于窄带范围内；

![epsilon|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231005185539.png)

比如函数 $f(x)=\sin(1/x)$ 在 $x=0$ 处就不连续，因为其不满足`epsilon-delta`描述；

![epsilon|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231005185446.png)

### 可导

如果函数 $f(x)$ 在点 $x_0$ 处连续，并且当 $x\to x_0$ 时，
$$
f\prime (x_0)=\lim_{\Delta x\to 0}\frac{f(x_0+\Delta x)-f(x_0)}{\Delta x}
$$
$f\prime(x_0)$ 存在，则称函数 $f(x)$ 在 $x_0$ 处可导；

如果一个函数在某一点处可导，则其在该点处的左导数和右导数均存在且相等，反之则不可导；
比如 $f(x)=|x|$ 在 $x=0$ 处就不可导，因为其左、右导数不相等；
## 极限

![limit|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231005171213.png)

### 组合的极限 - 未定式

如果两个算式的极限均为常数，那么它们的组合的极限即为其极限的组合；
但是一旦涉及到无穷大或0，则结果是未定的（却决于两者趋近于各自极限的速率），这种情况称为**未定式**；
以下为常见的几种未定式：
$$
\begin{align}
a_n-b_n&\to \infty-\infty\\
a_nb_n&\to(0)(\infty)\\
\frac{a_n}{b_n}&\to \frac{0}{0},\frac{\infty}{\infty}\\
(a_n)^{b_n}&\to0^0,1^\infty,\infty^0
\end{align}
$$
1. 对于 $\frac{0}{0}$ 和 $\frac{\infty}{\infty}$ 型，使用洛必达计算；
2. 对于$(0)(\infty)$ 型，变形为 $\frac{0}{0}$ 和 $\frac{\infty}{\infty}$ 型；
3. 对于 $\infty - \infty$ 型，构造分母通分计算；
4. 对于 $0^0,1^\infty,\infty^0$ 型，借助 $\lim u^v=e^{\ln v\ln u}$ 进行计算；
