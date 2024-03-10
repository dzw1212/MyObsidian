```cpp
ImGuiContext& g = *GImGui;
ImGuiIO& io = g.IO;
ImGuiMetricsConfig* cfg = &g.DebugMetricsConfig;
```

# Debug日志窗口


# ID堆栈窗口

```cpp
Checkbox("Show ID Stack Tool", &cfg->ShowIDStackTool);
```

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240307140616.png)
