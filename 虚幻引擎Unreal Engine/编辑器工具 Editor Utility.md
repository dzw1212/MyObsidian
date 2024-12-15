# 创建一个插件

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241126215814.png)

修改插件的`.uplugin`文件，使其先于游戏模块加载前加载；

```cpp
{
	"FileVersion": 3,
	"Version": 1,
	"VersionName": "1.0",
	"FriendlyName": "TestManager",
	"Description": "",
	"Category": "Other",
	"CreatedBy": "",
	"CreatedByURL": "",
	"DocsURL": "",
	"MarketplaceURL": "",
	"SupportURL": "",
	"CanContainContent": true,
	"IsBetaVersion": false,
	"IsExperimentalVersion": false,
	"Installed": false,
	"Modules": [
		{
			"Name": "TestManager",
			"Type": "Editor", //仅在编辑器下
			"LoadingPhase": "PreDefault" //先于游戏模块加载
		}
	]
}
```

# 添加一个Asset Action


创建一个AssetAction，将其放置在新建的插件模块下：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241127121052.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241127121312.png)

接下来需要解决`TestAssetActionUtility.h`中的包含问题：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241127121737.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241127121756.png)

在插件的模块的构建文件`TestManager.Build.cs`中：
1. 添加模块依赖；
![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241127234858.png)

2. 添加包含目录；
![780](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241127235004.png)


添加一个右键功能：
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241128014722.png)
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241128014750.png)

然后右键资源，就会发现多出的选项：
![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241128014904.png)

# 实现批量复制粘贴的功能

主要借助了：
`UEditorUtilityLibrary::GetSelectedAssetData()`：获取当前所有选中资源的信息；
`UEditorAssetLibrary::DuplicateAsset`：复制资源；

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241128230049.png)

UE会自动为入参参数创建输入控件：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241128230334.png)

# 消息对话框

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241128235431.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241128232312.png)

# 消息通知

桌面右下角弹出的消息弹窗；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241129000830.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241129000803.png)

# 资源自动添加前缀

主要借助了资源重命名接口：
`UEditorUtilityLibrary::RenameAsset`；

[[推荐资源命名前缀]]

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241129004857.png)

![650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241129004928.png)

# 删除未被引用的资源

主要借助接口：
`UEditorAssetLibrary::FindPackageReferencersForAsset`；
`ObjectTools::DeleteAssets`；

![950](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241129011124.png)

# 修复重定向

当在项目中重命名、移动或删除资源时，虚幻会创建一个重定向器（`Redirector`）来保持对旧资源路径的引用，以防止项目中的其他资源或代码失效；

重定向器的存在可以帮助在开发过程中保持项目的稳定性，但它们也可能导致项目的复杂性增加，尤其是在项目中存在大量重定向器时；

`Fix up Redirectors`操作会扫描文件夹下的所有重定向器，并更新所有引用这些重定向器的资源，使它们直接引用到新的资源路径，然后删除这些重定向器，这可以帮助减少项目的复杂性和潜在的错误；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130141348.png)

主要借助：
`FModuleManager::Get().LoadModuleChecked<XXX>(TEXT("XXX"))`：载入并验证某个模块；
`assetRegistryModule.Get().GetAssets`：筛选出所有符合条件的资源；
`assetToolsModule.Get().FixupReferencers`：执行修复；

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130145029.png)

在判断资源是否被引用之前，应该先执行修复重定向操作：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130145252.png)

# 自定义内容浏览器右键菜单项

## 删除目录下所有未使用的资源

在路径浏览窗口右键弹出的选项中，在`Delete`选项下面加一个自定义的选项`Delete Unused Asset`，用于遍历删除该目录下所有未被使用的资源；

[[调试方法#显示UI扩展点]]

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130161740.png)


在`TestModule`模块加载时添加右键菜单项：
![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130223513.png)

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130223709.png)

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130223841.png)


![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130212049.png)


![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130223945.png)

点击时执行`OnCustomMenuEntry`；

不需要遍历的目录：
- `Collections` 是一种虚幻引擎提供的资源组织方式，允许开发者将资源分组，而不改变资源在文件系统中的实际位置。它们类似于标签或虚拟文件夹；
- `Developers` 目录是一个特殊的目录，用于存放开发者个人的临时或实验性资源。这些资源通常不被纳入版本控制；

这两个目录下的资源不应该被删除；
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130220537.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241130224040.png)

## 删除目录下所有空文件夹

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241201003229.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241201003251.png)

# 创建详细删除控件

## 注册/注销控件

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208204308.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208204340.png)

