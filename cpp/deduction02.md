# 类型推导2

## 2. auto类型推导

### 2.1. auto类型推导 == 模板类型推导

如果你读了上一节模板类型推导内容，那么你几乎就掌握了`auto`类型推导，因为其和模板类型推导几乎完全一样。

例如，模板类型推导中有如下：

```c++
template <typename T>
void f(ParamType param);
```
一般调用形式：

```c++
f(expr);
```

在上述函数调用过程中，编译器会根据`expr`去推导类型`T`和`ParamType`。但是在变量通过`auto`声明的例子中，`auto`本身扮演着`T`的角色，类型说明(type specifier)和`ParamType`类似：

```c++
auto x = 27; // x的类型说明就是auto本身
const auto cx = x; // x的类型说明是 const auto
const auto& rx = x; // x的类型说明是 const auto&
```

在进行`x`、`cx`、`rx`类型推导过程中，编译器表现的就像是在进行对应的模板类型推导一样：

```c++
template <typename T>
void func_for_x(T param);

template <typename T>
void func_for_cx(const T param);

template <typename T>
void func_for_rx(const T& param);
```

所以，在上一节的三个规则：

1. type specifier 是一个指针或引用，但不是一个universal reference

2. type specifier 是一个 universal reference

3. type specifier 既不是指针也不是引用

对`auto`也是一样的：

```c++
auto x = 27; // 3. x既不是指针也不是引用
const auto cx = x; // 3. ~
const auto& rx = x; // 1. x是一个引用

auto&& uref1 = x; // 2. x是一个左值，所以 uref1的类型被推导为 int&
auto&& uref2 = cx; // 2. uref2的类型被推导为 const int&
auto&& uref3 = 32; // 2. 32是右值，uref3被推导为 int&&
```

上一节堆数组和函数的讨论，也同样适用于`auto`：

```c++
const char name[] = "Tom"; // name的类型是 const char[3]
auto arr1 = name; // arr1的类型是 const char*
auto& arr2 = name; // arr2的类型是 const char (&)[13]

void func(int, double); // func的类型是 void(int, double)
auto func1 = func; // func1的类型是 void(*)(int, double)
auto& func2 = func; // func2的类型是 void(&)(int, dobule)
```

到目前为止，`auto`表现的和模板类型推导一模一样，但是`auto`还是和模板类型推导有一些差别的。

### 2.2. auto类型推导 != 模板类型推导

c++98可以写出下面的声明初始化语句：

```c++
int x1 = 32;
int x2(32);
```

c++11支持了 uniform 初始化：

```c++
int x3 = {32};
int x4{32};
```

以上4个声明语句都得到同一个结果，`int`值为32。但是，如果将固定类型声明换成使用`auto`声明：

```c++
auto x1 = 32;
auto x2(32);
auto x3 = {32};
auto x4{32};
```

以上语句都可以通过编译，但是前两条语句会得到你想要的结果，后两条语句推导出来的类型却不是`int`。其会被推导为`std::initializer_list<int>`类型，并包含一个单一值32。

通过花括号声明的变量类型会被`auto`推导为`std::initializer_list<int>`，而模板类型推导则不会推导为这个类型，其直接拒绝，提示不能推导该类型。**这是二者唯一的区别**。

```c++
auto x = {12, 13}; //x->std::initializer_list<int>

template <typename T>
void f(T param);

f({12, 13}); // 错误，不能推导类型T
```

但是，如果显示给`ParamType`指定`std::initializer_list<T>`类型，则是可以推导的：

```c++
template <typename T>
void f(std::initializer_list<T> param);

f({12, 13}); // T->int, ParamType->std::initializer_list<int>
```

为啥设定`auto`可以推导花括号，而模板类型推导不可以？没有为啥，规则就是规则，需要记住。

还有一点值得一提就是，在C++14中，允许使用`auto`去推导返回类型，或者匿名函数中的参数。但是，**这种情况下使用的是模板类型推导规则，而不是`auto`的类型推导规则**。虽然几乎相同，但是在推导花括号时就体现出不一样了。

```c++
auto func() {
    return {1,2,3}; //错误，不能够推导{1,2,3}的类型
}

std::vector<int> v;
auto resetV = [&v](const auto& newV) { v = newV; }; //c++14

resetV({1,2,3}); //错误，不能够推导{1,2,3}的类型
```

按照`auto`的规则，应该时可以推导出花括号的类型的，但是这里却不能够推导出其类型。

### 参考

翻译自：《Effective Modern C++》Scott Meyers著