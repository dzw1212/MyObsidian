# 实现思想
将摄像机暂时移动到光源位置，应用`z-buffer`的隐藏面消除算法HSR，使用生成的深度信息计算阴影；
当像素处于阴影中时，一种简单而有效的方法是仅渲染其环境光，忽略漫反射光和高光；
当使用纹理对象储存阴影深度信息时，称为阴影纹理；

优点：简单，有强大的硬件支持；
缺点：不够准确，容易产生伪影；

# ShadowMap的分辨率

`Shadow Map`的分辨率并不需要与窗口（或者说视口）的分辨率一致。实际上，阴影映射的分辨率是可以独立调整的，这是因为阴影映射是在光源的视角下进行渲染的，而不是在相机的视角下；

阴影映射的分辨率会影响阴影的质量和性能 —— 分辨率越高，阴影的细节就越清晰，但是渲染的性能开销也越大；分辨率越低，阴影的细节就越模糊，但是渲染的性能开销就越小；因此需要在阴影的质量和性能之间找到一个平衡；

至于为什么阴影映射通常是正方形，这是因为光源的视角通常是一个全方位的视角，也就是说，光源可以照射到所有的方向。如果阴影映射是一个矩形，那么在某些方向上，阴影的质量就会比其他方向差。因此，为了保证所有方向上阴影的质量都一样，阴影映射通常是一个正方形；

当然，这并不是一个硬性的规定，你完全可以根据你的需求来选择阴影映射的形状和分辨率。例如，如果你的游戏场景是一个长方形的，那么你可能就会选择一个长方形的阴影映射。或者，如果你的游戏只在一个小范围内需要阴影，那么你可能就会选择一个小的阴影映射。总的来说，阴影映射的形状和分辨率应该根据你的具体需求来选择；

# 存储通道

在阴影映射（Shadow Mapping）中，我们需要存储深度信息，这是从光源的视角看到的每个像素的深度。由于深度信息是一个单通道的数据（只有一个值），所以我们只需要使用纹理的一个通道来存储它。通常，我们选择红色（R）通道来存储深度信息，这是因为在大多数图形API中，红色通道是第一个被处理的通道，所以使用红色通道可以稍微提高一点性能；
# 步骤
1. 将摄像机移动到光源位置然后渲染场景，填充深度缓冲区；
   不必为像素生成颜色，因此片段着色器不做任何事情；
```c
//Vertex Shader
#version 430
layout(location=0) in vec3 vertPos;
uniform mat4 shadowMVP;

void main()
{
	gl_Position = shadowMVP * vec4(vertPos, 1.0);
}
```
```c
//Fragment Shader
#version 430
void main() {}
```

2. 将z-buffer复制到纹理，生成阴影纹理；
   Ⅰ. 生成空的阴影纹理，然后使用`glCopyTexImage2D`将z-buffer复制到其中；
   Ⅱ. 在Step 1中自定义一个缓冲区，然后使用`glFrameBufferTexture`将阴影纹理附加到其上；

3. 渲染带阴影的场景；
	两个MVP矩阵：
		一个是将模型从局部空间转为裁剪空间的标准MVP矩阵；
		一个是用于从阴影纹理中查找深度信息的shadowMVP；
	需要注意的是，**OpenGL**中使用[-1 ... +1]的坐标，纹理贴图使用[0 ... 1]空间，因此需要一个额外的转换矩阵`B`，用于从摄像机空间转为纹理空间；
	$$
	B = \left[
	\begin{aligned}
	0.5&&0&&0&&0.5\\
	0&&0.5&&0&&0.5\\
	0&&0&&0.5&&0.5\\
	0&&0&&0&&1
	\end{aligned}
	\right]
	$$
	<center>shadowMVP2 = [B][shadowMVP(pass1)]</center>
```c
//Vertex Shader
void main()
{
	vVertPos = (mv_matrix * vec4(vertPos, 1.0)).xyz;
	vLightDir = light.position - vVertPos;
	vNormal = (norm_matrix * vec4(vertNormal, 1.0)).xyz;
	vHalfVec = (vLightDir - vVertPos).xyz;

	shadow_coord = shadowMVP * vec4(vertPos, 1.0);

	gl_Position = proj_matrix * mv_matrix * vec4(vertPos, 1.0);
}
```
```c
//Fragment Shader
layout(binding=0) uniform sampler2DShadow shadowTex;

void main()
{
	vec3 L = normalize(vLightDir);
	vec3 N = normalize(vNormal);
	vec3 V = normalize(-vVertPos);
	vec3 H = normalize(vHalfVec);

	float inShadow = textureProj(shadowTex, shadow_coord);

	fragColor = gloablAmbient * material.ambient 
				+ light.ambient * material.ambient;

	if (inShadow != 0.0)
	{
		fragColor += light.diffuse * material.diffuse 
		* max(dot(L,N), 0.0) + light.specular * material.specular 
		* pow(math(dot(H,N), 0.0), material.shininess * 3.0);
	}
}
```

其中，`textureProj`用于在shadowTex采样并判断值的相对大小；

![textureProj|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221121210352.png)

# 问题

## 子遮挡 self occlusion / 阴影痤疮 shadow acne

ShadowMap分辨率有限，对于深度的采样是离散的，每个texel的深度是一个常数值；
当光线与平滑表面有夹角时，就会出现“一部分在阴影中”的判定，导致出现黑色条纹；

![shadowMap|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221121211448.png)




![shadowAcne1|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221121211517.png)

![shadowAcne2|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221121211544.png)

解决方法:
1. 限制光源的角度，使之不足以出现明显的条纹； 
2. 增加shadowMap的分辨率，使得条纹更细（使得每个texel对应的pixel更少）；
3. 增加适当的深度偏移；

```c
//启用多边形偏移
glEnable(GL_PLOYGON_OFFSET_FILL);

//指定偏移量
void glPolygonOffset(Glfloat factor, Glfloat units);

//关闭多边形偏移
glDisable(GL_POLYGON_OFFSET_FILL)
```

但是如果选择的偏移过大，则会使得原本应该有阴影的地方看不到阴影；
该现象成为`阴影分离shadow detach`或`peter panning`；

![shadowDetach|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221121212401.png)

应当根据光线与平面的夹角斜率来动态改变偏移，比如：
```cpp
float bias = max(0.05 * (1.0 - dot(normal, lightDir)), 0.005);
```

  4. 二次深度阴影贴图（Second-depth Shadow Map）
  在深度贴图中，保存深度最小值和次小值，取中间值进行比较；
  该方法要求平面必须是有厚度的，计算开销大且效果不一定好；
  
  ![secondDepth|450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221121212847.png)

## 深度值比较

如果深度比较是在MVP变换之后进行的，
由于投影变换时会对物体进行压缩，**变换后其z值不代表实际距离**；
因此，进行深度比较时，要么用变换后的z值，要么用实际距离，不能混用；

## 阴影锯齿

采样率不足导致；

![alising|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221126214013.png)
解决方法：
1. 提高Shadow Map的分辨率；
2. 使用PCF；
[[PCF-PCSS]]