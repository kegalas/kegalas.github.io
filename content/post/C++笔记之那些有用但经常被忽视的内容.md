---
title: C++笔记之那些有用但经常被忽视的内容
date: 2023-09-06T21:52:23+08:00
draft: false
tags:
  - C++
  - 干货
categories: 其他计算机科学
markup: goldmark
---

本笔记会记录一些`C++`中，不是很常用（对于我自己来说）、可能被忽视（对于我自己来说）、我自己不是很熟悉需要记录来复习的、新标准（相较于`C++11`）引入的、可能有用的功能。不适合详细阅读过某一本`C++`大部头教材的人，比较适合对于`C++`的知识只停留在算法竞赛的人。

# std::endl

`std::endl`会立即刷新字符缓冲区，然后输出。但是`'\n'`不会。如果频繁地使用`std::endl`换行可能会导致性能问题，除非你非常确定这条消息在换行后必须要立即输出。

# std::clog

其写入字符到`stderr`中，但不是立即输出。而`std::cerr`会立即输出和刷新`stderr`。

# <=>（三路比较 ）运算符

这个运算符是`C++20`引入的，

- 如果a<b，那么(a<=>b) < 0
- 如果a>b，那么(a<=>b) > 0
- 如果a和b相等或等价，那么(a<=>b) == 0

其实`<=>`返回的是`std::strong_ordering`类型，某些时候给自己的类重载多种比较运算符会简单不少。其中a<b时返回`std::strong_ordering::less`，a == b时返回`std::strong_ordering::equal`，a>b时返回`std::strong_ordering::greater`。

# std::numeric_limits\<T\>

这个命名空间包含在\<limits\>头文件里，其可以给出一个基本数据类型的数据范围，例如

```cpp
std::numeric_limits<T>::lowest();//给出T的最小取值（有符号时为绝对值最大的负数）
std::numeric_limits<T>::min();//对于整数，给出最小取值，对于浮点数，给出最小的正数
std::numeric_limits<T>::max();//给出T的最大取值
std::numeric_limits<T>::epsilon();//浮点数给出最小精度，比如第一个大于1的数和1的差
```

# 大括号初始化变量

我们可以用如下方法初始化变量

```cpp
float a = 1.5f;
float b {1.5f};
```

第一种是传统方法，第二种方法有一点好处是，在隐式的基本类型转换中，如果类型收窄（Type Narrowing）会给出Warning，甚至会直接给出error。

```cpp
int i = 2.5f;//编译器无warning
float f = 2.5f;
int j {f};//编译器产生warning，有时候类型收窄确实可能导致错误。
int k {2.5f};//编译器会直接给出error
```

如果你确定类型收窄是你需要的，那么用显式的强制类型转换。

# \[\[nodiscard\]\]修饰

`C++17`引入。这个修饰是给函数使用的，如果函数有返回值，并且希望返回值不会被忽略，就可以使用这个修饰。

比如

```cpp
void normalize(vec3f & v);//把v变成单位向量
vec3f normalized(vec3f const & v);//返回v的单位向量，但v不变
```

这组可能分辨不清的函数，我们就可以把第二个函数加上这个修饰，编译器会在返回值没有被接收的时候发出warning。

# vector扩容（初步）

这里是一个简易版本的介绍，后续可能会专门出博客来详细解析`STL`的各种容器与算法（TODO）。

我们要首先明白`vector`的内存布局是怎样的。一般来说，除非你在全局变量里面声明`vector`，不然`vector`对象都是在栈中的。但是`vector`的内容（即`buffer`）却是分布在堆里。

首先我们区分一下`size`和`capacity`，前者是`vector`里面拥有的元素的个数，后者是`vector`可以放多少元素。

当`size=capacity`时，此时如果我们进行`push_back`，容量不够，不能放进去。之后`vector`就要进行扩容。此时，实际上的内存不是在`buffer`后面再给你新分配一些，与前面的连起来。而是重新找一块更大的内存，将原来的`buffer`整体复制到堆中的新位置，再把新的插入元素放到最后。栈中的`vector`对象则简单的把指向的`buffer`地址改成新的即可（当然还有修改`size`和`capacity`）。

每次扩容，`capacity`会变为原来大小的$1.1$至$2$倍。

