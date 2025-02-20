# 预设路径

```cpp
//获取项目目录
FString ProjectDir = FPaths::ProjectDir();

//获取内容目录
FString ContentDir = FPaths::ProjectContentDir();

//获取保存目录
FString SavedDir = FPaths::ProjectSavedDir();

//......
```

# 判断路径是否存在

```cpp
bool bExists = FPaths::FileExists(FullPath);
```
# 合并路径

```cpp
FString FullPath = FPaths::Combine(FPaths::ProjectDir(), TEXT("SubFolder"), TEXT("File.txt"));
```

# 获取绝对路径

```cpp
FString AbsolutePath = FPaths::ConvertRelativePathToFull(RelativePath);
```