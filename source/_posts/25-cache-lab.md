---
url: cache-lab
title: 缓存实验
date: 2020-05-11 16:14:23
categories: [技术]
tags: [计算机系统实验]
mathjax: true
---

CS:APP--Lab assignments #7 - Cache Lab: Understanding Cache Memories

<!--more-->

***若参考或转载请注明出处***

本实验将帮助你了解缓存对C程序性能的影响。

实验由两部分组成：

- 在第一部分中，你将编写一个小的C程序（大约200～300行），该程序模拟高速缓存的行为；
- 在第二部分中，你将优化一个小型矩阵转置函数，其目标是最大程度地减少高速缓存未命中的次数。

# 准备

## Files of the Assignment

你必须在64位x86-64计算机上运行此实验。

首先将`cachelab-handout.tar`复制到Linux目录中，然后发出命令

```shell
linux> tar xvf cachelab-handout.tar
```

这将创建一个名为`cachelab-handout`的目录，其中包含许多文件。你将要修改两个文件：`csim.c`和`trans.c`。要编译这些文件，请键入：

```shell
linux> make clean
linux> make
```

**警告**：不要让Windows WinZip程序打开你的`.tar`文件（许多Web浏览器设置为自动执行此操作），而应该将文件保存到Linux目录，然后使用Linux tar程序提取文件。通常，对于此类文件，你永远不要使用Linux以外的任何平台来修改文件。这样做可能会导致数据丢失（包括你所做的重要的工作！）。

## Reference Trace Files

首先安装`valgrind`：

```bash
sudo apt install valgrind
```

查看版本号以检测是否安装成功：

```bash
valgrind --version
```

实验文件目录的`traces`子目录包含参考跟踪文件的集合，我们将使用它们来评估你在Part A中编写的缓存模拟器的正确性。跟踪文件由名为`valgrind`的Linux程序生成。例如，在命令行中输入

```shell
linux> valgrind --log-fd=1 --tool=lackey -v --trace-mem=yes ls -l
```

运行可执行程序“`ls -l`”，按照它们发生的顺序捕获对其每个内存访问的跟踪，并在标准输出上打印它们。`Valgrind`内存跟踪具有以下形式：

```shell
I 0400d7d4,8
 M 0421c7f0,4
 L 04f6b868,8
 S 7ff0005c8,8
```

每行表示一个或两个内存访问。每行的格式是

```shell
[space]operation address,size
```

其中：

1. `operation`字段表示内存访问的类型：

   - “`I`”表示指令加载，
   - “`L`”表示数据加载，
   - “`S`”表示数据存储，
   - “`M`”表示数据修改（即数据加载+数据存储）。

   每个“`I`”之前都没有空格。每个“`M`”、“`L`”和“`S`”之前总是有一个空格。

2. `address`字段指定64位十六进制内存地址。

3. ` size`字段指定操作访问的字节数。

# Part A: Writing a Cache Simulator

> 在Part A中，你将在`csim.c`中编写一个缓存模拟器，该模拟器以`valgrind`内存跟踪为输入，在该跟踪上模拟缓存的命中/未命中行为，并输出命中、未命中和逐出的总数。
>
> 我们为你提供了参考缓存模拟器的二进制可执行文件，称为`csim-ref`，它可在`valgrind`跟踪文件上模拟具有任意大小和关联性的缓存行为。 在选择逐出哪个缓存行时，它使用LRU（最近使用）替换策略。
>
> 参考模拟器采用以下命令行参数：
>
> 用法：
>
> ```shell
> ./csim-ref [-hv] -s <s> -E <E> -b <b> -t <tracefile>
> ```
>
> - `-h`：打印使用情况信息的可选帮助标志
> - `-v`：显示跟踪信息的可选详细标志
> - `-s <s>`：设置的索引位数（$S=2^{s}$是设置的数量）
> - `-E <E>`：关联性（每组的行数）
> - `-b <b>`：块位数（$B=2^{b}$是块大小）
> - `-t <tracefile>`：要重放的`valgrind`跟踪的名称
>
> 命令行参数基于CS:APP3e中文版教科书第426页的符号（s，E和b）。
>
> Part A的工作是填写`csim.c`文件，以便它采用相同的命令行参数并产生与参考模拟器相同的输出。 请注意，该文件几乎完全为空。 你需要从头开始编写。
>
> **Programming Rules for Part A**
>
>- 在`csim.c`的标题注释中包括你的姓名和ID。
>
>- 你的`csim.c`文件必须在没有`warnings`的情况下进行编译才能获得分数。
>
> - 你的模拟器必须对任意`s`、`E`和`b`正确工作。这意味着你将需要使用`malloc`函数为模拟器的数据结构分配存储空间。键入“`man malloc`”以获取有关此功能的信息。
>
> - 在本实验中，我们仅对数据高速缓存性能感兴趣，因此你的模拟器应忽略所有指令高速缓存访问（以“`I`”开头的行）。回想一下，`valgrind`总是将“`I`”放在第一列（没有前导空格）中，并且将“`M`”，“`L`”和“`S`”放在第二列（带有前导空格）中。这可以帮助你解析跟踪。
>
> - 要获得Part A的分数，你必须在主函数末尾调用函数`printSummary`，其中包含命中，未命中和逐出的总数：
>
>   ```c
>   printSummary(hit_count, miss_count, eviction_count);
>   ```
>
>- 对于本实验，你应该假设内存访问已正确对齐，以使单个内存访问永远不会越过块边界。通过作出此假设，你可以忽略`valgrind`跟踪中的请求大小。

查看参考缓存模拟器`csim-ref`时可能会出现：

```bash
$ ./csim-ref -v -s 4 -E 1 -b 4 -t traces/yi.trace
bash: ./csim-ref: 权限不够
```

