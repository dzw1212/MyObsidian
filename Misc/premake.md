*https://premake.github.io/

premake是一个通过lua脚本生成工程文件的命令行工具，支持VS、XCode、GNU Make等；

## 生成工程文件

![premake1|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221210194412.png)

![premake2|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221210194527.png)

![premake3|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20221210195038.png)

如果不想使用默认的premake5.lua作为脚本名称，可以指定脚本：
```cmd
$ premake5 --file=MyProjectScript.lua vs2013
```



## 脚本示例

对于一个简单的c程序，
```lua
-- premake5.lua
workspace "HelloWorld"
   configurations { "Debug", "Release" }

project "HelloWorld"
   kind "ConsoleApp"
   language "C"
   targetdir "bin/%{cfg.buildcfg}"

   files { "**.h", "**.c" }

   filter "configurations:Debug"
      defines { "DEBUG" }
      symbols "On"

   filter "configurations:Release"
      defines { "NDEBUG" }
      optimize "On"
```

## 语法

### 函数与参数

每一行都是一个函数调用，为了便于阅读，通常可以省略括号：
```lua
workspace "HelloWorld" --等同于 workspace("HelloWorld")
```

但是如果是使用变量或者拼接字符串作为参数，则不能省略括号：
```lua
local lang = "C++"  
language (lang)  
  
workspace("HelloWorld" .. _ACTION)
```


### 值与列表

premake的大多数函数接受单个字符串或者字符串列表作为参数，如果碰到多个值，以最后一个为准；
```lua
language "C++"   -- the value is now "C++"
language "C"     -- the value is now "C"
```

列表支持组合和移除值；
```lua
--参数可以组合
defines { "DEBUG", "TRACE" }
defines { "WINDOWS" }         -- 当前列表为 { "DEBUG", "TRACE", "WINDOWS" }

--参数也可以移除
defines { "DEBUG", "TRACE" }
removedefines { "TRACE" }     -- 当前列表为 { "DEBUG" }
```

### 路径

1. 路径始终相对于脚本文件所在的路径；
2. 使用"/"作为路径分隔符；

### 工作区 workspace

使用关键词`workspace`定义；
每个build都有一个workspace作为project的容器，对应于VS中的solution；
workspace定义一组通用的configuration和platform，用于其所包含的所有project；
```lua
workspace "HelloWorld"
   configurations { "Debug", "Release" }
```

### 配置 configuration

使用关键词`configurations`定义；
configuration是一系列设置的集合，包括标志flags、开关switch、头文件、库搜索目录等；
大多数IDE提供的默认配置为`Debug`和`Release`，但是配置名不仅限于这些名称，可以使用你喜欢的任意名称，这些名称是没有意义的，有意义的是名称所对应的一系列设置；
```lua
workspace "HelloWorld"
   configurations { "Debug", "Release" }

   filter "configurations:Debug" --配置Debug相关设置
      defines { "DEBUG" }
      flags { "Symbols" }

   filter "configurations:Release" --配置Release相关设置
      defines { "NDEBUG" }
      optimize "On"
```

project无法向workspace中添加配置，只能删除对现有配置的支持，或者使用`configmap`将其映射到其他项目配置；
```lua
workspace "MyWorkspace"
    configurations { "Debug", "Development", "Profile", "Release" }

project "UnitTest"
    configmap {
        ["Development"] = "Debug",
        ["Profile"] = "Release"
    }
```


### 平台 platform

使用关键词`platforms`定义；
实际上，平台只是另一组用于构建配置的名称，提供了另一个用于配置项目的轴，可以使用某个configuration+某个平台的任意组合；
类似于configuration，platform的名称本身没有意义，对应的设置才有意义；
```lua
configurations { "Debug", "Release" }
platforms { "Win32", "Win64", "Xbox360" }

filter { "platforms:Win32" }
    system "Windows"
    architecture "x86"

filter { "platforms:Win64" }
    system "Windows"
    architecture "x86_64"

filter { "platforms:Xbox360" }
    system "Xbox360"
```

不同于configuration的是，platform是可选的，如果不设置则使用默认行为；

### 过滤器 filter

使用关键词`filter`定义；
filter用于提取特定的configuration、platform、operating system、target action等，将后续的设置限制在特定的环境内；
filter的格式为`Prefix:MatchPattern`；

其中前缀支持的有（如果未指定前缀，则默认为configurations）：
1. action
2. architecture
3. configurations
4. files
5. kind
6. language
7. options
8. platforms
9. system
10. toolset


