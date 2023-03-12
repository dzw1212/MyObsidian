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

```lua
workspace "HelloWorld" --等同于 workspace("HelloWorld")

--参数可以组合
defines { "DEBUG", "TRACE" }
defines { "WINDOWS" }         -- 当前列表为 { "DEBUG", "TRACE", "WINDOWS" }

--参数也可以移除
defines { "DEBUG", "TRACE" }
removedefines { "TRACE" }     -- 当前列表为 { "DEBUG" }
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

