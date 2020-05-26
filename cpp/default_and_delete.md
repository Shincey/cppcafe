# default & delete

## =default

* 我们可以通过将拷贝控制等成员定义为 =default 来显示地要求编译器生成合成的版本

  ```c++
  class People {
  public:
    People() = default;
    ~People() = default;
    People(const People&) = default;
    People & operator=(const People&);
  };
  People& People::operator=(const People&) = default;
  ```

* 就像其它任何类内声明的成员函数一样，当我们在类内用 =default 修饰成员的声明的时候，合成的函数将隐式的声明为内联的，如不想合成的成员是内联的，应该只对类外定义使用 =default，就像对拷贝赋值运算符所做的那样



## =delete

* 对于一些类定义拷贝构造函数和拷贝赋值运算符可能没有合理的意义，例如，iostream类阻止了拷贝，以避免多个对象写入或读取相同的IO缓冲。不定义拷贝控制成员这种策略是无效的，因为编译器会生成合成的版本

* 在新标准下，我们可以通过将拷贝构造函数和拷贝赋值运算符定义为**删除的函数**来阻止拷贝，删除的函数是这样的一种函数，我们虽然声明了它，但是不能以任何方式使用它们

  ```c++
  class People {
  public:
    People() = default;
    ~People() = default;
    People(const People&) = delete; //阻止拷贝
    People & operator=(const People&) = delete; //阻止赋值
  }
  ```

* **析构函数不能是删除的成员**,如果析构函数被删除，就无法销毁此类型的对象了，对于一个删除了析构函数的类型，编译器不允许定义该类型的变量或者创建该类的临时对象。而且，如果一个类有某个成员的类型删除了析构函数，我们也不能定义该类的变量或临时对象

  ```c++
  class People {
  public:
    People() = default;
    ~People() = delete; //我们不能销毁People类型的对象
  };
  People Tom; //错误：People的析构函数是删除的
  People *p = new People(); //正确：可以动态分配这种类型的对象，但是不能 delete p
  ```