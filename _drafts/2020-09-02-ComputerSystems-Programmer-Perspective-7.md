---
layout: post
title:  "深入理解计算机系统学习笔记七"
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

### RIO 的无缓冲输入输出函数
* rio_readn 和 rio_writen 函数，应用程序可以在内存和文件之间直接传送数据
* 两函数可能被应用信号处理程序的返回中断。那么每个函数会重启 read 和 write。允许被中断的系统调用。

```cpp
#include "csapp.h"
// 成功则返回传送的字节数，不成功则返回-1
// 对 rio_readn 而言，遇到 EOF 则返回 0
ssize_t rio_readn(int fd, void *usrbuf, size_t n);
ssize_t rio_writen(int fd, void *usrbuf, size_t n);

ssize_t rio_read(int fd, void *usrbuf, size_t n){
  size_t nleft = n;
  ssize_t nread;
  char *bufp = usrbuf;
  
  while(nleft > 0){
    if ((nread = read(fd, bufp, nleft)) < 0){
      if (errno == EINTR) // 被中断信号
        nread = 0;        // 被中断，重新调用 read 读取
      else
        return -1;
    }
    else if (nread == 0) {
      break;
    }

    nleft += nread;
    bufp += nread;
  }

  return n - nleft;
}

ssize_t rio_writen(int fd, void *usrbuf, size_t n){
  size_t nleft = n;
  ssize_t nwritten;
  char *bufp = usrbuf;
  
  while(nleft > 0){
    if ((nwritten = write(fd, bufp, nleft)) <= 0){
      if (errno == EINTR) // Interruted by sig handler return
        nwritten = 0;     // call write again
      else
        return -1;
    }
    
    nleft -= nwritten;
    bufp += nwritten;
  }
  
  return n;
}
```

### RIO 的带缓冲的输入函数
* `rio_readnb` 和 `rio_readlineb` 带缓冲区的读入版本，分别读入最多 n 个字符和读入一行。
* 带缓冲区的读是在 `rio_read(rio_t, void*, size_t)` 基础上实现的。
* `rio_readnb` 和 `rio_readlineb` 可以在同一个描述符上混合使用，但是他们和 `rio_read(int, void*, size_t)` 不可混用。

```cpp
#include "csapp.h"

// 定义 rio_t 结构体
#define RIO_BUFSIZE 8192
typedef struct{
  int rio_fd;                 // fd
  int rio_cnt;                // 未读字符计数
  char *rio_bufptr;           // 下一个未读缓冲区位置
  char rio_buf[RIO_BUFSIZE];  // 内部缓冲区
} rio_t;

void rio_readinitb(rio_t *rp, int fd){
  rp->rio_fd = fd;
  rp->rio_cnt = 0;
  rp->rio_bufptr = rp->rio_buf;
}

// 带缓冲区的读取 rio_read，一次读入，利用系统调用读入足够长度的字符串
static ssize_t rio_read(rio_t *rp, char *usrbuf, size_t n){
  int cnt;

  while(rp->rio_cnt <= 0){
    rp->rio_cnt = read(rp->rio_fd, rp->rio_buf, sizeof(rp->rio_buf));

    if (rp->rio_cnt < 0){
      if (errno != EINTER){
        return -1;
      }
    }
    else if (rp->rio_cnt == 0)
      return 0;
    else
      rp->rio_bufptr = rp->rio_buf;
  }

  cnt = n;
  if (rp->rio_cnt < n)
    cnt = rp->rio_cnt;
  memcpy(usrbuf, rp->rio_bufptr, cnt);
  rp->rio_bufptr += cnt;
  rp->rio_cnt -= cnt;
  return cnt;
}

// 成功返回读的字节数，若 EOF 则为0，出错则返回 -1
ssize_t rio_readlineb(rio_t *rp, void *usrbuf, size_t maxlen){
  int n, rc;
  char c, *bufp = usrbuf;

  for (n = 1; n < maxlen; n++){
    if ((rc = rio_read(rp, &c, 1)) == 1){
      *bufp++ = c;
      if (c == '\n') {
        n++;
        break;
      }
    }
    else if (rc == 0){
      if (n == 1)
        return 0;   // EOF 没有数据
      else
        break;      // EOF 有一些数据
    }
    else{
      return -1;    // 错误
    }
  }
  *bufp = 0;
  return n-1;
}

ssize_t rio_readnb(rio_t *rp, void *usrbuf, size_t n){
  ssize_t nleft = n;
  ssize_t nread;
  char *bufp = usrbuf;

  while (nleft > 0){
    if ((nread = rio_read(rp, bufp, nleft)) < 0)
      return -1;            // errno set by read
    else if (nread == 0)
      break;                // EOF
    nleft -= nread;
    bufp += nread;
  }

  return (n - nleft);       // return >= 0
}

// 示例程序
int main(int argc, char ** argv){
  int n;
  rio_t rio;
  char buf[MAXLINE];
  
  Rio_readinitb(&rio, STDIN_FILENO);
  while((n = Rio_readlineb(&rio, buf, MAXLINE)) != 0)
    Rio_writen(STDOUT_FILENO, buf, n);
}
```

## 读取文件元数据
* 通过 stat 和 fstat 检索文件的信息（也称为文件的元数据）。
* Linux 在 `sys/stat.h` 中定义了宏谓词来确定 st_mode 成员类型：`S_ISREG(m)`、`S_ISDIR(m)` 和 `S_ISSOCK(m)`，分别判断是否是一个普通文件、目录文件、网络套接字。

