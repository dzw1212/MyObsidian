# 动画资源处理

- 所有动画启用`Root Motion`与`Force Root Lock`；
- 所有`Start`与`Stop`、`Pivot`动画`Curve Compression Settings`改为`UniformIndexable`，并生成`Distance`曲线；
- 所有`Pivot`动画在末尾添加`PivotExit`动画状态通知；
- 所有`TurnInPlace`动画添加`root_rotation_Z`与`IsTurning`曲线；
# 触发蹲伏

Ctrl按下后进入蹲伏状态，抬起后回到正常状态；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118005406.png)

# 角色Crouch相关属性

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118011413.png)

# 进入Crouch与离开Crouch

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118214714.png)

# 状态机

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118214906.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118214842.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118214957.png)

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118215134.png)

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118215222.png)

# Crouch的不同点

## 最大速度限制

这个配置不同于Walk和Jog，这是单独的；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260118190903.png)

