# c++语法

## 内容总览

> 本笔记系统整理 C++ 核心语法，主线为「基础语法 → 类与对象 → 继承 → 多态」，各节附官方文档链接与可运行示例。

**章节导航**

| 章节 | 核心知识点 |
|---|---|
| **1. 语法知识** | 以下各节合集 |
| · 输入输出 | `std::cout` / `std::cin`、`std::getline`、格式化输出 |
| · 隐式转换 | 算术转换与整型提升、派生类→基类等隐式规则 |
| · 显式转换 | `static_cast` / `dynamic_cast` / `const_cast` / `reinterpret_cast` |
| · 异常捕获 | `try` / `catch` / `throw`、异常传播 |
| · 函数（方法） | 声明与定义、参数传递（值/引用/指针）、重载、内联、默认实参与占位参数 |
| · 数组 / 指针 | 数组与指针基础、数组退化为指针 |
| · 结构体作为参数传递 | 传递方式对比表、选择决策树、常见陷阱 |
| · 类 | 声明定义、成员方法、类内内联、头文件防重编译、构造/析构函数、静态成员、C++ 对象模型 |
| · 友元 | 友元函数/类、性质、优缺点、`operator<<` 应用、注意事项 |
| · 引用 | 本质与语法、传参与返回、常量引用、右值引用、值类别、`auto` 推导、引用折叠与转发引用、引用成员、引用与多态、`std::reference_wrapper`、结构化绑定 |
| · c++的面向对象特性 | 继承、继承方式、继承中的对象模型、多继承、虚继承（菱形继承） |
| · 多态 | 静态/动态多态、vtable/vptr 本质、纯虚函数与抽象类、虚析构与纯虚析构、重写/隐藏/重载、虚函数陷阱、对象切片、RTTI、CRTP 与 NVI、常见误区 |
| **2. 重要的c++库** | 头文件、命名空间与 STL 标准模板库 |
| · STL | 概述（六大组件 / 容器分类 / 迭代器）· 序列容器 · 关联容器 · 无序关联容器 · 容器适配器 · 迭代器 · 算法 · 函数对象与 Lambda · 工具与容器外组件 · 智能指针 · 速查表 |

**学习主线**

1. **基础**：输入输出 → 类型转换 → 异常 → 函数 → 数组/指针 → 结构体传参
2. **封装**：类（构造/析构、`static`、对象模型）
3. **继承**：继承方式 → 对象模型 → 多继承 → 虚继承
4. **多态**：虚函数 → 抽象类 → 虚析构 → 进阶（RTTI / CRTP / NVI）
5. **标准库**：STL —— 容器 → 迭代器 → 算法 → 函数对象 / Lambda → 工具与智能指针

**高频易错点**

- 基类析构函数不是 `virtual` → 基类指针 `delete` 派生对象时资源泄漏
- 忘记 `virtual`（或签名不匹配）→ 变成隐藏而非重写；用 `override` 让编译器把关
- 对象切片：按值传递基类对象会丢失派生部分
- 在构造 / 析构函数中调用虚函数不会多态
- 虚函数默认实参是静态绑定
- `vector` 扩容后旧迭代器 / 指针失效；删除用 `it = c.erase(it)` 接续
- `remove` / `unique` 要配合 `erase` 才真正删除元素

## 1.语法知识

### 输入输出

