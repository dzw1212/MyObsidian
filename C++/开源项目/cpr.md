*https://github.com/libcpr/cpr*

`CPR` 是一个 C++ 库，用于简化 `HTTP` 请求的发送。它是对流行的 C++ `HTTP` 客户端库 `libcurl` 的封装，提供了一个现代的、简洁的 API。

测试url可用：[[httpbin.org]]
解析json数据可用：[[nlohmann-json]]

```cpp
#include <cpr/cpr.h>
#include <iostream>

int main() {
    // 发送 GET 请求
    cpr::Response r = cpr::Get(cpr::Url{"http://httpbin.org/get"});
    std::cout << "Status code: " << r.status_code << std::endl; // HTTP 状态码
    std::cout << "Response body: " << r.text << std::endl; // 响应体

    // 发送 POST 请求
    r = cpr::Post(cpr::Url{"http://httpbin.org/post"},
                  cpr::Payload{{"key", "value"}});
    std::cout << "Status code: " << r.status_code << std::endl;
    std::cout << "Response body: " << r.text << std::endl;

    return 0;
}
```

带有`Header`和`Content`的`Post`请求：
```cpp
#include <cpr/cpr.h>
#include <iostream>

int main() {
    // 创建一个 Header 对象，包含多个头部字段
    cpr::Header headers = {
        {"Content-Type", "application/json"},
        {"Accept", "application/json"},
        {"Authorization", "Bearer your_token_here"}
    };

    // 创建一个 Body 对象，包含 JSON 格式的数据
    cpr::Body body = cpr::Body("{\"key1\":\"value1\", \"key2\":\"value2\"}");

    // 发送 POST 请求，同时附带 JSON 数据和自定义头部
    cpr::Response r = cpr::Post(
        cpr::Url{"http://httpbin.org/post"},
        headers,
        body
    );

    std::cout << "Status code: " << r.status_code << std::endl;
    std::cout << "Response body: " << r.text << std::endl;

    return 0;
}
```

# 整合到项目

直接使用自带的`CMakeLists.txt`生成项目；

在目录`D:\OpenSrcLib\cpr\cpr-master`下：
```console
mkdir build
cd build
cmake ..
```

生成`cpr.sln`后，使用`vs`进行编译（区分`Release`和`Debug`）；

生成的动态库文件：`D:\OpenSrcLib\cpr\cpr-master\build\bin\Debug`

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240508134820.png)


生成的静态库文件：`D:\OpenSrcLib\cpr\cpr-master\build\lib\Debug`

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240508135030.png)

将静态库文件进行链接，动态库文件放在exe同级目录下；

---

除此之外，还需要复制一些头文件；

`D:\OpenSrcLib\cpr\cpr-master\include`

`D:\OpenSrcLib\cpr\cpr-master\build\_deps\curl-src\include`

---

premake5.lua：

```lua
workspace "Curl"
    configurations { "Debug", "Release" }
    architecture "x86_64"

project "Curl"
    kind "ConsoleApp"
    language "C++"
    cppdialect "C++latest"

    targetdir "bin/%{cfg.buildcfg}"
    objdir "bin-int/%{cfg.buildcfg}"

    files
    {
        "src/**.h",
        "src/**.cpp",
    }

    defines
    {
        "_CRT_SECURE_NOT_WARNINGS",
    }

    includedirs
    {
        "src/",
        "include/",
        "curl-src/include/",
        "zlib-src/include/",
    }

	--库文件目录
    libdirs{}

	--库文件
    links
    {
        "lib/Debug/cpr.lib",
        "lib/Debug/libcurl-d_imp.lib",
        "lib/Debug/zlib.lib",
    }

    filter "configurations:Debug"
        defines { "DEBUG" }
        symbols "On"

    filter "configurations:Release"
        defines { "NDEBUG" }
        optimize "On"
```

# 非阻塞请求

```cpp
#include "cpr/cpr.h"
#include <iostream>
#include <chrono>

void TestCallback(const cpr::Response& response, void* userdata) {
    std::cout << "Status code: " << response.status_code << std::endl;
    std::cout << "Response text: " << response.text << std::endl;
    std::cout << *static_cast<std::string*>(userdata) << std::endl;
}

int main() {
    std::string strUserData = "test123";

    // 发起异步 GET 请求
    cpr::GetCallback(
        [&](const cpr::Response& r) { TestCallback(r, &strUserData); },
        cpr::Url{ "http://httpbin.org/delay/3" }
    );

    // 等待足够的时间以确保回调被执行
    std::this_thread::sleep_for(std::chrono::seconds(5));

    return 0;
}
```