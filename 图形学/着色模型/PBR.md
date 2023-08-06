# PBR的范畴

基于物理的光照+基于物理的材质+基于物理的摄像机，才是真正完整的基于物理的渲染系统；

![PBR scope|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805135907.png)


# 核心理论

## 人眼与可见光

我们人眼所能看到的颜色，并非物体本身的真实颜色，而是**不能被物体所吸收、从而被反射或散射出的颜色**；

![color|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806000025.png)

人类视觉系统（Human Visual System，`HVS`）：
其中有两种感光细胞：视锥细胞`Cones`与视杆细胞`Rods`，这些感光细胞将光子转换为视觉信号，再通过视神经传递到大脑的视觉皮层，最终产生感知图像；

![HVS|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806000405.png)

![cones|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806000624.png)

只有可见光才能被人眼所感知，即波长在400nm到700nm之间的电磁波；

![light|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806000759.png)

可见光的波长大概为蜘蛛丝宽度的三分之一到二分之一，蜘蛛丝宽度大概为人类头发的五十分之一；

![lightWave|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806001402.png)


## 材质的复折射率

复折射率（`Complex Index of Refraction`）是材质的特性，**描述了材质对光线的折射和吸收能力**；

实部描述了材质的折射能力，即光线在进入材质时的方向变化 —— 实部的值越大，折射角越大；
虚部描述了材质的吸收能力，即光线在通过材质时的能量损失 —— 虚部的值越大，吸收越强；

特定材质对光的选择性吸收是因为**所选择的光波频率与该材质原子中的电子振动的频率相匹配**。由于不同的原子和分子具有不同的固有振动频率，其将选择性地吸收不同频率的可见光；

![complexior|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806002422.png)


## 光线折射的条件

光在表面的折射条件是需要**在小于单一波长的距离内发生折射率的突然变化**；

![refract|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806002715.png)

折射率缓慢的逐渐变化不会导致光线的分离，而是导致其传播路径的弯曲。 当空气密度因温度而变化时，通常可以看到这种效果，例如海市蜃楼（mirages）和热形变（heat distortion）；

![heatDistortion|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806002812.png)


## 材质对光线的吸收与散射

材质对光线的吸收、散射和镜面反射决定了材质的外观（漫反射与散射是同一概念）；

- **吸收**（`Absorption`）决定了材质的外观颜色，**几乎任何材质的外观颜色都是由其吸收的波长决定的**；比如一个材质显示绿色，是因为其吸收了除绿色之外的光，反射或散射了绿色的光；
- **散射**（`Scattering`）决定了材质的浑浊程度，**散射能力越强越不透明**；一般来说固体和液体中的颗粒比光的波长更大，并且均匀地散射所有可见波长的光；但是相比较空气和玻璃，空气的散射能力要强于玻璃，因此可以看到蓝天和晚霞，玻璃相比空气更加透彻；

![absorbScatter|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806003817.png)

![absorbScatter|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806003902.png)

此外，材质对光的吸收和散射还与观察尺度相关；
比如在近距离内，空气和海水都观察不到散射；但是一旦观察尺度大到数公里，那么散射现象就会很明显；

![ocean|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806004647.png)

![air|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806004716.png)

## 光与物质的交互

![light|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806004350.png)

当一束光线入射到物体表面时，由于物体表面与空气两种介质之间折射率的快速变化，光线会发生反射和折射；

- 反射（Reflection）：**光线在两种介质交界处的直接反射即镜面反射**（Specular）。金属的镜面反射颜色为三通道的彩色，而非金属的镜面反射颜色为单通道的单色；

