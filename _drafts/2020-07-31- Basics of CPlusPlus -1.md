---
layout: post
title:  "C/C++ 基础总结"
date:   2020-08-01 06:00:00
categories: C/C++
tags: C/C++ 语言基础
mathjax: true
mermaid: true
excerpt_separator: <!--more-->
---

* content
{:toc}

最近可能要参加不少笔试面试，找了些 C/C++语言基础方面的材料，打算过一遍，这里记录总结一些自己有些模糊的概念。

<!--more-->

#### const
* 修饰变量，指针，引用：被修饰的目标不可被修改。类的 const 常量仅能在初始化了列表赋值。
* 修饰成员函数：该成员函数不可修改成员变量的值。const 可用于对重载函数的区分，const 的该类对象<span style="color:red">仅能调用 const 修饰成员函数重载</span>，非 const 的该类对象会优先匹配到没有 const 修饰的重载。
* const int* func(); 返回一个指向常量的指针变量。int* const func();返回一个指向变量的常指针，但是接受这个值的不一定是变量（类似于变量可以等于一个常量）。

#### static
* static 修饰的普通变量，没有初始化，会自动默认值初始化。
* static 修饰的普通函数（非成员函数）一定要包括写成员函数的文件（如果写在。cpp 里，包括相应的。h 并不能调用）才能调用。

#### this
* this 本身被隐式声明为 `ClassName const this;`

#### inline内联
* 相当于把内联函数的内容写到了函数调用处，不用执行进入函数的步骤。
* 编译器一般不内联循环、递归、switch 等复杂的内联函数。
* 隐式内联：在声明处定义的函数，除了虚函数外都会自动成为内联函数。
* 虚函数可以内联，但虚函数表现多态（因为运行时才知道具体调用函数）是不可以内联。内联虚函数只有编译器知道调用的对象是哪个类的时候，只有编译器具有实际对象而不是对象的指针或引用时才会发生。

#### volatile
* 声明变量是可能被一些未知因素给改变，所以不能对该对象进行优化。(例如用寄存器的值替代内存的值等)。

#### assert
* 是宏，不是函数。可在 include 之前使用#define NDEBUG 来禁止使用 assert 断言。
* 可以通过 `#define NDEBUG` 来关闭 assert。

#### sizeof
* 对数组是整个数组的空间大小，对指针是指针本身空间的大小。
* `sizeof` 作用于数组，不会将数组转化为指针进行计算空间大小。
  
#### 结构体、Union空间问题
* 结构体排列的变量大小有自动对齐排列的功能。每个字符加入去占有空间时，看上一个字符占用内存未知，再填充对齐到自己需要的位置。最后大小也要对齐到所包含变量类型最大长度的整数倍。
* Union 也是对齐最大数据类型长度的整数倍。不过是不同数据公用同一块空间。包含一个 13 个 char 的数组和一个 int 型变量的 union 占用空间大小为 16 个字节。
* #pragma pack（n） 设置结构体、union 和类成员变量对齐方式，n 取值可取 1,2,4,8 和 16。实际是取 n 和构造过程中对齐方式的最小值。
 ```cpp
 #pragma pack(push)
 #pragma pack(4)
 ```

 #### 位域
 * 在变量名称后加冒号 `:n`，改变变量存储的大小，指定变量存储的位数（存变量的最小 n 个 bit）。对存储大小，和取地址都有影响。普通变量（不是类的成员变量）不可声明位域。
```cpp
typedef unsigned int Bit; // 这里用无符号的int举例，也可以是其他有符号的类型，必须是整型或枚举类型
class MyClass1
{
public:
    // 数据存储的对齐方式，任按照Bit原始大小对齐，不过这里一共仅需4字节即可存储。
    // 指针无法指向类的位域，所以取地址运算对位域也不可用
	Bit mode : 3; // 仅存储变量的3位
	Bit mode1 : 2; // 仅存储变量的2位
}
```

#### extern
* extern 修饰变量和函数，扩大其作用域，使其可以在其他源文件和头文件使用。
* extern "C"让编译器把代码当成 C 语言代码处理。

