# RTX硬件

在显卡上新增了一种硬件（`RT Core`），用于大量的并行的计算光线与场景求交，从而加速`Ray Tracing`的计算；
# SPP

`SPP`（`Sampler Per Pixel`），表示对于每个像素采样的样本数；采样次数越多，图像的噪声就越少，但同时渲染的计算量也就越大；

在经典的直接光照+一次间接光照的`Path Tracing`中，一个`SPP`包括：

![SPP|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230927145007.png)

![SPP|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230927145043.png)

# 降噪

较低的`SPP`代表了较低的计算量，也代表了大量的噪声；

![denoise|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230927150040.png)

# 总结

实现RTRT的思路即为：使用`RTX`硬件进行较低的`SPP`（比如1`SPP`）运算，得到充斥大量噪声的结果，再使用合适的滤波方法进行降噪，恢复出原图像；
