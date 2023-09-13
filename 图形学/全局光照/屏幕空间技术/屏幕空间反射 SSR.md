屏幕空间反射（`Screen Space Reflection`，简称`SSR`）是一种在计算机图形学中常用的技术，用于模拟镜面反射的效果；这种技术的主要优点是可以在不需要额外的几何体或预计算的情况下，实现相对真实的反射效果；

# 思想

求出地面射线与场景的交点，使用交点的颜色对地面进行着色；

![ssr|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912225319.png)

## 求交点

### Linear March

在直到shading Point的法线与摄像机角度的情况下，可以计算出光线的方向，沿着光线的方向进行`step by step`的查找，根据深度图判断是否与场景产生交点；
结果的准确度依赖`step`的大小；

![ssr|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912225833.png)
### step - Min Pooling

对于这种不知道取多少步长的情况，希望能将空间进行划分，类似划分三维场景的`BVH`或`KD-Tree`；

![ssr|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912231401.png)

这里通过对深度图进行`Mipmap`来实现划分，与常规`Mipmap`中求平均不同，此处的`Mipmap`求的是一个区域内的深度的最小值，即`Min Pooling`；
在求交点时，就可以类似`BVH`场景中`Ray Marching`的做法，逐步缩小`step`进行查找；

![ssr|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912231746.png)

![ssr|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912232558.png)

![ssr|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912232651.png)


# 缺陷

屏幕空间中的信息是不完善的，`SSR`所实现的反射只能基于不完善的信息；
比如下图中，手心位置的反射信息是缺失的；

![ssr|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912234520.png)

又比如下图，反射出的窗帘明显过于短了；

![ssr|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912234745.png)

# 效果

`SSR`的思路几乎和`Path Tracing`完全相同，区别只是在屏幕空间而已；
因此`SSR`不仅可以实现镜面的反射，它几乎可以实现任何的光线追踪效果；

![ssr|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912224526.png)

![ssr|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912224604.png)

![ssr|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912224632.png)

![ssr|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230912224733.png)