由于扩容的时候要复制，这是很大的开销。所以如果你提前知道数据个数的具体、或者大概的范围，那么最好使用`vector<int> vec(n);`或者`vec.reserve(n);`（区别是前者会有`n`个初值为`0`的元素，后者没有元素，只有`capacity==n`；当然`resize(n)`时，如果新大小大于原来的大小，会把多出来的空间用`0`填补）来提前给够空间。如果你不确定那么没有什么办法，大概只能这样。

# auto与“C-Like”字符串字面量

`auto a = "test";`这个语句，`a`不会是`std::string`类型，而是`char const[]`类型。这也就意味着你也无法使用`auto b = "123"+"456";`。

# std::string的字符串字面量

这个特性是在`C++14`引入的。`auto s = "test"s;`，在原来的基础上，字符串后面加上`s`，即可推断为`std::string`类型。

不过使用之前要先`using namespace std::string_literals;`

# 原始字符串字面量

其用法为`R"(此处填入原始字符串)"`，这里面的原始字符串，不需要转义符，原本是什么，直接输出出来就是什么，而且转行也会被输出出来。这在我们的字符串是Windows目录时可能会比较方便，例如`auto s = R"(C:\Windows\SysWOW64\IME\SHARED)"`，不需要再像以前一样，给每个`\`换成`\\`了。

另外，`auto s = R"(C:\Windows\SysWOW64\IME\SHARED)"`还是被推断为`char const[]`，在`C++14`之后，声明`using namespace std::string_literals;`，之后`auto s = R"(C:\Windows\SysWOW64\IME\SHARED)"s;`可以推断为`std::string`

# std::string_view

`C++17`引入的功能，具体的用法和应该使用的地方都和`std::string const &`差不多，都是对于一个`string`的只读，并且不开额外空间。区别是，`const &`版本是建立了对原`string`的引用。`string`和`vector`很像，都是对象和`buffer`分开，而`std::string_view`就是一个对原`string`的`buffer`的只读的工具。推荐在新版本`C++`中使用这个功能。

# 函数参数什么时候用const &?

可能会有人在第一次学习到用`const &`来修饰形参，觉得这东西简直太好了，可以不用花费额外开销去复制。但其实不总是这样。

如果你传入的数据是`double`，`int`等开销本来就很小的变量，根本就不需要用`const &`，那反而还会增加开销。

- 在变量复制开销本来就很小时，不需要修饰符
- 如果你想要防止自己不小心修改了变量，只加`const`即可
- 如果你想要修改实参，则显然必须加且只加`&`传入引用（例如swap函数）
- 如果你传入的复制开销很大（例如一个图片类），又不需要修改，加`const &`

# 左值、右值（初步）

**左值**

左值指的是可以获取地址的表达式，例如变量、函数形参等。

**右值**

右值正好相反，就是不可获取地址的表达式。例如字面常量（除了字符串字面量），操作符的临时结果（例如`a=b+c;`中的`b+c`），函数的返回值。

具体什么是右值什么是左值可以在[https://zh.cppreference.com/w/cpp/language/value_category](https://zh.cppreference.com/w/cpp/language/value_category)查询。这里面还介绍了纯右值，亡值，将来我会出一篇博客介绍（TODO）。

**&引用**

`&`只能引用左值。也就是说`void fun(int & a);`这个函数，你传入`fun(1);`会编译失败，但是`int b=1;fun(b);`是可以编译成功的。

**const &引用**

`const &`既可以引用左值，又可以引用右值。

左右值的名称是来源于它们在与等号的位置关系，一般情况下左值在等号左边，右值在等号右边。不过不能用一个东西能否被赋值去判断左右值。最好的办法是看能否被引用。

# 小心返回引用的函数

例如

```cpp
int& fun(int x){
    int y = x+1;
    return y;
}
```

显然`y`的生命周期只能持续到函数结束，返回的引用就指向了一个无效的内存，这个要特别小心。但是如果你通过`new`分配了一个对象，再返回引用，则是可以的，这个对象的生命周期持续到你手动使用`delete`或者程序结束。不过并不推荐总是这样做，有时最好直接返回值。

# 引用可能会像迭代器一样失效

比如对`vector`进行的操作会涉及`capacity`的修改，比如在`set`中进行插入和删除操作，这些是会使原来的迭代器无效的。这同时会使引用失效。就比如我们介绍过`vector`在扩容时会复制元素到新的内存，原来的引用指向的内存就会被释放，变成无效内存，此时再进行操作是`UB`。

# 不要用引用来延长函数返回值的生命周期

```cpp
std::vector<int> fun(){...}

