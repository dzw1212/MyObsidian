# MyObsidian
for sync

```cpp
	model.m_vecImages.resize(gltfModel.images.size());
	for (size_t i = 0; i < gltfModel.images.size(); ++i)
	{
		tinygltf::Image& gltfImage = gltfModel.images[i];
		UCHAR* imageBuffer = nullptr;
		size_t imageBufferSize = 0;
		bool bDeleteBuffer = false;
		if (gltfImage.component == 3) //RGB 需要转为RGBA
		{
			imageBufferSize = gltfImage.width * gltfImage.height * 4;
			imageBuffer = new UCHAR[imageBufferSize];
			UCHAR* rgbaPtr = imageBuffer;
			UCHAR* rgbPtr = &gltfImage.image[0];
			for (size_t p = 0; p < gltfImage.width * gltfImage.height; ++p)
			{
				rgbaPtr[0] = rgbPtr[0]; // R
				rgbaPtr[1] = rgbPtr[1]; // G
				rgbaPtr[2] = rgbPtr[2]; // B
				rgbaPtr[3] = 255;       // A

				rgbaPtr += 4;
				rgbPtr += 3;
			}
			bDeleteBuffer = true;
		}
		else if (gltfImage.component == 4) //RGBA
		{
			imageBuffer = &gltfImage.image[0];
			imageBufferSize = gltfImage.image.size();
			bDeleteBuffer = false;
		}
		else
		{
			ASSERT(false, "Unsupport gltf image type");
		}

		auto& image = model.m_vecImages[i];
		image.uiWidth = static_cast<UINT>(gltfImage.width);
		image.uiHeight = static_cast<UINT>(gltfImage.height);
		image.strName = gltfImage.uri;

		CreateImageAndBindMemory(image.uiWidth, image.uiHeight,
			1, 1, 1,
			VK_SAMPLE_COUNT_1_BIT,
			VK_FORMAT_R8G8B8A8_SRGB,
			VK_IMAGE_TILING_OPTIMAL,
			VK_IMAGE_USAGE_TRANSFER_SRC_BIT | VK_IMAGE_USAGE_TRANSFER_DST_BIT | VK_IMAGE_USAGE_SAMPLED_BIT,
			VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT,
			image.Image, image.Memory);

		//copy之前，将layout从初始的undefined转为transfer dst
		ChangeImageLayout(image.Image,
			VK_FORMAT_R8G8B8A8_SRGB,				//image format
			1,										//mipmap levels
			1,										//layers
			1,										//faces
			VK_IMAGE_LAYOUT_UNDEFINED,				//src layout
			VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL);	//dst layout

		TransferImageDataByStageBuffer(imageBuffer, imageBufferSize, image.Image, image.uiWidth, image.uiHeight, texture, nullptr);


	}
```