无论是否在ubuntu中解压都会提示权限不够，此时需要增加权限：

```bash
$ chmod +x csim-ref
```

后续仍可依此方法添加权限。

再次运行，得到输出如下：

```bash
L 10,1 miss
M 20,1 miss hit
L 22,1 hit
S 18,1 hit
L 110,1 miss eviction
L 210,1 miss eviction
M 12,1 miss eviction hit
hits:4 misses:5 evictions:3
```

我们的目的就是编写一个和它功能相同的高速缓存模拟器。

*（没关系，前面可能看不懂，下面才是正文。）*

## 什么是高速缓存？

**高速缓存（cache）**是一个小而快速的存储设备，作为存储在更大、也更慢的设备中的数据对象的**缓冲区域**。

设想你需要利用一些工具去完成一项任务，第一次使用一个工具时，你会从工具箱中拿出它，用完之后你会放在手边，以便下次使用。如果手边的空间有限，你需要把比较不常用的工具放回工具箱中，把最常使用的工具留在手边。

这就类似高速缓存的思想。

实际上，高速缓存与下一层存储器之间是以**块（block）**作为传送单元（transfer unit）来传送数据的（更确切地说，应该是复制），即数据被划分成了若干块来进行复制，这就好比上面提到的“工具”。

当我们需要某个数据时，如果它恰好在“手边”，即它所在的块已经缓存在高速缓存中，这就是**缓存命中（cache hit）**；若不在则为**缓存不命中（cache miss）**。

当某个块出现缓存不命中时，我们需要将它加载到缓存中。如果缓存中有空位，则直接复制到那个位置。如果缓存已满，则只能覆盖掉一个缓存中现存的块，这个过程就是**替换（replacing）**或**驱逐（evicting）**，被驱逐的称为**牺牲块（victim block）**。

该驱逐哪一个呢？这就需要一种策略，叫**替换策略（replacement policy）**：

- 随便驱逐一个，即**随机替换策略**；
- 驱逐最近最不常用的那个，即**LRU替换策略**。

都哪些情况会出现缓存不命中呢？

- 开始的时候，“手边”是空的，当然没有数据对象在缓存中，这就是**冷不命中（cold miss）**或**强制性不命中（compulsory miss）**，这个空的缓存叫**冷缓存（cold cache）**，需要暖身（warmed up）；
- 类似于哈希表，缓存从下一层取出的块需要按照一定的放置策略（类似于哈希函数）来放到缓存中，若两个数据对象映射到同一个缓存块中，这就发生了**冲突不命中（conflict miss）**；
- 程序的每个阶段需要访问缓存块的某个相对稳定不变的集合，称为**工作集（working set）**。当缓存太小，不能容纳这个工作集，就出现**容量不命中（capacity miss）**。

---

回到正题，这个函数需要实现高速缓存的功能，我们需要理解高速缓存的构成和工作原理。

高速缓存可看成一个数组，数组的每一个元素称为一个**高速缓存组（cache set）**，每一组包含若干个**高速缓存行（cache line）**，每一行有一个**高速缓存块（cache block）**、一个**有效位（valid bit）**和一个**标记位（tag bit）**：

<img src="https://i0.hdslb.com/bfs/album/15de9e0b5f523b481a0acda70f1d4e4904a30aa0.png" alt="高速缓存" style="zoom:50%;" />

其中：

- 存储器地址有m位（上图(b)）：
  - t位标记（行匹配时对应行的标记位）；
  - s位组索引（组选择时对应组号）；
  - b位块偏移（字选择时对应第一个字节的偏移量）；
- 数组有$S=2^{s}$组；
- 每一组有E行：
  - E=1（S=C/B），称为**直接映射高速缓存（direct-mapped cache）**；
  - 1<E<C/B，称为**组相联高速缓存（set associative cache）**；
  - E=C/B（S=1），称为**全相联高速缓存（fully associative cache）**；
- 每一行有1个有效位、t=m-(b+s)个标记位和一个块；
- 每一块有$B=2^{b}$字节；

则高速缓存可用元组(S,E,B,m)表示，总大小（所有块大小之和）C=S\*E\*B。

对应每一组的不同分行，高速缓存有三种缓存映射方式，我们设计程序时需要分别考虑。

要求中也提到了必要时需使用LRU策略。

根据作业一的经验，我们需要从命令行读取参数。

现在对我们即将要做的事已经有一个大概的了解了，打开`csim.c`：

```c
#include "cachelab.h"

int main()
{
    printSummary(0, 0, 0);
    return 0;
}
```

......看来需要我们从零开始。

## Cache 结构

根据上面的分析，高速缓存是一个大小为S的数组，数组的每一项是高速缓存组（又可以看作一个大小为E的数组），高速缓存组的每一项是高速缓存行，高速缓存行又包含了有效位、标记位和块（块没什么用，我们不需要进行字抽取，可以忽略）。

所以可以利用二维数组进行具体的实现，而且可以将高速缓存行封装成一个结构体。

注意LRU策略需要判断每个缓存块存取的时间（可以理解为距上一次更新的时间），这也是每个缓存行的属性：

```c
typedef struct {
  int valid;
  int tag;
  int time_stamp;
} cache_line;
cache_line** cache = NULL;
```

## Cache 初始化

```c
void init_cache() {
  cache = (cache_line**)malloc(sizeof(cache_line*) * S);
  for (int i = 0; i < S; i++) {
    cache[i] = (cache_line*)malloc(sizeof(cache_line) * E);
    for (int j = 0; j < E; j++) {
      cache[i][j].valid = 0;
      cache[i][j].tag = -1;
      cache[i][j].time_stamp = -1;
    }
  }
}
```

## 更新缓存块