```lua
workspace "MyWorkspace"  
	configurations { "Debug", "Release" }  
  
	filter "configurations:Debug"  
		defines { "_DEBUG" } --定义仅适用于调试版本的符号
	
	filter "action:vs2010"  --仅在面向 Visual Studio 2010 时定义符号
		defines { "VISUAL_STUDIO_2005" }

	filter "action:vs*"  --为所有版本的 Visual Studio 定义一个符号
		defines { "VISUAL_STUDIO" }

	filter "kind:SharedLib or StaticLib"  --动态库或静态库
		defines { "LIBRARY_TARGET" }

	filter { "action:gmake*", "toolset:gcc" }  --gmake且g++
		buildoptions {  
			"-Wall", "-Wextra", "-Werror"  
		}

	-- Using an option like --renderer=opengl  
	filter "options:renderer=opengl"  
		files { "src/opengl/**.cpp" }

	filter "files:*.png"  --设置所有的png文件
		buildaction "Embed"

	filter "system:not windows"  --非window
		defines { "NOT_WINDOWS" }
	
	filter {}  --未定义任何筛选条件，适用于所有配置
		files { "**.cpp" }
```

### 生成选项 buildSettings

#### kind

关键词`kind ("kind")`；
设置创建对象的二进制类型，是可执行文件还是库文件；
|可选类型|描述|
|:-:|:-|
|ConsoleApp|控制台或命令行程序|
|WindowedApp|运行在桌面端的应用，这个区分在Linux上没什么用，但在windows和Max OS上很重要|
|SharedLib|共享库或DLL|
|StaticLib|静态库|
|Makefile|用于调用外部命令，实际没有指定二进制文件的类型|
|Utility|仅包含自定义生成规则|
|None|适用于仅包含网页、头文件或文档的项目|
|Packaging|用于创建.androidproj文件构建apk|
|SharedItems|不指定自身的生成选项，使用链接它的任何其他项目的生成选项|

#### files,removefiles

关键词`files {"file_list"}`；
设置源代码文件，通过一个或者多个文件模式来匹配对应的文件，多余的文件使用`removefiles()`移除；
```lua
files { "hello.cpp", "goodbye.cpp" } --从当前目录添加两个文件

files { "src/*.cpp" } --添加src目录下的所有cpp文件（不包含子目录）

files { "src/**.cpp" } --添加src目录下的所有cpp文件（包含子目录）
```

#### defines

关键词`defines { "symbols" }`；
定义编译或预处理符号；
```lua
defines { "DEBUG", "TRACE" }

defines { "CALLSPEC=__dllexport" }
```

#### includedirs

关键词`includedirs { "paths" }`；
指定包含目录；
```lua
includedirs { "../lua/include", "../zlib" }

includedirs { "../includes/**" }
```

#### pchheader,pchsource

指定预编译头；
```lua
pchheader "stdafx.h"
pchsource "stdafx.cpp" --在VS中需要指定预编译源文件，其他IDE一般只需要头文件即可
```

#### links,libdirs

`links { "Reference" }`指定链接库或项目；
```lua
workspace "MyWorkspace"
   configurations { "Debug", "Release" }
   language "C++"

   project "MyExecutable"
      kind "ConsoleApp"
      files "**.cpp"
      links { "MyLibrary" }

   project "MyLibrary"
      kind "SharedLib"
      files "**.cpp"
```

```lua
workspace "MyWorkspace"  
	configurations { "Debug", "Release" }  
	language "C++"  
	  
	project "MyExecutable"  
	kind "ConsoleApp"  
	files "**.cpp"  
	links { "LibraryA:static", "LibraryB:shared" } --链接同时指定链接类型
```

`libdirs { "paths" }`指定库的搜索目录；
```lua
libdirs { "../lua/libs", "../zlib" }

libdirs { "../libs/**" }
```

#### symbols

关键词`symbols "switch"`；
启用调试信息；

```lua
project "MyProject"
    symbols "On"
```

#### optimize

关键词`optimize "value"`；
指定优化级别；
|可选项|描述|
|:-:|:-|
|Off|无任何优化|
|On|较为平衡的优化|
|Debug|支持调试的优化|
|Size|大小越小越好的优化
|Speed|速度越快越好的优化|
|Full|完全优化|

#### buildoptions,linkoptions

关键词`buildoptions { "options" }`；
添加编译参数；
```lua
filter { "system:linux", "action:gmake" }
	buildoptions { "`wx-config --cxxflags`", "-ansi", "-pedantic" }
```

关键词`linkoptions { "options" }`；
添加链接参数；
```lua
filter { "system:linux", "action:gmake" }
	linkoptions { "`wx-config --libs`" }
```

#### targetname,targetdir

设置编译目标的名字或者位置；
```lua
project "MyProject"

	targetname "my_project"

	filter { "configurations:Debug" }
	    targetdir "bin/debug"

	filter { "configurations:Release" }
	    targetdir "bin/release"
```

### 命令行参数

