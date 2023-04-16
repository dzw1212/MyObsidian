*http://nilssondev.com/mono-guide/book/introduction/about-mono.html#about-mono*

`Mono`是微软的`.NET Framework`的一个开源实现，最初的目标是实现.NET Framework的跨平台，这部分工作已由`.NET Core`接任，目前Mono一般用于游戏引擎中实现脚本系统；

# 版本

Mono分为`经典Mono`和`.NET Core Mono`，经典Mono只支持到`C#7`或者说`.NET Framework 4.7.2`，.NET Core Mono则支持最新版本的C#；
经典Mono没有被集成到.NET中，但其更简单，也更适合游戏引擎；

## 为什么不用最新的Mono

最新的.NET Core Mono**不支持汇编重载**；
游戏引擎对脚本系统的一个要求是——能够在修改脚本后热重载该脚本，而不同重新启动整个编辑器；为了完成这个要求，需要先卸载旧的程序集，再加载新的程序集；
目前，.NET Core Mono尚不支持重载程序集，而经典Mono可以，因此选择经典Mono来构建脚本系统；

# 构建

## Windows平台

### 构建Mono运行时库

下载源代码，使用`msvc/mono.sln`进行构建（建议配置`Debug x64`和`Release x64`）；
*https://github.com/mono/mono*

### 复制.NET库

所需要的库文件都应该放置在`msvc/build/sgen/{platform}/`目录下，所需要的文件有：
- `lib`目录下：
	eglib.lib
	libgcmonosgen.lib
	libmini-sgen.lib
	libmonoruntime-sgen.lib
	libmono-static-gen.lib
	libmonoutils.lib
	mono-2.0-sgen.lib
	MonoPosixHelper.lib
- `bin`目录下：
	mono-2.0-sgen.dll
	MonoPosixHelper.dll

下载并安装Mono：*https://www.mono-project.com/download/stable/*

