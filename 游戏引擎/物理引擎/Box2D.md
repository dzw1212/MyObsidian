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

# 形状

## 圆形

```c++
b2CircleShape circle;
circle.m_p.Set(2.f, 3.f); //圆心
circle.m_radius(0.5f); //半径
```

## 多边形

顶点顺序：逆时针缠绕；
支持的最大顶点数：`b2_maxPolygonVertices = 8`；
```c++
b2Vec2 vertices[3];
vertices[0].Set(0.0f, 0.0f);
vertices[1].Set(1.0f, 0.0f);
vertices[2].Set(0.0f, 1.0f);

int32 count = 3;

b2PolygonShape polygon;
polygon.Set(vertices, count);
```

```c++
void SetAsBox(float hx, float hy);
void SetAsBox(float hx, float hy, const b2Vec2& center, float angle);
```

## 边缘形状

线段可以和圆形、多边形碰撞，但线段之间不能碰撞，碰撞要求至少其中一方有体积；
```c++
b2Vec2 v1(0.0f, 0.0f);
b2Vec2 v2(1.0f, 0.0f);

b2EdgeShape edge;
edge.SetTwoSided(v1, v2);
```

### 幽灵碰撞与幽灵顶点

假设地面由多个线段连接而成，当一个多边形沿着地面滑动时，就会与线段的内部顶点产生碰撞，称为`幽灵碰撞`；

![ghostCollision|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230328000750.png)


Box2D提供了一种叫做`幽灵顶点`的机制，用来避免幽灵碰撞；
```c++
b2Vec2 v0(1.7f, 0.0f);
b2Vec2 v1(1.0f, 0.25f);
b2Vec2 v2(0.0f, 0.0f);
b2Vec2 v3(-1.7f, 0.4f);

b2EdgeShape edge;
edge.SetOneSided(v0, v1, v2, v3); //顺序遵循逆时针缠绕
```

## 连锁形状

连锁形状提供了一种有效的方法来连接多个线段，能够自动消除幽灵碰撞，并提供单边碰撞；
```c++
b2Vec2 vs[4];
vs[0].Set(1.7f, 0.0f);
vs[1].Set(1.0f, 0.25f);
vs[2].Set(0.0f, 0.0f);
vs[3].Set(-1.7f, 0.4f);

b2ChainShape chain;
chain.CreateLoop(vs, 4);
```

```c++
//连接多个连锁形状
b2ChainShape::CreateChain(const b2Vec2* vertices,
						  int32 count,
						  const b2Vec2& prevVertex,
						  const b2Vec2& nextVertex）
```

```c++
//通过索引访问子线段
for (int32 i = 0; i < chain.GetChildCount(); ++i)
{
	b2EdgeShape edge;
	chain.GetChildEdge(&edge, i);
}
```

# 几何查询

## 形状 + 点

判断形状和点是否重叠；
如果是边缘形状或连锁形状，则总是返回false；
```c++
b2Transform transform;
transform.SetIdentity();
b2Vec2 point(5.0f, 2.0f);
bool hit = shape->TestPoint(transform, point);
```

## 形状 + RayCast

```c++
b2Transfrom transform;
transform.SetIdentity();

b2RayCastInput input;
input.p1.Set(0.0f, 0.0f);
input.p2.Set(1.0f, 0.0f);
input.maxFraction = 1.0f;

int32 childIndex = 0;

b2RayCastOutput output;

bool hit = shape->RayCast(&output, input, transform, childIndex);
if (hit)
{
	b2Vec2 hitPoint = input.p1 + output.fraction * (input.p2 - input.p1)
}
```

# 成对函数

## 重叠

测试两个形状之间的重叠情况；
```c++
b2Transform xfA = ..., xfB = ...;
bool overlap = b2TestOverlap(shapeA, indexA, shapeB, indexB, xfA, xfB);
```

## 接触面

![manifold|300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230328034056.png)

存储在`b2Manifold`中的数据被优化为内部使用，如果需要查看这些数据，最好使用`b2WorldManifold`来生成接触点和法线的世界坐标；

