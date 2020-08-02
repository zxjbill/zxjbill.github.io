---
layout: post
title:  "C/C++ 基础-GCC编译和gdb等基础"
date:   2020-08-01 23:59:59
categories: C/C++
tags: C/C++ 语言基础 GCC
mathjax: false
mermaid: false
---

* content
{:toc}

最近在看操作系统，有些地方总需要写写代码尝试一下。但好多东西都不是在windows下尝试的，没办法必须使用Linux环境下的C++。总归GCC/gdb的相关基础是要提上日程的，这里通过网上看的东西，稍微整理一些入门，增加一些入门知识，也备不时之需。

## GCC相关内容
### 什么是gcc和g++
* GCC是指GNU Compiler Collection，是一种支持多种语言的编译器。GNU编译套件包括包括C/C++, Objective-C, Fortran, Java, Ada和Go语言等。
* 对C语言gcc调用C compiler，对C++则g++调用C++ compiler。
* 两者区别：
  * .cpp和.c对于gcc时不一样的，对g++都作为.cpp文件编译
  * **g++会自动链接标准库STL，gcc不会**。
  * gcc编译.c时预定义宏比较少。gcc编译.cpp和g++编译.c和.cpp时都会添加额外的宏。
  * gcc编译c++为了使用STL，可添加参数-lstdc++。**但两者并不只是这个差距**。

下面是其他编译器或解释器名称。C<gcc>, C++<g++>, Java<编译器：gcj, 解释器：gij>, Objective-C<gobjc>

### gcc的特点
* 可移植性，支持多种平台常见的ARM，X86等。
* gcc可实现本地编译和跨平台交叉编译。
* 支持语言多。
* gcc模块化设计，可以添加新的语言和新的CPU架构支持。
* gcc自由软件。GNU。

### gcc编译过程
* 主要分为四个步骤：预处理Pre-Processing，编译Compiling，汇编Assembling，链接Linking，过程如下图。
* 预处理：将头文件、宏等进行展开，去除注释等。不会有语法检查。代码会大量扩充。
* 编译：根据不同语言编译器，进行编译，生成汇编代码。会检查语法。
* 汇编：生成目标代码。
* 链接：将程序需要的目标文件进行链接生成可执行文件。

<figure align="center">
  <img src = "/media/image/GccImage/GCCCompilingProcess.png" style="width:90%" />
</figure>

### gcc常用选项

| 选项名称     | 作用 |
|:-------|:------------|
| -o |产生目标(.i, .s, .o、可执行文件等)|
| -O/-O2 | 优化编译、链接，执行效率会高，编译会慢 |
| -E | 运行C预编译器, 产生.i|
| -S | 产生汇编文件后停止，.s|
| -c | 取消链接步骤，仅编译源码，并在最后生成目标文件|
| -Wall | 让gcc对源码有问题的地方发出警告 |
| -I [dir] | 将dir目录加入到头文件搜索目录路径中 |
| -L [dir] | 将dir目录加入到库的搜索目录路径中 |
| -lib | 链接lib库 |
| -g | 在目标文件中嵌入调试信息，例如gdb之类的 |
| -v | gcc在执行过程中的详细过程，gcc及相关程序版本号 |

使用的代码示例
```bash
gcc -E hello.c -o hello.i	对hello.c文件进行预处理，生成了hello.i 文件
gcc -S hello.i -o hello.s	对预处理文件进行编译，生成了汇编文件
gcc -c hello.s -o hello.o	对汇编文件进行编译，生成目标文件，不链接不包含子程序文件
gcc hello.o -o hello 		对目标文件进行链接，生成可执行文件，不给名字则为x.out
gcc hello.c -o hello 		直接编译链接成可执行目标文件
gcc -c hello.c 或 gcc -c hello.c -o hello.o 编译生成可重定位目标文件
gcc -Wall bad.c -o bad		提示编译器报出警告错误
```

多文件联合编译和分别独立编译
```bash
g++ linkHello.c test.c -o main 多个文件联合编译, 可以不添加.h库文件, 但是相应的.c文件要编译

分别独立编译，使得未修改文件无需再编译
g++ -c linkHello.c -o linkHello.o	编译linkHello.c生成不链接的目标文件.o
g++ -c test.c -o main.o				编译test.c生成不链接的目标文件.o
g++ linkHello.o mian.o -o main		链接的两个目标文件生成可执行文件
```

### 外部库的调用
* 静态库.a：链接时将库的代码链接到可执行文件中，运行时每个程序包含一份，不在需要静态库。
* 动态库.so或.sa：运行时才去链接到共享库代码，多个程序共享使用，减小程序体积。
* 一般的库文件所在位置：/usr/include及其期目录下的include, /usr/local/include, /usr/lib, /usr/local/lib, /lib。

### 生成静态库和共享库(动态库)
* 静态库：先生成目标文件.o，再打包`ar rcs hellolib.a hello.o`，将其打包为`hellolib.a`。ar是GUN归档工具，rcs表示replace and create(会替换已存在的libhello.a文件)。
* 静态库使用方法：编译时把它加入进去就行`gcc hellolib.a test.c -o hello`
* 动态库生成方法：也需要先生成目标文件.o, `gcc -shared -fPIC hello.o -o libhello.so`。-shared表示生成共享库格式， fPIC表示产生位置无关码，表示与加载的内存地址无关。
* 动态库使用：方法1. 将动态库放到/usr/lib目录。方法2. ~/.bash_profile中配置LD_LIBRARY_PATH变量。方法3. 配置/etc/ld.so.conf，配置后调用ldconfig更新ld.so.cache。

### 库搜索路径
* 搜索先后原则：从左到右搜索-l指定的目录->环境变量指定的目录(头文件C_INCLUDE_PATH, 库LIBRARY_PATH)->系统指定的目录。

## GDB相关内容
### GDB及其功能
* Unix及类Unix下的调试工具。


参考链接：
1. [Linux编译：gcc入门](https://www.cnblogs.com/QG-whz/p/5456720.html)
2. [g++以及gcc的区别](https://zhuanlan.zhihu.com/p/100050970)