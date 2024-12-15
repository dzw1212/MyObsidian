模块（Modules）是构建和组织代码的基本单元，它们帮助开发者将代码分割成可管理的部分，并且可以独立编译和链接；

模块通常包含一个或多个源文件和头文件，并且可以定义自己的依赖关系。


- **作用**：模块用于组织代码，使得代码可以被重用和独立编译。每个模块都有自己的构建规则和依赖关系。
- **定义**：在虚幻引擎中，模块通过一个名为`<ModuleName>.Build.cs`的文件来定义。这个文件指定了模块的依赖项、包含的源文件路径、公共和私有的包含路径等。

以下是一个典型的`Build.cs`文件：
```cpp
using UnrealBuildTool;

public class EditorUtility : ModuleRules
{
    public EditorUtility(ReadOnlyTargetRules Target) : base(Target)
    {
        // 使用显式或共享的预编译头文件
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
    
        // 公共依赖模块
        PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore" });

        // 私有依赖模块
        PrivateDependencyModuleNames.AddRange(new string[] {  });

        // 如果使用Slate UI，取消注释以下行
        // PrivateDependencyModuleNames.AddRange(new string[] { "Slate", "SlateCore" });
        
        // 如果使用在线功能，取消注释以下行
        // PrivateDependencyModuleNames.Add("OnlineSubsystem");

        // 如果要包含OnlineSubsystemSteam，请在uproject文件的插件部分中添加，并将Enabled属性设置为true
    }
}
```

当一个模块的功能需要被其他模块直接使用时，这些功能所在的模块应该被列为**公共依赖**，公共依赖会影响所有依赖当前模块的模块，因为它们可以直接访问公共依赖模块中的内容;

当一个模块的功能仅在当前模块内部使用，而不需要被其他模块直接访问时，这些功能所在的模块应该被列为**私有依赖**，私有依赖仅影响当前模块，不会影响依赖当前模块的其他模块；