auto const & v = fun();//现在fun的返回值的周期延长到和v一样，通常不会造成问题
auto const & v2 = fun()[0];//这是最危险的情况，fun的返回值其实生命周期已结束，v2引用的元素已经变成了无效内存，对v2的操作是UB
```

鉴于上述情况，最好不要用引用接受函数返回值，直接用传值的方式更好。

# struct/class的大括号初值

```cpp
struct St{
    int x;
    double y;
};

St st{1, 2.0};//也可以St st = {1, 2.0};
```

像这样，用大括号给结构体、类赋初值，其顺序和结构体内部声明的顺序要一致。

# 类拷贝（copy）

C++与Java、Python等不同，在下面的例子中

```cpp
struct Point{
    double x,y;
};

Point p1{1.0, 2.0};
Point p2 = p1;
```

用到了`p2 = p1`这一语句。在Java和Python中，`p1`、`p2`现在都指向同一个对象`Point(1.0, 2.0)`，而其本身并不是对象。在C++中，`p2`把`p1`的所有内容拷贝赋值给自己，它们两个是两个不同的对象（即使内容相等）。

在C++中，拷贝默认是深拷贝，而Java和Python中是浅拷贝。如果要在C++里面使用浅拷贝，则使用`Point & p3 = p1`即可（也可以用指针）。

除了是否新建一个对象以为，这两者的生命周期也不同。C++的对象在`p1`销毁时就销毁了，而在Java中`p1`销毁之后，由于还有`p2`指向这个对象，所以对象本身不会销毁。如果所有指向全都销毁，那么垃圾回收机制才会销毁这个对象。

另外，通常“相等”的概念也不同。在Java中，对两个对象变量使用`==`运算符，如果它们指向不同的对象，则不相等，否则相等。在C++中，我们一般会重载`==`运算符，判断两者的内容是否相等。

## 拷贝构造函数

## 赋值构造函数

# argc, argv

```cpp
int main(int argc, char* argv[]){
    //...
}
```

main函数可以带两个参数，按照传统我们把第一个参数叫`argc`，第二个参数叫`argv`。argc是一个整数，代表命令行中参数的个数，argv是每个参数的字符串。

命令行中参数通常由空格分开，例如

```bash
./g++.exe 1.cpp -o 1.exe -Wall
```

其中有五个参数，第一个为可运行文件本身的路径，后面的为运行它的参数。意味着argc等于5，argv存有五个字符串。一般我们会用atoi把字符串里的数字转化为int类型。

# file stream

在NOIP、NOI等竞赛中，一般会用freopen函数。这是一个C的函数，如果要更C++一点，我们会使用file stream。

```cpp
#include <fstream>

int main(){
    std::ofstream os{"1.txt"};

    if(os.good()){//在每次使用时都应该确保good
        os<<"hello world\n";
    }
    
    return 0;
}

```

如上为写数据时的使用例子。可以看到和cout的用法很像。

```cpp
#include <fstream>

int main(){
    std::ifstream is{"2.txt"};

    if(is.good()){//在每次使用时都应该确保good
        double x,y;
        is>>x>>y;
        std::cout<<x<<" "<<y<<"\n";
    }
    
    return 0;
}
```

如上为读数据的例子。可以看到和cin的用法很像。

另外，例如写到末尾还是覆盖，是文本还是二进制，这些都是可以设置的。具体参考[https://zh.cppreference.com/w/cpp/header/fstream](https://zh.cppreference.com/w/cpp/header/fstream)，这里简短的给出几个常用的

```cpp
std::ofstream os{"out.txt", std::ios::app}; //append而不是覆盖

std::ifstream is2{"in.tga", std::ios::binary}; //写二进制
std::ofstream os2{"out.tga", std::ios::binary}; //读二进制
```

# 重载<<和>>运算符

```cpp
struct Vec{
    double x,y;
};

std::istream& operator>>(std::istream& is, Vec& v){
    return is>>v.x>>v.y;
}

std::ostream& operator<<(std::ostream& os, Vec& v){
    return os<<v.x<<" "<<v.y;
}

std::cout<<Vec(2.0,3.0)<<"\n";//可以像对int一样使用cin cout
```

主要是方便打印、输入数据，例如你在编写一个数值计算库，需要用到很多向量、矩阵的输出。

# mutable和const的成员函数

```cpp
class Vec{
public:
    double x,y;
    int mutable sth;
    int sth2;

    void foo() const {
        sth++;//不会报错
    }