对于输入的地址，需要更新对应位置的缓存块（缓存行），故函数的参数为地址：

```c
void update(unsigned int addr)
```

根据设定的s和b大小，确定地址的组索引和块偏移：

```c
s_addr = (addr >> b) & ((1 << s) - 1);
t_addr = addr >> (s + b);
```

根据组索引找到对应组，再在组中的E行中寻找标记位相同的行，若有则说明命中，更新其时间戳，并直接返回：

```c
for (int i = 0; i < E; i++)
  if (cache[s_addr][i].tag == t_addr) {
    cache[s_addr][i].time_stamp = 0;
    hit_count++;
    return;
  }
```

> 根据要求，需要分别用`hit_count`、`miss_count`和`eviction_count`记录命中总次数、未命中总次数和块替换（逐出）总次数。

若上一步没有返回，则说明出现了缓存不命中，继续执行看看有没有空位：

```c
for (int i = 0; i < E; i++)
  if (!cache[s_addr][i].valid) {
    cache[s_addr][i].valid = 1;
    cache[s_addr][i].tag = t_addr;
    cache[s_addr][i].time_stamp = 0;
    miss_count++;
    return;
  }
```

若上一步还是没有返回，则说明这一组已经满了，需要使用LRU策略进行驱逐即块替换，即找到这一组中最近最不常用的（最长时间未更新的、时间戳最大的）：

```c 
int max_stamp = INT_MIN, max_stamp_index = -1;
miss_count++;
eviction_count++;
for (int i = 0; i < E; i++)
  if (cache[s_addr][i].time_stamp > max_stamp) {
    max_stamp = cache[s_addr][i].time_stamp;
    max_stamp_index = i;
  }
cache[s_addr][max_stamp_index].tag = t_addr;
cache[s_addr][max_stamp_index].time_stamp = 0;
return;
```

> 使用`INT_MIN`需要包含以下头文件：
>
> ```c
> #include <limits.h>
> ```

## 更新时间戳

需要适时更新时间戳，即将所有已缓存的块的时间戳全部增1：

```c
void update_time() {
  for (int i = 0; i < S; i++)
    for (int j = 0; j < E; j++)
      if (cache[i][j].valid)
        cache[i][j].time_stamp++;
}
```

## 打印用法

```c
void printUsage()
{
    printf("Usage: ./csim-ref [-hv] -s <num> -E <num> -b <num> -t <file>\n"
            "Options:\n"
            "  -h         Print this help message.\n"
            "  -v         Optional verbose flag.\n"
            "  -s <num>   Number of set index bits.\n"
            "  -E <num>   Number of lines per set.\n"
            "  -b <num>   Number of block offset bits.\n"
            "  -t <file>  Trace file.\n\n"
            "Examples:\n"
            "  linux>  ./csim-ref -s 4 -E 1 -b 4 -t traces/yi.trace\n"
            "  linux>  ./csim-ref -v -s 8 -E 2 -b 4 -t traces/yi.trace\n");
}
```

## 主函数

需要从命令行读入参数，则主函数的格式为：

```c
int main(int argc, char* argv[])
```

### 变量初始化

前面提到的三个计数器需要初始化为0：

```c
hit_count = miss_count = eviction_count = 0;
```

### 处理参数

回想一下我们的参数都有哪几种可能：

```
Usage: ./csim [-hv] -s <num> -E <num> -b <num> -t <file>
Options:
  -h         Print this help message.
  -v         Optional verbose flag.
  -s <num>   Number of set index bits.
  -E <num>   Number of lines per set.
  -b <num>   Number of block offset bits.
  -t <file>  Trace file.
```

需要使用`getopt`函数处理传入的参数。

> 使用`getopt`函数需要包含以下头文件：
>
> ```c
> #include <getopt.h>
> #include <stdlib.h>
> #include <unistd.h>
> ```

`getopt`函数有3个参数，分别为：

- 读入参数的个数
- 参数字符串指针
- 与参数匹配的字符串

当字符表示为`x`时说明命令为`-x`；当字符表示为`x:`时说明命令`-x`后跟了一个参数（保存在`optarg`中）。

且返回值为`int`型：

- 若参数字符和指定字符匹配，则返回该字符，并移动指针到下一个字符；
- 不匹配则返回字符`'?'`，同样移动指针到下一个字符；
- 到字符串末尾，返回-1。

所以我们只需将每次`getopt`函数读入的一个字符存到`opt`中，再用`switch`判断即可：

```c
while ((opt = getopt(argc, argv, "hvs:E:b:t:")) != -1) {
  switch (opt) {
    case 'h':
      printUsage();
      break;
    case 'v':
      v = 1;
      break;
    case 's':
      s = atoi(optarg);
      break;
    case 'E':
      E = atoi(optarg);
      break;
    case 'b':
      b = atoi(optarg);
      break;
    case 't':
      strcpy(filename, optarg);
      break;
  }
}
```

> 使用字符串函数需要包含以下头文件：
>
> ```c
> #include <string.h>
> ```

### 初始化 Cache

```c
init_cache();
```

### 读取文件

使用`fopen`函数，进行读操作`"r"`：

```c
FILE* fp = fopen(filename, "r");
if (!fp) {
  printf("No such file!\n");
  exit(-1);
}
```

使用`fgets`函数读取文件中一行文本存入`buffer`，再根据这一行的格式：

```
[space]operation address,size
```

进行相关操作。根据要求，只需考虑`operation`是`L`、`M`和`S`的情况：其中加载操作`L`和存储操作`S`都至多会产生一次缓存不命中；而数据修改操作`M`是先加载后存储，可能会产生两种情况：两次内存命中或一次不命中一次命中（并进行相应的替换）：

