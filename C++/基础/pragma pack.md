`#pragma pack(push, n)` 和 `#pragma pack(pop)` 是用于控制结构体内存对齐的编译指令。它们在C++中用于确保结构体的成员按指定的字节对齐方式排列，从而避免编译器默认的对齐方式带来的填充字节。

默认情况下，编译器会对结构体的成员进行[[字节对齐]]，以提高内存访问的效率。这通常会在结构体成员之间插入填充字节（`padding`），使得每个成员的地址是其大小的整数倍。例如：

```cpp
struct Example {
    char a;    // 1 byte
    int b;     // 4 bytes
};
```

在默认对齐方式下，`Example`结构体的大小可能是8字节，而不是5字节，因为编译器会在`a`和`b`之间插入3个填充字节，使得`b`的地址是4的倍数。

- `#pragma pack(push, n)`：将当前的对齐方式保存到内部堆栈，并设置新的对齐方式为`n`字节。
- `#pragma pack(pop)`：从内部堆栈中恢复之前保存的对齐方式。

使用`#pragma pack(push, 1)`和`#pragma pack(pop)`可以确保结构体的成员按1字节对齐，即没有填充字节。这在处理文件格式（如BMP文件头）或网络协议时非常有用，因为这些格式通常要求严格的字节对齐。


以下是一个使用`#pragma pack`的示例：

```cpp
#include <iostream>
#include <cstdint>

#pragma pack(push, 1)
struct BMPFileHeader {
    uint16_t fileType;     // 文件类型，必须是'BM'（0x4D42）
    uint32_t fileSize;     // 文件大小（以字节为单位）
    uint16_t reserved1;    // 保留字段，必须为0
    uint16_t reserved2;    // 保留字段，必须为0
    uint32_t offsetData;   // 像素数据的起始位置（以字节为单位）
};
#pragma pack(pop)

int main() {
    std::cout << "Size of BMPFileHeader: " << sizeof(BMPFileHeader) << " bytes" << std::endl;
    return 0;
}
```
