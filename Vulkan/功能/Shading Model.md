
一般来说，着色计算主要在Fragment Shader中进行。这是因为Fragment Shader为每个像素运行一次，能够为每个像素提供更精确的光照和颜色计算。这对于实现复杂的光照模型，如Phong或者Physically-Based Rendering (PBR)，以及详细的纹理和材质效果非常重要。

相比之下，Vertex Shader主要负责处理顶点数据，包括顶点的位置、颜色、法线、纹理坐标等，并将这些数据传递给Fragment Shader。在某些简单的或者性能敏感的情况下，你可能会在Vertex Shader中进行一些基础的光照计算，但这通常会导致质量较低，因为Vertex Shader只为每个顶点运行一次，然后在这些顶点之间插值得到像素的值。

## Blinn-Phong

### Ambient 环境光

`La`表示光照的ambient项，`Ma`表示材质的ambient项；
$$
Ambient=L_a*M_a
$$

### Diffuse 漫反射

`Ld`表示光照的diffuse项，`Md`表示材质的diffuse项，`N`为法线向量，`L`为光线向量；

$$
Diffuse=L_d*M_d*\max(0,\vec{N} \cdot \vec{L})
$$

### Specular 高光

`Ls`表示光照的specular项，`Ms`表示材质的specular项，`N`为法线向量，`H`为法线向量`L`和视线向量`V`的半程向量；
$$
Specular=L_s*M_s*\max(0,\vec{N} \cdot \vec{H})^p
$$

### 代码

```cpp
vec3 Light = normalize(lightUBO.position - inPosition);
vec3 View = normalize(vec3(0.f, 0.f, 0.f) - inPosition); //视图空间中摄像机位于原点
vec3 Half = normalize(View + Light);

float lightDistance = distance(inPosition, lightUBO.position);
float attenuationIntensify = lightUBO.intensify / (lightUBO.constant + lightUBO.linear * lightDistance + lightUBO.quadratic * lightDistance * lightDistance);

vec4 ambient = lightUBO.ambient * attenuationIntensify * materialUBO.ambient;
vec4 diffuse = lightUBO.diffuse * attenuationIntensify * materialUBO.diffuse * max(0.0, dot(inNormal, Light));
vec4 specular = lightUBO.specular * attenuationIntensify * materialUBO.specular * pow(max(0.0, dot(inNormal, Half)), materialUBO.shininess);

outColor = ambient + diffuse + specular;
```
## PBR

### 法线分布函数

在物理基础渲染（PBR）中，法线分布函数（NDF）是用于描述表面粗糙度的模型。以下是一些常见的NDF模型：

1. `Blinn-Phong`模型：这是一个经典的模型，它的公式为：
$$
D(\vec h)=\frac{n+2}{(2\pi)*\cos^{n}(\alpha)}
$$
其中，n是粗糙度参数，α是半向量h和表面法线的夹角。

优点：计算简单，易于理解。
缺点：在粗糙度较高时，其反射效果不够真实。

2. `Beckmann`模型：这是一个更为现代的模型，它的公式为：
其中，m是粗糙度参数，α是半向量h和表面法线的夹角。

优点：在粗糙度较高时，其反射效果更为真实。
缺点：计算复杂度较高。

3. `GGX/Trowbridge-Reitz`模型：这是一个最新的模型，它的公式为：
其中，m是粗糙度参数，n是表面法线，h是半向量。

优点：在各种粗糙度下，其反射效果都非常真实，且计算效率较高。
缺点：相比于其他模型，其理论基础较为复杂，不易理解。

![NDF|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230808102454.png)


![fresnel|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230808102649.png)

![GDF|200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230808102943.png)