- 折射（Refraction）：**从表面折射入介质的光，会发生吸收（absorption）和散射（scattering）**，而介质的整体外观由其散射和吸收特性的组合决定，其中：

	- 散射（Scattering）： **折射率的快速变化引起散射**，光的方向会改变（分裂成多个方向），但是光的总量或光谱分布不会改变，散射最终被视作的类型与观察尺度有关：

		 - 次表面散射（Subsurface Scattering）：观察像素小于散射距离时，散射被视作次表面散射；
		 - 漫反射（Diffuse）：观察像素大于散射距离时，散射被视作漫反射；
		 - 透射（Transmission）：入射光经过折射穿过物体后的出射现象，透射为次表面散射的特例；

	- 吸收（Absorption）： **具有复折射率的物质区域会引起吸收，具体原理是光波频率与该材质原子中的电子振动的频率相匹配**。复折射率（complex number）的虚部（imaginary part）确定了光在传播时是否被吸收（转换成其他形式的能量）。发生吸收的介质的光量会随传播的距离而减小（如果吸收优先发生于某些波长，则可能也会改变光的颜色），而光的方向不会因为吸收而改变。任何颜色色调通常都是由吸收的波长相关性引起的；

## 微平面理论 Microfacet Theory

微平面理论是`PBR`的一种模型，它描述了表面的微观几何结构对光的相互作用的影响；

微平面理论认为，表面由许多微小的平面组成，每个平面都有自己的法线方向和粗糙度。光线与这些微小平面相互作用时，会根据平面的法线方向和粗糙度来散射和反射；

粗糙度越高，表面的微平面越不规则，光线在表面上的散射越强，表面看起来越模糊；
粗糙度越低，表面的微平面越规则，光线在表面上的反射越强，表面看起来越光滑；

![rough|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805132557.png)

![roughness|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806004843.png)



微平面理论在PBR中被广泛应用，用于模拟各种材质的外观，包括金属、塑料、皮肤等；它可以提供更真实的光照效果，使渲染结果更加逼真；

![microfacet|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805123420.png)



## 菲涅尔反射 Fresnel Reflectance

光线以不同角度入射会有不同的反射率，在相同的入射角度下不同的物质也会有不同的反射率；万物皆有菲涅尔反射；

`F0`是即0度角入射的菲涅尔反射值，大多数非金属的`F0`范围是0.02~0.04，大多数金属的`F0`范围是0.7~1.0；

![fresnel|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805124628.png)

![fresnel|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806005035.png)

![fresnel|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806005057.png)

不同材质的菲涅尔效应的强弱是不同的，导体（如金属）的菲涅尔效应一般很弱，主要是因为导体本身的反射率就已经很强。就拿铝来说，其反射率在所有角度下几乎都保持86%以上，随角度变化很小，而绝缘体材质的菲涅尔效应就很明显，比如折射率为1.5的玻璃，在表面法向量方向的反射率仅为4%，但当视线与表面法向量夹角很大的时候，反射率可以接近100%，这一现象也导致了金属与非金属外观上的不同；

![fresnel|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806112253.png)

因此，通常将`F0`（即0度角入射时的菲涅尔反射率）作为材质的特征反射率，帮助对该材质的反射属性进行建模；

查询物体的折射率`IOR`：
https://pixelandpoly.com/ior.html

## 能量守恒 Energy Conservation

反射与散射是互斥的；
要使光线散射，首先需要让光线穿透物体表面，从而减少了反射的部分；一个物体材质的反射能力越强，其散射也就越差；

![energyConservation|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805131143.png)


## 次表面散射 Subsurface Scatter

在现实世界中，当光线照射到物体表面时，一部分光线会被反射，一部分光线会穿透物体表面并在物体内部散射，这种在物体内部的散射过程就是次表面散射；
次表面散射对于许多自然物体的视觉外观有重要影响，例如人的皮肤、蜡、牛奶、果肉等；

对于从物体表面折射的光，视材质的不同有不同的处理方法：

- 对于金属，折射光会立刻被吸收 - 能量被自由电子立即吸收；
![refract1|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805134736.png)

- 对于非金属（也称为电介质或绝缘体），一旦光在其内部折射，就表现为常规的参与介质，表现出吸收和散射两种行为；
![refract2|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805135250.png)


漫反射和次表面散射其实是相同物理现象，本质都是折射光的次表面散射的结果。唯一的区别是相对于观察尺度的散射距离。散射距离相较于像素来说微不足道，次表面散射便可以近似为漫反射。也就是说，光的折射现象，建模为漫反射还是次表面散射，取决于观察的尺度；

