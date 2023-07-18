`KTX（Khronos Texture）`是一个轻量级的纹理容器，用于OpenGL®、Vulkan®和其他GPU API。KTX文件包含纹理加载所需的所有参数。**单个文件**可以包含从简单的基底2D纹理到带mipmaps的立方体贴图阵列纹理的任何内容。包含的纹理可以是Basis Universal格式、OpenGL系列和Vulkan API及扩展支持的任何块压缩格式或未压缩的单平面格式；

https://github.com/KhronosGroup/KTX-Software

# 整合

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

# 使用流程

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

# 其他格式转ktx

## toktx

使用khronos官方的转换工具`toktx` -- 从 JPEG、PNG 或 netpbm 格式文件创建 KTX 文件；
编译`ktx-software`工程中的`toktx`项目，生成`toktx.exe`；
再根据文档中提供的命令行参数进行操作；

https://github.khronos.org/KTX-Software/ktxtools/toktx.html


## PVRTexTool

鉴于命令行方式太过繁琐与不直观，更推荐使用`Imagination`公司的`PVRTexTool`工具进行转换；
https://developer.imaginationtech.com/downloads/
PS：下载需要先登录，注册时不要使用QQ邮箱避免收不到验证码；

![PVRTexTool|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230718232504.png)

使用该工具可以方便地生成各种类型的ktx贴图；
![PVRTexTool2|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230718232815.png)

重新编码为所需的格式：
![PVRTexTool3|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230719013633.png)

