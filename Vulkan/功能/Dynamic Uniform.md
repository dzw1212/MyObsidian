动态Uniform常用于绘制多个Mesh，使用单个`Uniform Buffer`对象作为`Dynamic Uniform`，能够通过一个大的Uniform Buffer绘制具有不同矩阵的多个对象；对应的，将单个Uniform Buffer对应单个DescriptorSet的做法称为`Static Uniform`；

PS：如果需要的Dynamic Uniform超过8个，应该检查Physical Device的`maxDescriptorSetUniformBuffersDynamic`限制；

假如把CPU看作客户端，把GPU看作服务器端，那么`DescriptorSetLayout`即为**客户端的数据与服务器端的Shader之间通信的协议**，`DescriptorSet`即为**数据包**，协议会告诉Shader如何读取数据包中的二进制数据；

![descriptorSet|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230711125845.png)


# 优点

相比较与一个Uniform Buffer对应一个Descriptor的做法，Dynamic Uniform支持一个Uniform Buffer对应多个Descriptor，这种做法可以最大限度减少所需DescriptorSet的数量，并且部分更新数据有助于内存写入；

# 步骤

## Shader Binding

将需要Batch Instance的部分分离出来，作为Dynamic Uniform Buffer；
```cpp
layout (binding = 0) uniform UniformBufferObject
{
	mat4 view;
	mat4 proj;
} ubo;

layout (binding = 1) uniform DynamicUniformBufferObject
{
	mat4 model;
} dynamicUbo;

layout (binding = 2) uniform sampler2D texSampler;
```

## Uniform Buffer/Memory

使用指针而非vector，为了实现数据对齐；
```cpp
struct UniformBufferObject
{
	glm::mat4 view;
	glm::mat4 proj;
};

struct DynamicUniformBufferObject
{
	glm::mat4* model;
};
```

Step1、计算对齐字节数，需要是`minUniformBufferOffsetAlignment`的整数倍，根据计算得到的字计数创建整个`Uniform Buffer`；
PS：此处的算法必须需要minUboAligment为2的整数次方才正确；

```cpp
#define INSTANCE_NUM 7

auto physicalDeviceProperties = m_mapPhysicalDeviceInfo.at(m_PhysicalDevice).properties;
size_t minUboAlignment = physicalDeviceProperties.limits.minUniformBufferOffsetAlignment;
m_DynamicAlignment = sizeof(glm::mat4); //model
if (minUboAlignment > 0)
{
	m_DynamicAlignment = (m_DynamicAlignment + minUboAlignment - 1) & ~(minUboAlignment - 1);
}

VkDeviceSize dynamicUniformBufferSize = m_DynamicAlignment * INSTANCE_NUM;

m_DynamicUboData.model = (glm::mat4*)alignedAlloc(dynamicUniformBufferSize, m_DynamicAlignment);
ASSERT(m_DynamicUboData.model, "Allocate dynamic ubo data");

m_vecDynamicUniformBuffers.resize(m_vecSwapChainImages.size());
m_vecDynamicUniformBufferMemories.resize(m_vecSwapChainImages.size());

for (size_t i = 0; i < m_vecSwapChainImages.size(); ++i)
{
	CreateBufferAndBindMemory(dynamicUniformBufferSize, 
		VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT,
		VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT,
		m_vecDynamicUniformBuffers[i], 
		m_vecDynamicUniformBufferMemories[i]
	);
}
```

Step2、创建类型为`VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER_DYNAMIC`的`DescriptorPool`、`DescriptorSetLayout`和`DescriptorSet`；

```cpp
//ubo
VkDescriptorPoolSize uboPoolSize{};
uboPoolSize.type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
uboPoolSize.descriptorCount = static_cast<UINT>(m_vecSwapChainImages.size());

//dynamic ubo
VkDescriptorPoolSize dynamicUboPoolSize{};
dynamicUboPoolSize.type = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER_DYNAMIC;
dynamicUboPoolSize.descriptorCount = static_cast<UINT>(m_vecSwapChainImages.size());

//sampler
VkDescriptorPoolSize samplerPoolSize{};
samplerPoolSize.type = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
samplerPoolSize.descriptorCount = static_cast<UINT>(m_vecSwapChainImages.size());

std::vector<VkDescriptorPoolSize> vecPoolSize = {
	uboPoolSize,
	dynamicUboPoolSize,
	samplerPoolSize,
};

VkDescriptorPoolCreateInfo poolCreateInfo{};
poolCreateInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO;
poolCreateInfo.poolSizeCount = static_cast<UINT>(vecPoolSize.size());
poolCreateInfo.pPoolSizes = vecPoolSize.data();
poolCreateInfo.maxSets = static_cast<UINT>(m_vecSwapChainImages.size());

VULKAN_ASSERT(vkCreateDescriptorPool(m_LogicalDevice, &poolCreateInfo, nullptr, &m_DescriptorPool), "Create descriptor pool failed");
```