```c
while (fgets(buffer, 1000, fp)) {
  sscanf(buffer, " %c %xu,%d", &type, &addr, &temp);
  switch(type) {
    case 'L':
      update(addr);
      break;
    case 'M':
      update(addr);
    case 'S':
      update(addr);
      break;
  }
  update_time();
}
```

### 结束

```c
for (int i = 0; i < S; i++)
  free(cache[i]);
free(cache);
fclose(fp);

printSummary(hit_count, miss_count, eviction_count);

return 0;
```

## 完整代码

```c
// Author: SuperPung

#include "cachelab.h"
#include <getopt.h>
#include <stdlib.h>
#include <unistd.h>
#include <limits.h>
#include <string.h>
#include <stdio.h>

int h, v, s, E, b, S;
int hit_count, miss_count, eviction_count;

char filename[1000];
char buffer[1000];

typedef struct {
  int valid;
  int tag;
  int time_stamp;
} cache_line;
cache_line** cache = NULL;

void init_cache() {
  cache = (cache_line**)malloc(sizeof(cache_line*) * S);
  for (int i = 0; i < S; i++) {
    cache[i] = (cache_line*)malloc(sizeof(cache_line) * E);
    for (int j = 0; j < E; j++) {
      cache[i][j].valid = 0;
      cache[i][j].tag = -1;
      cache[i][j].time_stamp = -1;
    }
  }
}

void update(unsigned int addr) {
    int max_stamp = INT_MIN, max_stamp_index = -1;
    int s_addr, t_addr;

    s_addr = (addr >> b) & ((1 << s) - 1);
    t_addr = addr >> (s + b);

    for (int i = 0; i < E; i++)
        if (cache[s_addr][i].tag == t_addr) {
            cache[s_addr][i].time_stamp = 0;
            hit_count++;
            return;
        }

    for (int i = 0; i < E; i++)
        if (!cache[s_addr][i].valid) {
            cache[s_addr][i].valid = 1;
            cache[s_addr][i].tag = t_addr;
            cache[s_addr][i].time_stamp = 0;
            miss_count++;
            return;
        }

    miss_count++;
    eviction_count++;
    for (int i = 0; i < E; i++)
    if (cache[s_addr][i].time_stamp > max_stamp) {
        max_stamp = cache[s_addr][i].time_stamp;
        max_stamp_index = i;
    }
    cache[s_addr][max_stamp_index].tag = t_addr;
    cache[s_addr][max_stamp_index].time_stamp = 0;
    return;
}

void update_time() {
  for (int i = 0; i < S; i++)
    for (int j = 0; j < E; j++)
      if (cache[i][j].valid)
        cache[i][j].time_stamp++;
}

void printUsage()
{
    printf("Usage: ./csim-ref [-hv] -s <num> -E <num> -b <num> -t <file>\n"
            "Options:\n"
            "  -h         Print this help message.\n"
            "  -v         Optional verbose flag.\n"
            "  -s <num>   Number of set index bits.\n"
            "  -E <num>   Number of lines per set.\n"
            "  -b <num>   Number of block offset bits.\n"
            "  -t <file>  Trace file.\n\n"
            "Examples:\n"
            "  linux>  ./csim-ref -s 4 -E 1 -b 4 -t traces/yi.trace\n"
            "  linux>  ./csim-ref -v -s 8 -E 2 -b 4 -t traces/yi.trace\n");
}

int main(int argc, char* argv[])
{
    int opt, temp;
    char type;
    unsigned int addr;
    hit_count = miss_count = eviction_count = 0;

    while ((opt = getopt(argc, argv, "hvs:E:b:t:")) != -1) {
        switch (opt) {
            case 'h':
            printUsage();
            break;
            case 'v':
            v = 1;
            break;
            case 's':
            s = atoi(optarg);
            break;
            case 'E':
            E = atoi(optarg);
            break;
            case 'b':
            b = atoi(optarg);
            break;
            case 't':
            strcpy(filename, optarg);
            break;
        }
    }

		S = 1 << s;
    init_cache();

    FILE* fp = fopen(filename, "r");
    if (!fp) {
        printf("No such file!\n");
        exit(-1);
    }
    while (fgets(buffer, 1000, fp)) {
        sscanf(buffer, " %c %xu,%d", &type, &addr, &temp);
        switch(type) {
            case 'L':
            update(addr);
            break;
            case 'M':
            update(addr);
            case 'S':
            update(addr);
            break;
        }
        update_time();
    }

    for (int i = 0; i < S; i++)
        free(cache[i]);
    free(cache);
    fclose(fp);

    printSummary(hit_count, miss_count, eviction_count);

    return 0;
}
```

## 效果检测

输入：

```bash
./test-csim
```

得到输出如下：

```bash
                        Your simulator     Reference simulator
Points (s,E,b)    Hits  Misses  Evicts    Hits  Misses  Evicts
     3 (1,1,1)       9       8       6       9       8       6  traces/yi2.trace
     3 (4,2,4)       4       5       2       4       5       2  traces/yi.trace
     3 (2,1,4)       2       3       1       2       3       1  traces/dave.trace
     3 (2,1,3)     167      71      67     167      71      67  traces/trans.trace
     3 (2,2,3)     201      37      29     201      37      29  traces/trans.trace
     3 (2,4,3)     212      26      10     212      26      10  traces/trans.trace
     3 (5,1,5)     231       7       0     231       7       0  traces/trans.trace
     6 (5,1,5)  265189   21775   21743  265189   21775   21743  traces/long.trace
    27

TEST_CSIM_RESULTS=27
```

成功。

# Part B: Optimizing Matrix Transpose

