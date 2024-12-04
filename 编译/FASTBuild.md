*https://fastbuild.org/docs/home.html*

FASTBuild 是一个适用于 Windows、Linux 和 OS X 的高性能开源构建系统。它支持高度可扩展的编译、缓存和网络分发。

功能支持：
- 使用多个CPU核心并行编译；
- 内置编译缓存支持；
- 分布式编译，允许利用网络上未使用的CPU来加速编译；
- 无需安装，轻量级的可执行文件；
- 可在编译过程中收集统计信息，并在完成后生成报告；
- 可追踪并估算当前编译进度；
- 可生成`.vcxproj`和`.sln`以集成到VS中；

![255](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241024102150.png)

# 运行FASTBuild

为方便使用，推荐将`FBuild.exe`所在目录添加进环境变量；

```cmd
C:\Users\dzw>fbuild -version

FASTBuild v1.13 - Copyright 2012-2024 Franta Fulin - https://www.fastbuild.org
```

FASTBuild的编译需要当前目录下的一个名为`fbuild.ff`的配置文件（类似premake的`premake5.lua`），该文件的语法由FASTBuild定义，包含所有编译目标的配置；

比如一个配置文件如下：
```bff
.Compiler          = 'Bin\VC\cl.exe'
.Librarian         = 'Bin\VC\lib.exe'
.CompilerOptions   = '%1 /Fo%2 /c /Z7'
.LibrarianOptions  = '/NODEFAULTLIB /OUT:%2 %1'

Library( 'libA' )
{
.CompilerInputPath = 'Src\LibA\'
.CompilerOutputPath= 'Out\LibA\'
.LibrarianOutput   = 'Out\LibA\libA.lib'
}
Library( 'libB' )
{
.CompilerInputPath = 'Src\LibB\'
.CompilerOutputPath= 'Out\LibB\'
.LibrarianOutput   = 'Out\LibB\libB.lib'
}
```

然后通过`fbuild`命令来启动编译：
```cmd
C:\Users\dzw>fbuild libA
```


# 配置缓存

FastBuild会为每个编译单元（如.cpp文件）及其所有依赖（如头文件）计算一个哈希值，编译后，FastBuild将编译结果（如.obj文件）连同这个哈希值一起存储在缓存中，在之后的构建中，如果FastBuild发现某个编译单元的哈希值与缓存中的某个条目匹配，它就会直接使用缓存的结果，而不是重新编译；

缓存可大大减少以下情况的编译时间：
   - 切换分支但大部分代码没变
   - 多人协作时，获取他人的更改
   - 清理项目后重新构建

想要启用缓存功能，需要：
1. 兼容的编译设置
	一般来说缓存会自动开启，但是某些编译设置与缓存不兼容，会导致缓存无法开启：
		对于GCC/SNC/Clang：无限制；
		对于MSVC：必须使用`Z7`Debug模式，不能使用`clr`；

2. 配置缓存的位置
	缓存位置可以是本地目录，也可以是网络路径；
		`FASTBUILD_CACHE_PATH`指定；
		`ff`配置文件中通过`.CachePath`指定（覆盖环境变量）；

3. 激活缓存
	可以通过以下方式激活：
	1. 命名行参数`-cache[read|write]`；
	2. 环境变量`FASTBUILD_CACHE_MODE`的值`r/w/rw`；

# 配置分布式编译

需要Build的机器称为**主机**，能提供CPU帮助编译的机器称为**工作机**；

想要启用分布式编译，需要：
1. 兼容的编译设置
	某些编译设置与分布式编译不兼容：
		对于GCC/SNC/Clang，
			`--coverage/-ftest-coverage`无法分发；
			`.gcno`文件包含错误的路径；
			`.gcda`文件在错误的位置创建；
		对于MSVC：
			`ZW`无法使用；
			`PCH`文件无法分发，但使用`PCH`的对象可以分发；
			`/clr`无法分发；
			`/analyze`无法分发；

2. 编译器同步设置
	FASTBuild会自动将编译器配置同步到远程机器，以保证

3. 配置工作机可见性
	工作机是通过代理位置被发现的，代理位置是一个主机和工作机都能访问的网络路径，可以通过环境变量`FASTBUILD_BROKERRAGE_PATH`配置；

	工作机向该位置写入一个token表明其可用性，主机通过检查该代理位置来发现工作机；

4. 启动工作机进程
	当配置好代理位置后，在工作机上启动`FBuildWorker.exe`；

5. 激活分布式编译
	命令行中添加参数`-dist`；

## 步骤

工作机上：
- Step1、创建一个文件夹，如`D:\SharedFolder`，将该文件夹设为共享；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241102151443.png)

- Step2、工作机上设置环境变量`FASTBUILD_BROKERAGE_PATH`，值为刚才创建的共享文件夹；
- Step3、启动`FBuildWorker.exe`，在命令行中可以设置CPU的使用量；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241102151843.png)

主机上：
- Step1、添加`FASTBUILD_BROKERAGE_PATH`环境变量，值为工作机提供的共享文件夹，如`\\DESKTOP-XXX\SharedFolder`；
- 在开始`fbuild`时，添加参数`-dist`；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241102152104.png)

---

PS：如果不在一个局域网内，可以使用类似`Tailscale`等软件虚拟组网；
# bff配置文件格式

## 预定义符号

- `__LINUX__`：在Linux上运行时定义；
- `__OSX__`：在OS X上运行时定义；
- `__WINDOWS__`：在Windows上运行时定义；

## 内建变量

- `_CURRENT_BFF_DIR_`：当前解析的bff文件所在的目录；
- `_FASTBUILD_VERSION_STRING_`：字符串格式的FASTBuild的版本，如`v0.97`；
- `_FASTBUILD_VERSION_`：整数格式的FASTBuild的版本，如`97`；
- `FASTBUILD_EXE_PATH_`：运行`FBuild.exe`的目录；
- `_WORKING_DIR_`：启动`FBuild`时的目录；

## 内置函数

- `Alias`：为一个或多个目标创建别名；
- `Compiler`：为分布式编译定义一组编译器文件；
- `Copy`：复制一个或多个文件；
```bff
Copy( 'CopyFile' )
{
  .Source   = { 'library1/library1.dll'
                'library2/library2.dll' }
  .Dest     = 'out/bin/'
}
```

