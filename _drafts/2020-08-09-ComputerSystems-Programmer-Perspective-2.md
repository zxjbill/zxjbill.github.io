---
layout: post
title:  "深入理解计算机系统学习笔记二"
date:   2020-08-07 23:59:59
categories: 操作系统
tags: CS:APP 汇编
mathjax: true
mermaid: false
---

* content
{:toc}
程序的机器级表示。和汇编相关的一些东西。



## 程序编码
* gcc生成编代码`gcc -Og -S mstore.c`。注意`-S`和`-s`不一样，-s没有main和内部具体函数实现会报错。

```cpp
// 示例代码
long mult2(long , long);

void multstore(long x, long y, long *dest){
    long t = mult2(x, y);
    *dest = t;
}

```
* gcc生成可执行文件`gcc -Og -c mstore.c`。

```armasm
void multstore(long x, long y, long *dest)
x in %rdi, y in %rsi, dest in %rdx

multstore:
	pushq	%rbx				Save %rbx
	movq	%rdx, %rbx			Copy dest to %rbx
	call	mult2				Call mult2(x, y)
	movq	%rax, (%rbx)			Store result at *dest
	popq	%rbx				Restore %rbx
	ret
```

* gdb反汇编器在得到.o文件后,`gdb mstore.o`和在gdb里`x/14xb multstore`这个14和程序内容的长度有关。
* 也可以用OBJDUMP反汇编`objdump -d mstore.o`。

## 数据格式
* “字”代表16个字节，`q`：四字(64位)。 `l`：双字(32位)。`w`：一字(16位)。`b`：一个字节(8位)。 浮点数中`l`：双精度浮点数(64位)。`f`：单精度浮点数(32位)。

## 访问信息
* 16个寄存器分别由不同的标号。
* 操作数可能有三种类型：
  * 立即数：`$-0577`或`$0x1F`等直接获得值。
  * 寄存器：`%rax`等获得寄存器%rax的值。
  * 内存引用：`Imm(rb,ri,s)`格式，例如`260(%rcx, %rdx, 2)`则是获得内存位置`260 + (%rcx) + (%rdx) * 2`的值。`s`一般取1,2,4,8。
* `movl`操作会更新高位四个字节为0，其余操作仅更新目的地址的位。
* 命令传送 MOV指令，第一个是源操作数，第二个是目的操作数。目的地址和源地址不能同时是内存。
* movq只能以32位的立即数作为源通过符号拓展成64位来修改目的位置。(虽然q是转移64位)
* moveabsq以任何64位立即数作为源操作数，并且只能以寄存器作为目的。
* movz--和movs--均只能以寄存器作为目的。
* 没有movzlq这种双字转四字的操作，因为可以用movl替代，具有高位置为0的特点。(有符号的movslq是有的)
* cltq指令描述的是`movslq%eax,%rax``。
* %ebx不是内存地址的寄存器。常用于存放基地址。
* C语言的指针，在汇编和只需要间接引用内存地址即可。
* 作为返回值的局部变量通常放在寄存器中，这样返回更快。
* 压栈和出栈操作`pushq src, dest`和`popq src, dest`，记录的目的地址是栈底部元素所在地址，所以压栈要先变地址，出栈要先出后变地址。

## 算术和逻辑操作
* `leaq src dest`加载有效地址，`movq`的变形，仅有四字操作数类型。作用解释：是将内存地址(这个地址值)写入到目的地址。但常用于简单的计算。
* 一元操作`incq dest`，类似++和`decq dest`，类似--。可以以寄存器或内存为地址。
* 二元操作`addq res dest`和`subq dest`。以立即数，寄存器和内存位置为源地址，寄存器和内存为目的地址。
* 位移操作`salq k dest` 和 `shlq k dest` 左移(注意都是补0，效果相同)， `sarq k dest`算术 Arithmetic 右移，`shrq k dest`逻辑 Logical 右移。位移操作如果以寄存器作为位移位数`k`则会仅调用最低字节。
* 异或操作`xorq %rdx, %rdx`赋值0，与`movq $0,%rdx`效果相同。但在汇编时会`xorq %edx, %edx`使用更少的字节。
* 赋值0仅需改变低位时，可使用 `xorl %edx, %edx` (2字节) 替代四字命令。
* 乘法操作 `imulq` 双操作数实现了无符号乘法和补码乘法(位级行为相同)。
* 乘法操作 `imulq` (有符号的)和`mulq` (无符号的)单操作数时，默认目的操作数为`%rax`，另一个源操作数由指令指定。乘积放入 `%rdx` (高64位)和%`rax` (低64位)中。
* 除法操作 `idivq` (有符号)和 `divq` (无符号)单操作数。默认指定 `%rax` (低64位)和 `%rdx` (高64位)的128位作为被除数。指定的单操作数为出事。结果的商存储在 `%rax` 中，余数存储在 `%rdx` 中。
* 生成8字操作 `cqto` 和 `cqo` (Intel的指令集)。没有操作数。将 `%rax` 符号位补全到 `%rdx` 成为128位的数。

## 控制
* 条件码：单个位，都是标志最近操作的结果情况。CF进位标志(标志最高位产生进位，可以用于检测无符号溢出)，ZF零标志，SF符号标志(标志操作结果是否为负数)，OF溢出标志(检测正溢出和负溢出)。
* leaq不改变条件码(因为它计算的是地址)。
* 逻辑操作进位或溢出设置为0。
* 移位操作，进位设置为最后一个移出的位。溢出标志设置位0。
* `inc`和`dec`会设置溢出和零标志，但是不改变进位标志。
* 条件码设置指令，其不改变其他寄存器。