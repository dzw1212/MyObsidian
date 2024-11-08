# 层级结构

```txt
MyProject/
│
├── MyProject.sln
│
├── Project1/
│   ├── Project1.vcxproj
│   ├── Project1.vcxproj.filters
│   ├── Project1.vcxproj.user
│   ├── src/
│   │   └── main.cpp
│   └── include/
│       └── header.h
│
└── Project2/
    ├── Project2.vcxproj
    ├── Project2.vcxproj.filters
    ├── Project2.vcxproj.user
    └── resource.rc
```

# 相关文件
## .sln

`.sln` 文件（解决方案文件）

   - 位置：通常位于项目的根目录
   - 作用：
     - 包含整个解决方案的配置信息
     - 管理多个相关项目
     - 存储解决方案级别的设置和项目之间的依赖关系

## .vcxproj

`.vcxproj` 文件（项目文件）

   - 位置：每个项目的根目录
   - 作用：
     - 定义单个项目的配置和设置
     - 包含项目的源文件、头文件、资源文件等的列表
     - 指定编译器和链接器的选项

## .vcxproj.filters

`.vcxproj.filters` 文件（项目筛选器文件）

   - 位置：与对应的 .vcxproj 文件位于同一目录
   - 作用：
     - 定义 Visual Studio 解决方案资源管理器中的虚拟文件夹结构
     - 帮助组织和分类项目中的文件，但不影响实际的文件系统结构

## .vcxproj.user

`.vcxproj.user` 文件（用户特定的项目设置）

   - 位置：与对应的 .vcxproj 文件位于同一目录
   - 作用：
     - 存储特定用户的项目设置，如调试配置
     - 通常不包含在版本控制中，因为它是针对个人的

## .props

`.props` 文件（属性表）

   - 位置：可以在解决方案或项目目录中，具体位置可自定义
   - 作用：
     - 定义可以在多个项目间共享的属性和设置
     - 用于统一管理公共配置

## .rc

`.rc` 文件（资源脚本文件）

   - 位置：通常在项目的 "Resource Files" 文件夹中
   - 作用：
     - 定义 Windows 应用程序的资源，如图标、对话框、菜单等

# 常见配置

## Debug/Release/x86/x64

主要存储在`.vcxproj`文件中，
```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" xmlns="...">
  <!-- ... 其他项目设置 ... -->

  <ItemGroup Label="ProjectConfigurations">
    <ProjectConfiguration Include="Debug|Win32">
      <Configuration>Debug</Configuration>
      <Platform>Win32</Platform>
    </ProjectConfiguration>
    <ProjectConfiguration Include="Release|Win32">
      <Configuration>Release</Configuration>
      <Platform>Win32</Platform>
    </ProjectConfiguration>
    <!-- 可能还有其他平台的配置 -->
  </ItemGroup>

  <!-- Debug 配置特定设置 -->
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'">
    <!-- Debug 模式的编译器和链接器选项 -->
  </PropertyGroup>

  <!-- Release 配置特定设置 -->
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|Win32'">
    <!-- Release 模式的编译器和链接器选项 -->
  </PropertyGroup>

  <!-- ... 其他项目设置 ... -->
</Project>
```

可能有些用户的配置会覆盖`.vcxproj`中的配置，存储在`.vcxproj.user`中；

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="Current" xmlns="...">
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'">
    <!-- 用户特定的 Debug 设置覆盖 -->
  </PropertyGroup>
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|Win32'">
    <!-- 用户特定的 Release 设置覆盖 -->
  </PropertyGroup>
</Project>
```

除此之外，有时这些配置可能会提取到单独的`.props`文件中，然后在`.vcxproj`中引用；

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="...">
  <ImportGroup Label="PropertySheets">
    <Import Project="$(UserRootDir)\Microsoft.Cpp.$(Platform).user.props" Condition="exists('$(UserRootDir)\Microsoft.Cpp.$(Platform).user.props')" Label="LocalAppDataPlatform" />
  </ImportGroup>
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'">
    <!-- 共享的 Debug 设置 -->
  </PropertyGroup>
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|Win32'">
    <!-- 共享的 Release 设置 -->
  </PropertyGroup>
</Project>
```

```xml
<Import Project="Common.props" />
```

## 外部包含目录include

