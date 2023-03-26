*https://box2d.org/documentation/

Box2D是一个用于游戏的2D刚体模拟库。程序员可以在他们的游戏中使用它来使物体以逼真的方式移动，并使游戏世界更加互动。从游戏引擎的角度来看，物理引擎只是一个程序化动画的系统；

Box2D是用可移植的C++编写的，引擎中定义的大多数类型都是以b2开头的；

# 概述

## 术语

|概念|解释|
|:-|:-|
|`shape` 形状|比如圆形，多边形|
|`rigid body` 刚体|规定其强度无限大|
|`fixture` 夹具|用以将形状绑定到刚体上，并增加如密度、摩擦力等材质属性|
|`constraint` 约束|一种物理连接，用来去除刚体的自由度，比如将一个刚体固定在墙上，其只能绕着z轴转动，丢失了x和y两个轴的自由度|
|`contact constraint` 接触约束|一种特殊的约束，用来防止刚体的穿透，模拟摩擦和弹性（由Box2D自动创建）|
|`joint` 关节|一种用于将两个或多个刚体固定在一起的约束|
|`joint limit` 关节限位|用来限制关节的运动范围|
|`joint motor` 关节马达|用来根据关节的自由度驱动连接体的运动|
|`world` 世界|刚体、夹具、约束的集合|
|`solver` 解算器|用于推进时间，解决关节约束问题|
|`continuous collision` 连续碰撞|由于解算器使用离散的timestep推进刚体的运动，这可能会导致高速运动且连续碰撞时的隧穿效应`Tunneling Effect`（Box2D包含专门的算法来解决隧穿效应——第一碰撞时间+子步进解算器）|
## 模块组成

通用模块 + 碰撞模块 + 动力学模块；

- Common：包含用于内存分配、数学运算和设置的代码；
- Collision：包含形状、广相碰撞和碰撞查询；
- Dynamics：包含世界模拟、刚体、夹具、关节等；

## 创建与销毁

Box2D中的对象使用工厂模式进行管理，需要使用对应的接口来创建和销毁，禁止用其他的方式以防止内存泄漏问题；

比如：
```c++
b2Body* b2World::CreateBody(const b2BodyDef* def)
void b2World::DestroyBody(b2Body* body)

b2Fixture* b2Body::CreateFixture(const b2FixtureDef* def)
b2Fixture* b2Body::CreateFixture(const b2Shape* shape, float density)
void b2Body::DestroyFixture(b2Fixture* fixture)
```

# 示例

## 创建World

每个Box2D程序都以创建一个`b2World`对象开始，它管理内存、对象和模拟的物理中心；

```c++
b2Vec2 gravity(0.0f, -10.0f); //定义重力矢量
b2World world(gravity); //创建world
```

## 创建Ground Box

```c++
//Step1. 定义一个刚体的物理属性

b2BodyDef groundBodyDef;
groundBodyDef.position.Set(0.0f, -10.0f);

//Step2. 使用world来创建刚体

b2Body* groundBody = world.CreateBody(&groundBodyDef); 

//Step3. 创建一个地面多边形，并将形状设为盒子

b2PolygonShape groundBox;
groundBox.SetAsBox(50.0f, 10.0f); //宽100个单位，高20个单位

//Step4. 创建夹具，关联形状与刚体

groundBody->CreateFixture(&groundBox, 0.0f); 
//可以直接将形状传给刚体而不用自定义夹具属性

```

## 创建Dynamic Body

```c++
b2BodyDef bodyDef;
bodyDef.type = b2_dynamicBody; //默认为静态，需要手动设置类型为动态
bodyDef.position.Set(0.0f, 4.0f);
b2Body* body = world.CreateBody(&bodyDef);

//创建多边形
b2PolygonShape dynamicBox;
dynamicBox.SetAsBox(1.0f, 1.0f);

//自定义夹具
b2FixtureDef fixtureDef;
fixtureDef.shape = &dynamicBox;
fixtureDef.density = 1.0f; //默认密度为0，但动态刚体的密度绝不应该是0
fixtureDef.friction = 0.3f;
body->CreateFixture(&fixtureDef);
```

## 模拟World

### 积分器与时间步长

Box2D使用一种叫做**积分器**的算法，可以在离散的时间点上模拟物理学方程，因此需要给Box2D设置一个时间步长；
一般来说，物理引擎的步长至少是1/60秒，这个步长应该是稳定的，可变的步长会产生可变的结果，使得调试变得苦难，因此不应该使用可变的游戏帧率作为步长；

### 约束解算器与迭代

除了积分器外，Box2D还使用了**约束解算器**，约束解算器一次解决模拟中的所有约束，对于单个约束的情况可以完美解决，但对于多个约束的情况，当我们解决一个约束时，会打乱其他约束的条件，因此需要在所有约束条件上迭代若干次才能得到好的解决方案；
约束解算器有两个阶段：
1. 速度阶段：计算刚体正确移动所需要的脉冲；
2. 位置阶段：调整刚体的位置，减少重叠与关节脱离；
每个阶段都有自己的迭代次数（速度阶段建议为8，位置阶段建议为3），如果误差小到可以接受，也可以提前退出迭代；

### 循环

模拟的循环可以与游戏循环合并，即在每个游戏循环处调用一次`b2World::Step`，通常只要调用一次即可；

```c++
float timeStep = 1.0f / 60.0f;

int32 velocityIterations = 6;
int32 positionIterations = 2;

for (int32 i = 0; i < 60; ++i)
{
	world.Step(timeStep, velocityIterations, positionIterations);
	b2Vec2 position = body->GetPosition()
	float angle = body->GetAngle()
}
```

## 内存释放

当一个`b2World`离开作用域或被删除时，其下的所有刚体、夹具、关节等都会被释放；但是相关的指针需要手动设为`nullptr`；