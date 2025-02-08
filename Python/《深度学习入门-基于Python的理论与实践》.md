
numpy

```python
import numpy as np

x = np.array([1, 2, 3]) 
y = np.array([4, 5, 6])
print(x + y) # [5 7 9], 称为element-wise
print(x * 10) # [10 20 30], 称为broadcast

A = np.array([[1, 2], [3, 4], [5, 6]])
print(A.shape) # (3, 2), 矩阵的形状
print(A.dtype) # int64, 矩阵元素的数据类型

A[0][1] # 访问第0行第一列的元素

A = A.flatten() # 转为一维数组
print(A) # [1 2 3 4 5 6]

print(A[np.array([0, 3, 4])]) # 批量获取元素
print(A > 3) # [False False False  True  True  True]
print(A[A > 3]) # [4 5 6], 获取所有大于3的元素
```

matplotlib

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.arange(0, 6, 0.1) # 0到6，间隔为0.1
y1 = np.sin(x)
y2 = np.cos(x)

plt.plot(x, y1, label="sin")
plt.plot(x, y2, linestyle="--", label="cos")
plt.xlabel("x")
plt.ylabel("y")
plt.title("sin & cos")
plt.legend()
plt.show()
```

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201120850.png)


# 感知机

感知机(`perceptron`)是神经网络的起源算法之一，其接收多个输入信号、输出一个信号，感知机的信号只有流1/不流0两种状态，1表示“传递信号”，0表示“不传递信号”；

只有当每个输入的分量之和超过阈值 $\theta$ 后，感知机才会输出1，称为“神经元被激活”；

$$
y=\left\{
\begin{align}
0,\sum_{i=1}^{n}x_iw_i \le \theta\\
1,\sum_{i=1}^{n}x_iw_i > \theta
\end{align}
\right.
$$
假如使用一个2个参数的感知机来表示与门、或门、与非门逻辑，只需要改变参数的值即可，不需要改变模型自身；

==机器学习的课题就是将决定参数值的工作交给计算机自动进行，换言之，机器学习就是确定合适的参数的过程==；

而人要做的就是构造模型，并把训练数据交给计算机；

换一种写法，将阈值 $\theta$ 写为 $-b$ ，此时可将 $b$ 称为**偏置**，决定了神经元被激活的容易程度；$w$ 称为**权重**，决定了输入信号的重要程度；

$$
y=\left\{
\begin{align}
0,b+\sum_{i=1}^{n}x_iw_i \le 0\\
1,b+\sum_{i=1}^{n}x_iw_i > 0
\end{align}
\right.
$$
比如使用一个2输入的感知机来实现与门：
```python
def AND(x1, x2)
	x = np.array([x1, x2])
	w = np.array([0.5, 0.5])
	b = -0.7
	tmp = np.sum(w*x) + b
	if tmp <= 0:
		return 0
	else:
		return 1
```

## 单层感知机的局限性

一个2输入的单层感知机可以表示与门、或门、与非门，但是不能表示异或门，因为前者是线性的、而后者是非线性的；

或门：
![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201141955.png)

异或门：
![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201142036.png)



为了表示一个异或门，可以参考异或门电路的设计，构造一个多层感知机；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201145224.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201145246.png)



# 神经网络

感知机虽然可以复杂的运算，但是参数不宜得出，神经网络的目的就是借助计算机求出这些参数；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201145324.png)

## 激活函数

==激活函数是连接感知机与神经网络的桥梁==；

对于感知机输出的表达式中，当小于0时为0，大于0时为1，这正好可以用**阶跃函数**来表示：

$$
y = h(b+\sum_{i=1}^{n}x_iw_i)
$$
![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201150126.png)

节点a被函数h激活为节点y，此时的函数h就被称为激活函数；

神经网络中常用的激活函数为`sigmoid`函数：

$$
h(x)=\frac{1}{1+\exp(-x)}
$$
```python
def sigmoid(x):
    return 1 / (1 + np.exp(-x))
```

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201151228.png)

相对于阶跃函数只能返回0或1，`sigmoid`函数可以返回0~1之间的实数，不会有急剧的变化；

---

`ReLU`(`Rectified Linear Unit`)函数也是一种常用的激活函数：
$$
h(x)=\left\{
\begin{align} 
x,(x>0)\\
0,(x\le0)
\end{align}
\right.
$$
```python
def ReLU(x):
    return np.maximum(0, x)
```

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201152357.png)

## 多层神经网络

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201155552.png)

使用`numpy`的矩阵点乘来实现上图中的神经网络：

```python
def init_network():
	network = {}
	network['W1'] = np.array([[0.1, 0.3, 0.5], [0.2, 0.4, 0.6]])
	network['b1'] = np.array([0.1, 0.2, 0.3])
	network['W2'] = np.array([[0.1, 0.4], [0.2, 0.5], [0.3, 0.6]])
	network['b2'] = np.array([0.1, 0.2])
	network['W3'] = np.array([[0.1, 0.3], [0.2, 0.4]])
	network['b3'] = np.array([0.1, 0.2])
	return network

def forward(network, x):
	W1, W2, W3 = network['W1'], network['W2'], network['W3']
	b1, b2, b3 = network['b1'], network['b2'], network['b3']
	a1 = np.dot(x, W1) + b1
	z1 = sigmoid(a1)
	a2 = np.dot(z1, W2) + b2
	z2 = sigmoid(a2)
	a3 = np.dot(z2, W3) + b3
	y = identity_function(a3)
	return y

network = init_network()
x = np.array([1.0, 0.5])
y = forward(network, x)
print(y) # [ 0.31682708 0.69627909]
```

其中，`forward`表示从输入到输出方向的传递处理，反之`backward`则为从输出到输入方向的处理；`identity_function`表示恒等，为输出层的激活函数；

## 输出层的激活函数

一般来说，机器学习的问题可以分为**分类问题**（区分图像中的人物、数字）和**回归问题**（根据某个输入预测一个数值），分类问题用`softmax`函数，回归问题用恒等函数；

恒等函数会将输入原封不动地输出，可以用一根箭头表示：

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201160637.png)

`softmax`函数的分子是输入信号的指数函数，分母是所有输入信号的指数函数的和，相当于进行了一次`normalize`，求出了该信号占所有信号的比例；

$$
y_k=\frac{\exp(a_k)}{\sum_{i=1}^n\exp(a_i)}
$$

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201160857.png)

```python
def softmax(a):
    exp_a = np.exp(a)
    sum_exp_a = np.sum(exp_a)
    return exp_a / sum_exp_a
```

由于涉及到指数，当信号的值较大时会超出计算机能承受的数据大小，因此一般会对`softmax`进行优化以避免这种情况；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250201161523.png)

其中`C'`通常取所有信号中的最大值，

```python
def softmax(a):
    c = np.max(a)
    exp_a = np.exp(a-c) # 防止溢出
    sum_exp_a = np.sum(exp_a)
    return exp_a / sum_exp_a
```

## 输出层的神经元数量

需要根据待解决的问题来决定，比如如果问题是识别数字0-9，则可以将神经元设定为10个；