```c++
b2WorldManifold worldManifold;
worldManifold.Initialize(&manifold, transformA, shapeA.m_radius, transformB, shapeB.m_radius);
for (int32 i = 0; i < manifold.pointCount; ++i)
{
	b2Vec2 point = worldManifold.point[i];
}
```

在移动过程中接触点可能会改变，因此需要查询接触点的状态；
```c++
b2PointState state1[2], state2[2];
b2GetPointStates(state1, state2, &manifold1, &manifold2);
if (state1[0] == b2_removeState) {}
```

## 距离

`b2Distance`可以计算两个形状之间的距离；

## 撞击时间

`b2TimeOfImpact`用来确定两个移动形状发生碰撞的时间，以防止隧道效应；该函数需要两个形状和两个`b2Sweep`结构，该结构中定义了形状的初始和最终变换；

# 动态模块

## 刚体的分类

- `b2_staticBody`：静态体，不会移动，具有无限的质量，不与其他刚体发生碰撞；
- `b2_kinematicBody`：运动体，根据速度移动，具有无限的质量，不与其他刚体发生碰撞；
- `b2_dynamicBody`：动态体，根据受力移动，总是具有非零的有限的质量，能够与所有类型的刚体碰撞；

## 刚体的定义

`b2BodyDef`结构体含有创建和初始化刚体所需的数据，可以使用同一个定义来创建多个刚体；

### 类型

类型分为三种：静态体、运动体和动态体；
应该在创建刚体时指定类型，否则后续修改类型的开销很高；
```c++
b2BodyDef bodyDef;
bodyDef.type = b2_dynamicBody;
```

### 角度和位置

在创建时指定tranform，比先创建再移动的性能要好很多；

一个刚体有两个重要的点：
- 原点：夹具和关节都是基于原点来连接刚体；
- 质心：质心有附加形状的质量分布所决定，或者由`b2MassData`显式指定；

创建刚体的时候一般不知道质心，但会指定原点，如果后续刚体的质量属性发生变化，质心的位置会变化，但原点一般不变；
```c++
b2BodyDef bodyDef;
bodyDef.position.Set(0.0f, 2.0f); //设置原点
bodyDef.angle = 0.25f * b2_pi;
```

### 阻尼

阻尼（Damping）用来减少刚体的速度，是一种广义上的阻力；
阻尼与摩擦不同，摩擦只有在接触的时候才会产生，阻尼不能代替摩擦，两者是共存的；

阻尼的大小应该在0到无穷大之间，推荐使用0到0.1之间的值；
```c++
b2BodyDef bodyDef;
bodyDef.linearDamping = 0.0f; //线性阻尼
bodyDef.AngularDamping = 0.01f; //角阻尼
```

### 重力因子

可以使用重力因子（Gravity Scale）单独设置某个刚体的重力；
```c++
b2BodyDef bodyDef;
bodyDef.gravityScale = 0.0f;
```

### 休眠

每时每刻模拟World下的所有刚体的性能开销是非常大的，如果一个刚体允许休眠，其会在静态时进入休眠状态，以减小模拟的性能开销；
当一个休眠的刚体与一个活动的刚体碰撞时，会被唤醒；
```c++
b2BodyDef bodyDef;
bodyDef.allowSleep = true;
bodyDef.awake = true;
```

### 固定旋转

相当于禁止了旋转；
```c++
b2BodyDef bodyDef;
bodyDef.fixedRotation = true;
```

### 子弹

为了性能考虑动态刚体一般不启用连续碰撞检测（CCD），这可能会导致隧道效应；
Box2D中快速移动的物体可以被标记为子弹，子弹将对静态和动态物体启用CCD；
```c++
b2BodyDef bodyDef;
bodyDef.bullet = true;
```

### 激活

类似于休眠状态，一个刚体可以被标记为非激活状态，这意味着这个刚体不参与碰撞检测、射线投射等；
刚体可以被重新激活，但是激活一个刚体的开销几乎等同于重新创建一个刚体；
```c++
b2BodyDef bodyDef;
bodyDef.active = true;
```

### 用户数据

```c++
b2BodyDef bodyDef;
bodyDef.userData.pointer = reinterpret_cast<uintptr_t>(&myActor);
```

## 刚体的设置

### 质量属性