- 像素较大时，建模为漫反射；
![refract3|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805135631.png)

- 像素较小时，需要建模为次表面散射才能有真实的效果；
![refract4|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805135734.png)


## 基于真实世界的材质参数

PBR的正统材质参数往往都基于真实世界测量。真实世界中的物质可分为三大类：绝缘体（`Insulators`），半导体（`semiconductors`）和导体（`conductors`）。在渲染和游戏领域，我们一般只对其中的两个感兴趣：导体（金属）和绝缘体（电解质，非金属）。菲涅尔反射率代表材质的镜面反射颜色与强度，是真实世界材质的核心测量数值；

其中非金属具有灰色的镜面反射颜色，而金属具有彩色的镜面反射颜色，即非金属的`F0`是一个`float`,金属的`F0`是一个`float3`；

![fresnel|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805235433.png)

一般来说，导体会吸收而不是散射任何穿透表面的光线。这意味着理论上导体不会显示任何漫射光。但实际上，金属表面通常会有氧化物或其他残留物，会散射少量光线；

正是由于金属和其他物质之间的这种差异，导致一些渲染系统将金属度作为渲染参数之一；


## 法线分布函数NDF


## 几何分布函数GDF



# 渲染方程与BxDF

## 渲染方程 Rendering Equation

渲染方程（`Rendering Equation`）是计算机图形学中用于描述光在场景中如何传播的基本方程。它是由`James Kajiya`在1986年提出的。渲染方程考虑了光的直接照射、反射、折射和散射等多种效应，可以用来模拟复杂的光照和阴影效果；

![renderingEquation|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805141154.png)


$$
L_o=L_e+\int_{\Omega}L_i \cdot f_r \cdot (\omega_i \cdot n)d_{\omega_i}
$$
- `Lo`为p点的出射光亮度；
- `Le`为p点发出的光的亮度；
- `Li`为p点的入射光亮度；
- `fr`为p点的入射方向到出射方向的光的反射比例，即BXDF项；
- $\omega_i \cdot n$  为入射角度导致的入射光衰减；
- $\int_\Omega d_{\omega_i}$  为入射方向半球的积分；

## 反射方程 Reflect Equation

实际使用中常使用简化版本的反射方程来代替渲染方程；
相比渲染方程，反射方程不考虑物体的自发光，并且对于入射光通常只考虑直接照明；

$$
L_o=\int_{\Omega}L_i \cdot f_r \cdot (\omega_i \cdot n)d_{\omega_i}
$$

## BxDF

`BxDF`一般而言是对`BRDF`、`BTDF`、`BSDF`、`BSSRDF`等几种双向分布函数的一个统一的表示；

`BRDF（Bidirectional Reflectance Distribution Function）`：双向反射分布函数，描述了光线如何被物体表面反射。BRDF只考虑光线的反射，不考虑透射；

`BTDF（Bidirectional Transmittance Distribution Function）`：双向透射分布函数，描述了光线如何通过物体表面透射。BTDF只考虑光线的透射，不考虑反射；

`BSDF（Bidirectional Scattering Distribution Function）`：双向散射分布函数，是BRDF和BTDF的结合，描述了光线如何被物体表面反射和透射。BSDF可以用于描述透明或半透明物体的光学性质；

`BSSDF（Bidirectional Surface Scattering Distribution Function）`：双向表面散射分布函数，是BSDF的扩展，不仅考虑了光线在物体表面的反射和透射，还考虑了光线在物体内部的散射；BSSDF可以用于描述次表面散射（如人的皮肤、蜡、牛奶等）；

这些模型的区别在于它们考虑的光线传播方式不同 —— BRDF只考虑反射，BTDF只考虑透射，BSDF考虑反射和透射，BSSDF则还考虑了次表面散射。

简单来说，**BSDF  = BRDF + BTDF，BSSDF = BSDF plus**；

在上述这些BxDF中，**BRDF最为简单，也最为常用**。因为游戏和电影中的大多数物体都是不透明的，用BRDF就完全足够，而BSDF、BTDF、BSSRDF往往更多用于半透明材质和次表面散射材质；

