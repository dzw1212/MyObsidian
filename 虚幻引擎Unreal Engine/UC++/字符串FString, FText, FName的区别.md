在虚幻中，`FName`、`FText` 和 `FString` 是三种不同的字符串类型，它们各自有不同的用途和特性。以下是它们的主要区别：

# FName

- 用途：用于**标识和查找对象**，特别是在需要频繁比较和查找的场景中。
- 性能：非常**高效**，内部使用哈希表进行字符串存储和比较。
- 内存：使用字符串表，避免了重复字符串的内存开销。
- 可变性：一旦创建，`FName` 的值是**不可变**的。

  ```cpp
  FName MyName("MyActorName");
  if (MyName == FName("MyActorName")) { /* 名称匹配 */ }
  ```

# FText

- 用途：用于**本地化和国际化的文本**，适合需要翻译的用户界面文本和对话。
- 性能：相对较慢，因为它支持复杂的本地化功能。
- 内存：**较高的内存开销**，因为它包含了本地化数据。
- 可变：可以动态改变内容，支持格式化和本地化。

  ```cpp
  FText LocalizedText = NSLOCTEXT("Namespace", "Key", "Localized Text");
  FText FormattedText = FText::Format(FText::FromString("Hello, {0}!"), FText::FromString("Unreal Engine"));
  ```

# FString

- 用途：用于**一般的字符串操作**，如文件路径、日志信息等。
- 性能：性能适中，适合大多数字符串操作。
- 内存：直接存储字符串，内存开销取决于字符串长度。
- 可变：可以动态改变内容，支持各种字符串操作。

  ```cpp
  FString MyString = TEXT("Hello, World!");
  MyString.Append(TEXT(" Unreal Engine"));
  ```
