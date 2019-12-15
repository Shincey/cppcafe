# C++实现函数多返回值

**1. 使用结构体**

C++并不能像python那样一次返回多个值，但是却可以通过一些其它技巧来达到这一点。比如最容易想到的是定义结构体，在结构体中定义多个值，最后返回一个结构体。但是这样做就不得不在函数调用的文件开头也包含结构体的定义。

```c++
struct Student {
	std::string name;
	int id;
};
Student GetInfo() {
	return { "shy", 22 };
}
```

***

**2. 使用指针或引用传参**

利用指针或者引用传参到函数内，利用指针和引用的特性，在函数内改变外部变量的值，从而得到多返回值的效果。

```c++
void GetInfo(string & name, int * id) {
	name = "shy";
	*id = 22;
}
```

***

**3. 利用STL库中的`std::tuple`或`stl::pair`等**

就不探讨返回同类型的了，因为返回同类型的可以直接返回数组或者`vector`等类型来达到效果。如果要返回不同类型的值可以使用`std::tuple`处理多个返回值：

```c++
std::tuple<std::string, int> GetInfo() {
	return { "shy", 22 };
}
```

然后接收返回值的时候：

```c++
// 1.
std::string name;
int id;
std::tie(name, id) = GetInfo();

// 2.
auto [name, id] = GetInfo();
```

以上接收函数返回值当中，第2个方法最便捷，但第2个方法是C++17的标准，所以想要编译通过得确保你使用的是C++17。
