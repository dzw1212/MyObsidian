*https://developer.nvidia.com/legacy-texture-tools*

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240825212302.png)

除了nvDXT库之外，还包括以下四个命令行工具：

- `nvdxt.exe`：提供对`nvDXTlib`的访问途径；
- `detach.exe`：从`.DDS`文件中提取`MIP`；
- `stitch.exe`：将多个不同级别的`MIP`合并为一个`.DDS`文件；
- `readDXT.exe`：`DDS`转`TGA`；

# nvdxt

一些示例：
```cmd
nvdxt -cubeMap -list cubemapfile.lst -output cubemap.dds
nvdxt -cubeMap -file cubemapfile.tga
nvdxt -file test.tga -dxt1c
nvdxt -file *.tga
nvdxt -file c:\temp\*.tga
nvdxt -file temp\*.tga
nvdxt -file height_field_in_alpha.tga -n3x3 -alpha -scale 10 -wrap
nvdxt -file grey_scale_height_field.tga -n5x5 -rgb -scale 1.3
nvdxt -file normal_map.tga -norm
nvdxt -file image.tga -dudv -fade -fadeamount 10
nvdxt -all -dxt3 -gamma -outdir .\dds_dir -time
nvdxt -file *.tga -depth -max -scale 0.5
```

## 常用操作

- `-profile <profile name>`：读取由Photoshop插件创建的配置文件。
- `-quick`：使用快速压缩方法。
- `-quality_normal`：正常质量压缩。
- `-quality_production`：生产质量压缩。
- `-quality_highest`：最高质量压缩（这可能会非常慢）。
- `-rms_threshold <int>`：质量RMS误差。超过此值时，将执行广泛的搜索。
- `-prescale <int> <int>`：首先将图像重新缩放到此大小。
- `-rescale <nearest | hi | lo | next_lo>`：将图像重新缩放到最近的、最高的、下一个或下一个低的2的幂。
- `-rel_scale <float, float>`：原始图像的相对缩放。0.5是半尺寸，默认值为1.0, 1.0。

## 缩放选项

默认缩放选项为`RescaleCubic`；

- `-RescalePoint`：点过滤器。
- `-RescaleBox`：盒子过滤器。
- `-RescaleTriangle`：三角形过滤器。
- `-RescaleQuadratic`：二次过滤器。
- `-RescaleCubic`：立方体过滤器。
- `-RescaleCatrom`：Catmull-Rom过滤器。
- `-RescaleMitchell`：Mitchell过滤器。
- `-RescaleGaussian`：高斯过滤器。
- `-RescaleSinc`：Sinc过滤器。
- `-RescaleBessel`：贝塞尔过滤器。
- `-RescaleHanning`：Hanning过滤器。
- `-RescaleHamming`：Hamming过滤器。
- `-RescaleBlackman`：Blackman过滤器。
- `-RescaleKaiser`：Kaiser过滤器。

- `-clamp <int, int>`：最大图像尺寸。图像宽度和高度被限制。
- `-clampScale <int, int>`：最大图像尺寸。图像宽度和高度被缩放。
- `-window <left, top, right, bottom>`：压缩原始窗口的窗口。
- `-nomipmap`：不生成MIP贴图。
- `-nmips <int>`：指定生成的MIP贴图数量。
- `-rgbe`：图像是RGBE格式。
- `-dither`：添加抖动。
- `-sharpenMethod <method>`：锐化MIP贴图的方法。`<method>`可以是以下之一：
	- `None`：无
	- `Negative`：负片
	- `Lighter`：变亮
	- `Darker`：变暗
	- `ContrastMore`：增加对比度
	- `ContrastLess`：减少对比度
	- `Smoothen`：平滑
	- `SharpenSoft`：柔和锐化
	- `SharpenMedium`：中等锐化
	- `SharpenStrong`：强烈锐化
	- `FindEdges`：查找边缘
	- `Contour`：轮廓
	- `EdgeDetect`：边缘检测
	- `EdgeDetectSoft`：柔和边缘检测
	- `Emboss`：浮雕
	- `MeanRemoval`：均值去除
	- `UnSharp <radius, amount, threshold>`：非锐化
	- `XSharpen <xsharpen_strength, xsharpen_threshold>`：X锐化
	- `Custom`：自定义

