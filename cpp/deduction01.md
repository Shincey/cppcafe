# 类型推导

## 1. 模板类型推导

对于下面的代码：

```c++
// 声明
template <typename T>
void f(ParamType param);

// 调用
f(expr);
```

在编译期间，编译器会使用`expr`去推导`T`和`ParamType`的类型。而这两个类型往往是不同的，因为`ParamType`通常包含一些`const`、引用等一些装饰。比如：

```c++
template <typename T>
void f(const T& param);
// call
int x = 0;
f(x);
```

上述代码会将`T`推导为`int`，而`param`的类型被推导为`int &`。

**类型`T`的推导不仅仅根据`expr`，还根据`ParamType`**。以下有三个不同的案例：

### 1.1. ParamType是一个reference或pointer，但不是universal reference

在这种情况下，类型推导会：

1. 如果`expr`类型是引用，忽略引用部分。
2. 然后将`expr`和`ParamType`进行模式匹配，以确定`T`。

```c++
template <typename T>
void f(T& param); // param is a reference

// variable declarations
int x = 27; // x is an int
const int cx = x; // cx is a const int
const int& rx = x; // rs is a reference to x as a const int

// call
f(x); // T is int, param'type is int&
f(cx); // T is const int, param'type is const int&
f(rx); // T is const int, param'type is const int&
```

`rx`的引用会被忽略，`cx`和`rx`包含了`const`，`T`被推导为`const int`，因此`param`的类型被推导为`const int&`。对象的`const`性质会成为推导`T`类型的一部分，所以将`const`对象传递给`T&`参数是安全的。

即使将`param`的类型声明为`const T&`，也不会有太大变化，可能`T`会被推导为`int`，而`param`类型依旧是`const int&`。

当`param`是一个指针的时候，是同样的：

```c++
template <typename T>
void f(T* param); // param is a pointer
// declarations
int x =27;
const int *px = &x; // px is a ptr to x as a const int
// call
f(&x); // T -> int, param's type -> int*
f(px); // T -> const int, params'type -> const int*
```



### 1.2. ParamType是Universal Reference

universal reference声明像这样：`T&&`，但是它和右值引用是有区别的，另查。

1. 当`expr`是一个左值时，`T`和`ParamType`都会被推导为左值引用。这是模板类型推导中，唯一的一种情况，`T`会被推导为一个引用。尽管`ParamType`声明的时候是用右值引用声明的方式，但是它被推导为一个左值引用。
2. 当`expr`是一个右值，则使用 1.1 的规则。

```c++
template <typename T>
void f(T&& param);
// declarations
int x = 27;
const int cx = x;
const int& rx = x;
// call
f(x); // x in lvalue, so T -> int&, ParamType -> int&.
f(cx); // cx is lvalue, so T -> const int&, ParamType -> const int&.
f(rx); // rx is lvalue, so T -> const int&, ParamType -> const int&.
f(27); // 27 is rvalue, so T -> int, ParamType -> int&&.
```



### 1.3. ParamType既不是reference也不是Pointer

当`ParamType`既不是引用也不是指针时，通过值传递处理：

```c++
template <typename T>
void f(T param); // param is now passed by value
```

这意味着`param`将作为一个副本传入函数体内，`param`将作为一个完整新对象影响推导规则：

1. 如之前，如果`expr`的类型是引用，忽略引用。
2. 忽略引用性质之后，如果有`const`也忽略，如果有`volatile`也忽略。

```c++
int x = 32;
const int cx = x;
const int& rx = x;
// call
f(x); // T and ParamType -> int.
f(cx); // T and ParamType -> int.
f(rx); // T and ParamType -> int.
```

`const`在传递中被忽略似乎说的通，因为`expr`不能被更改，不意味着副本不能被更改。同时，**只有值传递的情况下，const才会被忽略**。

当`expr`是指向`const`对象的`cosnt`指针时，并且通过值传递：

```c++
template <typename T>
void f(T param); // param is still passed by value

const char* const ptr = "fun with pointers"; // ptr is const pointer to const object
// call
f(ptr); // // pass arg of type const const char* const
```

星号右边的`const`是声明指针是`const`的，即不能将指针更改指向其它内存。左边的`const`指不能通过指针更改指针指向的内容。（具体见c++ const）。当将`ptr`传递到`f(ptr)`时，`ptr`自身通过值传递，根据上面的规则，`ptr`的const将会被忽略，`param`的类型将被推导为`const char *`。也就是说，`ptr`执行内容的`const`性质被保留下来了，而`ptr`自身的`const`性质在复制一个新指针`param`时被忽略。

### 1.4. 数组作为参数

```c++
const char name[] = "Hello World"; // name's type is const char[13].
const char* ptrToName = name; // array decays to pointer.
```

虽然数组名也是指向数组第一个元素的地址，但其与指针还是有一些区别的。首先就是类型不一样，一个是`const char[13]`，一个是`const char*`。但是数组名是可以当成指针使用。

```c++
template <typename T>
void f(T param);
f(name);
```

数组通过值传递到一个模板函数里，其类型会被推断为指针类型。所以，上述代码执行后，`T`被推断为`const char*`。

如果是是通过传引用的方式呢：

```c++
template <typename T>
void f(T& param);
f(name);
```

虽然函数不能声明真正的数组参数，但是却可以声明一个引用参数指向数组。所以，这里的`T`会被推断为`const char[13]`，`param`的类型被推断为`const char& [13]`。也许可以利用这一点写一个模板函数在编译期返回数组长度：

```c++
template <typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept {
    return N;
}
```



### 1.5. 函数作为参数

在C++中，除了数组能被转成指针外，函数也是可以被转为函数指针的。

```c++
void func(int, double); // func's type is void(int, double).

template <typename T>
void f1(T param); // param passed by value

template <typename T>
void f2(T& param); // param passed by ref

f1(func); // param deduced as ptr-to-func; type is void(*)(int, double)
f2(func); //  param deduced as ref-to-func; type is void(&)(int, double)
```



### 1.6. 小结

1. 在模板类型推导中，参数是引用的话，其引用会被忽略。
2. 在处理universal reference时，左值需要特别对待。
3. 通过值传递的，`const`、`volatile`等性质会被忽略。
4. 数组名、函数名会被转化为指针，除非用于初始化引用。


### 参考

翻译自：《Effective Modern C++》Scott Meyers著