---
layout: post
title:  "深入理解计算机系统学习笔记六"
date:   2020-09-02 08:32:00
categories: 操作系统
tags: CS:APP 链接
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
输入/输出 I/O 时主存和外部设备之间复制数据的过程。本章主要关于 Unix I/O 的学习，了解 Unix I/O 有利于理解其他的系统概念，有时候除了使用系统级 I/O 别无选择。这章介绍的 I/O 概念对后续网络编程和并发性奠定坚实基础。
<!--more-->

## Unix I/O
* Linux 文件就是一个 m 个字节的序列，所有的 I/O 设备都被模型化为文件，所有的输入和输出都被当作对相应文件的读和写来执行，这种方式允许了 Linux 内核引出一个简单、低级的应用接口，称为 Unix I/O。
* 输入输出执行流程及方式：
  * 打开文件：通过内核打开相应文件，并宣告访问 I/O 设备，内核返回一个非负整数，叫作描述符，后续应用程序只记住描述符，内核记录打开文件的信息。
  * Linux shell 创建进程需要三个打开文件：标准输入，标准输出，标准错误（描述符分别是1，2，3）。头文件 `<unistd.h>` 定义了常量 STDIN_FILENO、STDOUT_FILENO、STDERR_FILENO 用于替代显示的描述符。
  * 改变当前文件位置。每个打开文件，内核保持一个文件位置 k，初始为0，表示了文件开头起始的字节偏移量。通过 seek 操作显示设置文件位置。
  * 读写文件。读会有 end-of-file 条件，条件触发显示文件读取达到文件的最末尾位置。
  * 关闭文件。释放打开时创建的数据结构。

## 文件
* 普通文件：任意数据。包含文本文件 text file 和 二进制文件 binary file。文本文件包含 ASCII 或 Unicode 字符的普通文件，二进制是其他所有文件。对内核而言，二级制文件和文本文件没有区别。文本文件每行以一个新行符 "\n" 结束。与 ASCII 的换行符是一样的，0x0a。
* 目录是包含一组链接的文件，每个连接是一个文件名，或另一个目录。"." 连接到本身，".." 连接到目录层次结构中父目录。mkdir 创建一个目录，ls 查看目录，rmdir 删除目录。
* 套间字 用于与另一个进程进行跨网通信的文件。
* 其他文件类型包括：命名通道、符号连接，以及字符和块设备等。
* 作为上下文一部分，每个进程都有当前工作目录，确定其目录层次结构中的当前位置。
* 目录层次可使用绝对路径名和相对路径名两种方式。相对路径名是利用 "." 和 ".." 来实现的。

## 打开和关闭文件
* C 语言打开和关闭文件通过 open 和 close 来实现。其中 open 还可以创建新文件。
* open 打开的一些参数可以用 `flag` 指定。 O_RDONLY 只读、O_WRONLY 只写、O_RDWR 可读可写、O_CREAT 文件不存在则创建一个截断的文件、O_TRUNC 文件存在则截断它、O_APPEND 每次写操作，设置文件位置到文件的结尾处。
* mode 参数指定新文件的访问权限。IR、IW、IX 分别描述可读、可写和可执行。 USR、GRP、OTH 分别描述使用者、所在组和所有人。
* 关闭 close。 

```cpp
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

// 成功则返回文件描述符，失败则返回 -1
int open(char *filename, int flags, mode_t mode);

// 调用方式
fd = Open("foo.txt", O_RDONLY, 0);
fd = Open("foo.txt", O_WRONLY|O_APPEND, 0);

#define DEF_MODE S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH
#define DEF_UMASK S_IWGRP|S_IWOTH

umask(DEF_UMASK);
fd = Open("foo.txt", O_CREAT|O_TRUNC|O_WRONLY, DEF_MODE);

// 关闭文件
int close(int fd);
```

## 读和写文件
* 读写通过 read 和 write 函数执行输入和输出。
* 把 read 和 write 函数的返回值称为不足值。

```cpp
#include <unistd.h>
// 成功返回读字节数，若 EOF 则返回 0，出错则返回 -1
// 最多 n 个字符到内存位置 buf。 
ssize_t read(int fd, void *buf, size_t n);
// 成功则返回写字节数，若出错则返回 -1
// 复制内存 buf 最多 n 个字节到描述符 fd 的当前文件位置。
ssize_t write(int fd, const void *buf, size_t n);
```

* lseek 函数，应用程序能够显示地修改当前文件的位置。
* `ssize_t` 和 `size_t` 有区别，`size_t` 被定义为 `unsigned long`；而 `ssize_t`  被定义为 `long`，是由符号的。
* 某些情况会导致读写值比要求的少：
  * 读到 EOF
  * 从终端读文本行。打开文件与终端相关联，每个 read 函数一次传送一个文本行，返回的不足值等于文本行大小。
  * 读和写网络套接字。由于缓冲约束或较长的网络延迟，可能引起 read 和 write 返回不足值。
  * 进程间通信，对 Linux 管道 pipe 调用 read 和 write 时，也有可能出现不足值。

## 用 RIO 包健壮地读写
* Robust I/O 健壮的包，会自动处理上文所述的不足值。RIO 提供两类不同的函数：
  * 无缓冲的输入输出函数。对二进制数据读写到网络和从网络读写二进制数据尤为有用。
  * 带缓冲的输入函数。允许高效从文件中读取文本行和二进制数据，这些文件内容缓存到应用级缓冲区中。带 RIO 输入函数时线程安全的，在同一个描述符上可以交错地调用。