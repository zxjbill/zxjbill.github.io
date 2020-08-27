---
layout: post
title:  "深入理解计算机系统学习笔记三"
date:   2020-08-25 08:37:00
categories: 操作系统
tags: CS:APP 优化程序性能
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
关于程序性能优化的内容。

 <!--more-->

 高效程序需要做到的以下几点：
  1. 适当的算法和数据结构
  2. 编译器能够转化为高效执行的源代码
  3. 可实现处理器的并行执行

妨碍优化的因素：程序行为中那些严重依赖于执行环境的方面。

程序优化的工作：
1. 消除不必要的工作
2. 指令级的并行能力优化
3. 通过代码剖析大型程序

汇编代码研究：研究内循环代码，识别降低性能的属性。预测操作系统如何进行并行执行和使用处理器资源。
通过关键路径决定执行一个循环需要的时间，关键路径是循环反复执行过程中形成的数据相关链。

## 优化编译器的能力和局限性
* 内存别名使用：编译器安全优化时只能认为不同指针能指向同一个位置。内存别名使用可能造成意向不到的结果。例如下面两段程序。

```cpp
void twiddle1 (long *xp, long *yp){
    *xp += *yp;
    *xp += *yp;
}

void twiddle1 (long *xp, long *yp){
    *xp += 2 * *yp;
}
```

```cpp
x = 1000; y = 3000;
*p = y;
*q = x;
t1 = *p; // 可能为 1000 或 3000，因为不确定内存地址 p 和 q 是否为同一地址
```

* 函数调用：函数的调用可修改全局程序状态的一部分，会导致每次调用对环境有改变。编译器只能认为每个函数调用都可能有副作用，所以会保持所有函数调用方式不变。
* 适当采用内联函数可减小函数调用开销，并且允许对展开代码进行进一步优化。GCC 通过命令行 "-finline" 指示，或通过优化等级 -o1 或者更高等级来实现。但是目前 GCC 仅尝试再单个文件中的函数内联。内联替换消除函数调用会导致符号调试器 (例如 GDB) 或代码剖析方式无法正确使用。

```cpp
long f();

long func1(){
    return f() + f() + f() + f();
}

long func2(){
    return 4 * f();
}

long counter = 0;
 
long f(){
    return counter++;
}

```

## 表示程序的性能
* 每元素的周期数 Cycles Per Element CPE，标志迭代程序的循环性能。
* 处理器的时钟周期用于表示执行的指令数目。
* 循环展开技术 loop unrolling：每次迭代计算两个元素。
* 使用一段循环前向求和或求积来进行性能评估。

## 程序示例
* 后续分析以下面这段程序为基础展开。

```cpp
// 设置数据类型, 初始化值，运算符方式
#define data_t double
#define IDENT 0
#define OP +

// 定义数据结构的类型
typedef struct{
    long len;
    data_t *data;
} vec_vec, *vec_ptr;

// 根据长度创建数据结构，并返回改数结构
vec_ptr new_vec(long len)
{
    // Allocate header structure
    vec_ptr result = (vec_ptr) malloc(sizeof(vec_rec));
    data_t *data = NULL;
    if (!result)
        return NULL;
    result->len = len;
    if (len > 0){
        data = (data_t *) calloc(len, sizeof(data_t));
        if (!data){
            free((void *) result);
            return NULL;
        }
    }

    result->data = data;
    return result;
}

// 检验边界，获取数据
int get_vec_element(vec_ptr v, long index, data_t *dest){
    if (index < 0 || index >= v->len)
        return 0;
    *dest = v->data[index];
    return 1;
}

// 返回数据长度
long vec_length(vec_ptr v){
    return v->len;
}

// 返回计算值
void combine1(vec_ptr v, data_t *dest){
    long i;
    *dest = IDENT;

    for (i = 0; i < vec_length(v); i++){
        data_t val;
        get_vet_elemnet(v, i, &val);
        *dest = *dest OP val;
    }
}
```

## 消除循环的低效率
* 代码移动：将循环计算或查询的相同变量提取到循环以外仅进行一次计算。

