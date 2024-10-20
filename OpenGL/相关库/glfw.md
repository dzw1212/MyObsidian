
*https://github.com/glfw/glfw*

GLFW是一个开源的、多平台的库，用于创建窗口、接收输入和处理事件，主要用于OpenGL、OpenGL ES和Vulkan图形应用程序的开发。它提供了一个简单的API，帮助开发者在不同操作系统上创建和管理窗口以及处理用户输入，而无需深入了解每个平台的底层细节。GLFW是专为游戏和实时图形应用程序设计的，因此它特别注重性能和简洁性；

# 编译配置

区分平台（由于glfw的更新，可能会新增c文件，因此该配置特定于某个版本，仅限于参考）；

```lua
project "GLFW"
	kind "StaticLib"
	language "C"

	targetdir "bin/%{cfg.buildcfg}"
    objdir "bin-int/%{cfg.buildcfg}"

	files
	{
		"include/GLFW/glfw3.h",
		"include/GLFW/glfw3native.h",
		"src/glfw_config.h",
		"src/context.c",
		"src/init.c",
		"src/input.c",
		"src/monitor.c",
		"src/vulkan.c",
		"src/window.c"
	}

	filter "system:windows"
		systemversion "latest"
		staticruntime "Off"

		files
		{
			"src/win32_init.c",
			"src/win32_joystick.c",
			"src/win32_monitor.c",
			"src/win32_time.c",
			"src/win32_thread.c",
			"src/win32_window.c",
			"src/wgl_context.c",
			"src/egl_context.c",
			"src/osmesa_context.c"
		}

		defines
		{
			"_GLFW_WIN32",
			"_CRT_SECURE_NO_WARNINGS"
		}

	filter "configurations:Debug"
		runtime "Debug"
		symbols "on"

	filter "configurations:Release"
		runtime "Release"
		optimize "on"

```

全部编译：

```cpp
project "GLFW"
	kind "StaticLib"
	language "C"

	targetdir "bin/%{cfg.buildcfg}"
    objdir "bin-int/%{cfg.buildcfg}"

    defines
    {
        "_GLFW_WIN32",
        "_CRT_SECURE_NO_WARNINGS"
    }

	files
	{
		"include/GLFW/glfw3.h",
		"include/GLFW/glfw3native.h",
		"src/glfw_config.h",
		"src/*.c",
	}

	filter "configurations:Debug"
		runtime "Debug"
		symbols "on"

	filter "configurations:Release"
		runtime "Release"
		optimize "on"

```

# 验证示例代码

```cpp
#include "GLFW/glfw3.h"
#include <iostream>

int main(void) {
    GLFWwindow* window;

    // 初始化库
    if (!glfwInit()) {
        std::cerr << "Failed to initialize GLFW\n";
        return -1;
    }

    // 创建一个窗口及其 OpenGL 上下文
    window = glfwCreateWindow(640, 480, "Hello GLFW", NULL, NULL);
    if (!window) {
        std::cerr << "Failed to create GLFW window\n";
        glfwTerminate();
        return -1;
    }

    // 使窗口的上下文成为当前
    glfwMakeContextCurrent(window);

    // 循环，直到用户关闭窗口
    while (!glfwWindowShouldClose(window)) {
        // 渲染这里

        // 交换缓冲区并处理事件
        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwTerminate(); // 清理资源
    return 0;
}
```

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240301113240.png)

# 设置应用图标

```cpp
GLFWimage images[1];
images[0].pixels = stbi_load("./asset/appicon.png", &images[0].width, &images[0].height, 0, 4); // RGBA
glfwSetWindowIcon(g_Window, 1, images);
stbi_image_free(images[0].pixels);
```