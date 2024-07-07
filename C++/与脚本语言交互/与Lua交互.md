# 下载Lua文件

*https://www.lua.org/download.html*

# 构建项目

## 编译Lua

### 使用自带的MakeFile（不推荐）

`Lua`文件夹中自带`MakeFile`，运行即可；

[[运行MakeFile]]

需要注意的是，该`MakeFile`不支持`cygmin`，如果无法准确识别使用的`make`的平台版本，可以手动显式修改；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240331154653.png)

最后会得到`liblua.a`（damn，vs中没法直接用）；

### 自行编译

```lua
workspace "Lua"
    configurations { "Debug", "Release" }
    architecture "x86_64"

project "Lua"
    kind "StaticLib"
    language "C"
    targetdir "bin/%{cfg.buildcfg}"
    objdir "bin-int/%{cfg.buildcfg}"

    files
    {
        "src/**.h",
        "src/**.c"
    }

    filter "configurations:Debug"
        defines { "DEBUG" }
        symbols "On"

    filter "configurations:Release"
        defines { "NDEBUG" }
        optimize "On"
```


## 项目配置

`include`Lua文件夹中`src`目录下的文件；
`link`上面编译出的Lua静态库；

```lua
workspace "InteractiveWithLua"
    configurations { "Debug", "Release" }
    architecture "x86_64"

project "InteractiveWithLua"
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
        "src",
        "lua/src",
    }

    libdirs{}

    links
    {
        "lua/bin/Debug/Lua.lib",
    }

    filter "configurations:Debug"
        defines { "DEBUG" }
        symbols "On"

    filter "configurations:Release"
        defines { "NDEBUG" }
        optimize "On"
```

# Cpp中运行Lua

## 步骤

1. **初始化Lua环境**：通过`luaL_newstate`创建一个新的Lua状态，并通过`luaL_openlibs`打开Lua标准库；

2. **加载和运行Lua脚本**：使用`luaL_dofile`加载并执行Lua脚本文件；

3. **调用Lua函数**：
	使用`lua_getglobal`获取Lua中定义的函数；
	使用`lua_pushnumber`将参数推送到Lua栈上；
	使用`lua_pcall`调用函数，指定参数和返回值的数量；
	使用`lua_tonumber`从栈上获取函数返回值；
	
4. **关闭Lua环境**：完成操作后，通过`lua_close`关闭Lua环境；

## 示例

示例lua文件`test.lua`：
```lua
local function LocalAddInLua(a, b) --local函数无法被cpp运行
    return a + b
end

function GlobalAddInLua(a, b)
    return a + b
end
```

```cpp
extern "C" {
#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"
}

#include <iostream>

int main() 
{
    lua_State* L = luaL_newstate(); // 创建一个新的Lua状态
    luaL_openlibs(L); // 打开Lua标准库

    // 加载并运行脚本
    if (luaL_dofile(L, "test.lua") != LUA_OK) {
        std::cerr << lua_tostring(L, -1) << std::endl;
        return -1;
    }

    lua_getglobal(L, "GlobalAddInLua"); // 获取函数
    lua_pushnumber(L, 10); // 推送第一个参数
    lua_pushnumber(L, 20); // 推送第二个参数

    // 调用函数，2个参数，1个返回值
    if (lua_pcall(L, 2, 1, 0) != LUA_OK) {
        std::cerr << lua_tostring(L, -1) << std::endl;
        return -1;
    }

    // 获取返回值
    if (lua_isnumber(L, -1)) {
        double result = lua_tonumber(L, -1);
        std::cout << "Result: " << result << std::endl;
    }

    lua_close(L); // 关闭Lua
    return 0;
}
```

# Lua中运行Cpp

## 步骤

 1. 编写C++函数

C++函数需要遵循Lua的C函数调用约定，即**接受一个`lua_State*`参数，并返回一个整数，表示返回值的数量**。

```cpp
extern "C" {
#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"
}

// C++函数，将被Lua调用
static int cppFunction(lua_State* L) {
    double arg1 = luaL_checknumber(L, 1); // 获取第一个参数
    double arg2 = luaL_checknumber(L, 2); // 获取第二个参数
    lua_pushnumber(L, arg1 + arg2); // 将结果推回Lua栈
    return 1; // 返回值的数量
}
```

2. 注册C++函数到Lua

在Lua脚本可以调用C++函数之前，你需要在C++代码中注册这个函数。

```cpp
// 注册函数
lua_register(L, "cppFunction", cppFunction);
```

3. 在Lua中调用C++函数

一旦函数被注册，你就可以在Lua脚本中像调用普通Lua函数一样调用它了。

```lua
result = cppFunction(10, 20)
print(result)  -- 输出: 30
```

## 示例

示例lua文件`test2.lua`：
```lua
print("call cpp function :", cppFunction(15, 55))
```

```cpp
extern "C" {
#include "lua.h"
#include "lauxlib.h"
#include "lualib.h"
}

#include <iostream>

// C++函数，将被Lua调用
static int cppFunction(lua_State* L) {
    double arg1 = luaL_checknumber(L, 1); // 获取第一个参数
    double arg2 = luaL_checknumber(L, 2); // 获取第二个参数
    lua_pushnumber(L, arg1 + arg2); // 将结果推回Lua栈
    return 1; // 返回值的数量
}

int main() 
{
    lua_State* L = luaL_newstate(); // 创建一个新的Lua状态
    luaL_openlibs(L); // 打开Lua标准库

    // 注册C++函数到Lua
    lua_register(L, "cppFunction", cppFunction);

    // 加载并执行test2.lua脚本
    if (luaL_dofile(L, "test2.lua") != LUA_OK) {
        std::cerr << lua_tostring(L, -1) << std::endl;
    }

    lua_close(L); // 关闭Lua
    return 0;
}
```