> 在Part B中，你将在`trans.c`中编写一个转置函数，该函数将导致尽可能少的高速缓存未命中。
>
> 设$A$表示矩阵，$A_{ij}$表示第`i`行第`j`列的分量。$A$的转置（表示为$A^{T}$)是一个矩阵，使得$A_{ij}=A_{ji}$。
>
> 为了帮助你入门，我们在`trans.c`中为你提供了一个示例转置函数，该函数计算$N\times M$矩阵$A$的转置并将结果存储在$M\times N$矩阵$B$中：
>
> ```c
> char trans_desc[] = "Simple row-wise scan transpose";
> void trans(int M, int N, int A[N][M], int B[M][N])
> ```
>
> 示例的转置函数是正确的，但是效率很低，因为访问模式会导致相对较多的缓存未命中。
>
> 在Part B中，你的工作是编写一个类似的函数，称为`transpose_submit`，该函数可最大程度地减少不同大小的矩阵之间的高速缓存未命中数：
>
> ```c
> char transpose_submit_desc[] = "Transpose submission";
> void transpose_submit(int M, int N, int A[N][M], int B[M][N]);
> ```
>
> 不要为你的`transpose_submit`函数更改描述字符串（`"Transpose submission"`）。 自动分频器搜索此字符串，以确定要评估分数的转置函数。
>
> **Programming Rules for Part B**
>
> - 在`trans.c`的标题注释中包括你的姓名和ID。
> - 你在`trans.c`中的代码必须在没有`warnings`的情况下进行编译才能获得分数。
> - 每个转置函数最多可以定义12个`int`类型的局部变量（出现此限制的原因是我们的测试代码无法计算对堆栈的引用。我们希望你限制对堆栈的引用，并专注于源阵列和目标阵列的访问模式）。
> - 不允许通过使用`long`类型的任何变量或使用任何技巧将多个值存储到单个变量中来回避上一条规则。
> - 你的转置功能可能不使用递归。
> - 如果选择使用辅助函数，则在辅助函数和顶级转置函数之间，一次最多不能有12个局部变量在堆栈上。例如，如果你的转置声明了8个变量，然后又调用了一个使用4个变量的函数，又调用了一个使用2个变量的函数，则堆栈上将有14个变量，这将违反该规则。
> - 你的转置函数可能不会修改数组`A`。但是，你可以对数组`B`的内容做任何想做的事情。
> - 不允许在代码中定义任何数组或使用`malloc`的任何变体。

需要做的，就是在`transpose_submit`函数中实现三种规模矩阵A的转置运算，使不命中的次数尽可能少。

根据注释“*A transpose function is evaluated by counting the number of misses on a 1KB direct mapped cache with a block size of 32 bytes.*”可知：

- cache的大小为1 024字节

- 每块大小为32字节

  `int`型为4字节大小，则每块可以存放8个`int`型数据。

- 为直接映射高速缓存

故缓存组有1 024/32=32组。

## 分析

**注意矩阵A的行数是N！！！**

由于矩阵是以行序为主序进行存储的，所以当连续地读取矩阵A时，一定会造成不连续地写入B。

但是当矩阵很小时，在冷不命中之后，整个矩阵都会被加载进入缓存，此时就一定会命中。（实际上矩阵很大，不要抱有这种幻想。）

所以我们需要对矩阵进行分块处理，将一个个小的矩阵加载进入缓存，以其较好的局部性来提高程序性能。

根据cache的大小（共32组，每组可存8个`int`，一共可存32\*8=256个`int`）：

- 对于32\*32的矩阵A和B，A的前8行可以存放在cache中，而第9行、第17行、第25行一定会与A的第一行发生冲突不命中；同时由于B是紧接着A进行存储的，所以B的第1行、第9行、第17行、第25行也一定会与A的第一行发生冲突不命中。即，**读取两个相差8行的`int`时一定会发生冲突不命中**。

  不仅如此，由于矩阵规模都是8的倍数，还可以发现**B的第i行一定会与A的第i行发生冲突不命中**。虽然读一行A之后会写一列而不是一行B，但位于**对角线上的元素一定会发生冲突不命中**。

- 对于64\*64的同理。

## `if (M == 32)`

由于cache最多可以存8行32列个`int`，所以选择8\*8的分块，将矩阵分成4\*4=16个小矩阵。

对于A中第一个小矩阵，当读取第一个`int`时，这一行的8个`int`都会加载入一个缓存块中。此时再写入B的第一个小矩阵的第一列，B的这一行的8个`int`会再次加载入同一个缓存块中，必发生冲突不命中。

然后我们还要读取A的小矩阵的第二个`int`，此时又会和B发生冲突不命中……

这是由于同时访问A和B的对角线元素导致的抖动，所以当读取A的第一个小矩阵的第一个`int`时，既然这一行都缓存了，那就干脆直接将这一行的8个`int`都一次性读取，再一次性写入B的第一个小矩阵的第一列，这样就不会出现反复的冲突不命中了。

```c
int row_head, col_head;
int tmp0, tmp1, tmp2, tmp3, tmp4, tmp5, tmp6, tmp7;
int i, j;
for (row_head = 0; row_head < N; row_head += 8)
  for (col_head = 0; col_head < M; col_head += 8)
    for (i = row_head; i < row_head + 8; i++) {
      tmp0 = A[i][col_head];
      tmp1 = A[i][col_head + 1];
      tmp2 = A[i][col_head + 2];
      tmp3 = A[i][col_head + 3];
      tmp4 = A[i][col_head + 4];
      tmp5 = A[i][col_head + 5];
      tmp6 = A[i][col_head + 6];
      tmp7 = A[i][col_head + 7];

      B[col_head][i] = tmp0;
      B[col_head + 1][i] = tmp1;
      B[col_head + 2][i] = tmp2;
      B[col_head + 3][i] = tmp3;
      B[col_head + 4][i] = tmp4;
      B[col_head + 5][i] = tmp5;
      B[col_head + 6][i] = tmp6;
      B[col_head + 7][i] = tmp7;
    }
```