    void bar() const{
        sth2++;//会报错
    }
};
```

把类成员函数用const修饰（放在参数列表之后），意味着，这个函数声称不会改变类内的所有成员变量的值。如果你想让几个特例可以修改，那么就把那个特例变量声明为mutable的即可。

# 类成员初始化

在C++ 11以前，我们初始化类成员一般只能在构造函数里进行，如

```cpp
class Vec{
public:
    double x,y;
    Vec():y(0),x(0){}
};
```

其中`:`后面跟着的这个叫做初始化列表，成员后跟着的括号里面的写入初始值，即可完成初始化。

注意，初始化列表里面的赋值顺序并不是初始化列表写出来的顺序，而是按照成员变量的声明顺序。例如上例，是先初始化x，再初始化y。这有时会导致UB，建议绝大部分时候，都要保持两者顺序一致。

当然你也有可能这样写

```cpp
Vec(int x_, int y_){ //而不是写:x(x_),y(y_)
    x = x_;
    y = y_;
}
```

这其实能看出一个C++程序员的水平。对于int、double这种内置类型还好。但如果x、y是class，那么`=`意味着拷贝赋值，意味着可能会先构造一个新的对象实例，再拷贝给x和y。而是用初始化列表，则会直接调用构造函数，总体上少了拷贝这一个步骤。

在C++ 11之后，我们也可以这样给初始值

```cpp
class Vec{
public:
    double x=0.0, y=0.0;
    Vec(){}
};
```

#  explicit关键字


```cpp
class Cl{
public:
    int n;
    explicit Cl(int n_):n(n_){}
};

void foo (Cl a) {}

foo(1);  //隐式调用Cl的构造函数，但因为其构造函数是explicit的，会报错
foo(Cl(1));//正常

```

个人认为这主要是方便强调一下传入的数据的类型，可能在调试环节比较有用。

# 构造函数相互调用

```cpp
Class Vec{
public:
    double x, y;
    Vec():Vec(0){}
    Vec(double a):Vec(a,a){}
    Vec(double x_, double y_):x(x_),y(y_){}
};
```

# 在C++里最好用nullptr而不是null

C++和C语言的`NULL`定义是不同的，在C++中`#define NULL 0`，而在C中`#define NULL ((void*)0)`。不得不这样改的原因是：C++不支持`void*`的隐式转换。可见NULL就是一个数字0，它会有如下问题

```cpp
void foo(int n){}
void foo(void *n){}
```

此时如果你调用

```cpp
foo(NULL);
```

则会有二义性问题。编译可能会无法通过。用nullptr则不会有这个问题。

当然，有时候我们会有以下三种写法

```cpp
if(p){}
if(p!=NULL){}
if(p!=nullptr){}
```

这更多的是一种风格问题，争论这个似乎是无用的。但是之前的二义性还是要小心的。

如果你懒得管，那么就永远使用nullptr。

# const与指针

众所周知，在指针类型的声明里，const放的位置不同会导致语义的不同。主要是指针是否可变，以及指针指向的内容是否可变。 如下表

|声明|所指内容可变？|指针本身可变？|
|-|-|-|
|T \*|是|是|
|T const \*|否|是|
|T \* const |是|否|
|T const \* const|否|否|

简单来说就是const在星号右边则指针自己不可变，在左边则所指内容不可变。也可以理解为，const修饰的是它左边的东西。要么修饰指针（即星号），要么修饰值（即T）

所以说，我更推荐`T const`的写法，而不是`const T`。但是我们也要知道，`const T *`是修饰值不可变。

# 智能指针 TODO

# this指针

有点类似于Python里的self参数，都是只能用在类里的。是一个指向对象自己的指针。

# 析构函数的析构顺序

在析构函数执行完毕后，类成员变量的析构顺序，是按照其声明顺序的反方向进行的。

# 有时候可以尝试集中处理Exception

```cpp
void handle_errors(){
    try{
        throw; //必要的，进行re-throw
    }
    catch(...){}
    catch(...){}
    catch(...){}
}

void foo(...){
    try{
        ...
    }
    catch(...){
        handle_errors();
    }
}

void bar(...){
    try{
        ...
    }
    catch(...){
        handle_errors();
    }
}
```

这可以复用代码，尤其是你的东西可能会抛出一样的exception的时候。

# RAII思想和其对于Exception内存泄漏的保护

RAII是Resource Acquisition Is Initialization的缩写，意为资源获取就是初始化。

