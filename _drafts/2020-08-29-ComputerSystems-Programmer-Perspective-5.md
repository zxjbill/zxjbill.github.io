---
layout: post
title:  "深入理解计算机系统学习笔记五"
date:   2020-08-29 08:53:00
categories: 操作系统
tags: CS:APP 链接
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
链接 linking 是将各种代码和数据片段收集并组合成一个单一文件的过程。使得分离编译成为可能。包括了静态链接，加载时共享库动态链接以及运行时共享库链接。
<!--more-->

* 链接 linking 是将各种代码和数据片段收集并组合成一个单一文件的过程。可执行于编译，加载或运行时。现代操作系统中，链接一般由链接器自动执行。
* 链接器使得分离编译 separate compilation 称为可能。
* 链接器为什么重要：
  * 构造大型程序
  * 避免一些危险的编程错误
  * 理解语言作用域规则如何实现
  * 理解其他重要的系统概念
  * 利用共享库
* 运行环境 Linux 的 x86-64 系统，使用的标准为 ELF-64 目标文件。

## 编译器驱动程序
* 以下的例子通过 `main.c` 和 `sum.c` 两个源文件组成的项目构成。
* 大多数编译器提供编译驱动程序。
* 编译程序过程分解为：预处理器 cpp，编译器 ccl，汇编器 as，链接器程序 ld等几个步骤。
* 执行时，shell 调用操作系统中一个叫作加载器 loader 的函数，将可执行文件 prog 中的代码和数据复制到内存，然后将控制转移到这个程序的开头。

```bash 
# 调用 GCC 驱动程序编译
gcc -Og -o prog main.c sum.c

# 可分解为以下步骤

# 预处理器 cpp
cpp [other arguments] main.c /tmp/main.i

# 编译器 ccl
ccl /tmp/main.i -Og [other arguments] -o /tmp/main.s

# 汇编器 as
as [other arguments] -o /tmp/main.o /tmp/main.s

# 链接器程序 ld
ld -o prog [system object files and args] /tmp/main.o /tmp/sum.o

# 执行文件
./prog
```

## 静态链接
* 静态链接器 static linker 以一组可重定向目标文件和命令行参数作为输入，生成一个完全链接的、可以加载和运行的可执行目标文件作为输出。
* 链接器必须完成的两个任务：
  * 符号解析：将每一个符号引用正好和一个符号定义关联起来。
  * 重定位：编译器和汇编器生成的代码和数据节都是从 0 开始编址，链接器需要重定位这些节，修改所有符号应用对应的内存位置。链接器使用汇编器产生的重定位条目的详细指令，不加甄别地执行重定位。
* 目标文件只是字节块的集合。这些块包含程序代码，程序数据，其他还包含引导链接器和加载器的数据结构。
* 链接器对目标机器的了解很少，编译目标文件的编译器和汇编器已经完成大部分工作。

## 目标文件
* 目标文件的三种形式：
  * 可重定位目标文件：包含二进制代码和数据，可在编译时与其他重定位目标文件组合，创建一个可执行目标文件。
  * 可执行目标文件：包含二进制代码和数据，其可以直接复制到内存执行。
  * 共享目标文件：一种特殊的重定位目标文件，在加载或运行时被动态地加载到内存并连接。
* 编译器和汇编器生成可重定位目标文件。
* 链接器生成可执行目标文件。
* 目标文件时按照特定的目标文件格式来组织的。各个系统的目标文件格式各不相同。Windows 使用可移植可执行 Portable Executable PE 格式，MacOS-X 使用 Mach-O 格式。现代 x86-64 Linux 和 Unix 系统使用可执行可链接格式 Executable and Linkable Format <span style="color:red">ELF</span>。无论是什么格式，基本的概念是相似的。

## 可重定位目标文件
* 一个典型的 ELF 可重定位目标文件的格式：分为两部分：描述目标文件的节，节两部分。典型的分为以下几部分。
  * 描述目标文件的节：节头部表。
  * 节：分为若干种节。如下：ELF 头、.text、.rodata、.data、.bss、.symtab、.rel.text、.rel.data、.debug、.line和.strlab。
