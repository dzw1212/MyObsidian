非真实感渲染（`Non-Photorealistic Rendering`，`NPR`），即风格化（`Stylization`）；

# 真实感渲染

需要正确的光照、阴影、基于现实的材质，尽量做到和照片分不出区别；
下图有一半是真实的，一半是渲染的；

![npr|150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230924210143.png)

# 非真实感渲染

虽然一眼看上去就知道不是真实的，但具有艺术的表现力；

## 基本思路

从真实感渲染出发，将想要的艺术风格，具象为具体的渲染步骤；

## 轮廓渲染

![npr|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230924225323.png)

B - Boundary：边界；
C - Crease：折痕；
M - Material Edge：材质的边界；
S - Sihouette：多个面共享的外边界；

### shader描边

如果视线与一个点的观察角度为掠射角时，则可以将该点看作是`Silhouette`边上的一个点；
也就是说，如果视线与一个shading Point的法线接近垂直，则可将该shading Pint视作`Silhouette`；

假如阈值为1度，则 $-89^\circ$ 到 $+89^\circ$  之间点可以被视作轮廓；通过调整阈值的大小，可以控制轮廓的粗细；

![npr|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230924231830.png)

缺点：轮廓的粗细不均匀，取决于法线变化的剧烈程度；

### 几何描边

在渲染一个模型时，同时渲染一个比原模型稍微大一点的相同的模型，其相比原模型多出来的部分可以被视作轮廓；

#### 背向扩增

在渲染图元时会分辨是正向还是背向，一般为了效率背向是不渲染的，因此可以在背向时对图元进行扩增，将扩增后的背向图元渲染为黑色，当正向看时就可以看到黑色的一圈轮廓；

可以沿着边、沿着顶点、或者沿着法线来扩增图元；

![npr|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230925124240.png)

![npr|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230925124300.png)

![npr|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230925124320.png)

### 图像后处理描边

#### 索贝尔边缘检测

`Sobel Detector`，设计某种`Filter Kernel`来对边缘进行检测；

Kernel左正右负，可检测竖向的轮廓；
Kernel上正下负，可检测横向的轮廓；

![npr|750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230925125410.png)



## 色块渲染

通过阈值化（`Thresholding`）以及量化（`Quantization`）将一个颜色的范围渲染为一整块色块；

![npr|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230925130521.png)


![npr|450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230925130643.png)