# 迪士尼原则的BxDF Disney-Principled BxDF

迪士尼动画工作室在SIGGRAPH 2012上著名的talk《Physically-based shading at Disney》中提出了迪士尼原则的BRDF（`Disney Principled BRDF`），奠定了后续游戏行业和电影行业PBR的方向和标准；

迪士尼原则BRDF（`Disney Principled BRDF`）是由迪士尼研究所提出的一种用于实时渲染的BRDF模型。它的目标是提供一种既能产生高质量渲染效果，又能在各种材质和照明条件下保持一致性的BRDF模型；

迪士尼原则BRDF包括两个主要部分：漫反射项和镜面反射项；
漫反射项使用了一个**改进的Lambert模型**，可以更好地模拟真实世界的漫反射行为；
镜面反射项则**基于GGX微表面模型**，可以模拟从完全光滑到非常粗糙的各种表面；

迪士尼原则BRDF的一个重要特性是它的参数都是物理意义明确的，如粗糙度、金属度、折射率等，这使得它非常容易使用和理解。此外，它还包括了一些额外的特性，如次表面散射、清漆层、各向异性等，可以模拟各种复杂的材质效果；

迪士尼原则的BRDF（Disney Principled BRDF）核心理念如下：
1. 应使用直观的参数，而不是物理类的晦涩参数；
2. 参数应尽可能少；
3. 参数在其合理范围内应该为0到1；
4. 允许参数在有意义时超出正常的合理范围；
5. 所有参数组合应尽可能健壮和合理；
以上五条原则，很好地保证了迪士尼原则的BRDF的易用性；

---

从本质上而言，`Disney Principled BRDF`模型是金属和非金属的混合型模型，最终结果是基于金属度`metallice`在金属BRDF和非金属BRDF之间进行线性插值；

正因为这套新的渲染理念**统一了金属和非金属的材质表述**，可以仅通过少量的参数来涵盖自然界中绝大多数的材质，并可以得到非常逼真的渲染品质；

也正因如此，在PBR的金属/粗糙度工作流中，固有色（baseColor）贴图才会同时包含金属的反射率值和非金属的漫反射颜色；

![disneyBRDF|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806121109.png)


## 工具与资源

### MERL 100 BRDF材质库

Matusik等人捕获的一组100个各向同性BRDF材质样本库，涵盖了各种材质，包括油漆，木材，金属，织物，石材，橡胶，塑料和其他合成材质。对学术与研究免费授权；

MERL BRDF主站 ：*http://merl.com/brdf*
Database地址：*https://people.csail.mit.edu/wojciech/BRDFDatabase/*

![MERL100|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806120112.png)

### BRDF Explorer

迪士尼为分析、比较和新开发BRDF模型而开发的可视化工具；
该工具在分析测量材质，比较现有模型，以及开发新模型方面具有无可估量的价值；

官方主页：*https://www.disneyanimation.com/technology/brdf.html*
GitHub地址：*https://github.com/wdas/brdf*

![BRDFexplorer|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806120152.png)

### BRDF Image Slice

将θh与θd作为横轴和纵轴，对观察到的材质的BRDF进行建模的2D图像切片；

![BRDFslice|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806120226.png)

![BRDFslice|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806120255.png)


## BRDF

- *baseColor*（基础色）：表面颜色，通常由纹理贴图提供；

- *subsurface*（次表面）：使用次表面近似控制漫反射形状；

- *metallic*（金属度）：金属（0 =电介质，1 =金属），这是两种不同模型之间的线性混合；金属模型没有漫反射成分，并且还具有等于基础色的着色入射镜面反射；

- *specular*（镜面反射强度）：入射镜面反射量，用于取代折射率；

- *specularTint*（镜面反射颜色）：对美术控制的让步，用于对基础色（base color）的入射镜面反射进行颜色控制；掠射镜面反射仍然是非彩色的；

- *roughness*（粗糙度）：表面粗糙度，控制漫反射和镜面反射；

- *anisotropic*（各向异性强度）：各向异性程度，用于控制镜面反射高光的纵横比（0 =各向同性，1 =最大各向异性）；

