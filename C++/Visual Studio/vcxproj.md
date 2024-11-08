xml格式；

抬头表明xml版本与所用编码；
```xml
<?xml version="1.0" encoding="utf-8"?>
```

---

`Project`元素为根节点，表明要使用的 MSBuild 版本以及将文件向 MSBuild.exe 传递时要执行的默认目标；

```xml
<Project DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
	......
</Project>
```

`xmlns`定义了一个默认命名空间，适用于`Project`元素及其子元素；
http://schemas.microsoft.com/developer/msbuild/2003 是 Microsoft 为 MSBuild 项目文件定义的标准命名空间，它确保 MSBuild 工具能够正确解析和处理项目文件中的元素和属性；

---

标签为`ProjectConfigurations`的`ItemGroup`元素中，包含了项目配置描述，如`Debug|Win32`、`Release|ARM`等；

`ProjectConfiguration`元素用于表示一个具体的配置，格式为`$(Configuration)|$(Platform)`，其包含`Configuration`和`Platform`两个元素；

```xml
<ItemGroup Label="ProjectConfigurations">
  <ProjectConfiguration Include="Debug|x64">
    <Configuration>Debug</Configuration>
    <Platform>x64</Platform>
  </ProjectConfiguration>
  <ProjectConfiguration Include="Release|x64">
    <Configuration>Release</Configuration>
    <Platform>x64</Platform>
  </ProjectConfiguration>
</ItemGroup>
```

---

标签为`Globals`的`PropertyGroup`元素表示项目级设置，通常文件中仅存在一个`Globals`，包含：
- `ProjectGuid`：项目的唯一标识符；
- `RootNamespace`：项目的根命名空间；
- `WindowsTargetPlatformVersion`：目标Windows平台的版本；
- `VCProjectVersion`：参考[[Visual C++版本号]]；
- `Keyword`：项目类型；

```xml
<PropertyGroup Label="Globals">
  <VCProjectVersion>17.0</VCProjectVersion>
  <Keyword>Win32Proj</Keyword>
  <ProjectGuid>{27f2fde5-48b7-4e35-b2ae-d0fa2b89c09c}</ProjectGuid>
  <RootNamespace>FASTBuildTestProj</RootNamespace>
  <WindowsTargetPlatformVersion>10.0</WindowsTargetPlatformVersion>
</PropertyGroup>
```

---

项目名为`Microsoft.Cpp.default.props`的`Import`元素包含项目的默认设置，`Import`元素用于导入其他项目文件或属性表；
- `Project`：指定要导入的文件路径；
- `$(VCTargetsPath)`是一个环境变量，通常指向 Visual Studio 安装目录下的MSBuild 目标文件路径；
- `Microsoft.Cpp.Default.props`：是一个默认的属性文件，包含了 Visual C++ 项目的基本属性设置；

```xml
<Import Project="$(VCTargetsPath)\Microsoft.Cpp.Default.props" />
```

---

标签为`Configuration`的`PropertyGroup`元素，包含了特定配置组合的附加属性；
- `ConfigurationType`：项目的输出类型；
	- `Application`：可执行文件；
	- `DynamicLibrary`：动态链接库DLL；
	- `StaticLibrary`：静态库Lib；
	- `Utility`：不生成任何二进制文件，通常用于构建过程中的自定义任务或工具；
	- `Makefile`：允许使用自定义的makefile来构建项目；
- `UseDebugLibraries`：是否使用调试库，一般仅在Debug模式下为true；
- `PlatformToolset`：参考[[平台工具集版本]]；
- `WholeProgramOptimization`：是否启用全程序优化，一般在Release/Publish模式下为true；
- `CharacterSet`：编码字符集；