```cpp
// 返回计算值
void combine1(vec_ptr v, data_t *dest){
    long i;
    long length = vec_length(v);
    *dest = IDENT;
    
    for (i = 0; i < length; i++){
        data_t val;
        get_vet_elemnet(v, i, &val);
        *dest = *dest OP val;
    }
}
```

## 减少过程调用
* 过程调用会带来开销，妨碍大多数形式的程序优化。
* 去掉过程调用意味着外部程序知道内部程序的数据结构能够直接使用内部数据，能正确设计获取内部数据的方式。但这样做往往破坏了程序模块性(类的封装)。有时候这样的处理是必要的。
* 去掉函数调用也并不是总能提升性能。

```cpp
data_t *get_vec_start(vec_ptr v){
    return v->data;
}

void combine3(vec_ptr v, data_t *dest){
    long i;
    long length = vec_length(v);
    data_t *data = get_vec_start(v);
    *dest = IDENT;
    
    for (i = 0; i < length; i++){
        *dest = *dest OP data[i];
    }
}
```

## 消除不必要的内存引用
* 采用变量来存储值(会采用寄存器存储)，和采用内存地址存储值，两种方式带来的效率影响完全不同。
* 寄存器存储值可直接用于计算，不用先从内存读取。
* 由于受到内存别名使用的限制，编译器可能无法优化使用内存存储变量的情况。
* 由于上一点的缘故，注意 `combine4` 和 `combine3` 并不是等价的。

```cpp
void combine4(vec_ptr v, data_t *dest){
    long i;
    long length = vec_length(v);
    data_t *data = get_vec_start(v);
    data_t acc = IDENT;
    
    for (i = 0; i < length; i++){
        acc = acc OP data[i];
    }

    *dest = acc;
}
```

## 理解现代处理器
* 进一步提高性能，考虑利用到处理器<span style="color:ret">微体系结构</span>的优化。
* 理解指令级并行。多条指令可以并行地执行，有呈现出一种简单顺序执行指令的现象。
* 两种下界描述程序最大性能：
  * 延迟界限：当程序必须严格顺序执行时。下一条指令开始前，这条指令必须结束。
  * 吞吐量界限：由于处理器功能单元的原始计算能力限制。这个时程序性能的终极界限。

### 整体操作
* 超标量 superscalar：意思是每个时钟周期可执行多个操作，并且是 out-of-order。
* 分为两个部分：指令控制单元 Instruction Control Unit，ICU 和 执行单元 Execution Unit，EU。如书上 358 页，图5-11。
* ICU 从指令高速缓存中读取指令。
* 分支预测 branch prediction，投机执行 speculative execution 技术。被用于预测分支的目标地址，并提前取出预测分支的指令，并对指令进行译码。
* 取指控制块包括分支预测。
* 指令译码逻辑接受实际程序指令，并将他们转换程一组基本操作(有时称为位操作)。
* EU 接收来至取指单元的操作。每个时钟周期会接受多个操作，这些操作会被分派到一组功能单元中，进行实际执行。
* 读写内存是由加载和存储单元实现的。均通过一个加法器来完成地址计算，通过数据<span style="color:ret">高速缓存 data cache </span>来访问内存。
* 退役单元 retirement unit 用于记录正在进行的处理，确保遵守机器级程序的顺序语义。其中有寄存器文件，内含整数、浮点数和最近的SSE、AVX寄存器。内部指令信息被保存在一个先进先出的队列中，这些信息会有两个选择：预测正确，则退役 retired，预测错误则清空 flushed。
* 任何对程序寄存器的更新都只会在指令退役时发生。执行单元可以直接将结果发送给彼此。
* 寄存器重命名 register renaming：控制操作数在执行单元传送的最常见机制。通过一张重命名表记录操作结果。

### 功能单元的性能
* 延迟 latency：完成运算所需的总时间。
* 发射时间 issue time：两个连续的同类型运算之间需要的最小时钟周期数。
* 容量 capacity：能够执行该运算的功能单元数量。
* 处理器吞吐量：对一个容量为 C，发送时间为I的操作来说，可获得的吞吐量为每个时钟周期 C/I 个操作。