- *sheen*（光泽度）：一种额外的掠射分量（grazing component），主要用于布料；

- *sheenTint*（光泽颜色）：对sheen（光泽度）的颜色控制；

- *clearcoat*（清漆强度）：有特殊用途的第二个镜面波瓣（specular lobe）；

- *clearcoatGloss*（清漆光泽度）：控制透明涂层光泽度，0 =“缎面（satin）”外观，1 =“光泽（gloss）”外观；

![BRDF|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805153842.png)

迪士尼原则BRDF的源代码：
*https://github.com/wdas/brdf/blob/main/src/brdfs/disney.brdf*
## BSDF

随后的2015年，迪士尼动画工作室在Disney Principled BRDF的基础上进行了修订，提出了Disney Principled BSDF；

![BSDF|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805154137.png)

Disney BRDF模型本质上是金属和非金属的混合型模型，对于Disney BSDF，Disney仍然延续了之前的设计理念，采用了混合的方式并结合已有的Disney BRDF模型进行实现。如下图，Disney新增了⼀个参数specTrans（镜面反射透明度）来控制BRDF 和BSDF的混合。基于specTrans完成混合后，再使用和Disney BRDF类似的方式，基于metallic再进行一次混合。

即**Disney BSDF模型的本质是金属BRDF、非金属BRDF、与Specular BSDF三者的混合型模型**；

![disneyBSDF|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806133144.png)

# BRDF模型

## 漫反射项 Diffuse BRDF

![diffuseBRDF|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805202526.png)

### Lambert模型

在`Lambertian`反射模型中，`BRDF`的漫反射部分被定义为材质的反射率（albedo）除以π；
这个公式的原理基于`Lambert`的余弦定律，即入射光的强度与表面法线和光线方向的夹角的余弦成正比。当我们将光线方向积分在半球上时，结果是π；
因此，为了保持能量守恒，我们需要将albedo除以π；
这样，当光线从所有方向均匀照射到表面时，表面反射的总能量不会超过表面接收的总能量；

$$
f_d=\frac{albedo}{\pi}
$$

### Disney Diffuse模型

迪士尼跟据对MERL 100材质库的观察，开发了一种用于漫反射的新的经验模型，以在光滑表面的漫反射菲涅尔阴影和粗糙表面之间进行平滑过渡；

$$
f_d = \frac{baseColor}{\pi}(1+(F_{D90}-1)(1-\cos{\theta_l})^5)(1+(F_{D90}-1)(1-\cos{\theta_v})^5)
$$

$$
F_{D90}=0.5+2*roughness*\cos^2{\theta_d}
$$
$\theta_d$  为`V`与`H`之间的夹角，$\theta_l$  为`L`与`N`之间的夹角，$\theta_v$  为`V`与`N`之间的夹角；

## 镜面反射项 Specular BRDF

![specularBRDF|1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805205246.png)


游戏业界目前最主流的基于物理的镜面反射BRDF模型是基于微平面理论（`microfacet theory`）的`Microfacet Cook-Torrance BRDF`；

微平面模型有两个假设：
1. 微观几何的尺度小于观察尺度（例如着色分辨率），但大于可见光波长的尺度，因此可以将每个表面点视为光学平坦平面，此时表面法线向量$\vec{N}$ 和半程向量$\vec{H}$ 基本相等；

![microfacet|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805214938.png)

2. 不考虑多次表面反射的作用；

![multifacet|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230805215106.png)

`Microfacet Cook-Torrance BRDF`的一般形式的Specular DRDF：

$$
f=\frac{D(\vec H)\space F(\vec V,\vec H)\space G(\vec L,\vec V,\vec H)}{4(\vec{N}\cdot \vec{L})(\vec{N}\cdot \vec{V})}
$$

$D(\vec H)$ ：法线分布函数（`Normal Distribution Function`），**描述微面元法线分布的概率，即正确朝向的法线的浓度**。即具有正确朝向，能够将来自l的光反射到v的表面点的相对于表面面积的浓度；