```xml
<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|x64'" Label="Configuration">
  <ConfigurationType>Application</ConfigurationType>
  <UseDebugLibraries>true</UseDebugLibraries>
  <PlatformToolset>v143</PlatformToolset>
  <CharacterSet>Unicode</CharacterSet>
</PropertyGroup>
<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|x64'" Label="Configuration">
  <ConfigurationType>Application</ConfigurationType>
  <UseDebugLibraries>false</UseDebugLibraries>
  <PlatformToolset>v143</PlatformToolset>
  <WholeProgramOptimization>true</WholeProgramOptimization>
  <CharacterSet>Unicode</CharacterSet>
</PropertyGroup>
```

---

项目名为`Microsoft.Cpp.props`的`Import`元素，`Microsoft.Cpp.props`属性表为许多特定于工具的属性定义了默认值，比如如果`CharacterSet`设置为`Unicode`，则`Microsoft.Cpp.props`中监测到后就会包含对应的`microsoft.Cpp.unicodesupport.props`；

```xml
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.props" />
```

---

标签名为`ExtensionSettings`的`ImportGroup`元素，包含属于生成自定义项的属性表的导入；

```xml
<ImportGroup Label="ExtensionSettings">
</ImportGroup>
```

---

标签名为`PropertySheets`的`ImportGroup`元素，包含用户属性表的导入，这些导入的列出顺序很重要，且会反映在属性管理器中。 项目文件通常包含此类导入组的多个实例，每个项目配置各一个；

```xml
<ImportGroup Label="PropertySheets" Condition="'$(Configuration)|$(Platform)'=='Debug|x64'">
  <Import Project="$(UserRootDir)\Microsoft.Cpp.$(Platform).user.props" Condition="exists('$(UserRootDir)\Microsoft.Cpp.$(Platform).user.props')" Label="LocalAppDataPlatform" />
</ImportGroup>
<ImportGroup Label="PropertySheets" Condition="'$(Configuration)|$(Platform)'=='Release|x64'">
  <Import Project="$(UserRootDir)\Microsoft.Cpp.$(Platform).user.props" Condition="exists('$(UserRootDir)\Microsoft.Cpp.$(Platform).user.props')" Label="LocalAppDataPlatform" />
</ImportGroup>
```

---

标签名为`UserMacros`的`PropertyGroup`元素，包含创建为变量的属性，这些属性用于自定义生成过程；

```xml
  <PropertyGroup Label="UserMacros" />
```

---

`ItemDefinitionGroup`元素，用于定义特定配置和平台组合的编译和链接选项；
- `ClCompile`：编译器相关配置：
	- `WarningLevel`：设置编译器的警告级别；
	- `SDLCheck`：是否启用安全开发生命周期检查；
	- `PreprocessorDefinitions`：定义预处理器宏；
		- `_DEBUG/NDEBUG`：启用/禁用调试相关代码；
		- `_CONSOLE`：表示这是一个控制台应用程序；
		- `%(PreprocessorDefinitions)`：继承其他定义；
	- `ConformanceMode`：是否启用标准一致性模式；
	- `FunctionLevelLinking`：是否启用函数级链接，启用后可减少可执行文件的大小；
	- `IntrinsicFunctions`：是否启用内置函数，启用后可提高性能；
	- `PrecompiledHeader`：是否使用预编译头文件；
	- `PrecompiledHeaderFile`：如果使用，指定预编译头文件；
	- `AdditionalIncludeDirectories`：额外包含目录；
	- `Optimization`：设置优化等级；
		- `Disabled`：不进行优化；
		- `MiniSpace`：优先最小化代码大小；
		- `MaxSpeed`：优先最大化代码速度；
		- `Full`：启用完整优化；
		- `Custom`：自定义优化设置；
	- `MinimalRebuild`：是否启用最小重建；
	- `StringPooling`：是否启用字符串池化；
	- `RuntimeLibrary`：是否启用多线程DLL运行时库；
	- `MultiProcessorCompilation`：是否启用多处理器编译；
	- `LanguageStandard`：指定C++标准；