premake支持两种类型的命令行参数：actions和options；
```lua
-- delete a file if the clean action is running
if _ACTION == "clean" then
   -- do something
end

-- use an option value in a configuration
targetdir ( _OPTIONS["outdir"] or "out" )
```

#### 创建新的option

```lua
newoption {
   trigger = "gfxapi",
   value = "API",
   description = "Choose a particular 3D API for rendering",
   allowed = {
      { "opengl",    "OpenGL" },
      { "direct3d",  "Direct3D (Windows only)" },
      { "software",  "Software Renderer" }
   },
   default = "opengl"
}

filter { "options:gfxapi=opengl" }
   links { "opengldrv" }

filter { "options:gfxapi=direct3d" }
    links { "direct3ddrv" }

filter { "options:gfxapi=software" }
    links { "softwaredrv" }
```

#### 创建新的action

```lua
newaction {
   trigger     = "install",
   description = "Install the software",
   execute = function ()
      -- copy files, etc. here
   end
}
```


### 作用域

作用域的继承：
```lua
-- 全局作用域，所有workspace可使用该列表
defines { "GLOBAL" }

workspace "MyWorkspaces"
  -- workspace作用域继承全局作用域，当前列表为{ "GLOBAL", "WORKSPACE" }
  defines { "WORKSPACE" }

project "MyProject"
  -- project作用域继承其所在workspace，当前列表为{ "GLOBAL", "WORKSPACE", "PROJECT" }
  defines { "PROJECT" }
```

作用域的合并：
```lua
-- 声明一个workspace
workspace "MyWorkspace"
  defines { "WORKSPACE1" }

-- 声明一个或多个project
project "MyProject"
  defines { "PROJECT" }

-- 使用相同的名字再次声明workspace，其作用域会相互合并
workspace "MyWorkspace"
  defines { "WORKSPACE2" }  -- 当前列表为{ "WORKSPACE1", "WORKSPACE2" }
```

使用通配符表示作用域：
```lua
project "*" -- 表示其所处workspace作用域下的所有project

workspace "*" -- 表示全局作用域下的所有workspace
```

### 添加源文件

```lua
files {
   "hello.h",  -- 可以是完整的文件名
   "*.c",      -- 也可以使用通配符
   "**.cpp"    -- 两个通配符可以遍历子文件夹
}

files { "../src/*.cpp" } -- 指定其他文件夹

removefiles { "a_file.c", "another_file.c" }
removefiles { "tests/*.c" } -- 移除指定类型的文件
```

### 链接库文件

```lua
libdirs { "libs", "../mylibs" } -- 寻找库文件

links { "png", "zlib" } -- 链接库文件
```

### 构建配置

```lua
workspace "MyWorkspace"
    configurations { "Debug", "DebugDLL", "Release", "ReleaseDLL" }
    -- configuration可以取任意名称，使其有含义的是后续的设置

   filter "configurations:Debug"
      defines { "DEBUG" }
      flags { "Symbols" }

   filter "configurations:Release"
      defines { "NDEBUG" }
      optimize "On"
```

### 平台

```lua
configurations { "Debug", "Release" }
	platforms { "Win32", "Win64", "Xbox360" } 
	-- 同configuration，可以取任意名称，需要后续设置
	-- 不同于configuration的是，platforms不是必要的
	
	filter { "platforms:Win32" }
	    system "Windows"
	    architecture "x86"
	
	filter { "platforms:Win64" }
	    system "Windows"
	    architecture "x86_64"
	
	filter { "platforms:Xbox360" }
	    system "Xbox360"
```

```lua
workspace "MyWorkspace"
    configurations { "Debug", "Release" }
    platforms { "Windows", "PS3" }

	project "MyProject"
	    removeplatforms { "PS3" } -- 该project移除PS3平台，不影响workspace
```

```lua
workspace "MyWorkspace"
    configurations { "Debug", "Development", "Profile", "Release" }

	project "UnitTest"
	    configmap { -- configmap用以重用之前的配置
	        ["Development"] = "Debug",
	        ["Profile"] = "Release"
	    }
```

### 过滤器

```lua
workspace "MyWorkspace"
    configurations { "Debug", "Release" }

    filter "configurations:Debug"
      defines { "DEBUG" }

    filter "configurations:Release"
      defines { "NDEBUG" }
```

### 项目设置

|函数|功能|
|:-:|:-|
|kind|指定二进制文件格式（exe，lib等）|
|files,removefiles|指定源文件|
|defines|定义编译器与预处理器符号|
|includedirs|指定包含文件目录|
|pchheader,pchsource|设置预编译头|
|links,libdirs|链接库文件、框架、或其他项目|
|symbols|开启调试信息|
|optimize|优化大小或速度|
|buildoptions,linkoptions|添加任意的构建标志|
|targetname,targetdir|设置目标文件的名称或位置|

