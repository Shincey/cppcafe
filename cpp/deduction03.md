# 类型推导3

## 3. decltype类型推导

### 3.1. decltype返回变量或表达式的值

`decltype`会返回给定表达式或变量的类型，它会确确实实的返回表达时或变量的类型，不会将引用或`const`等属性忽略。

```c++
const int i = 0; // decltype(i) 返回 const int

bool f(const int& ri); // decltype(ri) 返回 const int&
                       // decltype(f) 返回 bool(const int&)
                       // decltype(f(i)) 返回 bool

struct point {
    int x, y; // decltype(point::x) 返回 int
};            // decltype(point::y) 返回 int

template <typename T>
class vector {
public:
    T& operator[](std::size_t index);
};
vector<int> v; // decltype(v) 返回 vector<int>
v[0]; // decltype(v[0]) 返回 int&
```
似乎返回的结果就如我们想到的那样，没有任何改变，是什么就返回什么。

### 3.2. decltype推导函数返回类型

但其实容器通过实现操作符`[]`来索引容器中的内容，对于`operator[]`返回的类型通常推导为`T&`，但是其是依赖于容器的，并不是所有的都返回`T&`，有的会返回一个全新的对象。

c++11可能会写出如下代码：

```c++
template <typename Container, typename Index>
auto acess(Container& c, Index i) -> decltype(c[i]) {
    return c[i];
}
```

函数名称前面的`auto`没有起到类型推导的作用，它指示了c++11的`trailing return type`语法被使用，从参数列表后面的语句去推导返回类型。这也就会使得可以在返回类型推导中去使用函数参数。最终会返回`operator[]`推导出的类型。

c++14可以允许写出下面的代码，让`auto`去推导返回类型。特别的，这意味着编译器会从函数实现来推导函数返回类型：

```c++
template <typename Container, typename Index> // c++14 不完全正确
auto acess(Container& c, Index i) {
    return c[i];
}
```

上一节`auto`类型推导最后，说到使用`auto`推导函数返回类型，编译器使用的是模板类型推导规则。但这里有问题，如之前所说，大多数容器推导`operator[]`返回的类型是`T&`，但在模板类型推导一节中也说到，会把引用性质给忽略。

例如：

```c++
std::queue<int> d;
acess(d, 5) = 10; // 编译失败，提示赋值运算符左操作数必须是左值
```

`d[5]`返回`int&`，但是`auto`推导的`acess`的返回类型会把引用给忽略，实际返回的类型是`int`，这个`int`作为函数的返回值是一个右值。

所以，我们需要让`acess`返回类型和`operator[]`返回的类型一样，c++14允许通过`decltype(auto)`来实现这点，**`auto`指明类型需要被推导，`decltype`指明推导过程中应该使用`decltype`的规则**：

```c++
template <typename Container, typename Index> //c++14可以通过编译，需要优化
decltype(auto) acess(Container& c, Index i) {
    return c[i];
}
```

至此，`acess`的返回类型就和`c[i]`的类型完全一样。如果`c[i]`返回`T&`，则`acess`返回`T&`，如果`c[i]`返回一个对象，则`acess`也返回一个对象。

当然，`decltype(auto)`不仅可以用在函数返回类型推导中，同样也可以用在变量声明：

```c++
Widget w;
const Widget& cw = w;
auto myWidget1 = cw; // myWidget1 类型为 Widget
decltype(auto) myWidget2 = cw; // myWidget 类型为 const Widget&
```

### 3.3. decltype推导函数返回类型最终版本

前面提到需要优化的代码，看似运行没有任何问题，但是我们声明是这样子的：`decltype(auto) acess(Container& c, Index i);`，而使用的时候我们都是传入一个左值容器，运行貌似没有任何问题。但这不意味着，不会传入一个右值容器。右值是没有办法绑定到左值引用上的（除非是const左值引用，这里不讨论）。

> 前面有章节讨论左右值及其引用，没有搞懂的可以去浏览一下。

如果传递一个右值，是一个临时对象，会在调用语句结束的时候被销毁。似乎这么做没什么道理，但确实可以这么做，例如强行解释为拷贝一个临时容器对象中某个元素：

```c++
std::deque<std::string> makeStringDeque(); //工厂函数
auto s = acess(makeStringDeque(), 5); //拷贝临时容器中第5个元素。
```

所以，我们需要重新修改之前的代码，使得它既可以接受左值，又可以接受右值。在模板类型推导中，我们提到了universal reference在对于左值和右值有不一样的推导：

```c++
template <typename Container, typename Index>
decltype(auto) acess(Container&& c, Index i) { // c 是 universal reference
    return std::forward<Container>(c)[i];
}
```
在universal reference中使用`std::forward`转发类型目前先不在这里介绍了。其大致是将引用转发。（前面有一节介绍左值右值及其引用，里面大致介绍了`std::forward`）。

### 3.4. delctype推导表达式再讨论

将`decltype`应用到某个名称上，名称通常是一个左值表达式，但这不影响`decltype`行为。对于比名称更复杂的左值表达式，通常`decltype`返回一个左值引用。

因为大多数左值表达式的类型本质上包含一个左值引用限定符。函数返回左值，例如，通常返回左值引用。

`int x = 0;`如果`decltype(x)`返回`int`没有问题，但如果用`()`括起来`x`，C++将`(x)`表示式定义为左值，`(x)`是一个比名称更复杂的左值表达式。`decltype((x))`返回`int&`。（不知道是我没看懂，还是这个解释太牵强附会，反正记得就好）

该规则在c++14中，`decltype(auto)`推导函数返回类型是也行的通：

```c++
decltype(auto) f1() {
    int x = 0;
    return x; // decltype(x) 是 int，所以返回类型是 int
}

decltype(auto) f2() {
    int x = 0;
    return (x); // decltype((x)) 是 int&， 所以返回类型是 int&
}
```

`f2`返回了函数内临时变量的引用，所以函数调用结束时，该变量也会被销毁。所以，使用`decltype(auto)`还是多加小心吧。（只有充分了解其机制，才能避免）

比如对于指针的解引用操作，`decltype`也是会返回`T&`的，可以把它理解为上述规则，或者解引用的结果本身就是可以赋值的，所以推导为`T&`：

```c++
int * p = new int(3);
decltype(*p) // int&
```

### 3.5. 总结

1. 大多数情况下，`decltype`就是返回其类型，不做任何修改。
2. 当一个左值表达式比其类型T名称更复杂时，往往返回`T&`。
3. c++14的`decltype(auto)`，推导类型时使用的是`decltype`的规则。

### 参考

翻译自：《Effective Modern C++》Scott Meyers著