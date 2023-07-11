动态Uniform常用于绘制多个Mesh，使用单个`Uniform Buffer`对象作为`Dynamic Uniform`，能够通过一个大的Uniform Buffer绘制具有不同矩阵的多个对象；对应的，将单个Uniform Buffer对应单个DescriptorSet的做法称为`Static Uniform`；

PS：如果需要的Dynamic Uniform超过8个，应该检查Physical Device的`maxDescriptorSetUniformBuffersDynamic`限制；

假如把CPU看作客户端，把GPU看作服务器端，那么`DescriptorSetLayout`即为**客户端的数据与服务器端的Shader之间通信的协议**，`DescriptorSet`即为**数据包**，协议会告诉Shader如何读取数据包中的二进制数据；

![descriptorSet|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230711125845.png)


# 优点

相比较与一个Uniform Buffer对应一个Descriptor的做法，Dynamic Uniform支持一个Uniform Buffer对应多个Descriptor，这种做法可以最大限度减少所需DescriptorSet的数量，并且部分更新数据有助于内存写入；

# 步骤

## Shader Binding

使用动态Uniform，Shader中的Binding无需修改；
```cpp
layout (binding = 1) uniform UboInstance
{
	mat4 model; 
} uboInstance;
```

## Uniform Buffer/Memory

使用指针而非vector，为了实现数据对齐；
```cpp
struct UboDataDynamic {
  glm::mat4 *model = nullptr;
} uboDataDynamic;
```

Step1、计算对齐字节数，需要是`minUniformBufferOffsetAlignment`的整数倍；
PS：此处的算法必须需要minUboAligment为2的整数次方才正确；

```cpp
void prepareUniformBuffers()
{
	// Calculate required alignment based on minimum device offset alignment
	size_t minUboAlignment = vulkanDevice->properties.limits.minUniformBufferOffsetAlignment;
	dynamicAlignment = sizeof(glm::mat4);
	if (minUboAlignment > 0) {
		dynamicAlignment = (dynamicAlignment + minUboAlignment - 1) & ~(minUboAlignment - 1);
	}
```





https://zhuanlan.zhihu.com/p/54667292
https://github.com/SaschaWillems/Vulkan/tree/master/examples/dynamicuniformbuffer
https://zhuanlan.zhihu.com/p/550376820