$F(\vec V, \vec H)$ ：菲涅尔方程（`Fresnel Equation`），**描述不同角度下表面反射的光线所占的比率**；

$G(\vec L, \vec V, \vec H)$ ：几何函数（`Geometry Function`）：描述微平面自成阴影的属性，即**可视为光学平坦平面的未被遮蔽的表面点所占的百分比**；

$4(\vec N \cdot \vec L)(\vec N \cdot \vec V)$ ：校正因子（`Correction Factor`），作为微观几何的局部空间和整个宏观表面的局部空间之间变换的微平面量的校正；


此外对于该模型需要注意的是：
1. 分母中的点积项需要避免负值和零；
2. `Cook-Torrance BRDF`仅对单层微表面上的当个散射进行建模，没有考虑多次散射、分层材质、衍射等物理现象，仍有很大的改进空间；

### Specular D

法线分布函数`NDF`的常见模型有：
1. `Beckmann[1963]`
2. `Blinn-Phong[1977]`
3. `GGX [2007] / Trowbridge-Reitz[1975]`
4. `Generalized-Trowbridge-Reitz(GTR) [2012]`
5. `Anisotropic Beckmann[2012] `
6. `Anisotropic GGX [2015]`

#### GGX

目前业界最为主流的是`GGX（Trowbridge-Reitz）`模型：

$$
D_{GGX}=\frac{\alpha^2}{\pi((\vec N \cdot \vec H)^2(\alpha^2-1)+1)^2}
$$
`N`为表面法线（宏观表面的法线），`H`为入射光线与出射光线的半程向量（理想表面的法线），`α`为粗糙度；
由于粗糙度的存在，导致`N`与`H`之间存在偏差；

#### GTR

Disney将`Trowbridge-Reitz`模型进行了N次幂的推广，并将其取名为`Generalized-Trowbridge-Reitz`，即`GTR`，通过控制次幂可以模拟多种模型；

![GTR NDF|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806133946.png)

$$
D_{GTR}=\frac{c}{(\alpha^2\cos^2{\theta_h}+\sin^2{\theta_h})^\gamma}
$$
当$\gamma=1$  时，
$$
D_{GTR(\gamma=1)}=\frac{\alpha^2-1}{\pi\log(\alpha^2)(1+(\alpha^2-1)\cos^2_{\theta_h})}
$$

当$\gamma=2$  时，
$$
D_{GTR(\gamma=2)}=\frac{\alpha^2}{\pi(1+(\alpha^2-1)\cos^2_{\theta_h})^2}
$$
#### Anisotropic GTR2

$$
deno=\frac{(\vec H \cdot \vec X)^2}{\alpha_x^2}\frac{(\vec H \cdot \vec Y)^2}{\alpha_y^2}+(\vec H \cdot \vec N)^2
$$
$$
D_{anisoGTR2}=\frac{1}{\pi*\alpha_x*\alpha_y*deno^2}
$$

### Specular F

#### Schlick近似

对于菲涅尔项，业界方案一般都采用`Schlick`的`Fresnel近似`，因为计算成本低廉，而且精度足够：

$$
F_{Schlick}=F_0+(1-F_0)(1-(\vec V \cdot \vec H))^5
$$

`F0`为基础反射率或镜面反射率，指光线在垂直入射（即入射角为0度）时的反射比例；
`F0`的值取决于物体的材质，对于非金属材质通常较低，大约在0.02到0.04之间。对于金属材质通常较高，可以接近1；

对于非金属材质，可由以下公式求得：
$$
F_0=(\frac{n_1-n_2}{n_1+n_2})^2
$$
其中`n1`是空气的折射率（通常为1），`n2`是物体的折射率；
通常假设空气的折射率为1，由此公式可以写作：
$$
F_0=(\frac{n-1}{n+1})^2
$$
其中`n`为物体的折射率；

对于金属材质，`F0`通常由实验测量得到，因为**金属的反射率不仅取决于折射率，还取决于电导率**；

当介质之间的相对`IOR`接近1时，`Schlick`近似的误差比较大，此时可以直接用精准的菲涅尔方程进行计算：

