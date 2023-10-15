# 基础操作

鼠标悬停后，可以按F1打开在线帮助文档；

唤出视图旋转选择菜单：`~`键：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016000713.png)

显示统计信息：
	视图叠加层->统计信息；

![120](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016003450.png)

开关插件：编辑->偏好设置->插件；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016015313.png)



选取+g，抓取模式，后续x/y/z分别限制在三个轴；

shift+d，复制并直接进入抓取模式；

修改游标的位置：
	1. 目视定位：shift+鼠标右键；
	2. 吸附定位：进入编辑模式，选中物体顶点，网格->吸附->游标到选中项；

修改物体原点的位置：
	1. 先将游标移动到目的位置，再右键物体->设置原点->原点到3D游标；

右下角有小三角形的图标，表示可以二次操作，长按弹出二次操作选项；

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015203350.png)

可视化面的法向：
	视图叠加层点击显示法线；
![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015211129.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015212544.png)

X-Ray透视模式：
![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015220841.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015220908.png)

选中[[Blender#循环边 Loop]]或[[Blender#并排边 Ring]]：
循环边快捷键：Alt+鼠标左键；
并排边快捷键：Alt+Shift+鼠标左键；
BTW：循环面/并排面的快捷键相同，在面选择模式下即可；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015230049.png)

间隔式弃选：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015235431.png)

衰减编辑：拖拽过程中滑动滚轮，可以修改衰减半径；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016012220.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016012621.png)


![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016012311.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016013244.png)

# 面的基本操作

1. 细分；
![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015203656.png)

2. 挤出；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015203805.png)

3. 内插（外插，同时选择内外侧）；

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015203906.png)

4. 尖分，中心点连接其他顶点；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015204143.png)

5. 三角化；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015204402.png)
6. 融并；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015204535.png)

7. 切割，按k进入，切割后按回车确认，按Esc取消（按shift吸附中点，按A约束角度，按C穿透切割）；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015204944.png)

8. 填充，快捷键F，选择一圈的线作为边，进行面的填充（默认使用三角形）；如果选择栅格填充，则是使用四边形进行填充，此时要求边的个数为偶数；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015220246.png)

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015220556.png)

# 边的基本操作

1. 细分；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015222056.png)

2. 滑移，边的两个端点会沿着轨道进行移动；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015222456.png)

3. 融并，不同于常规的删除操作会删除临接的面，融并会将临界的面合并为一个面；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015223411.png)

4. 倒角，如果一条边过于锐利，可以使用倒角来抹平；可以修改倒角的段数或形状等；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015223625.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015223853.png)

5. 环切，沿着一圈切一条；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015225332.png)

6. 循环边滑移，快捷键 G + G；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015231535.png)

7. 循环边融并，快捷键 Ctrl + X；

8. 循环边法向收缩或扩张；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015231740.png)

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015231806.png)

9. 桥接循环边，连接属于同一物体的两条循环边（边数可以不同）；常用作连接两个几何体或者打洞；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015233425.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015234607.png)

# 顶点的基本操作

1. 缩放；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016001345.png)

2. 旋转；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016001450.png)

3. 滑移，顶点沿着临边作为轨道滑动；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016001619.png)

4. 连接/填充，快捷键F；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016002422.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016002748.png)

5. 连接并分割面，快捷键J，或者直接使用切割工具K；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016003201.png)

6. 合并，用于合并相距很近的顶点或者重叠的顶点；对于重叠的顶点，肉眼不好区分，可以选择全选所有顶点->右键->合并顶点->按距离；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016004442.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016004503.png)

或者打开自动合并选项；

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016004717.png)

7. 倒角，快捷键Ctrl+Shift+B；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016005000.png)

2邻边倒角：

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016005415.png)

3邻边倒角：

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016005518.png)

4邻边倒角：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016005646.png)


8. 挤出，快捷键E，得到一条边；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016010435.png)

9. 拓展点，快捷键Alt+D，如果两个顶点之间相距太远可以在中途拓展一个新顶点；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016011504.png)

# 基础概念

### 循环边 Loop

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015225722.png)
### 并排边 Ring

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015230245.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231015231330.png)

# 常用插件

## LoopTools

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231016015413.png)