- `CopyDir`：复制一个或多个目录；
```bff
CopyDir( 'CopyExample' )
{
  .SourcePaths = 'folder\'
  .Dest        = 'out\'
}
```

- `CSAssembly`：创建C#程序集；
- `DLL`：创建C/C++ DLL；
- `Error`：生成用户自定义报错；
```bff
.RequiredFASTBuildVersion = 97
If( ._FASTBUILD_VERSION_ < .RequiredFASTBuildVersion )
{
    Error( 'FASTBuild v0.97 or later is required (found $_FASTBUILD_VERSION_STRING_$)' );
}
```

- `Exec`：作为构建步骤，运行自定义可执行文件；
- `Executable`：构建C/C++可执行文件；
- `ForEach`：循环遍历；
```bff
ForEach( .A in .ArrayA,  ; One or more arrays to loop through
         .B in .ArrayB,
         ... )
{
  ; Content to repeat
}
```

- `If`：控制流；
```bff
If( !.MyBoolean || .AnotherBoolean )
{
    // Evaluated if at least one of .MyBoolean and .AnotherBoolean is true
}
```


- `Library`：构建C/C++ 静态库；
- `ListDependencies`：导出txt格式的目标依赖项说明文件；
```bff
ObjectList( 'SimpleObject' )
{
    .CompilerInputFiles = "C:/Test/SimpleObject.cpp"
    .CompilerOutputPath = "C:/Test/Output"
}

ListDependencies( 'SimpleDependencies' )
{
    .Source = 'SimpleObject'
    .SourcePattern = { '*.h', '*.c', '*.cpp' }
    .Dest = 'C:/Test/Output/Dependencies.txt'
}
```
生成的txt文件如下：
```txt
C:/Test/HeaderA.h
C:/Test/HeaderB.h
C:/Test/SimpleObject.cpp
```

- `ObjectList`：创建一组对象；
- `Print`：打印；
- `RemoveDir`：删除目录；
- `Settings`：配置全局设置；
- `Test`：构建并运行单元测试；
- `TextFile`：生成文本文件；
- `Unity`：生成`Unity/Blob`文件，以加速编译；
- `Using`：将结构体成员提升至当前作用域；
- `VCXProject`：生成与FASTBuild集成的VS项目；
- `VSProjectExternal`：引用外部VS项目，以便与FASTBuild接集成；
- `VSSolution`：生成与FASTBuild集成的VS解决方案；
- `XCodeProject`：生成与FASTBuild集成的XCode项目；


## 基础

空白（空格、制表符、回车、换行）在解析过程中会被忽略，没有语法意义；

注释：
```bff
// 注释1
； 注释2
```

字符串：
```bff
"包含一个`子字符串`"
`包含一个"子字符串"`

特殊字符" ` $等，可以使用^进行转义
```

### 指令

`#define / #undef`允许用户定义/取消定义自定义标记；
```bff
#define USE_CLANG_COMPILER
#if __WINDOWS__
    #undef USE_CLANG_COMPILER
#endif
#if USE_CLANG_COMPILER
    ...
```

`#if / #else / #endif`用于控制流；
```bff
#if USE_MSVC
    .CompilerOptions = '/c "%1" /Fo"%2"' 
#else
    .CompilerOptions = '-c "%1" -o "%2"' 
#endif

//支持逻辑运算符 && || !
```

`exists`用于确定环境变量是否存在；
```bff
#if exists(MY_ENV_VAR)
    #import MY_ENV_VAR
#endif
```

`file_exists`用于监测文件是否存在；
```bff
#if file_exists("file.dat")
    //
#endif
```

`#import`允许bff文件导入环境变量，与已有环境变量同名的变量会在`#import`范围内生效，类似局部变量一样；
```bff
#import VS120COMNTOOLS
.VisualStudio2012Compiler = '$VS120COMNTOOLS$\..\..\VC\bin\cl.exe'
```

`#include`指令允许bff文件包含另一个配置文件，方便配置文件按平台或逻辑分割；
```bff
#include "platforms/x86.bff"
#include "platforms/x64.bff"

#include "libs/core.bff"
#include "libs/graphics.bff"
#include "libs/sound.bff"

#include "game.bff"
```

`#once`类似cpp头文件的`#pragma once`，确保bff文件无论被包含多少次都只能解析一次；
### 变量

支持四种变量类型：字符串、整数、布尔值、数组；
变量声明以`.`开始，命名区分大小写；
```bff
.MyString = "hello"
.MyBool   = true
.MyInt    = 7
.MyArray  = { "aaa", "bbb", "ccc" }

.StringA = "hello"
.StringB = "$StringA$ FASTBuild user!"
```

变量的修改：
```bff
; 声明
.MyString = "hello"

; 加法
.MyString + " there"
          + " FASTBuild user" ; 自动修改为最后的引用

; 减法
.MyString - " user" 
          - "hello "
		  
; 重新赋值
.MyString = "hello"
```

变量可以在解析时动态构造：
```bff
.Config   = 'Debug'
.Platform = 'X64'
.Options  = ."CompilerOptions_$Config$_$Platform$" // 当前组合为.CompilerOptions_Debug_X64
```

### 作用域

变量在其声明的作用域的所有子作用域内有效，在作用域内对变量的修改只在当前作用域内生效；
```bff
.MyString = 'hello'
{
  .MyString = 'goodbye'
}
; Mystring 重新包含 'hello'

{
  .MyString + ' hello'  ; 此时为 'hello hello'
}
; Mystring 重新包含 'hello'
```

可以使用`^`前缀将子作用域内的变量推送到其父作用域，以覆盖父作用域内的值；

### 结构体

结构体中包含任意数量、任意类型的变量，比如其他结构体的数组；
```bff
.MyStruct =
[
  .MyString = "string"
  .MyInt    = 7
]
```

结构体中成员与当前命名空间隔离，可以使用`Using`将结构体的所有成员包含进当前作用域；
```bff
Using(.MyStruct)
```

还可以使用`Using`继承和覆盖其他结构体；
```bff
.StructA = [ .StringA = 'a' ]
.StructB =
[
  Using( .StructA ) // 继承：StructB 有了 StringA 成员
  .StringB = 'b'
]
```

