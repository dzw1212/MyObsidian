
# 优点

相对于载入多张贴图，载入一张由多个texture合成的贴图，能够显著减少对GPU有限的Texture寄存器的依赖；
GPU支持的`TextureArray`中的图片的数量也是有限的，可以通过`VkPhysicalDeviceLimits.maxImageArrayLayers`查询；




着色器中将采样器类型由`sampler2D`改为`sampler2DArray`；


http://xiaopengyou.fun/public/2019/08/19/14_TextureArray/