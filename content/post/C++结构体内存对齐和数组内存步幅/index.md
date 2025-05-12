---
title: "C++结构体内存对齐和数组内存步幅"
date: 2025-05-11T15:42:58+08:00
draft: false
tags:
  - C++
categories: 其他计算机科学
mathjax: true
markup: goldmark
---

# 内存对齐

众所周知，计组这门课告诉我们，对齐与没对齐的数据在访问时有效率差别，而C/C++作为一个接近底层和关注速度的语言，可以操控内存对齐。

一个类型的对齐值为`x`，则意味着这个类型的内存地址为`x`的倍数。

C++的内存对齐规则如下：

- 编译器可以选择一个`#pragma pack(n)`来规定一个对齐值`n`。（如果未设置则为无穷大，这点是我在`gcc`上推测出来的）
- 首个元素的内存偏移量为`0`
- 内建类型的对齐值为`min(其sizeof大小, n)`。
- 结构体的对齐值为`min(成员中最大的对齐值, n)`，并且`sizeof(该结构体)`是其对齐值的整倍数

其中内建类型，例如假设`n=4`，则`double`对齐到`4`，`int32_t`对齐到`4`，`int8_t`对齐到`1`。假设`n=8`，则`double`对齐到`8`，`int32_t`对齐到`4`，`int8_t`对齐到`1`。（数组内元素的对齐值等价于将数组拆成多个元素，即`float x[3];`等价于`float a; float b; float c;`）

例子如下

```cpp
// gcc默认值为无限大，不设置或者pack(0)都是无限大。而pack()不给参数似乎是由命令行决定，在我这里仍然是无限大。
#pragma pack(4)
struct A{
    char a;    //offset = 0
    double b;  //offset = 4, 虽然double是8字节，但是pack(4)
    char c;    //offset = 12
};//总大小16，因为a和b一共占了12个字节，c本来再占1个字节就可以，但是13不是4的整倍数，向上到16
```

```cpp
#pragma pack(8)
struct A{
    char a;    //offset = 0
    double b;  //offset = 8
    char c;    //offset = 16
};//总大小24，同前，17不是8的整倍数，向上到24
```

这其中`a`和`b`中的7字节空间被填充为空白，无法访问，因为暂时没有合适的符合对齐要求的元素可以插进去。然而，如果我们添加一个`int32_t`，其只要满足四字节对齐，就可以插进去

```cpp
#pragma pack(8)
struct A{
    char a;    //offset = 0
    int x;     //offset = 4
    double b;  //offset = 8
    char c;    //offset = 16
};//总大小24
```

除了插入一个`int32_t`，你还可以插入三个`int16_t`，七个`int8_t`。

然后我们来讨论一下为什么是结构体的对齐值为`min(成员中最大的对齐值, n)`，而不是`min(sizeof(成员)的最大值, n)`，如下例的嵌套结构

```cpp
#pragma pack(16)
struct A{
	char a;
	double b;
	char c;
};//总大小24

struct B{
	char x;
	A a;
	char c;
};//总大小40
```

如果结构体`B`的对齐值取决于成员对齐，那么`A`的对齐值可知为`8`，则`B`的对齐值也为8。此时有

```cpp
struct B{
	char x;   //offset=0
	A a;      //offset=8
	char c;   //offset=32
}; //c此时占据了第33个字节，需要向上到40符合8的倍数
```

但如果`B`的对齐值取决于成员大小，而显然`A`的大小为24，`B`的对齐值再怎么说也是`16`，但40显然不是16的倍数，所以并不是取决于成员大小。

在c++中，你还可以通过`alignas()`来设置某个成员的内存对齐，例如

```cpp
struct A{
	char a; // 0
	alignas(64) double b; // 64
	char c; // 72
}; //总大小128
```

其中`alignas(64)`并不意味着完整占据`64`字节，而只是把地址对齐到`64`字节，所以只用了八个字节，后面的字节还可以被别的成员使用。

为什么说不打`#pragma pack`这一行就意味着`n`无限大呢，如果真的有gcc中`n`默认取`4`，那么应该是

```cpp
#pragma pack(4)
struct A{
	char a; // 0
	alignas(64) double b; // 4
	char c; // 12
}; // 总大小为16
```

和上面什么都不设置的完全不同，故不设置必须是无限大才能解释这个现象。

当然，`alignas()`也只能设置为2的幂次，并且如果设置的值小于该类型原有的对齐，那么还是取原有的对齐。

另外，你还可以给结构体设置对齐

```cpp
struct alignas(32) A{
	char a;
	double b;
	char c;
};//总大小32

struct B{
	char x;
	A a;
	char c;
};//总大小96
```

但这里的`alignas`和前面给成员设置的时候略有不同，前面的`double`小于64，后面的空白位还是可以给接下来的成员用的。而这里的A虽然也小于32，但是结构体对齐在内部消化了这些空白位。与下面的例子对比起来更好理解

```cpp
struct A{
	char a;
	double b;
	char c;
};//总大小24

struct B{
	char x;
	alignas(32) A a;
	char c;
};//总大小64，暗示着a之后的空间还可以被c使用
```

# 数组内存步幅

这是个小众变态需求，我遇到的情况是glsl的std140要求数组的步幅等于数组元素的对齐大小。数组的对齐要求是元素大小的对齐并向上取整到16的倍数，而相邻的两个数组元素之间的地址差距，或者说步幅要等于这个对齐。

举个例子，一般来说`float[3]`的大小是12字节，而std140要求float首先对齐到16，然后每两个`float`之间的间距也是16，也就是总共大小为48字节。一个非常不成功的写法是

```cpp
struct A{
    alignas(16) float a[3];
    char b[4];
}; //大小为16字节

struct A{
    char c;
    alignas(16) float a[3];
    char b[4];
}; //大小为32字节
```

显然的，`alignas`仅仅只将数组的开头对齐到了16字节，并没有将每一个元素都对齐到16。有一个比较不美丽的解决方式。

```cpp
struct MyFloat{
	alignas(16) float v;
};

struct A{
    char a;
	MyFloat b[3];
	char c;
}; //总大小80
```

不美丽就不美丽在每次访问值都必须`a.b[i].v`，必须用成员运算符取值，我们可以用模板和重载运算符搞一个更好用、更泛用的

```cpp
template<typename T, size_t Size>
class Std140Array{
private:
    struct alignas(16) Element{
        T value;
    };

    Element elements[Size];

public:
    T& operator[](size_t idx){ return elements[idx].value; }
    const T& operator[](size_t idx) const { return elements[idx].value; }
};

struct A{
	char a;
	Std140Array<float, 3> b;
	char c;
}; //总大小80

A a;
a.b[1] = 1; // 好用
```

不知道还有没有什么其他更高雅的做法，总而言之目前这个我用的比较舒服。