- `-pause`：出错时等待键盘输入。
- `-flip`：上下翻转。
- `-timestamp`：仅更新更改的文件。
- `-list <filename>`：要转换的文件列表。
- `-cubeMap`：创建立方体贴图。立方体面可以通过`-list`选项指定单独的文件，也可以通过`-file`选项指定一个文件。
- `-volumeMap`：创建体积纹理。体积切片可以通过`-list`选项指定单独的文件，也可以通过`-file`选项指定一个文件。
- `-all`：当前目录中的所有图像文件。
- `-outdir <directory>`：输出目录。
- `-deep [directory]`：包括所有子目录。
- `-outsamedir`：输出目录与输入目录相同。
- `-overwrite`：如果输入是`.dds`文件，覆盖旧文件。
- `-forcewrite`：覆盖只读文件。
- `-file <filename>`：要处理的输入文件。接受通配符。
- `-output <filename>`：要写入的文件名（也可以指定`-outfile`）。
- `-append <filename_append>`：将此字符串附加到输出文件名。
- `-8 <dxt1c | dxt1a | dxt3 | dxt5 | u1555 | u4444 | u565 | u8888 | u888 | u555 | L8 | A8>`：使用此格式压缩8位图像。
- `-16 <dxt1c | dxt1a | dxt3 | dxt5 | u1555 | u4444 | u565 | u8888 | u888 | u555 | A8L8>`：使用此格式压缩16位图像。
- `-24 <dxt1c | dxt1a | dxt3 | dxt5 | u1555 | u4444 | u565 | u8888 | u888 | u555>`：使用此格式压缩24位图像。
- `-32 <dxt1c | dxt1a | dxt3 | dxt5 | u1555 | u4444 | u565 | u8888 | u888 | u555>`：使用此格式压缩32位图像。
- `-swapRB`：交换红色和蓝色通道。
- `-swapRG`：交换红色和绿色通道。
- `-gamma <float value>`：在过滤过程中进行伽马校正。
- `-outputScale <float, float, float, float>`：按此比例缩放输出（r,g,b,a）。
- `-outputBias <float, float, float, float>`：按此量偏移输出（r,g,b,a）。
- `-outputWrap`：将溢出值按输出格式取模。
- `-inputScale <float, float, float, float>`：按此比例缩放输入（r,g,b,a）。
- `-inputBias <float, float, float, float>`：按此量偏移输入（r,g,b,a）。
- `-binaryalpha`：将alpha视为0或1。
- `-alpha_threshold <byte>`：alpha参考值(0-255)。
- `-alphaborder`：用alpha=0的边框图像。
- `-alphaborderLeft`：用alpha=0的左边框图像。
- `-alphaborderRight`：用alpha=0的右边框图像。
- `-alphaborderTop`：用alpha=0的上边框图像。
- `-alphaborderBottom`：用alpha=0的下边框图像。
- `-fadeamount <int>`：每个MIP级别的淡化百分比。默认15。
- `-fadecolor`：在MIP级别上淡化贴图（颜色、法线或DuDv）。
- `-fadetocolor <hex color>`：要淡化到的颜色。
- `-custom_fade <n> <n fadeamounts>`：设置自定义淡化量。`n`是淡化量的数量。淡化量是(0,1)。
- `-fadealpha`：在MIP级别上淡化alpha。
- `-fadetoalpha <byte>`：要淡化到的alpha(0-255) 。
- `-border`：用颜色边框图像。
- `-bordercolor <hex color>`：边框的颜色。
- `-force4`：强制DXT1c始终使用四种颜色。
- `-weight <float, float, float>`：R、G和B的压缩权重。
- `-luminance`：将颜色值转换为L8格式的亮度。
- `-greyScale`：转换为灰度。
- `-greyScaleWeights <float, float, float, float>`：覆盖灰度转换权重（0.3086, 0.6094, 0.0820, 0）。
- `-brightness <float, float, float, float>`：每通道亮度。默认0.0，通常范围(0,1)。
- `-contrast <float, float, float, float>`：每通道对比度。默认1.0，通常范围(0.5, 1.5)。