* .text：已编译程序的机器代码。
* .rodata：只读数据。
* .data：已初始化的全局和静态变量。
* .bss：未初始化的全局和静态 C 变量，所有被初始化为0的全局或静态变量。目标文件中，这个节不占据实际空间，仅是占位符。运行时，内存中分配这些变量，初始值为0。bss Block Storage Start，也可以理解成 Better Save Space。
* .symtab：符号表，存放程序中定义和引用的函数和全局变量信息。不包含局部变量的条目。
* .rel.text：一个 .text 节中位置的列表，当链接器把这个目标文件和其他文件组合时，需要修改这些位置。一般调用外部函数或引用全局变量的指令都需要修改。一般可执行目标文件中并不需要重定位信息，因此通常忽略。
* .rel.data：被模块引用或定义的所有全局变量的重定位信息。
* .debug：一个调试符号表，其条目时程序中定义的局部变量和类型定义，程序中定义和定义的全局变量，以及原始的 C 源文件。只有 -g 选项调用编译器驱动程序时，才会得到这张表。
* .line：原始 C 源程序中的行号和 .text 节中机器指令之间的映射。只有 -g 选项编译才会得到。
* .strtab：一个字符串表，其中包括 .symtab 和 .debg 节中的符号表，以及节头部中的节名字。是以 null 节位的字符串序列。

## 符号和符号表
* 可重定位目标模块 m 都有一个符号表 .symtab，包含 m 定义和引用的符号的信息。在链接器的上下文中，有三种不同的符号：
  * 由模块 m 定义并能被其他模块引用的<span style="color:red">本模块的全局符号</span>。对应于非静态的 C 函数和全局变量。
  * 由其他模块定义并被模块 m 引用的<span style="color:red">其他模块的全局符号</span>。这些符号是外部符号，对应其他模块定义的非静态 C 函数和全局变量。
  * 仅被模块 m 定义和引用的<span style="color:red">局部符号</span>。对应带 static 属性的 C 函数和全局变量。这些符号在模块 m 中可见，但不能被其他模块引用。
* 符号表<span style="color:red">不包含</span>任何对应于<span style="color:red">本地非静态程序变量的任何符号</span>。这些符号在栈中管理。
* 局部的静态变量也是在 .data 或 .bss中定义和分配空间的，并在符号表中创建一个有唯一名字的本地链接器符号。
* 利用 static 属性隐藏变量和函数名字：在 C 中，源文件扮演模块的校色，任何带有 static 属性声明的全局变量或者函数都是模块私有的。不带有 static 声明的全局变量和函数都是共有的，可被其他模块访问。 // TODO 尝试检查一下
* 符号表由汇编器构造，使用编译器输出到汇编语言 .s 文件中的符号。.symtab 节中包含 ELF 符号表。

```cpp
typedef struct{
    int name;       /* 字符表字节偏移*/
    char type:4,    /* 数据或函数 */
         binding:4; /* 本地或全局 */
    char reserved;
    short section;  /* 每个符号都被分配到目标文件的某个节，就是一个到节头部表的索引。 */
    long value;     /* 符号的地址, 定义目标的节的起始位置的偏移或绝对运行时地址 */
    long size;      /* 目标的大小, 单位: 字节*/
} Elf64_Symbol;
```

* section 字段种由三个特殊的伪节 pseudosection，它们在节头部表中没有条目。ABS 代表不该被重定位的符号；UNDEF 代表未定义的符号，其他目标块定义的符号；COMMON 表示未被分配位置的未初始化的数据目标。COMMON 符号，value 字段给出对齐要求，而 size 给出最小的大小。只有可重定位目标文件中才有这些伪节，可执行目标文件中是没有的。
* COMMON 和 .bss 的区别：
  * COMMON  未初始化的全局变量
  * .bss    未初始化的静态变量，以及初始化为0的全局或静态变量。


## 符号解析
*  当一个不是在当前模块定义的符号（变量或函数名），会假设该符号在其他模块定义，生成链接符号表条目，并把它交给链接器来处理。
* 全局符号的解析困难：
  * 可能任何输入模块都不能找到这个被引用的符号定义，则输出错误信息并终止。
  * 当有多个目标文件都有相同名字的全局符号时，则标识错误或以某种方法选出一个定义并抛弃其他定义。