#### 结构体名称可以和函数相同不冲突
```cpp
typedef struct Student {
    int age;
} S;            // "S"和"Student"两个标识符名称空间不一样，"Student"在struct标识符这个空间内可找到

int Student()           // 正确，定义后 "Student" 只代表此函数
{

}

//void S() {}          // 错误，符号 "S" 已经被定义为一个 "struct Student" 的别名

int main() {
    Student();
    struct Student me;  // 或者 "S me", 用"Student"则struct是必须的，用以区分函数Student;
    return 0;
}
```

#### 全局匿名union
```cpp
#include <iostream>
static union {
    int i;
    double d;
};              // 类似这里不加变量名，声明了一个全局的匿名union。如果加了则不是匿名的。

int main()
{
    ::i = 20;           // 全局匿名union。
    std::cout << ::i << endl;

    static union
    {
        int i;
        double d;
    };                  // 不加变量名

    i = 30;             // 局部匿名union;
    std::cout << i << endl;
}
```

#### 用C语言实现C++ 类
主要是实现 C++面对对象的特性：封装，继承，多态
* 封装：利用函数指针把属性和方法封装到结构体内部
* 继承：结构体嵌套
* 多态：父类和子类的函数指针不同。

<span style="color:red">TODO 有坑待填！！！！！！！！！！！！！！！！！</span>

#### explicit 关键字
* 修饰构造函数，防止隐式转换和复制初始化
* 修饰转换函数，防止隐式转换，但按语境转换可以（显示转化，if 也显示转化为 bool, 基本变量类型初始化函数 int a()）。

```cpp
struct A
{
	A(int) { }
	operator bool() const { return true; }
};

struct B
{
	explicit B(int) {}
	// explicit operator bool() const { return true; }
	explicit operator int() const { return 1; }
};

struct C
{
	C(int b) {}
};

void doA(A a) {}

void doB(B b) {}

void funcNeedInt(int i) {}

int main()
{
	A a1(1);// OK：直接初始化
	A a2 = 1;// OK：复制初始化
	A a3{ 1 };// OK：直接列表初始化
	A a4 = { 1 };// OK：复制列表初始化
	A a5 = (A)1;// OK：允许 static_cast 的显式转换
	doA(1);// OK：允许从 int 到 A 的隐式转换
	if (a1);// OK：使用转换函数 A::operator bool() 的从 A 到 bool 的隐式转换
	bool a6（a1）;// OK：使用转换函数 A::operator bool() 的从 A 到 bool 的隐式转换
	bool a7 = a1;// OK：使用转换函数 A::operator bool() 的从 A 到 bool 的隐式转换
	bool a8 = static_cast<bool>(a1);  // OK ：static_cast 进行直接初始化

	B b1(1);// OK：直接初始化
	// B b2 = 1;// 错误：被 explicit 修饰构造函数的对象不可以复制初始化
	B b3{ 1 };// OK：直接列表初始化
	// B b4 = { 1 };// 错误：被 explicit 修饰构造函数的对象不可以复制列表初始化
	B b5 = (B)1;// OK：允许 static_cast 的显式转换
	// doB(1);// 错误：被 explicit 修饰构造函数的对象不可以从 int 到 B 的隐式转换
	// if (b1);// 错误，被 explicit修饰函数 B::operator int()的对象不可以隐式转化到bool，涉及到Int到bool的隐式转化。
				// 如果是 B::operator bool() 是可以按语境转化的。
	funcNeedInt(int(b1));	// 正确，显示转换
	// funcNeedInt(b1); // 错误：被 explicit 修饰转换函数 B::operator int() 的对象不可以隐式转换
	// bool bo(b1); // 错误，被 explicit修饰函数 B::operator int()的对象不可以隐式转化到bool，涉及到Int到bool的隐式转化。
	int b6(b1);// OK：被 explicit 修饰转换函数 B::operator int() 的对象可以从 B 到 int 的按语境转换
	// C cc(b1); // 错误，被 explicit修饰函数 B::operator int()的对象不可以隐式转化到int
	// int b7 = b1;// 错误：被 explicit 修饰转换函数 B::operator int() 的对象不可以隐式转换
	int b8 = static_cast<int>(b1);  // OK：static_cast 进行直接初始化

	return 0;
}
```

#### friend友元类和友元函数
* 能访问私有成员
* 破坏封装性
* 不可传递，单向性
* 友元声明的形式和数量不受限制

