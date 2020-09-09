---
layout: post
title:  "Linux 服务器信号系统"
date:   2020-09-07 20:39:00
categories: Linux
tags: CS:APP 网络编程
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
Linux 的信号通信机制相关情况总结。主要包含信号概述，和一些实例。
<!--more-->

## Linux 信号概述
### 发送信号
* 通过 C 程序和 shell 都可以给进程发送信号，API 都是 kill。
* 其中 pid 用于标记获得信号的进程：
  * pid > 0 信号发送给 pid 进程
  * pid = 0 发送给本进程组的其他进程
  * pid = -1 发送给除 init 进程以外的所有进程，需要发送者拥有对目标进程发送信号的权限
  * pid < -1 发送给组 ID 为 -pid 的进程组其他所有成员
* sig 指代信号值。
* 特例：
  * sig = 0 则不发送信号，用于检测目标进程 pid 或目标进程组 -pid 是否存在。但由于进程 pid 回绕，可能导致检测的 pid 不是期望被检测进程，并且检测方法并不是原子操作。
* 返回值：
  * 成功返回0
  * 失败返回 -1，并设置 errno，其中 EINVAL 无效的信号，EPERM 没有发送信号给任意一个目标进程的权限，ESRCH 目标进程或进程组不存在。

```cpp
#include <sys/types.h>
#include <signal.h>
int kill(pid_t pid, int sig);
```

### 信号处理方式
* 目标进程在接收信号时，需要定义一个接收函数来处理信号，处理函数的原型如下：

```cpp
#include <signal.h>
void (*signal(int signum, void (*handler))(int))(int);
// 或看作如下亦可
typedef void (* _sighandler_t)(int);
_sighandler_t signal(int signum, _sighandler_t handler));
``` 

* 一般将信号处理函数设计成可重入的，避免引发一些竞态条件。所以一般不调用不安全的函数。
* 除了自定义信号处理函数外，`bits/signum.h` 头文件中定义了两种其他处理方式，`SIG_IGN` 和 `SIG_DEL`，分别时忽略目标信号和默认处理方式。默认处理方式包括如下几种：结束进程 Term、忽略信号 Ign、结束进程并生成核心转储文件 Core、暂停进程 Stop 和 继续进程 Cont。
* 信号如表所示，其中信号值与系统指令集有关，除了一些常用的，有许多信号对应的信号值不一定相同。

| 信号     |  信号值      | 默认处理行为 | 发出信号的原因           |
|:-------|:------------|:----|:-----|
| SIGHUP | 1 | A | 终端挂起或者控制进程终止 |
| SIGINT | 2 | A | 键盘中断（如break键被按下） |
| SIGQUIT | 3 | C | 键盘的退出键被按下 |
| SIGILL | 4 | C | 非法指令 |
| SIGTRAP | 5 | C | 断点陷阱，用于调试 |
| SIGABRT | 6 | C | 由abort(3)发出的退出指令 |
| SIGFPE | 8 | C | 浮点异常 |
| SIGKILL | 9 | AEF | Kill信号 |
| SIGSEGV | 11 | C | 无效的内存引用 |
| SIGPIPE | 13 | A | 管道破裂: 写一个没有读端口的管道 |
| SIGALRM | 14 | A | 由alarm(2)发出的信号 |
| SIGTERM | 15 | A | 终止信号 |
| SIGUSR1 |  | A | 用户自定义信号1 |
| SIGUSR2 |  | A | 用户自定义信号2 |
| SIGCHLD |  | B | 子进程结束信号 |
| SIGCONT |  | | 进程继续（曾被停止的进程） |
| SIGTTIN |  | D | 后台进程企图从控制终端读 |
| SIGSTOP |  | DEF | 终止进程 |
| SIGTSTP |  | D | 控制终端（tty）上按下停止键 |
| SIGTTOU |  | D | 后台进程企图从控制终端写 |

其中 A：Term, B: Ign, C: Core, D: Stop, E: 不能捕捉信号, F: 不能被忽略。

## 信号函数
### signal 系统调用
* 使用 `signal` 为信号设置处理函数
* `sig` 信号类型，`_handler` 函数指针，用于处理该类型信号的函数指针。
* 调用成功则返回函数指针 `_handler`，失败时返回 `SIG_ERR`，并设置 errno。

```cpp
#include <signal.h>
_sighandler_t signal (int sig, _sighandler_t _handler);
```

### sigaction 系统调用
* 更健壮的信号处理函数设置接口 `sigaction`
* `sig` 信号类型，`act` 信号的处理方式，`oact` 信号之前的处理方式。
* `sigaction` 成功返回 0，失败返回 -1。
* ``

```cpp
#include <signal.h>

struct sigaction{
#ifdef __USE_POIX199309
  union{
    _sighandler_t sa_handler;
    void (*sa_sigaction) (int, siginfo_t*, void*);
  } __sigaction_handler;
#define sa_handler _sigaction_handler.sa_handler
#define sa_sigaction _sigaction_handler.sa_sigaction
#else
  _sighandler_t sa_handler;
#endif

  _sigset_t sa_mask;
  int sa_flags;
  void (*sa_restorer) (void); // 一般不使用
}

int sigaction(int sig, const struct sigaction* act, struct sigaction* oact);
```