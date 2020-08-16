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
* gcc 生成编代码 `gcc -Og -S mstore.c`。注意 `-S` 和 `-s` 不一样，-s 没有 main 和内部具体函数实现会报错。

```cpp
// 示例代码
long mult2(long , long);

void multstore(long x, long y, long *dest){
    long t = mult2(x, y);
    *dest = t;
}

```
* gcc 生成可执行文件 `gcc -Og -c mstore.c`。

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

* gdb 反汇编器在得到。o 文件后，`gdb mstore.o` 和在 gdb 里 `x/14xb multstore` 这个 14 和程序内容的长度有关。
* 也可以用 OBJDUMP 反汇编 `objdump -d mstore.o`。

## 数据格式
* “字”代表 16 个字节，`q`：四字（64 位）。 `l`：双字（32 位）。`w`：一字（16 位）。`b`：一个字节（8 位）。 浮点数中 `l`：双精度浮点数（64 位）。`f`：单精度浮点数（32 位）。

## 访问信息
* 16 个寄存器分别由不同的标号。
* 操作数可能有三种类型：
  * 立即数：`$-0577` 或 `$0x1F` 等直接获得值。
  * 寄存器：`%rax` 等获得寄存器%rax 的值。
  * 内存引用：`Imm(rb,ri,s)` 格式，例如 `260(%rcx, %rdx, 2)` 则是获得内存位置 `260 + (%rcx) + (%rdx) * 2` 的值。`s` 一般取 1,2,4,8。
* `movl` 操作会更新高位四个字节为 0，其余操作仅更新目的地址的位。
* 命令传送 MOV 指令，第一个是源操作数，第二个是目的操作数。目的地址和源地址不能同时是内存。
* movq 只能以 32 位的立即数作为源通过符号拓展成 64 位来修改目的位置。(虽然 q 是转移 64 位)
* moveabsq 以任何 64 位立即数作为源操作数，并且只能以寄存器作为目的。
* movz--和 movs--均只能以寄存器作为目的。
* 没有 movzlq 这种双字转四字的操作，因为可以用 movl 替代，具有高位置为 0 的特点。(有符号的 movslq 是有的)
* cltq 指令描述的是 `movslq%eax,%rax``。
* %ebx 不是内存地址的寄存器。常用于存放基地址。
* C 语言的指针，在汇编和只需要间接引用内存地址即可。
* 作为返回值的局部变量通常放在寄存器中，这样返回更快。
* 压栈和出栈操作 `pushq src, dest` 和 `popq src, dest`，记录的目的地址是栈底部元素所在地址，所以压栈要先变地址，出栈要先出后变地址。

## 算术和逻辑操作
* `leaq src dest` 加载有效地址，`movq` 的变形，仅有四字操作数类型。作用解释：是将内存地址（这个地址值）写入到目的地址。但常用于简单的计算。
* 一元操作 `incq dest`，类似++和 `decq dest`，类似--。可以以寄存器或内存为地址。
* 二元操作 `addq res dest` 和 `subq dest`。以立即数，寄存器和内存位置为源地址，寄存器和内存为目的地址。
* 位移操作 `salq k dest` 和 `shlq k dest` 左移（注意都是补 0，效果相同）， `sarq k dest` 算术 Arithmetic 右移，`shrq k dest` 逻辑 Logical 右移。位移操作如果以寄存器作为位移位数 `k` 则会仅调用最低字节。
* 异或操作 `xorq %rdx, %rdx` 赋值 0，与 `movq $0,%rdx` 效果相同。但在汇编时会 `xorq %edx, %edx` 使用更少的字节。
* 赋值 0 仅需改变低位时，可使用 `xorl %edx, %edx` （2 字节） 替代四字命令。
* 乘法操作 `imulq` 双操作数实现了无符号乘法和补码乘法（位级行为相同）。
* 乘法操作 `imulq` （有符号的）和 `mulq` （无符号的）单操作数时，默认目的操作数为 `%rax`，另一个源操作数由指令指定。乘积放入 `%rdx` （高 64 位）和%`rax` （低 64 位）中。
* 除法操作 `idivq` （有符号）和 `divq` （无符号）单操作数。默认指定 `%rax` （低 64 位）和 `%rdx` （高 64 位）的 128 位作为被除数。指定的单操作数为出事。结果的商存储在 `%rax` 中，余数存储在 `%rdx` 中。
* 生成 8 字操作 `cqto` 和 `cqo` (Intel 的指令集)。没有操作数。将 `%rax` 符号位补全到 `%rdx` 成为 128 位的数。

## 控制
* 条件码：单个位，都是标志最近操作的结果情况。CF 进位标志（标志最高位产生进位，可以用于检测无符号溢出），ZF 零标志，SF 符号标志（标志操作结果是否为负数），OF 溢出标志（检测正溢出和负溢出）。
* 举例：

```cpp
addq Sre, Dest	t = a + b				// 加法计算
CF (unsigned) t < (unsigned) a			// 无符号溢出
ZF t == 0								// 零
SF t < 0								// 负数
OF (a < 0 == b < 0) && (t < 0 != a < 0)	// 有符号溢出
```

```bash
cmpq Src2, Src1								# 类似计算了 Src1 - Src2, 但不设置值
CF (unsigned) Src1 - Src2 < (unsigned) Src1	# 无符号溢出
ZF Src1 == Src2								# 零
SF Src1 - Src2 < 0							# 负数
OF (Src1 < 0 == Src2 > 0) && (Src1 - Src2 > 0) || (Src1 >= 0 == Src2 < 0) && (Src1 - Src2 =< 0)	# 后面部需不需要取等号部分？待考证
```

```bash
testq Src2, Src1					# 类似计算 Src1 & Src2，但不设置值
ZF		Src1 & Src2 == 0
SF		Src1 & Src2 < 0
```

* leaq 不改变条件码（因为它计算的是地址）。
* 逻辑操作进位或溢出设置为 0。
* 移位操作，进位设置为最后一个移出的位。溢出标志设置位 0。
* `inc` 和 `dec` 会设置溢出和零标志，但是不改变进位标志。
* 条件码设置指令，其不改变其他寄存器。
* 访问条件码 `set-` ：后缀表示考虑的不同条件码组合，将目的位置设置为 0/1。 `n` 代表非，`e` 代表相等，`s` 代表负，`g` 大于，描述为 `(SF ^ OF) & ~ZF`，`ng` 大于等于，描述为

```bash
# 指令		同义名		设置条件				 效果
sete D		setz		D <- ZF					相等/零
setne D		setnz		D <- ~ZF				不等/非零
sets D					D <- SF					负数
setns D					D <- ~SF				非负数
setg D		setnle		D <- ~(OF ^ SF) & ~ZF	大于(有符号>)
setge D		setnl		D <- ~(OF ^ SF)			大于等于(有符号>=)
setl D		setnge		D <- OF ^ SF			小于(有符号<)
setle D		setng		D <- (OF ^ SF) | ZF		小于等于(有符号<=)
seta D		setnbe		D <- ~CF & ~ZF			超过(无符号>)
setae D		setnb		D <- ~CF				超过或相等(无符号>=)
setb D		setnae		D <- CF					低于(无符号<)
setbe D		setna		D <- CF | ZF			低于或相等(无符号<=)
```

* 跳转指令 `jmp` 和 `j-`。`jmp` 是无条件跳转，而 `j-` 是根据后缀跳转，跳转后最与访问条件指令类似。跳转目标分为三种对应举例：标号 `.L1`，寄存器 `*%rax`，内存地址 `*(rax)`。
* 调转指令在汇编之后产生的地址往往是下一条指令的相对位置（即在下一跳指令之后加上 `j-` 在汇编中指定位置的值）

```cpp
long absdiff(long x, long y)
{
	long result;
	if (x < y)
		result = y - x;
	else
		result = x - y;
	return result;
}
```

条件控制实现分支，现代处理器会精密判断分支走向，概率较高时也能较好性能，预测错误会付出严重的时间成本

```bash
# long absdiff_se(long x, long y)
# x in %rdi, y in %rdi
absdiff:
	cmpq	%rsi, %rdi
	jge		.L2
	movq	%rsi, %rax
	subq	%rdi, %rax
	ret
