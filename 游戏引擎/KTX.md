`KTX（Khronos Texture）`是一个轻量级的纹理容器，用于OpenGL®、Vulkan®和其他GPU API。KTX文件包含纹理加载所需的所有参数。**单个文件**可以包含从简单的基底2D纹理到带mipmaps的立方体贴图阵列纹理的任何内容。包含的纹理可以是Basis Universal格式、OpenGL系列和Vulkan API及扩展支持的任何块压缩格式或未压缩的单平面格式。Basis Universal目前包含两种格式，可快速转码为任何GPU支持的格式： LZ/ETC1S（结合了块压缩和超压缩）和UASTC（一种块压缩格式）。LZ/ETC1S以外的格式可使用Zstd和ZLIB进行超级压缩。

https://github.com/KhronosGroup/KTX-Software

- Include目录：
```
KTX\include
KTX\lib
KTX\other_include\KHR
```

- 参与编译的文件：
```
checkheader.c
filestream.c
hashlist.c
memstream.c
swap.c
texture.c
```

基本使用流程：

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

//...

ktxTexture_Destroy(ktxTexture);
```