## 贴图格式选项

默认为`dxt3`；

- `-dxt1c`：DXT1（仅颜色）
- `-dxt1a`：DXT1（1位alpha）
- `-dxt3`：DXT3
- `-dxt5`：DXT5n
- `-u1555`：未压缩的1:5:5:5
- `-u4444`：未压缩的4:4:4:4
- `-u565`：未压缩的5:6:5
- `-u8888`：未压缩的8:8:8:8
- `-u888`：未压缩的0:8:8:8
- `-u555`：未压缩的0:5:5:5
- `-p8c`：调色板8位（256色）
- `-p8a`：调色板8位（256色带alpha）
- `-p4c`：调色板4位（16色）
- `-p4a`：调色板4位（16色带alpha）
- `-a8`：8位alpha通道
- `-cxv8u8`：法线贴图格式
- `-v8u8`：EMBM格式（8位，两个分量有符号）
- `-v16u16`：EMBM格式（16位，两个分量有符号）
- `-A8L8`：8位alpha通道，8位亮度
- `-fp32x4`：fp32四通道（A32B32G32R32F）
- `-fp32`：fp32单通道（R32F）
- `-fp16x4`：fp16四通道（A16B16G16R16F）
- `-dxt5nm`：DXT5风格法线贴图
- `-3Dc`：3DC
- `-g16r16`：16位输入，两个分量
- `-g16r16f`：16位浮点，两个分量

## Mipmap过滤选项

默认为`Box`；

- `-Point`：点过滤器。
- `-Box`：盒子过滤器。
- `-Triangle`：三角形过滤器。
- `-Quadratic`：二次过滤器。
- `-Cubic`：立方体过滤器。
- `-Catrom`：Catmull-Rom过滤器。
- `-Mitchell`：Mitchell过滤器。
- `-Gaussian`：高斯过滤器。
- `-Sinc`：Sinc过滤器。
- `-Bessel`：贝塞尔过滤器。
- `-Hanning`：Hanning过滤器。
- `-Hamming`：Hamming过滤器。
- `-Blackman`：Blackman过滤器。
- `-Kaiser`：Kaiser过滤器。

## 压缩块大小

- `-n4`：法线贴图 4 样本
- `-n3x3`：法线贴图 3x3 滤波
- `-n5x5`：法线贴图 5x5 滤波
- `-n7x7`：法线贴图 7x7 滤波
- `-n9x9`：法线贴图 9x9 滤波
- `-dudv`：DuDv

## 高度信息的来源

- `-alpha`：alpha 通道
- `-rgb`：平均 RGB
- `-biased`：偏置平均 RGB
- `-red`：红色通道
- `-green`：绿色通道
- `-blue`：蓝色通道
- `-max`：RGB 中的最大值
- `-colorspace`：RGB 的混合
- `-norm`：标准化 mip 贴图（来源是法线贴图）
- `-toHeight`：创建高度图（来源是法线贴图）

## 法线/DuDv 贴图 dxt 选项

- `-aheight`：将计算的高度存储在 alpha 通道中
- `-aclear`：清除 alpha 通道
- `-awhite`：将 alpha 通道设置为 1.0
- `-scale <float>`：高度图的缩放比例。默认值为 1.0
- `-wrap`：环绕纹理。默认关闭
- `-minz <int>`：向上向量的最小值 (0-255)。默认值为 0

## 制作 Depth Sprite

- `-depth`

## Depth Sprite的来源

- `-alpha`：alpha 通道
- `-rgb`：平均 RGB（默认）
- `-red`：红色通道
- `-green`：绿色通道
- `-blue`：蓝色通道
- `-max`：RGB 中的最大值
- `-colorspace`：RGB 的混合

## Depth Sprite dxt 选项

- `-aheight`：将计算的深度存储在 alpha 通道中
- `-aclear`：在 alpha 通道中存储 0.0
- `-awhite`：在 alpha 通道中存储 1.0
- `-scale <float>`：深度精灵的缩放比例（默认 1.0）
- `-alpha_modulate`：在滤波过程中将颜色乘以 alpha
- `-pre_modulate`：在处理之前将颜色乘以 alpha