```cpp
//UniformBufferObject Binding
VkDescriptorSetLayoutBinding uboLayoutBinding{};
uboLayoutBinding.binding = 0; //对应Vertex Shader中的layout binding
uboLayoutBinding.descriptorCount = 1;
uboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
uboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT; //只需要在vertex stage生效
uboLayoutBinding.pImmutableSamplers = nullptr;

//Dynamic UniformBufferObject Binding
VkDescriptorSetLayoutBinding dynamicUboLayoutBinding{};
dynamicUboLayoutBinding.binding = 1; //对应Vertex Shader中的layout binding
dynamicUboLayoutBinding.descriptorCount = 1;
dynamicUboLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER_DYNAMIC;
dynamicUboLayoutBinding.stageFlags = VK_SHADER_STAGE_VERTEX_BIT; //只需要在vertex stage生效
dynamicUboLayoutBinding.pImmutableSamplers = nullptr;

//CombinedImageSampler Binding
VkDescriptorSetLayoutBinding samplerLayoutBinding{};
samplerLayoutBinding.binding = 2; ////对应Fragment Shader中的layout binding
samplerLayoutBinding.descriptorCount = 1;
samplerLayoutBinding.descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
samplerLayoutBinding.stageFlags = VK_SHADER_STAGE_FRAGMENT_BIT; //只用于fragment stage
samplerLayoutBinding.pImmutableSamplers = nullptr;

std::vector<VkDescriptorSetLayoutBinding> vecDescriptorLayoutBinding = {
	uboLayoutBinding,
	dynamicUboLayoutBinding,
	samplerLayoutBinding,
};

VkDescriptorSetLayoutCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO;
createInfo.bindingCount = static_cast<UINT>(vecDescriptorLayoutBinding.size());
createInfo.pBindings = vecDescriptorLayoutBinding.data();

VULKAN_ASSERT(vkCreateDescriptorSetLayout(m_LogicalDevice, &createInfo, nullptr, &m_DescriptorSetLayout), "Create descriptor layout failed");
```

```cpp
VkDescriptorSetAllocateInfo allocInfo{};
allocInfo.sType = VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO;
allocInfo.descriptorSetCount = static_cast<UINT>(m_vecSwapChainImages.size());
allocInfo.descriptorPool = m_DescriptorPool;

std::vector<VkDescriptorSetLayout> vecDupDescriptorSetLayout(m_vecSwapChainImages.size(), m_DescriptorSetLayout);
allocInfo.pSetLayouts = vecDupDescriptorSetLayout.data();

m_vecDescriptorSets.resize(m_vecSwapChainImages.size());
VULKAN_ASSERT(vkAllocateDescriptorSets(m_LogicalDevice, &allocInfo, m_vecDescriptorSets.data()), "Allocate desctiprot sets failed");

for (size_t i = 0; i < m_vecSwapChainImages.size(); ++i)
{
	//ubo
	VkDescriptorBufferInfo descriptorBufferInfo{};
	descriptorBufferInfo.buffer = m_vecUniformBuffers[i];
	descriptorBufferInfo.offset = 0;
	descriptorBufferInfo.range = sizeof(UniformBufferObject);

	VkWriteDescriptorSet uboWrite{};
	uboWrite.sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
	uboWrite.dstSet = m_vecDescriptorSets[i];
	uboWrite.dstBinding = 0;
	uboWrite.dstArrayElement = 0;
	uboWrite.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER;
	uboWrite.descriptorCount = 1;
	uboWrite.pBufferInfo = &descriptorBufferInfo;

	//dynamic ubo
	VkDescriptorBufferInfo dynamicDescriptorBufferInfo{};
	dynamicDescriptorBufferInfo.buffer = m_vecDynamicUniformBuffers[i];
	dynamicDescriptorBufferInfo.offset = 0;
	dynamicDescriptorBufferInfo.range = 64;

	VkWriteDescriptorSet dynamicUboWrite{};
	dynamicUboWrite.sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
	dynamicUboWrite.dstSet = m_vecDescriptorSets[i];
	dynamicUboWrite.dstBinding = 1;
	dynamicUboWrite.dstArrayElement = 0;
	dynamicUboWrite.descriptorType = VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER_DYNAMIC;
	dynamicUboWrite.descriptorCount = 1;
	dynamicUboWrite.pBufferInfo = &dynamicDescriptorBufferInfo;

	//sampler
	VkDescriptorImageInfo imageInfo{};
	imageInfo.imageLayout = VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL;
	imageInfo.imageView = m_TextureImageView;
	imageInfo.sampler = m_TextureSampler;

	VkWriteDescriptorSet samplerWrite{};
	samplerWrite.sType = VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET;
	samplerWrite.dstSet = m_vecDescriptorSets[i];
	samplerWrite.dstBinding = 2;
	samplerWrite.dstArrayElement = 0;
	samplerWrite.descriptorType = VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER;
	samplerWrite.descriptorCount = 1;
	samplerWrite.pImageInfo = &imageInfo;

	std::vector<VkWriteDescriptorSet> vecDescriptorWrite = {
		uboWrite,
		dynamicUboWrite,
		samplerWrite,
	};

	vkUpdateDescriptorSets(m_LogicalDevice, static_cast<UINT>(vecDescriptorWrite.size()), vecDescriptorWrite.data(), 0, nullptr);
}
```

