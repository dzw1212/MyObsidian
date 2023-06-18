
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


## 描述符集

![descriptorSet|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230606125513.png)


Descriptor Set Layout：

![descriptorSetLayoyt|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230606131216.png)

Descriptor Set Layout + Descriptor Set：

![descriptor|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230606131644.png)


Descriptor Set Pool：

![descriptorSetPool|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230606125617.png)

create Descript Set：

![descriptorSets|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230606131836.png)

UBO使用`VkDescriptorBufferInfo`，Texture Sampler使用`VkDescriptorImageInfo`，二者都被封装在`VkWriteDescriptorSet`中，通过`vkUpdateDescriptorSets`来填充allocate出的Descriptor Set；
![updateDescriptorSet|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607111309.png)

在`vkCmdDraw`之前，使用`vkCmdBindDescriptorSets`绑定Descriptor Set；
![bindDescriptorSet|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607112402.png)


总体流程：
![totoalFlow|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607182306.png)


## 贴图

内存申请：
![textureMemory|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607012756.png)


指定参数：
![textureParam|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607013756.png)

Mipmap：
![textureMipmap|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607015324.png)


## Command Buffer

![commandBuffer|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607034622.png)

创建`Command Pool`，并指定具有`Graphic`功能的`Queue Family Index`；
![createCommandPool|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607034925.png)

根据`Swap Chain`中图片的数量来创建`Command Buffer`；
![allocateCommandBuffer|650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607035130.png)

选择指定的一张`Command Buffer`，进行`Begin/End Command Buffer`；
![beginEndCommandBuffer|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607035349.png)


## Swap Chain

![swapChain|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230607035913.png)


## Push Constant

Vulkan规定这些`Push Constant`数据必须至少有128字节大小，尽管它们可以更大；
例如，在NVIDIA 1080ti上的最大尺寸是256字节，可以通过查看 `VkPhysicalDeviceL`中的`maxPushConstantSize`参数来查询这个限制；



## Pipeline Dynamic State

设置为Dynamic State的对象必须在Command Buffer中进行填充，否则其内容是undefined的；
![dynamicState|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230608001758.png)


常见的Dynamic State及其设置方法：
![dynamicStateDefine|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230608001826.png)

![setDynamicState|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230608002144.png)


## Query Pool

![QueryPool|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230608004336.png)

![queryPoo2|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230608004432.png)

## Compute Shader

使用Compute Shader来辅助计算粒子系统；
![computeShader|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230608015320.png)

粒子数据结构：

```cpp
#define NUM_PARTICLES (1024*1024) // total number of particles to move
#define NUM_WORK_ITEMS_PER_GROUP 64 // # work-items per work-group
#define NUM_X_WORK_GROUPS ( NUM_PARTICLES / NUM_WORK_ITEMS_PER_GROUP )

struct pos
{
	glm::vec4; // positions
};
struct vel
{
	glm::vec4; // velocities
};
struct col
{
	glm::vec4; // colors
};
```

```cpp
layout( std140, set = 0, binding = 0 ) buffer Pos
{
	vec4 Positions[ ]; // array of structures
};

layout( std140, set = 0, binding = 1 ) buffer Vel
{
	vec4 Velocities[ ]; // array of structures
};

layout( std140, set = 0, binding = 2 ) buffer Col
{
	vec4 Colors[ ]; // array of structures
};
```


创建DescriptorLayout，将每个UBO对象的`descriptorType`设为`VK_DESCRIPTOR_TYPE_STORAGE_BUFFER`，将`stageFlags`设为`VK_SHADER_STAGE_COMPUTE_BIT`；

![computeShaderLayout|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610160732.png)

![computeShaderLayout2|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610180237.png)




构建用于Compute Shader的`Compute Pipeline`：

![ComputePipeline|550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610141756.png)

![computePipeline|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610151746.png)


创建所需的buffer：

![computeBuffer|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610190559.png)

分配memory并bind：

![computeShader|650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610190643.png)

填充数据：

![computeFillData|750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610193916.png)


工作组，工作项与执行控件：
![work-groups|750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610195707.png)

![work-groups2|750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610200011.png)

在shader中进行计算：

![computerShader|750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610200131.png)

启动computer shader：

![computerShaderLaunch|750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610200241.png)


## Specialization Constants

![specializationConstant|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610201509.png)

一言以蔽之，就是将常量的复制阶段，从shader->spv的阶段，延迟到调用`vkCreateComputePipelines`时。

![specializationConstant|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610204156.png)

![specializationConstant|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610211159.png)

对于多个constant的情况：

![specializationConstant|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610215308.png)


## 同步机制

![synchronization|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610223940.png)


使用Semaphore：

![semaphore|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610224627.png)

使用fence：

![fence|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610225140.png)

![fence|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230610230727.png)


## 反走样与多重采样

走样即信号的失真：
![aliasing|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230611020134.png)

多重采样：
![multisample|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230611020234.png)

![multisample|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230611020324.png)