### 处理器操作的抽象模型
* 数据流 data-flow：展示不同操作之间的数据相关是如何限制他们的执行顺序。这些限制形成了图中的关键路径 critical path，这是执行一组机器指令所需时钟周期的一个下届。
* 循环片段寄存器类型：只读，只写，局部，循环。

## 循环展开
* 执行循环展开的关键路径和没有执行循环展开的关键路径相同。
* 编译器可很容易的执行循环展开，只要优化等级够高。优化等级3或者更高等级调用 GCC 可执行循环展开。

```cpp
void combine5(vec_ptr v, data_t *dest){
    long i;
    long length = vec_length(v);
    long limit = length - 1;
    data_t *data = get_vec_start(v);
    data_t acc = IDENT;

    for (i = 0; i < limit; i+=2){
        acc = (acc OP data[i]) OP data[i + 1];
    }

    for (; i < length; i++){
        acc = acc OP data[i];
    }

    *dest = acc;
}
```

## 提高并行性
### 多个积累变量
* 多个积累量可加快程序执行。
* 虽然乘法是更复杂的运算，但浮点数乘法可达到的吞吐量几乎是浮点数加法的两倍。
* 只有保持执行该操作的所有功能单元的流水线都是满的，程序才能达到这个操作的吞吐量极限。对于延迟为 L，容量为 C，这就要求循环展开因子 $k \geqslant L \times C$。
* 由于补码具有可交换和可结合行，溢出时亦是如此。因此在整数运算时，所有可能的情况， `combine6` 和 `combine5` 计算结果都相同。
* 浮点数的加法和乘法时不可结合的。由于四舍五入或溢出，`combine6` 和 `combine5` 计算结果可能不相同。
* 往往这样的修改，是因为“性能的翻倍”比“奇怪数据模式产生不同结果的风险”更重要。但应该与潜在用于协商是否有特殊条件出现。也因此大多数编译器不会尝试对浮点数进行这种变换。

```cpp
// 多个积累变量展开 k * k 展开. 此处为 2 * 2 展开
void combine6(vec_ptr v, data_t *dest){
    long i;
    long length = vec_length(v);
    long limit = length - 1;
    data_t *data = get_vec_start(v);
    data_t acc0 = IDENT;
    data_t acc1 = IDENT;

    for (i = 0; i < limit; i+=2){
        acc0 = acc0 OP data[i] ;
        acc1 = acc1 OP data[i + 1];
    }

    for (; i < length; i++){
        acc0 = acc0 OP data[i];
    }

    *dest = acc0 OP acc1;
}
```

### 重新结合变换
* 结合方式的改变：称为重新结合变换 reassociation transformation。
* 此方法可以将一些计算脱离关键路径。
* 同样这种方式对于整数的乘法和加法没有影响，对<span style="color:red">浮点数的加法和乘法</span>需要再次评估重新结合的严重影响。
* 当前版本 GCC 会对整数运算执行重新结合，但并不一定都有效果。循环展开，并行积累多个值是更可靠的方法。

```cpp
// 重新结合变换 2 * 1a 展开
void combine7(vec_ptr v, data_t *dest){
    long i;
    long length = vec_length(v);
    long limit = length - 1;
    data_t *data = get_vec_start(v);
    data_t acc = IDENT;

    for (i = 0; i < limit; i+=2){
        acc = acc OP (data[i] OP data[i + 1]);
    }

    for (; i < length; i++){
        acc = acc OP data[i];
    }

    *dest = acc;
}
```

### 用向量指令达到更高的并行度
* SSE Streaming SIMD Extensions 流SIMD拓展。
* SIMD Single-Instruction, Multiple-Data 单指令多数据。
* AVX advanced vector extension 高级向量拓展。使用特殊的向量寄存器 vector register， `%ymm0 - %ymm15`
* 例如 `vmulps (%rcx), %ymm0, %ymm1` 会并行执行 8 个乘法，并将得到的 8 个乘积保持到向量寄存器 `%ymm1` 中。
* AVX 不包含64位整数的并行乘法。    // TODO 验证一下

