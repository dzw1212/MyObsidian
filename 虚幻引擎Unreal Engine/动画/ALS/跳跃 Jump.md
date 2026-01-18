# 跳跃过程分解

从地面开始跳跃：
1. 开始跳跃 **Start**；
2. 跳跃后尚未达到最高点 **StartLoop**；
3. 到达最高点 **Apex**；
4. 下坠中 **FallingLoop**；
5. 落地 **FallingLand**；
6. 落地后恢复 **LandRecovery**；

从高处落下：
4. 下坠中 **FallingLoop**；
5. 落地 **FallingLand**；
6. 落地后恢复 **LandRecovery**；

# 状态机