# 使用LuaJIT

[[LuaJIT]]

## 下载LuaJIT

*https://repo.or.cz/w/luajit-2.0.git*

在`luajit/src/`下有一个`msvcbuild.bat`，需要在`Visual Studio Command Prompt`环境下运行：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240331171047.png)

生成的`luajit.lib`是`x86`的（damn！）；

要想生成`x64`版本的静态库，就需要用`x64`版本的`VS Command Prompt`调用`msvcbuild.bat`；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240331174638.png)

## 构建项目

需要注意的是，链接的是`lua51.lib`和`luajit.lib`，并且将`lua51.dll`放在运行目录下；

```lua
workspace "InteractiveWithLua"
    configurations { "Debug", "Release" }
    architecture "x86_64"

project "InteractiveWithLua"
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
        "src",
        "luajit/src",
    }

    libdirs{}

    links
    {
        "luajit/src/lua51.lib",
        "luajit/src/luajit.lib",
    }

    filter "configurations:Debug"
        defines { "DEBUG" }
        symbols "On"

    filter "configurations:Release"
        defines { "NDEBUG" }
        optimize "On"
```

## 确保运行在JIT下

LuaJIT引入了一个全局变量`jit`，你可以在运行时检查这个变量来确定是否在使用LuaJIT：

```lua
if jit then
    print("Running on LuaJIT:", jit.version)
else
    print("Running on standard Lua")
end
```

# 选择性加载库

在Lua中，如果你想阉割掉不需要的库，比如`debug`库，以减小内存占用或增加安全性，你可以在初始化Lua环境时不加载这些库。Lua提供了一系列的库初始化函数，如`luaopen_base`（基础库）、`luaopen_table`（表库）、`luaopen_io`（I/O库）、`luaopen_string`（字符串库）、`luaopen_math`（数学库）等，以及`luaopen_debug`（调试库）。

默认情况下，当你调用`luaL_openlibs`函数时，Lua会加载所有标准库。如果你想自定义加载哪些库，你可以不调用`luaL_openlibs`，而是只显式调用你想加载的库的初始化函数。

```cpp
// 初始化Lua环境，但不包括debug库
void luaL_openlibs_no_debug(lua_State *L) {
    static const luaL_Reg loadedlibs[] = {
        {"_G", luaopen_base},
        {LUA_LOADLIBNAME, luaopen_package},
        {LUA_TABLIBNAME, luaopen_table},
        {LUA_IOLIBNAME, luaopen_io},
        {LUA_OSLIBNAME, luaopen_os},
        {LUA_STRLIBNAME, luaopen_string},
        {LUA_MATHLIBNAME, luaopen_math},
        {LUA_DBLIBNAME, luaopen_debug}, // 如果不需要debug库，这行可以注释掉或删除
        {NULL, NULL}
    };
    const luaL_Reg *lib;
    for (lib = loadedlibs; lib->func; lib++) {
        if (lib->name) {
            luaL_requiref(L, lib->name, lib->func, 1);
            lua_pop(L, 1);  /* remove lib */
        }
    }
}

int main(void) {
    lua_State *L = luaL_newstate();
    // 使用自定义的函数来打开需要的库
    luaL_openlibs_no_debug(L);

    // 你的代码...

    lua_close(L);
    return 0;
}
```

在这个示例中，`luaL_openlibs_no_debug`函数基于`luaL_openlibs`进行了修改，通过注释掉或删除与`luaopen_debug`相关的行，来阻止`debug`库被加载。你可以根据需要调整这个列表，只包含你的应用实际需要的库。

# 调试

## 查看堆栈

假如我有多个lua文件，每个文件中都有很多地方调用了一个cpp函数，当这个cpp函数被调用时，我想知道哪个lua在哪里调用了它？

要实现这个功能，你可以通过修改C++函数来添加日志记录，记录调用该函数的Lua脚本的信息：

```cpp
//原函数
int my_cpp_function(lua_State* L) {
    // 函数实现
    return 0;
}

//修改后的函数
int my_cpp_function(lua_State* L) {
    lua_Debug ar;
    if (lua_getstack(L, 1, &ar)) { // 获取调用栈的当前层
        lua_getinfo(L, "nSl", &ar); // 获取具体信息
        printf("Called from %s at line %d\n", ar.short_src, ar.currentline); // 打印文件名和行号
    }

    // 函数实现
    return 0;
}
```

## 设置Hook

```cpp
#include <lua.hpp> // 确保你已经正确包含了Lua的头文件

// 钩子函数
void lineHook(lua_State *L, lua_Debug *ar) {
    lua_getinfo(L, "nSl", ar);
    printf("正在执行 %s 的第 %d 行\n", ar->short_src, ar->currentline);
    // 这里可以加入更多的调试逻辑
}

int main() {
    lua_State *L = luaL_newstate();
    luaL_openlibs(L);

    // 设置钩子
    lua_sethook(L, lineHook, LUA_MASKLINE, 0);

    // 执行Lua脚本
    if (luaL_dofile(L, "test2.lua") != LUA_OK) {
        const char *error = lua_tostring(L, -1);
        printf("执行出错: %s\n", error);
    }

    lua_close(L);
    return 0;
}
```

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240331190543.png)

# 第三方库

[[lua-sol2]]

# 性能优化

[[luajit性能优化]]

