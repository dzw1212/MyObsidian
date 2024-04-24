观察者模式是一种设计模式，允许多个对象监听某一个对象的状态变化，当这个对象状态变化时，会自动通知所有监听它的对象。在C++中实现观察者模式，通常涉及到两个主要的角色：观察者（`Observer`）和被观察者（`Subject`）；

# 观察者

观察者是一个接口，提供一个更新接口，用于在被观察者状态变化时被调用。

# 被观察者

被观察者持有观察者的列表，并提供方法来添加或删除观察者。当被观察者的状态发生变化时，它会遍历这个列表，通知所有的观察者。

# 示例

```cpp
#include <iostream>
#include <list>
using namespace std;

// 观察者基类
class Observer {
public:
    virtual void update(int state) = 0;
};

// 被观察者基类
class Subject {
private:
    list<Observer*> observers; // 观察者列表
    int state; // 被观察的状态

public:
    void attach(Observer* observer) {
        observers.push_back(observer);
    }

    void detach(Observer* observer) {
        observers.remove(observer);
    }

    void notify() {
        for (Observer* observer : observers) {
            observer->update(state);
        }
    }

    void setState(int state) {
        this->state = state;
        notify();
    }

    int getState() {
        return state;
    }
};

// 具体的观察者
class ConcreteObserver : public Observer {
public:
    void update(int state) override {
        cout << "Observer state updated to " << state << endl;
    }
};

// 使用示例
int main() {
    Subject subject;

    ConcreteObserver observer1, observer2;

    subject.attach(&observer1);
    subject.attach(&observer2);

    subject.setState(1); // 更新状态，观察者会被通知

    return 0;
}
```