* C++ 和 java 链接器符号的重整：重载函数，在编译器会将每个方法和参数组合成一个对于链接器来说是唯一的名字，这个过程叫重整。
* 类的重整：相反的过程叫恢复。类 `Foo` 被编码成 `3Foo`。方法 `Foo::bar(int, long)` 被编码成 `bar__3Fooil`。全局变量和模板名字的策略是相似的。

### 链接器如何解析多重定义的全局符号
* 强/弱符号：编译器向汇编器输出每个全局符号，会有强弱之分。函数和已初始化的全局变量是强符号，未初始化的全局变量是弱符号。
* Linux 链接器使用以下规则来处理多重定义的符号名：
  * 不允许由多个同名的强符号。
  * 如果有一个强符号和多个弱符号，则选强符号。
  * 如果有多个弱符号同名，则任意选择一个弱符号。
* 上述规则说明一个变量定义后再赋值和直接初始化是由差别的，特别是全局变量。
* `GCC -fno -common` 可以告诉链接器遇到多重定义的全局符号时，除法一个错误。或者使用 `-Werror` 选项，将所有警告编程错误。 // TODO -fno -common 有没有短横线
* 上述规则也说明了为什么一个全局变量要分为初始化和未初始化两种情况。未初始化则要分配给 COMMON，留给链接器处理。
* 静态符号的构造必须是唯一的，所以编译器可以直接分配成 .data 或 .bss。

### 与静态库链接
* 静态库机制：将所有相关的目标模块打包成一个单独的文件。当链接器构造一个输出可执行文件时，它只复制静态库被应用程序引用的目标模块。
* 添加静态库的两种方法：
  * 通过编译器辨认出标准函数的调用，直接生成相应的代码。例如 Pascal，因为其仅提供了一小部分标准函数。
  * 将所有标准函数都放在一个单独的可重定位目标模块/独立可重定位文件/单独的静态库文件中，应用程序员把该模块链接到可执行文件中。这种方案是 C 的实现方案。可将标准函数和编译器的实现分离开。
* 静态库是一种成为存档 archive 的特殊文件格式，存放在磁盘中。存档文件时一组连接起来的可重定位目标文件的集合，有一个头部用于描述每个成员目标文件的大小和位置，存档文件名由后缀 `.a` 标识。
* `ar` 可用于创建静态库 `ar rcs libvector.a addvec.o multvec.o`。会生成 `libvector.a` 的库 // TODO 研究下 ar 工具。
* 链接静态库的方式：

```bash
gcc -c main2.c

# 以下两种方式等效
gcc -static -o -prog2c main2.c ./libvector.a
gcc -static -o -prog2c main2.c -L. -lvector
```

* 其中 `-static` 告诉编译器驱动程序，链接器应该构建一个完全链接的可执行目标文件，可加载到内存并运行，加载时无须进一步链接。`-lvector` 参数时 `libvector.a` 的缩写，`-L.` 参数告诉链接器在当前目录下查找 `libvector.a`。
* `-c` 取消链接，仅编译源码，生成可重定位目标文件。

### 链接器如何使用静态库来解析引用
* 静态库的链接解析：
  * 链接器维护可重定位目标文件的集合 E、未解析的符号集合 U 和前面输入文件中已定义的符号集合 D。
  * 输入文件是目标文件，则将文件添加到 E 修改 U 和 D 来反应 f 的符号定义和应用。
  * 输入文件是存档文件（库文件），则检索 U 中为解析符号与存档文件成员定义的符号，如果匹配了存档文件成员 m，则将文件成员 m 添加到E，并修改 U 和 D 反应 m 的应用。并且不会添加不在 U 中的符号。
  * 当链接完成时，如果 U 非空， 则报错。
* 因此当存档文件先链接，而引用该存档文件的目标文件后链接则会导致引用的符号不会被添加。所以一般将库放到命令行的结尾。如果库并非独立，还应该对它们进行合适的排序。
* 有时为了满足依赖需求的特殊处理方式：在命令行上重复库，将由依赖关系的库合并成一个单独的存档文件。

## 重定位
* 在完成符号解析后，进行重定位工作，主要分为两步：
  * 重定位节和符号定义。相同类型的节合并为同一类型的新的聚合节。将运行时内存地址赋值给新的聚合节，赋值给输入模块定义的每个节，以及赋给输入模块定义的每个符号。完成这一步，程序中每条指令和全局变量都有唯一的运行时内存地址。
  * 重定位节中的符号引用。修改代码节和数据节中对每个符号的引用，使其指向正确的地址。依赖于可重定位目标模块中成为重定位条目的数据结构。

