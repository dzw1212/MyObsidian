
# 纹理坐标 TexCoord

![texcoord|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230131090038.png)

# 纹理环绕 Texture Wrap

当纹理坐标超出(0.0, 1.0)的范围，就会根据指定的纹理环绕方式来绘制纹理；

![wrap|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230131090236.png)

```c++
glTextureParameteri(m_uiTextureId, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTextureParameteri(m_uiTextureId, GL_TEXTURE_WRAP_T, GL_REPEAT);

//如果选择的是clamp To Border方式，还需要指定border颜色
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

# 纹理过滤 Texture Filter

纹理坐标不依赖于分辨率，可以是任意浮点值，因此当纹理像素Texture Pixel(纹素，Texel)与屏幕分辨率不是一一对应时，会根据指定的纹理过滤方式来选择要显示的颜色；

`GL_NEAREST`，邻近过滤，选择最接近纹理坐标的像素颜色；

![nearest|180](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230131091301.png)

`GL_LINEAR`，线性过滤，使用双线性插值得到混合颜色；

![linear|180](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230131091420.png)

一般来说，线性过滤的效果比邻近过滤更平滑、锯齿感更少；

![nearestVSlinear|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230131091651.png)

```c++
glTextureParameteri(m_uiTextureId, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTextureParameteri(m_uiTextureId, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

# 多级渐远纹理 Mipmap

预处理一张纹理，使其生成多个分辨率大小不同的子纹理，称为Mipmap，用来应对分辨率不同的场景；

![mipmap|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230131092137.png)

对于不同层级的Mipmap纹理之间的过滤，有特殊的纹理过滤方式；
|过滤方式|描述|
|:-|:-|
|`GL_NEAREST_MIPMAP_NEAREST`|使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样|
|`GL_LINEAR_MIPMAP_NEAREST`|使用最邻近的多级渐远纹理级别，并使用线性插值进行采样|
|`GL_NEAREST_MIPMAP_LINEAR`|在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样|
|`GL_LINEAR_MIPMAP_LINEAR`|在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样|

```c++
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
```

# 加载与卸载图像

## 使用stb_image库

```c++
int nWidth, nHeight, nChannel;

stbi_set_flip_vertically_on_load(1); //载入图像时翻转Y轴

stbi_uc* imageData = nullptr;
{
	imageData = stbi_load(strPath.c_str(), &nWidth, &nHeight, &nChannel, 0);
	CORE_ASSERT(imageData, "Failed to load image");
}

//数据使用完毕后
stbi_image_free(imageData);
```

# 创建纹理

```c++
glCreateTextures(GL_TEXTURE_2D, 1, &m_uiTextureId);
```

# 生成纹理

```c++
glTexImage2D(GL_TEXTURE_2D, //指定纹理目标，生成与当前绑定的纹理对象在同一目标的纹理
		     0,        //mipmap的级别 
			 GL_RGB,   //纹理存储格式
			 width,    //纹理宽度
			 height,   //纹理高度
			 0,        //保留位
			 GL_RGB,   //源图像的格式
			 GL_UNSIGNED_BYTE, //源图像的数据类型
			 imageData);  //源图像数据
```


# 激活纹理单元

不同显示支持不同数量的纹理单元，一般都支持至少32个；
如果没有显式激活纹理单元，则默认激活0；

```c++
//在绑定纹理之前先激活纹理单元
glActiveTexture(GL_TEXTURE0); 
```

# 绑定纹理

```c++
glBindTexture(GL_TEXTURE_2D, m_uiTextureId);
```

# 设置各种纹理参数

```c++
glTextureParameteri(m_uiId, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTextureParameteri(m_uiId, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTextureParameteri(m_uiId, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTextureParameteri(m_uiId, GL_TEXTURE_WRAP_T, GL_REPEAT);
```

# 使用纹理

```c++
//Fragment Shader中，传入图像的slot号
uniform sampler2D textureSlot;

//使用GLSL内建的texture()函数进行纹理采样
color = texture(textureSlot, texCoord);

```