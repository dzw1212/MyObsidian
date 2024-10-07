`<random>` 头文件提供了多种随机数分布（distributions），这些分布可以用于生成符合特定概率分布的随机数。这些分布通常与随机数引擎（如 [[std-mt19937]]）一起使用，以生成具有特定统计特性的随机数；

# 均匀分布 Uniform Distribution

- **`std::uniform_int_distribution`**：生成一个指定范围内的均匀分布的整数。常用于需要随机整数的场景，如随机索引、模拟投掷骰子等。

- **`std::uniform_real_distribution`**：生成一个指定范围内的均匀分布的浮点数。常用于需要连续均匀数据的模拟，如物理模拟中的随机事件。

# 正态分布 Normal Distribution

- **`std::normal_distribution`**：生成符合正态分布（高斯分布）的随机数。常用于需要模拟自然现象或社会现象中的误差或变异，如测量误差、股价变动等。

# 伯努利分布 Bernoulli Distribution

- **`std::bernoulli_distribution`**：生成布尔值（true 或 false），模拟单次伯努利试验的结果。常用于模拟具有两种可能结果的随机过程，如抛硬币。

# 二项分布 Binomial Distribution

- **`std::binomial_distribution`**：生成符合二项分布的随机数，即在固定次数的独立伯努利试验中成功的次数。常用于模拟重复试验的成功次数，如抛硬币多次的正面次数。

# 泊松分布 Poisson Distribution

- **`std::poisson_distribution`**：生成符合泊松分布的随机数，模拟在固定时间或空间内发生的独立事件的数量。常用于模拟电话呼入、网站点击等随机事件的数量。

# 几何分布 Geometric Distribution

- **`std::geometric_distribution`**：生成符合几何分布的随机数，即在第一次成功之前进行的伯努利试验次数。常用于模拟直到第一次成功所需的试验次数，如连续抛硬币直到出现正面。

# 指数分布 Exponential Distribution

- **`std::exponential_distribution`**：生成符合指数分布的随机数，模拟两个连续事件之间的时间间隔。常用于模拟无记忆过程中的等待时间，如公交车到站的时间间隔。

---

```cpp
#include <iostream>
#include <random>

int main() {
    std::random_device rd;  // 随机数种子
    std::mt19937 gen(rd()); // 标准 mersenne_twister_engine
    std::normal_distribution<> d(5.0, 2.0); // 均值为5，标准差为2的正态分布

    for (int n = 0; n < 10; ++n) {
        std::cout << d(gen) << '\n'; // 生成并打印随机数
    }

    return 0;
}
```