Step3、使用`vkCmdBindDescriptorSets`分别填充每个实例的数据；

```cpp
for (UINT i = 0; i < INSTANCE_NUM; ++i)
{
	UINT uiDynamicOffset = i * static_cast<UINT>(m_DynamicAlignment);

	vkCmdBindDescriptorSets(commandBuffer,
		VK_PIPELINE_BIND_POINT_GRAPHICS, //descriptorSet并非Pipeline独有，因此需要指定是用于Graphic Pipeline还是Compute Pipeline
		m_GraphicPipelineLayout, //PipelineLayout中指定了descriptorSetLayout
		0,	//descriptorSet数组中第一个元素的下标 
		1,	//descriptorSet数组中元素的个数
		&m_vecDescriptorSets[uiIdx],
		1, //启用动态Uniform偏移
		&uiDynamicOffset	//指定动态Uniform的偏移
	);

	vkCmdDrawIndexed(commandBuffer, static_cast<UINT>(m_Indices.size()), 1, 0, 0, 0);
}
```

Step4、更新`Dynamic Uniform Buffer`；

```cpp
UniformBufferObject ubo{};
ubo.view = m_Camera.GetViewMatrix();
ubo.proj = m_Camera.GetProjMatrix();
//OpenGL与Vulkan的差异 - Y坐标是反的
ubo.proj[1][1] *= -1.f;

void* uniformBufferData;
vkMapMemory(m_LogicalDevice, m_vecUniformBufferMemories[uiIdx], 0, sizeof(ubo), 0, &uniformBufferData);
memcpy(uniformBufferData, &ubo, sizeof(ubo));
vkUnmapMemory(m_LogicalDevice, m_vecUniformBufferMemories[uiIdx]);

glm::mat4* pModelMat = nullptr;
for (UINT i = 0; i < INSTANCE_NUM; ++i)
{
	pModelMat = (glm::mat4*)(((uint64_t)m_DynamicUboData.model + (i * m_DynamicAlignment)));
	*pModelMat = glm::translate(glm::mat4(1.f), { i * 1.25f, 0.f, 0.f });
}

void* dynamicUniformBufferData;
vkMapMemory(m_LogicalDevice, m_vecDynamicUniformBufferMemories[uiIdx], 0, m_DynamicAlignment * INSTANCE_NUM, 0, &dynamicUniformBufferData);
memcpy(dynamicUniformBufferData, m_DynamicUboData.model, m_DynamicAlignment * INSTANCE_NUM);
vkUnmapMemory(m_LogicalDevice, m_vecDynamicUniformBufferMemories[uiIdx]);
```

PS：`Dynamic Uniform Buffer`一般不添加`VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`属性，而是使用以下方法手动刷新Buffer缓存，这么做的好处是支持部分更新Buffer，从而拥有更小的开销；
```cpp
VkMappedMemoryRange memoryRange {};
memoryRange.sType = VK_STRUCTURE_TYPE_MAPPED_MEMORY_RANGE;
memoryRange.memory = uniformBuffers.dynamic.memory;
memoryRange.size = sizeof(uboDataDynamic);
vkFlushMappedMemoryRanges(device, 1, &memoryRange);
```

# 效果展示

![instance|750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230715025335.png)


# 参考

https://zhuanlan.zhihu.com/p/54667292
https://github.com/SaschaWillems/Vulkan/tree/master/examples/dynamicuniformbuffer