### 重定位条目
* 汇编器在遇到最终位置未知的目标引用，就会生成一个重定位条目，告诉链接器合并时需要修改这个引用。代码重定位条目放在 .rel.text 中。已初始化数据重定位放在 .rel.data 中。

```cpp
typedef struct{
    long offset;    /* 引用位置的相对便宜量 */
    long type:64,   /* 重定位类型 */
         symbol:32; /* 符号表的索引 */
    long addend;    /* 有符号常数，一些类型重定位需要使用它来对被修改引用做偏移调整 */
} Elf64_Rela;
```

* 重定位类型：ELF 定义了 32 种不同的重定位类型。以下是两种最基本的重定位类型：
  * R_X86_64_PC32 重定位一个使用 32 位 PC 相对地址的引用。相对于当前程序计数器 PC 运行值得偏移量。例如 call 指令得目标。addend 可以用于上一个指令到该指令的位置。
  * R_X86_64_32 重定位使用 32 位绝对地址得引用。通过绝对寻址，CPU 直接在指令中编码的 32 位值作为有效地址，不需要进一步修改。
* 上述两种重定位类型支持 x86-64 小型代码模型，该模型可执行目标文件的代码和数据总体大小小于 2 GB，GCC 默认使用的小型代码模型。大于 2 GB 的程序可以使用 -mcmodel=medium 和 -mcmodel=large 标识来编译。

### 重定位符号引用
* 重定位算法的伪代码
* 利用 PC 位置重定位的计算中，addend 指代重定位的指令位置到下一个指令的距离。
* 绝对引用重定位中，addend 为 0。 // TODO 了解下机器码如何工作
  
```cpp
foreach section s{
    foreach relocation entry r{
        refptr = s + r.offset;  // 重定位位置的存储位置

        // 计算相对于 PC 的重定位位置
        if (r.type == R_X86_64_PC32){
            refaddr = ADDR(s) + r.offset;   // 计算运行时，被重定位地点的地址
            *refptr = (unsigned) (ADDR(r.symbol) + r.addend - refaddr); // 计算差值
        }

        // 计算绝对位置
        if (r.type == R_X86_64_32){
            *refptr = (unsigned) (ADDR(r.symbol) + r.addend);   // 计算绝对位置
        }
    }
}
```
## 可执行目标文件
* 典型的 ELF 可执行目标文件的各类信息：   // TODO 加图
* ELF 头描述文件的总体格式，包括程序的入口点。
* .text、.rodata、.data 与可重定位目标文件中的节类似，但已经重定位处理了。
* .init 定义了一个叫做 _init 的函数，初始化代码时会调用。
* 没有 .rel 节，因为已经完全重定位了。
* 通过 OBJDUMP 得到的程序头部表中包含以下信息： off 目标文件中的偏移量； vaddr/paddr 内存地址；align 对齐要求；filesz 目标文件中的段大小；memsz 内存中的段大小；flags 运行时访问权限。
* 可执行文件的连续的片被映射到连续的内存段中，程序头部表描述了这种映射关系。可能初始化两个内存段：代码段（只读-可执行 r-x）和数据段（可读/写 rw-）。代码段包括 ELF 头、程序头部表、.init、.text 和 .rodata。数据段包括初始化的 .data 和未初始化或初始化为0的 .bss，目标文件已使用的内存中的段大小是 .data，未使用的是 .bss。
* 对齐效果所以有如下结论 $vaddr mod align = off mod align$

## 加载可执行目标文件
* `./prog` 调用加载器 loader 运行可执行目标文件。Linux 程序都可以通过调用 execve 行数来调用加载器。
* 加载：将可执行文件中的代码和数据从磁盘复制到内存中，然后通过跳转到程序的第一条指令或入口点来运行该程序，这整个过程叫做加载。
* 运行时的内存映像 // TODO 加图

## 动态链接共享库
* 静态库缺陷：
  1. 当更新静态库时，不得不显式地将其与程序重新链接。
  2. 有许多库使用标准 I/O 函数，这样的函数会被每个程序拷贝一份到程序对应的内存中去，这是对内存的极大浪费。
