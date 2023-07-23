# 字体库

## stb_font

*https://nothings.org/stb/font/*

`stb_font`是一个开源的 C 语言库，用于在计算机图形中渲染位图字体。这个库是由 Sean Barrett 创建的，他是一位知名的游戏开发者和开源贡献者，stb_font 是他的 stb 库系列的一部分，这个系列包括各种用于游戏开发和其他图形应用的实用工具；
在使用 `stb_font`时，你可以选择一个字体，然后生成一个包含该字体所有字符的位图。然后，你可以使用这个位图在屏幕上渲染文本；

以下代码定义了一个 Consolas Bold 字体，字体大小为 16，字符集为 US ASCII。然后，它定义了一个位图和一个字符数组，用于存储字体的所有字符。 
```c
#define STB_FONT_consolas_bold_16_usascii_BITMAP_WIDTH 128
#define STB_FONT_consolas_bold_16_usascii_BITMAP_HEIGHT 48
#define STB_FONT_consolas_bold_16_usascii_BITMAP_HEIGHT_POW2 64
#define STB_FONT_consolas_bold_16_usascii_FIRST_CHAR 32
#define STB_FONT_consolas_bold_16_usascii_NUM_CHARS 95
#define STB_FONT_consolas_bold_16_usascii_LINE_SPACING 20
extern unsigned char STB_FONT_consolas_bold_16_usascii_BITMAP[STB_FONT_consolas_bold_16_usascii_BITMAP_WIDTH*STB_FONT_consolas_bold_16_usascii_BITMAP_HEIGHT_POW2];
extern stb_fontchar STB_FONT_consolas_bold_16_usascii[STB_FONT_consolas_bold_16_usascii_NUM_CHARS];
```

与`FreeType`的区别：
1. `stb_font`是一个轻量级的库，主要用于生成和渲染位图字体，功能相对较少，但使用起来非常简单；
2. `FreeType`是一个更复杂的库，它提供了对各种字体格式（包括`TrueType`和`OpenType`）的全面支持，不仅用于渲染矢量字体，也可以用于处理字体的各种复杂特性，如连字和变体；

 
## FreeType