一个体有质量（标量）、质心（向量）和转动惯量（标量）；
对于静态体，质量和转动惯量被设置为零；当一个刚体有固定的旋转时，其转动惯量为零；

通常情况下，当刚体附加到一个夹具上时，质量属性会自动建立；也可以在运行时显式设置一个质量属性；
```c++
void b2Body::SetMassData(const b2MassData* data);

void b2Body::ResetMassData(); //恢复到由夹具决定的质量属性

float b2Body::GetMass() const;

float b2Body::GetInertia(); //惯性

const b2Vec2& b2Body::GetLocalCenter() const;

void b2Body::GetMassData(b2MassData* data) const;
```

### 状态信息

```c++
void b2Body::SetType(b2BodyType type);
b2BodyType b2Body::GetType();

void b2Body::SetBullet(bool flag;
bool b2Body::IsBullet() const;

void b2Body::SetSleepingAllowed(bool flag);
bool b2Body::IsSleepingAllowed() const;

void b2Body::SetAwake(bool flag);
bool b2Body::IsAwake() const;

void b2Body::SetEnabled(bool flag);
bool b2Body::IsEnabled() const;

void b2Body::SetFixedRotation(bool flag);
bool b2Body::IsFixedRotation() const;
```

### 位置和速度

```c++
bool b2Body::SetTransform(const b2Vec2& position, float angle);
const b2Transform& b2Body::GetTransform() const;

const b2Vec2& b2Body::GetPosition() const;

float b2Body::GetAngle() const;

const b2Vec2& b2Body::GetWorldCenter() const;
const b2Vec2& b2Body::GetLocalCenter() const;
```

### 力与脉冲

```c++
void b2Body::ApplyForce(const b2Vec2& force, const b2Vec2& point;

void b2Body::ApplyTorque(float torque); //扭力

void b2Body::ApplyLinearImpulse(const b2Vec2& impulse, const b2Vec2& point);

void b2Body::ApplyAngularImpulse(float impulse);
```

施加力与脉冲会唤醒休眠的刚体，如果希望既有力刚体又能保持休眠，可以参考如下做法：
```c++
if (myBody->IsAwake() == true)
{
	myBody->ApplyForce(myForce, myPoint);
}
```

### 坐标转换

```c++
b2Vec2 b2Body::GetWorldPoint(const b2Vec2& localPoint);

b2Vec2 b2Body::GetWorldVector(const b2Vec2& localVector);

b2Vec2 b2Body::GetLocalPoint(const b2Vec2& worldPoint);

b2Vec2 b2Body::GetLocalVector(const b2Vec2& worldVector);
```

### 遍历夹具

```c++
if（b2Fixture* f = body->GetFixtureList(); f; f = f->GetNext()
{
	MyFixtureData* data = (MyFixtureData*)f->GetUserData();
}
```

# 夹具

一个刚体可以有零个、一个或者多个夹具，带有夹具的刚体又称为复合体（Compound Body）；

## 创建与销毁

```c++
b2Body* myBody;
b2FixtureDef fixtureDef；
fixtureDef.shape = &myShape；
fixtureDef.density = 1.0f；
b2Fixture* myFixture = myBody->CreateFixture(&fixtureDef);

myBody->DestroyFixture(myFixture);
myFixture = nullptr;
```

## 密度

夹具的密度参与母体质量属性的计算，密度可以是0或某个正数，建议所有的夹具使用类似的密度，以保证结构的稳定性；

当设置密度时，刚体的质量属性不会调整，需要调用`ResetMassData`；
```c++
b2Fixture* fixture;
fixture->SetDensity(5.0f);

body->ResetMassData();
```

## 摩擦

Box2D支持静态和动态摩擦，但对两者使用相同的参数；
摩擦参数通常设置在0到1之间，但可以是任何非负值，摩擦值为0会关闭摩擦，值为1会使摩擦力变强；
计算两个形状之间的摩擦力需要结合两个夹具的摩擦参数、使用几何平均法计算的，只要其中一方的摩擦力为0，则两者之间的摩擦力也为0；

```c++
b2Fixture* fixtureA;
b2Fixture* fixtureB;
float friction;

friction = sqrtf(fixtureA->friction * fixtureB->friction);
```

