策略（`Strategy`）设计模式是一种行为设计模式，它允许在运行时选择算法或行为的方式。策略模式定义了一系列算法，并将每一个算法封装起来，使它们可以互相替换，此模式让算法的变化独立于使用算法的客户。这种类型的设计模式属于行为型模式。

假设我们有一个排序的应用场景，我们希望根据不同的情况使用不同的排序算法（例如，快速排序、冒泡排序等）。我们可以使用策略模式来实现这一点。

首先，定义一个策略接口：

```cpp
class SortStrategy {
public:
    virtual ~SortStrategy() {}
    virtual void sort(std::vector<int>& dataset) = 0;
};
```

然后，实现具体的策略：

```cpp
class QuickSortStrategy : public SortStrategy {
public:
    void sort(std::vector<int>& dataset) override {
        // 实现快速排序算法
    }
};

class BubbleSortStrategy : public SortStrategy {
public:
    void sort(std::vector<int>& dataset) override {
        // 实现冒泡排序算法
    }
};
```

定义一个策略选择器，会根据上下文选择不同的策略：

```cpp
class Sorter {
private:
    SortStrategy* strategy;
public:
    Sorter(SortStrategy* strategy) : strategy(strategy) {}
    
    void setStrategy(SortStrategy* strategy) {
        this->strategy = strategy;
    }
    
    void sort(std::vector<int>& dataset) {
        strategy->sort(dataset);
    }
};
```

```cpp
int main() {
    std::vector<int> dataset = {5, 3, 2, 4, 1};

    Sorter sorter(new QuickSortStrategy());
    sorter.sort(dataset);
    // 数据集现在是通过快速排序算法排序的

    sorter.setStrategy(new BubbleSortStrategy());
    sorter.sort(dataset);
    // 数据集现在是通过冒泡排序算法排序的

    return 0;
}
```

策略模式通常包含三个角色：

1. `Context`（上下文）：持有一个`Strategy`类的引用。
2. `Strategy`（策略）：策略类的接口，定义了每个策略或行为必须实现的操作。
3. `ConcreteStrategy`（具体策略）：实现`Strategy`接口的类，提供具体的算法或行为。

核心思想：**算法可以在运行时被选择和替换，而客户端代码（在这个例子中是`Sorter`类）不需要改变**；

---

以上是使用派生类+虚函数完成策略模式，除此之外，还可以使用函数指针或`std::function`实现策略模式；

[[《Effective C++》#函数指针与Strategy模式]]

