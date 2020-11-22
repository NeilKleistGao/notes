# Lambda表达式
## “+”的用法
可以使用`+`来标记一个Lambda表达式，强制编译器将该Lambda表达式声明为一个函数指针：
```c++
#include <iostream>
#include <type_traits>

int main()
{
    auto funcPtr = +[](){}; //如果去掉+，下面的断言无法通过：Lambda表达式和函数指针不是同一个数据类型
    static_assert(std::is_same<decltype(funcPtr), void (*)()>::value);
}
```