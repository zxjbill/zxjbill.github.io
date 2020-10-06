---
layout: post
title:  "开源项目 oatpp 学习"
date:   2020-09-25 15:29:00
categories: Tips
tags: oatpp github
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}

1. 类继承以自己为模板的类 ？？
2. 宏用于产生代码
3. constexpr 和 const 的区别
4. override 的意义，是否可用于非虚函数
5. make_tuple 函数的意义
6. lock_guard<oatpp::concurrency::SpinLock> lock(m_taskLock); Processor.hpp
7. (void) item; 函数的某个形参 item 声明了，但未使用，通过添加这一语句防止编译器警告。
8. unique_lock() 比 lock_guard 更细粒度的锁。unique_lock 提供了解锁 .unlock()，第二参数等功能。另外两种锁都不能复制，但 `std::unique_lock<std::mutex> guards = std::move(unique_lock_guard)` 是被允许的移动的。
9. `std::atomic<T> obj` 可通过 `obj.compare_exchange_wea(T& expected, T val) noexcept` 或 `atomic_compare_exchange_weak(volatile atomic<T>* obj, T* expected, T val) noexcept` 实现 compare and exchange contained value (weak)。相同则交换，并返回 true。不同则修改 expected 为 obj 的 T 值，并返回 false。
10. 自旋锁 spinLock 原地等待，消耗 CPU 资源，应用场景是不应该被长期持有的临界资源。
11. 如何做到输入内容解析和返回的？