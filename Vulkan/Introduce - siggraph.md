
# 与OpenGL的区别

![vulkan-opengl|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603004906.png)

除此之外，Vulkan的屏幕坐标是Y-Down的，与OpenGL相反；

# Vulkan亮点

## Command Buffer

可以在CPU和GPU之间进行并行处理，从而提高渲染性能；
此外还可以重复使用，从而减少了CPU和GPU之间的通信开销；
支持多线程提交；

![commandbuffer|450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603005455.png)

## Pipeline State Object

在OpenGL中，`Pipeline State`是当前的Color、Texture、Shader、Transformation等一系列状态的组合，这导致修改会占用很大的开销；

Vulkan强迫用户在使用前就填充好`Pipeline State Object(PSO)`对象，一旦创建好几乎不可改变，并且可以提前创建好很多需要的PSO，从而节省实时开销；

![Pipeline|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603010212.png)



## 框架结构

理论支持的框架：

![diagram|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603010511.png)

最常见的架构：

![typocalDiagram|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603010641.png)


## SPIR-V

![spirv|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603011225.png)

SPIRV作为一种`half-compiler`的文件类型，其被编译成机器码的速度远快于Shader源码，加快了运行速度；
此外将Shader源码编译为SPIRV文件还避免了源码泄露的风险；