# 智能指针

智能指针的行为类似常规指针，重要的区别是它负责自动释放所指向的对象。`shared_ptr`允许多个指针指向同一个对象，`unique_ptr`则独占所指向的对象，标准库还定义了一个名为`weak_ptr`的伴随类，是一种弱引用，指向`shared_ptr`所管理的对象。都定义在`<memory>`头文件里。

## shared_ptr

智能指针的使用方式和普通指针类似，包括解引用和在一个条件语句里使用智能指针检测是否为空：

```c++
shared_ptr<string> p1;

if (p1 && p1->empty()) {
    *p1 = "hi";
}
```

`shared_ptr`和`unique_ptr`都有的操作

|操作 | 解释|
|---|---|
|`shared_ptr<T> sp`|空智能指针|
|`unique_ptr<T> up`||
|`p`|将p作为一个条件判断是否为空|
|`*p`|解引用p，获得它指向的对象|
|`p->mem`|等价`(*p).mem`|
|`p.get()`|返回p中保存的指针，要小心使用，若智能指针释放了其对象，返回的指针所指向的对象也就消失了|
|`swap(p,q)`|交换p和q中的指针|
|`p.swap(q)`||

`shared_ptr`独有的操作

|操作|解释|
|---|---|
|`make_shared<T>(args)`|返回一个`shared_ptr`，指向一个动态分配的类型为T的对象，使用args初始化对象|
|`shared_ptr<T> p(q)`|p是`shared_ptr`的拷贝，此操作会递增q中的计数器，q中的指针必须能转化为T*|
|`p = q`|p和q都是`shared_ptr`，所保存的指针必须能相互转换。此操作会递减p的引用计数，递增q的引用计数；若p的引用计数变为0,则将其管理的原内存释放|
|`p.unique()`|返回`p.use_count`是否为1|
|`p.use_count()`|返回与p共享对象的智能指针数量；可能很慢，主要用于调试|

使用`new`初始化`shared_ptr`

```c++
shared_ptr<double> p1;
shared_ptr<int> p2(new int(32));
```

接受指针参数的智能指针构造函数是`explicit`，因此我们不能将一个内置指针隐式转换为一个智能指针，必须使用直接初始化形式来初始化一个智能指针：

```c++
shared_ptr<int> p1 = new int(1024); // 错误，必须直接初始化
shared_ptr<int> p2(new int(1024)); // 正确，使用直接初始化
```

```c++
shared_ptr<int> clone(int p) {
    return new int(p); // 错误，不能隐式转换
}
shared_ptr<int> clone(int p) {
    return shared_ptr<int>(new int(p)); // 正确
}
```

定义和改变`shared_ptr`的其它方法

|操作|解释|
|---|---|
|`shared_ptr<T> p(q)`|p管理内置指针q所指的对象；q必须指向new分配的内存，且能够转换为T*类型|
|`shared_ptr<T> p(u)`|p从unique_ptr u那里接管了对象的所有权，将u置为空|
|`shared_ptr<T> p(q, d)`|p接管了内置指针q所指向的对象的所有权。q必须能够转换为T*类型。p将使用可调用对象d来替代delete|
|`shared_ptr<T> p(p2, d)`|p是 shared_ptr p2 的拷贝，p将使用可调用对象d来替代delete|
|`p.reset()`|若p是唯一指向其对象的shared_ptr, reset 会释放此对象。若传递了可选的参数内置指针q，会令p指向q，否则会将p置为空。若还传递了参数d，将会调用d而不是delete来释放q|
|`p.reset(q)`||
|`p.reset(q, d)`||



## unique_ptr

与`shared_ptr`不同的是，某个时刻，只能有一个`unique_ptr`指向一个给定对象。构造`unique_ptr`没有类似`make_shared`库函数调用，只能使用内建指针直接初始化。

```c++
unique_ptr<int> p(new int(32));
```

`unique_ptr`拥有它指向的对象，因此`unique_ptr`不支持普通的拷贝或赋值操作。

`unique_ptr`特有的操作：

|操作|解释|
|---|---|
|`unique_ptr<T> u1`|空`unique_ptr`，可以指向类型为T的对象。u1会使用delete来释放它的指针；u2会使用一个类型为D的可调用对象来释放它的指针|
|`unique_ptr<T, D> u2`||
|`unique_ptr<T, D> u(d)`|空`unique_ptr`，指向类型为T的对象，用类型为D的对象d替代delete|
|`u = nullptr`|释放u指向的对象，将u值为空|
|`u.release()`|u放弃对指针的控制权，返回指针，并将u置为空|
|`u.reset()`|释放u指向的对象|
|`u.reset(q)`|如果提供了内置指针q，令u指向这个对象，否则将u置为空|
|`u.reset(nullptr)`||


## weak_ptr

`weak_ptr`是一种不控制所指向对象生存期的智能指针，它指向由一个`shared_ptr`管理的对象。将一个`weak_ptr`绑定到一个`shared_ptr`不会改变`shared_ptr`的引用计数。一旦最后一个指向对象的`shared_ptr`被销毁，对象就会被释放。即使有`weak_ptr`指向对象，对象也还是会被释放。

|操作|解释|
|---|---|
|`weak_ptr<T> w`|空`weak_ptr`可以指向类型为T的对象|
|`weak_ptr<T> w(sp)`|与`shared_ptr`指向相同对象的`weak_ptr`。T必须能转换为sp指向的类型|
|`w = p`|p可以是一个`shared_ptr`或一个`weak_ptr`。赋值后，w与p共享对象|
|`w.reset()`|将w置为空|
|`w.use_count()`|与w共享对象的`shared_ptr`的数量|
|`w.expired()`|若`w.use_count()`为0返回true，否则返回false|
|`w.lock()`|如果`w.expired()`为true，返回一个空`shared_ptr`，否则返回一个指向w的对象的`shared_ptr`|


## `unique_ptr`和数组

```c++
unique_ptr<T[]> u;
unique_ptr<T[]> u(p); //p指向动态分配的数组
u[i];
```

