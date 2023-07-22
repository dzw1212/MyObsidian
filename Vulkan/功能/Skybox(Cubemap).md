
# 生成Cubemap

从网上搜索现成的`Cubemap`，或者使用工具进行转换；
比如使用`PVRTexTool`生成，可以通过6个面各自的贴图组合而成，也可以从一张大的贴图生成（要求宽高比为2：1）；

![PVRTexTool1|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230722232849.png)

![PVRTexTool2|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230722232933.png)

最后导出之前记得`Encode`成你的渲染器所需的格式；

# Skybox思路

`Skybox`的做法很简单，就是将一张`Cubemap`映射到一个正方体的内壁上，将摄像机置于正方体中心，同时
1. 禁用深度测试，使`Cubemap`的深度为1（最远）；
2. 对立方体屏蔽摄像机的平移分量；

具体到`vulkan`中，需要：
1. 载入立方体模型与`Cubemap`贴图；
2. 创建用于`Skybox`的`Shader`、`UniformBuffer`、`DescriptorPool/Set/SetLayout`，`Pipeline`；
3. 在渲染场景物体之前先渲染`Skybox`；

# 具体做法

## 载入Cubemap

设置`Image`的imageType为`VK_IMAGE_TYPE_2D`，flags为`VK_IMAGE_CREATE_CUBE_COMPATIBLE_BIT`；
设置`ImageView`的类型为`VK_IMAGE_VIEW_TYPE_CUBE`；

```cpp
ktxResult result;
ktxTexture* ktxTexture;
result = ktxTexture_CreateFromNamedFile(texture.m_Filepath.string().c_str(), KTX_TEXTURE_CREATE_LOAD_IMAGE_DATA_BIT, &ktxTexture);
ASSERT(result == KTX_SUCCESS, std::format("KTX load image {} failed", texture.m_Filepath.string()));

ASSERT(ktxTexture->glFormat == GL_RGBA);
ASSERT(ktxTexture->numDimensions == 2);

ktx_uint8_t* ktxTextureData = ktxTexture_GetData(ktxTexture);
texture.m_Size = ktxTexture_GetSize(ktxTexture);

texture.m_uiWidth = ktxTexture->baseWidth;
texture.m_uiHeight = ktxTexture->baseHeight;
texture.m_uiMipLevelNum = ktxTexture->numLevels;
texture.m_uiLayerNum = ktxTexture->numLayers;
texture.m_uiFaceNum = ktxTexture->numFaces;

if (texture.IsTextureArray())
{
	auto& physicalDeviceInfo = m_mapPhysicalDeviceInfo.at(m_PhysicalDevice);
	UINT uiMaxLayerNum = physicalDeviceInfo.properties.limits.maxImageArrayLayers;
	ASSERT(texture.m_uiLayerNum <= uiMaxLayerNum, std::format("TextureArray {} layout count {} exceed max limit {}", texture.m_Filepath.string(), texture.m_uiLayerNum, uiMaxLayerNum));
}

CreateImageAndBindMemory(texture.m_uiWidth, texture.m_uiHeight, texture.m_uiMipLevelNum, texture.m_uiLayerNum, texture.m_uiFaceNum,
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
	texture.m_uiFaceNum,					//faces
	VK_IMAGE_LAYOUT_UNDEFINED,				//src layout
	VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL);	//dst layout

TransferImageDataByStageBuffer(ktxTextureData, texture.m_Size, texture.m_Image, texture.m_uiWidth, texture.m_uiHeight, texture, ktxTexture);

ktxTexture_Destroy(ktxTexture);
```

```cpp
//填充iamge数据
//...
std::vector<VkBufferImageCopy> vecBufferCopyRegions;
for (UINT face = 0; face < texture.m_uiFaceNum; ++face)
{
	for (UINT layer = 0; layer < texture.m_uiLayerNum; ++layer)
	{
		for (UINT mipLevel = 0; mipLevel < texture.m_uiMipLevelNum; ++mipLevel)
		{
			size_t offset;
			KTX_error_code ret = ktxTexture_GetImageOffset(pKtxTexture, mipLevel, layer, face, &offset);
			ASSERT(ret == KTX_SUCCESS);
			VkBufferImageCopy bufferCopyRegion = {};
			bufferCopyRegion.imageSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
			bufferCopyRegion.imageSubresource.mipLevel = mipLevel;
			bufferCopyRegion.imageSubresource.baseArrayLayer = (texture.m_uiFaceNum > 1) ? (face + layer * 6) : (layer);
			bufferCopyRegion.imageSubresource.layerCount = 1;
			bufferCopyRegion.imageExtent.width = uiWidth >> mipLevel;
			bufferCopyRegion.imageExtent.height = uiHeight >> mipLevel;
			bufferCopyRegion.imageExtent.depth = 1;
			bufferCopyRegion.bufferOffset = offset;
			vecBufferCopyRegions.push_back(bufferCopyRegion);
		}
	}
}

vkCmdCopyBufferToImage(singleTimeCommandBuffer, stagingBuffer, image, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL,
	static_cast<UINT>(vecBufferCopyRegions.size()), vecBufferCopyRegions.data());
```

