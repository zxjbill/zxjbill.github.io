---
layout: post
title:  "深入理解计算机系统学习笔记八"
date:   2020-09-03 20:15:00
categories: 操作系统
tags: CS:APP 网络编程
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
终于进入到心心念念的网络编程。本节依赖之前学过的许多概念，进程、信号、字节顺序、内存映射以及动态内存分配，都扮演着重要的角色。需要理解客户端-服务器编程模型，如何编写使用因特网提供的服务的客户端-服务器程序。最后将这些概念结合起来，开发一个虽小但功能齐全的 Web 服务器，能够为真实的 Web 浏览器提供静态和动态的文本和图形内容。

<!--more-->

## 客户端-服务器编程模式
* 每个网络引用都是基于<span style="color:red">客户端-服务器模型</span>的。
* 基本操作：事务 transaction。事务分为以下四步：
  * 客户端发起请求，发器一个事务；
  * 服务器接收请求，并处理请求；
  * 服务器给客户端发送响应，并等待下一个请求；
  * 客户端收到响应，并处理响应。

## 网络相关内容
* 

## 全球 IP 因特网
* 套接字接口处于客户端和 TCP/IP 层之间，采用系统待用的形式。TCP/IP 的相关函数处于系统内核模式。
* IP 地址是一个 32 位无符号整数。TCP / IP 为任意整数数据项定义了统一的网络字节顺序，大端字节顺序。
* `hostname -i` 确定自己的主机点分十进制地址。

```cpp
#include <arpa/inet.h>

struct in_addr{
    uint32_t s_addr;
};
// 返回按照网络字节顺序的值
uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
// 返回按照主机字节顺序的值
uint32_t ntonl(uint32_t netlong);
uint16_t ntons(uint16_t netshort);

// 成功返回1， 若 src 是非法点分十进制地址则为 0，若出错则为 -1
int inet_pton(AF_INET, const *src, void *dst);
// 成功则指向点分十进制字符串的指针，若出错则为NULL
const char* inet_ntop(AF_INET, const void *src, char *dst, socklen_t size);
```