# const

### 初始化

- const对象一旦创建后其值就不能再改变，所以const对象必须__初始化__
- 只能在const类型的对象上执行__不改变其内容的操作__



### 默认状态下 const对象仅在文件内有效

- 如果想在多个文件之间共享const对象，必须在变量的定义前添加extern关键字

  ```c++
  //file_1.cc 定义并初始化一个变量，改变量能被其他文件访问
  extern const int bufSize = fcn();
  //file_1.h
  extern const int bufSize;	//同一个
  ```



### 引用

- 常量引用是对const的引用

- 初始化常量引用时允许用任意表达式做为初始值，只要该表达式的结果能转换成引用类型即可

  ```c++
  //常量引用被绑定到另外一个类型
  double dval = 3.14;
  const int &ri = dval;
  //编译器把上述代码变成如下形式
  const int temp = dval;
  const int &ri = temp;
  //这种情况下ri绑定了一个临时量对象,当ri不是常量时，改变ri只会影响temp对象
  ```

- 常量引用可能引用一个并非const的对象，常量引用仅对引用可参与的操作做出了限定，对引用的对象本身是不是一个常量未做限定

  ```c++
  int i = 42;
  const int &r1 = i; //可以通过其他途径改变i
  ```

### 指针

- 要想存放常量对象的地址，只能使用指向常量的指针

  ```c++
  const double pi = 3.14;
  const double *ptr = &pi;
  ```

- 指向常量的指针没有规定其所指对象必须是一个常量，仅仅要求不能通过该指针改变其所指对象的值，没有规定不能通过其他途径改变

### const指针

- 指针是对象而引用不是，允许把指针本身定为常量，常量指针必须**初始化**，一旦初始化完成，它的值（存放在指针中的地址）就不能再改变

- *const 隐含说明不变的是指针本身的值而不是指向的值

  ```c++
  // 1
  int num = 0;
  int *const ptrNum = &num;
  /*指向非常量的常量指针，ptrNum将会一直指向num不能改变，但是可以改变其所指对象的值*/
  *ptrNum = 1;

  // 2
  const double pi = 3.14159;
  const double *const ptrPi = &pi;
  /*指向常量的常量指针，不论是其所存对象的地址还是所指对象的值都不可以改变*/
  ```

### 顶层const

- **顶层const**：表示指针本身是个常量，可以表示任意的对象是常量

- **底层const**：表示指针所指的对象是一个常量，与指针和引用等复合类型的基本类型部分有关

  ```c++
  int i = 0;
  int *const p1 = &i; //顶层const
  const int ci = 42; //顶层const
  const int *p2 = &ci; //底层const
  const int *const p3 = p2; //靠右的const是顶层，靠左的const是底层
  const int &r = ci; //用于声明引用的const都是底层const
  ```

- 拷贝时底层const不受什么影响，但是执行拷贝的对象必须具有相同的底层const，一般来说非常量可以转换成常量，反之则不行

  ```c++
  int *p = p3; //错误，p3包含顶层const定义，而p没有
  p2 = p3;
  p2 = &i; // int* 可以转换成 const int*
  int &r = ci; //错误，普通的 int& 不能绑定到 int 常量上
  const int &r2 = i; //const int& 可以绑定到一个普通的 int 上
  ```

### constexpr和常量表达式

- **常量表达式** 是指值不会改变并且在编译过程就能得到计算结果的表达式

  ```c++
  const int max_size = 20;
  const int limit = max_size + 7 ;
  int staff_size = 27; // staff_size 不是常量表达式
  const int sz = get_size(); // sz是一个常量，但是具体值得等到运行时才能获得，所以也不是常量表达式
  ```

- **constexpr变量**  c++11允许将变量声明为constexpr类型一遍由编译器来验证变量的值是否是一个常量表达式，声明为constexpr的变量一定是一个常量，而且必须用常量表达式初始化

  ```c++
  constexpr int mf = 20;
  constexpr int limit = mf + 1;
  constexpr int sz = size(); //只有当size是一个constexpr函数时才正确
  ```

- **字面值类型**  算术类型、枚举、引用和指针都属于字面值类型，自定义类、IO库、string类型不属于，一个constexpr指针的初始值必须是nullptr或者0，或者是存储于某个固定地址中的对象，**定义于函数体之外的对象其地址固定不变，能用来初始化constexpr指针**，一般函数体内的变量的地址不是固定不变的，但允许函数定义一类有效范围超出函数本身的变量（局部静态对象），其地址也是固定不变的

- **指针和constexpr**  constexpr把他所定义的对象置为了顶层const

  ```c++
  constexpr int *q = nullptr; //q是一个指向整数的常量指针
  const int *p = nullptr; //p是一个指向整型常量的指针
  ```

  