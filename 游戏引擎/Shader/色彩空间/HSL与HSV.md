`HSL(hue, saturation, lightness)`：色相，饱和度，亮度；
`HSV/HSB(hue, saturation, value/brightness)`：色相，饱和度，亮度值；

`HSL`与`HSV`是`RGB`色彩模型的替代，更贴近人类视觉对色彩的感知；

- 色相：颜色的种类（红、蓝等）；
- 饱和度：表示颜色的纯度，从纯色到灰色；
- 亮度值：颜色的明亮程度；


![HSL/HSV|450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230413105139.png)

# 区别

`HSL`模拟了现实世界中不同颜料混合在一起创造颜色的方式，`lightness`维度表示黑色和白色颜料的不同比例，完全饱和的颜色位于`lightness=0.5`的圆圈周围；

与`HSV/HSB`的主要区别在于，`HSL`的亮度
范围为黑色到白色，而`HSV/HSB`的亮度值是基于颜色的亮度，范围为黑色到纯色；

`HSL`中的黑色和白色，在`HSV`中相当于用黑色和白色光线照耀有色物体得到的颜色；

# 转换

## RGB到HSB

```c++
vec3 rgb2hsb( in vec3 c )
{
    vec4 K = vec4(0.0, -1.0 / 3.0, 2.0 / 3.0, -1.0);
    vec4 p = mix(vec4(c.bg, K.wz),
                 vec4(c.gb, K.xy),
                 step(c.b, c.g));
    vec4 q = mix(vec4(p.xyw, c.r),
                 vec4(c.r, p.yzx),
                 step(p.x, c.r));
    float d = q.x - min(q.w, q.y);
    float e = 1.0e-10;
    return vec3(abs(q.z + (q.w - q.y) / (6.0 * d + e)), d / (q.x + e), q.x);
}
```

## HSB到RGB

```c++
//  Function from Iñigo Quiles
//  https://www.shadertoy.com/view/MsS3Wc
vec3 hsb2rgb( in vec3 c ){
    vec3 rgb = clamp(abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),6.0)-3.0)-1.0,0.0,1.0 );
    rgb = rgb*rgb*(3.0-2.0*rgb);
    return c.z * mix(vec3(1.0), rgb, c.y);
}
```