> 📖 官方文档（cppreference）：[输入/输出库（I/O）总览](https://zh.cppreference.com/w/cpp/io) · [std::cout](https://zh.cppreference.com/w/cpp/io/cout) · [std::cin](https://zh.cppreference.com/w/cpp/io/cin) · [std::endl](https://zh.cppreference.com/w/cpp/io/manip/endl)
> 📖 官方文档（MSVC · Microsoft Learn）：[iostream 参考（标准输入/输出流，含 std::cout / std::cin）](https://learn.microsoft.com/zh-cn/cpp/standard-library/iostream)

```c++
//输出hello world
cout<<"hello world"<<endl
```

**cout：标准输出流**

**endl：换行刷新解释符，换行时强制刷新缓冲区**

**可以使用 << 连接多段输出内容**

```c++
//使用容器装载输入内容
#include<iostream>
using namespace std
int main{
	string input
	cin>>input
	cout<<input
}
//等待用户输入
//只输入回车会一直等待
```

**cin：标准输入流**

**未引用空间std时在使用标准库内容时加前缀std::表引用空间**

### 隐式转换

> 📖 官方文档（cppreference）：[隐式转换（Implicit conversions）](https://zh.cppreference.com/w/cpp/language/implicit_conversion)
> 📖 官方文档（MSVC · Microsoft Learn）：[标准转换（Standard conversions）](https://learn.microsoft.com/zh-cn/cpp/cpp/standard-conversions)

**执行赋值操作A=B时，AB类型不匹配的情况下编译器会尝试将右侧类型转为左侧类型**

**由高精度转低精度时会发生数据丢失**

**执行数学计算时，编译器会将所有变量转为其中更大的类型**

**string不会发生隐式转换**

### 显式转换

> 📖 官方文档（cppreference）：[显式类型转换（explicit cast）](https://zh.cppreference.com/w/cpp/language/explicit_cast) · [std::to_string](https://zh.cppreference.com/w/cpp/string/basic_string/to_string) · [std::stoi / stol / stoll 等](https://zh.cppreference.com/w/cpp/string/basic_string/stol)
> 📖 官方文档（MSVC · Microsoft Learn）：[类型转换与类型安全（现代 C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/type-conversions-and-type-safety-modern-cpp) · [static_cast 运算符](https://learn.microsoft.com/zh-cn/cpp/cpp/static-cast-operator) · [\<string\> 数值转换函数（std::to_string / std::stoi 等）](https://learn.microsoft.com/zh-cn/cpp/standard-library/string-functions)

**1.在表达式前加上(类型) ，执行强制转换**

**2.使用to_string(变量名)方法将其他类型转为字符串(引用头文件string)**

**同理使用stoxx()方法将字符串转为xx类型(stoi,stol,stoll,stoul,stoull,stod,stof)**

**对应int,long long long,unsigned long ,unsigned long long,double,flaot**

### 异常捕获

> 📖 官方文档（cppreference）：[异常处理（try / catch / throw）](https://zh.cppreference.com/w/cpp/language/exceptions)
> 📖 官方文档（MSVC · Microsoft Learn）：[现代 C++ 异常与错误处理最佳实践（try/catch/throw）](https://learn.microsoft.com/zh-cn/cpp/cpp/errors-and-exception-handling-modern-cpp)

```c++
//语法
try{
    //异常捕获的代码块
}
catch(const excepttion&){
    //捕获到后执行的代码
}
```

### 函数（方法）

> 📖 官方文档（cppreference）：[函数（声明 / 形参实参 / 函数重载）](https://zh.cppreference.com/w/cpp/language/functions) · [默认实参](https://zh.cppreference.com/w/cpp/language/default_arguments) · [内联函数 inline](https://zh.cppreference.com/w/cpp/language/inline)
> 📖 官方文档（MSVC · Microsoft Learn）：[函数（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/functions-cpp) · [函数重载](https://learn.microsoft.com/zh-cn/cpp/cpp/function-overloading) · [默认实参](https://learn.microsoft.com/zh-cn/cpp/cpp/default-arguments) · [内联函数（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/inline-functions-cpp)

#### 声明

**严格来说 C++ 只有"函数"；"方法"一般指类中的成员函数（见后文"类"一节）**

**声明（declaration）：只告诉编译器函数的名字、返回类型和形参列表，以分号结尾，不含函数体**

**定义（definition）：声明 + 函数体。一个函数在整个程序中只能有一个定义，但可以被声明多次**

**调用某个函数之前，编译器必须已经看到它的声明或定义**

**1.在头文件中声明**：把函数声明写进头文件（.h/.hpp），需要用到这些函数的源文件 #include 该头文件即可，避免每个文件重复手写声明

**2.源代码中声明和定义**：函数体（定义）写在 .cpp 源文件中，并应包含自己的头文件，方便编译器核对声明与定义是否一致

**命名一般使用驼峰命名法（如 getMax、isValid）**

```c++
// 头文件 mymath.hpp —— 只声明，以分号结尾、无函数体
int add(int a, int b);      // 形参名可省略：int add(int, int);

// 源文件 mymath.cpp —— 定义
int add(int a, int b) {     // 需与声明保持一致
    return a + b;
}
```

```c++
#include <iostream>

int multiply(int a, int b);   // 先声明，再使用

int main() {
    std::cout << multiply(3, 4);   // 输出 12
    return 0;
}

int multiply(int a, int b) {   // 定义放在调用之后同样合法
    return a * b;
}
```

#### 函数参数

**声明和定义时写的参数称形参（parameter），调用时传入的值称实参（argument）**

```c++
int add(int a, int b);   // a、b 是形参

int main() {
    int x = add(3, 5);   // 3、5 是实参
    return 0;
}
```

**值传递：形参是实参的拷贝，函数内修改形参不影响实参**

**若要修改实参本身，必须传引用（int&）或指针（int*）**

**按 const 引用传递（const int&）：只读、不拷贝、效率高，适合传递大对象（见后文"结构体作为参数传递"）**

```c++
#include <iostream>

void byValue(int a)    { ++a; }                 // 改的是拷贝
void byRef(int& a)     { ++a; }                 // 直接改原变量
void byPointer(int* a) { if (a) ++(*a); }       // 经地址改原变量，可判空

int main() {
    int n = 10;
    byValue(n);        // n 仍为 10
    byRef(n);          // n 变为 11
    byPointer(&n);     // n 变为 12
    std::cout << n;    // 输出 12
    return 0;
}
```

**可以在形参处给参数设置默认值；调用时省略该实参则自动使用默认值，这类参数称可选参数（默认实参）**

**默认值只需写一次：函数先声明后定义时，默认值写在声明处、定义处不可重复指定；若函数没有单独声明（定义就是第一个声明），则默认值写在定义处**

```c++
// 头文件 math.hpp —— 声明处给出默认值
int power(int base, int exp = 2);

// 源文件 math.cpp —— 定义处不再重复写 "= 2"
int power(int base, int exp) {
    int result = 1;
    for (int i = 0; i < exp; ++i) result *= base;
    return result;
}

int main() {
    power(3);      // exp 用默认值 2 → 9
    power(3, 4);   // → 81
    return 0;
}
```

**普通参数必须在所有可选参数之前：一旦某个参数有默认值，它右侧的所有参数都必须有默认值**

**调用时只能从右往左连续省略实参，不能跳过中间的参数**

```c++
void f(int a, int b = 1, int c = 2);   // 合法：默认值集中在右侧
// void g(int a = 1, int b);           // 非法：a 有默认值，b 却必须传

int main() {
    f(1);          // a=1 b=1 c=2
    f(1, 5);       // a=1 b=5 c=2
    f(1, 5, 6);    // a=1 b=5 c=6
    // f(1, , 6);  // 非法：不能跳过中间的 b
    return 0;
}
```

**支持多参数设置默认值（见上例）**

**允许只写类型、省略参数名的参数，即占位参数：函数体内不能使用它，但调用时仍须照常传入对应实参（常用于操作符重载区分前置/后置 ++、预留接口、避免"未使用参数"告警）**

```c++
void log(int level, int);      // 第二个参数是占位参数
void log(int level, int) {     // 定义处同样省略参数名
    // 函数体内只使用 level
}

int main() {
    log(3, 0);   // 第二个实参仍需传入，但函数不会用到它
    return 0;
}
```

#### 重载函数

**重载（overload）：同一作用域内函数名相同、但形参的数量、类型或顺序不同的多个函数**

**不可以返回值区分函数重载：仅返回类型不同的两个函数属于重复声明，编译会报错**

**调用时编译器根据实参自动选择"最佳匹配"的那个重载**

```c++
#include <iostream>
#include <string>

void show(int x)                { std::cout << "int "    << x << '\n'; }
void show(double x)             { std::cout << "double " << x << '\n'; }
void show(const std::string& s) { std::cout << "string " << s << '\n'; }

int main() {
    show(42);                    // 精确匹配 show(int)
    show(3.14);                  // 精确匹配 show(double)
    show(std::string("hi"));     // 匹配 show(const std::string&)
    show('A');                   // 无 char 版本，提升为 int → 输出 "int 65"
    return 0;
}
```

**编译器只考虑"调用点之前已经声明"的重载：若某个重载漏声明或声明得太晚，调用时它不会被选中，实参可能被隐式转换后错误匹配到别的重载**

```c++
#include <iostream>

void show(int x) { std::cout << "int " << x << '\n'; }   // 目前只有 int 版本可见

int main() {
    show(2.5);   // double 版本尚未声明 → 匹配 show(int)，输出 "int 2"（小数被截断）
    return 0;
}

void show(double x) { std::cout << "double " << x << '\n'; }   // 声明太晚，main 里不可见
```

**引用形参可以用 const 区分重载：`void f(int&)` 与 `void f(const int&)` 是合法重载，调用时分别匹配"可修改的左值"与"常量左值/右值"**

**普通值形参不能靠 const 区分重载：顶层 const 会被忽略，`void f(int)` 与 `void f(const int)` 会被当成同一个函数（重复声明）**

```c++
#include <iostream>

void f(int& a)       { std::cout << "int&\n"; }        // 可修改的左值
void f(const int& a) { std::cout << "const int&\n"; }  // 常量左值或右值

int main() {
    int n = 1;
    const int c = 2;
    f(n);   // 左值且可修改 → 匹配 int&
    f(c);   // const 左值只能绑 const int&
    f(5);   // 右值不能绑 int& → 匹配 const int&
    return 0;
}
```

**可选参数（默认实参）会影响重载：若省略实参后同时有多个重载可匹配，编译器会报"二义性"错误，而不会替你随便选一个**

```c++
void g(int a) {}
void g(int a, int b = 10) {}

int main() {
    // g(5);   // 编译错误：g(int) 与 g(int, int=10) 都能匹配 → 二义性
    g(5, 1);   // 合法：只有 g(int, int) 唯一匹配
    return 0;
}
```

**因此默认实参与重载可以共存，但必须保证"每次调用都能唯一匹配一个重载"，不能出现二义性**

#### 内联函数

**使用 inline 关键字修饰函数，提示编译器把函数体在调用处直接"展开"，从而省去一次真正的函数调用**

```c++
inline int square(int x) { return x * x; }

int main() {
    int y = square(5);
    // 若内联成功，等价于 int y = 5 * 5;——没有真正的"调用-返回"过程
    return 0;
}
```

**调用时，直接将函数代码块插入调用位置**

**可以减小函数使用的开销**：省去函数调用时的压栈、跳转、返回等额外操作，适合短小且被频繁调用的函数

**会增大函数文件体积**：每个调用点都要复制一份函数体，调用点越多，可执行文件越大（代码膨胀）

**递归函数避免使用内联**：递归要反复调用自身，无法真正展开（展开会无限复制）

**函数过于复杂时编译器可能会忽略inline**：inline 只是"建议"，最终是否内联由编译器决定

**现代编译器无论写不写 inline 都会自动做优化；inline 更重要的作用是允许把函数定义放在头文件里（被多个源文件包含也不会违反"一个函数只能有一个定义"）**

```c++
// 头文件 utils.hpp —— inline 函数的定义可以直接放在头文件
inline int square(int x) { return x * x; }   // 多个 .cpp include 都合法
```

#### 变量类型（声明时使用）

**auto：自 C++11 起表示"类型推导"——由初始化表达式自动推断变量类型（使用 auto 必须先初始化；它已不再是旧 C 语言中"自动存储期"的含义）**

```c++
auto i = 42;                    // i 被推断为 int
auto d = 3.14;                  // d 被推断为 double
auto p = &d;                    // p 被推断为 double*
auto name = std::string("hi");  // name 被推断为 std::string
```

**static（静态）：存放在静态存储区、生命周期贯穿整个程序运行期的变量**

**函数内的局部 static 变量：生命周期为整个程序运行期，但只在首次执行到声明语句时初始化一次，之后再调用不会重新初始化**

**static 修饰的全局变量与函数具有"内部链接"，只能在声明它的源文件中访问，不可跨文件使用**

```c++
#include <iostream>

void nextId() {
    static int id = 0;   // 只初始化这一次，多次调用之间值一直保留
    ++id;
    std::cout << "id = " << id << '\n';
}

int main() {
    nextId();   // id = 1
    nextId();   // id = 2
    nextId();   // id = 3（若换成普通局部变量，每次都会重新从 0 开始）
    return 0;
}
```

**register（寄存器变量）：旧标准中建议编译器把变量放进 CPU 寄存器以加快访问速度；C++11 起已弃用、C++17 起已移除该关键字，现代代码不应再使用**

**现代编译器自带优化，一般情况无需（也无法）手动指定寄存器**

**旧标准下寄存器变量无法获取地址——CPU 寄存器没有内存地址**

**extern：声明"某个全局变量已在其他源文件中定义"，从而允许本文件使用它（extern 只是声明，并不分配存储空间）**

**同一全局变量只能定义一次：若两个源文件重复定义同名全局变量，链接时会产生 multiple definition 错误**

```c++
// a.cpp —— 定义全局变量（整个程序只允许这一处定义）
int sharedCount = 0;

// b.cpp —— 声明并使用 a.cpp 中定义的全局变量
extern int sharedCount;    // 只是声明：链接时去其他编译单元找它的定义
void inc() { ++sharedCount; }
```

### 数组

> 📖 官方文档（cppreference）：[数组（Array）](https://zh.cppreference.com/w/cpp/language/array)
> 📖 官方文档（MSVC · Microsoft Learn）：[数组（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/arrays-cpp)

**线性分配的内存空间**

**二维数组作为参数时，必须指明列数**

### 指针

> 📖 官方文档（cppreference）：[指针（Pointer）](https://zh.cppreference.com/w/cpp/language/pointer)
> 📖 官方文档（MSVC · Microsoft Learn）：[指针（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/pointers-cpp)

**存储其他变量内存地址的变量**

**类型名后加*声明指针 **

**使用&获取变量地址，使用*解引用**

**避免使用野指针**

**野指针：未初始化，内存被释放，内存位置超出作用域，内存未定义/分配的指针**

**&和*允许混用，混用时由右向左结合**

**指向常量的指针：添加关键字const,允许改变指向地址，不能改变指向地址存储的值**

**常量指针：不能改变地址，可以改变值，不能指向常量**

```c++
const int* a;//指向常量的指针
int* const b;//指针常量
const int* const c;//指向常量的指针常量
```

### 结构体作为参数传递

> 📖 官方文档（cppreference）：[函数参数传递（按值 / 引用 / 指针）](https://zh.cppreference.com/w/cpp/language/functions) · [引用（Reference）](https://zh.cppreference.com/w/cpp/language/reference) · [值类别（左值 / 右值）](https://zh.cppreference.com/w/cpp/language/value_category)
> 📖 官方文档（MSVC · Microsoft Learn）：[函数（C++，实参传递）](https://learn.microsoft.com/zh-cn/cpp/cpp/functions-cpp) · [引用（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/references-cpp) · [左值与右值（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/lvalues-and-rvalues-visual-cpp)

在 C++ 中，结构体（`struct`）本质上是**默认访问权限为 `public` 的类**，因此其作为函数参数的传递方式与类完全一致。选择哪种方式主要取决于：**数据大小、是否需要修改、是否可为空、以及性能要求**。

#### 📊 传递方式对比速查表

| 传递方式 | 语法 | 是否拷贝 | 可否修改原对象 | 可否传 `nullptr` | 适用场景 |
|:---|:---|:---:|:---:|:---:|:---|
| **值传递** | `void f(MyStruct s)` | ✅ 是 | ❌ 否（仅改副本） | ❌ 否 | 小型/平凡结构体、函数内部需要独立副本 |
| **常量引用** | `void f(const MyStruct& s)` | ❌ 否 | ❌ 否 | ❌ 否 | **最推荐**：大型结构体只读输入参数 |
| **非常量引用** | `void f(MyStruct& s)` | ❌ 否 | ✅ 是 | ❌ 否 | 需要原地修改输出/输入输出参数 |
| **指针传递** | `void f(MyStruct* s)` | ❌ 否 | ✅ 是（需判空） | ✅ 是 | 参数可选（可能为空）、兼容 C 风格接口 |
| **右值引用** | `void f(MyStruct&& s)` | ❌ 否（转移所有权） | ✅ 是（源对象置为有效但未指定状态） | ❌ 否 | 接收临时对象、实现“移动语义”避免深拷贝 |

---

#### 🔍 详细说明

##### 1. 值传递（Pass by Value）
```cpp
void processByValue(Point p) {
    p.x = 0; // 仅修改副本，不影响调用方
}
```

- **机制**：调用拷贝构造函数生成完整副本。
- **性能**：小型结构体（如 `struct Point { int x,y; }`）通常被编译器优化为寄存器传递，开销极低。
- **⚠️ 注意**：若结构体包含 `std::vector`、`std::string` 或裸指针，会触发深拷贝。现代 C++ 中若函数**本就需要副本**，可故意用值传递 + `std::move` 内部优化（见最佳实践）。

##### 2. 常量引用传递（Pass by `const &`）🥇 默认推荐

```c++
void printData(const Config& cfg) {
    std::cout << cfg.name << ":" << cfg.value;
    // cfg.name = "x"; // ❌ 编译错误：const 保护
}
```

- **机制**：传递隐式指针，**零拷贝**。
- **优势**：安全（防误改）、高效（无复制开销）、支持多态（若 struct 含虚函数）。
- **适用**：90% 以上的“只读输入参数”场景。

##### 3. 非常量引用传递（Pass by `&`）

```c++
void normalize(Vector& v) {
    double len = std::sqrt(v.x*v.x + v.y*v.y);
    v.x /= len; v.y /= len; // 直接修改原对象
}
```

- **机制**：绑定到实参，修改直接影响调用方。
- **语义**：明确表示“此参数将被修改”（Out/In-Out 参数）。
- **⚠️ 注意**：不可接受临时对象（如 `normalize(Point{1,2});` ❌ 编译失败）。

##### 4. 指针传递（Pass by `*`）

```c++
cpp
void updateIfExists(Data* d) {
    if (d) d->status = ACTIVE; // 必须判空
}
```

- **机制**：传递地址。
- **现代 C++ 观点**：**优先用引用**，除非参数明确允许“空值”。若需可空语义，更推荐 `std::optional<std::reference_wrapper<T>>` 或 `std::optional<T>`。

##### 5. 右值引用传递（Pass by `&&`，C++11）

```
cpp
void sink(std::string&& s) {
    global_cache.insert(std::move(s)); // 转移所有权，避免拷贝
}
// 调用：sink(std::move(my_string)); 或 sink("temporary");
```

- **机制**：仅绑定到右值（临时对象或显式 `std::move` 的结果）。
- **用途**：实现“接管资源”的函数（如工厂接收、缓存插入、移动赋值）。
- **⚠️ 警告**：调用后源对象进入“有效但未指定状态”，不应再使用其值。

#### 🧭 现代 C++ 选择指南（决策树）

```
需要修改原对象？ 
  ├─ 是 → 参数可否为空？ 
  │        ├─ 是 → 用指针 T* 或 std::optional<T&>
  │        └─ 否 → 用非常量引用 T&
  └─ 否 → 结构体是否小型/可平凡复制（trivially copyable）？
           ├─ 是 → 值传递 T（编译器通常优化为寄存器）
           └─ 否 → 常量引用 const T&
```

💡 **高级技巧：需要副本时的“值传递+移动”惯用法**

```
// 旧写法（两次拷贝）
void cache(Config cfg) { 
    global_cfg = cfg; // 拷贝
}

// 现代写法（支持拷贝或移动，由调用方决定）
void cache(Config cfg) {
    global_cfg = std::move(cfg); // 若传入右值，则移动；若传入左值，则拷贝
}
// 调用：cache(local_cfg);        // 拷贝构造 → 移动赋值
//       cache(std::move(tmp_cfg)); // 移动构造 → 移动赋值
```

#### 📝 完整示例

```
#include <iostream>
#include <string>
#include <vector>

struct Employee {
    int id;
    std::string name;
    std::vector<double> salaries;
};

// 1. 只读输入 → const &
void printInfo(const Employee& e) {
    std::cout << e.id << ": " << e.name << "\n";
}

// 2. 原地修改 → &
void addBonus(Employee& e, double amount) {
    e.salaries.push_back(amount);
}

// 3. 需要副本 → 值传递 + std::move
void archive(Employee e) { // 故意按值接收
    std::cout << "Archiving " << e.name << "...\n";
    // 内部直接使用 e，若调用方传入右值则自动移动
}

int main() {
    Employee emp{1, "Alice", {5000.0, 5500.0}};

    printInfo(emp);               // const &
    addBonus(emp, 6000.0);        // &
    archive(emp);                 // 拷贝构造 → 移动使用
    archive(std::move(emp));      // 移动构造 → 移动使用（emp 之后不可读）

    return 0;
}
```

#### ⚠️ 常见陷阱



| 问题                               | 原因                           | 解决方案                                               |
| :--------------------------------- | :----------------------------- | :----------------------------------------------------- |
| 函数内修改了引用参数，调用方未察觉 | 误用 `&` 代替 `const &`        | 明确标注 `[[in]]`/`[[out]]` 注释或使用 `gsl::not_null` |
| 传递大型结构体导致性能瓶颈         | 使用值传递未意识到深拷贝       | 改为 `const T&`，或确保实现移动构造                    |
| 指针传递未判空导致段错误           | `if(!p)` 遗漏                  | 优先用引用；若必须可空，用 `std::optional` 或断言      |
| 右值引用参数被多次使用             | `std::move` 后源对象状态未定义 | 移动后不再访问原变量，或重新赋值                       |

#### 📌 总结

- **只读输入**：`const T&`（默认选择）
- **修改输出**：`T&`
- **小型/平凡类型**：`T`（值传递，依赖编译器优化）
- **可空参数**：`T*` 或 `std::optional<T>`
- **转移所有权**：`T&&` + `std::move`

> 💡 现代 C++ 核心原则：**“默认用 `const &`，需要修改用 `&`，需要副本且类型较大时用值传递+`std::move`，指针仅用于可空或兼容 C 接口。”**

### 类

> 📖 官方文档（cppreference）：[类（Class）](https://zh.cppreference.com/w/cpp/language/class)
> 📖 官方文档（MSVC · Microsoft Learn）：[类与结构（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/classes-and-structs-cpp)

#### 声明与定义

**一般声明在头文件（.h/.hpp），定义在源文件（.cpp）**

**类的定义写在头文件里；成员函数的实现可以放到源文件（见下一节）**

```c++
class 类名 {
访问修饰符:                 // public / private / protected，可多次出现
    成员变量;               // 建议给类内初始值
    构造函数与析构函数;
    成员方法;
};                          // ← 结尾分号不能省略
```

**访问权限包含 public、private、protected**

- `public`：类外可访问
- `private`：仅类内可访问（class 的默认权限）
- `protected`：类内及派生类可访问

**高版本 C++（C++11 起）允许在声明中直接给成员赋值（类内初始值）**

**类中的成员要初始化**：通过构造函数或类内初始值完成，否则成员的值是未定义的

**实例化**：可以在栈上或堆上实例化；在别的源文件使用该类，需 `#include` 它的头文件

```c++
// ===== Person.h =====
#pragma once
#include <string>

class Person {
public:                             // 公有：外部可访问
    Person() = default;             // 默认构造函数
    Person(const std::string& name, int age);

    void print() const;             // 成员方法（声明）

private:                            // 私有：仅类内可访问
    std::string name_ = "unknown";  // 类内初始值（C++11）
    int age_ = 0;
};
```

```c++
// ===== Person.cpp =====
#include "Person.h"
#include <iostream>

Person::Person(const std::string& name, int age)
    : name_(name), age_(age) {}     // 成员初始化列表

void Person::print() const {
    std::cout << name_ << " (" << age_ << ")\n";
}
```

```c++
// ===== main.cpp =====
#include "Person.h"

int main() {
    Person a;                                   // 栈上实例化（用默认构造）
    Person b("Alice", 20);                      // 栈上 + 有参构造

    Person* c = new Person("Bob", 22);          // 堆上实例化
    c->print();
    delete c;                                   // 堆对象需手动释放

    a.print();
    b.print();
    return 0;
}
```

**注意**：类定义结尾的 `};` 分号不能丢；成员不初始化时值是未定义的随机值。

#### 成员方法的声明与定义

```c++
//1.在类中声明与定义
//2.类中声明，源文件中定义
//返回值 类名::方法名(参数列表){
	
}
```

**推荐第二种**：接口与实现分离，减少编译依赖、加快增量构建（详细对比见下）

```c++
// ===== Rectangle.h =====
#pragma once

class Rectangle {
public:
    Rectangle(double w, double h) : width_(w), height_(h) {}
    double area() const;        // 声明：const 表示方法内不修改成员变量
    double perimeter() const;   // 声明
private:
    double width_;
    double height_;
};
```

```c++
// ===== Rectangle.cpp =====
#include "Rectangle.h"

double Rectangle::area() const {          // 类名::方法名
    return width_ * height_;
}

double Rectangle::perimeter() const {
    return 2 * (width_ + height_);
}
```

```c++
// ===== main.cpp =====
#include <iostream>
#include "Rectangle.h"

int main() {
    Rectangle r(3.0, 4.0);
    std::cout << r.area() << '\n';        // 12
    std::cout << r.perimeter() << '\n';   // 14
    return 0;
}
```

**注意：模板类、模板成员函数、`constexpr` 函数不能只把定义放在 `.cpp`**——它们必须把定义写在头文件中（使用处需要实例化或编译期求值），否则会出现链接错误

```c++
// 模板成员函数的定义必须写在头文件里
template <typename T>
class Box {
public:
    explicit Box(T v) : value_(v) {}
    T get() const;                 // 声明
private:
    T value_;
};

template <typename T>
T Box<T>::get() const {            // 定义也在头文件里
    return value_;
}
```

##### 📊 核心区别速查表

| 维度           | 类内声明 + 定义（In-class）                            | 类内声明 + 源文件定义（Out-of-class）                        |
| :------------- | :----------------------------------------------------- | :----------------------------------------------------------- |
| **隐式属性**   | 默认隐式 `inline`                                      | 普通外部链接（非 `inline`）                                  |
| **ODR 规则**   | 允许在多个翻译单元重复出现（链接器自动合并）           | 必须严格遵循单一定义规则，重复包含会引发链接错误             |
| **编译依赖**   | 修改实现体 → **所有**包含该头文件的 `.cpp` 需重新编译  | 修改实现体 → **仅**对应 `.cpp` 重编，头文件不变则其他模块不受影响 |
| **头文件污染** | 实现中用到的 `#include` 会泄漏到所有包含该头文件的单元 | 依赖可完全封装在 `.cpp` 中，接口头文件保持干净               |
| **内联倾向**   | 编译器更倾向于内联（尤其短小函数）                     | 需显式 `inline`、LTO 或 PGO 才可能跨文件内联                 |
| **适用场景**   | 极短逻辑/模板/`constexpr`/头文件库/性能关键路径        | 复杂逻辑/依赖较多/需隐藏实现/中大型工程                      |

##### 🔍 深度对比

###### 1. 隐式 `inline` 与 ODR（单一定义规则）

- **类内定义**：C++ 标准规定，**在类体内定义的成员函数隐式具有 `inline` 属性**。这不仅是内联建议，更是 ODR 的例外：即使该头文件被多个 `.cpp` 包含，链接器也会合法合并相同定义，不会报 `multiple definition` 错误。
- **类外定义**：默认不具备 `inline` 属性。若将定义放在头文件中且未加 `inline`，被多个 `.cpp` 包含时必定触发链接期冲突。因此必须将实现移至单一 `.cpp` 文件。

###### 2. 编译依赖与增量构建效率

- **类内**：头文件同时暴露接口与实现。任何实现细节修改（甚至只是改了一行日志），所有 `#include` 该头文件的翻译单元都会触发重新编译。在大型项目中会导致构建时间指数级增长。
- **类外**：头文件仅含声明（函数签名）。修改 `.cpp` 中的实现体时，只要签名不变，其他模块**无需重新编译**，显著加速增量构建。这是工业级 C++ 项目的标准实践。

###### 3. 头文件依赖与封装性

```c++
// ❌ 类内定义：头文件被实现依赖污染
#include <complex_math_lib.h> // 所有使用者被迫包含此重量级头文件
class Processor {
public:
    Result run() {
        return ComplexLib::compute(); // 实现细节暴露
    }
};

// ✅ 类外定义：接口干净，依赖隔离
// Processor.h
class Processor {
public:
    Result run(); // 仅声明
};

// Processor.cpp
#include "Processor.h"
#include <complex_math_lib.h> // 依赖仅存在于编译单元内部
Result Processor::run() { return ComplexLib::compute(); }
```

类外定义支持**前向声明**（Forward Declaration）与 **PIMPL 惯用法**，大幅降低编译耦合度。

##### 4. 内联行为与性能真相

- **误区**：`inline` = 一定内联。现代编译器早已将其视为 **ODR 合规声明**，而非强制性能指令。
- **现实**：
  - 类内定义的短小函数（如 getter/setter）通常被内联。
  - 类外定义的函数，开启 LTO（`-flto`/`/LTCG`）后，编译器同样能跨翻译单元内联。
  - 若函数体超过 ~10 行、含循环/递归/异常处理，无论写在哪，编译器通常**拒绝内联**（避免指令缓存污染）。

##### 5. 调试与可维护性

- **类内**：类定义冗长，逻辑与接口混杂，跳转阅读成本高；调试时单步进入可能直接看到汇编或优化后代码。
- **类外**：接口与实现分离，IDE 导航清晰；可独立对 `.cpp` 设置断点、覆盖率分析；便于单元测试打桩（Mock）。

##### 💻 代码示例对比

```c++
// ========== 方式 A：类内声明 + 定义 ==========
// Utils.h
class StringUtils {
public:
    // 隐式 inline，适合极短逻辑
    static bool isBlank(const std::string& s) {
        return s.empty() || std::all_of(s.begin(), s.end(), ::isspace);
    }
};

// ========== 方式 B：类内声明 + 源文件定义 ==========
// StringUtils.h
class StringUtils {
public:
    static std::string toUpperCase(std::string_view input);
    static std::vector<std::string> split(const std::string& s, char delim);
};

// StringUtils.cpp
#include "StringUtils.h"
#include <algorithm>
#include <cctype>

std::string StringUtils::toUpperCase(std::string_view input) {
    std::string result(input);
    std::transform(result.begin(), result.end(), result.begin(), ::toupper);
    return result;
}

std::vector<std::string> StringUtils::split(const std::string& s, char delim) {
    // 复杂逻辑实现...
}
```

##### 🧭 现代 C++ 选择指南（决策树）

```c++

函数体是否 ≤ 3~5 行？且逻辑简单/无循环/无虚调用？
  ├─ 是 → 类内定义（隐式 inline，适合 getter/setter/类型转换）
  └─ 否 → 是否属于头文件模板库 / constexpr / 需跨 TU 内联？
           ├─ 是 → 类内定义 或 显式 inline 类外定义
           └─ 否 → 类外定义（放 .cpp）
                    ├─ 依赖重？ → 前向声明 + 仅 .cpp 包含头文件
                    ├─ 需隐藏实现？ → PIMPL 或接口抽象类
                    └─ 虚拟函数？ → 通常类外定义（实现多态分发）
```

##### 💡 现代工程实践建议

1. **默认类外定义**：90% 的业务逻辑、工具函数、算法实现应放在 `.cpp`。
2. **显式 `inline` 替代隐式内联**：若需头文件定义但函数较长，显式写 `inline` 表明意图，避免隐式规则歧义。
3. **利用 LTO 打破边界**：开启 `-flto` 后，类外定义也能被内联，无需为性能牺牲模块化。
4. **`constexpr`/`consteval` 函数**：必须在头文件定义（编译期求值要求），通常直接写在类内。
5. **模板成员函数**：必须全放在头文件（实例化依赖），建议类内或类外 `inline` 均可。

##### ⚠️ 常见误区澄清



| 误区                    | 真相                                                         |
| :---------------------- | :----------------------------------------------------------- |
| “类内定义一定更快”      | 仅对极短函数成立；长函数反而增大代码体积，降低 I-Cache 命中率 |
| “`inline` 是性能关键字” | C++17 起主要作用是 ODR 豁免，性能由编译器启发式决定          |
| “类外定义不能被内联”    | LTO/PGO 下跨 TU 内联已成熟，现代编译器优化能力远超源码布局   |
| “所有方法都应放 `.cpp`” | 模板、`constexpr`、头文件库、极小 accessor 放类内更合理      |

##### 📌 总结



| 场景                              | 推荐写法                             |
| :-------------------------------- | :----------------------------------- |
| Getter/Setter、类型转换、极短校验 | `类内定义`（隐式 inline）            |
| 模板类/模板方法、`constexpr` 函数 | `头文件定义`（类内或显式 `inline`）  |
| 业务逻辑、算法、含重量级依赖      | `类内声明 + .cpp 定义`               |
| 需快速增量编译的大型项目          | 严格分离 `.h` / `.cpp`，配合前向声明 |
| 需跨翻译单元内联的非模板函数      | 显式 `inline` + 放头文件，或开启 LTO |

> 📘 **核心原则**：**“接口求稳定，实现求隔离”**。类外定义是工程化 C++ 的基石；类内定义是性能与表达力的特化工具。现代编译器已大幅模糊两者性能差异，**构建效率、可维护性、封装性**应成为首要决策依据。

#### 类中的内联函数

**以第一种方式声明的方法自动成为内联函数**

**第二种方法使用inline变为内联**

**只是建议指令，代码过于复杂时编译器不会将函数变为内联函数**

#### 防止重复编译的方式

```c++
//1
#ifndef A_H
#define A_H
	//代码内容
#endif
//2
#pragma once
	//代码内容
#pragma endregion
```

#### 结构体实现类

**C++ 中允许用 `struct` 实现类，与 `class` 几乎完全一样**（可以有成员变量、成员函数、构造/析构函数、继承、虚函数等）

**唯一区别是默认权限：**
- `struct` 的默认访问权限是 `public`，默认继承方式也是 `public`
- `class` 的默认访问权限是 `private`，默认继承方式也是 `private`

```c++
struct Point {          // 默认 public
    int x;              // 类外可直接访问
    int y;

    Point(int x_, int y_) : x(x_), y(y_) {}   // 构造函数
    int sum() const { return x + y; }         // 成员方法
};

class Point2 {          // 默认 private
    int x;              // 类外不可直接访问
    int y;
public:
    Point2(int x_, int y_) : x(x_), y(y_) {}
    int sum() const { return x + y; }
};

int main() {
    Point p(1, 2);
    p.x = 10;                       // 合法：struct 默认 public

    Point2 q(3, 4);
    // q.x = 10;                    // 非法：class 默认 private
    return 0;
}
```

**默认继承方式也遵循同样的规则：**

```c++
struct Base {};

struct D1 : Base {};    // 等价于 public 继承
class  D2 : Base {};    // 等价于 private 继承
```

**惯例**：`struct` 常用于只包含公开数据的简单聚合类型（如坐标、配置项）；`class` 用于有封装和行为的对象。

#### 构造与析构函数

> 📖 官方文档（cppreference）：[构造函数](https://zh.cppreference.com/w/cpp/language/constructor) · [拷贝构造函数](https://zh.cppreference.com/w/cpp/language/copy_constructor) · [移动构造函数](https://zh.cppreference.com/w/cpp/language/move_constructor) · [析构函数](https://zh.cppreference.com/w/cpp/language/destructor)
> 📖 官方文档（MSVC · Microsoft Learn）：[构造函数（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/constructors-cpp) · [复制构造函数与复制赋值运算符](https://learn.microsoft.com/zh-cn/cpp/cpp/copy-constructors-and-copy-assignment-operators-cpp) · [移动构造函数与移动赋值运算符](https://learn.microsoft.com/zh-cn/cpp/cpp/move-constructors-and-move-assignment-operators-cpp) · [析构函数（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/destructors-cpp)

**构造函数在创建对象时自动调用，负责初始化；析构函数在对象销毁时自动调用，负责清理。**

##### 构造函数分类总览

| 类型 | 写法 | 说明 |
| :--- | :--- | :--- |
| 默认构造 | `A()` | 无参，或所有参数都有默认值 |
| 有参构造 | `A(int, int)` | 接收外部数据初始化对象 |
| 拷贝构造 | `A(const A&)` | 用同类型对象初始化新对象 |
| 移动构造 | `A(A&&) noexcept` | 转移右值资源（C++11） |
| 委托构造 | `A() : A(0) {}` | 调用同类另一个构造函数（C++11） |
| 继承构造 | `using Base::Base;` | 继承基类构造函数（C++11） |
| 列表构造 | `A(std::initializer_list<T>)` | 支持花括号初始化（C++11） |

##### 声明与实现

```c++
/*构造函数
类名(参数列表){
	...
}*/

/*析构函数
~类名(){
 ...
}*/
```

**构造函数在创建对象时自动调用一次，可以被重载**

**析构函数在对象销毁时自动调用，执行清理操作；无参数、不可重载，每个类只有一个**

**若未声明构造函数/析构函数，编译器会在需要时隐式生成默认版本；但一旦自己声明了构造函数，默认构造函数就不再隐式生成**

```c++
#include <iostream>

class Demo {
public:
    Demo()  { std::cout << "构造\n"; }   // 构造函数：与类同名、无返回值
    ~Demo() { std::cout << "析构\n"; }   // 析构函数：~类名、无参数
};

int main() {
    Demo d;        // 进入作用域：构造
    return 0;      // 离开作用域：析构
}
```

##### 构造函数的调用与实现

**按参数分为无参构造与有参构造；按实现方式分为普通构造与拷贝构造**

```c++
/*拷贝构造的写法
类名(const 类名& 对象名){
    // 把 other 的属性拷贝到本对象
}*/
```

**拷贝构造必须使用引用传递（`const 类名&`）**

**若用值传递，形参会先拷贝一次，而拷贝本身又要调用拷贝构造，导致无限递归**

**const 的作用是防止传入的对象被修改**

```c++
class Point {
    int x_, y_;
public:
    Point() : x_(0), y_(0) {}                 // 无参构造
    Point(int x, int y) : x_(x), y_(y) {}     // 有参构造
    Point(const Point& other)                 // 拷贝构造
        : x_(other.x_), y_(other.y_) {}
};
```

###### 调用

```c++
// 1. 括号法
A a;          // 默认构造
A a2(10);     // 有参构造
A a1(a2);     // 拷贝构造
	
// 2. 显式法（等号）
A a;          // 默认构造
A a1 = A(10); // 有参构造（C++17 起无额外拷贝）
A a2 = A(a1); // 拷贝构造
A(10);        // 匿名对象：当前语句结束后立即回收

// 3. 隐式转换（可读性差，不推荐）
A a1 = 10;    // 等价于 A a1 = A(10); 有参构造
A a2 = a1;    // 拷贝构造
```

##### explicit 与隐式转换

**单参数构造函数会被编译器用于隐式类型转换；加 `explicit` 可禁止这种隐式转换**

```c++
class Vec {
public:
    explicit Vec(int n) : n_(n) {}   // 禁止 int 隐式转成 Vec
private:
    int n_;
};

void f(Vec v) { (void)v; }

int main() {
    Vec a(10);      // OK：直接初始化
    // Vec b = 10;  // 错误：explicit 禁止隐式转换
    // f(10);       // 错误：不能把 int 隐式转成 Vec
    f(Vec(10));     // OK：显式构造
    return 0;
}
```

##### 拷贝函数的调用时机

```c++
// 1. 用一个已存在的对象初始化新对象
A a1 = A(a2);

// 2. 值传递传参：形参是实参的副本，会调用拷贝构造
void takeByValue(A a) { }

// 3. 值方式返回对象：返回时会创建副本（现代编译器通常用 RVO/移动优化掉）
A makeA() {
    A a3;
    return a3;
}
```

**编译器隐式生成的特殊成员函数**

**默认构造函数（空实现）、拷贝构造函数（逐成员拷贝）、拷贝赋值运算符、析构函数；C++11 起还会按需生成移动构造与移动赋值**

**注意：编译器不会生成"有参构造函数"——它无法知道你想要的参数类型**

**一旦自己声明了任何构造函数（包括拷贝构造），编译器就不再隐式生成默认构造函数**

**若自己声明了拷贝构造 / 拷贝赋值 / 析构，通常需要同时考虑其余几个（Rule of Three / Five）**

##### = default 与 = delete

**`= default`：显式要求编译器生成默认实现（比手写空函数更好，能保持"平凡"特性）**

**`= delete`：显式禁用某个函数**

```c++
class NonCopyable {
public:
    NonCopyable() = default;
    ~NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;             // 禁止拷贝构造
    NonCopyable& operator=(const NonCopyable&) = delete;  // 禁止拷贝赋值
};

int main() {
    NonCopyable a;
    // NonCopyable b = a;   // 错误：拷贝构造被 delete
    return 0;
}
```

##### 深拷贝与浅拷贝

**浅拷贝**：逐成员按值复制，指针成员只复制地址（两个对象指向同一块内存）

**深拷贝**：为指针成员重新申请一块内存，再把内容复制过去（两个对象各自独立）

```c++
class A {
    int b_;
    int* c_;
public:
    A(int b, int v) : b_(b), c_(new int(v)) {}
    A(const A& other)                 // 拷贝构造：深拷贝
        : b_(other.b_), c_(new int(*other.c_)) {}
    ~A() { delete c_; }
};

int main() {
    A a1(10, 160);
    A a2 = a1;        // 调用深拷贝：a2.c_ 是独立的新内存
    // 若不自定义拷贝构造，默认浅拷贝会让 a2.c_ == a1.c_，
    // 两个对象析构时对同一地址 delete 两次 → 崩溃
}
```

##### 拷贝赋值与 Rule of Three / Five / Zero

**拷贝赋值运算符 `operator=` 给已存在的对象赋值；管理资源时同样要深拷贝，并处理自赋值**

```c++
#include <algorithm>
#include <cstddef>

class Buffer {
    int* data_;
    std::size_t size_;
public:
    explicit Buffer(std::size_t n) : data_(new int[n]), size_(n) {}
    Buffer(const Buffer& other)                     // 拷贝构造
        : data_(new int[other.size_]), size_(other.size_) {
        std::copy(other.data_, other.data_ + other.size_, data_);
    }
    Buffer& operator=(const Buffer& other) {        // 拷贝赋值
        if (this == &other) return *this;           // 处理自赋值
        int* newData = new int[other.size_];        // 先分配，异常安全
        std::copy(other.data_, other.data_ + other.size_, newData);
        delete[] data_;
        data_ = newData;
        size_ = other.size_;
        return *this;
    }
    ~Buffer() { delete[] data_; }
};
```

| 规则 | 内容 |
| :--- | :--- |
| Rule of Three | 需要自定义析构 / 拷贝构造 / 拷贝赋值中的任一个，通常三个都要 |
| Rule of Five | 在 Three 的基础上再加移动构造与移动赋值（C++11） |
| Rule of Zero | 用 RAII 容器（`vector`、`unique_ptr` 等）管理资源，一个都不用写 |

**初始化列表**

**C++ 提供初始化列表语法，用于在构造对象时初始化成员**

```c++
// 构造函数名(形参列表) : 成员1(值1), 成员2(值2), ... { }
```

**参数可以是具体值，也可以是构造函数的形参**

```c++
class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {}   // 用形参初始化成员
    Point() : x_(0), y_(0) {}               // 用具体值初始化成员
};
```

**以下成员必须使用初始化列表（不能靠赋值）：**

| 成员类型 | 原因 |
| :--- | :--- |
| `const` 成员 | 只能初始化，不能赋值 |
| 引用成员 | 必须在创建时绑定到对象 |
| 无默认构造的类类型成员 | 必须显式调用其构造函数 |
| 基类（其构造函数带参） | 必须在派生类初始化列表中调用 |

**初始化顺序只取决于成员在类中的声明顺序，与列表书写顺序无关**

##### 移动构造函数与移动语义（C++11）

**移动构造 `A(A&& other)` 接收右值（临时对象或 `std::move` 的对象），直接"窃取"其资源，避免深拷贝**

```c++
#include <cstddef>
#include <utility>

class Buffer {
    int* data_;
    std::size_t size_;
public:
    explicit Buffer(std::size_t n) : data_(new int[n]), size_(n) {}
    Buffer(const Buffer& other)                 // 拷贝构造：复制一份
        : data_(new int[other.size_]), size_(other.size_) {
        for (std::size_t i = 0; i < size_; ++i) data_[i] = other.data_[i];
    }
    Buffer(Buffer&& other) noexcept             // 移动构造：转移资源
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;                  // 原对象置空，避免重复释放
        other.size_ = 0;
    }
    ~Buffer() { delete[] data_; }
};

int main() {
    Buffer a(100);
    Buffer b = std::move(a);    // 移动：a 不再拥有资源
    Buffer c = Buffer(50);      // 临时对象 → 移动构造
    return 0;
}
```

**要点：**
- 移动构造应标记 `noexcept`，否则标准容器扩容时可能改用拷贝
- 被移动后的对象仍能安全析构，但不应再假设它持有资源
- 若类没有自定义拷贝/移动/析构，编译器会按需自动生成移动版本

##### 委托构造函数（C++11）

**一个构造函数可以调用同类中的另一个构造函数，减少重复初始化代码**

```c++
#include <string>
#include <utility>

class Config {
    int port_;
    std::string host_;
    bool debug_;
public:
    Config(int p, std::string h, bool d)
        : port_(p), host_(std::move(h)), debug_(d) {}
    Config(int p, std::string h) : Config(p, std::move(h), false) {}  // 委托
    Config() : Config(8080, "localhost", false) {}                    // 委托
};
```

##### 类对象作为类成员

**初始化列表可以把参数传给成员对象的构造函数**

**构造顺序：先构造成员对象（按声明顺序），再执行本类构造函数的函数体；析构顺序相反**

```c++
#include <iostream>

class Member {
public:
    Member()  { std::cout << "Member 构造\n"; }
    ~Member() { std::cout << "Member 析构\n"; }
};

class Owner {
    Member m_;          // 成员对象
public:
    Owner()  { std::cout << "Owner 构造体\n"; }
    ~Owner() { std::cout << "Owner 析构体\n"; }
};

int main() {
    Owner o;
    return 0;
}
// 输出：Member 构造 → Owner 构造体 → Owner 析构体 → Member 析构
```

##### 析构函数的调用时机与析构顺序

**自动调用时机：**

| 场景 | 触发条件 |
| :--- | :--- |
| 局部对象 | 离开作用域时 |
| 全局/静态对象 | 程序结束时（按构造逆序） |
| `new` 的对象 | 执行 `delete` 时 |
| 容器元素 | `erase` / `clear` / 扩容时 |
| 临时对象 | 完整表达式结束时 |

**析构顺序与构造顺序严格相反：派生类函数体 → 成员对象（声明逆序）→ 基类（继承逆序）**

```c++
#include <iostream>

struct Base1 { ~Base1() { std::cout << "Base1\n"; } };
struct Base2 { ~Base2() { std::cout << "Base2\n"; } };

struct Derived : Base1, Base2 {          // 继承顺序：Base1 → Base2
    ~Derived() { std::cout << "Derived\n"; }
};

int main() {
    Derived d;
    return 0;      // 输出：Derived → Base2 → Base1
}
```

##### 虚析构函数

**通过基类指针 `delete` 派生类对象时，若基类析构函数不是 `virtual`，只会调用基类析构，派生类资源泄漏 → 未定义行为**

```c++
#include <iostream>

class Base {
public:
    virtual ~Base() { std::cout << "~Base\n"; }   // 必须为 virtual
};

class Derived : public Base {
public:
    ~Derived() override { std::cout << "~Derived\n"; }
};

int main() {
    Base* p = new Derived;
    delete p;      // 输出 ~Derived 再 ~Base
    return 0;
}
```

**规则：只要类可能被公开继承（或包含虚函数），基类析构函数就应声明为 `virtual`**

##### 构造函数/析构函数与异常安全

**构造函数中抛异常**：已构造完成的基类与成员会按逆序自动析构；对象本身不会创建成功

**析构函数中抛异常**：危险！若在栈展开（异常处理）期间再次抛出异常，会调用 `std::terminate()`。析构函数默认是隐式 `noexcept`，清理失败时应内部 `catch` 并吞掉

```c++
class Safe {
public:
    ~Safe() noexcept {
        try {
            // 可能失败、但不允许外抛的清理操作
        } catch (...) {
            // 记录日志并吞掉异常
        }
    }
};
```

#### 静态成员（static关键字）

**变量**：

**属于类而不是对象，所有对象共享同一份数据**

**存放在静态存储区，程序运行期间一直存在**

**类内声明，类外定义/初始化（C++17 起可用 `inline static` 在类内直接定义）**

**可以通过对象或类名访问（受访问权限限制）**

**函数**：

**所有对象共享同一个函数，没有 `this` 指针**

**静态成员函数只能访问静态成员，不能直接访问普通成员**

```c++
#include <iostream>

class Counter {
    static int count_;          // 类内声明
public:
    Counter()  { ++count_; }
    ~Counter() { --count_; }
    static int alive() { return count_; }   // 静态成员函数
};
int Counter::count_ = 0;        // 类外定义并初始化（必要）

int main() {
    Counter a, b;
    std::cout << Counter::alive();   // 2，可用类名调用
    return 0;
}
```

#### c++对象模型

##### 成员变量与成员函数的存储

**对象的内存 = 非静态成员变量 +（可选）虚表指针 vptr + 对齐填充字节**

编译器会给每个空对象分配1字节内存（用于保证不同对象的地址不同）

仅非静态成员变量属于对象，编译器会按照内存对齐的规则分配内存空间

静态变量与非静态成员函数不属于对象，不占用对象的内存空间，所有对象共享

| 内容 | 存储位置 | 是否占用对象内存 |
| :--- | :--- | :---: |
| 非静态成员变量 | 对象内部 | ✅ |
| 虚表指针 vptr | 对象内部（含虚函数时） | ✅ |
| 非静态成员函数 | 代码段（.text） | ❌ |
| 静态成员变量 | 静态存储区 / 数据段 | ❌ |
| 静态成员函数 | 代码段 | ❌ |

```c++
#include <iostream>

class Empty {};                              // 空类

class A {
    int x_;                                  // 非静态成员变量
    void f() {}                              // 非静态成员函数（不占对象空间）
    static int s_;                           // 静态成员变量（不占对象空间）
public:
    virtual void g() {}                      // 有虚函数 → 对象含 vptr
};

int main() {
    std::cout << sizeof(Empty) << '\n';      // 1：空类占位 1 字节
    std::cout << sizeof(A) << '\n';          // 16（64 位）：vptr(8) + int(4) + 填充(4)
    return 0;
}
```

##### 内存对齐与填充

**对齐（alignment）**：每个成员相对对象起始地址的偏移，必须是其自身对齐要求的整数倍

**填充（padding）**：为满足对齐而在成员之间插入的空字节；对象总大小还会"向上取整"到最大成员对齐数的整数倍

```c++
struct S1 { char c; int i; };   // 1 + 3(填充) + 4 = 8 字节
struct S2 { int i; char c; };   // 4 + 1 + 3(填充) = 8 字节
struct S3 { char a; char b; };  // 1 + 1 = 2 字节（无需填充）
```

**结论：成员按"从大到小"排列通常能减少填充、节省内存。**

##### 空类与空基类优化（EBO）

**空类大小至少为 1 字节**（保证不同对象地址不同）；但**空基类子对象可以被优化掉**，不占派生类空间：

```c++
struct Empty {};

struct Derived1 : Empty {
    int i;                                            // 空基类被优化掉
};
static_assert(sizeof(Derived1) == sizeof(int));       // 成立

// C++20 起，空成员也可用 [[no_unique_address]] 优化
struct X {
    int i;
    [[no_unique_address]] Empty e;   // 可与 i 共享存储
};
```

**注意：两个同类型的基类子对象必须有不同地址，所以某些情况下 EBO 会被禁用（如空基类与第一个非静态成员同类型时）。**

##### this指针

隐含每一个非静态成员函数内的指针，指向调用该函数的对象

**当函数形参与成员同名时，使用this区分**

**或在类的非静态函数中返回对象本身（返回引用），常用于链式编程**

**`this` 不能被修改、也不能取地址**（旧教材称其"本质是指针常量"；按标准它是一个纯右值指针，只是不允许赋值）

```c++
class Counter {
    int n_ = 0;
public:
    Counter& inc() { ++n_; return *this; }   // 返回 *this 支持链式调用
    int value() const { return n_; }
};

int main() {
    Counter c;
    c.inc().inc().inc();      // 链式编程，最终 n_ == 3
    return 0;
}
```

**在 `const` 成员函数中，`this` 的类型是 `const 类名*`，因此不能修改成员变量。**

##### 空指针访问成员函数

**通过空指针调用非静态成员函数属于未定义行为（UB）**——只是多数编译器上，若函数体内不访问任何非静态成员变量，恰好不会崩溃

创建一个类的空指针。可以通过此指针调用非静态的成员函数；但是不能访问非静态成员变量，也不能通过成员函数访问成员变量

```c++
class Demo {
    int n_ = 0;
public:
    void safe()   { /* 不访问成员：多数编译器上"侥幸"能跑，但仍是 UB */ }
    void unsafe() { n_ = 1; }   // 访问成员 → 解引用空指针 → 崩溃
};

int main() {
    Demo* p = nullptr;
    p->safe();     // 未定义行为，不要依赖！
    // p->unsafe(); // 实际会崩溃
    return 0;
}
```

##### 虚函数表（vtable）与虚指针（vptr）

**类中一旦有虚函数（含虚析构函数），对象会多出一个隐藏指针 `vptr`**（主流 64 位 ABI 中位于对象起始处，占 8 字节），指向该类的虚函数表 `vtable`

- `vtable` 由编译器在编译期生成，存放在只读数据段，**每个类一份、所有对象共享**
- `vtable` 按声明顺序存放虚函数地址；派生类重写则替换对应槽位，新增虚函数追加到末尾
- `vptr` 在每个构造函数中被设置为"当前正在构造的类"的 vtable

```c++
#include <iostream>

class Base {
public:
    virtual void f() { std::cout << "Base::f\n"; }
    virtual void g() { std::cout << "Base::g\n"; }
    int x_ = 0;
};

class Derived : public Base {
public:
    void f() override { std::cout << "Derived::f\n"; }   // 覆盖槽位 0
};

int main() {
    std::cout << sizeof(Base) << '\n';   // 16：vptr(8) + int(4) + 填充(4)

    Derived d;
    Base* p = &d;
    p->f();     // 动态绑定 → Derived::f（运行时查 vptr 对应的 vtable）
    p->g();     // 未重写 → Base::g
    return 0;
}
```

**动态绑定只发生在"通过基类指针/引用调用虚函数"时**：直接对象调用（`b.f()`）、非虚函数调用，以及在构造/析构函数中调用虚函数，都是静态绑定。

##### 构造与析构期间的对象模型

**构造/析构期间，`vptr` 会在各层构造函数中被反复改写，因此此时调用虚函数不会发生动态绑定：**

- 构造函数中调用虚函数 → 只调用"当前正在构造的类"的版本
- 析构函数同理 → 只调用"当前正在析构的类"的版本
- 这也是**构造函数不能是虚函数**的原因（虚函数依赖已就绪的 vptr）

```c++
#include <iostream>

class Base {
public:
    Base() { f(); }             // 构造期间：调用 Base::f，而非 Derived::f
    virtual void f() { std::cout << "Base::f\n"; }
};

class Derived : public Base {
public:
    void f() override { std::cout << "Derived::f\n"; }
};

int main() {
    Derived d;    // 输出 Base::f
    return 0;
}
```

##### 继承下的对象布局

**单继承**：派生类对象 = 基类子对象（含 vptr）+ 派生类成员；基类部分在前，`Base*` 指向 `Derived` 无需调整地址

**多继承**：按基类声明顺序依次包含多个基类子对象；**每个多态基类各有一个 vptr**，转成非首个基类指针时会发生**地址调整**

**虚继承**：菱形继承中虚基类子对象只保留一份，通过偏移表间接访问，解决数据冗余与二义性

```c++
#include <iostream>

class A { public: virtual void fa() {} int x_ = 1; };
class B { public: virtual void fb() {} int y_ = 2; };
class C : public A, public B {};

int main() {
    C c;
    A* pa = &c;
    B* pb = &c;
    std::cout << (void*)pa << '\n';
    std::cout << (void*)pb << '\n';   // 与 pa 不同：pb 指向 B 子对象（含第二个 vptr）
    return 0;
}
```

**对象切片（object slicing）**：把派生类对象**按值**赋给基类对象时，只复制基类部分，派生类成员与多态能力都会丢失

```c++
Derived d;
Base b2 = d;      // 切片：只保留 Base 部分
b2.f();           // 调用 Base::f（不再是 Derived::f）
Base& r = d;      // 引用不切片，仍可多态
```

##### RTTI（运行时类型识别）

**RTTI 依赖 vtable 中的类型信息，用于 `typeid` 与 `dynamic_cast`**

```c++
#include <iostream>
#include <typeinfo>

class Base { public: virtual ~Base() = default; };
class Derived : public Base { public: void only() {} };

int main() {
    Base* p = new Derived;

    std::cout << typeid(*p).name() << '\n';         // 动态类型：Derived

    if (Derived* d = dynamic_cast<Derived*>(p)) {   // 向下转型成功则非空
        d->only();
    }
    delete p;
    return 0;
}
```

- 对**多态类型**（含虚函数），`typeid(*p)` 在运行时确定真实类型
- `dynamic_cast` 要求基类为多态类型；转换失败时指针返回 `nullptr`、引用抛 `std::bad_cast`
- `static_cast` 不做运行期检查，向下转型不安全

##### 对象模型速查表

| 场景 | 含 vptr？ | 对象大小/布局特点 |
| :--- | :---: | :--- |
| 空类 | ❌ | 1 字节占位 |
| 普通类 | ❌ | 仅非静态成员变量 + 对齐填充 |
| 含虚函数 | ✅ 1 个 | 头部插入 vptr |
| 单继承 | ✅ 1 个 | 基类子对象在前，与派生类共用 vptr |
| 多继承 | ✅ N 个 | 每个多态基类一个 vptr，转型需调整地址 |
| 虚继承 | ✅ + 偏移信息 | 虚基类子对象仅一份，布局复杂 |

##### 代码验证：sizeof 与 offsetof

```c++
#include <cstddef>
#include <iostream>

struct Point { int x; int y; };

int main() {
    std::cout << sizeof(Point) << '\n';        // 8
    std::cout << offsetof(Point, y) << '\n';   // 4（y 相对对象起始的偏移）
    return 0;
}
```

> 💡 **提示**：`offsetof` 仅对**标准布局类型（standard-layout）**有良好定义；查看真实内存布局可用 GDB 打印地址，或观察不同编译器的实现差异。

### 友元

**关键字 friend：修饰类或函数，使其能访问另一个类的私有（private）与受保护（protected）成员。**

**分为友元全局函数、友元类、友元成员函数。**

**友元关系是单向的**：A 被 B 声明为友元，只表示 A 能访问 B 的私有成员，反之不成立。

#### 实现

```c++
#include <iostream>

class B;                        // 前置声明，解决循环依赖

class C {                       // 友元成员函数所属的类，须先声明
public:
    void accessB(const B& b);
};

class B {
    int secret_ = 42;

    friend void showB(const B&);        // ① 友元全局函数
    friend class C;                     // ② 友元类
    friend void C::accessB(const B&);   // ③ 友元成员函数
};

// 全局友元函数（可在类之后定义）
void showB(const B& b) {
    std::cout << b.secret_ << '\n';     // 可访问私有成员
}

// 友元成员函数
void C::accessB(const B& b) {
    std::cout << b.secret_ << '\n';     // 可访问私有成员
}

int main() {
    B b;
    showB(b);        // 42
    C c;
    c.accessB(b);    // 42
    return 0;
}
```

- ① 全局函数 `showB` 可访问 `B` 的私有成员 `secret_`
- ② 友元类 `C` 的所有成员函数都可访问 `B` 的私有成员
- ③ 友元成员函数只有 `C::accessB` 能访问 `B` 的私有成员

**声明顺序**：友元类及友元成员函数所属的类应声明在"授予友元"的类上方（示例中 `C` 在 `B` 之前）。

#### 友元的性质

| 性质 | 说明 |
| :--- | :--- |
| 单向性 | A 是 B 的友元，不代表 B 是 A 的友元 |
| 不可传递 | A 是 B 的友元、B 是 C 的友元，不代表 A 是 C 的友元 |
| 不可继承 | 基类的友元不能访问派生类的私有成员（友元不是成员） |
| 非成员 | 友元函数不是成员函数，不占对象内存、不通过 `this` 调用 |
| 声明位置无关 | 友元声明写在 public / private / protected 下效果相同 |

#### 友元与封装（优缺点）

**优点**
- 让外部函数/类访问私有成员，避免为"能访问"而把成员改成 public
- 常用于运算符重载（如 `operator<<`）和密切协作的类

**缺点**
- 削弱封装、增加耦合；友元不是成员，破坏了信息隐藏
- 应尽量少用，优先提供公有接口 / getter

#### 典型应用：重载输出运算符 `<<`

```c++
#include <iostream>

class Point {
    int x_, y_;
public:
    Point(int x, int y) : x_(x), y_(y) {}
    friend std::ostream& operator<<(std::ostream& os, const Point& p);
};

std::ostream& operator<<(std::ostream& os, const Point& p) {
    os << '(' << p.x_ << ", " << p.y_ << ')';
    return os;
}

int main() {
    Point p(3, 4);
    std::cout << p << '\n';   // (3, 4)
    return 0;
}
```

#### 注意事项

- 友元声明只是"授权"，不会改变类的成员结构，也不占用对象内存
- 友元成员函数所属的类必须先声明，且该成员函数需已声明
- 类与类互相引用时使用前置声明（`class B;`）
- 友元是编译期概念，运行时没有额外开销

### 引用

> 📖 官方文档（cppreference）：[引用（Reference，含右值引用）](https://zh.cppreference.com/w/cpp/language/reference) · [值类别（左值 / 右值 / 将亡值）](https://zh.cppreference.com/w/cpp/language/value_category) · [移动语义 std::move](https://zh.cppreference.com/w/cpp/utility/move)
> 📖 官方文档（MSVC · Microsoft Learn）：[引用（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/references-cpp) · [右值引用声明符 &&](https://learn.microsoft.com/zh-cn/cpp/cpp/rvalue-reference-declarator-amp-amp) · [左值与右值（C++）](https://learn.microsoft.com/zh-cn/cpp/cpp/lvalues-and-rvalues-visual-cpp) · [\<utility\> 函数（std::move）](https://learn.microsoft.com/zh-cn/cpp/standard-library/utility-functions)

**引用（Reference）是已存在变量的"别名"（Alias）**：它不创建新变量，而是让两个名字绑定同一块内存，对引用的所有读写都会直接作用在原变量上。

> 引用**必须初始化**且**不可改绑**，只要正确初始化就不会出现野指针/空引用问题，比指针更安全。现代 C++ 中**能用引用就不用指针**。

#### 语法

```c++
类型 &引用名 = 目标变量;      // & 挨着变量名写（老式写法）
类型& 引用名 = 目标变量;      // & 挨着类型写（现代写法，语义上"类型是 X 的引用"）

int a = 10;
int &b = a;      // 把 b 绑定为 a 的引用（别名）
b = 20;          // 通过引用修改 → 原变量 a 变为 20
```

#### 作用

**给变量起别名，引用与被引用变量绑定同一块内存，二者地址完全相同**：

```c++
int a = 10;
int &b = a;
std::cout << (&a == &b);   // 输出 1（true）：地址一致
```

#### 注意事项（引用的三大性质）

**1. 引用必须初始化**

```c++
int &b;    // ❌ 编译错误：引用声明时必须绑定目标（不存在"空引用"）
```

**2. 初始化后不可再更改绑定的变量（不可"改绑"）**

引用终身绑定第一次绑定的目标，没有语法能把引用改成绑定另一个变量。

**3. 引用不是独立变量：对引用赋值是修改原变量，而不是让引用改绑**

```c++
int a = 10, c = 20;
int &b = a;
b = c;                    // 等价于 a = c → a 变为 20
                          // ❌ 并不会让 b 改去绑定 c，b 仍然是 a 的别名
std::cout << a << " " << c;   // 输出 20 20
```

补充：

- 引用名遵循普通标识符规则：**同一作用域内不能与其它变量重名**（可遮蔽外层同名变量，但不推荐）。
- 引用作为**类的成员**时，必须在构造函数**初始化列表**中绑定（见"构造与析构函数"一节）。
- `int &b = a;` 要求 `a` 是**可修改的左值**；字面量/临时值必须用 `const` 引用或右值引用绑定（见下文）。

#### 使用引用传参

**引用形参可以直接修改实参**，效果与指针传参相同，但无需解引用、无需判空，书写更简洁：

```c++
void change(int &x, int &y) {
	int temp = x;
	x = y;
	y = temp;
}

int a = 1, b = 2;
change(a, b);   // a 与 b 的值被交换：引用形参可以修改实参
```

> 按值 / `const&` / `&` / `*` / `&&` 传参的性能对比、适用场景与选择决策树，见上文「结构体作为参数传递」。

#### 引用作为返回值

**1. 禁止返回局部变量的引用（会产生"悬空引用"）**

```c++
int &bad() {
	int local = 10;
	return local;   // ❌ 局部变量在函数返回时被销毁，
}                   //    返回的引用指向已释放内存 → 未定义行为(UB)
int &r = bad();     // r 成为悬空引用(dangling reference)，访问其值结果不可预测
```

返回引用/指针时，目标必须是**函数结束后依然存活**的对象，例如：

- 全局变量或 `static` 静态局部变量；
- 类的成员变量（成员函数中返回 `this->成员`）；
- 形参传入的引用 / 指针所指向的对象。

**2. 函数返回引用时，函数调用本身可以作为左值**

```c++
int g_value = 10;            // 全局变量：生命周期为整个程序

int &get() {
	return g_value;          // ✅ 返回全局变量的引用
}

get() = 1000;                // 调用作为左值被赋值 → 等价于 g_value = 1000
std::cout << g_value;        // 输出 1000
```

> 重载 `operator[]`、迭代器的 `operator*` 时常利用该特性，让"取出的元素"既能读又能写。

#### 引用的本质

**编译器通过一个"自身 const、不可改指向的指针"（`int *const`）来实现引用**：绑定后指向固定，每次使用时自动解引用。

```c++
// 底层等价模型（便于理解，并非标准强制要求）：
int a = 10;
int *const p = &a;   // 引用 b ≈ 一个不能改指向的指针 p
                     // 之后对 b 的读写 ≈ 对 *p 的读写，且 p 不能再指向其它变量
```

但**引用在语义上不是指针变量**：

- 引用是原变量的别名，`&b` 与 `&a` 完全一致；
- 通常**不单独占用内存**（标准规定引用是否需要存储是"未指明"的，绝大多数编译器直接优化掉）；
- `sizeof(b)` 是被引用类型的大小，而不是地址的大小；
- 不能定义"引用的数组"，也没有"指向引用的指针"（`int&* p;` ❌）。

> 可对照上文「指针」一节：引用 ≈ 自带自动解引用的 `int *const`（初始化即绑定的常量指针）。

#### 常量引用

```c++
const int &a = 10;
// 编译器处理为：int temp = 10; const int &a = temp;
// 临时量 temp 的生存期被延长到引用 a 的作用域结束
```

要点：

- 常量引用可以绑定**字面量 / 临时对象**；非常量引用不行（`int &r = 10;` ❌）。
- 常用于修饰形参：**防止函数内部误改实参**，同时**避免值拷贝开销**。
- 只读访问大对象（如 `struct`、`std::string`、`std::vector`）时优先用 `const &`。

```c++
void print(const std::string &s) {   // 只读、零拷贝
	std::cout << s;
}
```

#### 右值引用（C++11）

先区分两种值类别：

- **左值（lvalue）**：有名字、能取地址、可出现在赋值号左边（如变量 `a`）；
- **右值（rvalue）**：临时对象、字面量、或经 `std::move` 得到的"将亡值"。

```c++
int a = 10;
int &x = a;          // ✅ 左值引用绑定左值
int &y = 10;         // ❌ 编译错误：左值引用不能绑定右值
const int &z = 10;   // ✅ const 左值引用可以绑定右值（生成临时量）

int &&r = 10;        // ✅ 右值引用（&&）专门用来绑定右值
```

右值引用的主要用途是**移动语义**：把临时对象的资源（如堆内存）直接"转移"过来，避免深拷贝：

```c++
std::string tmp = "hello";
std::string s = std::move(tmp);   // 移动而非拷贝：tmp 的堆内存转移给 s
                                  // 之后 tmp 处于"有效但未指定"的状态，不应再读取
```

> 与移动构造 / 移动赋值、以及 `&&` 传参的联系，见上文「构造与析构函数」「结构体作为参数传递」两节。

#### 值类别与引用绑定规则

**C++11 起把表达式的值类别分为三类：**

| 值类别 | 说明 | 例子 |
| :--- | :--- | :--- |
| 左值 lvalue | 有名字、可取地址 | 变量 `a`、`arr[i]`、`*p` |
| 纯右值 prvalue | 纯临时值、无名字 | 字面量 `10`、`a + b`、函数返回值 |
| 将亡值 xvalue | 即将销毁、资源可被转移 | `std::move(a)`、`static_cast<T&&>(a)` |

**不同引用可绑定的值类别：**

| 引用类型 | 可绑定 | 说明 |
| :--- | :--- | :--- |
| `T&` | 仅左值 | 非常量左值引用 |
| `const T&` | 左值 + 右值 | 绑右值会延长其生存期 |
| `T&&` | 仅右值 | 右值引用（prvalue / xvalue） |

```c++
int a = 1;
int& r1 = a;              // ✅ 左值引用绑左值
// int& r2 = 2;           // ❌ 左值引用不能绑右值
const int& r3 = 2;        // ✅ const 左值引用可绑右值
int&& r4 = 2;             // ✅ 右值引用绑右值
```

**注意：具名的右值引用本身是左值**——`r4` 是有名字的变量，不能再被右值引用绑定：

```c++
// int&& r5 = r4;          // ❌ r4 是左值
int&& r5 = std::move(r4);  // ✅ 需 std::move 转成右值
```

#### 引用与 auto 类型推导

**`auto` 默认按值推导（会去掉引用与顶层 const）；要得到引用必须显式写 `&`：**

```c++
int a = 10;
const int c = 20;

auto x = a;          // int（拷贝）
auto& y = a;         // int&（左值引用）
const auto& z = c;   // const int&
auto&& w = 30;       // int&&（转发引用形式，绑定右值）
```

| 写法 | 推导结果 | 说明 |
| :--- | :--- | :--- |
| `auto v = e;` | 值类型 | 拷贝/移动，去引用、去顶层 const |
| `auto& v = e;` | 左值引用 | `e` 必须是左值 |
| `const auto& v = e;` | 常量左值引用 | 可绑左值/右值，只读大对象首选 |
| `auto&& v = e;` | 转发引用 | 随 `e` 的左右值性推导 |

#### 引用折叠与转发引用（万能引用）

**模板形参写成 `T&&` 且发生类型推导时，它是"转发引用（万能引用）"，能同时接收左值和右值：**

```c++
#include <iostream>
#include <utility>

void target(int&)  { std::cout << "绑定左值\n"; }
void target(int&&) { std::cout << "绑定右值\n"; }

template <typename T>
void relay(T&& arg) {                 // T&& 是转发引用
    target(std::forward<T>(arg));     // 按原始值类别转发（完美转发）
}

int main() {
    int a = 1;
    relay(a);    // 左值 → 绑定左值
    relay(2);    // 右值 → 绑定右值
    return 0;
}
```

**引用折叠规则**（"只要出现一次左值引用，结果就是左值引用"）：

| 折叠前 | 折叠后 |
| :--- | :--- |
| `T& &` | `T&` |
| `T& &&` | `T&` |
| `T&& &` | `T&` |
| `T&& &&` | `T&&` |

**`std::move` 与 `std::forward` 的区别：**

- `std::move(x)`：无条件把 `x` 转成右值（用于"我确定不再需要它"）
- `std::forward<T>(x)`：保持 `T` 推导出的原始值类别，用于转发引用中的完美转发

#### 引用与指针的区别

| 对比项 | 引用 `T&` | 指针 `T*` |
|:---|:---|:---|
| 是否必须初始化 | **必须**（无"空引用"） | 不必须（不初始化即野指针） |
| 能否改变绑定的对象 | ❌ 终身绑定 | ✅ 可随时指向其它对象 |
| 能否表示"空" | ❌ | ✅ `nullptr` |
| 访问目标 | 直接用名字 | `*p` 解引用 |
| 指针算术 / 地址运算 | ❌ 不支持 | ✅ 支持（如遍历数组） |
| 占用内存 | 通常不单独占用 | 单独占用一个地址大小的内存 |
| 能否放进数组 | ❌（没有"引用数组"） | ✅ |
| 使用建议 | **默认首选** | 可空参数、动态分配、兼容 C 接口等场景 |

#### 引用作为类成员

**引用成员必须在构造函数初始化列表中绑定；含引用成员的类，拷贝赋值运算符会被隐式删除：**

```c++
class Holder {
    int& ref_;                              // 引用成员
public:
    explicit Holder(int& r) : ref_(r) {}    // 必须用初始化列表绑定
    void set(int v) { ref_ = v; }           // 通过引用成员修改原变量
};

int main() {
    int a = 10;
    Holder h(a);
    h.set(20);            // 原变量 a 变为 20

    Holder h2 = h;        // ✅ 拷贝构造：h2.ref_ 绑定的仍是 a
    // h2 = h;            // ❌ 拷贝赋值被隐式删除（引用不可改绑）
    return 0;
}
```

#### 引用与多态（基类引用绑定派生类）

**基类引用可以绑定派生类对象并触发动态绑定；引用/指针不会发生"对象切片"，按值才会：**

```c++
#include <iostream>

class Base {
public:
    virtual void speak() const { std::cout << "Base\n"; }
};
class Derived : public Base {
public:
    void speak() const override { std::cout << "Derived\n"; }
};

void talk(const Base& b) { b.speak(); }   // 基类引用形参

int main() {
    Derived d;
    talk(d);        // 输出 Derived（引用不切片，动态绑定）
    Base b = d;     // 按值拷贝 → 对象切片
    b.speak();      // 输出 Base
    return 0;
}
```

#### 引用与容器：std::reference_wrapper

**标准容器不能存放引用（`std::vector<int&>` 非法）；需要引用语义时用 `std::reference_wrapper`（`std::ref`）：**

```c++
#include <functional>
#include <iostream>
#include <vector>

int main() {
    int a = 1, b = 2;
    std::vector<std::reference_wrapper<int>> v{a, b};   // 存放引用包装器
    v[0].get() = 100;                                   // 修改的是原变量 a
    std::cout << a << '\n';                             // 100
    return 0;
}
```

#### C++17 结构化绑定（借引用访问成员）

**结构化绑定可把聚合/元组的成员"解包"为名字；用 `auto&` 时得到的是成员引用：**

```c++
#include <tuple>

int main() {
    std::tuple<int, int> t{1, 2};
    auto& [x, y] = t;    // x、y 是 t 成员的引用
    x = 10;              // 修改 t 的第 0 个元素
    return 0;
}
```

#### 常见误区

| 错误写法 / 误解 | 错误原因 | 正确理解 / 做法 |
|:---|:---|:---|
| `int &b;` | 引用未初始化 | 声明时必须绑定目标变量 |
| 认为 `b = c;` 是让 b 改绑到 c | 把引用当指针 | `b = c` 是把 c 的值赋给 a，b 仍绑定 a |
| 返回局部变量的引用/指针 | 局部变量已销毁 | 返回全局 / 静态 / 成员 / 引用参数指向的对象 |
| `int &r = 10;` | 非常量左值引用不能绑定右值 | 改用 `const int &` 或 `int &&` |
| 对引用做 `&b + 1` 等指针运算 | 引用不是指针变量 | 需要指针运算时改用指针 |

### c++的面向对象特性

> 📖 官方文档（cppreference）：[派生类（继承 / 多继承 / 虚基类）](https://zh.cppreference.com/w/cpp/language/derived_class) · [访问说明符（成员访问控制）](https://zh.cppreference.com/w/cpp/language/access) · [无限定的名字查找](https://zh.cppreference.com/w/cpp/language/unqualified_lookup) · [有限定的名字查找](https://zh.cppreference.com/w/cpp/language/qualified_lookup) · [using 声明](https://zh.cppreference.com/w/cpp/language/using_declaration)
> 📖 官方文档（MSVC · Microsoft Learn）：[继承 (C++)](https://learn.microsoft.com/zh-cn/cpp/cpp/inheritance-cpp) · [单一继承](https://learn.microsoft.com/zh-cn/cpp/cpp/single-inheritance) · [多个基类（含虚基类 / 内存布局）](https://learn.microsoft.com/zh-cn/cpp/cpp/multiple-base-classes) · [成员访问控制 (C++)](https://learn.microsoft.com/zh-cn/cpp/cpp/member-access-control-cpp) · [using 声明](https://learn.microsoft.com/zh-cn/cpp/cpp/using-declaration)

**面向对象的三大特性**：**封装（Encapsulation）**、**继承（Inheritance）**、**多态（Polymorphism）**。封装把数据与操作数据的方法绑定为类，对外只暴露必要接口；继承让派生类复用基类的成员；多态让同一接口在不同对象上表现出不同行为（C++ 通过虚函数实现）。本节聚焦 **继承**。

#### 继承

**继承（Inheritance）**：让一个类（**派生类 / 子类**，derived class）获得另一个类（**基类 / 父类**，base class）的成员，从而复用代码并建立 `is-a`（"是一种"）关系——例如 `Student` 是一种 `Person`。

语法：

```c++
//语法： class 子类 ：继承方式 父类
class A {
};

class B : public A {   // 注意：关键字是 public，不是 pubilc
};
```

- **基类（base / 父类）**：被继承的类。
- **派生类（derived / 子类）**：继承得到的类。
- `class` 定义时默认继承方式是 **private**；`struct` 定义时默认是 **public**。
- 派生类对象内部**包含一个完整的基类子对象**（"is-a" 的物理体现）。
- 派生类不能访问基类的 `private` 成员。

示例：

```c++
#include <iostream>
#include <string>

class Person {
protected:                 // 派生类可访问
    std::string name_;
    int age_ = 0;
public:
    Person(std::string name, int age) : name_(name), age_(age) {}
    void introduce() const {
        std::cout << "I am " << name_ << ", " << age_ << " years old.\n";
    }
};

class Student : public Person {   // 公有继承
    std::string school_;
public:
    Student(std::string name, int age, std::string school)
        : Person(name, age), school_(school) {}   // 先构造基类
    void study() const {
        std::cout << name_ << " studies at " << school_ << ".\n";
    }
};

int main() {
    Student s("Tom", 18, "MIT");
    s.introduce();   // 继承自基类
    s.study();       // 派生类自己的成员
    return 0;
}
```

输出：

```text
I am Tom, 18 years old.
Tom studies at MIT.
```

**派生类定义的完整语法**：

```c++
class 派生类名 : 继承方式 基类名 {
    // 派生类自己的成员
};

// 多继承形式：
class 派生类名 : 继承方式1 基类1, 继承方式2 基类2, ... {
};
```

**继承 vs 组合（is-a / has-a）**：

- **继承**表达 `is-a`（"是一种"）：`Student` **是一种** `Person`。
- **组合**表达 `has-a`（"有一个"）：`Car` **有一个** `Engine`。
- 能用组合表达的关系优先用组合——继承会带来强耦合，派生类对象会"背负"基类的全部成员。

```c++
class Engine { /* ... */ };

class Car {          // 组合：Car 有一个 Engine
    Engine engine_;  // 作为成员，而不是继承
};
```

**`final`（C++11）**：禁止一个类被继承。

```c++
class Base {};

class Final final : public Base {};   // Final 不可再被继承

// class Sub : public Final {};       // 编译错误：无法从 final 类派生
```

**继承构造函数（C++11）**：用 `using Base::Base;` 把基类构造函数"引入"派生类，省去逐个转发的样板代码。

```c++
#include <iostream>

class Base {
public:
    int v;
    Base(int x) : v(x) { std::cout << "Base(" << x << ")\n"; }
};

class Derived : public Base {
public:
    using Base::Base;   // 继承 Base 的构造函数
};

int main() {
    Derived d(42);      // 直接调用继承来的 Base(int)
    std::cout << d.v << '\n';
    return 0;
}
```

输出：

```text
Base(42)
42
```

#### 继承方式

根据父类继承的访问权限不同分为私有继承，公共继承，保护继承。

标明的继承权限表明父类中的成员在子类中的最低访问权限，即父类中高于该最低权限的成员在子类强制变为最低权限。

子类不可访问父类的私有成员。

三种继承方式下，基类成员在派生类中的访问级别：

| 基类中的成员 | public 继承 | protected 继承 | private 继承 |
|---|---|---|---|
| `public`    | `public`    | `protected` | `private` |
| `protected` | `protected` | `protected` | `private` |
| `private`   | 不可访问    | 不可访问    | 不可访问  |

规律：派生类中的访问级别 = `min(继承方式, 成员在基类中的访问级别)`；`private` 成员无论哪种继承都不可访问。

| 继承方式 | 含义 | 派生类对象能否当基类对象用（能否多态） |
|---|---|---|
| `public` | 保持 `is-a` 关系 | 能 |
| `protected` | 只对派生类内部开放 | 不能 |
| `private` | 只作为实现细节，不对外暴露 | 不能 |

示例：

```c++
#include <iostream>

class Base {
public:    int a = 1;
protected: int b = 2;
private:   int c = 3;   // 派生类不可访问
};

class Pub : public Base {
public:
    void show() { std::cout << a << ' ' << b << '\n'; }   // a->public, b->protected
};

class Prot : protected Base {
public:
    void show() { std::cout << a << ' ' << b << '\n'; }   // a->protected, b->protected
};

int main() {
    Pub  p; p.show();     // 1 2
    Prot q; q.show();     // 1 2
    // p.a;   // 合法（public 继承下 a 仍是 public）
    // q.a;   // 编译错误（protected 继承下 a 变成 protected）
    return 0;
}
```

输出：

```text
1 2
1 2
```

**三种继承方式的语义与使用场景**：

- **public 继承（接口继承）**：保持 `is-a` 关系，基类的 public 成员在派生类仍是 public，允许派生类对象当基类对象用（多态的基础）。最常用。
- **protected 继承**：基类的 public / protected 成员在派生类降为 protected，只对派生类及其后代开放，对外部不可见。
- **private 继承（实现继承）**：基类的 public / protected 成员在派生类降为 private，只作为实现细节，"用基类来实现派生类"，不构成 `is-a` 关系。

**继承方式对成员的影响**：

- 继承方式只改变基类成员在派生类中的**可访问性**；被继承的成员（含静态成员、嵌套类型 / 类型别名、成员函数）本身依然存在于派生类中。
- 派生类**无法**访问基类的 `private` 成员（无论哪种继承方式）。
- 友元关系**不被继承**：基类的友元不是派生类的友元。

**用 `using` 恢复被降级的访问级别**：`using Base::成员;` 会把该名字的访问级别设为 using 声明所在区域的级别。

```c++
#include <iostream>

class Base {
protected:
    int x = 7;
};

class Derived : private Base {   // private 继承：x 在 Derived 中变为 private
public:
    using Base::x;               // 把 x 提升回 public
};

int main() {
    Derived d;
    std::cout << d.x << '\n';    // 7
    return 0;
}
```

输出：

```text
7
```

#### 继承中的对象模型

子类会继承父类中的所有成员。

父类中的私有成员会被隐藏而不可访问，但分配子类内存时父类私有成员仍然占用空间。

可以使用 vs studio（Visual Studio）中的开发人员工具查看文件中的对象模型，例如在"开发者命令提示符"中运行：

```text
cl /d1 reportSingleClassLayoutStudent main.cpp
```

创造子类对象时，构造顺序从父类构造到子类构造。析构顺序相反（先子类析构，再父类析构）。

完整顺序：

1. 基类构造函数
2. 成员对象（按声明顺序）的构造函数
3. 派生类构造函数体

析构顺序完全相反。

示例（验证私有成员占空间 + 构造析构顺序）：

```c++
#include <iostream>

class Base {
    int secret_ = 0;     // private：派生类不可访问，但仍占空间
public:
    int value_ = 0;
    Base()  { std::cout << "Base ctor\n"; }
    ~Base() { std::cout << "Base dtor\n"; }
};

class Derived : public Base {
public:
    int extra_ = 0;
    Derived()  { std::cout << "Derived ctor\n"; }
    ~Derived() { std::cout << "Derived dtor\n"; }
};

int main() {
    std::cout << "sizeof(Base)=" << sizeof(Base)
              << " sizeof(Derived)=" << sizeof(Derived) << '\n';
    Derived d;
    return 0;
}
```

输出（64 位 MinGW g++；`sizeof` 说明私有成员 `secret_` 确实占了空间）：

```text
sizeof(Base)=8 sizeof(Derived)=12
Base ctor
Derived ctor
Derived dtor
Base dtor
```

> 注：字节数随平台 / 编译器 / 对齐方式变化（32 位下通常为 `sizeof(Base)=4 sizeof(Derived)=8`）。

**空基类优化（EBO，Empty Base Optimization）**：空基类作为基类时可以被优化为不占额外空间（作为**成员**时则至少占 1 字节）。

```c++
#include <iostream>

struct Empty {};                        // 空类：sizeof == 1
struct WithEmpty  { Empty e; int x; };  // 作为成员：Empty 仍占位置
struct DerivedEBO : Empty { int x; };   // 作为基类：空基类优化（EBO）

int main() {
    std::cout << sizeof(Empty) << ' '
              << sizeof(WithEmpty) << ' '
              << sizeof(DerivedEBO) << '\n';
    return 0;
}
```

输出（64 位 MinGW g++）：

```text
1 8 4
```

> `Empty` 本身 `sizeof == 1`；作为成员时 `WithEmpty` 为 8（对齐）；作为基类时经由 EBO，`DerivedEBO` 只有 4——空基类没有额外占空间。

**成员偏移 `offsetof`**：对**标准布局类型**，可用 `offsetof(类型, 成员)` 取得成员相对对象起始的字节偏移（对非标准布局类型使用属未定义行为 / 由实现定义）。

```c++
#include <cstddef>
#include <iostream>

struct S { int a; double b; };

int main() {
    std::cout << offsetof(S, a) << ' ' << offsetof(S, b) << '\n';
    return 0;
}
```

输出：

```text
0 8
```

**指针转换与地址**：

- 派生类指针 → 基类指针（上调 upcast）是**隐式且安全**的；若基类子对象不在对象起始处，编译器会自动调整地址。
- 用 `reinterpret_cast` 在不相干的指针类型之间强转是危险的，不具备可移植性。

##### 同名函数的访问

子类调用父类的同名函数，访问父类的同名静态成员需限定作用域。

同名静态对象调用分为对象访问和类名访问。

**名称隐藏（name hiding）**：派生类中一旦定义了与基类同名的成员（函数或变量），基类的**所有**同名重载都会被隐藏，直接调用会优先匹配派生类的版本。

三种处理方式：

1. 用作用域限定符显式调用基类版本：`Base::func()`；
2. 用 `using Base::func;` 把基类的名字引入派生类作用域，使重载共同参与（避免隐藏）；
3. 同名成员变量同理，用 `Base::var` 区分。

示例：

```c++
#include <iostream>

class Base {
public:
    static int count;
    void func()      { std::cout << "Base::func()\n"; }
    void func(int x) { std::cout << "Base::func(int) " << x << '\n'; }
};
int Base::count = 100;

class Derived : public Base {
public:
    static int count;          // 隐藏 Base::count
    void func() { std::cout << "Derived::func()\n"; }   // 隐藏 Base 的两个 func
};
int Derived::count = 200;

class Derived2 : public Base {
public:
    using Base::func;          // 引入基类重载，避免隐藏
    void func() { std::cout << "Derived2::func()\n"; }
};

int main() {
    Derived d;
    d.func();            // Derived::func()
    d.Base::func();      // Base::func()
    d.Base::func(7);     // Base::func(int) 7

    // 同名静态成员：对象访问 / 类名访问
    std::cout << Derived::count << ' ' << Derived::Base::count << '\n';
    std::cout << d.count << ' ' << d.Base::count << '\n';

    Derived2 e;
    e.func();            // Derived2::func()
    e.func(3);           // Base::func(int) 3（被 using 引入）
    return 0;
}
```

输出：

```text
Derived::func()
Base::func()
Base::func(int) 7
200 100
200 100
Derived2::func()
Base::func(int) 3
```

**名称查找规则**：在派生类中写 `d.f(...)` 时，编译器先在**派生类作用域**查找名字 `f`，一旦找到就停止查找、不再去基类；随后只在这个作用域找到的候选里做重载解析。因此基类的同名重载会**整体**被隐藏，而不是参与重载。

```c++
#include <iostream>

class Base {
public:
    int x = 1;
    void f(int)    { std::cout << "Base::f(int)\n"; }
    void f(double) { std::cout << "Base::f(double)\n"; }
};

class Derived : public Base {
public:
    int x = 2;                                        // 隐藏 Base::x
    void f(int) { std::cout << "Derived::f(int)\n"; } // 隐藏 Base 的两个 f
};

int main() {
    Derived d;
    std::cout << d.x << ' ' << d.Base::x << '\n';   // 2 1
    d.f(1);         // Derived::f(int)
    d.f(1.5);       // 仍调用 Derived::f(int)（Base::f(double) 被隐藏）
    d.Base::f(1.5); // Base::f(double)
    return 0;
}
```

输出：

```text
2 1
Derived::f(int)
Derived::f(int)
Base::f(double)
```

**隐藏 vs 重写（override）**：

- **隐藏**：只要名字相同就发生，与参数、是否为虚函数无关。
- **重写**：仅当基类函数是 `virtual` 且派生类函数**签名相同**时才发生（多态的基础）。即使构成重写，名字查找仍遵循上述规则。

**最佳实践**：派生类中尽量不要使用与基类相同的名字；若确实需要，用 `using Base::名字;` 显式引入基类候选，或始终用 `Base::` 限定。

#### 多继承

c++允许子类继承多个父类。

```c++
//class 子类 ：继承属性 父类1,继承属性 父类2,...{
//};
```

示例：

```c++
#include <iostream>

class Flyable   { public: void fly()  { std::cout << "fly\n"; } };
class Swimmable { public: void swim() { std::cout << "swim\n"; } };

class Duck : public Flyable, public Swimmable {};

int main() {
    Duck d;
    d.fly();
    d.swim();
    return 0;
}
```

输出：

```text
fly
swim
```

子类内存分配包含所有父类的成员。

多个父类出现同名成员时必须通过作用域区分：

```c++
#include <iostream>

class A { public: int value = 1; };
class B { public: int value = 2; };   // 与 A::value 同名

class C : public A, public B {};

int main() {
    C c;
    // c.value;       // 错误：二义性（ambiguous）
    std::cout << c.A::value << ' ' << c.B::value << '\n';  // 1 2
    return 0;
}
```

输出：

```text
1 2
```

不建议使用（容易产生二义性和复杂的对象模型，能用组合 / 接口替代就替代）。

**内存布局**：派生类对象中，各基类子对象按**声明的继承顺序**依次排列（再跟派生类自己的成员，考虑对齐）。因此指向不同基类子对象的指针，其地址可能不同。

```c++
#include <iostream>

class A { public: int a = 1; };
class B { public: int b = 2; };
class C : public A, public B { public: int c = 3; };

int main() {
    C obj;
    A* pa = &obj;   // 指向 A 子对象
    B* pb = &obj;   // 指向 B 子对象（编译器自动加上偏移）
    std::cout << "A offset = " << ((char*)pa - (char*)&obj) << '\n'
              << "B offset = " << ((char*)pb - (char*)&obj) << '\n';
    return 0;
}
```

输出：

```text
A offset = 0
B offset = 4
```

> `A` 子对象在偏移 0，`B` 子对象在偏移 4。用基类指针互相比较 / 相减时要注意这一点。

**建议**：多继承只在"需要组合多个接口 / 多个独立职责"时使用；能用**组合**或**单一继承 + 接口**替代时就替代，可显著降低复杂度与二义性风险。

#### 使用虚继承解决棱形继承问题

两个父类继承同一基类，子类继承两个父类导致存储的基类数据重复问题（棱形 / 菱形继承，diamond problem）。

例如 `A` 派生出 `B`、`C`，`D` 同时继承 `B` 和 `C`，则 `D` 中会有**两份 `A` 的数据**，访问 `A` 的成员会产生二义。

在基类前加上关键字 virtual 使继承基类变为虚继承：

```c++
class A {};
class B : virtual public A {};   // 虚继承
class C : virtual public A {};   // 虚继承
class D : public B, public C {}; // D 中只保留一份 A
```

虚继承本质为将子类中存储的父类数据变为指向虚基类表的虚基类指针（vbptr，virtual base pointer）。

对比示例（`sizeof` 体现数据是否重复）：

```c++
#include <iostream>

class A { public: int a = 0; };

// 普通菱形继承：两份 A
class B1 : public A {};
class C1 : public A {};
class D1 : public B1, public C1 {};

// 虚继承：只保留一份 A
class B2 : virtual public A {};
class C2 : virtual public A {};
class D2 : public B2, public C2 {};

int main() {
    std::cout << "sizeof(A)="  << sizeof(A)
              << " sizeof(D1)=" << sizeof(D1)
              << " sizeof(D2)=" << sizeof(D2) << '\n';
    D2 d;
    d.a = 5;                 // 不再二义（只有一份 A）
    std::cout << d.a << '\n';
    return 0;
}
```

输出（64 位 MinGW g++）：

```text
sizeof(A)=4 sizeof(D1)=8 sizeof(D2)=24
5
```

要点：

- 普通菱形继承 `D1` 里有两份 `A`（`sizeof(D1)=8`）；虚继承 `D2` 只保留一份共享的 `A`，但需额外的虚基类指针，`sizeof(D2)` 变大（示例中为 24）。
- 虚继承只解决**数据冗余 / 二义性**，代价是访问虚基类成员需经虚基类指针间接寻址，有额外开销。
- 只有在确实需要"菱形共享同一基类"时才用虚继承；日常优先用组合。

**虚基类的构造顺序**：虚基类由**最派生类（most-derived class）**负责初始化；中间类对虚基类的初始化会被**忽略**。构造顺序为：虚基类 → 普通基类（按继承顺序）→ 派生类自身。

```c++
#include <iostream>

class A {
public:
    A(int x) { std::cout << "A(" << x << ")\n"; }
};
class B : virtual public A {
public:
    B() : A(1) { std::cout << "B\n"; }   // 非最派生类：对虚基类 A 的初始化被忽略
};
class C : virtual public A {
public:
    C() : A(2) { std::cout << "C\n"; }
};
class D : public B, public C {
public:
    D() : A(3) { std::cout << "D\n"; }   // 最派生类负责初始化虚基类 A
};

int main() {
    D d;
    return 0;
}
```

输出（`A(3)` 生效，`B`/`C` 里的 `A(1)`/`A(2)` 被忽略）：

```text
A(3)
B
C
D
```

**典型例子**：标准库的 `std::iostream` 同时继承 `std::istream` 和 `std::ostream`，而这两者又虚继承 `std::basic_ios`——正是用虚继承避免同一 `basic_ios` 基类出现两份。

**代价与适用场景**：

- 虚继承会引入虚基类指针 / 虚基类表，访问虚基类成员需间接寻址，有**时间与空间开销**，对象布局也更复杂。
- 只有在确实需要"菱形共享同一个基类"时才使用；其他情况下优先用**组合**。

#### 多态

**多态（Polymorphism）**：同一接口在不同对象上表现出不同行为——用**基类指针 / 引用**指向派生类对象，调用同名函数时会执行派生类重写的版本。

分为静态多态和动态多态：

- **静态多态（编译期绑定）**：函数重载、运算符重载、模板，编译器在编译期就确定调用哪个函数。
- **动态多态（运行期绑定）**：通过**虚函数**实现。

函数与运算符的重载属于静态多态。

派生类与虚函数实现动态多态，即使用 virtual 关键字在基类中声明虚函数、在子类中重写。

**即在使用父类指针或引用时可以指向子类对象**，并调用到子类重写的实现。

**动态多态的三大必要条件**：

1. **继承**：存在基类与派生类；
2. **重写**：派生类重写基类的虚函数；
3. **基类指针 / 引用调用**：通过基类指针或引用调用该虚函数（按值传递会发生对象切片，不构成多态）。

示例（同一 `Shape*` 接口，不同图形各自实现）：

```c++
#include <iostream>

class Shape {
public:
    virtual double area() const { return 0.0; }
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double r_;
public:
    explicit Circle(double r) : r_(r) {}
    double area() const override { return 3.14159 * r_ * r_; }
};

class Rectangle : public Shape {
    double w_, h_;
public:
    Rectangle(double w, double h) : w_(w), h_(h) {}
    double area() const override { return w_ * h_; }
};

int main() {
    Shape* shapes[] = { new Circle(1.0), new Rectangle(2.0, 3.0) };
    for (Shape* s : shapes) {
        std::cout << s->area() << '\n';   // 动态绑定：调用各自重写的 area()
    }
    for (Shape* s : shapes) delete s;     // 虚析构保证正确释放
    return 0;
}
```

输出：

```text
3.14159
6
```

##### 多态的本质

**对象中会存储一个指向虚函数表的指针（vptr，虚指针）；虚函数表（vtable）存储该类（实际是对象所属类）的所有虚函数地址。**

- 每个**对象**（不是类）在构造时被设置 vptr，指向所属类的 vtable。
- **未重写时**，子类的虚函数表中存储继承的父类虚函数地址；**重写后**子类虚函数地址覆盖掉父类虚函数地址。
- 通过基类指针 / 引用调用虚函数时，运行时经 `vptr → vtable` 查表，找到实际对象所属类的函数地址——这就是**动态绑定**。

> 关于 vtable / vptr 的内存细节，另见 `#### c++对象模型 → ##### 虚函数表（vtable）与虚指针（vptr）`。

**`override` 与 `final`（C++11）**：

- `override`：显式声明"这是重写"，若签名与基类虚函数不匹配会**编译报错**，避免"本想重写却变成名称隐藏"。
- `final`：禁止该虚函数被进一步重写（也可用于禁止类被继承）。

纯虚函数与抽象类

```c++
//纯虚函数
//virtual 类型 函数名()=0;
virtual void A() = 0;
```

纯虚函数即不实现的虚函数。

任何拥有纯虚函数的类成为**抽象类**，无法实例化。

继承抽象类的子类必须实现抽象类的纯虚函数；只要还有纯虚函数未实现，该子类仍然是抽象类。

**接口类**：只包含纯虚函数（通常还带虚析构）的抽象类，用于定义"能做什么"。

示例：

```c++
#include <iostream>

class Shape {
public:
    virtual double area() const = 0;   // 纯虚函数
    virtual ~Shape() = default;
};

class Square : public Shape {
    double side_;
public:
    explicit Square(double s) : side_(s) {}
    double area() const override { return side_ * side_; }
};

int main() {
    // Shape s;            // 错误：抽象类无法实例化
    Square sq(3.0);
    std::cout << sq.area() << '\n';      // 9
    return 0;
}
```

输出：

```text
9
```

##### 虚析构与纯虚析构

将析构函数写为虚函数、纯虚函数，得到**虚析构**与**纯虚析构**。

纯虚析构用于抽象类。

**当父类指针析构时不会调用子类析构函数，子类析构含有成员变量时可能引发内存泄漏。**

```c++
#include <iostream>

class Base {
public:
    ~Base() { std::cout << "~Base\n"; }         // 非虚析构
};
class Derived : public Base {
public:
    ~Derived() { std::cout << "~Derived\n"; }
};

int main() {
    Base* p = new Derived;
    delete p;    // 只调用 ~Base（~Derived 被绕过）→ 潜在泄漏
    return 0;
}
```

输出：

```text
~Base
```

**使用虚析构解决此问题**：把基类析构函数声明为 `virtual`，`delete` 基类指针时便会先调用派生类析构、再调用基类析构。

```c++
#include <iostream>

class Base {
public:
    virtual ~Base() { std::cout << "~Base\n"; }  // 虚析构
};
class Derived : public Base {
public:
    ~Derived() { std::cout << "~Derived\n"; }
};

int main() {
    Base* p = new Derived;
    delete p;    // 先 ~Derived，再 ~Base
    return 0;
}
```

输出：

```text
~Derived
~Base
```

**纯虚析构需在外部（类外）实现**，否则链接错误：

```c++
#include <iostream>

class Base {
public:
    virtual ~Base() = 0;   // 纯虚析构（仍需提供函数体）
};
Base::~Base() { std::cout << "~Base\n"; }

class Derived : public Base {
public:
    ~Derived() override { std::cout << "~Derived\n"; }
};

int main() {
    Base* p = new Derived;
    delete p;
    return 0;
}
```

输出：

```text
~Derived
~Base
```

**要点**：

- 只作为多态基类使用的类，析构函数应声明为 `virtual`。
- 虚析构具有**传递性**：基类析构为虚，派生类析构自动也是虚。
- 构造函数**不能**是虚函数；析构函数**可以**（且多态基类应当）是虚函数。
- 纯虚析构使类成为抽象类，但仍必须提供函数体。

##### 重写、隐藏与重载

| 概念 | 发生条件 | 绑定时机 |
|---|---|---|
| **重载（overload）** | 同一作用域内、同名但参数不同 | 编译期 |
| **重写（override）** | 派生类中与基类**虚函数签名相同** | 运行期 |
| **隐藏（hide）** | 派生类出现与基类同名的成员（不论是否虚） | 编译期 |

**重写（override）的条件**：

1. 基类函数必须是 `virtual`；
2. 函数名、参数列表、`const` 限定完全相同；
3. 返回类型相同，或为**协变返回类型**（派生类返回更具体的类型）。

用 `override` 关键字让编译器校验，签名不匹配会直接报错：

```c++
#include <iostream>

class Base {
public:
    virtual void f() const { std::cout << "Base::f\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() const override { std::cout << "Derived::f\n"; }  // 正确重写
    // void g() override {}   // 编译错误：没有可重写的基类虚函数
};

int main() {
    Base* p = new Derived;
    p->f();           // Derived::f（多态）
    delete p;
    return 0;
}
```

输出：

```text
Derived::f
```

##### 虚函数的两个陷阱

**陷阱一：默认实参是静态绑定。** 虚函数带默认参数时，实际使用的默认值是**基类声明**里的值（按静态类型确定），与动态绑定到的函数可能不一致。

```c++
#include <iostream>

class Base {
public:
    virtual void f(int x = 1) { std::cout << "Base::f " << x << '\n'; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f(int x = 2) override { std::cout << "Derived::f " << x << '\n'; }
};

int main() {
    Base* p = new Derived;
    p->f();      // 调用 Derived::f，但默认实参取 Base 的 1
    delete p;
    return 0;
}
```

输出：

```text
Derived::f 1
```

**陷阱二：构造 / 析构函数中调用虚函数不产生多态。** 构造基类子对象时 `vptr` 尚指向基类，此时调用虚函数只会执行基类的版本。

```c++
#include <iostream>

class Base {
public:
    Base() { f(); }              // 构造中调用虚函数：不产生多态
    virtual void f() { std::cout << "Base::f\n"; }
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void f() override { std::cout << "Derived::f\n"; }
};

int main() {
    Derived d;       // 输出 Base::f（构造基类时 vptr 指向 Base）
    return 0;
}
```

输出：

```text
Base::f
```

##### 对象切片（Object Slicing）

按值传递 / 用基类对象接收派生类对象时，只有基类部分被拷贝，派生类新增部分被"切掉"，多态失效。改用**指针 / 引用**可避免。

```c++
#include <iostream>

class Base {
public:
    virtual void who() const { std::cout << "Base\n"; }
    virtual ~Base() = default;
};
class Derived : public Base {
public:
    void who() const override { std::cout << "Derived\n"; }
};

void byValue(Base b) { b.who(); }         // 切片：b 是 Base，派生部分被切掉
void byRef(const Base& b) { b.who(); }    // 多态：b 引用实际 Derived

int main() {
    Derived d;
    byValue(d);   // Base（切片）
    byRef(d);     // Derived（多态）
    return 0;
}
```

输出：

```text
Base
Derived
```

##### RTTI：dynamic_cast 与 typeid

RTTI（运行时类型识别）可以在运行期获知多态对象的真实类型：

- **`dynamic_cast`**：用于多态类型之间的安全转换。向下转型（基类→派生类）时，**指针**失败返回 `nullptr`，**引用**失败抛 `std::bad_cast`（需 `#include <typeinfo>`）。
- **`typeid`**：返回 `std::type_info`，可比较运行时类型。

前提：类型必须是**多态类型**（至少有一个虚函数），否则 RTTI 行为不可靠。

```c++
#include <iostream>
#include <typeinfo>

class Base {
public:
    virtual ~Base() = default;   // 有虚函数才是多态类型，才能用 RTTI
};
class Derived : public Base {};
class Other : public Base {};

int main() {
    Base* p = new Derived;

    // dynamic_cast 向下转型：成功返回指针，失败返回 nullptr
    if (dynamic_cast<Derived*>(p)) std::cout << "is Derived\n";
    if (dynamic_cast<Other*>(p))   std::cout << "is Other\n";
    else                           std::cout << "not Other\n";

    // typeid：比较运行时类型
    std::cout << (typeid(*p) == typeid(Derived)) << '\n';   // 1（true）

    delete p;
    return 0;
}
```

输出：

```text
is Derived
not Other
1
```

##### 多态的代价与静态多态（CRTP）

**动态多态的开销**：虚函数调用需经 `vptr → vtable` 间接寻址，通常**无法内联**，比普通函数调用略慢；对象还会多一个 `vptr`。

**CRTP（奇异递归模板模式）**：派生类把自己作为模板参数传给基类，从而在**编译期**完成绑定，可内联、无 vptr 开销——即"静态多态"。

```c++
#include <iostream>

template <typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();   // 编译期绑定，可内联
    }
};

class Impl : public Base<Impl> {
public:
    void implementation() { std::cout << "Impl\n"; }
};

int main() {
    Impl i;
    i.interface();   // Impl
    return 0;
}
```

输出：

```text
Impl
```

**NVI（非虚接口）惯用法**：用非虚的公有函数固定流程，把可变部分交给保护的虚函数，实现"接口稳定 + 行为可定制"。

```c++
#include <iostream>

class Base {
public:
    void run() { doRun(); }        // 非虚接口（NVI）
    virtual ~Base() = default;
protected:
    virtual void doRun() = 0;      // 派生类实现
};

class Impl : public Base {
protected:
    void doRun() override { std::cout << "Impl::doRun\n"; }
};

int main() {
    Impl i;
    i.run();    // Impl::doRun
    return 0;
}
```

输出：

```text
Impl::doRun
```

##### 常见误区汇总

- **忘记 `virtual`**：派生类同名函数变成**隐藏**而非重写，基类指针调用仍走基类版本。加上 `override` 可让编译器帮你发现。
- **基类析构函数不是虚函数**：通过基类指针 `delete` 派生对象时，派生类析构不会被调用 → 资源泄漏。多态基类应写 `virtual ~Base() = default;`。
- **对象切片**：按值传递 / 赋值给基类对象会让多态失效，应使用指针或引用。
- **用基类数组存放派生对象**：`Base arr[10]; arr[0] = Derived();` 同样发生切片。
- **在构造 / 析构函数中调用虚函数**：不会调用到派生类版本。
- **依赖虚函数的默认实参**：默认实参是静态绑定，行为可能与预期不符。
- **抽象类直接实例化**、**纯虚析构忘记提供函数体**（会链接错误）。
- **对非多态类型使用 `dynamic_cast`**：编译报错或行为不符预期。

### 模板

> 📖 官方文档（cppreference）：[模板（Templates）总览](https://zh.cppreference.com/w/cpp/language/templates) · [函数模板](https://zh.cppreference.com/w/cpp/language/function_template) · [类模板](https://zh.cppreference.com/w/cpp/language/class_template) · [模板特化](https://zh.cppreference.com/w/cpp/language/template_specialization) · [参数包（可变参数模板）](https://zh.cppreference.com/w/cpp/language/parameter_pack) · [类型别名 / 别名模板](https://zh.cppreference.com/w/cpp/language/type_alias)
> 📖 官方文档（MSVC · Microsoft Learn）：[模板 (C++)](https://learn.microsoft.com/zh-cn/cpp/cpp/templates-cpp) · [函数模板](https://learn.microsoft.com/zh-cn/cpp/cpp/function-templates) · [类模板](https://learn.microsoft.com/zh-cn/cpp/cpp/class-templates) · [using 声明](https://learn.microsoft.com/zh-cn/cpp/cpp/using-declaration)

用于泛型编程。

c++提供函数模板与类模板。

**泛型编程（Generic Programming）**：编写与具体类型无关的代码，把"类型"（或编译期常量）作为参数，编译器在**编译期**为每种实际类型实例化出对应版本。标准库 STL 几乎全部建立在模板之上。

#### 函数模板

建立一个通用的函数，不具体指定函数返回值和形参类型，而使用**虚拟类型**（模板参数）。

##### 语法

```c++
// 声明一个虚拟类型 T，typename 可替换为 class（二者等价）
template <typename T>
T add(T a, T b) {
    return a + b;
}
// 调用分 2 种：
int x = add(10, 10);        // 1. 编译器自动推导 T=int
int y = add<int>(10, 10);   // 2. 显式指定 T=int
```

- `template <typename T>` 与 `template <class T>` **完全等价**；`typename` 更现代、语义更清晰。
- 可以有多个模板参数：`template <typename T, typename U>`。
- 模板参数也可以"不是类型"，而是编译期常量 —— 见下文「非类型模板参数」。

```c++
#include <iostream>

template <typename T>
T add(T a, T b) { return a + b; }

int main() {
    std::cout << add(10, 20) << '\n';            // 30（自动推导 T=int）
    std::cout << add<double>(1.5, 2.5) << '\n';  // 4（显式指定 T=double）
    return 0;
}
```

输出：

```text
30
4
```

**函数模板使用时，自动推导的 T 类型必须一致。**

**模板必须确定 T 的数据类型才能使用，包括无形参和返回值的函数。**

**只有在显式指定类型时函数模板才会发生隐式类型转换。**

**返回类型自动推导（C++14）**：用 `auto` 让返回类型由表达式推导，可在参数类型不同的情况下工作。

```c++
template <typename T, typename U>
auto add2(T a, U b) { return a + b; }   // C++14

add2(1, 2.5);   // 返回 double：3.5
```

##### 调用规则

**如果函数模板与普通函数都可以调用，优先调用普通函数。**

**允许使用空模板参数列表强制调用函数模板，即显式指定类型为空。**

**函数模板允许发生函数重载。**

**如果函数模板更能匹配传入参数，优先调用函数模板。**

尽量避免同时使用普通函数，防止二义性。

```c++
#include <iostream>

void f(int) { std::cout << "普通函数 f(int)\n"; }

template <typename T>
void f(T) { std::cout << "模板 f(T)\n"; }

int main() {
    f(1);     // 普通函数优先
    f<>(1);   // 空模板参数列表：强制调用模板
    f(1.5);   // 模板更匹配（普通函数需类型转换）
    return 0;
}
```

输出：

```text
普通函数 f(int)
模板 f(T)
模板 f(T)
```

##### 局限性

函数模板不能实现数组和具体类的模板等，必须**具体化（全特化）**实现。

```c++
#include <iostream>
#include <cstring>

template <typename T>
bool equal_val(T a, T b) { return a == b; }

// 全特化：对 const char* 按字符串内容比较
template <>
bool equal_val<const char*>(const char* a, const char* b) {
    return std::strcmp(a, b) == 0;
}

int main() {
    std::cout << equal_val(1, 1) << '\n';           // 1
    std::cout << equal_val("abc", "abc") << '\n';   // 1（走特化，按内容比较）
    return 0;
}
```

输出：

```text
1
1
```

> 全特化写法：`template <> 返回类型 函数名<具体类型>(参数...) { ... }`，具体化版本优先被调用。

#### 模板类

建立一个包含虚拟类型成员的类。

##### 语法

```c++
// class 允许用 typename 代替（此处二者等价）
template <class T, class P = bool>   // P 有默认类型
class A {
public:
    T a1;
    P a2;
};

// 使用：
A<int> obj;          // T=int, P=bool
A<int, char> obj2;   // T=int, P=char
```

**模板类没有自动类型推导**（必须写出 `A<int>`，不能像函数模板那样省略）。

**允许在模板参数列表中指定默认值**：

```c++
template <class T = int, class P = bool>
class A {
public:
    T a1;
    P a2;
};
A<> obj;   // 等价于 A<int, bool>
```

> 注意：类型模板参数的默认值是**类型**（如 `int`），不是数值；若要数值，需用**非类型模板参数** `template <class T, int N = 10>`。

**类型默认值 + 非类型默认值**示例：

```c++
#include <iostream>

template <class T = int, int N = 3>
class Array {
    T data_[N]{};
public:
    int size() const { return N; }
    T& operator[](int i) { return data_[i]; }
};

int main() {
    Array<> arr;             // T=int, N=3
    arr[0] = 42;
    std::cout << arr.size() << ' ' << arr[0] << '\n';   // 3 42
    return 0;
}
```

输出：

```text
3 42
```

##### 模板类中的成员函数

模板类中，泛型对象的成员函数在**调用时**才被创建（**按需实例化，lazy instantiation**）。

即可以在创建模板时让一个泛型对象调用不同类的成员函数，调用指定类型时才确定可以调用的成员函数。

> 只有真正被调用的成员函数才会被实例化并检查错误；未被调用到的成员即使对该类型非法也不会报错。

##### 模板类对象作函数参数

有 3 种方式：

```c++
template <class T>
class Test {
public:
    T a;
};

// 1. 指定模板类型：只接受 Test<int>
void testa(const Test<int>& t) {}

// 2. 参数模板化：接受任意 Test<T>
template <class T>
void testb(const Test<T>& t) {}

// 3. 整个类型模板化：接受任意类型
template <class T>
void testc(const T& t) {}
```

> `typeid(t).name()` 可在运行时查看实际类型名（需 `#include <typeinfo>`）。

##### 模板类继承

子类在继承模板类的时候必须指定模板类中泛型的数据类型。

子类变为类模板时，继承的类模板可以指定为泛型。

```c++
#include <iostream>

template <class T>
class Base {
protected:
    T v_;
public:
    Base(T v) : v_(v) {}
    void show() const { std::cout << v_ << '\n'; }
};

// 1) 子类指定基类的具体类型
class DerivedInt : public Base<int> {
public:
    DerivedInt(int v) : Base<int>(v) {}
};

// 2) 子类本身仍是模板，基类类型随之泛化
template <class T>
class DerivedT : public Base<T> {
public:
    DerivedT(T v) : Base<T>(v) {}
};

int main() {
    DerivedInt a(1);         a.show();   // 1
    DerivedT<double> b(2.5); b.show();   // 2.5
    return 0;
}
```

输出：

```text
1
2.5
```

##### 类模板成员函数的类外实现

```c++
template <class T>
class Test {
    T a_;
public:
    Test(T a);       // 构造函数声明
    void show();     // 成员函数声明
};

// 类外实现构造函数
template <class T>
Test<T>::Test(T a) : a_(a) {}

// 类外实现成员函数
template <class T>
void Test<T>::show() { std::cout << a_ << '\n'; }
```

完整示例（可运行）：

```c++
#include <iostream>

template <class T>
class Test {
    T a_;
public:
    Test(T a);
    void show();
};

template <class T>
Test<T>::Test(T a) : a_(a) {}

template <class T>
void Test<T>::show() { std::cout << a_ << '\n'; }

int main() {
    Test<int> t(7);
    t.show();        // 7
    return 0;
}
```

输出：

```text
7
```

**分文件编写类模板时，成员函数在编写时链接不到。**

**解决：分文件时直接包含 `.cpp` 源文件，或将 `.h` 头文件与 `.cpp` 中的定义写到一起改为 `.hpp` 文件。**

> 根因见下文「模板实例化与分离编译」：模板只有被使用（实例化）时才生成代码。

##### 类模板与友元

将全局函数变为类模板友元，分为**类内实现**与**类外实现**。

**类内实现**（每个 `Box<T>` 各有一个友元）：

```c++
#include <iostream>

template <class T>
class Box {
    T v_;
public:
    explicit Box(T v) : v_(v) {}
    friend std::ostream& operator<<(std::ostream& os, const Box<T>& b) {
        return os << "Box(" << b.v_ << ")";
    }
};

int main() {
    Box<int> b(5);
    std::cout << b << '\n';      // Box(5)
    return 0;
}
```

输出：

```text
Box(5)
```

**类外实现**（先在类内声明友元函数模板，再在类外定义）：

```c++
#include <iostream>

template <class T>
class Box {
    T v_;
public:
    explicit Box(T v) : v_(v) {}
    template <class U>
    friend std::ostream& operator<<(std::ostream& os, const Box<U>& b);
};

template <class U>
std::ostream& operator<<(std::ostream& os, const Box<U>& b) {
    return os << "Box(" << b.v_ << ")";
}

int main() {
    Box<int> b(5);
    std::cout << b << '\n';      // Box(5)
    return 0;
}
```

输出：

```text
Box(5)
```

#### 非类型模板参数

模板参数不必是类型，也可以是**整型、枚举、指针、引用等编译期常量**；它们直接参与代码生成（如数组长度）。

```c++
#include <iostream>

template <typename T, int N>
T scale(T x) { return x * N; }

int main() {
    std::cout << scale<int, 3>(10) << '\n';   // 30（N 是编译期常量 3）
    return 0;
}
```

输出：

```text
30
```

> 与类型模板参数一样可以有默认值：`template <class T, int N = 10>`（默认的是**数值**）。

**编译期计算示例**（非类型参数 + 类模板全特化）：

```c++
#include <iostream>

template <int N>
struct Factorial { static constexpr int value = N * Factorial<N - 1>::value; };

template <>
struct Factorial<0> { static constexpr int value = 1; };   // 全特化作为递归出口

int main() {
    std::cout << Factorial<5>::value << '\n';   // 120
    return 0;
}
```

输出：

```text
120
```

#### 模板特化与偏特化

- **全特化**：为某一组**具体类型**提供专门实现（如 `equal_val<const char*>`、`Factorial<0>`）。
- **偏特化（部分特化）**：模板有多个参数时，只固定其中一部分，或对参数形式作限定（如 `Pair<T, T>`、`Pair<T, U*>`）。**只有类模板允许偏特化**；函数模板只能全特化（可用重载替代）。

```c++
#include <iostream>

template <class T, class U>
struct Pair { void who() const { std::cout << "generic\n"; } };

template <class T>
struct Pair<T, T> { void who() const { std::cout << "same\n"; } };      // 两类型相同

template <class T, class U>
struct Pair<T, U*> { void who() const { std::cout << "pointer\n"; } };  // 第二参为指针

int main() {
    Pair<int, double>  a; a.who();   // generic
    Pair<int, int>     b; b.who();   // same
    Pair<int, double*> c; c.who();   // pointer
    return 0;
}
```

输出：

```text
generic
same
pointer
```

> 匹配优先级：**全特化 > 偏特化 > 主模板**。

#### 可变参数模板（Variadic Templates）

`class...` / `typename...` 表示**参数包**，可接受任意个参数；C++17 提供**折叠表达式**简化展开。

```c++
#include <iostream>

void print() { std::cout << '\n'; }          // 递归 / 展开的出口

template <class T, class... Rest>
void print(T first, Rest... rest) {
    std::cout << first;
    ((std::cout << ' ' << rest), ...);       // C++17 折叠表达式
    std::cout << '\n';
}

int main() {
    print(1, 2.5, "abc");   // 1 2.5 abc
    print(42);              // 42
    print();                // 空行
    return 0;
}
```

输出：

```text
1 2.5 abc
42

```

> `sizeof...(rest)` 可取得参数包中参数个数；`std::forward` + 参数包是完美转发的基础。

#### 别名模板（using）

用 `using` 给模板起别名，比 `typedef` 更直观，且**支持模板参数**。

```c++
#include <iostream>
#include <vector>
#include <map>
#include <string>

template <class T>
using Vec = std::vector<T>;              // 别名模板

template <class K, class V>
using Dict = std::map<K, V>;

int main() {
    Vec<int> v{1, 2, 3};
    Dict<std::string, int> d{{"a", 1}};
    std::cout << v.size() << ' ' << d.size() << '\n';   // 3 1
    return 0;
}
```

输出：

```text
3 1
```

> `typedef` 无法直接定义带模板参数的别名，故 C++11 起推荐用 `using`。

#### 模板实例化与分离编译

- **实例化**：编译器用具体类型（或常量）替换模板参数，生成真正的代码。
  - **隐式实例化**：代码用到哪个类型，就为它生成一份。
  - **显式实例化**：`template int add<int>(int, int);` 强制在某处生成。
- **分离编译问题**：模板定义必须对"使用点"可见。若函数 / 类模板的定义只放在 `.cpp`，而使用点在别的翻译单元，编译器看不到使用点、不会实例化，链接时就会**找不到符号**。
- **三种处理方式**：
  1. 把模板定义**全部写在头文件**（最常用）；
  2. 在某个 `.cpp` 里**显式实例化**需要的类型；
  3. 让头文件 `#include` 对应的 `.cpp`（或直接把声明与定义合并为 `.hpp`）。

> 这正是上文「类模板成员函数的类外实现」中"分文件链接不到"的根本原因。

### 文件操作

> 📖 官方文档（cppreference）：[文件输入/输出库](https://zh.cppreference.com/w/cpp/io/basic_ifstream) · [std::basic_ifstream](https://zh.cppreference.com/w/cpp/io/basic_ifstream) · [std::basic_ofstream](https://zh.cppreference.com/w/cpp/io/basic_ofstream) · [std::basic_fstream](https://zh.cppreference.com/w/cpp/io/basic_fstream) · [打开模式 ios_base::openmode](https://zh.cppreference.com/w/cpp/io/ios_base/openmode) · [文件系统库 filesystem](https://zh.cppreference.com/w/cpp/filesystem)
> 📖 官方文档（MSVC · Microsoft Learn）：[\<fstream\>](https://learn.microsoft.com/zh-cn/cpp/standard-library/fstream) · [basic_ifstream 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/basic-ifstream-class) · [basic_ofstream 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/basic-ofstream-class) · [basic_fstream 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/basic-fstream-class) · [ios_base 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/ios-base-class) · [\<filesystem\>](https://learn.microsoft.com/zh-cn/cpp/standard-library/filesystem)

**文件操作**通过 `<fstream>` 提供的**文件流**完成：把文件当成输入 / 输出流，用法与 `std::cin` / `std::cout` 一致。

#### 文件流概述

| 类 | 用途 | 默认打开模式 |
|---|---|---|
| `std::ifstream` | 读文件 | `ios::in` |
| `std::ofstream` | 写文件 | `ios::out` |
| `std::fstream` | 读写文件 | — |

三者都定义在 `<fstream>` 中，分别是 `basic_ifstream` / `basic_ofstream` / `basic_fstream` 的 typedef。

#### 打开与关闭

打开模式（可用 `|` 组合）：

| 模式 | 含义 |
|---|---|
| `std::ios::in` | 读 |
| `std::ios::out` | 写（默认会截断） |
| `std::ios::app` | 追加（写到文件末尾） |
| `std::ios::trunc` | 打开时清空（`out` 的默认行为） |
| `std::ios::ate` | 打开后立刻定位到末尾 |
| `std::ios::binary` | 二进制模式 |

```c++
std::ofstream out("data.txt", std::ios::app);   // 追加模式
if (!out) { /* 打开失败：文件不可写等 */ }
```

- 用 `is_open()` 或 `if (!stream)` 检查是否成功；**不要假设一定打开成功**。
- 文件流是 RAII 对象：**离开作用域自动关闭**，通常无需手动 `close()`（也可显式 `close()`）。

#### 文本文件读写

- **写入**：用 `<<`（与 `cout` 相同）。
- **读取**：用 `>>`（按**空白**分割，一次读一个"词"）或 `std::getline`（按**行**读取）。

```c++
#include <fstream>
#include <iostream>
#include <string>

int main() {
    {
        std::ofstream out("demo.txt");
        out << "hello\n" << 42 << '\n' << 3.14 << '\n';
    }   // 离开作用域，自动关闭

    std::ifstream in("demo.txt");
    std::string word;
    while (in >> word) std::cout << word << '\n';   // 按"词"读
    return 0;
}
```

输出：

```text
hello
42
3.14
```

**逐行读取**（推荐用 `getline`）：

```c++
#include <fstream>
#include <iostream>
#include <string>

int main() {
    {
        std::ofstream out("lines.txt");
        out << "line1\nline2\nline3\n";
    }

    std::ifstream in("lines.txt");
    std::string line;
    int n = 0;
    while (std::getline(in, line)) {     // 逐行读，读到结尾自动停止
        std::cout << ++n << ": " << line << '\n';
    }
    return 0;
}
```

输出：

```text
1: line1
2: line2
3: line3
```

#### 二进制文件读写

二进制模式（`std::ios::binary`）不做任何字符转换，用 `write` / `read` + `reinterpret_cast` 直接读写内存：

```c++
#include <fstream>
#include <iostream>

struct Point { int x; int y; };

int main() {
    Point p{3, 4};
    {
        std::ofstream out("p.bin", std::ios::binary);
        out.write(reinterpret_cast<const char*>(&p), sizeof(p));
    }

    Point q{};
    {
        std::ifstream in("p.bin", std::ios::binary);
        in.read(reinterpret_cast<char*>(&q), sizeof(q));
    }
    std::cout << q.x << ' ' << q.y << '\n';
    return 0;
}
```

输出：

```text
3 4
```

> `write` 需要 `const char*`、`read` 需要 `char*`，因此用 `reinterpret_cast` 转换；`sizeof(p)` 是要读写的字节数。

#### 文件定位与状态

| 操作 | 含义 |
|---|---|
| `seekg(pos)` / `seekp(pos)` | 读 / 写指针移动到**绝对**位置 |
| `seekg(off, dir)` / `seekp(off, dir)` | 相对定位（`beg` / `cur` / `end`） |
| `tellg()` / `tellp()` | 返回当前位置 |
| `eof()` | 是否到达文件末尾 |
| `fail()` | 是否发生错误 |
| `good()` | 状态是否正常 |
| `clear()` | 清除错误状态 |

```c++
#include <fstream>
#include <iostream>

int main() {
    {
        std::ofstream out("seek.txt");
        out << "ABCDEFGHIJ";
    }

    std::ifstream in("seek.txt");
    in.seekg(3);                          // 移动到偏移 3
    std::cout << char(in.get()) << '\n';  // D

    in.clear();                           // 清除状态
    in.seekg(0, std::ios::end);           // 定位到末尾
    std::cout << in.tellg() << '\n';      // 10
    return 0;
}
```

输出：

```text
D
10
```

> `seekg` / `tellg` 用于输入流（g = get），`seekp` / `tellp` 用于输出流（p = put）。

#### C++17 filesystem

`<filesystem>` 提供跨平台的路径与文件系统操作：

```c++
#include <filesystem>
#include <fstream>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    std::ofstream("fs_demo.txt") << "x";   // 写 1 字节

    std::cout << fs::exists("fs_demo.txt") << '\n';                    // 1
    std::cout << fs::file_size("fs_demo.txt") << '\n';                 // 1
    std::cout << (fs::path("a") / "b.txt").generic_string() << '\n';   // a/b.txt
    return 0;
}
```

输出：

```text
1
1
a/b.txt
```

| 常用 | 含义 |
|---|---|
| `fs::exists(p)` | 是否存在 |
| `fs::is_directory(p)` / `fs::is_regular_file(p)` | 类型判断 |
| `fs::file_size(p)` | 文件大小 |
| `fs::create_directory(p)` / `fs::remove(p)` | 创建 / 删除 |
| `fs::path("a") / "b.txt"` | 路径拼接（用 `/`） |
| `fs::directory_iterator(p)` | 遍历目录 |

#### 常见陷阱

- **不检查打开失败** → 后续读写静默无效；务必 `if (!f) { ... }`。
- 写模式 `out` 默认**截断**文件；要追加必须用 `std::ios::app`。
- `>>` 与 `std::getline` 混用：`>>` 会把换行符留在缓冲区，需 `in.ignore()` 处理。
- 二进制读写必须带 `std::ios::binary`，否则 Windows 下 `\n` 会被转换。
- 读取出错（如已到 eof）后再 `seekg`，需先 `clear()` 清除状态位。

## 2.重要的c++库

### STL

> 📖 官方文档（cppreference）：[容器库总览](https://zh.cppreference.com/w/cpp/container) · [算法库](https://zh.cppreference.com/w/cpp/algorithm) · [迭代器库](https://zh.cppreference.com/w/cpp/iterator) · [std::vector](https://zh.cppreference.com/w/cpp/container/vector) · [std::list](https://zh.cppreference.com/w/cpp/container/list) · [std::deque](https://zh.cppreference.com/w/cpp/container/deque)
> 📖 官方文档（MSVC · Microsoft Learn）：[C++ 标准库概述](https://learn.microsoft.com/zh-cn/cpp/standard-library/cpp-standard-library-overview) · [vector 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/vector-class) · [list 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/list-class) · [deque 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/deque-class) · [array 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/array-class-stl) · [basic_string 类](https://learn.microsoft.com/zh-cn/cpp/standard-library/basic-string-class)

**STL（Standard Template Library，标准模板库）** 是 C++ 标准库的核心部分，提供一套通用的**容器 + 算法 + 迭代器**，通过模板实现，与具体数据类型无关。

#### STL 概述

**六大组件**：

| 组件 | 作用 | 代表 |
|---|---|---|
| 容器（Containers） | 管理数据集合 | `vector`、`list`、`map`、`set` |
| 算法（Algorithms） | 对区间做操作 | `sort`、`find`、`copy` |
| 迭代器（Iterators） | 连接容器与算法的"泛型指针" | `begin()`、`end()`、`vector<int>::iterator` |
| 仿函数（Functors） | 可像函数一样调用的对象（谓词 / 比较器） | `std::less`、lambda |
| 适配器（Adaptors） | 修改组件接口 | `stack`、`queue`、`priority_queue` |
| 空间配置器（Allocators） | 管理内存分配 | `std::allocator` |

**容器分类**：

| 类别 | 特点 | 容器 |
|---|---|---|
| 序列容器 | 元素按位置线性存放 | `vector`、`array`、`deque`、`list`、`forward_list`、`string` |
| 关联容器 | 自动按 key 排序（红黑树） | `set`/`multiset`、`map`/`multimap` |
| 无序关联容器 | 哈希表，不排序 | `unordered_set`、`unordered_map`（及其 multi 版） |
| 容器适配器 | 基于其他容器封装的接口 | `stack`、`queue`、`priority_queue` |

**迭代器五类**（从弱到强）：

1. 输入迭代器（只读、单遍）
2. 输出迭代器（只写、单遍）
3. 前向迭代器（可读写、多遍）
4. 双向迭代器（可 `--`，如 `list`）
5. 随机访问迭代器（可 `+n` / `[i]`，如 `vector`、`deque`、`array`）

> 算法对迭代器类别有要求：如 `std::sort` 需要随机访问迭代器，`list` 不满足，因此要用其成员函数 `l.sort()`。

#### 序列容器

##### vector

**① 声明与初始化**

| 写法 | 含义 |
|---|---|
| `vector<int> v;` | 空容器 |
| `vector<int> v(n);` | `n` 个默认值（元素为 `0`） |
| `vector<int> v(n, val);` | `n` 个 `val` |
| `vector<int> v = {1, 2, 3};` | 列表初始化（C++11） |
| `vector<int> v(other);` | 拷贝构造 |
| `vector<int> v(first, last);` | 用迭代器区间 `[first, last)` 构造 |

```c++
#include <iostream>
#include <vector>
#include <cstddef>

int main() {
    std::vector<int> v1;                 // 空
    std::vector<int> v2(3);              // 3 个 0
    std::vector<int> v3(3, 7);           // 3 个 7
    std::vector<int> v4 = {1, 2, 3};     // 列表初始化
    std::vector<int> v5(v4);             // 拷贝构造
    std::vector<int> v6(v4.begin(), v4.begin() + 2);  // 迭代器区间

    auto print = [](const std::vector<int>& v) {
        std::cout << v.size() << ':';
        for (int x : v) std::cout << ' ' << x;
        std::cout << '\n';
    };
    print(v1);    // 0:
    print(v2);    // 3: 0 0 0
    print(v3);    // 3: 7 7 7
    print(v4);    // 3: 1 2 3
    print(v5);    // 3: 1 2 3
    print(v6);    // 2: 1 2
    return 0;
}
```

输出：

```text
0:
3: 0 0 0
3: 7 7 7
3: 1 2 3
3: 1 2 3
2: 1 2
```

**② 四种遍历方式**

```c++
#include <iostream>
#include <vector>
#include <cstddef>

int main() {
    std::vector<int> v = {10, 20, 30};

    for (std::size_t i = 0; i < v.size(); ++i) std::cout << v[i] << ' ';    // 下标
    std::cout << '\n';

    for (auto it = v.begin(); it != v.end(); ++it) std::cout << *it << ' '; // 迭代器
    std::cout << '\n';

    for (int x : v) std::cout << x << ' ';                                  // 范围 for
    std::cout << '\n';

    for (const int& x : v) std::cout << x << ' ';                           // const 引用（只读、免拷贝）
    std::cout << '\n';
    return 0;
}
```

输出：

```text
10 20 30
10 20 30
10 20 30
10 20 30
```

- **下标 `v[i]`**：最快，但**不检查越界**；
- **迭代器**：通用，便于配合算法；
- **范围 `for`**：最简洁；
- **`const int&`**：只读且避免拷贝，遍历大对象时推荐。

**③ 二维 vector**

```c++
#include <iostream>
#include <vector>

int main() {
    // 构造 3 行 4 列、初值为 0 的二维表
    std::vector<std::vector<int>> grid(3, std::vector<int>(4, 0));
    grid[1][2] = 9;

    for (const auto& row : grid) {
        for (int x : row) std::cout << x << ' ';
        std::cout << '\n';
    }
    std::cout << "rows=" << grid.size() << " cols=" << grid[0].size() << '\n';
    return 0;
}
```

输出：

```text
0 0 0 0
0 0 9 0
0 0 0 0
rows=3 cols=4
```

> 常用写法：`vector<vector<int>> grid(rows, vector<int>(cols, 初值));`。注意 `>>` 在 C++11 起可直接连写（旧标准需写成 `> >`）。

**动态数组**：连续内存，支持随机访问；是默认首选的容器。

```c++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {1, 2, 3};
    v.push_back(4);        // 尾部追加
    v.emplace_back(5);     // 原地构造（C++11，效率更高）

    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';     // 1 2 3 4 5

    std::cout << v.size() << ' ' << v.front() << ' ' << v.back() << '\n';  // 5 1 5
    v[0] = 10;             // [] 不检查越界
    std::cout << v.at(0) << '\n';   // 10（at 会检查越界并抛 out_of_range）
    return 0;
}
```

输出：

```text
1 2 3 4 5
5 1 5
10
```

**size 与 capacity**：`size` 是元素个数，`capacity` 是已分配空间可容纳的元素数；超出时按倍数扩容（元素被搬到新内存）。

```c++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;
    std::cout << v.capacity() << '\n';   // 0
    v.reserve(10);                       // 预留容量
    std::cout << v.capacity() << '\n';   // 10（>=10）
    for (int i = 0; i < 5; ++i) v.push_back(i);
    std::cout << v.size() << ' ' << v.capacity() << '\n';  // 5 10
    v.resize(8);                         // 改变元素个数（补默认值）
    std::cout << v.size() << '\n';       // 8
    return 0;
}
```

输出：

```text
0
10
5 10
8
```

**迭代器失效**：`vector` 扩容后，原有迭代器 / 指针 / 引用全部失效。用 `reserve` 可避免反复扩容：

```c++
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;
    v.reserve(100);                       // 预留容量，避免多次扩容
    int* p = v.data();
    for (int i = 0; i < 10; ++i) v.push_back(i);
    std::cout << (p == v.data()) << '\n'; // 1：容量足够，未重新分配
    return 0;
}
```

输出：

```text
1
```

> 失效规则：`push_back` / `insert` 可能使**全部**迭代器失效（扩容时）；`erase` 使**被删位置及其后**的迭代器失效。

**④ 常用操作速览**

增删改查 + 排序的连贯示例：

```c++
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v = {3, 1, 2};     // {3, 1, 2}

    v.push_back(4);                     // 尾插   → {3,1,2,4}
    v.insert(v.begin(), 0);             // 头插   → {0,3,1,2,4}
    v.erase(v.begin() + 1);             // 删索引1 → {0,1,2,4}
    v.pop_back();                       // 删尾   → {0,1,2}

    std::sort(v.begin(), v.end());      // 排序   → {0,1,2}
    auto it = std::find(v.begin(), v.end(), 2);

    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';                  // 0 1 2
    std::cout << (it != v.end()) << '\n';   // 1（找到）

    v.clear();
    std::cout << v.empty() << '\n';     // 1（已清空）
    return 0;
}
```

输出：

```text
0 1 2
1
1
```

**成员函数一览**：

| 成员 | 作用 | 参数 / 返回 / 备注 |
|---|---|---|
| `vector<T> v;` | 默认构造空容器 | — |
| `vector<T> v(n);` / `v(n, val);` | 构造 n 个元素（默认值 / val） | O(n) |
| `assign(n, val)` / `v = {…};` | 重填 / 赋值 | `assign` 先清空再填 |
| `at(i)` | 带越界检查的访问 | 越界抛 `std::out_of_range` |
| `operator[](i)` | 下标访问 | **不检查**越界，O(1) |
| `front()` / `back()` | 首 / 尾元素引用 | 空容器为 UB |
| `data()` | 指向底层数组的指针 | 可对接 C 接口 |
| `begin()` / `end()` | 首 / 尾后迭代器 | 另有 `cbegin` / `cend` |
| `rbegin()` / `rend()` | 反向迭代器 | — |
| `empty()` / `size()` | 是否为空 / 元素个数 | O(1) |
| `capacity()` | 当前可容纳的元素数 | — |
| `reserve(n)` | 预留至少 n 的容量 | 容量不足才重分配 |
| `shrink_to_fit()` | 收缩容量到 `size` | C++11，非强制 |
| `clear()` | 清空元素（容量不变） | O(n) |
| `push_back(x)` / `emplace_back(args…)` | 尾部追加 / 原地构造 | 均摊 O(1)，可能使迭代器失效 |
| `pop_back()` | 删除尾元素 | 空容器为 UB |
| `insert(pos, x)` / `emplace(pos, args…)` | 指定位置插入 | O(n) |
| `erase(pos)` / `erase(first, last)` | 删除元素 / 区间 | **返回下一个迭代器** |
| `resize(n)` / `resize(n, val)` | 改变元素个数 | 不足补默认值 / val |
| `swap(other)` | 交换两个容器 | O(1) |

##### array

**定长数组（C++11）**：大小在编译期确定，存在栈上，不退化、自带 `size()`。

```c++
#include <iostream>
#include <array>

int main() {
    std::array<int, 4> a = {1, 2, 3, 4};
    std::cout << a.size() << ' ' << a.front() << ' ' << a.back() << '\n';  // 4 1 4
    for (int x : a) std::cout << x << ' ';
    std::cout << '\n';   // 1 2 3 4
    return 0;
}
```

输出：

```text
4 1 4
1 2 3 4
```

**成员函数一览**：

| 成员 | 作用 | 参数 / 返回 / 备注 |
|---|---|---|
| `array<T, N> a;` / `a = {…};` | 定义 / 初始化 | 大小 `N` 编译期固定 |
| `at(i)` | 带越界检查的访问 | 越界抛 `out_of_range` |
| `operator[](i)` | 下标访问 | 不检查越界 |
| `front()` / `back()` | 首 / 尾元素 | — |
| `data()` | 指向首元素的指针 | 可传给 C 函数 |
| `begin()` / `end()` | 迭代器 | 另有 `cbegin` / `cend` |
| `empty()` | 是否为空（`N == 0`） | C++11，`constexpr` |
| `size()` / `max_size()` | 元素个数 / 最大值 | 均等于 `N` |
| `fill(val)` | 全部填 `val` | O(N) |
| `swap(other)` | 交换内容 | O(N) |
| `std::get<i>(a)` | 编译期下标访问 | `i` 必须是常量表达式 |
| `std::tuple_size<array<T,N>>::value` | 元素个数 | 等于 `N` |

##### deque

**双端队列**：两端都能高效插入 / 删除，支持随机访问（内存分段，非完全连续）。

```c++
#include <iostream>
#include <deque>

int main() {
    std::deque<int> d;
    d.push_back(2);
    d.push_back(3);
    d.push_front(1);      // 头部插入
    for (int x : d) std::cout << x << ' ';
    std::cout << '\n';    // 1 2 3
    std::cout << d.front() << ' ' << d.back() << '\n';  // 1 3
    return 0;
}
```

输出：

```text
1 2 3
1 3
```

**成员函数一览**：

| 成员 | 作用 | 参数 / 返回 / 备注 |
|---|---|---|
| 构造 / `assign` / `operator=` | 初始化 / 赋值 | 同 `vector` |
| `at(i)` / `operator[](i)` | 访问 | `at` 检查越界 |
| `front()` / `back()` | 首 / 尾元素 | — |
| `begin()` / `end()` / `rbegin()` / `rend()` | 迭代器 | **随机访问**迭代器 |
| `empty()` / `size()` | 大小 | O(1)；**无 `capacity()`** |
| `push_back(x)` / `emplace_back(…)` | 尾部追加 / 构造 | O(1) |
| `push_front(x)` / `emplace_front(…)` | 头部追加 / 构造 | O(1) |
| `pop_back()` / `pop_front()` | 删除尾 / 首 | O(1) |
| `insert` / `emplace` / `erase` | 插 / 删 | O(n) |
| `clear()` | 清空 | — |
| `resize(n)` / `resize(n, val)` | 改变元素个数 | — |
| `swap(other)` / `shrink_to_fit()` | 交换 / 收缩容量 | — |

##### list

**双向链表**：任意位置 `O(1)` 插入 / 删除；**不支持随机访问**；提供成员函数 `sort` / `reverse` / `splice` / `merge`。

```c++
#include <iostream>
#include <list>

int main() {
    std::list<int> l = {3, 1, 2};
    l.push_front(0);
    l.sort();                              // 链表用成员 sort（不能用 std::sort）
    for (int x : l) std::cout << x << ' ';
    std::cout << '\n';                     // 0 1 2 3
    l.reverse();
    std::cout << l.front() << ' ' << l.back() << '\n';  // 3 0
    return 0;
}
```

输出：

```text
0 1 2 3
3 0
```

**成员函数一览**：

| 成员 | 作用 | 参数 / 返回 / 备注 |
|---|---|---|
| 构造 / `assign` / `operator=` | 初始化 / 赋值 | — |
| `front()` / `back()` | 首 / 尾元素 | — |
| `begin()` / `end()` / `rbegin()` / `rend()` | 迭代器 | **双向**（不能 `+n`） |
| `empty()` / `size()` | 大小 | O(1) |
| `push_front(x)` / `pop_front()` / `emplace_front(…)` | 头部操作 | O(1) |
| `push_back(x)` / `pop_back()` / `emplace_back(…)` | 尾部操作 | O(1) |
| `insert` / `emplace` / `erase` | 插 / 删 | O(1)（已知位置） |
| `clear()` / `resize(n)` / `swap(other)` | 清空 / 改大小 / 交换 | — |
| `splice(pos, other)` | 把 `other` 的节点**移动**到 `*this` | 不拷贝元素，O(1) |
| `sort()` / `sort(cmp)` | 链表专用排序 | O(n log n) |
| `merge(other)` | 合并两个**有序**链表 | — |
| `reverse()` | 反转 | O(n) |
| `unique()` | 删除**相邻**重复元素 | 常先 `sort()` |
| `remove(val)` / `remove_if(pred)` | 删除等于 val / 满足条件的元素 | O(n) |

##### forward_list

**单向链表（C++11）**：只支持前向遍历，比 `list` 更省内存；用 `before_begin` / `insert_after` 等操作。

```c++
#include <iostream>
#include <forward_list>

int main() {
    std::forward_list<int> fl;
    fl.push_front(3);
    fl.push_front(2);
    fl.push_front(1);
    for (int x : fl) std::cout << x << ' ';
    std::cout << '\n';    // 1 2 3
    return 0;
}
```

输出：

```text
1 2 3
```

**成员函数一览**：

| 成员 | 作用 | 参数 / 返回 / 备注 |
|---|---|---|
| 构造 / `assign` / `operator=` | 初始化 / 赋值 | — |
| `front()` | 首元素 | — |
| `begin()` / `end()` / `cbegin()` / `cend()` | 迭代器 | **前向**（无反向迭代器） |
| `before_begin()` / `cbefore_begin()` | 首元素之前的"虚拟"位置 | 头插 / 头删的基础 |
| `empty()` / `max_size()` | 大小相关 | **无 `size()`** |
| `push_front(x)` / `pop_front()` / `emplace_front(…)` | 头部操作 | O(1) |
| `insert_after(pos, x)` / `emplace_after(pos, …)` | 在 `pos` 之后插入 | O(1) |
| `erase_after(pos)` / `erase_after(first, last)` | 删除 `pos` 之后的元素 | O(1) |
| `clear()` / `swap(other)` | 清空 / 交换 | — |
| `splice_after(pos, other)` | 移动节点 | O(1) |
| `sort()` / `merge(other)` / `reverse()` / `unique()` | 链表算法 | — |
| `remove(val)` / `remove_if(pred)` | 删除元素 | O(n) |

##### string

`std::string` 是 `std::basic_string<char>` 的别名，本质是"字符的动态数组"，用法与 `vector<char>` 类似。

```c++
#include <iostream>
#include <string>

int main() {
    std::string s = "hello";
    s += " world";                                  // 拼接
    std::cout << s << '\n';                         // hello world
    std::cout << s.size() << ' ' << s.substr(0, 5) << '\n';  // 11 hello
    std::cout << s.find("world") << '\n';           // 6
    return 0;
}
```

输出：

```text
hello world
11 hello
6
```

**成员函数一览**：

| 成员 | 作用 | 参数 / 返回 / 备注 |
|---|---|---|
| 构造 / `operator=` / `assign` | 初始化 / 赋值 | 支持 `"abc"`、`(n, c)`、`(other, pos, len)` |
| `size()` / `length()` / `empty()` | 长度 / 是否为空 | `size` 与 `length` 等价 |
| `capacity()` / `reserve(n)` / `shrink_to_fit()` | 容量管理 | — |
| `at(i)` / `operator[](i)` | 访问字符 | `at` 检查越界 |
| `front()` / `back()` | 首 / 尾字符 | — |
| `data()` / `c_str()` | 底层字符指针 | 与 C 接口交互 |
| `append(s)` / `operator+=` / `push_back(c)` | 追加 | — |
| `insert(pos, s)` / `erase(pos, len)` | 插 / 删 | — |
| `clear()` | 清空 | — |
| `substr(pos, len)` | 取子串 | 返回新 `string` |
| `find(s)` / `rfind(s)` | 查找（从前 / 从后） | 返回下标，失败为 `npos` |
| `compare(s)` | 比较 | `<0` / `0` / `>0` |
| `replace(pos, len, s)` | 替换 | — |
| `stoi` / `stod` / `stoi` / `to_string` | 字符串 ↔ 数值 | 定义在 `<string>` |

##### 序列容器对比

| 容器 | 内存 | 随机访问 | 头插/删 | 尾插/删 | 中间插/删 |
|---|---|---|---|---|---|
| `vector` | 连续 | ✅ | ❌ 慢 | ✅ 均摊 O(1) | ❌ O(n) |
| `array` | 连续、定长 | ✅ | ❌ | ❌ | ❌ |
| `deque` | 分段连续 | ✅ | ✅ O(1) | ✅ O(1) | O(n) |
| `list` | 双向链表 | ❌ | ✅ O(1) | ✅ O(1) | ✅ O(1) |
| `forward_list` | 单向链表 | ❌ | ✅ O(1) | ❌ | 其后 O(1) |

> 选择原则：**默认用 `vector`**；频繁头插 / 双端操作选 `deque`；频繁中间插删选 `list`；定长用 `array`。

#### 关联容器

关联容器基于**红黑树**（自平衡二叉搜索树），元素始终**按 key 有序**，查找 / 插入 / 删除均为 `O(log n)`。

| 容器 | 说明 |
|---|---|
| `set` | 有序、不重复的 key 集合 |
| `multiset` | 有序、允许重复 key |
| `map` | 有序的 key → value 映射，key 唯一 |
| `multimap` | 有序、允许重复 key 的映射 |

**区间操作**：`lower_bound`（第一个 ≥ k）、`upper_bound`（第一个 > k）、`equal_range`（两者的区间）。

```c++
#include <iostream>
#include <map>

int main() {
    std::map<int, char> m = {{1,'a'},{3,'c'},{5,'e'},{7,'g'}};
    auto lo = m.lower_bound(3);   // 第一个 >= 3
    auto hi = m.upper_bound(5);   // 第一个 > 5
    std::cout << lo->first << ' ' << hi->first << '\n';   // 3 7

    auto range = m.equal_range(3);   // [lower_bound, upper_bound)
    std::cout << range.first->first << ' ' << range.second->first << '\n';  // 3 5
    return 0;
}
```

输出：

```text
3 7
3 5
```

**自定义比较器**（默认 `std::less`，即升序）：

```c++
#include <iostream>
#include <set>

struct Desc {
    bool operator()(int a, int b) const { return a > b; }   // 降序
};

int main() {
    std::set<int, Desc> s = {3, 1, 2};
    for (int x : s) std::cout << x << ' ';
    std::cout << '\n';       // 3 2 1
    return 0;
}
```

输出：

```text
3 2 1
```

> `map` 同理：`std::map<K, V, Compare>`。

##### set

**有序、去重**的集合：插入重复元素会被忽略，遍历时按 key 升序。

**成员函数一览**：

| 成员 | 作用 | 备注 |
|---|---|---|
| `set<T> s;` / `s = {…};` | 构造 / 初始化 | 自动排序、去重 |
| `insert(x)` / `emplace(args…)` | 插入 | 返回 `pair<迭代器, bool>`（bool 表示是否新插入） |
| `erase(k)` | 按 key 删除 | 返回删除个数（0 / 1） |
| `erase(it)` / `erase(first, last)` | 按迭代器 / 区间删除 | 返回下一个迭代器 |
| `find(k)` | 查找 | 失败返回 `end()` |
| `count(k)` | 计数 | `set` 中恒为 0 或 1 |
| `lower_bound(k)` / `upper_bound(k)` / `equal_range(k)` | 区间操作 | — |
| `empty()` / `size()` | 大小 | O(1) |
| `begin()` / `end()` | 迭代器 | **双向**，按序 |
| `clear()` / `swap(other)` | 清空 / 交换 | — |

```c++
#include <iostream>
#include <set>

int main() {
    std::set<int> s = {3, 1, 2, 2, 1};   // 自动去重、排序
    for (int x : s) std::cout << x << ' ';
    std::cout << '\n';                   // 1 2 3

    s.insert(5);
    std::cout << s.count(3) << ' ' << s.count(9) << '\n';   // 1 0

    auto it = s.find(2);
    if (it != s.end()) std::cout << "found " << *it << '\n';  // found 2
    return 0;
}
```

输出：

```text
1 2 3
1 0
found 2
```

##### multiset

**有序、允许重复 key** 的集合。接口与 `set` 几乎相同，差别在于：

- `insert` **总是成功**（不因重复而失败）；
- `count(k)` 返回等于 k 的元素个数（可能 > 1）；
- `erase(k)` 删除**全部**等于 k 的元素（按迭代器删则只删一个）；
- `equal_range(k)` 返回所有等于 k 的区间。

```c++
#include <iostream>
#include <set>
#include <iterator>

int main() {
    std::multiset<int> ms = {1, 2, 2, 3, 3, 3};
    std::cout << ms.count(2) << ' ' << ms.count(3) << '\n';   // 2 3
    auto range = ms.equal_range(3);
    std::cout << std::distance(range.first, range.second) << '\n';  // 3
    return 0;
}
```

输出：

```text
2 3
3
```

##### map

**有序的 key → value 映射**，key 唯一，遍历时按 key 升序。

**成员函数一览**：

| 成员 | 作用 | 备注 |
|---|---|---|
| `map<K,V> m;` / `m = {{k,v},…};` | 构造 / 初始化 | 按 key 排序 |
| `operator[](k)` | 访问；key 不存在则**插入**默认值 | 只读场景避免使用 |
| `at(k)` | 带检查访问 | 不存在抛 `out_of_range` |
| `insert({k,v})` / `emplace(k,v)` | 插入 | 返回 `pair<迭代器, bool>` |
| `erase(k)` / `erase(it)` | 删除 | — |
| `find(k)` / `count(k)` | 查找 / 计数 | — |
| `lower_bound` / `upper_bound` / `equal_range` | 区间操作 | — |
| `empty()` / `size()` | 大小 | O(1) |
| `begin()` / `end()` | 迭代器 | 双向，按 key 有序 |
| `clear()` / `swap(other)` | 清空 / 交换 | — |

```c++
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> age;
    age["Tom"] = 18;
    age["Ann"] = 20;                 // 按 key 自动排序
    age.insert({"Bob", 22});
    age.emplace("Zoe", 19);

    for (const auto& [name, a] : age)   // C++17 结构化绑定
        std::cout << name << ':' << a << ' ';
    std::cout << '\n';   // Ann:20 Bob:22 Tom:18 Zoe:19

    std::cout << age.at("Tom") << ' ' << age.count("Sam") << '\n';  // 18 0
    return 0;
}
```

输出：

```text
Ann:20 Bob:22 Tom:18 Zoe:19
18 0
```

> `operator[]` 在 key 不存在时会**插入**默认值；只读访问用 `at()`（不存在抛 `out_of_range`）或 `find()`。

##### multimap

**有序、允许重复 key** 的映射。与 `map` 的差别：

- **没有 `operator[]`**（key 不唯一，无法确定取哪个值）；
- `insert` 总是成功；`count` / `equal_range` 可返回多个；
- 用 `equal_range(k)` 遍历同一 key 的所有值。

```c++
#include <iostream>
#include <map>
#include <string>

int main() {
    std::multimap<std::string, int> mm;
    mm.insert({"Tom", 90});
    mm.insert({"Tom", 85});
    mm.insert({"Ann", 88});
    std::cout << mm.count("Tom") << '\n';   // 2
    auto range = mm.equal_range("Tom");
    for (auto it = range.first; it != range.second; ++it)
        std::cout << it->first << ':' << it->second << ' ';
    std::cout << '\n';   // Tom:90 Tom:85
    return 0;
}
```

输出：

```text
2
Tom:90 Tom:85
```

#### 无序关联容器

无序关联容器基于**哈希表**，元素**不排序**，平均查找 `O(1)`（最坏 `O(n)`）。

| 容器 | 说明 |
|---|---|
| `unordered_set` / `unordered_multiset` | 哈希集合（去重 / 允许重复） |
| `unordered_map` / `unordered_multimap` | 哈希映射（key 唯一 / 允许重复） |

**哈希相关操作**：

- `bucket_count()`：桶的数量
- `load_factor()`：负载因子 = 元素数 / 桶数
- `rehash(n)` / `reserve(n)`：调整桶数 / 预留元素数
- `max_load_factor()`：最大负载因子（超出会触发 rehash）

```c++
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> s;
    s.rehash(50);
    std::cout << (s.bucket_count() >= 50) << '\n';   // 1
    for (int i = 0; i < 10; ++i) s.insert(i);
    std::cout << s.size() << '\n';                    // 10
    std::cout << (s.load_factor() <= s.max_load_factor()) << '\n';  // 1
    return 0;
}
```

输出：

```text
1
10
1
```

> 与有序容器对比：`map` 有序、`O(log n)`；`unordered_map` 无序、平均 `O(1)`。需要**有序遍历 / 区间查询**用 `map`，只需**快速查找**用 `unordered_map`。

##### unordered_set

**去重、不排序**的哈希集合。

**成员函数一览**：

| 成员 | 作用 | 备注 |
|---|---|---|
| `unordered_set<T> s;` / `s = {…};` | 构造 / 初始化 | 去重、**不排序** |
| `insert(x)` / `emplace(args…)` | 插入 | 返回 `pair<迭代器, bool>` |
| `erase(k)` / `erase(it)` | 删除 | — |
| `find(k)` / `count(k)` | 查找 / 计数 | 平均 O(1) |
| `empty()` / `size()` | 大小 | — |
| `bucket_count()` / `load_factor()` | 桶数 / 负载因子 | — |
| `rehash(n)` / `reserve(n)` | 调整桶数 / 预留元素数 | — |
| `begin()` / `end()` | 迭代器 | **前向**，顺序不定 |
| `clear()` / `swap(other)` | 清空 / 交换 | — |

```c++
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> us = {3, 1, 2, 2, 1};   // 去重，不排序
    std::cout << us.size() << '\n';                 // 3
    std::cout << us.count(2) << ' ' << us.count(9) << '\n';   // 1 0
    return 0;
}
```

输出：

```text
3
1 0
```

##### unordered_multiset

**允许重复 key、不排序**的哈希集合。与 `unordered_set` 的差别同 `multiset`：

- `insert` 总是成功；
- `count(k)` 返回等于 k 的个数（可 > 1）；
- `erase(k)` 删除**全部**等于 k 的元素；
- `equal_range(k)` 返回所有等于 k 的区间。

```c++
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_multiset<int> ums = {1, 2, 2, 3, 3, 3};
    std::cout << ums.count(2) << ' ' << ums.count(3) << '\n';   // 2 3
    std::cout << ums.size() << '\n';                            // 6
    return 0;
}
```

输出：

```text
2 3
6
```

##### unordered_map

**不排序**的键值映射。

**成员函数一览**：

| 成员 | 作用 | 备注 |
|---|---|---|
| `unordered_map<K,V> m;` / `m = {{k,v},…};` | 构造 / 初始化 | 不排序 |
| `operator[](k)` | 访问；key 不存在则**插入**默认值 | — |
| `at(k)` | 带检查访问 | 不存在抛 `out_of_range` |
| `insert({k,v})` / `emplace(k,v)` | 插入 | 返回 `pair<迭代器, bool>` |
| `erase(k)` / `erase(it)` | 删除 | — |
| `find(k)` / `count(k)` | 查找 / 计数 | 平均 O(1) |
| `bucket_count()` / `load_factor()` / `rehash(n)` / `reserve(n)` | 哈希管理 | — |
| `empty()` / `size()` | 大小 | — |
| `begin()` / `end()` | 迭代器 | 前向，顺序不定 |
| `clear()` / `swap(other)` | 清空 / 交换 | — |

```c++
#include <iostream>
#include <unordered_map>
#include <string>

int main() {
    std::unordered_map<std::string, int> m;
    m["apple"] = 1;
    m["banana"] = 2;
    m["cherry"] = 3;
    std::cout << m.at("banana") << '\n';   // 2
    std::cout << m.count("apple") << '\n'; // 1
    std::cout << m.size() << '\n';         // 3
    return 0;
}
```

输出：

```text
2
1
3
```

##### unordered_multimap

**允许重复 key、不排序**的映射。与 `unordered_map` 的差别：**无 `operator[]`**，用 `count` / `equal_range` 处理同一 key 的多个值。

```c++
#include <iostream>
#include <unordered_map>
#include <string>
#include <iterator>

int main() {
    std::unordered_multimap<std::string, int> umm;
    umm.insert({"Tom", 90});
    umm.insert({"Tom", 85});
    umm.insert({"Ann", 88});
    std::cout << umm.count("Tom") << '\n';                              // 2
    auto range = umm.equal_range("Tom");
    std::cout << std::distance(range.first, range.second) << '\n';      // 2
    return 0;
}
```

输出：

```text
2
2
```

#### 容器适配器

适配器**基于已有容器**封装出特定接口，**不提供迭代器**。

| 适配器 | 底层默认容器 | 接口 |
|---|---|---|
| `stack` | `deque` | LIFO（后进先出）：`push` / `pop` / `top` |
| `queue` | `deque` | FIFO（先进先出）：`push` / `pop` / `front` / `back` |
| `priority_queue` | `vector` | 优先级队列：`push` / `pop` / `top`（默认大顶堆） |

##### stack

**后进先出（LIFO）**。

**成员函数一览**：

| 成员 | 作用 | 备注 |
|---|---|---|
| `stack<T> st;` | 构造 | 默认底层 `deque` |
| `push(x)` / `emplace(args…)` | 入栈 | — |
| `pop()` | 出栈（**不返回**元素） | 需先 `top()` 取值 |
| `top()` | 栈顶元素 | 空栈为 UB |
| `empty()` / `size()` | 大小 | — |
| `swap(other)` | 交换 | — |

```c++
#include <iostream>
#include <stack>

int main() {
    std::stack<int> st;
    st.push(1);
    st.push(2);
    st.push(3);
    std::cout << st.top() << ' ' << st.size() << '\n';   // 3 3
    st.pop();
    std::cout << st.top() << '\n';                       // 2
    return 0;
}
```

输出：

```text
3 3
2
```

##### queue

**先进先出（FIFO）**。

**成员函数一览**：

| 成员 | 作用 | 备注 |
|---|---|---|
| `queue<T> q;` | 构造 | 默认底层 `deque` |
| `push(x)` / `emplace(args…)` | 入队 | — |
| `pop()` | 出队（**不返回**元素） | 需先 `front()` 取值 |
| `front()` / `back()` | 队首 / 队尾 | — |
| `empty()` / `size()` | 大小 | — |
| `swap(other)` | 交换 | — |

```c++
#include <iostream>
#include <queue>

int main() {
    std::queue<int> q;
    q.push(1);
    q.push(2);
    q.push(3);
    std::cout << q.front() << ' ' << q.back() << '\n';   // 1 3
    q.pop();
    std::cout << q.front() << '\n';                      // 2
    return 0;
}
```

输出：

```text
1 3
2
```

##### priority_queue

**优先级队列**，默认**大顶堆**（`top()` 是最大元素）。

 **成员函数一览**：

| 成员 | 作用 | 备注 |
|---|---|---|
| `priority_queue<T> pq;` | 构造（默认大顶堆） | 默认底层 `vector` |
| `priority_queue<T, Container, Compare>` | 指定底层容器 / 比较器 | `Compare` 默认 `less<T>` |
| `push(x)` / `emplace(args…)` | 入堆 | O(log n) |
| `pop()` | 弹出堆顶 | O(log n) |
| `top()` | 堆顶元素 | 按 `Compare` 取最值 |
| `empty()` / `size()` | 大小 | — |
| `swap(other)` | 交换 | — |

```c++
#include <iostream>
#include <queue>
#include <vector>
#include <functional>

int main() {
    std::priority_queue<int> pq;          // 默认大顶堆
    for (int x : {3, 1, 4, 1, 5, 9, 2, 6}) pq.push(x);
    std::cout << pq.top() << '\n';        // 9

    std::priority_queue<int, std::vector<int>, std::greater<int>> minq;  // 小顶堆
    for (int x : {3, 1, 4, 1, 5, 9, 2, 6}) minq.push(x);
    std::cout << minq.top() << '\n';      // 1
    return 0;
}
```

输出：

```text
9
1
```

#### 迭代器

> 📖 官方文档（cppreference）：[迭代器库（\<iterator\>）](https://zh.cppreference.com/w/cpp/iterator) · [迭代器概念 LegacyIterator](https://zh.cppreference.com/w/cpp/named_req/Iterator) · [iterator_traits](https://zh.cppreference.com/w/cpp/iterator/iterator_traits) · [迭代器类别标签](https://zh.cppreference.com/w/cpp/iterator/iterator_tags)
> 📖 官方文档（MSVC · Microsoft Learn）：[迭代器](https://learn.microsoft.com/zh-cn/cpp/standard-library/iterators) · [\<iterator\>](https://learn.microsoft.com/zh-cn/cpp/standard-library/iterator) · [iterator_traits 结构](https://learn.microsoft.com/zh-cn/cpp/standard-library/iterator-traits-struct) · [迭代器函数](https://learn.microsoft.com/zh-cn/cpp/standard-library/iterator-functions) · [迭代器概念（C++20）](https://learn.microsoft.com/zh-cn/cpp/standard-library/iterator-concepts)

**迭代器（Iterator）** 是对「指针」的泛化抽象：它让**算法**无需了解**容器**的内部结构，只通过 `[begin, end)` 半开区间操作元素——这是 STL「容器与算法解耦」的关键。

##### 一、迭代器底层定义

所有迭代器都首先满足最基础的 **`LegacyIterator`** 要求：

- 满足 `CopyConstructible`、`CopyAssignable`、`Destructible`、`Swappable`；
- `std::iterator_traits<It>` 提供 `value_type` / `difference_type` / `reference` / `pointer` / `iterator_category` 这 5 个成员。

必需表达式（`r` 为 `It` 类型左值）：

| 表达式 | 返回类型 | 语义 |
|---|---|---|
| `*r` | 未指定（可解引用） | 取当前元素 |
| `++r` | `It&` | 前进到下一元素 |

**`std::iterator_traits<It>` 的 5 个关联类型**：

| 成员类型 | 含义 |
|---|---|
| `value_type` | 元素类型 |
| `difference_type` | 迭代器差值类型（有符号整数） |
| `reference` | `*it` 的类型 |
| `pointer` | `it->` 的类型 |
| `iterator_category` | 所属迭代器类别标签 |

```c++
#include <iostream>
#include <iterator>
#include <type_traits>
#include <vector>
#include <list>

int main() {
    using VI = std::vector<int>::iterator;
    using LI = std::list<int>::iterator;
    std::cout << std::is_same<std::iterator_traits<VI>::iterator_category,
                              std::random_access_iterator_tag>::value << '\n';  // 1
    std::cout << std::is_same<std::iterator_traits<LI>::iterator_category,
                              std::bidirectional_iterator_tag>::value << '\n';  // 1
    std::cout << std::is_same<std::iterator_traits<int*>::value_type, int>::value << '\n';  // 1
    return 0;
}
```

输出：

```text
1
1
1
```

> `std::iterator` 基结构（C++17 起**弃用**）只是为省去手写这 5 个 typedef，现已被 `iterator_traits` 取代。

##### 二、五类迭代器

| 类别 | 能力 | 代表容器 / 类型 |
|---|---|---|
| 输入 | 只读、单遍 | `istream_iterator` |
| 输出 | 只写、单遍 | `ostream_iterator`、插入迭代器 |
| 前向 | 读写、多遍 | `forward_list`、`unordered_*` |
| 双向 | 可前可后 | `list`、`set`/`map`/`multiset`/`multimap` |
| 随机访问 | 常数时间跳转 | `vector`、`deque`、`array`、`string` |

> 约定：以下每类**只列相对上一类的新增操作**；未列出的操作默认继承自下层类别。

###### 1. 输入迭代器（Input Iterator）

在 `LegacyIterator` 之上新增：满足 `EqualityComparable`；**单遍**（递增后旧副本可能失效）。

| 新增表达式 | 返回 / 类型 | 语义 |
|---|---|---|
| `i == j` / `i != j` | 可转 `bool` | 比较 |
| `*i` | 可转 `value_type` | 读当前元素 |
| `i->m` | — | 等价 `(*i).m` |
| `r++` | 可转 `const X&` | 前进（后置） |
| `*r++` | 可转 `value_type` | 读并前进 |

**属于该类的容器 / 类型**：`std::istream_iterator`、`std::istreambuf_iterator`。

###### 2. 输出迭代器（Output Iterator）

在 `LegacyIterator` 之上新增：**只写、单遍**，不要求比较、不能回读。

| 新增表达式 | 语义 |
|---|---|
| `*r = o` | 写入值 `o` |
| `r++` | 前进（后置） |
| `*r++ = o` | 写入并前进 |

**属于该类的容器 / 类型**：`std::ostream_iterator`、`std::ostreambuf_iterator`、`std::back_insert_iterator`、`std::front_insert_iterator`、`std::insert_iterator`。

###### 3. 前向迭代器（Forward Iterator）

在 `LegacyInputIterator` 之上新增（**无新表达式**）：

- 满足 `DefaultConstructible`（可默认构造「空」迭代器）；
- **多遍（multi-pass）保证**：多个副本可各自独立解引用与递增；
- `reference` 必须是**真正的引用**（`T&` / `const T&`），不能是代理对象。

**属于该类的容器 / 类型**：`std::forward_list`、`std::unordered_set`、`std::unordered_multiset`、`std::unordered_map`、`std::unordered_multimap`。

###### 4. 双向迭代器（Bidirectional Iterator）

在 `LegacyForwardIterator` 之上新增：

| 新增表达式 | 返回 / 类型 | 语义 |
|---|---|---|
| `--r` | `X&` | 后退 |
| `r--` | 可转 `const X&` | 后退（后置） |
| `*r--` | `reference` | 读并后退 |

**属于该类的容器 / 类型**：`std::list`、`std::set`、`std::multiset`、`std::map`、`std::multimap`（以及 `std::filesystem::path::iterator`）。

###### 5. 随机访问迭代器（Random Access Iterator）

在 `LegacyBidirectionalIterator` 之上新增（全部 **O(1)**）：

| 新增表达式 | 返回 / 类型 | 语义 |
|---|---|---|
| `r += n` / `r -= n` | `X&` | 原地跳转 |
| `r + n` / `n + r` / `r - n` | `X` | 返回跳转后的迭代器 |
| `b - a` | `difference_type` | 两迭代器距离 |
| `a[n]` | 可转 `reference` | 等价 `*(a + n)` |
| `i < j` / `i > j` / `i <= j` / `i >= j` | 可转 `bool` | 全序比较 |

**属于该类的容器 / 类型**：`std::vector`、`std::deque`、`std::array`、`std::string`、`std::string_view`、`std::span`（C++20）；**裸指针 `T*`** 也是随机访问迭代器。

```c++
#include <iostream>
#include <vector>
#include <list>
#include <forward_list>
#include <iterator>

int main() {
    // 随机访问：vector 支持 +、[]、-、比较
    std::vector<int> v = {10, 20, 30, 40, 50};
    auto a = v.begin();
    std::cout << *(a + 2) << ' ' << a[3] << ' ' << (v.end() - v.begin()) << ' '
              << (a < v.end()) << '\n';   // 30 40 5 1

    // 双向：list 支持 --
    std::list<int> l = {1, 2, 3};
    auto it = l.end();
    --it;
    std::cout << *it << '\n';             // 3

    // 前向：forward_list 只支持 ++
    std::forward_list<int> fl = {7, 8, 9};
    auto fit = fl.begin();
    ++fit;
    std::cout << *fit << '\n';            // 8
    return 0;
}
```

输出：

```text
30 40 5 1
3
8
```

> 易错点：`forward_list` 与四个 `unordered_*` 只有**前向**迭代器；`deque` 是随机访问但**非连续**（不满足 C++20 `contiguous_iterator`）。

##### 三、迭代器适配器（Iterator Adaptors）

适配器**基于已有迭代器**改变其行为：

| 适配器 | 用途 | 注意事项 |
|---|---|---|
| `std::reverse_iterator` | 反向遍历（`++` 映射为底层 `--`） | 需底层为双向 / 随机访问；`base()` 取底层迭代器 |
| `std::back_insert_iterator` | 输出：`*i = x` → `push_back(x)` | 需容器有 `push_back` |
| `std::front_insert_iterator` | 输出：`*i = x` → `push_front(x)` | 需容器有 `push_front`（`vector` 不行） |
| `std::insert_iterator` | 输出：`*i = x` → `insert(pos, x)` | 构造时给定插入位置 |
| `std::move_iterator` | `*i` 变为右值（移动语义） | 由 `make_move_iterator` 生成 |
| `std::istream_iterator` | 从输入流读取 | 默认构造 = 流末尾哨兵 |
| `std::ostream_iterator` | 向输出流写入 | 可指定分隔符 |

配套工厂函数：`std::back_inserter(c)` / `std::front_inserter(c)` / `std::inserter(c, pos)` / `std::make_move_iterator(it)`。

C++20/23 新增：`std::common_iterator`、`std::counted_iterator`、`std::move_sentinel`；**C++23** `std::basic_const_iterator` / `std::const_iterator`（把任意迭代器适配为只读）。

```c++
#include <iostream>
#include <vector>
#include <list>
#include <iterator>
#include <algorithm>

int main() {
    std::vector<int> v = {1, 2, 3};

    for (auto rit = v.rbegin(); rit != v.rend(); ++rit) std::cout << *rit << ' ';
    std::cout << '\n';                                            // 3 2 1

    std::vector<int> dst;
    std::copy(v.begin(), v.end(), std::back_inserter(dst));       // 尾部插入
    std::cout << dst.size() << '\n';                              // 3

    std::list<int> l;
    std::copy(v.begin(), v.end(), std::front_inserter(l));        // 头部插入
    std::cout << l.front() << '\n';                               // 3

    std::vector<int> w = {1, 3};
    auto pos = std::next(w.begin());
    std::fill_n(std::inserter(w, pos), 1, 2);                     // 中间插入
    for (int x : w) std::cout << x << ' ';
    std::cout << '\n';                                            // 1 2 3
    return 0;
}
```

输出：

```text
3 2 1
3
3
1 2 3
```

```c++
#include <iostream>
#include <sstream>
#include <vector>
#include <iterator>
#include <string>
#include <algorithm>

int main() {
    std::vector<int> v = {1, 2, 3};
    std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, ","));
    std::cout << '\n';                                            // 1,2,3,

    std::istringstream iss("10 20 30");
    std::istream_iterator<int> eos, it(iss);
    int sum = 0;
    while (it != eos) { sum += *it; ++it; }
    std::cout << sum << '\n';                                     // 60

    std::vector<std::string> src = {"a", "b"};
    std::vector<std::string> dst(
        std::make_move_iterator(src.begin()),
        std::make_move_iterator(src.end()));                      // 移动构造
    std::cout << dst.size() << '\n';                              // 2
    return 0;
}
```

输出：

```text
1,2,3,
60
2
```

##### 四、iterator 库常用成员函数与其它特性

**常用自由函数**（均在 `<iterator>`）：

| 函数 | 用途 | 关键点 |
|---|---|---|
| `std::advance(it, n)` | 原地前进 `n` 位 | 双向 / 随机访问可 `n<0`；随机访问 O(1)，否则 O(n) |
| `std::next(it, n=1)` | 返回前进后的新迭代器 | 需输入迭代器 |
| `std::prev(it, n=1)` | 返回后退后的新迭代器 | 需双向迭代器 |
| `std::distance(first, last)` | 求两迭代器距离 | 随机访问 O(1)，否则 O(n) |
| `std::iter_swap(a, b)` | 交换 `*a` 与 `*b` | — |
| `std::begin/end`、`cbegin/cend`、`rbegin/rend`、`crbegin/crend` | 取各类迭代器 | C++11 / C++14 |
| `std::size`、`std::empty`、`std::data` | 元素个数 / 判空 / 首指针 | C++17 |

```c++
#include <iostream>
#include <vector>
#include <iterator>

int main() {
    std::vector<int> v = {10, 20, 30, 40, 50};
    auto it = v.begin();
    std::advance(it, 2);
    std::cout << *it << '\n';                                  // 30
    std::cout << *std::next(it) << '\n';                       // 40
    std::cout << *std::prev(it) << '\n';                       // 20
    std::cout << std::distance(v.begin(), v.end()) << '\n';    // 5
    return 0;
}
```

输出：

```text
30
40
20
5
```

**其它特性**：

- **迭代器失效**：`vector` 扩容时全部失效；`erase` 使被删位置及其后失效；`list` / `forward_list` / `map` / `set` 只使**被删元素**的迭代器失效。边遍历边删用 `it = c.erase(it);`。
- **`iterator` vs `const_iterator`**：`cbegin()` / `cend()` 返回只读迭代器，不能修改元素。
- **`iterator_category` 标签**：`input_iterator_tag`、`output_iterator_tag`、`forward_iterator_tag`、`bidirectional_iterator_tag`、`random_access_iterator_tag`（C++20 增 `contiguous_iterator_tag`）；算法据此选择最优实现。

**Legacy 要求 ↔ C++20 概念对照**：

| Legacy（C++17） | C++20 概念 |
|---|---|
| `LegacyIterator` | `std::input_or_output_iterator` |
| `LegacyInputIterator` | `std::input_iterator` |
| `LegacyOutputIterator` | `std::output_iterator` |
| `LegacyForwardIterator` | `std::forward_iterator` |
| `LegacyBidirectionalIterator` | `std::bidirectional_iterator` |
| `LegacyRandomAccessIterator` | `std::random_access_iterator` |
| （无对应） | `std::contiguous_iterator` |

#### 算法（`<algorithm>` / `<numeric>`）

算法通过**迭代器区间** `[first, last)` 操作数据，不依赖具体容器类型。

**非修改序列算法**：

```c++
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    auto it = std::find(v.begin(), v.end(), 3);
    std::cout << (it != v.end()) << ' ' << *it << '\n';   // 1 3

    std::cout << std::count_if(v.begin(), v.end(),
                 [](int x) { return x % 2 == 0; }) << '\n';   // 2

    std::cout << std::all_of(v.begin(), v.end(),
                 [](int x) { return x > 0; }) << '\n';        // 1

    std::for_each(v.begin(), v.end(), [](int x) { std::cout << x << ' '; });
    std::cout << '\n';                                        // 1 2 3 4 5
    return 0;
}
```

输出：

```text
1 3
2
1
1 2 3 4 5
```

| 算法 | 作用 |
|---|---|
| `find` / `find_if` | 查找元素 / 满足条件的元素 |
| `count` / `count_if` | 计数 |
| `all_of` / `any_of` / `none_of` | 条件判定 |
| `for_each` | 对每个元素执行操作 |

**修改序列算法**：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>

int main() {
    std::vector<int> v = {1, 2, 2, 3, 3, 3};

    std::vector<int> out;
    std::transform(v.begin(), v.end(), std::back_inserter(out),
                   [](int x) { return x * 10; });
    for (int x : out) std::cout << x << ' ';
    std::cout << '\n';    // 10 20 20 30 30 30

    auto last = std::unique(v.begin(), v.end());   // 去重（相邻重复），返回新逻辑结尾
    v.erase(last, v.end());
    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';    // 1 2 3

    std::reverse(v.begin(), v.end());
    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';    // 3 2 1
    return 0;
}
```

输出：

```text
10 20 20 30 30 30
1 2 3
3 2 1
```

| 算法 | 作用 |
|---|---|
| `copy` / `copy_if` | 复制 |
| `transform` | 逐元素变换 |
| `fill` | 填充 |
| `remove` / `remove_if` | 逻辑删除（配合 `erase` 才是真删除） |
| `unique` | 去除**相邻**重复（常先 `sort`） |
| `reverse` | 反转 |

> `remove` 不改变容器大小，只把保留元素前移并返回新的逻辑结尾；真正删除用 erase-remove 惯用法：`c.erase(std::remove(c.begin(), c.end(), x), c.end());`。

**排序算法**：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>

int main() {
    std::vector<int> v = {5, 2, 9, 1, 7};
    std::sort(v.begin(), v.end());                        // 升序
    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';                                    // 1 2 5 7 9

    std::sort(v.begin(), v.end(), std::greater<int>());   // 降序
    for (int x : v) std::cout << x << ' ';
    std::cout << '\n';                                    // 9 7 5 2 1

    std::nth_element(v.begin(), v.begin() + 2, v.end());  // 第 3 小归位
    std::cout << v[2] << '\n';                            // 5
    return 0;
}
```

输出：

```text
1 2 5 7 9
9 7 5 2 1
5
```

| 算法 | 作用 |
|---|---|
| `sort` | 快速排序（不稳定） |
| `stable_sort` | 稳定排序 |
| `partial_sort` | 部分排序（前 k 个有序） |
| `nth_element` | 第 n 小归位（平均 O(n)） |
| `is_sorted` | 判断是否已排序 |

**二分查找**（区间必须已排序）：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>

int main() {
    std::vector<int> v = {1, 3, 5, 7, 9};
    std::cout << std::binary_search(v.begin(), v.end(), 5) << '\n';   // 1
    std::cout << std::binary_search(v.begin(), v.end(), 4) << '\n';   // 0

    auto lo = std::lower_bound(v.begin(), v.end(), 5);   // 第一个 >= 5
    std::cout << std::distance(v.begin(), lo) << '\n';   // 2
    return 0;
}
```

输出：

```text
1
0
2
```

**集合操作**（区间必须已排序）：

```c++
#include <iostream>
#include <vector>
#include <algorithm>
#include <iterator>

int main() {
    std::vector<int> a = {1, 2, 3, 4};
    std::vector<int> b = {3, 4, 5, 6};
    std::vector<int> u, inter;
    std::set_union(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(u));
    std::set_intersection(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(inter));
    for (int x : u) std::cout << x << ' ';
    std::cout << '\n';       // 1 2 3 4 5 6
    for (int x : inter) std::cout << x << ' ';
    std::cout << '\n';       // 3 4
    return 0;
}
```

输出：

```text
1 2 3 4 5 6
3 4
```

| 算法 | 作用 |
|---|---|
| `set_union` / `set_intersection` | 并 / 交 |
| `set_difference` / `set_symmetric_difference` | 差 / 对称差 |

**堆操作**：

```c++
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v = {3, 1, 4, 1, 5, 9, 2, 6};
    std::make_heap(v.begin(), v.end());      // 建大顶堆
    std::cout << v.front() << '\n';          // 9
    v.push_back(10);
    std::push_heap(v.begin(), v.end());
    std::cout << v.front() << '\n';          // 10
    std::pop_heap(v.begin(), v.end());
    v.pop_back();
    std::cout << v.front() << '\n';          // 9
    return 0;
}
```

输出：

```text
9
10
9
```

| 算法 | 作用 |
|---|---|
| `make_heap` | 建堆 |
| `push_heap` / `pop_heap` | 入堆 / 出堆 |
| `sort_heap` | 堆排序 |

**数值算法 `<numeric>`**：

```c++
#include <iostream>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> v = {1, 2, 3, 4};
    std::cout << std::accumulate(v.begin(), v.end(), 0) << '\n';   // 10（求和）
    std::cout << std::accumulate(v.begin(), v.end(), 1,
                 [](int a, int b) { return a * b; }) << '\n';      // 24（求积）

    std::vector<int> seq(5);
    std::iota(seq.begin(), seq.end(), 1);   // 1,2,3,4,5
    for (int x : seq) std::cout << x << ' ';
    std::cout << '\n';                       // 1 2 3 4 5
    return 0;
}
```

输出：

```text
10
24
1 2 3 4 5
```

| 算法 | 作用 |
|---|---|
| `accumulate` | 累加 / 累积 |
| `iota` | 填充递增序列 |
| `partial_sum` / `inner_product` | 前缀和 / 内积 |

#### 函数对象与 Lambda

**仿函数（函数对象）**：重载了 `operator()` 的类，可像函数一样调用，常用作算法的谓词 / 比较器。

**标准函数对象**（`<functional>`）：`less`、`greater`、`plus`、`minus`、`equal_to` 等。

**Lambda（C++11）**：就地定义匿名函数，语法 `[捕获](参数) -> 返回类型 { 函数体 }`。

| 捕获 | 含义 |
|---|---|
| `[]` | 不捕获 |
| `[x]` | 按值捕获 x |
| `[&x]` | 按引用捕获 x |
| `[=]` | 按值捕获所有用到的外部变量 |
| `[&]` | 按引用捕获所有用到的外部变量 |

```c++
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    int threshold = 3;
    std::vector<int> v = {1, 2, 3, 4, 5};
    int cnt = std::count_if(v.begin(), v.end(),
              [threshold](int x) { return x > threshold; });   // 按值捕获
    std::cout << cnt << '\n';        // 2

    int sum = 0;
    std::for_each(v.begin(), v.end(), [&sum](int x) { sum += x; });  // 按引用捕获
    std::cout << sum << '\n';        // 15
    return 0;
}
```

输出：

```text
2
15
```

**`std::function` 与 `std::bind`**：`function` 可统一包装函数、lambda、仿函数；`bind` 绑定部分参数（占位符 `std::placeholders::_1` 等）。

```c++
#include <iostream>
#include <functional>
#include <vector>
#include <algorithm>

int add(int a, int b) { return a + b; }

int main() {
    std::function<int(int, int)> f = add;      // 包装普通函数
    std::cout << f(2, 3) << '\n';              // 5

    std::function<int(int)> add10 = std::bind(add, std::placeholders::_1, 10);
    std::cout << add10(5) << '\n';             // 15

    std::vector<int> v = {3, 1, 2};
    std::sort(v.begin(), v.end(), std::less<int>());   // 标准函数对象
    std::cout << (v[0] == 1) << '\n';          // 1
    return 0;
}
```

输出：

```text
5
15
1
```

#### 工具与容器外组件

##### pair / tuple / tie

- `pair<A, B>`：两个值的组合，成员 `first` / `second`。
- `tuple<...>`：任意个值的组合，用 `std::get<i>(t)` 访问。
- `std::tie` / 结构化绑定：把值解包到变量。

```c++
#include <iostream>
#include <tuple>
#include <string>

int main() {
    std::pair<int, std::string> p = {1, "one"};
    std::cout << p.first << ' ' << p.second << '\n';   // 1 one

    std::tuple<int, double, char> t = {1, 2.5, 'x'};
    std::cout << std::get<0>(t) << ' ' << std::get<1>(t) << ' ' << std::get<2>(t) << '\n';  // 1 2.5 x
    std::cout << std::tuple_size<decltype(t)>::value << '\n';   // 3

    int a; double b; char c;
    std::tie(a, b, c) = t;                          // 解包
    std::cout << a << ' ' << b << ' ' << c << '\n'; // 1 2.5 x

    auto [x, y, z] = t;                             // C++17 结构化绑定
    std::cout << x << ' ' << y << ' ' << z << '\n'; // 1 2.5 x
    return 0;
}
```

输出：

```text
1 one
1 2.5 x
3
1 2.5 x
1 2.5 x
```

##### optional（C++17）

表示"**可能有值，也可能没有**"，避免用魔法值（如 `-1`、`nullptr`）表达失败。

```c++
#include <iostream>
#include <optional>

std::optional<int> parse_positive(int x) {
    if (x > 0) return x;
    return std::nullopt;              // 无值
}

int main() {
    auto a = parse_positive(5);
    auto b = parse_positive(-1);
    std::cout << a.has_value() << ' ' << a.value_or(0) << '\n';   // 1 5
    std::cout << b.has_value() << ' ' << b.value_or(0) << '\n';   // 0 0
    if (a) std::cout << *a << '\n';                               // 5
    return 0;
}
```

输出：

```text
1 5
0 0
5
```

##### variant（C++17）

**类型安全的联合体（union）**：同一时刻只保存一种类型。

```c++
#include <iostream>
#include <variant>
#include <string>

int main() {
    std::variant<int, std::string> v;
    v = 42;
    std::cout << std::get<int>(v) << '\n';         // 42
    v = std::string("hi");
    std::cout << std::get<std::string>(v) << '\n'; // hi
    std::cout << v.index() << '\n';                // 1（当前是第 1 个类型）
    return 0;
}
```

输出：

```text
42
hi
1
```

##### any（C++17）

可存放**任意类型**的值，用 `std::any_cast<T>` 取回。

```c++
#include <iostream>
#include <any>
#include <string>

int main() {
    std::any a = 10;
    std::cout << std::any_cast<int>(a) << '\n';           // 10
    a = std::string("text");
    std::cout << std::any_cast<std::string>(a) << '\n';   // text
    std::cout << a.has_value() << '\n';                   // 1
    return 0;
}
```

输出：

```text
10
text
1
```

##### string_view（C++17）

字符串的**只读视图**（不拥有、不拷贝），适合做只读函数参数，避免构造临时 `std::string`。

```c++
#include <iostream>
#include <string_view>

int main() {
    std::string_view sv = "hello world";
    std::cout << sv.size() << '\n';          // 11
    std::cout << sv.substr(0, 5) << '\n';    // hello
    return 0;
}
```

输出：

```text
11
hello
```

> 注意：`string_view` 不持有数据，**不能超出被引用字符串的生命周期**使用。

#### 智能指针（`<memory>`）

用于自动管理堆内存（RAII），避免手动 `new` / `delete` 导致的内存泄漏。

| 智能指针 | 所有权 | 说明 |
|---|---|---|
| `unique_ptr` | 独占 | 不可拷贝，只能 `std::move`；开销与裸指针相同 |
| `shared_ptr` | 共享 | 引用计数，最后一个销毁时释放 |
| `weak_ptr` | 弱引用 | 不增加引用计数，用于观察 `shared_ptr`、打破循环引用 |

**unique_ptr（独占）**：

```c++
#include <iostream>
#include <memory>

struct Foo {
    Foo()  { std::cout << "Foo ctor\n"; }
    ~Foo() { std::cout << "Foo dtor\n"; }
    void hi() { std::cout << "hi\n"; }
};

int main() {
    auto p = std::make_unique<Foo>();
    p->hi();
    auto q = std::move(p);          // 转移所有权（不能拷贝）
    std::cout << (p == nullptr) << '\n';   // 1
    return 0;
}
```

输出：

```text
Foo ctor
hi
1
Foo dtor
```

**shared_ptr 与 weak_ptr（共享 / 弱引用）**：

```c++
#include <iostream>
#include <memory>

int main() {
    auto sp = std::make_shared<int>(42);
    std::cout << *sp << ' ' << sp.use_count() << '\n';   // 42 1
    {
        auto sp2 = sp;                                   // 共享所有权
        std::cout << sp.use_count() << '\n';             // 2
    }
    std::cout << sp.use_count() << '\n';                 // 1

    std::weak_ptr<int> wp = sp;                          // 弱引用，不影响计数
    std::cout << wp.expired() << ' ' << sp.use_count() << '\n';   // 0 1
    if (auto locked = wp.lock()) std::cout << *locked << '\n';    // 42
    return 0;
}
```

输出：

```text
42 1
2
1
0 1
42
```

**自定义删除器**：

```c++
#include <iostream>
#include <memory>

int main() {
    auto del = [](int* p) {
        std::cout << "custom delete " << *p << '\n';
        delete p;
    };
    std::unique_ptr<int, decltype(del)> p(new int(7), del);
    std::cout << *p << '\n';   // 7
    return 0;
}
```

输出：

```text
7
custom delete 7
```

#### STL 速查表

**按操作选容器**：

| 需求 | 首选 |
|---|---|
| 默认、随机访问、尾部增删 | `vector` |
| 定长数组 | `array` |
| 双端增删 | `deque` |
| 频繁中间插删 / 链表 | `list` / `forward_list` |
| 有序、去重、区间查询 | `set` / `map` |
| 快速查找、无需有序 | `unordered_set` / `unordered_map` |
| 后进先出（LIFO） | `stack` |
| 先进先出（FIFO） | `queue` |
| 始终取最值 | `priority_queue` |

**常用操作复杂度**：

| 操作 | vector | deque | list | set/map | unordered_* |
|---|---|---|---|---|---|
| 随机访问 | O(1) | O(1) | O(n) | — | — |
| 查找 | O(n) | O(n) | O(n) | O(log n) | 平均 O(1) |
| 头部插入 | O(n) | O(1) | O(1) | — | — |
| 尾部插入 | 均摊 O(1) | O(1) | O(1) | — | — |
| 中间插删 | O(n) | O(n) | O(1) | — | — |

**常见陷阱**：

- `vector` 扩容后，旧的迭代器 / 指针 / 引用**全部失效**。
- `remove` / `unique` 不会真正删除元素，需配合 `erase`（erase-remove 惯用法）。
- `operator[]` 不检查越界；只读访问用 `at()`（越界抛异常）或 `find()`。
- `list` 不能用 `std::sort`（需随机访问迭代器），要用成员 `sort()`。
- 二分查找与集合操作要求区间**已排序**。
- `shared_ptr` 循环引用会内存泄漏，用 `weak_ptr` 打破。
