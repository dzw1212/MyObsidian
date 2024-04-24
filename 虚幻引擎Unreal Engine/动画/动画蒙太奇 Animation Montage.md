创建收枪和掏枪的动画蒙太奇；

# 创建蒙太奇

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317224915.png)

一般刚创建好的蒙太奇用的是`DefaultSlot`，可以点击`Slot manage`查看有哪些可用的插槽；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317230151.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317230207.png)

此处选择`FullBody`和`UpperBody`两个插槽，分别对应全身动画（人物不动时）和半身动画（人物移动时）；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317230525.png)

# 播放蒙太奇

播放蒙太奇需要`Slot`，对于持枪/收枪的蒙太奇我们需要`FullBody`和`UpperBody`两个`Slot`，为此需要修改动画蓝图；

`Cache FullBody Pose`中，使用`FullBody Slot`可以替换掉全身的动画；
`Cache UpperBody Pose`中，使用一个`Layered Blend per Bone`来分割上下半身，使用`UpperBody Slot`可以替换掉上半身的动画；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317232749.png)

`Layered Blend per Bone`中选择一个骨骼来分割上下半身；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317234656.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240317233647.png)


根据是否在移动来判断使用`FullBody Slot`还是`UpperBody Slot`：

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240318000440.png)

