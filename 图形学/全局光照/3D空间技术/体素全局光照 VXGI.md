体素全局光照`Voxel Global Illumination (VXGI) `是一种实时全局光照技术。它通过将场景转换为稀疏的三维体素网格（即体素），然后在这个网格上进行光线追踪，来模拟光线在场景中的传播；这种方法可以捕捉到间接光照，包括反射和折射，从而使场景看起来更加真实；

`VXGI` 是由 `NVIDIA` 开发的，主要用于游戏和其他实时渲染应用。它可以在支持`DirectX 11` 或更高版本的任何 GPU 上运行，但在 `NVIDIA` 的 `Maxwell` 和更高级别的 GPU 上运行时，可以获得最佳性能；

# 步骤

Step1、将场景离散化为分级的体素（`Hierarchy Voxel`），只有包含几何体的地方才有体素，找到被直接光源照亮的体素；

![VXGI|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910194535.png)

Step2、从摄像机出发，考虑每个shading Point的属性（diffuse还是glossy），假如是glossy的，就计算对应光线方向的锥形范围内每个二次光源的贡献；假如是diffuse的，就取几个方向、计算每个方向的锥形范围内每个二次光源的贡献；

![VXGI|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910195529.png)

![VXGI|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230910195605.png)

# 缺陷

1. 场景体素化的开销太高，对于大型场景不友好；