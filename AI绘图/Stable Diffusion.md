*https://jalammar.github.io/illustrated-stable-diffusion/

*https://zhuanlan.zhihu.com/p/597247221

Stable Diffusion（以下简称SD）是一种机器学习模型，经过训练可以逐步对随机高斯噪声进行去噪以获得感兴趣的样本，比如说生成图像；

# 组成部分

`语义解析(Text-Understanding)`：一个特别的**Transform**语言模型，输入是人类语言，输出是**语义向量**；
`图片生成器(Image Generator)`：由**图片信息生成器**和**图片解码器**构成；

- 图片信息生成器：不直接生成图片，而是生成较低纬度的图片信息，即*隐空间信息(latent space information)*，这也是SD相比其他扩散模型速度更快、显存占用更低的原因 —— 一般的扩散模型是直接生成图片，因此需要生成的信息更多，负载也更大；在生成隐空间信息的过程中，*schedule算法*控制生成的进度，*神经网络unet*执行生成的过程，整个生成迭代过程大概需要50~100次，隐变量的质量也随着迭代次数而提升；

- 图片解码器：接收图片的隐变量，将其升维放大还原成一张完整的图片；

![SD_component|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320092200.png)

![SD_component2|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320102444.png)

# 还原图片

## Diffusion模型扩散过程

首先使用random函数生成一个隐变量大小的纯噪声②，加上语义解码器输出的语义向量①，在图片信息生成器中执行扩散过程，unet根据语义向量不断去除噪声，同时不断向隐变量中注入语义信息，50~100次之后就得到了一个有语义的隐变量③，在此期间schedule算法统筹整个去噪的过程；

![SD_diffusion|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320111756.png)

![SD_diffusion|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320113239.png)

## 训练扩散模型

训练扩散模型，简单来说就是训练unet学会**去噪(denoising)**；

Step1. 从训练集中选取一张加噪后的图片与对应的噪声强度；
Step2. 输入unet，让unet预测出噪声图；
Step3. 计算预测的噪声图与实际噪声图之间的误差；
Step4. 根据误差反馈更新unet的参数；

![SD_unetTrain|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320132325.png)

## 迭代去噪

使用unet预测噪声等级，使加噪后的图片减去该等级的噪声图，就得到的轻微降噪的图片；然后重复该过程，直到得到近似的原图片；

还原出的图片和训练集保有相同的像素规律，也就是说也可以==根据训练集来改变还原图片的画风==；

![SD_denoising|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320132952.png)

![SD_denoising2|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320133035.png)


# 结合语义信息

## 关联语义与图片 - CLIP模型

首先，有一个结合人类语言与计算机视觉的数据集，其中的每张图片都带有一定数量的人类语言描述标签；

![SD_clip1|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320134732.png)

CLIP模型中包含一个**图片Encoder**和**文字Encoder**，分别用于将图片和文字压缩成图片embedding变量和文字embedding变量，使用**余弦相似度**来比较两个embedding变量的相似性；

![SD_clip2|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320140327.png)

一开始，CLIP模型中的参数都是初始值，计算出的两个embedding变量大概率是大相径庭的，然后根据对比出的结果来反向更新CLIP模型的参数，不断重复这个过程，直到——对于对应的图片和文字，变量的近似度为1；对于不对应的图片和文字，变量的近似度为0；

## 注入语义信息 - Attention机制

使用CLIP模型的文本Encoder将自然语言转为语义embedding变量，与加噪图、噪声强度一同作为参数传入，由unet解析出噪声图；

![SD_clip3|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320203141.png)

![SD_clip4|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230320203749.png)