## 唤出控件

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241205001102.png)

创建一个名为"AdvanceDeletion"的`NomadTab`控件：
[[Slate#Slot槽位]]
![650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241205001246.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241205001325.png)

然后调出这个名为"AdvanceDeletion"的控件：
![575](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241205001410.png)

## 给控件传递参数

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241205001747.png)

创建时传递参数给控件：
![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241205001846.png)


控件读取参数：
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241205002713.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241205002829.png)

## 构建列出所有资源的ListView

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208150623.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208150721.png)

`SListView`的关键点是，通过`.ListItemSorece`指定源数据，与通过`.OnGenerateRow`指定每一行如何构建；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208150754.png)

构建每一行的控件：
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208151000.png)

### CheckBox

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208151253.png)

### TextBlock

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208151317.png)

### Button

关键点：`SListView`的`RebuildList`方法，可以刷新整个列表；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208151349.png)

## 功能按钮

### 删除所有选中的资源

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208165500.png)

### 全部选中

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208165608.png)

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208165631.png)


### 全部取消选中

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208165651.png)

## 筛选条件 ComboBox

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208183613.png)

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208183659.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208183805.png)

当点击选项时触发：
![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208183844.png)

## 跳转到资源所在的文件浏览器

![870](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208190402.png)

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241208190430.png)

# 给菜单项添加图标

Step1、将图标放进资源文件夹，一般是`模块名/Resources/`；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241211013820.png)


Step2、注册对应的图标；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241211013859.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241211013936.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241211013956.png)

Step3、`AddMenuEntry`时执行`SlateStyleSet`与图标名；

![750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241211014118.png)

# 快速创建材质蓝图

创建一个基于EditorUtilityWidget的C++类：

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214181851.png)


创建一个EditorUtilityWidget蓝图，将其父类设为刚才创建的C++类：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214175839.png)

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214181730.png)

在类中创建一个UFNCTION和UPROPERTY：

![750](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214183656.png)

## 使用DetailView访问类的成员变量

借助`DetailView`细节视图，在控件上显示对应的UPROPERTY：

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214183923.png)

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214184020.png)

在Pre Construct时，给`DetailView`设置对象：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214185126.png)

然后运行控件，就能看到成功访问到了类的成员变量：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214185223.png)


## 筛选出Texture2D资源

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214203216.png)

## 检查要创建的Material路径是否可用

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214205129.png)

## 通过代码创建Material资源

```cpp
#include "AssetToolsModule.h"
#include "Factories/MaterialFactoryNew.h"
```

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241214212434.png)


## 贴图连接到对应节点

根据后缀名来区分贴图，但首先需要支持用户自定义命名规则：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215010142.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215010345.png)

### BaseColor

![650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215010526.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215010550.png)


### Metallic

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215010618.png)

---

其他（Roughness，Normal，AO等）类似；

## 对于混合通道贴图

这种贴图可能R通道表示Normal，G通道表示Roughness，需要特殊设置；

`SamplerType`和`CompressionType`需要改成`Masks`；

用一个enum区分普通贴图与混合通道贴图：

![650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215012614.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215013404.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215013854.png)

通过偏移指定R/G/B通道；

![650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215015103.png)

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215015135.png)

## 创建材质实例

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215022805.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215022844.png)

# 操作关卡内的Actor

操作关卡内的Actor，属于运行时，因此需要使用[[子系统 Subsystem]]中的`UEditorActorSystem`；

关键接口：
`GetSelectedLevelActors()`
`GetAllLevelActors()`

## 获取EditorActorSubsystem

创建一个基于EditorUtilityWidget的C++类：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215030052.png)


![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215025927.png)


## 选中关卡中相似的Actor

关键接口：
`EditorActorSubsystem->GetSelectedLevelActors`
`EditorActorSubsystem->GetAllLevelActors`
`EditorActorSubsystem->SetActorSelectionState`

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215174208.png)


![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215173815.png)

## 批量复制Actor

主要接口：
`EditorActorSubsystem->DuplicateActor`


![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215183717.png)

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215183755.png)
![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215183853.png)



# 拓展关卡内的右键菜单

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215214533.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215214616.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241215214813.png)

## 锁定/解锁可选中状态

当一个Actor锁定选中时，给它加一个tag，当OnSelected回调中检测到该tag，则不选中该Actor；同理，解锁时删除该tag即可；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241216010114.png)

通过`USection`获取`OnActorSelected`的回调：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241216010240.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241216010332.png)


# 自定义编辑器快捷键

