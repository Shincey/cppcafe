# auto & decltype

## auto

`auto`会忽略掉顶层`const`，保留底层`const`
```cpp
const int ci = i, &cr = ci;
auto b = ci; // b是一个整数(ci的顶层const被改变)
auto c = cr; // c是一个整数(cr是ci的别名，ci本身是一个顶层const)
```

如果希望推断出的`auto`类型为是一个顶层`const`，则需要明确指出`const auto f = ci;`
设置一个类型为`auto`的引用时，初始值的顶层常量属性仍然保留
```cpp
auto &d = ci; // d是一个整型常量引用，绑定到ci
auto &e = 42; // 错误，不能为非常量引用绑定字面值
const auto &f = 42; // 正确，可以为常量引用绑定到字面值
```


## decltype

```cpp
decltype(f()) sum  = x; // sum的类型就是函数f的返回类型，编译器不实际调用函数f
```

`decltype(var)`会返回变量的类型，包括顶层`const`和引用在内。引用从来都是作为所指对象的同义词出现，除了`delctype`
```cpp
const int ci = 0, &cj = ci;
decltype(ci) x = 0; // x 的类型是 const int
decltype(cj) y = x; // y 的类型时 const int &, y 绑定到 x
decltype(cj) z; // 错误：z是一个引用，必须初始化
```

如果表达式内容是解引用操作，则`decltype`会得到引用类型。解引用指针可以得到指针所指的对象，而且还能给这个对象赋值，因此`decltype(*p)`的结果类型就是`int &`而非`int`：
```cpp
int i = 42, *p = &i;
decltype(*p) c; // 错误：c是int &，必须初始化
```
`decltype((var))`得到的永远是引用，`decltype(var)`只有当`var`是引用时得到的才是引用。