# 过滤选项

这些过滤器（filters）在图像处理和信号处理领域中用于不同的插值和重采样方法。每种过滤器都有其独特的特性和适用场景。以下是对这些过滤器的简要介绍及其适用场景：

1. **Point（点过滤器）**：
   - **特点**：最简单的过滤器，直接取最近的像素值。
   - **适用场景**：需要快速处理且对质量要求不高的场景。

2. **Box（盒子过滤器）**：
   - **特点**：平均值过滤器，计算邻域内像素的平均值。
   - **适用场景**：简单的平滑处理，但可能会导致图像模糊。

3. **Triangle（三角形过滤器）**：
   - **特点**：线性插值，权重随距离线性变化。
   - **适用场景**：比盒子过滤器稍好的平滑效果，适用于简单的插值。

4. **Quadratic（二次过滤器）**：
   - **特点**：二次插值，权重随距离二次变化。
   - **适用场景**：需要更平滑的插值效果。

5. **Cubic（立方体过滤器）**：
   - **特点**：三次插值，权重随距离三次变化。
   - **适用场景**：高质量的插值，适用于图像放大。

6. **Catrom（Catmull-Rom过滤器）**：
   - **特点**：一种特殊的三次插值，平滑且锐利。
   - **适用场景**：需要平滑且锐利的图像插值。

7. **Mitchell（Mitchell过滤器）**：
   - **特点**：平滑且减少振铃效应的三次插值。
   - **适用场景**：高质量图像处理，减少振铃效应。

8. **Gaussian（高斯过滤器）**：
   - **特点**：权重随距离呈高斯分布，平滑效果好。
   - **适用场景**：需要平滑处理且减少噪声的场景。

9. **Sinc（Sinc过滤器）**：
   - **特点**：理论上最优的重建滤波器，但计算复杂。
   - **适用场景**：高质量重建，适用于高精度要求的场景。

10. **Bessel（贝塞尔过滤器）**：
    - **特点**：基于贝塞尔函数，平滑效果好。
    - **适用场景**：需要平滑处理的场景。

11. **Hanning（Hanning过滤器）**：
    - **特点**：权重随距离呈余弦分布，平滑效果好。
    - **适用场景**：需要平滑处理且减少边缘效应的场景。

12. **Hamming（Hamming过滤器）**：
    - **特点**：类似Hanning，但有不同的余弦系数。
    - **适用场景**：需要平滑处理且减少边缘效应的场景。

13. **Blackman（Blackman过滤器）**：
    - **特点**：权重随距离呈三项余弦分布，平滑效果更好。
    - **适用场景**：需要极高平滑效果的场景。

14. **Kaiser（Kaiser过滤器）**：
    - **特点**：基于Kaiser窗函数，平滑且灵活。
    - **适用场景**：需要平滑处理且可调节参数的场景。


- **快速处理**：Point, Box
- **简单插值**：Triangle, Quadratic
- **高质量插值**：Cubic, Catrom, Mitchell
- **平滑处理**：Gaussian, Bessel, Hanning, Hamming, Blackman, Kaiser
- **高精度重建**：Sinc

# 贴图格式

这些贴图格式在图像处理和纹理压缩中有不同的用途和特点。以下是对这些格式的简要介绍及其适用场景：

## 压缩格式
1. **DXT1c**：DXT1（仅颜色）
   - **特点**：不支持alpha通道，压缩比高。
   - **适用场景**：不需要透明度的纹理。

2. **DXT1a**：DXT1（1位alpha）
   - **特点**：支持1位alpha通道（透明或不透明）。
   - **适用场景**：简单透明度效果的纹理。

3. **DXT3**：
   - **特点**：支持4位alpha通道，适用于有明确边界的透明度。
   - **适用场景**：带有硬边透明度的图像。

4. **DXT5**：DXT5n
   - **特点**：支持4位渐变alpha通道，适用于平滑的透明度过渡。
   - **适用场景**：高质量alpha通道的纹理。

## 未压缩格式
5. **u1555**：未压缩的1:5:5:5
   - **特点**：16位颜色格式，1位alpha。
   - **适用场景**：需要简单透明度的高质量纹理。

