常用的文件相关的类有`FFileHelper`、`FFileReader`、`FFileWriter`；

# FileReader/FileWriter

二进制流类，提供了相对`FFileHelper`更底层的文件控制，适合需要精细管理文件流的场景，需要手动打开关闭文件；

## FileReader读取

```cpp
#include "HAL/PlatformFilemanager.h"
#include "Misc/FileHelper.h"
#include "Serialization/Archive.h"

void ReadDataFromFile(const FString& FilePath)
{
    // 获取平台文件管理器
    IPlatformFile& PlatformFile = FPlatformFileManager::Get().GetPlatformFile();

    // 打开文件进行读取
    TUniquePtr<FArchive> FileReader(PlatformFile.OpenRead(*FilePath));

    if (FileReader)
    {
        // 准备读取的数据变量
        int32 IntegerValue;
        float FloatValue;
        FString StringValue;

        // 使用 << 操作符从文件中读取数据
        *FileReader << IntegerValue;
        *FileReader << FloatValue;
        *FileReader << StringValue;

        // 关闭文件
        FileReader->Close();

        // 输出读取的数据
        UE_LOG(LogTemp, Log, TEXT("Read Integer: %d"), IntegerValue);
        UE_LOG(LogTemp, Log, TEXT("Read Float: %f"), FloatValue);
        UE_LOG(LogTemp, Log, TEXT("Read String: %s"), *StringValue);
    }
    else
    {
        // 处理文件打开失败的情况
        UE_LOG(LogTemp, Error, TEXT("Failed to open file for reading: %s"), *FilePath);
    }
}
```

## FileWriter写入

通过 `FArchive` 的子类实现文件写入功能；

```cpp
#include "HAL/PlatformFilemanager.h"
#include "Misc/FileHelper.h"
#include "Serialization/FileWriter.h"

void WriteDataToFile(const FString& FilePath)
{
    // 获取平台文件管理器
    IPlatformFile& PlatformFile = FPlatformFileManager::Get().GetPlatformFile();

    // 打开文件进行写入
    TUniquePtr<FArchive> FileWriter = TUniquePtr<FArchive>(PlatformFile.CreateFileWriter(*FilePath));

    if (FileWriter)
    {
        // 准备要写入的数据
        int32 IntegerValue = 42;
        float FloatValue = 3.14f;
        FString StringValue = TEXT("Hello Unreal");

        // 使用 << 操作符将数据写入文件
        *FileWriter << IntegerValue;
        *FileWriter << FloatValue;
        *FileWriter << StringValue;

        // 关闭文件
        FileWriter->Close();
    }
    else
    {
        // 处理文件打开失败的情况
        UE_LOG(LogTemp, Error, TEXT("Failed to open file for writing: %s"), *FilePath);
    }
}
```


# FFileHelper

`FFileHelper`是更为高级的文件类，提供了一组静态接口处理文本/二进制文件的读写，不需要手动打开与关闭文件；

## 文本文件

读取：
```cpp
#include "Misc/FileHelper.h"
#include "Misc/Paths.h"

void ReadTextFile(const FString& FilePath)
{
    FString FileContent;
    if (FFileHelper::LoadFileToString(FileContent, *FilePath))
    {
        UE_LOG(LogTemp, Log, TEXT("File Content: %s"), *FileContent);
    }
    else
    {
        UE_LOG(LogTemp, Error, TEXT("Failed to read file: %s"), *FilePath);
    }
}
```

写入：
```cpp
#include "Misc/FileHelper.h"
#include "Misc/Paths.h"

void WriteTextFile(const FString& FilePath, const FString& Content)
{
    if (FFileHelper::SaveStringToFile(Content, *FilePath))
    {
        UE_LOG(LogTemp, Log, TEXT("Successfully wrote to file: %s"), *FilePath);
    }
    else
    {
        UE_LOG(LogTemp, Error, TEXT("Failed to write to file: %s"), *FilePath);
    }
}
```

## 二进制文件

读取：
```cpp
#include "Misc/FileHelper.h"
#include "Misc/Paths.h"

void ReadBinaryFile(const FString& FilePath)
{
    TArray<uint8> FileData;
    if (FFileHelper::LoadFileToArray(FileData, *FilePath))
    {
        UE_LOG(LogTemp, Log, TEXT("Successfully read binary file: %s"), *FilePath);
    }
    else
    {
        UE_LOG(LogTemp, Error, TEXT("Failed to read binary file: %s"), *FilePath);
    }
}
```

写入：
```cpp
#include "Misc/FileHelper.h"
#include "Misc/Paths.h"

void WriteBinaryFile(const FString& FilePath, const TArray<uint8>& Data)
{
    if (FFileHelper::SaveArrayToFile(Data, *FilePath))
    {
        UE_LOG(LogTemp, Log, TEXT("Successfully wrote binary file: %s"), *FilePath);
    }
    else
    {
        UE_LOG(LogTemp, Error, TEXT("Failed to write binary file: %s"), *FilePath);
    }
}
```

## 配合FArchive
