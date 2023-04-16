# C Sharp

`C#`是一种基于C和C++、专门为.NET设计的编程语言；
.NET应用程序使用C#、F#或VB编写，代码编译为跨语言的`公共中间语言CIL`，编译后的代码存储在.dll文件或者.exe程序的程序集文件中，当.NET程序运行时，公共语言运行库CLR获取程序集并使用`实施编译器JIT`将其转换为能够在计算机的特定体系结构上运行的计算机代码；

![CSharp与NET|380](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230416020532.png)


# .NET Framework

`.NET Framework`是用于在Windows上生成和运行应用程序的软件开发框架，为`.NET平台`的一部分；
不同的.NET Framework版本可以并行安装；

## 体系结构

.NET框架最主要的两个组件分别是`公共语言运行库CLR`和`基类库BCL`；
- 公共语言运行库CLR：负责管理代码的执行；
- 基类库BCL：提供丰富的类库；

## 现状

由于.NET框架与Windows平台深度绑定，不支持跨平台，因此热度逐渐降低；
目前.NET框架已经安装在数以亿计的计算机上，因此更新频率很低；

# .NET

`.NET`是一个开发人员平台，由工具、编程语言，以及各种库组成；
.NET有多种实现：
1. `.NET Framework`：.NET最原始的实现方式，支持在Windows上运行网站、服务、桌面应用等；
2. `.NET Core`：是.NET的跨平台实现，简称为`.NET`；
3. `Xamarin/Mono`：是在所有主要的移动操作系统上运行应用的.NET实现；

## 与Framework的区别

.NET与.NET Framework共享许多相同的组件，可以共享代码；
它们之间的区别有：
1. .NET跨平台，.NET Framework只能在Windows上运行；
2. .NET开源并接收社区贡献，.NET Framework不接受直接的贡献；
3. .NET更新快，包含更创新的代码；
4. .NET独自交付，.NET Framework由Windows负责更新；

# .NET Standard

`.NET Standard`是跨.NET实现的通用的API规范，允许相同的代码和库在不同的.NET实现上运行；