结构体支持使用`+`进行连接，该运算会递归应用于结构体的每个成员；
```bff
.StructA =
[
  .ArrayOfStrings     = { 'String1', 'String2' }
  .String             = 'String'
]
.StructB =
[
  .ArrayOfStrings     = { 'String3' }
  .String             = '!'
]
.StructC = .StructA
         + .StructB
// StructC 现在包含 .ArrayOfStrings = { 'String1', 'String2', 'String3' } 和 .String = 'String!'
```

遍历结构体数组：
```bff
.ConfigX86 =
[ 
  .Compiler   = "compilers/x86/cl.exe"
  .ConfigName = "x86"
]
.ConfigX64 =
[
  .Compiler = "compilers/x64/cl.exe"
  .ConfigName = "x64"
]
.Configs = { .ConfigX86, .ConfigX64 }

ForEach( .Config in .Configs )
{
  Using( .Config )
  Library( "Util-$ConfigName$" )
  {
    .CompilerInputPath  = 'libs/util/'
    .CompilerOutputPath = 'out/$ConfigName$/'
    .LibrarianOutput 	= 'out/$ConfigName$/Util.lib'
  }
}
```

### 函数

bff中提供了一些内置函数，比如`Library('Name')`创建静态库，`Executable('Name')`创建可执行文件等；

函数参数的数量和语法因不同函数而异，如果参数不是必要的，则应该省略括号；
```bff
Settings
{
    // 全局设置
}
```

函数的成员允许在Build时替换某些配置值，而不是在解析时；
```bff
Library( "mylib" )
{
  .CompilerInputPath = 'Code\'  ;编译Code\下的所有cpp文件
  .CompilerOutputPath= 'Tmp\'
  .CompilerOptions   = '%1 /Fo%2 /nologo /c' ; 预留替换位置
}
```
假如`Code\`目录下有三个文件`a.cpp, b.cpp, c.cpp`，编译器将被调用三次，在每次调用时，`%1`和`%2`都会被替换为适当的值，在本例中，分别为`a.cpp`和`Tmp/a.obj`、`b.cpp`和`Tmp/b.obj`、`c.cpp`和`Tmp/c.obj`；

用户自定义函数规则：
```bff
function FunctionA()
{
  Print( 'Hello' )
}

function FunctionB( .Arg .Arg2 )
{
  If( .Arg1 == .Arg2 )
  {
    Print( 'Equal' )
  }
}

//调用上面的用户函数
FunctionA()
FunctionB( 'ALiteralString' .APreviouslyDeclaredString )
```

## 内置占位符

这些占位符用于动态替换编译器命令行中的特定参数，在编译过程中会被 FastBuild 替换为实际的文件路径或其他相关信息；

### 用于CompilerOptions

- `%1`：每次调用编译器时的输入文件，由`CompilerInputPath`等选项指定；
- `%2`：编译对象的输出名称，由`CompilerOutputPath`等选项指定；
- `%3`：PCH的obj文件，用于链接（仅限MSVC）；
- `%4`：用于`CompilerForceUsing`配合`/FU %4`拓展项目；

### 用于PCHOptions

- `%1`：用于生成PCH的输入文件，由`PCHInputFile`选项指定；
- `%2`：生成PCH文件的输出名称，由`PCHOutputFie`选项指定；
- `%3`：PCH的obj文件，用于链接（仅限MSVC）；
- `%4`：用于`CompilerForceUsing`配合`/FU %4`拓展项目；

### 用于LinkerOptions

- `%1`：链接的输入文件，由`Libraries`和`Libiaries2`指定；
- `%1[0]`：仅由`Libraries`指定；
- `%1[1]`：仅由`Libraries2`指定；
- `%2`：输出目录，由`LinkerOutput`指定；
- `%3`：由`LinkerAssemblyResources`，配合`/ASSEMBLYRESOURCE"%3"`使用（仅限MSVC）；

### 用于LibrarianOptions

- `%1`：需要封装的obj文件；
- `%2`：输出的lib文件，由`LibrarianOutput`指定；

### 用于ExecArguments

- `%1`：输入文件，由`ExecInput`指定；
- `%2`：输出文件，由`ExecOutput`指定；

# 引用外部VS项目

[[VS项目文件]]

通过内置函数`VSProjectExternal`引用外部VS项目，以便于FASTBuild集成；

```bff
VSProjectExternal( alias )               // (optional) Alias
{
  // 输入选项
  .ExternalProjectPath
  .ProjectGuid
  .ProjectTypeGuid
  
  // 编译配置选项
  .ProjectConfigs
}

// ProjectConfigs - structs in the following format
//-------------------------------------------------
.ProjectConfig =
[
  .Platform
  .Config
]
```

## VSProjTypeExtractor

*https://github.com/lucianm/VSProjTypeExtractor*

VS项目类型GUID提取器（仅限Windows），可以配合`VSProjectExternal`一起使用，从外部VS项目中提取`ProjectTypeGuid`；

由两个DLL组成，分别是`VSProjTypeExtractorManaged.dll`和`VSProjTypeExtractor.dll`，使用这两个DLL需要`.NET Framework v4.8`，将这两个DLL与`FBuild.exe`放在同一目录下即可；

==如果使用该工具，则`VSProjectExternal`的所有选项中，除了`ExternalProjectPath`之外的其他选项都可以忽略，直接使用解析出的配置；==

## 输入选项

- `ExternalProjectPath`：必填，字符串类型，`vcxproj`文件所在的位置，比如：
```bff
.ExternalProjectPath = 'Setup\Source\ExtProject\ExtProject.wixproj'
```

- `ProjectGuid`：如果使用**VSProjTypeExtractor**则可以忽略，字符串类型，项目通用唯一标识符；
```bff
.ProjectGuid = '{FA3D597E-38E6-4AE6-ACA1-22D2AF16F6A2}'
```

- `ProjectTypeGuid`：如果使用**VSProjTypeExtractor**则可以忽略，字符串类型，项目类型通用唯一标识符；
```bff
.ProjectGuid = '{FA3D597E-38E6-4AE6-ACA1-22D2AF16F6A2}'
```

## 编译配置选项

- `ProjectConfigs`：可选，项目配置结构体数组，项目配置结构体如下：
```bff
.ProjectConfig =
[
  // Basic Options
  .Platform  // 平台 Win32, X64, PS3 等
  .Config    // 配置 Debug, Release 等
]
```

比如以下配置：
```bff
.DebugConfig      = [ .Platform = 'Win32' .Config = 'Debug' ]
.ReleaseConfig    = [ .Platform = 'Win32' .Config = 'Release' ]
.ProjectConfigs   = { .DebugConfig, .ReleaseConfig }
```

如果该参数不填，则使用以下默认值：
```bff
.X86DebugConfig   = [ .Platform = 'Win32' .Config = 'Debug' ]
.X86ReleaseConfig = [ .Platform = 'Win32' .Config = 'Release' ]
.X64DebugConfig   = [ .Platform = 'X64'   .Config = 'Debug' ]
.X64ReleaseConfig = [ .Platform = 'X64'   .Config = 'Release' ]
.ProjectConfigs   = {
                        .X86DebugConfig, .X86ReleaseConfig, .X64DebugConfig, .X64ReleaseConfig 
                    }
