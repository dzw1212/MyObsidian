`BMP`（`Bitmap`）是一种无压缩的图像文件格式，广泛用于Windows操作系统；
它的结构相对简单，适合初学者学习图像处理；

以下是BMP文件格式的详细说明：

# 结构

一个BMP文件由以下几个部分组成：

1. **文件头（File Header）**
2. **信息头（DIB Header 或 Info Header）**
3. **颜色表（Color Table，针对8位及以下的图像）**
4. **像素数据（Pixel Data）**

## 文件头（File Header）

文件头包含文件的基本信息，如文件类型、文件大小和像素数据的起始位置。文件头的结构如下：

```cpp
#pragma pack(push, 1)
struct BMPFileHeader {
    uint16_t fileType;     // 文件类型，必须是'BM'（0x4D42）
    uint32_t fileSize;     // 文件大小（以字节为单位）
    uint16_t reserved1;    // 保留字段，必须为0
    uint16_t reserved2;    // 保留字段，必须为0
    uint32_t offsetData;   // 像素数据的起始位置（以字节为单位）
};
#pragma pack(pop)
```

## 信息头（DIB Header 或 Info Header）

信息头包含图像的详细信息，如图像宽度、高度、颜色深度等。最常见的是BITMAPINFOHEADER结构：

```cpp
#pragma pack(push, 1)
struct BMPInfoHeader {
    uint32_t size;            // 信息头的大小（以字节为单位）
    int32_t width;            // 图像宽度（以像素为单位）
    int32_t height;           // 图像高度（以像素为单位）
    uint16_t planes;          // 颜色平面数，必须为1
    uint16_t bitCount;        // 每像素的位数（1、4、8、16、24、32）
    uint32_t compression;     // 压缩类型（0=不压缩）
    uint32_t sizeImage;       // 图像数据大小（以字节为单位）
    int32_t xPixelsPerMeter;  // 水平分辨率（像素/米）
    int32_t yPixelsPerMeter;  // 垂直分辨率（像素/米）
    uint32_t colorsUsed;      // 实际使用的颜色数
    uint32_t colorsImportant; // 重要的颜色数
};
#pragma pack(pop)
```

## 颜色表（Color Table）

颜色表用于调色板图像（1、4、8位图像），存储图像中使用的颜色。每个颜色由一个RGB三元组表示。

## 像素数据（Pixel Data）

像素数据是图像的实际像素值。对于24位图像，每个像素由3个字节表示（B、G、R）。像素数据的存储顺序是从左到右，从下到上（即从图像的左下角开始）。

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240907025601.png)

# BMP的行对齐

在BMP文件中，像素数据的长度和图像尺寸之间的关系需要考虑到行对齐（`padding`）的影响。BMP文件的每一行像素数据必须是4字节对齐的，这意味着每一行的字节数必须是4的倍数。如果一行的字节数不是4的倍数，则需要添加填充字节（`padding bytes`）使其对齐；

比如上图中，图片宽度为666，每个像素4bit即半个字节，则每行长度为333字节，需要填充到336字节；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240907025900.png)

因此像素数据的长度为336 x 666 = 223776字节，对应信息头中的`03 6A 20`；

# 代码示例

以下是一个简单的4位BMP文件的示例代码，展示了如何读取和处理BMP文件：

```cpp
#include <iostream>
#include <fstream>
#include <vector>

#pragma pack(push, 1)
struct BMPFileHeader {
    uint16_t fileType;     // 文件类型，必须是'BM'
    uint32_t fileSize;     // 文件大小
    uint16_t reserved1;    // 保留字段，必须为0
    uint16_t reserved2;    // 保留字段，必须为0
    uint32_t offsetData;   // 像素数据的偏移量
};

struct BMPInfoHeader {
    uint32_t size;         // 信息头大小
    int32_t width;         // 图像宽度
    int32_t height;        // 图像高度
    uint16_t planes;       // 颜色平面数，必须为1
    uint16_t bitCount;     // 每像素位数
    uint32_t compression;  // 压缩方式
    uint32_t sizeImage;    // 图像大小
    int32_t xPelsPerMeter; // 水平分辨率
    int32_t yPelsPerMeter; // 垂直分辨率
    uint32_t clrUsed;      // 使用的颜色数
    uint32_t clrImportant; // 重要颜色数
};

struct RGBQuad {
    uint8_t blue;
    uint8_t green;
    uint8_t red;
    uint8_t reserved;
};
#pragma pack(pop)

int main() {
    std::ifstream file("D:\\dev\\TextureFilter\\pixel_4bit.bmp", std::ios::binary);
    if (!file) {
        std::cerr << "无法打开文件" << std::endl;
        return 1;
    }

    BMPFileHeader fileHeader;
    BMPInfoHeader infoHeader;

    // 读取文件头
    file.read(reinterpret_cast<char*>(&fileHeader), sizeof(fileHeader));
    // 读取信息头
    file.read(reinterpret_cast<char*>(&infoHeader), sizeof(infoHeader));

    // 输出文件头和信息头信息
    std::cout << "文件类型: " << std::hex << fileHeader.fileType << std::dec << std::endl;
    std::cout << "文件大小: " << fileHeader.fileSize << std::endl;
    std::cout << "像素数据偏移量: " << fileHeader.offsetData << std::endl;
    std::cout << "图像宽度: " << infoHeader.width << std::endl;
    std::cout << "图像高度: " << infoHeader.height << std::endl;
    std::cout << "每像素位数: " << infoHeader.bitCount << std::endl;

    // 如果是4位或8位位图，读取调色板
    if (infoHeader.bitCount == 4 || infoHeader.bitCount == 8) {
        int paletteSize = (infoHeader.bitCount == 4) ? 16 : 256;
        std::vector<RGBQuad> palette(paletteSize);
        file.read(reinterpret_cast<char*>(palette.data()), paletteSize * sizeof(RGBQuad));

        // 输出调色板信息
        for (int i = 0; i < paletteSize; ++i) {
            std::cout << "调色板[" << i << "]: ("
                << (int)palette[i].red << ", "
                << (int)palette[i].green << ", "
                << (int)palette[i].blue << ")" << std::endl;
        }
    }

    // 关闭文件
    file.close();
    return 0;
}
```