.L2:
	movq	%rdi, %rax
	subq	%rsi, %rax
	ret
```

条件传送实现，顺序执行，当两种计算足够简单时效率会更高。例如计算仅一个加法指令，GCC 会采用条件传送编译。
```bash
# long absdiff_se(long x, long y)
# x in %rdi, y in %rdi
absdiff:
	movq	%rsi, %rax
	subq	%rdi, %rax
	movq	%rdi, %rdx
	subq	%rsi, %rdx
	cmpq	%rsi, %rdi
	cmovge	%rdx, %rax			# 条件赋值，此次赋值条件是大于等于
	ret
```

* 条件传送存可能存在在不合理性。举例如下：
  
```cpp
long cread(long *xp)
{
	return (xp ? *xp : 0);
}
```

不合理的汇编代码
```bash
# long cread(long *xp)
# xp in register %rdi
cread:
	movq	(%rdi), %rax
	testq	%rdi,	%rdi
	movl	$0, %edx
	cmove	%rdx, %rax
	ret
```

* 循环 `do-while` `while` `for`的实现。 

`do-while` 汇编实现的类C表达
```cpp
loop:
	body-statement
	t = test-expr;
	if (t)
		goto loop;
```

`while` 汇编实现的类C表达
```cpp
// 往中间跳
	goto test;
loop:
	body-statement
test:
	t = test-expr;
	if (t)
		goto loop;
```

```cpp
// guarded-do 较高等级优化编译时，例如 -o1， GCC会采用这种策略.
// 采用这种策略编译，编译器往往可以优化初始的测试。
t = test-expr;
if (!t)
	goto done;
loop:
	body-statement
	t = test-expr;
	if (t)
		goto loop;
done:
```

`for` 汇编实现的类C表达
```cpp
init-expr;
goto test;
loop:
	body-statement
	update-expr;
test:
	t = test-expr;
	if (t)
		goto loop;
```

```cpp
	init-expr;
	t = test-expr;
	if (!t)
		goto done;
loop:
	body-statement
	update-expr;
	t = test-expr;
	if (t)
		goto loop;
done: 
```

* 简单的 `for` 循环转化成 `while` 循环对含有 `continue` 的不适用。`continue` 会导致跳过 `update-expr`。
* `switch` 分支在索引开关情况比较多（例如 4 个以上），并且值得范围跨度比较小时，会使用跳转表。开关可能经过某种处理简化分支的可能性。switch可以分支到上百种情况，但使用一次调表访问处理。
* `%rsp` 栈指针和 `%rip` 程序计数器。