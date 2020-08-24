---
layout: post
title:  "深入理解计算机系统学习笔记-附录"
date:   2020-08-23 23:59:59
categories: 操作系统
tags: CS:APP 处理器体系结构
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
简单介绍处理器硬件设计的相关内容。

<!--more-->

## 前言
* 指令集体系架构 Instruction-set Architecture, ISA
* "Y86-64" 是一种指令集。
* HCL Hardware Control Language 硬件控制语言。用以描述处理器设计。
* 处理器设计步骤：
  * 顺序处理器设计。每个时钟周期完成一条指令。
  * 流水线化的处理器。每条指令拆解为五步，分别由不同的硬件部分或阶段来处理。每个时钟周期可以同时处理五条指令的不同阶段。
  * 设计一些工具来研究和测试处理器设计。Y86-64 汇编器、Y86-64 程序模拟器和两个顺序处理器设计和一个流水线化处理器设计的模拟器。
* Verilog 硬件描述语言。

## Y86-64 指令集体系结构
### 程序员可见的状态
* 寄存器： `%rax、%rcx、%rdx、%rbx、%rsp、%rbp、%rsi、%rdi、%r8-%r14` 共十五个寄存器，忽略 `%r15` 作为简化。
* `%rsp` 作为栈顶地址。其余寄存器没有特殊含义和固定值。
* 条件码：`ZF、SF、OF`。
* 程序计数器 `PC`：存放当前正在执行指令的地址。
* 内存：一个很大的字节数组。采用虚拟地址来引用内存位置。硬件和软件联合将虚拟地址翻译成物理地址。
* 状态码 Stat：程序执行的总体状态。指示正常运行或某种异常。

### Y86-64指令
* Y86-64 基本上是 x86-64 指令集的一个子集。仅由8字节数据。
* 指令：源类型-目的类型-数据操作。不允许内存到内存。`i` 立即数、`r` 寄存器、`m` 内存地址。
* 4个整数操作指令 `OPq`：`addq、subq、andq、xorq`。设置三个条件码 `ZF、SF、OF` (零、符号和溢出)。
* 7个跳转指令 `jXX`：`jmp、jle、jl、je、jne、jge、jg`。
* 6个条件传送指令 `comvXX`：`cmovle、cmovl、cmove、cmovne、cmovge、cmovg`
* `call` 指令：返回地址入栈和跳转到目的地址。 `ret` 指令从这样的调用种返回。
* `pushq` 和 `popq` 指令
* `halt` 指令：停止指令的执行。会使得处理器停止运行，并将状态码设置为HLT。
* Y86-64 和 x86-64 的不同
  * Y86-64 相对简单，但不紧凑。
  * Y86-64 寄存器字段的位置固定。x86-64 寄存器位置不固定。
  * Y86-64 仅支持长度位8个字节的数据。x86-64 可将常数编码成 1，2，4 或 8 个字节。

### 指令编码
* 第一个字节：指明指令的类型。
* 多选指令：整数，分支和条件传送指令。高 4 位 code 代码部分，低 4 位 function 功能部分。代码值取 0~0xB，共11种。

```bash
#   指令                编码        指令                编码
    halt                00          nop                 10
    rrmovq rA, rB       20 rA rA    irmovq V, rB        30 F rB
    rmmovq rA, D(rB)    40 rA rB D  mrmovq D(rB), rA    50 rA rB D
    OPq rA, rB          6fn rA rB   jXX Dest            7fn Dest
    cmovXX rA, rB       2fn rA rB   call Dest           80 Dest
    ret                 90          pushq rA            A0 rA F
    popq rA             B0 rA F

# 多选指令功能部分情况
#   整数        分支        条件传送
    addq 60     jmp 70      rrmovq 20
    subq 61     jle 71      cmovle 21
    andq 62     jl  72      cmovl  22
    xorq 63     je  73      cmove  23
                jne 74      cmovne 24
                jge 75      cmovge 25
                jg  76      cmovg  26

# 寄存器对应关系
#   数字    寄存器      数字    寄存器
    0       %rax        1       %rcx
    2       %rdx        3       %rbx
    4       %rsp        5       %rbp
    6       %rsi        7       %rdi
    8       %r8         9       %r9 
    A       %r10        B       %r11
    C       %r12        D       %r13
    E       %r14        F       无寄存器
```

* 仅有 irmovq 可将立即数存储到寄存器中。
* 此处分支指令和调用指令的目的是一个绝对地址。采用相对寻址会使得代码更简洁。
* 整数采用小端法。
* 每条合法的字节编码必须有唯一解释。根据第一个地址的含义，可以确定此条指令的附加字节长度和含义。因此知道指令的起始位置即唯一确定指令意义。、

### Y86-64 异常
* 状态码 Stat：

```bash
#   值  名字    含义
    1   AOK     正常操作
    2   HLT     遇到器执行halt指令  # 会使得处理器停止
    3   ADR     遇到非法地址
    4   INS     遇到非法指令
```

### Y86-64 程序
* `.` 开头的词汇是汇编器伪指令 assembler directives。
* `.pos x` 声明地址。
* `.align` 指明对齐。
* 指令集模拟器 YIS。模拟 Y86-64 机器代码程序的执行。

// TODO 看到 4.1.5 结束