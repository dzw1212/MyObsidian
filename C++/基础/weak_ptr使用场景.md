# 特点

1. `weak_ptr`只能从`shared_ptr`对象构建，或者从另一个weak_ptr获取；
2. weak_ptr不影响对象的声明周期，即其存在与否并不影响对象的引用计数器；当weak_ptr指向的对象因为shared_ptr计数器为0而被释放后，其lock()方法将返回空值；
3. weak_ptr没有重载`operator->`和`operator*`操作符，因此不能直接通过weak_ptr使用对象；
4. weak_ptr提供了`expired()`方法和`lock()`方法，前者用于判断weak_ptr指向的对象是否已被销毁，后者返回weak_ptr指向的对象的shared_ptr指针（使用该方法会使得shared_pre的计数器加1，如果对象不存在，则返回空值）；

# 使用场景

1. 当想要使用对象，但不想管理对象时（只需要在使用前判断该对象是否还存在即可），可以使用weak_ptr；

2. 解决shared_ptr的循环引用问题；