#### typename关键字

#### 构造函数和析构函数相关
* 构造函数不能直接使用例如类 A 的对象指针 a `A *a;`，调用 `a->A::A()` 在 GNUC 中无法调用。
* 作为替代可以调用 `new (p) A()`，其中 p 是一个分配好地址的指针 p。 定位 `new`。
* 如果析构函数没有意义，并且数组 `a` 的每个元素是数据类型，不具有指针类型，`delete a` 和 `delete[] a` 是没有区别的。

## using 
* `using namespace_name::name` 引入命名空间内的一个成员。
* `using namespace namespace_name` 引入整个命名空间

## enum 枚举类型
* `enum color1` 不限定作用域的枚举类型
* `enum class color2` 限定作用域的枚举类型。

```cpp
enum Color{red1, green1, blue1};

enum class Color1
{
    red,
    green,
    blue,
};

// 不限定作用域
Color c = red1;
// 限定作用域 Color1:: 是必须的
Color1 cc = Color1::red;
```

## decltype
// TODO 没懂

## 左值引用和右值引用
* 右值引用：不可复制，实现转移语义 Move sementics 和精确传递 Perfect Forwarding。
  * 消除对象交互不必要的对象拷贝，节省资源，提高效率。
  * 简洁明确定义泛型函数。

## 宏
* 宏定义类似于函数的实现，括号内的内容进行一一替换。没有类型检查。

## 成员初始化列表
* 好处：
  * 高效
  * 有些场合必须初始化：
    * 常量成员
    * 引用类型
    * 没有默认构造函数的类的对象
* initializer_list 列表初始化。可用于列表初始化一个对象，如下几种典型用法。

```cpp
#include <iostream>
#include <vector>
#include <initializer_list>
 
template <class T>
struct S {
    std::vector<T> v;
    S(std::initializer_list<T> l) : v(l) {
         std::cout << "constructed with a " << l.size() << "-element list\n";
    }
    void append(std::initializer_list<T> l) {
        v.insert(v.end(), l.begin(), l.end());
    }
    std::pair<const T*, std::size_t> c_arr() const {
        return {&v[0], v.size()};  // 在 return 语句中复制列表初始化
                                   // 这不使用 std::initializer_list
    }
};
 
template <typename T>
void templated_fn(T) {}
 
int main()
{
    S<int> s = {1, 2, 3, 4, 5}; // 复制初始化
    s.append({6, 7, 8});      // 函数调用中的列表初始化
 
    std::cout << "The vector size is now " << s.c_arr().second << " ints:\n";
 
    for (auto n : s.v)
        std::cout << n << ' ';
    std::cout << '\n';
 
    std::cout << "Range-for over brace-init-list: \n";
 
    for (int x : {-1, -2, -3}) // auto 的规则令此带范围 for 工作
        std::cout << x << ' ';
    std::cout << '\n';
 
    auto al = {10, 11, 12};   // auto 的特殊规则
 
    std::cout << "The list bound to auto has size() = " << al.size() << '\n';
 
//!    templated_fn({1, 2, 3}); // 编译错误！“ {1, 2, 3} ”不是表达式，
                             // 它无类型，故 T 无法推导
    templated_fn<std::initializer_list<int>>({1, 2, 3}); // OK
    templated_fn<std::vector<int>>({1, 2, 3});           // 也 OK
}
```

## 运行时多态
* 当类具有虚函数时，其则具有虚函数表指针，占用相应字节。
* 运行时多态通过虚函数实现。

## 虚析构函数
* 为了解决基类指针指向派生类对象，用基类指针删除派生类对象。

## 纯虚函数
* `virtual int A() = 0;` 将成员函数 A 声明成纯虚函数。
* 纯虚函数仅是一个接口，实现要在子类完成，并且含有纯虚函数的类称为抽象类，不能实例化成对象。

## 虚函数指针
* 虚函数指针：在含有虚函数的类的对象中，指向虚函数表，在运行时确定。
* 虚函数表：在程序的只读数据段 (.rodata section)，存放虚函数指针，如果派生类实现了基类的某个虚函数，则在虚表中覆盖原来基类的那个虚函数指针，在编译时根据类的声明创建。