```

# 自定义编译器

通过内置函数`Compiler`来定义编译器，并指定如何与编译器进程交互，以实现依赖关系追踪、分发、缓存等功能；

指定编译器可以简单使用一句`.Compiler = XXX`，也可以使用自定义的编译器，这是因为分布式编译时工作机的平台可能和主机不同，需要同步编译器与相关dll到工作机上执行编译以确保编译器一致；而链接过程耗时少，在主机上执行，因此没有必要自定义链接器；

```bff
Compiler( 'name' )              // 别名
{
  .Executable                   //（必须）主要编译器可执行文件路径
  .ExtraFiles                   // (可选) 额外的文件，通常是一些dll

  // Extra Options
  .CompilerFamily               // (optional) Explicitly specify compiler type (default: auto)
  .AllowDistribution            // (optional) Allow distributed compilation (if available) (default: true)
  .ExecutableRootPath           // (optional) Override default path for executable distribution
  .SimpleDistributionMode       // (optional) Allow distribution of otherwise unsupported "compilers" (default: false)
  .CustomEnvironmentVariables   // (optional) Environment variables to set on remote host
  .ClangRewriteIncludes         // (optional) Use Clang's -frewrite-includes option when preprocessing (default: true)
  .ClangGCCUpdateXLanguageArg   // (optional) Update -x language arg for second pass of compilation (default: false)
  .VS2012EnumBugFix             // (optional) Enable work-around for bug in VS2012 compiler (default: false)
  .Environment                  // (optional) Environment variables to use for local build
  .AllowResponseFile            // (optional) Allow response files to be used if not auto-detected (default: false)
  .ForceResponseFile            // (optional) Force use of response files (default: false)
  
  // Temporary Options
  .UseLightCache_Experimental   // (optional) Enable experimental "light" caching mode (default: false)
  .UseRelativePaths_Experimental// (optional) Enable experimental relative path use (default: false)
  .SourceMapping_Experimental   // (optional) Use Clang's -fdebug-source-map option to remap source files
  .ClangFixupUnity_Disable      // (optional) Disable preprocessor fixup for Unity files (default: false)
}
```

## 基础选项

- `Executable`：FASTBuild 将调用的主要编译器可执行文件的路径；
- `ExtraFiles`：（可选）额外文件，分布式编译时这些文件也会同步到远程机器；

比如：
```bff
Compiler( 'Test' )
{
  .Executable	= 'C:\compiler\compiler.exe'       // dest: compiler.exe
  .ExtraFiles = { 'C:\compiler\subdir\helper.dll'
                  'C:\cruntime\mvscrt.dll' 
}
```

对于[[编译器#MSVC]]，通常需要包含以下文件：
1. `c1.dll`：C编译器前端；
2. `c1xx.dll`：C++编译器前端；
3. `c2.dll`：后端代码生成器；

一个MSVC编译器的示例配置：
```bff
Compiler('MSVC')
{
    .Executable = 'C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64\cl.exe'
    .ExtraFiles = {
        'C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64\c1.dll'
        'C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64\c1xx.dll'
        'C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.35.32215\bin\Hostx64\x64\c2.dll'
    }
}
```

相关文件的路径可以通过[[编译器#查找MSVC编译器目录]]查找；
## 额外选项

- `CompilerFamily`：（可选，字符串，默认auto）一般FASTBuild会根据可执行文件的名称来推测编译器的类型，但是也支持显式执行：
	`msvc`, `clang`, `clang-cl`, `gcc`, `snc`, `codewarrior-wii`, `greenhills-wiiu`, `cude-nvcc`, `qt-rcc`, `vbcc`, `orbis-wave-psslc`, `csharp`

- `AllowDistribution`：（可选，布尔值，默认true）编译器是否支持分布式；

- `ExecutableRootPath`：（可选，字符串）覆盖`.ExtraFiles`的地址，方便更灵活地在远程主机上复制文件层级结构；

- 后略；

## 示例配置

以下为VS2019的示例配置：
```bff
// VisualStudio 2019 x64 Compiler
.VS2019_BasePath         = 'C:\Program Files (x86)\Microsoft Visual Studio\2019\2019\Community'
.VS2019_Version          = '14.28.29333'
Compiler( 'Compiler-VS2019-x64' )
{
    .Root       = '$VS2019_BasePath$/VC/Tools/MSVC/$VS2019_Version$/bin/Hostx64/x64'
    .Executable = '$Root$/cl.exe'
    .ExtraFiles = { '$Root$/c1.dll'
                    '$Root$/c1xx.dll',
                    '$Root$/c2.dll',
                    '$Root$/atlprov.dll', // Only needed if using ATL
                    '$Root$/msobj140.dll'
                    '$Root$/mspdb140.dll'
                    '$Root$/mspdbcore.dll'
                    '$Root$/mspdbsrv.exe'
                    '$Root$/mspft140.dll'
                    '$Root$/msvcp140.dll'
                    '$Root$/msvcp140_atomic_wait.dll' // Required circa 16.8.3 (14.28.29333)
                    '$Root$/tbbmalloc.dll' // Required as of 16.2 (14.22.27905)
                    '$Root$/vcruntime140.dll'
                    '$Root$/vcruntime140_1.dll' // Required as of 16.5.1 (14.25.28610)
                    '$Root$/1033/clui.dll'
                    '$Root$/1033/mspft140ui.dll' // Localized messages for static analysis
                  }
}
```

更多的示例配置可以在*https://fastbuild.org/docs/functions/compiler.html* 中查找；

# 创建编译对象列表

使用`ObjectList`指定编译对象列表，通常用于稍后链接为静态/动态库或可执行文件；

```bff
ObjectList( alias )
{
	//编译相关选项
	.Compiler
	.CompilerOptions
	.CompilerOutputPath //存放中间文件的目录
	.CompilerOutputExtension //（可选）指定生成的文件的拓展名，如.obj或.o
	.CompilerOutputKeepBaseExtension //（可选）是否追加拓展名
	.CompilerOutputPrefix //（可选）执行前缀名

	//输入文件（均为可选）
	.CompilerInputPath //查找输入文件的目录，默认当前
	.CompilerInputPattern //根据样式匹配输入文件，默认*.cpp
	.CompilerInputPathRecurse //是否递归文件夹，默认true
	.CompilerInputExcludePath //要排除的目录
	.CompilerInputExcludedFiles //要排除的文件
	.CompilerInputFiles //显式指定所有要编译的文件
	.CompilerInputFilesRoot //对应指定文件所在的目录
	.CompilerInputUnity
	.CompilerInputAllowNoFiles //无输入文件时不报错
	.CompilerInputObjectLists //使用其他ObjectList的输入文件作为输入
	.CompilerForceUsing //配合编译器参数/FU使用

	//缓存与分布式（均为可选）
	.AllowCaching //是否允许缓存，默认true
	.AllowDistribution //是否允许分布式编译，默认true

	//预处理（均为可选）
	.Preprocessor //指定预处理器，如果需要的话
	.PreprocessorOptions //预处理器参数

	//预编译头文件（均为可选）
	.PCHInputFile //指定预编译头文件
	.PCHOutputFile //预编译头文件编译后输出路径
	.PCHOptions //编译PCH的参数

	//其他（均为可选）
	.PreBuildDependencies //使Target先于ObjectList构建
	.Hidden //从-showatrgets中隐藏target
}
```

# 生成文件

## Obj文件列表

```bff
ObjectList( alias )         ; Alias
{
  ; options for compilation
  .Compiler                 ; Compiler to use
  .CompilerOptions          ; Options for compiler
  .CompilerOutputPath       ; Path to store intermediate objects
  .CompilerOutputExtension  ; (optional) Specify the file extension for generated objects (default .obj or .o)
  .CompilerOutputKeepBaseExtension ; (optional) Append extension instead of replacing it (default: false)
  .CompilerOutputPrefix     ; (optional) Specify a prefix for generated objects (default none)

  ; Specify inputs for compilation
  .CompilerInputPath           ; (optional) Path to find files in
  .CompilerInputPattern        ; (optional) Pattern(s) to use when finding files (default *.cpp)
  .CompilerInputPathRecurse    ; (optional) Recurse into dirs when finding files (default true)
  .CompilerInputExcludePath    ; (optional) Path(s) to exclude from compilation
  .CompilerInputExcludedFiles  ; (optional) File(s) to exclude from compilation (partial, root-relative of full path)
  .CompilerInputExcludePattern ; (optional) Pattern(s) to exclude from compilation
  .CompilerInputFiles          ; (optional) Explicit array of files to build
  .CompilerInputFilesRoot      ; (optional) Root path to use for .obj path generation for explicitly listed files
  .CompilerInputUnity          ; (optional) Unity to build (or Unities)
  .CompilerInputAllowNoFiles   ; (optional) Don't fail if no inputs are found
  .CompilerInputObjectLists    ; (optional) ObjectList(s) whos output should be used as an input
  
  ; Cache & Distributed compilation control
  .AllowCaching             ; (optional) Allow caching of compiled objects if available (default true)
  .AllowDistribution        ; (optional) Allow distributed compilation if available (default true)

  ; Custom preprocessor support
  .Preprocessor             ; (optional) Compiler to use for preprocessing
  .PreprocessorOptions      ; (optional) Args to pass to compiler if using custom preprocessor
  
  ; Additional compiler options
  .CompilerForceUsing       ; (optional) List of objects to be used with /FU

  ; (optional) Properties to control precompiled header use
  .PCHInputFile             ; (optional) Precompiled header (.cpp) file to compile
  .PCHOutputFile            ; (optional) Precompiled header compilation output
  .PCHOptions               ; (optional) Options for compiler for precompiled header

  ; Additional options
  .PreBuildDependencies     ; (optional) Force targets to be built before this ObjectList (Rarely needed,
                            ; but useful when a ObjectList relies on generated code).
  .Hidden                   ; (optional) Hide a target from -showtargets (default false)
}
```

**PS：在生成exe，lib或dll之前，需要先使用`ObjectList`将源文件编译成obj；**
## 可执行文件

```bff
Executable( alias )        ; (optional) Alias
{
  .Linker //指定链接器
  .LinkerOutput //链接后的输出目录
  .LinkerOptions //链接参数
  .Libraries //要链接的文件，比如obj文件
  .Libraries2              ; (optional) Secondary libraries to link into executable
  .LinkerLinkObjects       ; (optional) Link objects used to make libs instead of libs (default false)
  .LinkerAssemblyResources ; (optional) List of assembly resources to use with %3
  
  .LinkerStampExe          ; (optional) Executable to run post-link to "stamp" executable in-place
  .LinkerStampExeArgs      ; (optional) Arguments to pass to LinkerStampExe 
  .LinkerType              ; (optional) Specify the linker type. Valid options include: 
                           ; auto, msvc, gcc, snc-ps3, clang-orbis, greenhills-exlr, codewarrior-ld
                           ; Default is 'auto' (use the linker executable name to detect)
  .LinkerAllowResponseFile ; (optional) Allow response files to be used if not auto-detected (default: false)
  .LinkerForceResponseFile ; (optional) Force use of response files (default: false)

  ; Additional options
  .PreBuildDependencies    ; (optional) Force targets to be built before this Executable (Rarely needed,
                           ; but useful when Executable relies on externally generated files).                           

  .Environment             ; (optional) Environment variables to use for local build
                           ; If set, linker uses this environment
                           ; If not set, linker uses .Environment from your Settings node
}
```


## 静态库

```bff
Library( alias )            ; (optional) Alias
{
  ; options for compilation
  .Compiler                 ; Compiler to use
  .CompilerOptions          ; Options for compiler
  .CompilerOutputPath       ; Path to store intermediate objects
  .CompilerOutputExtension  ; (optional) Specify the file extension for generated objects (default .obj or .o)
  .CompilerOutputPrefix     ; (optional) Specify a prefix for generated objects (default none)

  ; Options for librarian
  .Librarian                ; Librarian to collect intermediate objects
  .LibrarianOptions         ; Options for librarian
  .LibrarianType            ; (optional) Specify the librarian type. Valid options include:
                            ; auto, msvc, ar, ar-orbis, greenhills-ax
                            ; Default is 'auto' (use the librarian executable name to detect)
  .LibrarianOutput          ; Output path for lib file
  .LibrarianAdditionalInputs; (optional) Additional inputs to merge into library
  .LibrarianAllowResponseFile ; (optional) Allow response files to be used if not auto-detected (default: false)  
  .LibrarianForceResponseFile ; (optional) Force use of response files (default: false)

  ; Specify inputs for compilation
  .CompilerInputPath           ; (optional) Path to find files in
  .CompilerInputPattern        ; (optional) Pattern(s) to use when finding files (default *.cpp)
  .CompilerInputPathRecurse    ; (optional) Recurse into dirs when finding files (default true)
  .CompilerInputExcludePath    ; (optional) Path(s) to exclude from compilation
  .CompilerInputExcludedFiles  ; (optional) File(s) to exclude from compilation (partial, root-relative of full path)
  .CompilerInputExcludePattern ; (optional) Pattern(s) to exclude from compilation
  .CompilerInputFiles          ; (optional) Explicit array of files to build
  .CompilerInputFilesRoot      ; (optional) Root path to use for .obj path generation for explicitly listed files
  .CompilerInputUnity          ; (optional) Unity to build (or Unities)
  .CompilerInputAllowNoFiles   ; (optional) Don't fail if no inputs are found
  .CompilerInputObjectLists    ; (optional) ObjectList(s) whos output should be used as an input

  ; Cache & Distributed compilation control
  .AllowCaching             ; (optional) Allow caching of compiled objects if available (default true)
  .AllowDistribution        ; (optional) Allow distributed compilation if available (default true)
  
  ; Custom preprocessor support
  .Preprocessor             ; (optional) Compiler to use for preprocessing
  .PreprocessorOptions      ; (optional) Args to pass to compiler if using custom preprocessor
  
  ; Additional compiler options
  .CompilerForceUsing       ; (optional) List of objects to be used with /FU

  ; (optional) Properties to control precompiled header use
  .PCHInputFile             ; (optional) Precompiled header (.cpp) file to compile
  .PCHOutputFile            ; (optional) Precompiled header compilation output
  .PCHOptions               ; (optional) Options for compiler for precompiled header

  ; Additional options
  .PreBuildDependencies     ; (optional) Force targets to be built before this library (Rarely needed,
                            ; but useful when a library relies on generated code).

  .Environment              ; (optional) Environment variables to use for local build
                            ; If set, librarian uses this environment
                            ; If not set, librarian uses .Environment from your Settings node
  .Hidden                   ; (optional) Hide a target from -showtargets (default false)
}
```

## 动态库

```bff
DLL( alias )               ; (optional) Alias
{
  .Linker                  ; Linker executable to use
  .LinkerOutput            ; Output from linker
  .LinkerOptions           ; Options to pass to linker
  .Libraries               ; Libraries to link into DLL
  .Libraries2              ; (optional) Secondary libraries to link into executable
  .LinkerLinkObjects       ; (optional) Link objects used to make libs instead of libs (default true)
  .LinkerAssemblyResources ; (optional) List of assembly resources to use with %3
  
  .LinkerStampExe          ; (optional) Executable to run post-link to "stamp" executable in-place
  .LinkerStampExeArgs      ; (optional) Arguments to pass to LinkerStampExe
  .LinkerType              ; (optional) Specify the linker type. Valid options include: 
                           ; auto, msvc, gcc, snc-ps3, clang-orbis, greenhills-exlr, codewarrior-ld
                           ; Default is 'auto' (use the linker executable name to detect)
  .LinkerAllowResponseFile ; (optional) Allow response files to be used if not auto-detected (default: false)
  .LinkerForceResponseFile ; (optional) Force use of response files (default: false)

  ; Additional options
  .PreBuildDependencies    ; (optional) Force targets to be built before this DLL (Rarely needed,
                           ; but useful when DLL relies on externally generated files).

  .Environment             ; (optional) Environment variables to use for local build
                           ; If set, linker uses this environment
                           ; If not set, linker uses .Environment from your Settings node
}
```
# Target别名

可用`Alias`为一个或多个target创建别名，通常用于简化配置，别名之间也可以嵌套；

格式：
```bff
Alias('name')
{
	.Targets //必需，需要配置别名的目标
	.Hidden  //可选，默认false，配置为true后-showtargtes不显示
}
```

比如可以用一个别名表示多个配置：
```bff
Alias('main')
{
	.Targets = {'main-x86', 'main-x64'}
}
```

或者用一个别名来简化配置：
```bff
Alias('Editor')
{
	.Targets = 'bin/Editor/X64/Release/Editor.exe'
}
```

# 命令行参数

## fbuild

- `-cache/-cacheread/-cachewrite`：启用build缓存，分别对应可读可写/只读/只写；
	- `-cacheinfo`：查看缓存详情；
	- `-cachecompressionlevel [level]`：控制缓存压缩等级，默认1，在网络带宽有限时提高压缩等级有助于减少网络传输压力，但响应地会增加压缩时间；
		- -128~-1：使用`LZ4`少量压缩；
		- 0：不压缩；
		- 1~12：使用`Zstd`大量压缩；
	- `-cachetrim [mb]`：限制缓存的大小；
	- `-cacheverbose`：提供有关缓存的额外信息，有助于debug；
- `-clean`：相当于rebuild；
- `-compdb`：不进行build，而是生成一个名为`compile_commands.json`的文件，其中包含输入文件、输出目录、所用编译选项等信息；
- `-config <path>`：使用指定的bff文件（默认会使用当前目录下名为`fbuild.bdd`的文件）；
- `-continueafterdbmove`：fastbuild在build时会在当前目录下生成一个数据库文件`fbuild.windows.fdb`，一般情况下build过程中该文件不能移动，该参数允许检测到数据库文件移动后忽略报错并替换数据库文件继续build；
- `-dbfile <path>`：显式指定数据库文件的生成目录；
- `-debug`：（仅限windows）启动build时弹出一个消息框，允许连接一个调试器；
- `-dist`：启动分布式编译；
	- `-distcompressionlevel [level]`：控制分发压缩等级，默认-1，压缩选项同`cachecompressionlevel`；
	- `-distverbose`：提供有关分布式编译的详细信息；
- `-dot [target]`：为已知依赖关系生成`fbuild.gv`文件，可在第三方工具（如`Graphviz`）中可视化；
- `-fixuperrorpaths`：将`GCC/SNC/Clang`的警告、错误等格式化为VS的格式，从而支持在VS中双击查看，使用`-vs`选项时默认启用；
- `-forceremote`：禁止使用本地CPU进行分布式编译，禁止使用缓存；一般用来排查故障；
- `-help`：查看帮助信息；
- `-ide`：包含一些常在IDE中的选项，相当于`-nopreogress`，`-fixuperrorpaths`，`-wrapper`；
- `-j[x]`：一般来说fastbuild会通过检测主机上的硬件内核数量来确定最佳本地线程数量，该选项允许自定义本地并行任务数（对分布式编译的工作机没有影响）；
- `-monitor`：输出文件`%TEMP%/FastBuild/FastBuildLog.log`供第三方工具进行可视化，该文件会在build过程中持续更新；
- `-nofastcancel`：通常情况下当一个任务失败后会终止其他线程上的任务，该选项允许其他任务继续进行；
- `-nolocalrace`：禁止远程启动作业时的本地竞争，一般用于调试；
- `-noprogress`：不显示进度条；
- `-nostoponerror`：通常情况下当遇到编译错误时会立即停止，该选项允许尽可能多地编译后再停止，该选项包含了`-nofastcancel`；
- `-nosummaryonerror`：仅在build成功后打印`-summary`信息；
- `-nounity`：指定Unity中所有文件像在Unity之外单独指定一样build；
- `-profile`：生成Chrome Tracing格式的文件`fbuild_profile.json`，可用于[[chrome-tracing性能分析]]；
- `-progress`：强制显示进度条；
- `-quiet`：不显示编译输出；
- `-report=[html|json]`：在build结束后生成详细报告，默认html格式；
- `-showcmdoutput`：显示外部进程的全部输出；
- `-showcmds`：显示调用外部工具时传递给它们的完整命令行；
- `-showdeps`：显示指定target的依赖关系，有助于debug；
- `-showtargets/showalltargets`：显示所有target，不包含/包含设为hide的target；
- `-summary`：build结束后显示摘要；
- `-verbose`：显示用于debug的详细信息；
- `-version`：查看当前fastbuild的版本；
- `-wait`：限制在已运行的build结束后再排队进行build；
- `-why`：显示正在build的每个项目的原因；
- `-wrapper`：（仅限windows）通过一个中间子进程启动fbuild，以便从VS中终止build；

# MSVC编译选项

[[msvc编译选项]]

FASTBuild中通过`CompilerOptions`来指定编译参数；
比如：
```bff
.CompilerOptions    = '"%1"'           // Input
                    + ' /Fo"%2"'       // Output
                    + ' /c'
                    + ' /nologo'
                    + ' /W4'
                    + ' /WX'
                    + ' /EHsc'
                    + ' /ZI'
                    + ' /MDd'
```

# RC资源文件处理

VS中经常有一些资源文件`.rc`，需要使用`rc.exe`将其编译为`.res`文件，然后同`obj`文件一起参与链接过程；

FASTBuild中可以定义一个`Exec`过程来处理这件事：
```bff
Exec ( 'Resource' )
{
	.ExecExecutable = 'xxx/rc.exe'
	.ExecInput = 'xxx.rc'
	.ExecOutput = 'xxx.res'
	.ExecArguments  = ' /i "xxx"'
					+ ' /i "xxx"'
					+ ' /fo "%2"'
					+ ' "%1"'
	.ExecAlwaysShowOutput = true //方便调试
}
```

```bff
Executable ( 'exe' )
{
	...
	.Libraries = { 'obj', 'Resource' }
}
```

==BTW：输入文件一定要是最后一个参数，否则会编译失败；==

# Demo

一个简单的`Hello World`程序，使用win10 vs2022 x64的配置进行编译；

```bff
//仅适用于Windows10/11 x64使用vs2022
.Platfrom = 'x64'

//Windows SDK，lib与include目录选择性添加
.WindowsSDKBasePath = 'C:\Program Files (x86)\Windows Kits\10'
.WindowsSDKVersion = '10.0.22621.0'

.WindowsSDKLibBasePath = '$WindowsSDKBasePath$\Lib\$WindowsSDKVersion$'
.WindowsSDKLibPath_um = '$WindowsSDKLibBasePath$\um\$Platfrom$'
.WindowsSDKLibPath_ucrt = '$WindowsSDKLibBasePath$\ucrt\$Platfrom$'
.WindowsSDKLibPath_ucrt_enclave = '$WindowsSDKLibBasePath$\ucrt_enclave\$Platfrom$'

.WindowsSDKIncludeBasePath = '$WindowsSDKBasePath$\Include\$WindowsSDKVersion$'
.WindowsSDKIncludePath_cppwinrt = '$WindowsSDKIncludeBasePath$\cppwinrt'
.WindowsSDKIncludePath_shared = '$WindowsSDKIncludeBasePath$\shared'
.WindowsSDKIncludePath_um = '$WindowsSDKIncludeBasePath$\um'
.WindowsSDKIncludePath_ucrt = '$WindowsSDKIncludeBasePath$\ucrt'
.WindowsSDKIncludePath_winrt = '$WindowsSDKIncludeBasePath$\winrt'

//定义vs2022 x64 编译器与链接器
.VS2022_BasePath         = 'C:\Program Files\Microsoft Visual Studio\2022\Community'
.VS2022_Version          = '14.41.34120'
.VS2022SDKBasePath       = '$VS2022_BasePath$/VC/Tools/MSVC/$VS2022_Version$'
.VS2022CompilerPath      = '$VS2022SDKBasePath$/bin/Host$Platfrom$/$Platfrom$'
Compiler( 'Compiler-VS2022-x64' )
{
    .Executable = '$VS2022CompilerPath$/cl.exe'
    .ExtraFiles = { '$VS2022CompilerPath$/c1.dll'
                    '$VS2022CompilerPath$/c1xx.dll',
                    '$VS2022CompilerPath$/c2.dll',
                    '$VS2022CompilerPath$/atlprov.dll', //仅在使用ATL(Active Template Library)时需要
                    '$VS2022CompilerPath$/msobj140.dll'
                    '$VS2022CompilerPath$/mspdb140.dll'
                    '$VS2022CompilerPath$/mspdbcore.dll'
                    '$VS2022CompilerPath$/mspdbsrv.exe'
                    '$VS2022CompilerPath$/mspft140.dll'
                    '$VS2022CompilerPath$/msvcp140.dll'
                    '$VS2022CompilerPath$/msvcp140_atomic_wait.dll'
                    '$VS2022CompilerPath$/tbbmalloc.dll'
                    '$VS2022CompilerPath$/vcruntime140.dll'
                    '$VS2022CompilerPath$/vcruntime140_1.dll'
                    '$VS2022CompilerPath$/2052/clui.dll' //2052代表简体中文的区域设置代码LCID
                    '$VS2022CompilerPath$/2052/mspft140ui.dll'
                  }
}

.VS2022_Compiler = 'Compiler-VS2022-x64'
.VS2022_Linker = '$VS2022SDKBasePath$/bin/Host$Platfrom$/$Platfrom$/link.exe'

//编译选项配置
.Compiler           = '$VS2022_Compiler$'
.CompilerOptions    = '"%1"'           // Input
                    + ' /Fo"%2"'       // Output
                    + ' /c'
                    + ' /nologo'
                    + ' /W4'
                    + ' /WX'
                    + ' /EHsc'
                    + ' /ZI'
                    + ' /MDd'

//基础include目录
.BaseIncludePaths   = ' /I"./"'
                    + ' /I"$VS2022SDKBasePath$/include"'
                    + ' /I"$WindowsSDKIncludePath_ucrt$"'
.CompilerOptions    + .BaseIncludePaths


//链接选项配置
.Linker             = '$VS2022_Linker$'
.LinkerOptions      = ' /OUT:"%2"'     // Output
                    + ' "%1"'          // Input
                    + ' /WX'
                    + ' /NOLOGO'
                    + ' /DEBUG'

.LibPaths           = ' /LIBPATH:"$VS2022SDKBasePath$/lib/$Platfrom$"'
                    + ' /LIBPATH:"$WindowsSDKLibPath_um$"'
                    + ' /LIBPATH:"$WindowsSDKLibPath_ucrt$"'
                    //+ ' /LIBPATH:"$WindowsSDKLibPath_ucrt_enclave$"'
.LinkerOptions      + .LibPaths

ObjectList( 'HelloWorld-Obj' )
{
    .CompilerInputPath  = '\'
    .CompilerOutputPath = 'Out\'
}

Executable( 'HelloWorld' )
{
    .Libraries          = { "HelloWorld-Obj" }
    .LinkerOutput       = 'Out\HelloWorld.exe'
}

// All
//------------------------------------------------------------------------------
Alias( 'all' ) { .Targets = { 'HelloWorld' } }

```

# 遇到的问题

## 编译出的exe包体过大

有些编译选项会影响编译出的可执行文件的大小，比如：

- 调试信息存储方式：`/Z7`和`/Zd`会将调试信息写入obj文件导致exe大一些，而`/Zi`是生成独立的pdb文件；
- 使用静态库还是动态库：参考[[运行库配置]]，使用静态链接`/MT`和`/MTd`时生成的包体会比使用动态链接的更大；
- 优化等级：最高等级的优化`/O1`生成的代码最小；
- 安全检查：禁用安全检查`/GS-`会减少大小（不推荐）；
- 对代码和数据的处理：`/OPT:REF`移除未使用的函数和数据，`/OPT:ICF`合并相同的函数和数据，有助于减少大小；
- 对内联函数的处理：`/Zc:inline`移除未使用的内联函数，有助于减少大小；

如果之前是使用VS进行编译，可以查看VS的编译配置，并对比修改：
![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241027125911.png)

## 提示找不到lib库

可能是以下原因：
1. 某些头文件中会通过`#pargma comment(lib, "xxx.lib")`的方式指定依赖库，这种方式会在编译时自动添加到链接器的输入中（一般这种依赖库依赖于平台，会通过宏定义的方式组合库名，因此不能直接全局搜索库名）；
2. 编译器会根据目标平台或配置默认添加一些库，如果确认不需要该库，可以通过链接选项`/NODEFAULTLIB`排除；
3. 链接选项中，链接库目录`/LIBPATH`要写在链接库名之前；

## 全局作用域干扰问题

在编写bff时，我通常会使用以下的这种结构：
```bff
//链接选项配置
.Linker             = '$VS2022_Linker$'
.LinkerOptions      = ' /OUT:"%2"'     // Output
                    + ' "%1"'          // Input
                    + ' /WX'
                    + ' /NOLOGO'
                    + ' /DEBUG'

DLL ('aaa')
{
	.AdditionalLibPath = {
		'aaaaa',
		'bbbbb',
		'bbbccc'
	}
	ForEach (.Path in .AdditionalLibPath ){
		^LinkerOptions + ' /LIBPATH:"$Path$"'
	}
	.LinkerOptions + '/ MT'
}
```

结果导致`DLL('aaa')`之后`LinkerOptions`被修改了，这属于对bff语法的一个误读，`^`是**将变量提升到上一个使用的作用域，而不是父作用域**，因此会直接修改到全局作用域；所以在使用`^`之前，应该确保在想要的作用域有提前使用，即使是提前`.LinkerOptions + ''`这样无意义的语句也行；