
# 优点

相对于载入多张贴图，载入一张由多个texture合成的贴图，能够显著减少对GPU有限的Texture寄存器的依赖；
GPU支持的`TextureArray`中的图片的数量也是有限的，可以通过`VkPhysicalDeviceLimits.maxImageArrayLayers`查询；

# 步骤

使用`ktx`库读取贴图的层级数：
```cpp
texture.m_uiLayerNum = ktxTexture->numLayers;

if (texture.m_uiLayerNum > 1)
{
	auto& physicalDeviceInfo = m_mapPhysicalDeviceInfo.at(m_PhysicalDevice);
	UINT uiMaxLayerNum = physicalDeviceInfo.properties.limits.maxImageArrayLayers;
	ASSERT(texture.m_uiLayerNum <= uiMaxLayerNum);
}
```

自此，引入了`LayerCount`的概念，在创建`Image`、`ImageView`、`ImageMemoryBarrier`等贴图相关资源的时候都需要填入对应的层级数；
```cpp

CreateImageAndBindMemory(texture.m_uiWidth, texture.m_uiHeight, texture.m_uiMipLevelNum, texture.m_uiLayerNum,
	VK_SAMPLE_COUNT_1_BIT,
	VK_FORMAT_R8G8B8A8_SRGB,
	VK_IMAGE_TILING_OPTIMAL,
	VK_IMAGE_USAGE_TRANSFER_SRC_BIT | VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT,
	VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT,
	texture.m_Image, texture.m_Memory);

//copy之前，将layout从初始的undefined转为transfer dst
ChangeImageLayout(texture.m_Image,
	VK_FORMAT_R8G8B8A8_SRGB,				//image format
	texture.m_uiMipLevelNum,				//mipmap levels
	texture.m_uiLayerNum,					//layers
	VK_IMAGE_LAYOUT_UNDEFINED,				//src layout
	VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL);	//dst layout
```

类似于`MipLevel`，在`vkCmdCopyBufferToImage`的时候也要逐层添加`VkBufferImageCopy`；
```cpp
std::vector<VkBufferImageCopy> vecBufferCopyRegions;

for (UINT layer = 0; layer < texture.m_uiLayerNum; ++layer)
{
	for (UINT mipLevel = 0; mipLevel < texture.m_uiMipLevelNum; ++mipLevel)
	{
		size_t offset;
		KTX_error_code ret = ktxTexture_GetImageOffset(pKtxTexture, mipLevel, layer, 0, &offset);
		ASSERT(ret == KTX_SUCCESS);
		VkBufferImageCopy bufferCopyRegion = {};
		bufferCopyRegion.imageSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
		bufferCopyRegion.imageSubresource.mipLevel = mipLevel;
		bufferCopyRegion.imageSubresource.baseArrayLayer = layer;
		bufferCopyRegion.imageSubresource.layerCount = 1;
		bufferCopyRegion.imageExtent.width = uiWidth >> mipLevel;
		bufferCopyRegion.imageExtent.height = uiHeight >> mipLevel;
		bufferCopyRegion.imageExtent.depth = 1;
		bufferCopyRegion.bufferOffset = offset;
		vecBufferCopyRegions.push_back(bufferCopyRegion);
	}
}

vkCmdCopyBufferToImage(singleTimeCommandBuffer, stagingBuffer, image, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL,
			static_cast<UINT>(vecBufferCopyRegions.size()), vecBufferCopyRegions.data());
```


在创建`ImageView`时，设置类型为`VK_IMAGE_VIEW_TYPE_2D_ARRAY`，取代原来的`VK_IMAGE_VIEW_TYPE_2D`；
```cpp
texture.m_ImageView = CreateImageView(texture.m_Image, VK_IMAGE_VIEW_TYPE_2D_ARRAY,
		VK_FORMAT_R8G8B8A8_SRGB,	//格式为sRGB
		VK_IMAGE_ASPECT_COLOR_BIT,	//aspectFlags为COLOR_BIT
		texture.m_uiMipLevelNum,
		texture.m_uiLayerNum
```

对应的，着色器中将采样器类型由`sampler2D`改为`sampler2DArray`；
```cpp
layout (binding = 2) uniform sampler2DArray texSampler;
```


`TextureArray`已经传递给了着色器，将2D的多层贴图想象成一个3D的贴图，贴图的层级也就对应这个3D贴图的`z`值，采样函数`texture()`支持采样3D纹理，因此基本的想法是给每个Instance传递一个对应贴图的Layer（为了方便理解之后统称为贴图的索引），作为uv坐标的z值参与采样；传递方式选择`DynamicUniform`；
[[Dynamic Uniform]]
```cpp
//vert
layout (location = 3) out vec3 outUV;

layout (binding = 1) uniform DynamicUniformBufferObject
{
	mat4 model;
	float textureIndex;
} dynamicUbo;

outUV = vec3(inTexCoord, dynamicUbo.textureIndex);
```

```cpp
//frag
layout(location = 3) in vec3 inUV;

outColor = texture(texSampler, inUV, textureLod);
```

程序端修改`DynamicUniformBufferObject`结构体的定义，重新计算对齐字节，谨慎计算偏移并更新UniformBuffer；
```cpp
struct DynamicUniformBufferObject
{
	glm::mat4* model;
	float* fTextureIndex;
};
```

```cpp
//创建dynamic uniform buffer
auto physicalDeviceProperties = m_mapPhysicalDeviceInfo.at(m_PhysicalDevice).properties;
size_t minUboAlignment = physicalDeviceProperties.limits.minUniformBufferOffsetAlignment;
m_DynamicAlignment = sizeof(glm::mat4) + sizeof(float);
if (minUboAlignment > 0)
{
	m_DynamicAlignment = (m_DynamicAlignment + minUboAlignment - 1) & ~(minUboAlignment - 1);
}

m_DynamicUboBufferSize = m_DynamicAlignment * INSTANCE_NUM;

m_DynamicUboData.model = (glm::mat4*)alignedAlloc(m_DynamicUboBufferSize, m_DynamicAlignment);
m_DynamicUboData.fTextureIndex = (float*)((size_t)m_DynamicUboData.model + sizeof(glm::mat4));
```

```cpp
//更新dynamic uniform buffer
glm::mat4* pModelMat = nullptr;
float* pTextureIdx = nullptr;
for (UINT i = 0; i < INSTANCE_NUM; ++i)
{
	pModelMat = (glm::mat4*)((size_t)m_DynamicUboData.model + (i * m_DynamicAlignment));
	*pModelMat = glm::translate(glm::mat4(1.f), { ((float)i - (float)(INSTANCE_NUM / 2)) * 1.25f, 0.f, 0.f });
	pTextureIdx = (float*)((size_t)m_DynamicUboData.fTextureIndex + (i * m_DynamicAlignment));
	*pTextureIdx = (float)(i);
}

void* dynamicUniformBufferData;
vkMapMemory(m_LogicalDevice, m_vecDynamicUniformBufferMemories[uiIdx], 0, m_DynamicUboBufferSize, 0, &dynamicUniformBufferData);
memcpy(dynamicUniformBufferData, m_DynamicUboData.model, m_DynamicUboBufferSize);
vkUnmapMemory(m_LogicalDevice, m_vecDynamicUniformBufferMemories[uiIdx]);
```

# 效果

![](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230718010118.png)

# 参考

http://xiaopengyou.fun/public/2019/08/19/14_TextureArray/