* 共享库：在运行或加载时，可以加载到任意内存地址，并和一个在内存中的程序链接起来。这个过程叫动态链接，由动态链接器来完成。
* 共享库在 Linux 系统中通常用 `.so` 后缀表示。微软的操作系统一般用 `DLL` 表示。
* 两种不同的方式来“共享”：
  * 对一个库只有一个 `.so` 文件。所有引用该库的可执行目标文件共享这个 so 文件的代码和数据。
  * 在内存中一个共享库的 .text 节的一个副本可以被不同的正在运行的进程共享。
* 共享库生产过程 `gcc -shared -fpic -o libvector.so addvec.c multvec.c`。-fpic 指示编译器生成与位置无关的代码，-shared 指示链接器创建共享的目标文件。
* 共享库链接 `gcc -o prog21 main2.c ./libvector.so` 
* 在链接过程中没有任何库里的代码和数据被复制到了可执行文件中，而时复制了一些重定位和符号表信息，使得运行时可解析对库中代码和数据的引用。

## 从应用程序中加载和链接共享库
* 应用程序在它运行时要求动态链接器加载和链接某个共享库，无需再编译时将这些库链接到应用。可利用于分发软件，构建高性能 Web 服务器。
* Linux 系统为动态链接器提供了一个简单的接口，允许应用程序运行时加载和链接共享库。

```cpp
#include <dlfcn.h>
// 成功则返回指向句柄的指针，若出错则返回 NULL。
// flag 参数为 RTLD_NOW 现在立即解析和 RTLD——LAZY 推迟符号解析直到执行来自库中的代码
void *dlopen(const cahr *filename, int flag);

// 成功返回指向符号的指针，出错返回 NULL
void *dlsym(void *handle, char *symbol);

// 卸载共享库，成功返回 0，失败返回 -1
int dlclose(void * handle);

// 返回一个字符串，描述最近发生的调用错误。没有错误返回 NULL
const char *dlerror(void);
```

* 共享库和 Java 本地接口：允许 Java 程序调用“本地的” C 和 C++ 函数。Java 解释器利用 dlopen 接口（或类似接口）动态链接函数。

## 位置无关代码
* 位置无关代码 Position-Independent Code PIC：可以加载而无需重定位的代码（数据任然会有自己的读/写数据块）。GCC 使用 -fpic 选项则是生成这样的 PIC 代码。共享库的编译总是使用该选项。

### PIC 数据引用
* 内存任意位置加载目标模块，数据段和代码段距离总是保持不变的。
* 全局偏移量表 Global Offset Table GOT：每个被这个目标模块引用的全局数据目标（过程或全局变量）都有一个 8 字节条目，编译器为 GOT 每个条目生成一个重定位记录。加载时，动态链接器会重定位 GOT 中每个条目，使得它包含目标的正确的绝对地址。每个引用全局目标的目标模块都有自己的 GOT。

### PIC 函数调用
* 延迟绑定：产生重定位记录，在程序第一次调用该过程时，通过动态链接器再绑定该过程的地址。
* 延迟绑定通过 GOT 和过程连接表 Procedure Linkage Table PLT 来实现的。
* 一个目标模块调用定义在共享库的任何函数，那么它就有自己的 GOT 和 PLT。GOT 是数据段的一部分，而 PLT 是代码段的一部分。
* 通过类似程序实现第一次调用

```bash
# 全局偏移量表 GOT
GOT[0]: addr of .dynamic
GOT[1]: addr of reloc entries
GOT[2]: addr of dynamic linker
GOT[3]: 0x4005b6 # sys startup
GOT[4]: 0x4005c6 # 在第一次调用后，链接器将此处数据改为对应函数的地址
GOT[5]: 0x4005d6

# 过程链接表 PLT
4005a0:pushq *GOT[1]
4005a6:jmpq *GOT[2]
-----
4005c0:jmpq *GOT[4]
4005c6:pushq $0x1
4005cb:jmpq 4005a0
```

## 库打桩机制
* 打桩基本思想：给一个需要打桩的目标函数，创建一个<span style="color:red">包装函数</span>，原型与目标函数一样，使用某种特殊的打桩机制，欺骗系统调用包装函数而不是目标函数，包装函数会执行它自己的逻辑，再调用目标函数，再将目标函数的返回值，返回给调用者。
* 打桩可发生在编译时、链接时或程序加载和执行时。