## 一些限制因素
### 寄存器溢出
* 并行度 p 超出了可用的寄存器数量，诉诸溢出 spilling。 会导致将某些临时值存放到内存中，通常在运行时堆栈上分配空间。
* 会导致多个积累变量优势消失。
* 而 x86-64 有 16 个寄存器，16 个 YMM 寄存器存浮点数，一般是足够多的，大多数循环出现寄存器溢出之前就将达到吞吐量限制。

### 分支预测和预测错误处罚
* 条件传送指令替代传统的条件转移实现。通过条件传送指令可将分支实现为普通指令流水线处理的一部分，比米娜猜测条件是否满足，因此猜测错误页没有处罚。
* 关于分支预测有以下原则：
  * 不要过分关心可预测的分支。现代处理器对有规律的模式和长期趋势有较好的预测。
  * 书写适合用条件传送实现的代码。

```cpp
// 随机数据上 CPE 大约 13.5。对于可预测的数据 CPE 为2.5-3.5
void minmax1(long a[], long b[], long n) {
    long i;
    for (i = 0; i < n; i++) {
        if (a[i] > b[i]) {
            long t = a[i];
            a[i] = b[i];
            b[i] = t;
        }
    }
}

// 功能性风格重写条件操作
// 无论是随机的还是可预测的，CPE 大约为4.0。
void minmax2(long a[], long b[], long n) {
    long i;
    for (i = 0; i < n; i++) {
        long min = a[i] < b[i] ? a[i] : b[i];
        long min = a[i] < b[i] ? b[i] : a[i];
        a[i] = min;
        b[i] = max;
    }
}
```

## 理解内存的性能
### 加载的性能
* 加载操作会产生延时，加载单元也有个数限制。
* 确定一台机器的加载延迟，可通过类似链表结构确定。上一条的加载地址决定下一条加载操作的地址。会得到 CPE 的值于 L1 级 cache 的 4 周期访问时间是一致的。

### 储存的性能
* 如果写/读相关 write/read dependency则会导致性能下降。
* 在加载一个地址时，存储单元缓冲区地址有没有要读取的地址，有则必须等待写入。没有相关性时，可通过存储缓冲区将储存操作类似后台执行，读取速度不受限。而有相关性时，则会出现存储缓冲区不能很好工作的情况。

## 应用：性能提高技术
* 高级设计：采用适当的算法和数据结构，避免使用会产生糟糕性能的算法或编码技术。
* 基本编码原则。
  * 消除连续的函数调用。有选择地妥协程序的模块化。
  * 消除不必要的内存引用。适当采用中间变量。
* 低级优化。结构化代码使用更多硬件功能。
  * 展开循环，降低开销。
  * 多个累积变量和重新结合技术。
  * 使用功能性风格重写条件操作，使得编译采用条件数据传送。

## 确认和消除性能瓶颈
* 如何使用代码剖析程序 code profiler，在程序执行时收集性能数据的分析工具。
* 系统优化的通用原则：Amdahl 定律。

### 程序剖析
* GPROF Unix 系统提供的剖析程序。
  * 确定每个函数被调用的所花费的 CPU 时间。
  * 确定每个函数被调用的次数。
* GPROF 剖析的步骤：
  1. 程序为剖析而编译和链接。添加关键字 `-pg`。使用 `-Og` 确保正确跟踪函数调用。
  2. 正常运行程序。会产生一个文件 gmon.out。
  3. 用 GPROF 来分析 gmon.out 的数据。

```bash
# 步骤一
linux> gcc -Og -pg prog.c -o prog

# 步骤二
linux> ./prog file.txt

# 步骤三
linux> gprof prog
```

* 报告通常包含两部分
  * 各个执行函数花费的时间及执行次数等信息。按照降序排列。通常 GPROF 不会显示库函数的调用，可用通过“包装函数”来查看库函数的调用情况。
  * 函数的调用历史。
* 需要注意的属性：
  * 即使不是很准确。根据某个时间间隔进行计时。对于运行时间少于1s的程序来说，统计数字只能看成粗略估计。
  * 假设没有执行内联替换，则调用信息可靠。
  * 默认不会对库函数进行计时。库函数计时会被算到其调用函数的时间中。