`HSL(hue, saturation, lightness)`：色相，饱和度，亮度；
`HSV/HSB(hue, saturation, value/brightness)`：色相，饱和度，色彩值；
HSL与HSV是RGB色彩模型的替代，更贴近人类视觉对色彩的感知；

![HSL/HSV|450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230413105139.png)

## HSL

HSL模拟了现实世界中不同颜料混合在一起创造颜色的方式，lightness维度表示黑色和白色颜料的不同比例，完全饱和的颜色位于lightness=0.5的圆圈周围；

## HSV

HSV模拟了颜色在光线下的显示；

## 区别

HSL中的黑色和白色，在HSV中相当于用黑色和白色光线照耀有色物体得到的颜色；

## 转换

### RGB到HSB

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

### HSB到RGB

```c++
//  Function from Iñigo Quiles
//  https://www.shadertoy.com/view/MsS3Wc
vec3 hsb2rgb( in vec3 c ){
    vec3 rgb = clamp(abs(mod(c.x*6.0+vec3(0.0,4.0,2.0),6.0)-3.0)-1.0,0.0,1.0 );
    rgb = rgb*rgb*(3.0-2.0*rgb);
    return c.z * mix(vec3(1.0), rgb, c.y);
}
```