测试：

```bash
./test-trans -M 32 -N 32
```

得到输出如下：

```bash
Function 0 (1 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 0 (Transpose submission): hits:1766, misses:287, evictions:255

Summary for official submission (func 0): correctness=1 misses=287

TEST_TRANS_RESULTS=1:287
```

miss次数已经降到287了，小于300，成功。

## `if (M == 64)`

对于64\*64的矩阵，cache最多可以存储4行。若仍按8\*8分块，则A自身就会出现冲突不命中（隔4行），显然不行。

类比一下，我们可以用4\*4的分块吗？

因为一个缓存块就可以存8个`int`，只用4个的话太浪费了，也不行。

那么，既然大矩阵可以分块，那么小矩阵也可以分块啊！

我们对8\*8的小矩阵分成4个4\*4的小小矩阵，左上角和右下角的转置算法不变，只需要处理一下左下角和右上角的转置。

- 上半部分（4行8列）：

  一次性读A的小矩阵一行（8个），读4次，每次的前4个（左上角）存入B的小小矩阵的左上角一列。这是左上角的正常转置。

  每次的后4个（右上角）为了不冲突，不能存入B的左下角；为了不浪费，可以暂时存到B的右上角一列（由于左上角转置结束，此时的上半部分对应的缓存块已经全是B了）。这时B的右上角是B的左下角最终的状态。

  ```c
  for (i = row_head; i < row_head + 4; i++) {
    tmp0 = A[i][col_head];
    tmp1 = A[i][col_head + 1];
    tmp2 = A[i][col_head + 2];
    tmp3 = A[i][col_head + 3];
    tmp4 = A[i][col_head + 4];
    tmp5 = A[i][col_head + 5];
    tmp6 = A[i][col_head + 6];
    tmp7 = A[i][col_head + 7];

    B[col_head][i] = tmp0;
    B[col_head + 1][i] = tmp1;
    B[col_head + 2][i] = tmp2;
    B[col_head + 3][i] = tmp3;

    B[col_head][i + 4] = tmp4;
    B[col_head + 1][i + 4] = tmp5;
    B[col_head + 2][i + 4] = tmp6;
    B[col_head + 3][i + 4] = tmp7;
  }
  ```

- 下半部分（4行8列）：

  需要将A左下角的转置存入B的右上角，将刚才暂存在B右上角的实际左下角的状态平移到B的左下角。

  一次性读A的小矩阵一行（8个），读4次，每次的后4个（右下角）存入B的小小矩阵的右下角一列。这是右下角的正常转置。

  每次的前4个（左下角）需要存到B的右上角，但是此时B的右上角还没取出来，此时变量个数已经达到上限，导致算法无法继续进行。一定是哪里出了问题。

  分析发现，每次取出B右上角需要4个变量，所以每次就剩下4个变量可供A转置使用。所以每次次只能读A的小一行（4个）、对应B右上角的小一列（也可以是A的小一列、B的小一行）。

  总结可以发现，A每读一行需要写B的一列、读一列需要写B的一行，所以这两种方式为后续操作造成的冲突次数是等价的。但是因为需要将B从右上角平移到左下角，即读一行就对应写一行、读一列就对应写一列，所以明显读一行导致的冲突次数更少。

  综上，处理A的左下角时，先读A的小一列、B右上角的小一行，再将A的左下角小一列存入B的右上角小一行，再将B右上角的小一行存入B左下角的小一行。

  A的右下角正常转置（因为取列，无法避免冲突，所以直接赋值即可）。

  ```c
  for (j = col_head; j < col_head + 4; j++) {
    tmp4 = A[row_head + 4][j];
    tmp5 = A[row_head + 5][j];
    tmp6 = A[row_head + 6][j];
    tmp7 = A[row_head + 7][j];

    tmp0 = B[j][row_head + 4];
    tmp1 = B[j][row_head + 5];
    tmp2 = B[j][row_head + 6];
    tmp3 = B[j][row_head + 7];

    B[j][row_head + 4] = tmp4;
    B[j][row_head + 5] = tmp5;
    B[j][row_head + 6] = tmp6;
    B[j][row_head + 7] = tmp7;

    B[j + 4][row_head] = tmp0;
    B[j + 4][row_head + 1] = tmp1;
    B[j + 4][row_head + 2] = tmp2;
    B[j + 4][row_head + 3] = tmp3;

    B[j + 4][row_head + 4] = A[row_head + 4][j + 4];
    B[j + 4][row_head + 5] = A[row_head + 5][j + 4];
    B[j + 4][row_head + 6] = A[row_head + 6][j + 4];
    B[j + 4][row_head + 7] = A[row_head + 7][j + 4];
  }
  ```

测试：

```bash
./test-trans -M 64 -N 64
```

得到输出如下：

```bash
Function 0 (1 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 0 (Transpose submission): hits:9074, misses:1171, evictions:1139

Summary for official submission (func 0): correctness=1 misses=1171

TEST_TRANS_RESULTS=1:1171
```

miss次数已经降到了1 171，小于1 300，成功。

## `if (M == 61)`

由于不规则矩阵没有明显的规律，可以尝试不同分块。

下面采用16\*16分块。

