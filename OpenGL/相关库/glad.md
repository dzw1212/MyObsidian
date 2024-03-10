*https://github.com/Dav1dde/glad*

*https://glad.dav1d.de/*

GLAD是一个用于管理OpenGL函数指针的库，它通过一个在线服务生成C或C++代码，使得开发者能够在他们的应用程序中加载和使用OpenGL函数。OpenGL本身是一个底层的图形API，提供了大量的绘图功能，但它是由显卡驱动实现的，这意味着OpenGL函数的实际执行代码存在于运行它的系统上的驱动软件中。因此，为了在程序中使用OpenGL函数，你需要在运行时从驱动软件中查询并获取这些函数的地址。这个过程称为“加载OpenGL函数”，GLAD就是用来简化这一过程的工具之一；

# 选择版本

以下是一些基本的指导步骤：
1. 访问GLAD在线服务：首先，访问GLAD的在线服务网站。
2. 选择语言：在“Language”选项中，通常你会选择“C/C++”。
3. 选择规范：在“Specification”选项中，选择“OpenGL”。
4. 选择API版本：
	在“API gl”部分，选择你需要的OpenGL版本。如果你不确定，OpenGL 3.3 是一个常用的起点，因为它是现代OpenGL编程的基础，同时兼容大多数现代硬件。
	“Version”下拉菜单允许你选择具体的版本号。选择“3.3”或你项目所需的其他版本。
5. 选择模式：在“Profile”选项中，你可以选择“Core”或“Compatibility”。
	Core：只包含该版本及以后版本的功能，不包含任何被废弃的特性。推荐使用Core，特别是对于新的项目。
	Compatibility：包含所有功能，包括旧版本的和已废弃的特性。
6. 生成加载器：
	勾选“Generate a loader”选项，GLAD会生成用于加载OpenGL函数的代码。
7. 扩展（可选）：
	如果你需要特定的OpenGL扩展，可以在“Extensions”部分搜索并选择它们。对于大多数项目，你可能不需要手动选择扩展，除非你有特定的需求。
8. 生成：
	点击页面底部的“GENERATE”按钮。GLAD会根据你的选择生成一个ZIP文件，其中包含了所有必要的源代码和头文件。
9. 下载并集成：
	下载生成的ZIP文件，并将其解压到你的项目中合适的位置。确保你的项目包含了GLAD生成的源文件，并且正确设置了头文件的包含路径。

# 使用基本步骤

1. 生成加载器代码：访问GLAD的在线服务（如GLAD web service），选择你的OpenGL版本和需要的扩展，然后生成并下载加载器代码；
2. 集成到项目中：将生成的代码添加到你的项目中。这通常包括两个文件：`glad.c`（或`glad.cpp`）和`glad.h`；
3. 初始化GLAD：在你的应用程序初始化OpenGL之后（例如，在创建OpenGL上下文之后），调用GLAD提供的初始化函数来加载OpenGL函数指针；
4. 使用OpenGL：一旦GLAD初始化完成，你就可以在你的程序中自由使用OpenGL函数了；

# 验证示例代码

需要先配置好[[glfw]]；

```cpp
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>

int main() {
    // 初始化GLFW
    if (!glfwInit()) {
        std::cerr << "Failed to initialize GLFW" << std::endl;
        return -1;
    }

    // 创建窗口
    GLFWwindow* window = glfwCreateWindow(800, 600, "GLAD and GLFW Example", NULL, NULL);
    if (!window) {
        std::cerr << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }

    // 使窗口的OpenGL上下文成为当前线程的当前上下文
    glfwMakeContextCurrent(window);

    // 初始化GLAD
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
        std::cerr << "Failed to initialize GLAD" << std::endl;
        return -1;
    }

    // 设置视口
    glViewport(0, 0, 800, 600);

    // 注册窗口大小改变的回调函数
    glfwSetFramebufferSizeCallback(window, [](GLFWwindow* window, int width, int height) {
        glViewport(0, 0, width, height);
    });

    // 渲染循环
    while (!glfwWindowShouldClose(window)) {
        // 清除颜色缓冲
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // 检查并调用事件，交换缓冲
        glfwPollEvents();
        glfwSwapBuffers(window);
    }

    // 释放资源
    glfwTerminate();
    return 0;
}
```

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240301143542.png)
