# 预置的uniform

- `float iTime`：自着色器开始运行以来的时间，单位s；
- `vec3 iResolution`：当前输出图像的分辨率，格式为`vec3(w, h, depth)`，深度通常为1.0；
- `vec4 iMouse`：`iMouse.xy`为鼠标当前位置，`iMouse.zw`为鼠标的点击位置；
- `float iTimeDelta`：上一帧与当前帧之间的时间差，单位s；
- `int iFrame`：当前的运行帧数；
- `vec4 iDate`：`iDate.x`为年份，`iDate.y`为月份，`iDate.z`为日期，`iDate.w`为秒数；
- `float iSampleRate`：音频采样率；
- `vec3 iChannelResolution[4]`：每个通道的分辨率；
- `float iChannelTime[4]`：每个通道的时间；
- `sampler2D iChannel0|iChannel1|iChannel2|iChannel3`：输入通道的纹理采样器；

# 常用函数

- `step(edge, x)`：小于`edge`为0.0，大于`edge`为1.0；
- `smoothstep(edge0, edge1, x)`：小于`edge0`为0.0，大于`edge1`为1.0，之间则插值；

# 绘制

## 函数曲线

主要通过`smoothstep`来设置线条的宽度，从而绘制函数（0.0-1.0区间内）；

<body>
    <div id="glsl_editor"></div>
</body>
```glsl
float plot_func(float x, float y)
{
    return abs(y - pow(x, 2.0));
}

void main() 
{
    vec2 st = gl_FragCoord.st/iResolution.xy;
    vec3 greenColor = vec3(0.0, 1.0, 0.0);
    float t = smoothstep(0.005, 0.0, plot_func(st.x, st.y));
    gl_FragColor = vec4(t * greenColor, 1.0);
}
```

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241110195332.png)