```c
for(row_head = 0; row_head + 16 < N; row_head += 16)
		for(col_head = 0; col_head + 16 < M; col_head += 16)
		{
			for(i = row_head; i < row_head + 16; i++)
			{
				tmp0 = A[i][col_head + 0];
				tmp1 = A[i][col_head + 1];
				tmp2 = A[i][col_head + 2];
				tmp3 = A[i][col_head + 3];
				tmp4 = A[i][col_head + 4];
				tmp5 = A[i][col_head + 5];
				tmp6 = A[i][col_head + 6];
				tmp7 = A[i][col_head + 7];
				B[col_head + 0][i] = tmp0;
				B[col_head + 1][i] = tmp1;
				B[col_head + 2][i] = tmp2;
				B[col_head + 3][i] = tmp3;
				B[col_head + 4][i] = tmp4;
				B[col_head + 5][i] = tmp5;
				B[col_head + 6][i] = tmp6;
				B[col_head + 7][i] = tmp7;

				tmp0 = A[i][col_head + 8];
				tmp1 = A[i][col_head + 9];
				tmp2 = A[i][col_head + 10];
				tmp3 = A[i][col_head + 11];
				tmp4 = A[i][col_head + 12];
				tmp5 = A[i][col_head + 13];
				tmp6 = A[i][col_head + 14];
				tmp7 = A[i][col_head + 15];
				B[col_head + 8][i] = tmp0;
				B[col_head + 9][i] = tmp1;
				B[col_head + 10][i] = tmp2;
				B[col_head + 11][i] = tmp3;
				B[col_head + 12][i] = tmp4;
				B[col_head + 13][i] = tmp5;
				B[col_head + 14][i] = tmp6;
				B[col_head + 15][i] = tmp7;
			}
		}
	for(i = row_head; i < N; i++)
		for(j = 0; j < M; j++)
			B[j][i] = A[i][j];
	for(i = 0; i < row_head; i++)
		for(j = col_head; j < M; j++)
			B[j][i] = A[i][j];
```

测试：

```bash
./test-trans -M 61 -N 67
```

得到输出如下：

```bash
Function 0 (1 total)
Step 1: Validating and generating memory traces
Step 2: Evaluating performance (s=5, E=1, b=5)
func 0 (Transpose submission): hits:6216, misses:1963, evictions:1931

Summary for official submission (func 0): correctness=1 misses=1963

TEST_TRANS_RESULTS=1:1963
```

miss次数为1 963，小于2 000，成功。

# Evaluation

本节介绍如何评估你的工作，该实验的总分是60分：

- Part A: 27 Points
- Part B: 26 Points
- Style: 7 Points

## Evaluation for Part A

对于Part A，我们将使用不同的缓存参数和跟踪来运行你的缓存模拟器。有8个测试用例，每个3分，最后一个用例除外，6分：

```shell
linux> ./csim -s 1 -E 1 -b 1 -t traces/yi2.trace linux> ./csim -s 4 -E 2 -b 4 -t traces/yi.trace linux> ./csim -s 2 -E 1 -b 4 -t traces/dave.trace
linux> ./csim -s 2 -E 1 -b 3 -t traces/trans.trace
linux> ./csim -s 2 -E 2 -b 3 -t traces/trans.trace
linux> ./csim -s 2 -E 4 -b 3 -t traces/trans.trace
linux> ./csim -s 5 -E 1 -b 5 -t traces/trans.trace
linux> ./csim -s 5 -E 1 -b 5 -t traces/long.trace
```

你可以使用参考模拟器`csim-ref`为每个测试用例获得正确的答案。在调试期间，请使用`-v`选项详细记录每次命中和未命中。

对于每个测试用例，输出正确数量的缓存命中、未命中和逐出将使你对该测试用例有充分的了解。 你报告的每次命中、未命中和逐出的次数都相当于该测试用例的1/3功劳。 也就是说，如果特定测试用例价值3分，并且你的模拟器输出正确的命中率和未命中数，但是报告了错误的驱逐数，那么你将获得2分。

## Evaluation for Part B

对于Part B，我们将在三种不同大小的输出矩阵上评估你的`transpose_submit`函数的正确性和性能：

- 32×32(M = 32, N = 32)
- 64×64(M = 64, N = 64)
- 61×67(M = 61, N = 67)

对于每种矩阵大小，通过使用`valgrind`提取函数的地址跟踪，然后使用参考模拟器在具有参数（s = 5，E = 1，b= 5）的高速缓存上重放此跟踪，可以评估`transpose_submit`函数的性能。

你对每种矩阵大小的性能得分与未命中数`m`具有线性关系，直至某个阈值：

- 32×32: 8 points if `m` < 300, 0 points if `m` > 600
- 64×64: 8 points if `m` < 1,300, 0 points if `m` > 2,000
- 61×67: 10 points if `m` < 2,000, 0 points if `m` > 3,000

你的代码必须正确才能接收特定大小的任何性能分。你的代码只需要针对这三种情况是正确的，并且可以针对这三种情况专门对其进行优化。特别地，函数可以明确检查输入大小并实现针对每种情况优化的单独代码，这是完全可以的。

## Evaluation for Style

代码风格有7分。这些将由教师手动分配。风格指南可在课程网站上找到。

教师将检查Part B中的代码是否存在非法数组和过多的局部变量。

# Working on the Lab

## Working on Part A

我们为你提供了一个称为`test-csim`的自动分级程序，该程序可以在线上测试缓存模拟器的正确性。在运行测试之前，请确保编译模拟器：

```shell
linux> make
linux> ./test-csim
                      Your simulator Reference simulator
Points (s,E,b)    Hits Misses Evicts   Hits Misses Evicts
     3 (1,1,1)       9      8      6      9      8      6  traces/yi2.trace
     3 (4,2,4)       4      5      2      4      5      2  traces/yi.trace
     3 (2,1,4)       2      3      1      2      3      1  traces/dave.trace
     3 (2,1,3)     167     71     67    167     71     67  traces/trans.trace
     3 (2,2,3)     201     37     29    201     37     29  traces/trans.trace
     3 (2,4,3)     212     26     10    212     26     10  traces/trans.trace
     3 (5,1,5)     231      7      0    231      7      0  traces/trans.trace
     6 (5,1,5)  265189  21775  21743 265189  21775  21743  traces/long.trace
    27
```

