
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

1. SPIRV作为一种`half-compiler`的文件类型，其被编译成机器码的速度远快于Shader源码，加快了运行速度；
2. 将Shader源码编译为SPIRV文件还避免了源码泄露的风险；
3. 语法错误会在SPIRV阶段出现，而不是运行时；

![glslangValidator|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603135815.png)

```cmd
glslangValidator.exe -V sample-vert.vert -o sample-vert.spv
```

或者使用Google封装的`glslc`来编译SPV文件：
```cmd
glslc.exe –target-env=vulkan sample-vert.vert -o sample-vert.spv
```

`glslc`相比`glslangValidator`的部分优点：
1. glslc支持shader文件内的`#include`操作；
2. glslc支持shader文件内的`#define`操作，如下：
```cmd
glslc.exe --target-env=vulkan -DNUMPONTS=4 sample-vert.vert -o sample-vert.spv
```

## 创建Shader Module

![shaderModule|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603135507.png)


## 图元类型

```cpp
typedef enum VkPrimitiveTopology {
    VK_PRIMITIVE_TOPOLOGY_POINT_LIST = 0,
    VK_PRIMITIVE_TOPOLOGY_LINE_LIST = 1,
    VK_PRIMITIVE_TOPOLOGY_LINE_STRIP = 2,
    VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST = 3,
    VK_PRIMITIVE_TOPOLOGY_TRIANGLE_STRIP = 4,
    VK_PRIMITIVE_TOPOLOGY_TRIANGLE_FAN = 5,
    VK_PRIMITIVE_TOPOLOGY_LINE_LIST_WITH_ADJACENCY = 6,
    VK_PRIMITIVE_TOPOLOGY_LINE_STRIP_WITH_ADJACENCY = 7,
    VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST_WITH_ADJACENCY = 8,
    VK_PRIMITIVE_TOPOLOGY_TRIANGLE_STRIP_WITH_ADJACENCY = 9,
    VK_PRIMITIVE_TOPOLOGY_PATCH_LIST = 10,
    VK_PRIMITIVE_TOPOLOGY_MAX_ENUM = 0x7FFFFFFF
} VkPrimitiveTopology;
```

![topology|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603104254.png)



## 创建与填充Data Buffer

![dataBuffer|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230603150259.png)


## 管线数据结构

![pipelineStage|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230605001521.png)

管线数据结构包括：
![pipelineDatastruct|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230605031455.png)

其中加粗的部分可以设置为动态阶段；

支持被设为动态阶段的有：
```cpp
VK_DYNAMIC_STATE_VIEWPORT -- vkCmdSetViewport( )
VK_DYNAMIC_STATE_SCISSOR -- vkCmdSetScissor( )
VK_DYNAMIC_STATE_LINE_WIDTH -- vkCmdSetLineWidth( )
VK_DYNAMIC_STATE_DEPTH_BIAS -- vkCmdSetDepthBias( )
VK_DYNAMIC_STATE_BLEND_CONSTANTS -- vkCmdSetBendConstants( )
VK_DYNAMIC_STATE_DEPTH_BOUNDS -- vkCmdSetDepthZBounds( )
VK_DYNAMIC_STATE_STENCIL_COMPARE_MASK -- vkCmdSetStencilCompareMask( )
VK_DYNAMIC_STATE_STENCIL_WRITE_MASK -- vkCmdSetStencilWriteMask( )
VK_DYNAMIC_STATE_STENCIL_REFERENCE -- vkCmdSetStencilReferences( )
```

管线数据结构中几乎所有的组成部分都是固定大小的，除了`Descriptor Sets`和`Push Constants`，因此可以将管线数据结构看成由两部分组成：
![pipelineData|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230606004347.png)

