```cpp
#include "Misc/FileHelper.h"
#include "Serialization/Csv/CsvParser.h"

void AMyActor::ReadCSVFile()
{
    FString FilePath = FPaths::ProjectContentDir() / TEXT("Path/To/YourFile.csv");
    FString CSVData;

    // 读取 CSV 文件内容
    if (FFileHelper::LoadFileToString(CSVData, *FilePath))
    {
        // 使用 FCSVParser 解析 CSV 数据
        const FCsvParser Parser(CSVData);
        const TArray<TArray<FString>>& Rows = Parser.GetRows();

        // 遍历每一行
        for (const TArray<FString>& Row : Rows)
        {
            // 遍历每一列
            for (const FString& Cell : Row)
            {
                UE_LOG(LogTemp, Log, TEXT("Cell: %s"), *Cell);
            }
        }
    }
    else
    {
        UE_LOG(LogTemp, Error, TEXT("Failed to load CSV file: %s"), *FilePath);
    }
}
```