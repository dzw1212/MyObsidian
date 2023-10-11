
对于一些非常亮的噪声（在图像中表现为明亮的点，通常在`TAA`或`Temporal Clamping`中会产生），使用高斯滤波等滤波方法无法有效去除，还会将明亮的点转为明亮的块；

这样的噪点称为离群值（`Outlier`），或者在图形学中称为`Firefly`，需要在滤波之前处理掉；

![RTRT|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230928182834.png)

# 检测

对于每个`Pixel`，取一个区域（如 $7\times 7$ ），计算出这个区域的均值 $\mu$ 和方差 $\sigma^2$ ，绝大多数正常值的范围应该在 $[\mu-k\sigma, \mu+k\sigma]$ 之间，则不在这个范围内的点被认为是`Outlier`；

# 去除

将该点的值`Clamp`到 $[\mu-k\sigma, \mu+k\sigma]$ 之间；
