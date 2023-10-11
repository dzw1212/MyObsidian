光传播体积（`Light Propagation Volumes`，简称`LPV`）是一种实时全局光照技术；

这种技术的主要思想是将光的传播过程模拟为一个离散的过程，并将这个过程存储在一个体积纹理中；
这种技术可以处理间接光照，包括反射和折射等效果；
# 步骤

Step1、寻找哪些点可以作为二次光源；

依然使用`Shadow Map`；

![LPV|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910190639.png)


Step2、将这些二次光源点放入场景的虚拟3D网格中，并使用球谐函数来表示其`Radiance`；

![LPV|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910192238.png)

Step3、模拟光线的传播过程，每个二次光源点网格的每次传播，可以将光线传播到邻接的6个网格；
重复此过程，直到所有光线稳定下来；

![LPV|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910192415.png)

Step4、对于每个`Vertex`，找到其对应的网格，对其进行着色，从而渲染整个场景；

# 缺陷

1. 整个网格内的Radiance是一致的，如果一个网格内恰好有遮挡物，则会出现不应该照亮的地方被照亮、也就是漏光的情况；

![LPV|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910192937.png)

![LPV|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910193009.png)

解决方法：`Cascade/Multi Scale Grid`；