同上，也是主要`.vcxproj`，有可能`.vcxproj.user`或`.props`；

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" xmlns="...">
  <!-- ... 其他项目设置 ... -->

  <ItemDefinitionGroup Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'">
    <ClCompile>
      <AdditionalIncludeDirectories>C:\ExternalLibs\include;$(SolutionDir)CommonIncludes;%(AdditionalIncludeDirectories)</AdditionalIncludeDirectories>
      <!-- 其他编译器设置 -->
    </ClCompile>
    <!-- 其他设置 -->
  </ItemDefinitionGroup>

  <ItemDefinitionGroup Condition="'$(Configuration)|$(Platform)'=='Release|Win32'">
    <ClCompile>
      <AdditionalIncludeDirectories>C:\ExternalLibs\include;$(SolutionDir)CommonIncludes;%(AdditionalIncludeDirectories)</AdditionalIncludeDirectories>
      <!-- 其他编译器设置 -->
    </ClCompile>
    <!-- 其他设置 -->
  </ItemDefinitionGroup>

  <!-- 可能还有其他平台的配置 -->

  <!-- ... 其他项目设置 ... -->
</Project>
```

## 链接库目录 lib文件

同上，也是主要`.vcxproj`，有可能`.vcxproj.user`或`.props`；

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" xmlns="...">
  <!-- ... 其他项目设置 ... -->

  <ItemDefinitionGroup Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'">
    <Link>
      <!-- 链接库目录 -->
      <AdditionalLibraryDirectories>C:\ExternalLibs\lib;$(SolutionDir)Libs;%(AdditionalLibraryDirectories)</AdditionalLibraryDirectories>
      
      <!-- 要链接的lib文件 -->
      <AdditionalDependencies>example1.lib;example2.lib;%(AdditionalDependencies)</AdditionalDependencies>
    </Link>
  </ItemDefinitionGroup>

  <ItemDefinitionGroup Condition="'$(Configuration)|$(Platform)'=='Release|Win32'">
    <Link>
      <AdditionalLibraryDirectories>C:\ExternalLibs\lib;$(SolutionDir)Libs;%(AdditionalLibraryDirectories)</AdditionalLibraryDirectories>
      <AdditionalDependencies>example1.lib;example2.lib;%(AdditionalDependencies)</AdditionalDependencies>
    </Link>
  </ItemDefinitionGroup>

  <!-- ... 其他配置 ... -->
</Project>
```

## 编译成lib或exe或dll

同上，也是主要`.vcxproj`，有可能`.vcxproj.user`或`.props`；

`ConfigurationType`的值有：
- `Application`：生成exe；
- `StaticLibrary`：生成静态库；
- `DynamicLibrary`：生成动态库；
- `Utility`：用于其他用途的项目类型；

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project DefaultTargets="Build" xmlns="...">
  <ItemGroup Label="ProjectConfigurations">
    <!-- 项目配置定义 -->
  </ItemGroup>

  <PropertyGroup Label="Globals">
    <!-- 全局属性 -->
  </PropertyGroup>

  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.Default.props" />

  <!-- 这里定义了项目类型 -->
  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|Win32'" Label="Configuration">
    <ConfigurationType>Application</ConfigurationType>
    <!-- 其他配置 -->
  </PropertyGroup>

  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|Win32'" Label="Configuration">
    <ConfigurationType>Application</ConfigurationType>
    <!-- 其他配置 -->
  </PropertyGroup>

  <!-- 可能还有其他平台的配置 -->

  <!-- 其他项目设置 -->
</Project>
```


## 分成多个子项目

.sln 文件是整个解决方案的核心，它定义了解决方案中包含的所有项目以及它们之间的关系；

- `Project`定义了一个子项目；
- `ProjectSection(ProjectDependencies)`定义了项目之间的依赖关系；
- `{8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942}`为项目的全局唯一标识符GUID，用于VS内部标识项目类型，作为用户不需要了解；

```sln
Microsoft Visual Studio Solution File, Format Version 12.00
# Visual Studio Version 16
VisualStudioVersion = 16.0.31515.178
MinimumVisualStudioVersion = 10.0.40219.1
Project("{8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942}") = "MainExe", "MainExe\MainExe.vcxproj", "{GUID1}"
    ProjectSection(ProjectDependencies) = postProject
        {GUID2} = {GUID2}
        {GUID3} = {GUID3}
    EndProjectSection
EndProject
Project("{8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942}") = "LibProject1", "LibProject1\LibProject1.vcxproj", "{GUID2}"
EndProject
Project("{8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942}") = "LibProject2", "LibProject2\LibProject2.vcxproj", "{GUID3}"
EndProject
Global
    # 全局配置设置
EndGlobal
```