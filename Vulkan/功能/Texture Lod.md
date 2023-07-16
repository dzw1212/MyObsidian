此处使用ktx库载入ktx图片；

创建`Image`，以及对应的`ImageView`，`ImageMemory`：
```cpp
ktxResult result;
ktxTexture* ktxTexture;
result = ktxTexture_CreateFromNamedFile(texture.m_Filepath.string().c_str(), KTX_TEXTURE_CREATE_LOAD_IMAGE_DATA_BIT, &ktxTexture);
ASSERT(result == KTX_SUCCESS, std::format("KTX load image {} failed", texture.m_Filepath.string()));

ktx_uint8_t* ktxTextureData = ktxTexture_GetData(ktxTexture);
texture.m_Size = ktxTexture_GetSize(ktxTexture);

texture.m_uiWidth = ktxTexture->baseWidth;
texture.m_uiHeight = ktxTexture->baseHeight;
texture.m_uiMipLevels = ktxTexture->numLevels;

CreateImageAndBindMemory(texture.m_uiWidth, texture.m_uiHeight, texture.m_uiMipLevels,
	VK_SAMPLE_COUNT_1_BIT,
	VK_FORMAT_R8G8B8A8_SRGB,
	VK_IMAGE_TILING_OPTIMAL,
	VK_IMAGE_USAGE_TRANSFER_SRC_BIT | VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT,
	VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT,
	texture.m_Image, texture.m_Memory);

//copy之前，将layout从初始的undefined转为transfer dst
ChangeImageLayout(texture.m_Image,
	VK_FORMAT_R8G8B8A8_SRGB,				//image format
	texture.m_uiMipLevels,					//mipmap levels
	VK_IMAGE_LAYOUT_UNDEFINED,				//src layout
	VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL);	//dst layout

std::vector<size_t> vecMipOffset;
for (UINT i = 0; i < texture.m_uiMipLevels; ++i)
{
	ktx_size_t offset;
	KTX_error_code ret = ktxTexture_GetImageOffset(ktxTexture, i, 0, 0, &offset);
	ASSERT(ret == KTX_SUCCESS, "KTX get image offset failed");
	vecMipOffset.push_back(offset);
}

TransferImageDataByStageBuffer(ktxTextureData, texture.m_Size, texture.m_Image, texture.m_uiWidth, texture.m_uiHeight, vecMipOffset);

ktxTexture_Destroy(ktxTexture);

//transfer之后，将layout从transfer dst转为shader readonly
ChangeImageLayout(texture.m_Image,
	VK_FORMAT_R8G8B8A8_SRGB,
	texture.m_uiMipLevels,
	VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL,
	VK_IMAGE_LAYOUT_SHADER_READ_ONLY_OPTIMAL);

texture.m_ImageView = CreateImageView(texture.m_Image,
	VK_FORMAT_R8G8B8A8_SRGB,	//格式为sRGB
	VK_IMAGE_ASPECT_COLOR_BIT,	//aspectFlags为COLOR_BIT
	texture.m_uiMipLevels
);
```

```cpp
void VulkanRenderer::TransferImageDataByStageBuffer(void* pData, VkDeviceSize imageSize, VkImage& image, UINT uiWidth, UINT uiHeight, std::vector<size_t>& vecMipOffset)
{
	VkBuffer stagingBuffer;
	VkDeviceMemory stagingBufferMemory;

	CreateBufferAndBindMemory(imageSize, VK_BUFFER_USAGE_TRANSFER_SRC_BIT,
		VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT,
		stagingBuffer, stagingBufferMemory);

	void* imageData;
	vkMapMemory(m_LogicalDevice, stagingBufferMemory, 0, imageSize, 0, (void**)&imageData);
	memcpy(imageData, pData, static_cast<size_t>(imageSize));
	vkUnmapMemory(m_LogicalDevice, stagingBufferMemory);

	VkCommandBuffer singleTimeCommandBuffer = BeginSingleTimeCommand();

	std::vector<VkBufferImageCopy> vecBufferCopyRegions;
	for (UINT i = 0; i < vecMipOffset.size(); i++) 
	{
		VkBufferImageCopy bufferCopyRegion = {};
		bufferCopyRegion.imageSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
		bufferCopyRegion.imageSubresource.mipLevel = i;
		bufferCopyRegion.imageSubresource.baseArrayLayer = 0;
		bufferCopyRegion.imageSubresource.layerCount = 1;
		bufferCopyRegion.imageExtent.width = uiWidth >> i;
		bufferCopyRegion.imageExtent.height = uiHeight >> i;
		bufferCopyRegion.imageExtent.depth = 1;
		bufferCopyRegion.bufferOffset = vecMipOffset[i];
		vecBufferCopyRegions.push_back(bufferCopyRegion);
	}

	vkCmdCopyBufferToImage(singleTimeCommandBuffer, stagingBuffer, image, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL, 
		static_cast<UINT>(vecBufferCopyRegions.size()), vecBufferCopyRegions.data());

	EndSingleTimeCommand(singleTimeCommandBuffer);

	vkDestroyBuffer(m_LogicalDevice, stagingBuffer, nullptr);
	vkFreeMemory(m_LogicalDevice, stagingBufferMemory, nullptr);
}
```

创建`ImageSampler`的过程中指定`Lod`：
```cpp
//设置mipmap相关参数
createInfo.mipmapMode = VK_SAMPLER_MIPMAP_MODE_LINEAR;
createInfo.mipLodBias = 0.f;
createInfo.minLod = 0.f;
createInfo.maxLod = static_cast<float>(m_Texture.m_uiMipLevels);
```

在`UniformBuffer`中新增一个Lod变量，用来指定Lod等级（临时做法）；
```cpp
struct UniformBufferObject
{
	glm::mat4 view;
	glm::mat4 proj;
	float lod;
};
```

修改Shader，在`texture()`采样中指定Lod等级；
```cpp
//vert
layout (location = 2) out float textureLod;

layout (binding = 0) uniform UniformBufferObject
{
	mat4 view;
	mat4 proj;
	float lod;
} ubo;

textureLod = ubo.lod;

//frag
layout(location = 2) in float textureLod;

outColor = texture(texSampler, fragTexCoord, textureLod);
```

效果：
![textureLod|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230716174422.png)

![textureLod2|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230716174504.png)
