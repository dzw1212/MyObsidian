官方介绍：*https://www.unrealengine.com/zh-CN/blog/cropout-casual-rts-game-sample-project* 

俯视角休闲RTS游戏，需要管理资源进行采集与建造，最终以造出虚幻标志雕像作为胜利条件（奇观胜利）；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241019153204.png)



相关视频：
	*https://www.bilibili.com/video/BV18p4y1V7yF/*
	*https://www.bilibili.com/video/BV11K421Y7jU/*

技术特点：
	自定义视角；
	程序化生成内容；
	数据保存与载入；
	Nanite；
	Lumen；
	[[Common UI]]；
	[[增强输入 EnhancedInput]]；
	几何体脚本；
	多平台打包项目；

# 程序化生成地形与Actor

岛屿生成器插件；

岛屿生成步骤：
	Reset动态网格体组件，Clear所有Spawn Points；
	在一定范围内，随机位置生成随机半径的锥体Cone，并将这些Cone添加到一个Box中（可以看作一种容器）；
	将网格体固化Soldify，通过体素化将相邻的锥体融为一体，并将锥体网格体进行平顺化Smoothing与细分Tessellation；
	平面切割并压平顶部，使锥体形成地面；
	发送“岛屿生成完毕”的事件；
	给岛屿设置随机颜色；

游戏初始化步骤：
	接收到“岛屿生成完毕”的事件后，检查是否有存档：
		如果有，只从Game Instance中加载数据；
		如果没有，开始随机生成；
	随机生成：
		1. 生成一个Town Hall；

# 自动化保存