其要求，对象在构造函数中获取资源，在析构函数中释放资源。为什么这是好的呢？他可以减少你管理内存的工作量，你不需要手动去到处写delete来释放内存。它会在对象生命周期结束的时候自动释放，也就避免了内存泄漏。

而对于Exception，它也可以很好的保护内存。C++的Exception在throw的时候，保证可以释放创建的局部对象。于是就可以自动释放内存，而不用担心是否在throw前正确处理了内存。

像C语言这样的东西，申请完内存需要free才能释放，如果在函数中提前返回了，无法运行到free这一行，那么内存就泄漏了。如下例

```c
void foo(){
    int *a = malloc(...);
    ...
    if(...){
        ...
        return; // 这里没有释放内存，产生泄漏
    }
    ...
    free(a);
    return;
}
```

所以如果C++调用了某个C库，又没有很好的释放内存，就可能会造成泄漏。如果你想避免这个，可能可以尝试用C++的类包装一下，提供析构函数。

上例在exception结构中类似下例

```cpp
void foo(){
    int *a = new...;
    ...
    if(...){
        ...
        throw ...;// 这里没有释放内存，产生泄漏。但是如果有析构函数则可以避免
    }
    ...
    delete a;
    return;
}
```

另外，RAII也就要求你，不能在析构函数本身中出现提前的throw。

# noexcept修饰

修饰函数时，如果给出noexcept，则意味着你保证：

- 函数的操作不会失败
- 从外部来看，任何Exception是不可见的。或者说所有Exception都在内部处理了。

如果noexcept函数还是抛出了一个Exception，那么程序会终止。

我们可以给noexcept提供一个条件，满足条件是函数才是noexcept的。

```cpp
void foo() noexcept(n<9){...} // 当n<9时noexcept
void bar() noexcept( noexcept(foo) ){...} // 当foo是noexcept时，bar是noexcept
```

# 注意给assert的参数加上括号

因为assert其实是宏，所以

```cpp
assert(min(a,b)==a); //不好
assert((min(a,b)==a)); //好
```

否则，宏替换可能会出问题，而且你无法察觉。

# static_assert

assert是给运行时用的，而static_assert就是给编译时用的。

```
static_assert(bool_exp, "msg"); // C++11可用
static_assert(bool_exp); //C++17可用
```

# 用-DNDBUG忽略所有assert

这是g++的编译选项，只需使用这个参数即可

```
g++ -DNDBUG ...
```

# 编译器warning的编译参数

默认编译并不会打开很多warning，在生产环境中，推荐使用

```
-Wall -Wextra -Wpedantic -Wshadow -Werror -fsanitize=undefined,address
```

来开启更多warning。

# Cmake使用

TODO

# doctest/catch2/gtest使用

TODO

# 不要在调试的时候到处写cout、cerr

首先这会干扰到原本的代码逻辑，你删除的时候可能会删错、忘删。另外，这并不适合写测试样例来检测是否正确，更好的办法是，定义一个函数

```cpp
void log(std::ostream& os, ...) {...}
```

把要输出测试的东西放到ostream里，方便测试样例获取

```cpp
std::ostringstream oss;
log(oss, ...);
ASSERT_STREQ(oss.str(), "123");
```

如果要调试最好使用GDB等工具。

# GDB使用

TODO

# 使用g++/clang检测内存错误使用、未定义行为等

即之前介绍到的，只需要添加`-fsanitize=undefined,address`编译选项即可。在运行的时候，如果出现这种错误，就会提供报错信息。

# 使用valgrind检测内存泄漏、死锁问题等

我们首先编译好程序，然后通过

```
valgrind [options] ./program [program options]
```

来执行检查。可选参数有

- `--tool=memcheck`，用来检查内存泄漏、对无效内存读写等
- `--tool=helgrind`，用来检查死锁
- `--leak-check=full`，用来显示内存泄漏的详细信息
- `-v/--verbose`，用来显示额外信息

# 使用end迭代器的值是未定义行为

```cpp
std::vector<int> vec;
auto it = vec.end();
std::cout<<*it; // UB
```

# std::distance

求得左闭右开区间`[it1, it2)`的大小。

```cpp
std::vector<int> vec{1,2,3};
auto x = std::distance(std::begin(vec), std::end(vec)); // 3
```

如果满足老式随机访问迭代器，那么复杂度是常数。否则复杂度为线性。

# RTTI

todo

# typeid().name()

TODO
