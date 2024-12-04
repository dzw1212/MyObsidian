.rc 文件是 Windows 资源脚本文件，用于定义应用程序的资源，如图标、位图、菜单、对话框、字符串等；

.rc 文件的格式是由微软制定的，它是 Windows 操作系统的一部分，用于定义应用程序的资源。资源脚本文件格式是 Windows API 的一部分，最初是为 Windows 平台上的应用程序开发而设计的；

微软提供了资源编译器（如 `rc.exe`）来处理这些 .rc 文件，将它们编译成二进制资源文件（.res），然后可以链接到可执行文件中；

虽然 .rc 文件在 MSVC 中使用得最为广泛，但它们并不是 MSVC 独有的，其他编译器也可以支持它们，例如，`MinGW` 和 `Cygwin` 等工具链也可以处理 .rc 文件。通常，这些工具会使用 `windres.exe` 来编译 .rc 文件；

---

通常，.rc 文件会包含一个头文件（如 resource.h），其中定义了所有资源标识符的常量，这有助于在代码中引用资源时保持一致性；



# VS添加资源文件

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241115145048.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241115151135.png)


导入后，VS会新建或修改已存在的`.rc`与`resource.h`文件，添加导入的资源；







# premake中指定rc文件

在使用 Premake 配置项目时，你可以通过在 `premake5.lua` 文件中指定资源文件来包含 `.rc` 文件；Premake 会自动处理这些文件并将它们包含在生成的项目中；

```lua
workspace "MyWorkspace"
    configurations { "Debug", "Release" }

project "MyProject"
    kind "WindowedApp"
    language "C++"
    files { "src/**.cpp", "src/**.h", "resources/**.rc" }  -- 包含 .rc 文件

    filter "configurations:Debug"
        defines { "DEBUG" }
        symbols "On"

    filter "configurations:Release"
        defines { "NDEBUG" }
        optimize "On"
```

# FASTBuild中指定rc文件

[[FASTBuild#RC资源文件处理]]