### 编译时打桩
* `-I.` 参数告诉 C 预处理器在搜索系统目录前，应该先搜索当前目录查找。而包装函数的文件使用目标函数来编译的。

```bash
gcc -DCOMPILETIME -c mymalloc.c
gcc -I. -o intc int.c mymolloc.o # 注意是先生成好了 mymolloc.o 的

./intc
malloc(32)=0x9ee10
free(0x9ee10)
```

c 文件如下

```cpp
// int.c
#include<stdio.h>
#include<malloc.h>

int main()
{
  int *p = malloc(32);
  free(p);
  return(0);
}
```

```cpp
// 新建本地 malloc.h
#define malloc(size) mymalloc(size)
#define free(size) myfree(ptr)

void *mymalloc(size_t size);
void myfree(void *ptr);
```

```cpp
// 新建包装函数 malloc.c
#ifdef COMPILETIME
#include <stdio.h>
#include <malloc.h>

void *mymalloc(size_t size){
  void *ptr = malloc(size);
  printf("malloc(%d)=%p\n",
    int size, ptr);
  return ptr;
}

void myfree(void *ptr){
  free(ptr);
  print("free(%p)\n", ptr);
}
#endif

```

### 链接时打桩
* Linux 静态链接器支持 -Wl,option 把 option 传递给链接器，让链接器把对符号 option 的引用解析成 __wrap_option，把对符号 __real_option 的引用解析为 option。

```bash
gcc -DLINKTIME -c mymalloc.c
gcc -c int.c
gcc -Wl,--wrap,malloc -Wl,--wrap,free -o intl int.o mymalloc.o

./intc
malloc(32)=0x9ee10
free(0x9ee10)
```

```cpp
#ifdef LINKTIME
#include <stdio.h>

void *__real_malloc(size_t size);
void __real_free(void *ptr);

void *__wrap_malloc(size_t size){
  void *ptr = __real_malloc(size);
  printf("malloc(%d)=%p\n",
    int size, ptr);
  return ptr;
}

void __wrap_free(void *ptr){
  __real_malloc(ptr);
  print("free(%p)\n", ptr);
}

#endif
```

### 运行时打桩
* 利用动态链接器的 LD_PRELOAD 环境变量,
* 利用 dlsym 调用返回指向目标函数的指针。

```bash
gcc -DRUNTIME -shared -fpic -o mymalloc.so mymalloc.c -ldl
gcc -o intr int.c

# 在 bash shell 中运行程序
LD_PRELOAD="./mymalloc.so" ./intr
```

```cpp
#ifdef RUNTIME
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>

void *malloc(size_t size){
  void *(*mallocp)(size_t size);
  char *error;

  // 获得库内 malloc 函数的地址
  mallocp = dlsym(RTLD_NEXT, "malloc");
  if (error = dlerror() != NULL){
    fputs(error, stderr);
    exit(1);
  }

  char *ptr = mallocp(size);
  printf("malloc(%d) = %p\n", (int) size, ptr);
  return ptr;
}

void free(void *ptr){
  void (*freep)(void* ptr);
  char *error;

  // 获得库内 free 函数地址
  freep = dlsym(RTLD_NEXT, "free");
  if (error = dlerror() != NULL){
    fputs(error, stderr);
    exit(1);
  }

  freep(ptr);
  printf("free(%p)\n", ptr);
}

#endif
```

## 处理目标文件的工具
* GNU binutils 包很有帮助
* AR: 创建静态库，插人、删除、列出和提取成员。
* STRINGS: 列出一个目标文件中所有可打印的字符串。
* STRIP: 从目标文件中删除符号表信息。
* NM: 列出一个目标文件的符号表中定义的符号。
* SIZE: 列出目标文件中节的名字和大小。
* READELF: 显示一个目标文件的完整结构，包括ELF 头中编码的所有信息。包含 SIZE 和NM 的功能。
* OBJDUMP: 所有二进制工具之母。能够显示一个目标文件中所有的信息。它最大的作用是反汇编.text 节中的二进制指令。

Linux 系统为操作共享库还提供了LDD 程序：
* LDD: 列出一个可执行文件在运行时所需要的共享库。