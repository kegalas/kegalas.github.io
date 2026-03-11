---
title: "C++最佳实践学习笔记"
date: 2026-02-10T15:22:00+08:00
draft: false
tags:
  - C++
categories: 其他计算机科学
mathjax: true
markup: goldmark
---


# 前言

本博客和之前写的[现代C++学习笔记](../现代c-学习笔记)侧重点不同，那一篇偏重C++的特性学习，而本篇则注重于怎么去用、什么时候用、什么时候不用这些特性。

本博客算是对市面上已有的经验的总结，主要来源于：

https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines

https://medium.com/@amalpp42/idioms-in-c-f6b1c19fa605

https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms

https://freegeektime.com/posts/100040501/

https://github.com/romenrg/evergreen-skills-developers

# 泛用

## 直接在代码中表达意图

比如，在函数参数中`change_speed(Speed s)`就要比`change_speed(double s)`要好，后者不知道是什么单位的。返回值也是同理。

然后，最好能将一段代码封装成小函数，取上恰当的名字，例如

```cpp
for(int i = 0; i < size; ++i){
    // ... 进行一些查找操作
}
```

就没有下面的代码好

```cpp
FindXXX(v);
```

## 能用新的c++标准就用，能用标准库就用

新标准提供的东西能够写出更明白、更安全、甚至性能更好的代码。例如c++11提供的智能指针、c++20提供的`std::span`、`std::ranges`。

能用标准库就用，别自己造轮子。自己造轮子花费的大量时间，很可能得不到多少收益，甚至不一定打得过标准库。如果标准库中没有，或者难用（如正则表达式），优先找成熟、文档齐全的第三方库。你必须明白为什么要自己造轮子，才能去造。例如：别人都没这个功能；最小化依赖；极致性能。

## 保证静态类型安全

常见的静态类型安全问题包含：

- 使用`union`
- 类型转换
- 传递数组指针
- 下标超出范围
- 窄化转换

解决方法是：

- 使用`std::variant`，可见[zhuanlan.zhihu.com/p/595288133](https://zhuanlan.zhihu.com/p/595288133)、[zhuanlan.zhihu.com/p/597987861](https://zhuanlan.zhihu.com/p/597987861)。
- 减少使用，或者用模板来替代一部分转换。
- 使用`std::span`
- 使用`std::span`
- 减少使用，使用`gsl::narrow_cast`。使用静态分析工具来检查。

## 能在编译期进行检查的事就别放到运行期

## 不能在编译期检查的也要在运行期检查

# 性能

## 不要不经过测试就说代码有性能问题
