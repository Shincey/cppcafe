# 函数指针

函数指针指向的是函数而非对象，和其它指针一样，函数指针指向函数类型，由函数的返回类型和形参类型共同决定，与函数名无关：

```cpp
bool func(const int &, const int &); // 函数类型是 bool(const int&, const int&)
bool (*pf)(const int &, const int &); // pf指向一个函数，未初始化
pf = func; // 或者 pf = &func; &取地址符是可选的
bool a = pf(1, 2); // 调用 func 函数
bool b = (*pf)(1, 2); // 等价调用
bool c = func(1, 2); //等价调用
```

指向不同函数类型的指针之间不存在转换规则，可以为指针赋值`nullptr`或者值为0的整型常量表达式：

```cpp
int add(const int &, const int &);
pf = add; // 错误：返回类型不匹配
pf = nullptr;
```

若遇到重载函数，则函数指针必须与其中某个函数精确匹配：

```cpp
void f(int *);
void f(int);
void (*pf)(int) = f; // pf 指向 void f(int);
void (*pf)(float) = f; // 错误：没有一个可以匹配
```

函数指针可以作为形参：

```cpp
void func(const int &a, bool pf(const int &)); // 第二个形参是函数类型，会自动转化为函数指针
void func(const int &a, bool (*pf)(const int &)); // 等价声明，显示转换为函数指针
```

可以简化：

```cpp
// F1 和 F2 是函数类型
typedef bool F1(const int &, const int &);
typedef decltype(func) F2; // 等价类型

// FP1 和 FP2 是函数指针
typedef bool (*FP1)(const int &, const int &);
typedef decltype(func) *F2; // 等价类型

using F = int (int *, int); // F 是函数类型，不是函数指针
using PF = int (*)(int *, int); // PF 是函数指针
```

函数可以返回一个函数指针，不能返回一个函数类型：

```
PF f1(int); // ✅
F f1(int); // ❎
F * f1(int); // ✅
```

可以使用尾置返回类型的方式声明一个返回函数指针的函数：

```cpp
int (*f1(int))(int *, int); // ✅ 直接声明
auto f1(int) -> int (*)(int *, int); // 尾置返回类型；
```