- `Link`：链接相关配置：
	- `SubSystem`：子系统类型；
	- `GenerateDebugInformation`：是否生成调试信息；
	- `EnableCOMDATFolding`：是否启用COMDAT折叠，启用后可以在链接时合并重复的代码和数据段，减少可执行文件的大小；
	- `OptimizeReferences`：是否启用引用优化，启用时可以在链接时移除未使用的函数和数据，减少可执行文件的大小；
- `Lib`：链接库相关配置：
	- `AdditionalDependencies`：指定额外依赖库；

```xml
  <ItemDefinitionGroup Condition="'$(Configuration)|$(Platform)'=='Release|x64'">
    <ClCompile>
      <PrecompiledHeader>Use</PrecompiledHeader>
      <PrecompiledHeaderFile>DazelPCH.h</PrecompiledHeaderFile>
      <WarningLevel>Level3</WarningLevel>
      <PreprocessorDefinitions>DAZEL_PLATFORM_WINDOWS;DAZEL_BUILD_DLL;DAZEL_ENABLE_ASSERTS;_CRT_SECURE_NOT_WARNINGS;GLFW_INCLUDE_NONE;DAZEL_RELEASE;%(PreprocessorDefinitions)</PreprocessorDefinitions>
      <AdditionalIncludeDirectories>src;Vender\spdlog\include;Vender\GLFW\include;Vender\GLAD\include;Vender\ImGui;Vender\glm;Vender\stb_image;Vender\entt\include;Vender\yaml-cpp\include;Vender\ImGuizmo;C:\VulkanSDK\Include;Vender\Box2D\include;Vender\mono\include;Vender\filewatch;%(AdditionalIncludeDirectories)</AdditionalIncludeDirectories>
      <Optimization>Full</Optimization>
      <FunctionLevelLinking>true</FunctionLevelLinking>
      <IntrinsicFunctions>true</IntrinsicFunctions>
      <MinimalRebuild>false</MinimalRebuild>
      <StringPooling>true</StringPooling>
      <RuntimeLibrary>MultiThreadedDLL</RuntimeLibrary>
      <MultiProcessorCompilation>true</MultiProcessorCompilation>
      <LanguageStandard>stdcpplatest</LanguageStandard>
    </ClCompile>
    <Link>
      <SubSystem>Windows</SubSystem>
      <EnableCOMDATFolding>true</EnableCOMDATFolding>
      <OptimizeReferences>true</OptimizeReferences>
    </Link>
    <Lib>
      <AdditionalDependencies>opengl32.lib;Vender\mono\lib\Release\libmono-static-sgen.lib;Ws2_32.lib;Winmm.lib;Version.lib;Bcrypt.lib;C:\VulkanSDK\Lib\shaderc_shared.lib;C:\VulkanSDK\Lib\spirv-cross-core.lib;C:\VulkanSDK\Lib\spirv-cross-glsl.lib;%(AdditionalDependencies)</AdditionalDependencies>
    </Lib>
  </ItemDefinitionGroup>
```

---

`ItemGroup`元素，包含项目中的源文件等，不支持使用通配符或宏；

```xml
<ItemGroup>
  <ClCompile Include="main.cpp">
    <TreatWarningAsError Condition="'$(Configuration)|$(Platform)'=='Debug|x64'">true</TreatWarningAsError>
    <TreatWarningAsError Condition="'$(Configuration)|$(Platform)'=='Release|x64'">true</TreatWarningAsError>
  </ClCompile>
</ItemGroup>
```

---

项目名为`Microsoft.Cpp.targets`的`Import`元素，定义 C++ 目标，例如生成、清理等；

```xml
<Import Project="$(VCTargetsPath)\Microsoft.Cpp.targets" />
```

---

标签名为`ExtensionTargets`的`ImportGroup`元素，包含生成自定义目标文件的导入；

```xml
<ImportGroup Label="ExtensionTargets">
</ImportGroup>
```

