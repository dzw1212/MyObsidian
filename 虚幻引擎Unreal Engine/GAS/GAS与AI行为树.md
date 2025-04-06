
[[AI行为树]]

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250309225947.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250309230045.png)

---

# 自定义AIController

添加模块：

![1100](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250310004127.png)

自定义一个`AIController`：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250310012152.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250310012223.png)

`Character`添加`AIController`和`BT`，并运行行为树：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250310010751.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250310012256.png)


# AI逻辑

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250312215303.png)

# 远程攻击与EQS

[[AI行为树#EQS系统]]

远程攻击时，需要考虑攻击路径上有障碍物存在的情况，因此使用EQS来判断最佳攻击位置；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250312224332.png)


添加三个EQS Test：
1. Trace测试，仅筛选，筛选出到Player可见的位置；
2. Trace测试，仅筛选，筛选出自身可见的位置；
3. Distance测试，仅得分，评选出离自身最近的位置；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250312231104.png)

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250312231142.png)

