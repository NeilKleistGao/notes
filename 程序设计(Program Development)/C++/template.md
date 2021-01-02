# C++ template
## 返回类型推断
从C++14起，可以使用`auto`关键字，让编译器自动推断函数返回值类型（不一定是模板函数）：
```c++
template <typename T1, typename T2>
auto max(T1 a, T2 b) {
    return b < a ? a : b;
}
```

可以使用后置返回类型进行标记：
```c++
template <typename T1, typename T2>
auto max(T1 a, T2 b) -> decltype(b < a ? a : b) {
    return b < a ? a : b;
}
```

为了防止参数中传递引用，可以使用`std::decay`（或`std::decay_t`）消除：
```c++
template <typename T1, typename T2>
auto max(T1 a, T2 b) -> typename std::decay<decltype(b < a ? a : b)>::type {
    return b < a ? a : b;
}
```

可以使用`std::common_type`（或`std::common_type_t`）表示两个类型中更加通用的那个：
```c++
template <typename T1, typename T2>
std::common_type_t<T1, T2> max(T1 a, T2 b) {
    return b < a ? a : b;
}
```

## 默认模板参数
模板参数的默认值不必出现在整个列表的后面：
```c++
template <typename T1 = double, typename T2>
T1 add(T1 a, T2 b) {
    return a + b;
}
```

但在模板实例化时，仍然按顺序进行填充：
```c++
int main() {
    std::cout << ::add(12.5, 5) << std::endl; // 17.5
    std::cout << ::add<int>(12.5, 5) << std::endl; // 17
    return 0;
}
```

## 模板重载
```c++
template <typename T1, typename T2>
std::common_type_t<T1, T2> max(T1 a, T2 b) {
    std::cout << "called max<>(T1, T2)" << std::endl; // printed
    return b < a ? a : b;
}


template <typename T1, typename T2, typename T3>
auto max(T1 a, T2 b, T3 c) {
    return max(max(a, b), c);
}

int max(int a, int b) {
    std::cout << "called max(int, int)" << std::endl;
    return b < a ? a : b;
}


int main() {
    std::cout << ::max(5, 7, 9) << std::endl;
    return 0;
}
```

```c++
template <typename T1, typename T2>
std::common_type_t<T1, T2> max(T1 a, T2 b) {
    std::cout << "called max<>(T1, T2)" << std::endl;
    return b < a ? a : b;
}

int max(int a, int b) {
    std::cout << "called max(int, int)" << std::endl; // printed
    return b < a ? a : b;
}

template <typename T1, typename T2, typename T3>
auto max(T1 a, T2 b, T3 c) {
    return max(max(a, b), c);
}


int main() {
    std::cout << ::max(5, 7, 9) << std::endl;
    return 0;
}
```

## Partial Usage
一个模板类/函数只有在被使用时才会进行模板实例化（instantiation）。

## 友元
```c++
template <typename T>
class Foo {
public:
    T value;

    friend std::ostream& operator<< (std::ostream& stream, const Foo& f){
        stream << f.value;
        return stream;
    }
};
```

这样的写法会导致每实例化一个`Foo`类，都会连带实例化该函数，而不管该函数是否使用。

可以通过前置申明解决这个问题：
```c++
template <typename T>
class Foo;

template <typename T>
std::ostream& operator<< (std::ostream& stream, const Foo<T>& f);

template <typename T>
class Foo {
public:
    T value;

    friend std::ostream& operator<< <T> (std::ostream& stream, const Foo<T>& f) {
        stream << f.value;
        return stream;
    }
};
```

## 模板别名
```c++
template <typename T>
using vector = std::vector<T>; // 不能使用typedef
```

## 参数推导
不要使用引用！引用会阻止类型退化。在某些情况下使用传值方式，甚至使用移动语义。

## 模板化聚合
聚合类：仅由公有非静态数据构成的类/结构体和非虚函数，不能包含用户提供/`explicit`修饰/继承的构造函数，不能有虚/私有/保护基类。

例子：
```c++
template <typename T>
struct Foo {
    T value;
};

int main() {
    Foo<int> foo{12};
    std::cout << foo.value << std::endl;
    return 0;
}
```

从C++17开始，可以将推断引导（deduction guides）作用在聚合类上：
```c++
template <typename T>
struct Foo {
    T value;
};

Foo() -> Foo<int>;

int main() {
    Foo foo1{};
    foo1.value = 10;
    std::cout << foo1.value << std::endl;
    return 0;
}
```