需要注意的是，在`Vulkan`中使用`Cubemap`有一条潜规则 —— 一般来说`Cubemap`的`Face`数量为6，`Layer`数量为1，在创建`Image`、`ImageView`、`VkImageSubresourceRange`、`VkBufferImageCopy`的时候，要将`LayerCount`的值填写为`FaceCount` —— 也就是说将`Face`当成`Layer`看待；

## 载入立方体模型

这个没什么困难的，略过；

## Skybox 着色器

```cpp
//vert
#version 450

layout (location = 0) in vec3 inPosition;

layout (location = 0) out vec3 outTexCoord;

layout (binding = 0) uniform UniformBufferObject
{
	mat4 modelView;
	mat4 proj;
} ubo;

void main() {
	outTexCoord = inPosition;

    gl_Position = ubo.proj * ubo.modelView * vec4(inPosition, 1.0);
}
```

```cpp
//frag
#version 450

layout (location = 0) in vec3 inTexCoord;

layout (location = 0) out vec4 outColor;

layout (binding = 1) uniform samplerCube texSamplerCubeMap; //注意类型为samplerCube

void main() {
    outColor = texture(texSamplerCubeMap, inTexCoord);
}
```

## 创建UniformBuffer/Descriptor相关资源

这个也没什么问题，略过；

## 创建Skybox Pipeline

渲染管线的大部分与场景物体的渲染管线一致，不同的地方有：
1. 要绑定的`ShaderModule`不同（`Skybox`有单独的着色器）；
2. `Raserization State`中，设置剔除模式为`VK_CULL_MODE_FRONT_BIT`，因为`Skybox`不需要从外部可见；
3. `Depth Stencil State`中，禁用深度检测与深度写入功能；

```cpp
depthStencilStateCreateInfo.depthTestEnable = VK_FALSE; //作为背景，始终在最远处，不进行深度检测
depthStencilStateCreateInfo.depthWriteEnable = VK_FALSE;
```

4. `Color Blend State`中，禁用颜色混合；

```cpp
colorBlendAttachment.blendEnable = VK_FALSE;
```

## 渲染

在渲染场景物体之前，
```cpp
if (m_bEnableSkybox)
{
	vkCmdBindPipeline(commandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, m_SkyboxGraphicPipeline);
	VkBuffer skyboxVertexBuffers[] = {
		m_SkyboxModel.m_VertexBuffer,
	};
	VkDeviceSize skyboxOffsets[]{ 0 };
	vkCmdBindVertexBuffers(commandBuffer, 0, 1, skyboxVertexBuffers, skyboxOffsets);
	vkCmdBindIndexBuffer(commandBuffer, m_SkyboxModel.m_IndexBuffer, 0, VK_INDEX_TYPE_UINT32);
	vkCmdBindDescriptorSets(commandBuffer,
		VK_PIPELINE_BIND_POINT_GRAPHICS,
		m_SkyboxGraphicPipelineLayout,
		0, 1,
		&m_vecSkyboxDescriptorSets[uiIdx],
		0, NULL);

	vkCmdDrawIndexed(commandBuffer, static_cast<UINT>(m_SkyboxModel.m_vecIndices.size()), 1, 0, 0, 0);
}
```

同时更新`UniformBuffer`数据：
```cpp

struct SkyboxUniformBufferObject
{
	glm::mat4 modelView;
	glm::mat4 proj;
};
//-------------
m_SkyboxUboData.modelView = m_Camera.GetViewMatrix();
m_SkyboxUboData.modelView[3] = glm::vec4(0.0f, 0.0f, 0.0f, 1.0f); //移除平移分量
m_SkyboxUboData.proj = m_Camera.GetProjMatrix();
//OpenGL与Vulkan的差异 - Y坐标是反的
m_SkyboxUboData.proj[1][1] *= -1.f;

void* uniformBufferData;
vkMapMemory(m_LogicalDevice, m_vecSkyboxUniformBufferMemories[uiIdx], 0, sizeof(SkyboxUniformBufferObject), 0, &uniformBufferData);
memcpy(uniformBufferData, &m_SkyboxUboData, sizeof(SkyboxUniformBufferObject));
vkUnmapMemory(m_LogicalDevice, m_vecSkyboxUniformBufferMemories[uiIdx]);
```

# 效果

![Skybox|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230722232221.png)

## 踩坑点

1. `Vulakn`中将`Cubemap`的`Face`视为`Layer`；
2. `Skybox`的渲染要先于场景物体的渲染；