```cpp
#include <unistd.h>
#include <sys/stat.h>

// 成功返回0，失败返回-1
int stat(const char *filename, struct stat *buf);
int fstat(int fd, struct stat*buf);

struct stat{
dev_t st_dev;             /* Device */
ino_t st_ino;             /* inode */
mode_t st_mode;           /* Protection and file type */
nlink_t st_nlink;         /* Number of hard links */
uid_t st_uid;             /* User ID of owner */
gid_t st_gid;             /* Group ID of owner */
dev_t st_rdev;            /* Device type (if inode device) */
off_t st_size;            /* Total size, in bytes */
unsigned long st_blksize; /* Block size for filesystem I/O */
unsigned long st_blocks;  /* Number of blocks allocated */
time_t st_atime           /* Time of last access*/
time_t st_mtime           /* Time of last modification */
time_t st_ctime           /* Time of last change */
};

int main(int argc, char **argv){
  struct stat stat;
  char *type, *readok;

  Stat(argv[1], &stat);
  if (S_ISREG(stat.st_mode))
    tpye = "regular";
  else if (S_ISDIR(stat.st_mode))
    type = "directory";
  else
    type = "other";
  
  if (stat.st_mode & S_IRUSR)
    readok = "yes";
  else
    readok = "no";
  
  printf("type: %s, read: %s\n", type, readok);
  exit(0);
}
```

## 读取目录内容
* readdir 系列函数读取目录的内容。

```cpp
#include <sys/types.h>
#include <dirent.h>
#include "csapp.h"

struct dirent{
  ino_t d_ino;        // 索引号，文件位置
  char d_name[256];   // 文件名
}

// 成功则返回处理的指针，出错则返回 NULL
DIR *opendir(const char *name);

int closedir(DIR *dirp);

// 成功则指向下一个目录项的指针；失败则返回 NULL
struct dirent *readdir(DIR *dirp);

int main(int argc, char **argv){
  DIR *streamp;
  struct dirent *dep;

  streamp = Opendir(argv[1]);

  errno = 0;
  while((dep = readdir(streamp)) != NULL){
    printf("Found file: %s\n", dep->d_name);
  }

  if (errno != 0)
    unix_error("readdir error");

  Closedir(streamp);
  exit(0);
}

```

## 共享文件
* 共享 Linux 文件的概念，需要了解内核是如何表示打开的文件的。内核用三个数据结构表示打开的文件：
  * 描述符表 descriptor table：每个进程都有独立的描述符表，表项由进程打开的文件描述符来索引。每个打开的描述符表项指向文件表中的一个表项。
  * 文件表 file table:打开文件的集合由一张文件表来表示，所有进程共享这张表。每个文件表的表项组成：当前文件位置、引用计数 reference count（指向当前表项的描述表表项的数目），以及一个指向 v-node 表中对应表项的指针。
  * v-node 表 v-node table：所有进程共享这张表，每个表项包含 stat 结构中的大多数信息，包括 st_mode 和 st_size 成员。
* 每个进程一张描述符表。每个描述符一张文件表。每个文件一个 v-node 表。
* 通过不同描述符索引同一个文件，则会多个文件表对应一个 v-node 表，每个文件表有一个的文件位置。
* 不同进程可通过同一个描述符引用同一个文件，则会有不同描述符表的表项指向同一文件表。

## I/O 重定向
* Linux shell 提供了 I/O 重定向操作符，允许用户将磁盘文件和标准输入输出联系起来。
* `ls > foo.txt` 可将输入重定向到磁盘文件 foo.txt。
* `dup2` 赋值描述符表项 oldfd 到描述符表表项 newfd，覆盖描述符表项 newfd 以前的内容。如果 newfd 已经打开，dup2 会在复制 oldfd 之前关闭 newfd。

```cpp
#include <unistd.h>
// 成功则返回非负的描述符，出错则返回-1
// 此处是把 newfd 对应的指针指向了 oldfd 对应的文件表。
int dup2(int oldfd, int newfd);
```

## 标准 I/O
* C 语言定义了一组高级输入输出函数，称为标准 I/O 库。
* 库 `libc` 提供了打开和关闭文件的函数 `fopen` 和 `fclose`，读和写字节的函数 `fread` 和 `fwrite`，读和写字符串的函数 `fgets` 和 `fputs`，以及更复杂的格式化 I/O 函数 `scanf` 和 `printf`。
* 标准 I/O 库将一个打开文件模型化为一个流。一个流指的是一个指向 FILE 类型的结构的指针。每个 ANSI C 程序开始时都打开了三个流 stdin、stdout、stderr，分别是标准输入、标准输出和标准错误。
* FILE 是对文件<span style="color:red">描述符和流缓冲区</span>的抽象。

### I/O 函数的选择
* 上述过程提到的 I/O 操作有：Unix 系统 I/O，RIO 和 标准 I/O 函数。
* 选择标准：
  * 只要有可能，使用标准 I/O。
  * 不要使用 scanf 或 rio_readlineb 来读取二进制文件。
  * 对网络套接字的 I/O 使用 RIO 函数。
* 标准 I/O 流在某种意义上是全双工的，但应用到套接字上会出现限制：
  * 跟在输出函数之后的输入函数。
  * 跟在输入函数之后的输出函数。
* 在套接字编程中，不建议使用标准 I/O 函数。需要格式化输出时，使用 sprintf 函数格式化一个字符串，用 rio_writen 将其发送到套接口。需要格式化输入时，使用 rio_readlineb 来读一个完整的文本行，然后用 sscanf 从文本行中提取不同的字段。