在C++中，NULL宏被定义为：
```c++
#ifdef __cplusplus
#define NULL 0
#else
#define NULL ((void*)0)
#endif
```

因此，NULL既可以表示空指针，也可以表示0，具有二义性；

c11中新推出的nullptr只表示空指针；