可以使用`b2Contact::SetFriction`覆盖默认的混合摩擦（通常在`b2ContactListener`的回调中完成）；

## 恢复力

恢复力（Resititution）用来使物理反弹，通常被设置为0到1之间，0意味着非弹性碰撞，1意味着完全弹性碰撞；

混合恢复力的计算：
```c++
b2Fixture* fixtureA;
b2Fixture* fixtureB;

float restitution;

restitution = b2Max(fixtureA->restitution, fixtureB->restitution);
```

可以使用`b2Contact::SetRestitution`覆盖默认的混合恢复力（通常在`b2ContactListener`的回调中完成）；

## 碰撞过滤

某些情况下，有些夹具之间需要没有碰撞，这就需要过滤掉这些夹具，Box2D中是通过`屏蔽位`实现的；
```c++
b2FixtureDef playerFixtureDef, monsterFixtureDef;
playerFixtureDef.filter.categoryBits = 0x0002;
monsterFixtureDef.filter.categoryBits = 0x004;
playerFixtureDef.filter.maskBits = 0x0004;
monsterFixtureDef.filter.maskBits = 0x0002;

uint16 catA = fixtureA.filter.categoryBits;
uint16 maskA = fixtureA.filter.maskBits;
uint16 catB = fixtureB.filter.categoryBits;
uint16 maskB = fixtureB.filter.maskBits;

if ((catA & maskB) != 0 && (catB & maskA) != 0)
{
	//怪物之间不碰撞，玩家之间不碰撞，怪物与玩家之间碰撞
}
```

还可以分配`碰撞组`，具有相同组索引的夹具总是碰撞（正数）或总是不碰撞（负数）；
```c++
//总是碰撞
fixture1Def.filter.groupIndex = 2;
fixture2Def.filter.groupIndex = 2;

//总是不碰撞
fixture3Def.filter.groupIndex = -8;
fixture4Def.filter.groupIndex = -8;
```

如果同时使用了屏蔽位和碰撞组，则==碰撞组的优先级高于屏蔽位==；

此外，Box2D中还有一些默认的碰撞过滤规则：
1. 静态体的夹具只能和动态体碰撞；
2. 运动体的夹具只能和动态体碰撞；
3. 同一母体上的夹具不会互相碰撞；
4. 可以选择性地启用/禁用由关节连接的刚体上夹具之间的碰撞；

可以使用`b2Fixture::GetFilterData`和`b2Fixture::SetFilterData`来修改夹具的`b2Filter`结构体，但修改过后直到下一次step才会生效；

## 传感器

传感器是一个能够检测到碰撞而不产生反应的装置，任何夹具都可以被标记为传感器，传感器只有在至少其中一方是动态体的时候才生效；

传感器不产生接触点，有两种方法获取传感器的状态：
```c++
b2Contact::IsTouching

b2ContactListener::BeginContact
b2ContactListener::EndContact
```

# 关节

关节使用`b2JointDef`进行定义，所有关节都连接在两个不同的刚体之间，允许将关节连接到静态体上，但不会有任何效果；

关节连接的体之间默认不会碰撞，可以设置`collideConnected`以允许连接体之间的碰撞；

## 创建与销毁

```c++
b2World* myWorld;

b2RevoluteJointDef jointDef;

jointDef.bodyA = myBodyA;
jointDef.bodyB = myBodyB;

jointDef.anchorPoint = myBodyA->GetCenterPosition();

b2RevoluteJoint* joint = (b2RevoluteJoint*)myWorld->CreateJoint(&jointDef);

// ... do stuff ...

myWorld->DestroyJoint(joint);
joint = nullptr;
```

## 获取数据

```c++
b2Body* b2Joint::GetBodyA();
b2Body* b2Joint::GetBodyB();

b2Vec2 b2Joint::GetAnchorA();
b2Vec2 b2Joint::GetAnchorB();

void* b2Joint::GetUserData();

b2Vec2 b2Joint::GetReactionForce();
float b2Joint::GetReactionTorque();
```

## 分类

### 距离关节

两个物体之间的锚点的距离是恒定的；