对于每个测试，它都会显示你获得的积分数、缓存参数、输入的跟踪文件以及模拟器和参考模拟器的结果比较。

以下是有关Part A工作的一些提示和建议：

- 对小trace（例如`traces/dave.trace`）进行初始调试。

- 参考模拟器采用可选的`-v`参数，该参数启用详细输出，显示由于每次内存访问而发生的命中、未命中和逐出。你不需要在`csim.c`代码中实现此功能，但是我们强烈建议你这样做。通过允许你直接将模拟器与参考跟踪文件上的参考模拟器的行为进行比较，它将帮助你进行调试。

- 我们建议你使用`getopt`函数来解析命令行参数。你需要以下头文件：

  ```c
  #include <getopt.h>
  #include <stdlib.h>
  #include <unistd.h>
  ```

  键入“`man 3 getopt`”以详细了解。

- 每个数据加载（L）或存储（S）操作最多可能导致一个高速缓存未命中。数据修改操作（M）被视为加载，然后将其存储到同一地址。因此，M操作可能导致两个高速缓存命中，或者未命中和命中再加上可能的逐出。

- 如果你想使用15-122中的C0样式，则可以包含头文件`contracts.h`。为方便起见，我们在文件目录中提供了。

## Working on Part B

我们为你提供了一个称为`test-trans.c`的自动分级程序，该程序可以测试你向自动分级机注册的每个移调功能的正确性和性能。

你可以在`trans.c`文件中注册多达100个版本的转置函数。 每个转置版本具有以下形式：

```c
/* Header comment */
char trans_simple_desc[] = "A simple transpose";
void trans_simple(int M, int N, int A[N][M], int B[M][N])
{
  /* your transpose code here */
}
```

在`trans.c`中的寄存器函数例程中，通过调用以下形式向自动分级器注册特定的转置函数：

```c
registerTransFunction(trans_simple, trans_simple_desc);
```

在运行时，自动分级机将评估每个已注册的转置函数并打印结果。 当然，注册函数之一必须是你要提交的`transpose_submit`函数：

```c
registerTransFunction(transpose_submit, transpose_submit_desc);
```

有关如何工作的示例，请参见默认的`trans.c`函数。

自动分级机将矩阵大小作为输入。它使用`valgrind`生成每个已注册转置函数的跟踪。然后，它通过在具有参数（s = 5，E = 1，b = 5）的缓存上运行参考模拟器来评估每个跟踪。

例如，要在32×32矩阵上测试注册的转置函数，重建`test-trans`，然后使用M和N的适当值运行它：

```shell
linux> make
linux> ./test-trans -M 32 -N 32
Step 1: Evaluating registered transpose funcs for correctness:
func 0 (Transpose submission): correctness: 1
func 1 (Simple row-wise scan transpose): correctness: 1
func 2 (column-wise scan transpose): correctness: 1
func 3 (using a zig-zag access pattern): correctness: 1

Step 2: Generating memory traces for registered transpose funcs.

Step 3: Evaluating performance of registered transpose funcs (s=5, E=1, b=5)
func 0 (Transpose submission): hits:1766, misses:287, evictions:255
func 1 (Simple row-wise scan transpose): hits:870, misses:1183, evictions:1151
func 2 (column-wise scan transpose): hits:870, misses:1183, evictions:1151
func 3 (using a zig-zag access pattern): hits:1076, misses:977, evictions:945

Summary for official submission (func 0): correctness=1 misses=287
```

在此示例中，我们在`trans.c`中注册了四个不同的转置函数。 `test-trans`程序测试每个注册功能，显示每个功能的结果，并提取结果以供正式提交。

这是有关Part B工作的一些提示和建议。

- `test-trans`程序将函数`i`的跟踪保存在文件`trace.fi`中（因为`valgrind`引入了许多与你的代码无关的堆栈访问，所以我们从跟踪中过滤掉了所有堆栈访问。 这就是为什么我们禁止使用局部数组并限制局部变量数量的原因）。
  宝贵的调试工具，可以帮助你准确了解每个转置功能的来历。 要调试特定功能，只需使用详细选项通过引用模拟器运行其跟踪：

  ```shell
  linux> ./csim-ref -v -s 5 -E 1 -b 5 -t trace.f0
  S 68312c,1 miss
  L 683140,8 miss
  L 683124,4 hit
  L 683120,4 hit
  L 603124,4 miss eviction
  S 6431a0,4 miss
  ...
  ```

- 由于正在直接映射的缓存上评估转置函数，因此冲突遗漏是一个潜在的问题。考虑一下代码中可能发生冲突遗漏的可能性，尤其是对角线。尝试考虑将减少这些冲突未命中次数的访问模式。

- 阻塞是减少缓存未命中的有用技术。*公众号后台回复“`waside-blocking`”以获得更多信息。*

## Putting it all Together

我们为你提供了一个名为`./driver.py`的驱动程序，可以对你的模拟器和转置代码进行完整的评估。这和你的教师评估你的作业使用的是同一程序。驱动程序使用`test-csim`评估模拟器，并使用`test-trans`评估三种矩阵大小的提交的转置函数。然后打印出你的结果和所获得的积分的摘要。

要运行驱动程序，请键入：

```shell
linux> ./driver.py
```

# Handing in Your Work

每次在`cachelab-handout`目录中键入`make`时，`Makefile`都会创建一个名为`userid-handin.tar`的压缩文件，其中包含当前的`csim.c`和`trans.c`文件。

重要说明：请勿在Windows或Mac计算机上创建要上交的压缩包，也不要上交任何其他存档格式的文件，例如`.zip`、`.gzip`或`.tgz`文件。