6. **u4444**：未压缩的4:4:4:4
   - **特点**：16位颜色格式，4位alpha。
   - **适用场景**：需要中等质量透明度的纹理。

7. **u565**：未压缩的5:6:5
   - **特点**：16位颜色格式，无alpha。
   - **适用场景**：不需要透明度的高质量纹理。

8. **u8888**：未压缩的8:8:8:8
   - **特点**：32位颜色格式，8位alpha。
   - **适用场景**：高质量且包含alpha通道的纹理。

9. **u888**：未压缩的0:8:8:8
   - **特点**：24位颜色格式，无alpha。
   - **适用场景**：高质量不透明纹理。

10. **u555**：未压缩的0:5:5:5
    - **特点**：15位颜色格式，无alpha。
    - **适用场景**：不需要透明度的中等质量纹理。

## 调色板格式
11. **p8c**：调色板8位（256色）
    - **特点**：使用256色调色板，压缩比高。
    - **适用场景**：颜色数量有限的纹理。

12. **p8a**：调色板8位（256色带alpha）
    - **特点**：使用256色调色板，支持alpha通道。
    - **适用场景**：颜色数量有限且需要透明度的纹理。

13. **p4c**：调色板4位（16色）
    - **特点**：使用16色调色板，压缩比更高。
    - **适用场景**：颜色数量非常有限的纹理。

14. **p4a**：调色板4位（16色带alpha）
    - **特点**：使用16色调色板，支持alpha通道。
    - **适用场景**：颜色数量非常有限且需要透明度的纹理。

## 特殊用途格式
15. **a8**：8位alpha通道
    - **特点**：单通道alpha图像。
    - **适用场景**：仅需要透明度信息的纹理。

16. **cxv8u8**：法线贴图格式
    - **特点**：用于法线贴图，两个8位分量。
    - **适用场景**：法线贴图。

17. **v8u8**：EMBM格式（8位，两个分量有符号）
    - **特点**：两个8位有符号分量。
    - **适用场景**：环境映射凹凸贴图（EMBM）。

18. **v16u16**：EMBM格式（16位，两个分量有符号）
    - **特点**：两个16位有符号分量。
    - **适用场景**：高精度环境映射凹凸贴图（EMBM）。

19. **A8L8**：8位alpha通道，8位亮度
    - **特点**：8位亮度和8位alpha。
    - **适用场景**：需要亮度和透明度信息的纹理。

20. **fp32x4**：fp32四通道（A32B32G32R32F）
    - **特点**：32位浮点四通道。
    - **适用场景**：高动态范围（HDR）图像。

21. **fp32**：fp32单通道（R32F）
    - **特点**：32位浮点单通道。
    - **适用场景**：高精度单通道数据。

22. **fp16x4**：fp16四通道（A16B16G16R16F）
    - **特点**：16位浮点四通道。
    - **适用场景**：高动态范围（HDR）图像。

23. **dxt5nm**：DXT5风格法线贴图
    - **特点**：用于法线贴图的DXT5格式。
    - **适用场景**：法线贴图。

24. **3Dc**：3DC
    - **特点**：高质量法线贴图压缩格式。
    - **适用场景**：法线贴图。

25. **g16r16**：16位输入，两个分量
    - **特点**：两个16位分量。
    - **适用场景**：高精度图像数据。

26. **g16r16f**：16位浮点，两个分量
    - **特点**：两个16位浮点分量。
    - **适用场景**：高精度浮点图像数据。


- **不透明或简单透明度**：DXT1c, DXT1a
- **明确边界的透明度**：DXT3
- **平滑透明度过渡**：DXT5
- **高质量无压缩**：u8888, u888
- **中等质量无压缩**：u1555, u4444, u565, u555
- **颜色数量有限**：p8c, p8a, p4c, p4a
- **仅透明度信息**：a8
- **法线贴图**：cxv8u8, dxt5nm, 3Dc
- **高精度图像数据**：fp32x4, fp32, fp16x4, g16r16, g16r16f
- **环境映射凹凸贴图**：v8u8, v16u16
