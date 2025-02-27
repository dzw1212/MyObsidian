# Beckman NDF

$$
D(h)=\frac{e^{-\frac{\tan^2{\theta_h}}{\alpha^2}}}{\pi \alpha^2\cos^4{\theta_h}}
$$
$\alpha$ ：粗糙程度，越小则越接近镜面；
$\theta_h$ ：半程向量与法线之间的夹角；

![NDF|400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230915001838.png)


![NDF|320](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230915001940.png)

# GGX NDF

也叫做`Trowbridge-Reitz NDF`；

## 特点 - 长尾

相比`Beckman NDF`，`GGX NDF`的衰减速度更小，对应的高光的变化更自然、高光的范围也更大；

![NDF|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230915010916.png)

![NDF|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230915011158.png)

# GTR NDF

即`General Trowbridge-Reitz NDF`，由迪士尼提出，可以通过系数 $\gamma$ 来控制拖尾的大小；
当 $\gamma = 2$ 时，`GTR`就是`TR`也就是`GGX`分布；
当 $\gamma=\infty$ 时，`GTR`就是`Beckman`分布；

![NDF|500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230917141110.png)