![fresnelEquation|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806140720.png)


### Specular G

几何分布函数`GDF`的常见模型有：
1. `Smith [1967]`
2. `Cook-Torrance [1982]`
3. `Neumann [1999]`
4. `Kelemen [2001]`
5. `Implicit [2013]`
6. `Smith联合遮蔽阴影函数`（`Smith Joint Masking-Shadowing Function`）

其中Smith联合遮蔽阴影函数又具有如下几种形式：
1. 分离遮蔽阴影型（Separable Masking and Shadowing）
2. 高度相关掩蔽阴影型（Height-Correlated Masking and Shadowing）
3. 方向相关掩蔽阴影型（Direction-Correlated Masking and Shadowing）
4. 高度-方向相关掩蔽阴影型（Height-Direction-Correlated Masking and Shadowing）

其中最为常用的也是最简单的形式为 —— 分离遮蔽阴影（Separable Masking and Shadowing Function）；

分离遮蔽阴影将几何项G分为两个独立的部分：光线方向（light）和视线方向（view），并对两者用相同的分布函数来描述。根据这种思想，结合法线分布函数（NDF）与Smith几何阴影函数，于是有了以下新的Smith几何项：
1. `Smith-GGX`
2. `Smith-Beckmann`
3. `Smith-Schlick`
4. `Schlick-Beckmann`
5. `Schlick-GGX`

除此之外，各向同性和各向异性的`GDF`函数也是不一样的：
- 各向同性（Isotropic）：这个版本假设表面的粗糙度在所有方向上都是相同的，是最常用的版本，计算相对简单；
- 各向异性（Anisotropic）：这个版本允许表面在不同的方向上有不同的粗糙度，可以用来模拟一些特殊的表面效果，如刷金属或拉丝玻璃等，但是计算会更复杂一些；
#### Schlick-GGX

其中`Schlick-GGX`的公式为（近似模型，计算成本低但不够精确）：
$$
G_{Schlick-GGX}=G_1(\vec L)G_1(\vec V)
$$
$$
G_1(\vec x)=\frac{(\vec N \cdot \vec x)}{(\vec N \cdot \vec x)(1-k)+k}
$$
$$
k=\frac{\alpha}{2} \space \space \space \alpha=(\frac{roughness+1}{2})^2
$$
#### Smith-GGX

`Smith-GGX`的公式为（更精准的模型，计算成本较高）：

$$
G_{Smith-GGX}=G_{GGX}(\vec L)G_{GGX}(\vec V)
$$
$$
G_{GGX}(\vec x)=\frac{2(\vec N \cdot \vec x)}{(\vec N \cdot \vec x)+\sqrt{\alpha^2+(1-\alpha^2)(\vec N \cdot \vec x)^2}}
$$
$$
\alpha=(0.5+\frac{roughness}{2})^2
$$

对于清漆层，直接使用$\alpha=0.25$的固定值进行计算；


# 结合BRDF的反射方程

$$
L_o=\int_{\Omega}(k_d(DiffuseBRDF)+k_s(SpecularBRDF))L_i(\omega_i\cdot n)d_{\omega_i}
$$


# PBR材质

![PBRMaterial|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806195914.png)


# PBR光照







































# 附录

## 常见材质的F0值

### 金属

![f0|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806113230.png)

### 绝缘体（电介质）

![f0|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806113312.png)

### 半导体

![f0|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230806113332.png)






# 参考

*https://github.com/QianMo/PBR-White-Paper/*

*https://marmoset.co/posts/basic-theory-of-physically-based-rendering/*

*https://media.disneyanimation.com/uploads/production/publication_asset/48/asset/s2012_pbs_disney_brdf_notes_v3.pdf*

*https://raw.githubusercontent.com/QianMo/PBR-White-Paper/master/bonus/%5BPBR-White-Paper%5D%20PBR-Material-F0-Quick-Reference-Chart.pdf*

*https://www.jianshu.com/p/f92c9037355e*

*https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf*

*https://blog.selfshadow.com/publications/s2013-shading-course/hoffman/s2013_pbs_physics_math_notes.pdf*

