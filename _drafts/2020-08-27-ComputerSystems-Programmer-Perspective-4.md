---
layout: post
title:  "深入理解计算机系统学习笔记四"
date:   2020-08-27 21:53:00
categories: 操作系统
tags: CS:APP 存储器层次结构
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
本篇主要关于存储器系统 memory system。主要了解基本的存储技术，并描述它们如何被组织成层次结构的。高速缓存存储器的使用。C 程序的局部性。“存储器山”。

<!--more-->

## 存储计数
### 随机访问存储器
* 随机访问存储器 Random-Access Memory RAM：分为静态 SRAM 和动态 DRAM 两类。静态 RAM 更贵。
* 静态 SRAM：存在双稳态 bistable，可无限期地保持在两个不同的电压配置或状态。电路会恢复到稳态值。
* 动态 DRAM：由一个电容和一个访问晶体管组成。将电位存储为一个对小电容的充电。对干扰敏感，被扰乱后永不恢复。
  
#### 传统的 DRAM：
* DRAM 芯片中的单元被分为 d 个超单元 supercell，每个超单元都由 w 个 DRAM 单元组成。一般称为“单元”，或一个“字”，这里叫做“超单元”。
* 每个 DRAM 芯片与内存控制器链接，可一次在 CPU 和 DRAM 址间传送 w 位。
* 通过行地址 Row Access Strobe 和列地址 Column Access Strobe 两个请求量来实现 DRAM 的地址访问。会有内部行缓冲区，作为某行到某个单元个的转换。

#### 内存模块
* DRAM 芯片封装在内存模块中，再插到主板上。以64位为块传送数据到内存控制器和从内存控制器传入数据。

#### 增强的 DRAM
* 快页模式 Fast Page Mode DRAM：如果两个访问是在同一行，则可仅去除一次整行数据。
* 扩展数据输出 Extended Data Out DRAM
* 同步 Synchronous DRAM：比上述两种异步的存储器更快的输出超单元的内容。
* 双倍速率