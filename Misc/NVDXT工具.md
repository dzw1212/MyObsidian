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

