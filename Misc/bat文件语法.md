`.bat` 文件，也称为批处理文件，是Windows操作系统中用于自动化执行命令序列的脚本文件；以下是一些基本的 `.bat` 文件语法和命令：

1. 变量使用百分号包围，如`%VARIABLE%`；
2. 使用`@`防止命令本身被回显；
3. 通常在脚本开始处使用`@ECHO OFF`关闭自身及之后的回显；

```cpp
//显示消息或者开启/关闭命令的回显
ECHO ON
ECHO hello,world!
ECHO OFF

//注释，不执行
REM 这是一个注释

//设置环境变量
SET VARIABLE=true

//调用另一个bat文件
CALL another.bat

//暂停执行，等待用户按任意键继续
PAUSE

//跳转到对应标签处
GOTO label
:label
ECHO jump to label

//条件判断
IF %VARIABLE%==value ECHO Value is matched

//循环
FOR %%G IN (1 2 3) DO ECHO %%G
```

如果要执行cmd中的命令，直接输入即可：

```cpp
@ECHO OFF
REM 执行 premake5 命令
premake5 vs2019

ECHO Premake 执行完成。
PAUSE
```
