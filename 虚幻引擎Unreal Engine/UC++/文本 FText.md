在虚幻中，`FText` 是一种用于本地化和国际化的文本类型。它与 `FString` 和 `FName` 不同，`FText` 主要用于需要翻译和本地化的文本。

# 使用方法

1. **创建 `FText` 对象**：
   - 使用 `FText::FromString` 从 `FString` 创建 `FText`。
   - 使用 `FText::FromName` 从 `FName` 创建 `FText`。
   - 使用 `NSLOCTEXT` 宏创建本地化文本。

2. **格式化 `FText`**：
   - 使用 `FText::Format` 进行文本格式化。

3. **本地化支持**：
   - `FText` 支持多语言翻译和本地化，适用于需要在不同语言环境中显示的文本。


- **`FText::FromString`**：将 `FString` 转换为 `FText`。
- **`FText::FromName`**：将 `FName` 转换为 `FText`。
- **`NSLOCTEXT`**：用于创建本地化文本，参数包括命名空间、键和值。
- **`FText::Format`**：用于格式化文本，类似于 `printf` 的功能。

# 示例代码

```cpp
// 创建 FText 对象
FText TextFromString = FText::FromString(TEXT("Hello, World!"));
FText TextFromName = FText::FromName("HelloName");

// 使用 NSLOCTEXT 宏创建本地化文本
FText LocalizedText = NSLOCTEXT("Namespace", "Key", "Localized Text");

// 格式化 FText
FText FormatPattern = FText::FromString(TEXT("Hello, {0}!"));
FText FormattedText = FText::Format(FormatPattern, FText::FromString(TEXT("Unreal Engine")));
```

# 使用场景

- **UI 文本**：在用户界面中显示的文本通常使用 `FText` 以便于本地化。
- **对话系统**：游戏中的对话文本需要支持多语言，适合使用 `FText`。
- **日志和调试信息**：虽然日志信息通常使用 `FString`，但如果需要本地化，也可以使用 `FText`。
