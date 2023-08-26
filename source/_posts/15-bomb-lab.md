---
url: bomb-lab
title: 拆弹实验
date: 2020-05-03 19:47:32
categories: [技术]
tags: [计算机系统实验]
---

Bomb Lab: Defusing a Binary Bomb

<!--more-->

*我经过漫长的分析和调试最终完成了本实验，完成后撰写本文用时约两天，**若参考或转载请注明出处**。*

> Dr. Evil's Insidious Bomb, Version 1.1
> Copyright 2011, Dr. Evil Incorporated. All rights reserved.
>
> LICENSE:
>
> Dr. Evil Incorporated (the PERPETRATOR) hereby grants you (the VICTIM) explicit permission to use this bomb (the BOMB).  This is a time limited license, which expires on the death of the VICTIM. The PERPETRATOR takes no responsibility for damage, frustration, insanity, bug-eyes, carpal-tunnel syndrome, loss of sleep, or other harm to the VICTIM.  Unless the PERPETRATOR wants to take credit, that is.  The VICTIM may not distribute this bomb source code to any enemies of the PERPETRATOR.  No VICTIM may debug, reverse-engineer, run "strings" on, decompile, decrypt, or use any other technique to gain knowledge of and defuse the BOMB.  BOMB proof clothing may not be worn when handling this program.  The PERPETRATOR will not apologize for the PERPETRATOR's poor sense of humor.  This license is null and void where the BOMB is prohibited by law.

大意为

> Dr.Evil的阴险炸弹，版本1.1
> 版权声明：2011，Dr.Evil公司版权所有。
>
> 许可声明：
>
> Dr.Evil公司（犯罪者）特此授予您（受害者）使用该炸弹（BOMB）的明确许可。这是一个有时间限制的许可证，在受害者死亡时到期。破坏者对损坏，沮丧，精神错乱，虫眼，腕管综合症，睡眠不足或对受害者造成的其他伤害概不负责，除非犯罪者想要获得荣誉。受害者不得将此炸弹源代码分发给犯罪者的任何敌人。受害者不得调试，逆向工程，在其上运行“字符串”，反编译，解密或使用任何其他技术来了解和拆除BOMB。处理此程序时，可能不能穿防弹衣。犯罪者不会因犯罪者的幽默感而道歉。在法律禁止BOMB的情况下，此许可无效。

本实验考察了对汇编语言的分析，对程序控制、过程调用的汇编级实现的理解。故做本实验之前，应先掌握CS:APP第三章《程序的机器级表示》部分。即使你未掌握汇编语言相关的知识，阅读本文后你也会有很大收获。

本文所有操作均基于以下环境：

- OS: Ubuntu 18.04.4 LTS (Linux ubuntu 5.3.0-46-generic x86_64)
- Debugger: GNU gdb (Ubuntu 8.1-0ubuntu3.2) 8.1.0.20180409-git

*有关实验的基本介绍参见实验说明。*

# 准备

阅读`bomb.c`，可以发现：

- `bomb`支持两种输入方式。

  为避免重复输入，建议将分析出的password预先存入文本文件`passwd.txt`中。

- `bomb`总共设置了6个`phase`，每个`phase`包括：

  - `read_line()`：读取一行作为输入的password；
  - `phase_X()`：第`X`个`phase`调用对应此函数以检验password是否正确；
  - `phase_defused()`：每个`phase`在检验密码正确后都会运行此函数。

  只有password全部正确，拆弹才成功，否则将爆炸：

  ```shell
  BOOM!!!
  The bomb has blown up.
  ```

拆弹的过程，就是通过分析`objdump`反汇编生成的代码，并利用`gdb`调试得到每一个`phase`的password，完成总共6个`phase`即拆弹成功。

下面是关于`objdump`和`gdb`的具体用法：

- [`objdump`命令](https://man.linuxde.net/objdump)
- [`gdb`命令](https://man.linuxde.net/gdb)

> `gdb`中使用`x`命令来打印内存的值，格式为“`x/<nfu> <addr>`”。含义为以`<f>`格式打印从`<addr>`开始的`<n>`个长度单元为`<u>`的内存值。参数具体含义如下：
>
> - `<n>`：输出单元的个数。
>
> - `<f>`：输出格式。
>
>   `s` 输出字符串；
>
>   `x` 按十六进制格式显示变量；
>
>   `d` 按十进制格式显示变量；
>
>   `u` 按十六进制格式显示无符号整型；
>
>   `o` 按八进制格式显示变量；
>
>   `t` 按二进制格式显示变量；
>
>   `a` 按十六进制格式显示变量；
>
>   `c` 按字符格式显示变量；
>
>   `f` 按浮点数格式显示变量。
>
> - `<u>`：标明一个单元的长度。
>
>   `b`是1个`byte`；
>
>   `h`是2个`byte`（halfword）；
>
>   `w`是4个`byte`（word）；
>
>   `g`是8个`byte`（giant word）。

# 拆弹开始

首先利用`objdump`工具反汇编，并将结果重定向存入`bomb.s`（或`bomb.asm`、`bomb.txt`）文件以便分析。（注意此处操作务必在Linux环境下进行，若在Windows或macOS环境下可能会无法生成注释，并可能无法将数值显示为十六进制）

```shell
objdump -d bomb > bomb.s
```

生成的汇编文件包括**标号地址**、**标号名字**、**指令地址**、**指令机器码**以及指令机器码反汇编得到的**指令**等部分。

运行

```shell
gdb bomb
```

拆弹开始。

# main

首先分析`main`函数，找到`phase_1`对应的汇编语句部分：

```assembly
  400e32:	e8 67 06 00 00       	callq  40149e <read_line>
  400e37:	48 89 c7             	mov    %rax,%rdi
  400e3a:	e8 a1 00 00 00       	callq  400ee0 <phase_1>
  400e3f:	e8 80 07 00 00       	callq  4015c4 <phase_defused>
  400e44:	bf a8 23 40 00       	mov    $0x4023a8,%edi
  400e49:	e8 c2 fc ff ff       	callq  400b10 <puts@plt>
```

结合`bomb.c`可知，程序首先调用`read_line`函数读取一行字符串（400e32），并将其返回值`input`从`%rax`寄存器传入`%rdi`寄存器（400e37）。（由于`read_line`函数的作用易知，故本文不再对其进行详细分析）

分析剩余`main`函数可知，其调用每个`phase`函数时，`input`字符串地址都存储于`%rdi`寄存器中，即**`%rdi`指向`input`字符串**。

后续直接分析各`phase`函数即可。

# Phase 1: *string*

`phase_1`对应的汇编语句如下：

```assembly
0000000000400ee0 <phase_1>:
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq
```

---

分段分析：

---

```assembly
  400ee0:	48 83 ec 08          	sub    $0x8,%rsp
```

400ee0在`%rsp`中分配8字节空间压栈

---

```assembly
  400ee4:	be 00 24 40 00       	mov    $0x402400,%esi
```

400ee4将地址`0x402400`传入`%esi`寄存器，直接`gdb`打断点查看此处内容：

``` shell
(gdb) b explode_bomb
(gdb) x/s 0x402400
```

得到输出如下：

```shell
Breakpoint 1 at 0x40143a
0x402400:       "Border relations with Canada have never been better."
```

可以看到地址`0x402400`指向的是字符串`Border relations with Canada have never been better.`，推测`phase_1`的密码就是此字符串（后续称之为“密码串”，对应`input`字符串为“输入串”）。

---

```assembly
  400ee9:	e8 4a 04 00 00       	callq  401338 <strings_not_equal>
```

400ee9<span id = "p1">调用`strings_not_equal`函数</span>（**推测**可能是判断两字符串是否相等，[点此分析`strings_not_equal`函数](#equal)）

---

```assembly
  400eee:	85 c0                	test   %eax,%eax
  400ef0:	74 05                	je     400ef7 <phase_1+0x17>
  400ef2:	e8 43 05 00 00       	callq  40143a <explode_bomb>
  400ef7:	48 83 c4 08          	add    $0x8,%rsp
  400efb:	c3                   	retq
```

400eee和400ef0判断`%eax`（`strings_not_equal`函数的返回值）是否为0：

- 若为0（输入串和密码串相等）则跳到400ef7（400ee0+0x17），弹出栈并**返回**；
- 若不为0（输入串和密码串不等）则运行400ef2执行`explode_bomb`函数使炸弹爆炸

---

至此，得知密码串就是password。即`phase_1`的password为

```shell
Border relations with Canada have never been better.
```

# Phase2: *loop*

`phase_2`对应的汇编语句如下：

```assembly
0000000000400efc <phase_2>:
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp
  400f02:	48 89 e6             	mov    %rsp,%rsi
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  400f1a:	01 c0                	add    %eax,%eax
  400f1c:	39 03                	cmp    %eax,(%rbx)
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq
```

---

分段分析：

---

```assembly
  400efc:	55                   	push   %rbp
  400efd:	53                   	push   %rbx
  400efe:	48 83 ec 28          	sub    $0x28,%rsp
  400f02:	48 89 e6             	mov    %rsp,%rsi
```

400efe在`%rsp`中分配40字节空间压栈，400f02将`%rsp`地址给`%rsi`

---

``` assembly
  400f05:	e8 52 05 00 00       	callq  40145c <read_six_numbers>
```

400f05<span id = "p2">调用`read_six_numbers`函数</span>（**推测**可能是读取6个数字，[点此分析`read_six_numbers`函数](#6numbers)）。

分析完`read_six_numbers`函数后，可以明确：输入的6个整数依次存放于`0(%rsp)`、`4(%rsp)`、`8(%rsp)`、`12(%rsp)`、`16(%rsp)`、`20(%rsp)`中

---

```assembly
  400f0a:	83 3c 24 01          	cmpl   $0x1,(%rsp)
  400f0e:	74 20                	je     400f30 <phase_2+0x34>
  400f10:	e8 25 05 00 00       	callq  40143a <explode_bomb>
  400f15:	eb 19                	jmp    400f30 <phase_2+0x34>
  400f17:	8b 43 fc             	mov    -0x4(%rbx),%eax
  400f1a:	01 c0                	add    %eax,%eax
  400f1c:	39 03                	cmp    %eax,(%rbx)
  400f1e:	74 05                	je     400f25 <phase_2+0x29>
  400f20:	e8 15 05 00 00       	callq  40143a <explode_bomb>
  400f25:	48 83 c3 04          	add    $0x4,%rbx
  400f29:	48 39 eb             	cmp    %rbp,%rbx
  400f2c:	75 e9                	jne    400f17 <phase_2+0x1b>
  400f2e:	eb 0c                	jmp    400f3c <phase_2+0x40>
  400f30:	48 8d 5c 24 04       	lea    0x4(%rsp),%rbx
  400f35:	48 8d 6c 24 18       	lea    0x18(%rsp),%rbp
  400f3a:	eb db                	jmp    400f17 <phase_2+0x1b>
  400f3c:	48 83 c4 28          	add    $0x28,%rsp
  400f40:	5b                   	pop    %rbx
  400f41:	5d                   	pop    %rbp
  400f42:	c3                   	retq
```

400f0a和400f0e判断`%rsp`指向的值（输入的第1个整数）是否为1：

- 若不为1则继续执行400f10使炸弹爆炸；
- 若为1则跳到400f30（400efc+0x34）把`%rsp+4`的地址（指向输入的下一个整数）给`%rbx`，400f35把`%rsp+24`的地址（指向6个整数结束的位置）给`%rbp`，再跳回400f17（400efc+0x1b）把`%rbx-4`的地址（指向当前整数的上一个整数）给`%eax`，400f1a将`%eax`的值（上一个整数）乘2，400f1c和400f1e将`%eax`（上一个整数的2倍）和`%rbx`指向的值（当前整数）对比：
  - 若不等则继续执行400f20使炸弹爆炸；
  - 若相等则跳到400f25（400efc+0x29）将`%rbx`加4（使`%rbx`指向下一个整数），400f29和400f2c判断`%rbx`（当前整数位置）是否等于`%rbp`（6整数结束的位置）：
    - 若相等则说明6整数判断完毕，继续执行400f2e跳到400f3c弹出栈并返回；
    - 若不等则跳到400f17循环。

可以看出此部分的循环等价于：

```c
if (passwd[0] != 1)// %rsp
	explode_bomb();
p_tail = passwd + 6;// %rsp + 24->%rbp
for (p_loc = passwd + 1; p_loc != p_tail; p_loc++) {// %rsp + 4->%rbx; %rbx != %rbp
	temp = *(p_loc - 1) * 2;// %eax
	if (*p_loc != temp)
		explode_bomb();
}
return temp;
```

即判断是否满足“**第一个整数为`1`，后面5个整数依次为前一个的2倍**”；不满足则使炸弹爆炸

---

至此，得到`phase_2`的password为

```shell
1 2 4 8 16 32
```

# Phase 3: *switch*

`phase_3`对应的汇编语句如下：

```assembly
0000000000400f43 <phase_3>:
  400f43:	48 83 ec 18          	sub    $0x18,%rsp
  400f47:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  400f4c:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  400f60:	83 f8 01             	cmp    $0x1,%eax
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>
  400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)
  400f6f:	77 3c                	ja     400fad <phase_3+0x6a>
  400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
  400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
  400f7c:	b8 cf 00 00 00       	mov    $0xcf,%eax
  400f81:	eb 3b                	jmp    400fbe <phase_3+0x7b>
  400f83:	b8 c3 02 00 00       	mov    $0x2c3,%eax
  400f88:	eb 34                	jmp    400fbe <phase_3+0x7b>
  400f8a:	b8 00 01 00 00       	mov    $0x100,%eax
  400f8f:	eb 2d                	jmp    400fbe <phase_3+0x7b>
  400f91:	b8 85 01 00 00       	mov    $0x185,%eax
  400f96:	eb 26                	jmp    400fbe <phase_3+0x7b>
  400f98:	b8 ce 00 00 00       	mov    $0xce,%eax
  400f9d:	eb 1f                	jmp    400fbe <phase_3+0x7b>
  400f9f:	b8 aa 02 00 00       	mov    $0x2aa,%eax
  400fa4:	eb 18                	jmp    400fbe <phase_3+0x7b>
  400fa6:	b8 47 01 00 00       	mov    $0x147,%eax
  400fab:	eb 11                	jmp    400fbe <phase_3+0x7b>
  400fad:	e8 88 04 00 00       	callq  40143a <explode_bomb>
  400fb2:	b8 00 00 00 00       	mov    $0x0,%eax
  400fb7:	eb 05                	jmp    400fbe <phase_3+0x7b>
  400fb9:	b8 37 01 00 00       	mov    $0x137,%eax
  400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
  400fc2:	74 05                	je     400fc9 <phase_3+0x86>
  400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
  400fc9:	48 83 c4 18          	add    $0x18,%rsp
  400fcd:	c3                   	retq
```

---

分段分析：

---

```assembly
  400f43:	48 83 ec 18          	sub    $0x18,%rsp
  400f47:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  400f4c:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
```

400f43在`%rsp`中分配24字节空间压栈，400f47把`%rsp+12`的地址给`%rcx`，400f4c把`%rsp+8`的地址给`%rdx`

```assembly
%rdx = %rsp + 8
%rcx = %rsp + 12
```

---

```assembly
  400f51:	be cf 25 40 00       	mov    $0x4025cf,%esi
  400f56:	b8 00 00 00 00       	mov    $0x0,%eax
  400f5b:	e8 90 fc ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
```

400f51将地址`0x4025cf`传入`%esi`寄存器，直接`gdb`查看此处内容：

```shell
(gdb) x/s 0x4025cf
```

得到输出如下：

```shell
0x4025cf:       "%d %d"
```

根据`phase_2`的经验，结合后续400f5b再次调用了`sscanf`函数，可以**推测**`phase_3`的password可能是两个整数，中间以1个空格隔开。而且输入的两个整数存储于上述的两个地址中。

---

```assembly
  400f60:	83 f8 01             	cmp    $0x1,%eax
  400f63:	7f 05                	jg     400f6a <phase_3+0x27>
  400f65:	e8 d0 04 00 00       	callq  40143a <explode_bomb>
  400f6a:	83 7c 24 08 07       	cmpl   $0x7,0x8(%rsp)
  400f6f:	77 3c                	ja     400fad <phase_3+0x6a>
  ...
  400fad:	e8 88 04 00 00       	callq  40143a <explode_bomb>
```

400f60和400f63判断`%eax`是否为1：

- 若不为1则继续执行400f65使炸弹爆炸；
- 若为1则跳到400f6a（400f43+0x27），和400f6f判断`%rsp+8`指向的值（输入的第1个整数）是否超过7:
  - 若超过7则跳到400fad（400f43+0x6a）使炸弹爆炸（由`ja`针对无符号数可以得知**第一个整数在0～7之间**）；
  - 若不超过7则继续执行

---

```assembly
  400f71:	8b 44 24 08          	mov    0x8(%rsp),%eax
```

400f71将`%rsp+8`指向的值（输入的第1个整数）赋给`%eax`

---

```assembly
  400f75:	ff 24 c5 70 24 40 00 	jmpq   *0x402470(,%rax,8)
```

400f75根据`%rax`的值（输入的第一个整数）跳转到相应地址存放的地址，由于第一个整数在0～7之间，`gdb`以单字节十六进制查看8*8=64个对应地址：

```shell
(gdb) x/64xb 0x402470
```

得到输出如下：

```shell
0x402470:       0x7c    0x0f    0x40    0x00    0x00    0x00    0x00    0x00
0x402478:       0xb9    0x0f    0x40    0x00    0x00    0x00    0x00    0x00
0x402480:       0x83    0x0f    0x40    0x00    0x00    0x00    0x00    0x00
0x402488:       0x8a    0x0f    0x40    0x00    0x00    0x00    0x00    0x00
0x402490:       0x91    0x0f    0x40    0x00    0x00    0x00    0x00    0x00
0x402498:       0x98    0x0f    0x40    0x00    0x00    0x00    0x00    0x00
0x4024a0:       0x9f    0x0f    0x40    0x00    0x00    0x00    0x00    0x00
0x4024a8:       0xa6    0x0f    0x40    0x00    0x00    0x00    0x00    0x00
```

由于机器为小端法（little endian），整理后得到输入的第一个整数为0～7时依次对应跳转到的地址依次为：`0x400f7c`、`0x400fb9`、`0x400f83`、`0x400f8a`、`0x400f91`、`0x400f98`、`0x400f9f`和`0x400fa6`

---

```assembly
  400f7c:	b8 cf 00 00 00       	mov    $0xcf,%eax
  400f81:	eb 3b                	jmp    400fbe <phase_3+0x7b>
  400f83:	b8 c3 02 00 00       	mov    $0x2c3,%eax
  400f88:	eb 34                	jmp    400fbe <phase_3+0x7b>
  400f8a:	b8 00 01 00 00       	mov    $0x100,%eax
  400f8f:	eb 2d                	jmp    400fbe <phase_3+0x7b>
  400f91:	b8 85 01 00 00       	mov    $0x185,%eax
  400f96:	eb 26                	jmp    400fbe <phase_3+0x7b>
  400f98:	b8 ce 00 00 00       	mov    $0xce,%eax
  400f9d:	eb 1f                	jmp    400fbe <phase_3+0x7b>
  400f9f:	b8 aa 02 00 00       	mov    $0x2aa,%eax
  400fa4:	eb 18                	jmp    400fbe <phase_3+0x7b>
  400fa6:	b8 47 01 00 00       	mov    $0x147,%eax
  400fab:	eb 11                	jmp    400fbe <phase_3+0x7b>
	...
  400fb2:	b8 00 00 00 00       	mov    $0x0,%eax
  400fb7:	eb 05                	jmp    400fbe <phase_3+0x7b>
  400fb9:	b8 37 01 00 00       	mov    $0x137,%eax
  400fbe:	3b 44 24 0c          	cmp    0xc(%rsp),%eax
  400fc2:	74 05                	je     400fc9 <phase_3+0x86>
  400fc4:	e8 71 04 00 00       	callq  40143a <explode_bomb>
  400fc9:	48 83 c4 18          	add    $0x18,%rsp
  400fcd:	c3                   	retq
```

类似`switch`语句，结合上述得到的8个地址，此部分等价于：

```c
switch (passwd[0]) {
  case 0: result = 207; break;// 0x400f7c
  case 1: result = 311; break;// 0x400fb9
  case 2: result = 707; break;// 0x400f83
  case 3: result = 256; break;// 0x400f8a
  case 4: result = 389; break;// 0x400f91
  case 5: result = 206; break;// 0x400f98
  case 6: result = 682; break;// 0x400f9f
  case 7: result = 327; break;// 0x400fa6
}
```

每个`case`赋值后，都跳到400fbe（400f43+0x7b）和400fc2将`%rsp+12`指向的值（输入的第2个整数）和`%eax`的值（`result`值）对比：

- 若二者相等则跳到400fc9弹出栈并**返回**；
- 若二者不等则继续执行400fc4使炸弹爆炸

综上得知输入的第1个整数可以有8种情况，而输入的第2个整数因第1个整数而异

---

至此，得到`phase_3`的password为

```shell
0 207
```

或

```shell
1 311
```

或

```shell
2 707
```

或

```shell
3 256
```

或

```shell
4 389
```

或

```shell
5 206
```

或

```shell
6 682
```

或

```shell
7 327
```

# Phase 4: *recursion*

`phase_4`对应的汇编语句如下：

```assembly
000000000040100c <phase_4>:
  40100c:	48 83 ec 18          	sub    $0x18,%rsp
  401010:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  401015:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  40101a:	be cf 25 40 00       	mov    $0x4025cf,%esi
  40101f:	b8 00 00 00 00       	mov    $0x0,%eax
  401024:	e8 c7 fb ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  401029:	83 f8 02             	cmp    $0x2,%eax
  40102c:	75 07                	jne    401035 <phase_4+0x29>
  40102e:	83 7c 24 08 0e       	cmpl   $0xe,0x8(%rsp)
  401033:	76 05                	jbe    40103a <phase_4+0x2e>
  401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>
  40103a:	ba 0e 00 00 00       	mov    $0xe,%edx
  40103f:	be 00 00 00 00       	mov    $0x0,%esi
  401044:	8b 7c 24 08          	mov    0x8(%rsp),%edi
  401048:	e8 81 ff ff ff       	callq  400fce <func4>
  40104d:	85 c0                	test   %eax,%eax
  40104f:	75 07                	jne    401058 <phase_4+0x4c>
  401051:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  401056:	74 05                	je     40105d <phase_4+0x51>
  401058:	e8 dd 03 00 00       	callq  40143a <explode_bomb>
  40105d:	48 83 c4 18          	add    $0x18,%rsp
  401061:	c3                   	retq
```

---

分段分析：

---

40100c～401024与`phase_3`的400f43～400f5b相同，即`phase_4`的password同样可能是两个整数，中间以1个空格隔开，而且输入的两个整数分别存储于`%rsp+8`和`%rsp+12`两个地址中

---

```assembly
  401029:	83 f8 02             	cmp    $0x2,%eax
  40102c:	75 07                	jne    401035 <phase_4+0x29>
  ...
  401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>
```

401029和40102c判断`%eax`（`sscanf`函数的返回值）是否为2，即判断输入的是否为2个整数：

- 不为2则跳到401035（40100c+0x29）使炸弹爆炸；
- 为2则继续执行

---

```assembly
  40102e:	83 7c 24 08 0e       	cmpl   $0xe,0x8(%rsp)
  401033:	76 05                	jbe    40103a <phase_4+0x2e>
  401035:	e8 00 04 00 00       	callq  40143a <explode_bomb>
  40103a:	ba 0e 00 00 00       	mov    $0xe,%edx
```

40102e和401033判断`%rsp+8`指向的值（输入的第1个整数）是否不超过14:

- 若超过14则继续执行401035使炸弹爆炸（由此得知**输入的第1个整数范围是0～14**）；
- 若不超过14则跳到40103a（40100c+0x2e）把`%edx`赋为14

---

```assembly
  40103f:	be 00 00 00 00       	mov    $0x0,%esi
  401044:	8b 7c 24 08          	mov    0x8(%rsp),%edi
  401048:	e8 81 ff ff ff       	callq  400fce <func4>
```

40103f把`%esi`赋为0，401044将`%rsp+8`指向的值（输入的第1个整数）给`%edi`，401048<span id = "p4">调用了`func4`函数</span>（无法推测func4函数的功能，[点此分析`func4`函数](#func4)）

---

```assembly
  40104d:	85 c0                	test   %eax,%eax
  40104f:	75 07                	jne    401058 <phase_4+0x4c>
  401051:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
  401056:	74 05                	je     40105d <phase_4+0x51>
  401058:	e8 dd 03 00 00       	callq  40143a <explode_bomb>
  40105d:	48 83 c4 18          	add    $0x18,%rsp
  401061:	c3                   	retq
```

40104d和40104f判断`%eax`的值（`func4`函数的返回值）是否为0:

- 若不为0则跳到401058（40100c+0x4c）使炸弹爆炸（由此得知**`func4`函数的返回值必须为0**）；
- 若为0则继续执行401051和401056判断`%rsp+12`指向的值（输入的第2个整数）是否为0:
  - 若为0则跳到40105d（40100c+0x51）弹出栈并**返回**；
  - 若不为0则继续执行401058使炸弹爆炸（由此得知**输入的第2个整数必须为0**）

---

分析完毕，已经得知`phase_4`的password由两个整数构成，而第2个整数必为0，第一个整数是使`func4`函数返回值为0且在0～14之间的数。

编写C程序求第一个整数：

```c
#include <stdio.h>

int func4(int, int, int);
int main()
{
    int min = 0;
    int max = 14;
    for (int i = 0; i < 15; i++) {
        if (!func4(i, min, max))
            printf("%d\n", i);
    }
    return 0;
}
```

得到输出如下：

```shell
0
1
3
7

```

即第1个整数可以是`0`、`1`、`3`或`7`。

至此，得到`phase_4`的password为

```shell
0 0
```

或

```shell
1 0
```

或

```shell
3 0
```

或

```shell
7 0
```

# Phase 5: *pointer*

`phase_5`对应的汇编语句如下：

```assembly
0000000000401062 <phase_5>:
  401062:	53                   	push   %rbx
  401063:	48 83 ec 20          	sub    $0x20,%rsp
  401067:	48 89 fb             	mov    %rdi,%rbx
  40106a:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  401071:	00 00
  401073:	48 89 44 24 18       	mov    %rax,0x18(%rsp)
  401078:	31 c0                	xor    %eax,%eax
  40107a:	e8 9c 02 00 00       	callq  40131b <string_length>
  40107f:	83 f8 06             	cmp    $0x6,%eax
  401082:	74 4e                	je     4010d2 <phase_5+0x70>
  401084:	e8 b1 03 00 00       	callq  40143a <explode_bomb>
  401089:	eb 47                	jmp    4010d2 <phase_5+0x70>
  40108b:	0f b6 0c 03          	movzbl (%rbx,%rax,1),%ecx
  40108f:	88 0c 24             	mov    %cl,(%rsp)
  401092:	48 8b 14 24          	mov    (%rsp),%rdx
  401096:	83 e2 0f             	and    $0xf,%edx
  401099:	0f b6 92 b0 24 40 00 	movzbl 0x4024b0(%rdx),%edx
  4010a0:	88 54 04 10          	mov    %dl,0x10(%rsp,%rax,1)
  4010a4:	48 83 c0 01          	add    $0x1,%rax
  4010a8:	48 83 f8 06          	cmp    $0x6,%rax
  4010ac:	75 dd                	jne    40108b <phase_5+0x29>
  4010ae:	c6 44 24 16 00       	movb   $0x0,0x16(%rsp)
  4010b3:	be 5e 24 40 00       	mov    $0x40245e,%esi
  4010b8:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  4010bd:	e8 76 02 00 00       	callq  401338 <strings_not_equal>
  4010c2:	85 c0                	test   %eax,%eax
  4010c4:	74 13                	je     4010d9 <phase_5+0x77>
  4010c6:	e8 6f 03 00 00       	callq  40143a <explode_bomb>
  4010cb:	0f 1f 44 00 00       	nopl   0x0(%rax,%rax,1)
  4010d0:	eb 07                	jmp    4010d9 <phase_5+0x77>
  4010d2:	b8 00 00 00 00       	mov    $0x0,%eax
  4010d7:	eb b2                	jmp    40108b <phase_5+0x29>
  4010d9:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
  4010de:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  4010e5:	00 00
  4010e7:	74 05                	je     4010ee <phase_5+0x8c>
  4010e9:	e8 42 fa ff ff       	callq  400b30 <__stack_chk_fail@plt>
  4010ee:	48 83 c4 20          	add    $0x20,%rsp
  4010f2:	5b                   	pop    %rbx
  4010f3:	c3                   	retq
```

---

分段分析：

---

```assembly
  401062:	53                   	push   %rbx
  401063:	48 83 ec 20          	sub    $0x20,%rsp
  ...
  40106a:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  401071:	00 00
  401073:	48 89 44 24 18       	mov    %rax,0x18(%rsp)
  ...
  4010d9:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
  4010de:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  4010e5:	00 00
  4010e7:	74 05                	je     4010ee <phase_5+0x8c>
  4010e9:	e8 42 fa ff ff       	callq  400b30 <__stack_chk_fail@plt>
  4010ee:	48 83 c4 20          	add    $0x20,%rsp
  4010f2:	5b                   	pop    %rbx
  4010f3:	c3                   	retq
```

<span id ="stackCanacy">`%fs:40`是FS段寄存器上偏移地址`0x28`上的数据，这是一个[随机量](https://stackoverflow.com/questions/10325713/why-does-this-memory-address-fs0x28-fs0x28-have-a-random-value)，起到[stack canary](https://unix.stackexchange.com/questions/453749/what-sets-fs0x28-stack-canary)的作用。此部分利用stack canary确保`24(%rsp)`的数值（栈底8字节，401063）在函数调用前后不改变，若改变则执行4010e9调用`__stack_chk_fail`函数跳出，以防栈溢出（stack overflow）。</span>

此部分与拆弹联系较弱，可以跳过。

---

```assembly
  401067:	48 89 fb             	mov    %rdi,%rbx
  ...
  401078:	31 c0                	xor    %eax,%eax
  40107a:	e8 9c 02 00 00       	callq  40131b <string_length>
```

401067将`%rdi`的地址（指向输入串）赋给`%rbx`，401078将`%eax`设为0，40107a调用`string_length`函数，返回输入串的长度`%eax`

---

```assembly
  40107f:	83 f8 06             	cmp    $0x6,%eax
  401082:	74 4e                	je     4010d2 <phase_5+0x70>
  401084:	e8 b1 03 00 00       	callq  40143a <explode_bomb>
  ...
  4010d2:	b8 00 00 00 00       	mov    $0x0,%eax
  4010d7:	eb b2                	jmp    40108b <phase_5+0x29>
```

40107f和401082判断`%eax`的值（输入串的长度）是否为6：

- 若不为6则继续执行401084使炸弹爆炸（由此得知**输入串长度必须为6**）；
- 若为6则跳到4010d2（401062+0x70）将`%eax`赋为0，4010d7跳到40108b（401062+0x29）

---

```assembly
  401089:	eb 47                	jmp    4010d2 <phase_5+0x70>
  40108b:	0f b6 0c 03          	movzbl (%rbx,%rax,1),%ecx
  40108f:	88 0c 24             	mov    %cl,(%rsp)
  401092:	48 8b 14 24          	mov    (%rsp),%rdx
  401096:	83 e2 0f             	and    $0xf,%edx
  401099:	0f b6 92 b0 24 40 00 	movzbl 0x4024b0(%rdx),%edx
  4010a0:	88 54 04 10          	mov    %dl,0x10(%rsp,%rax,1)
  4010a4:	48 83 c0 01          	add    $0x1,%rax
  4010a8:	48 83 f8 06          	cmp    $0x6,%rax
  4010ac:	75 dd                	jne    40108b <phase_5+0x29>
```

此时`%eax`为0，40108b将`%rbx`指向的值加上`%rax`的值（输入串的第`%rax+1`个字符）赋给`%ecx`，40108f将`%cl`（输入串的第`%rax+1`个字符低8位）赋给`%rsp`指向的值，401092再把`%rsp`指向的值（输入串的第`%rax+1`个字符低8位）赋给`%rdx`，401096将`%edx`（输入串的第`%rax+1`个字符低8位）和`0xf`按位取与（取低8位的低4位，结果存放于`%edx`中），401099将`0x4024b0`作为基地址、`%rdx`指向的值（取与后的值）作为偏移量，并将偏移后的值赋给`%edx`，4010a0将`%dl`（`%edx`低8位）赋给`%rsp+%rax+16`指向的值，4010a4`%rax`作为索引加1，4010a8判断`%rax`是否为6:

- 不为6则跳到40108b（401062+0x29）循环；
- 为6则继续执行4010ae

可以看出此处的循环等价于：

```c
int i = 0;// %eax
for (i = 0; i != 6; i++) {// %rax
	ch = passwd[i];// %rbx->%ecx
  index = ch;// %cl->%rsp->rdx
  index &= 0xf;// %edx
  ch = func[index];// 0x4024b0 + index->%edx
  str[i] = ch;// %dl->%rsp + 16 + i
}
```

由输入串的6个字符，每个字符的低4位作为偏移量，`0x4024b0`作为基地址，最终映射生成新的`str`串

---

```assembly
  4010ae:	c6 44 24 16 00       	movb   $0x0,0x16(%rsp)
  4010b3:	be 5e 24 40 00       	mov    $0x40245e,%esi
  4010b8:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  4010bd:	e8 76 02 00 00       	callq  401338 <strings_not_equal>
  4010c2:	85 c0                	test   %eax,%eax
  4010c4:	74 13                	je     4010d9 <phase_5+0x77>
  4010c6:	e8 6f 03 00 00       	callq  40143a <explode_bomb>
  4010cb:	0f 1f 44 00 00       	nopl   0x0(%rax,%rax,1)
  4010d0:	eb 07                	jmp    4010d9 <phase_5+0x77>
  4010d2:	b8 00 00 00 00       	mov    $0x0,%eax
  4010d7:	eb b2                	jmp    40108b <phase_5+0x29>
  4010d9:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
```

4010ae将`%rsp+22`指向的值（`str[6]`）赋为`'\0'`，4010b3将地址`0x40245e`赋给`%esi`，4010b8将`%rsp+16`的地址（`str`）赋给`%rdi`，4010bd调用`strings_not_equal`函数（相等返回0，不等返回1），4010c2和4010c4判断返回值`%eax`是否为0：

- 若不为0则继续执行4010c6使炸弹爆炸（由此得知**返回值须为0**，即**生成的`str`串须等于`0x40245e`处的字符串**）；
- 若为0则跳到4010d9（401062+0x77）继续执行则弹出栈并**返回**

直接`gdb`查看`0x40245e`处的字符串：

```shell
(gdb) x/s 0x40245e
```

得到输出如下：

```shell
0x40245e:       "flyers"
```

即我们新生成的`str`串须为“`flyers`”

---

`gdb`查看基地址`0x4024b0`处的字符串：

```shell
(gdb) x/s 0x4024b0
```

得到输出如下：

```shell
0x4024b0 <array.3449>:  "maduiersnfotvbylSo you think you can stop the bomb with ctrl-c, do you?"
```

即基地址指向的字符串为“`maduiersnfotvbyl`”，从中找到“`flyers`”对应的下标为`9`、`15`、`14`、`5`、`6`和`7`，转换为十六进制为`0x9`、`0xf`、`0xe`、`0x5`、`0x6`和`0x7`——这就是输入串6个字符的低4位的值。

借助ASCII表，可以发现`phase_5`的密码并不唯一。

![ASCII](https://i0.hdslb.com/bfs/album/2cf57953a35dbc2f450f5a3ca60c720cbbd7dae3.png)

- 低4位为`0x9`的字符有：`)`、`9`、`I`、`Y`、`i`、`y`
- 低4位为`0xf`的字符有：`/`、`?`、`O`、`_`、`o`
- 低4位为`0xe`的字符有：`.`、`>`、`N`、`^`、`n`、`~`
- 低4位为`0x5`的字符有：`%`、`5`、`E`、`U`、`e`、`u`
- 低4位为`0x6`的字符有：`&`、`6`、`F`、`V`、`f`、`v`
- 低4位为`0x7`的字符有：`'`、`7`、`G`、`W`、`g`、`w`

至此，得到`phase_5`的password为（不唯一）

```shell
9/N567
```

# Phase 6: *link list*

`phase_6`对应的汇编语句如下：

```assembly
00000000004010f4 <phase_6>:
  4010f4:	41 56                	push   %r14
  4010f6:	41 55                	push   %r13
  4010f8:	41 54                	push   %r12
  4010fa:	55                   	push   %rbp
  4010fb:	53                   	push   %rbx
  4010fc:	48 83 ec 50          	sub    $0x50,%rsp
  401100:	49 89 e5             	mov    %rsp,%r13
  401103:	48 89 e6             	mov    %rsp,%rsi
  401106:	e8 51 03 00 00       	callq  40145c <read_six_numbers>
  40110b:	49 89 e6             	mov    %rsp,%r14
  40110e:	41 bc 00 00 00 00    	mov    $0x0,%r12d
  401114:	4c 89 ed             	mov    %r13,%rbp
  401117:	41 8b 45 00          	mov    0x0(%r13),%eax
  40111b:	83 e8 01             	sub    $0x1,%eax
  40111e:	83 f8 05             	cmp    $0x5,%eax
  401121:	76 05                	jbe    401128 <phase_6+0x34>
  401123:	e8 12 03 00 00       	callq  40143a <explode_bomb>
  401128:	41 83 c4 01          	add    $0x1,%r12d
  40112c:	41 83 fc 06          	cmp    $0x6,%r12d
  401130:	74 21                	je     401153 <phase_6+0x5f>
  401132:	44 89 e3             	mov    %r12d,%ebx
  401135:	48 63 c3             	movslq %ebx,%rax
  401138:	8b 04 84             	mov    (%rsp,%rax,4),%eax
  40113b:	39 45 00             	cmp    %eax,0x0(%rbp)
  40113e:	75 05                	jne    401145 <phase_6+0x51>
  401140:	e8 f5 02 00 00       	callq  40143a <explode_bomb>
  401145:	83 c3 01             	add    $0x1,%ebx
  401148:	83 fb 05             	cmp    $0x5,%ebx
  40114b:	7e e8                	jle    401135 <phase_6+0x41>
  40114d:	49 83 c5 04          	add    $0x4,%r13
  401151:	eb c1                	jmp    401114 <phase_6+0x20>
  401153:	48 8d 74 24 18       	lea    0x18(%rsp),%rsi
  401158:	4c 89 f0             	mov    %r14,%rax
  40115b:	b9 07 00 00 00       	mov    $0x7,%ecx
  401160:	89 ca                	mov    %ecx,%edx
  401162:	2b 10                	sub    (%rax),%edx
  401164:	89 10                	mov    %edx,(%rax)
  401166:	48 83 c0 04          	add    $0x4,%rax
  40116a:	48 39 f0             	cmp    %rsi,%rax
  40116d:	75 f1                	jne    401160 <phase_6+0x6c>
  40116f:	be 00 00 00 00       	mov    $0x0,%esi
  401174:	eb 21                	jmp    401197 <phase_6+0xa3>
  401176:	48 8b 52 08          	mov    0x8(%rdx),%rdx
  40117a:	83 c0 01             	add    $0x1,%eax
  40117d:	39 c8                	cmp    %ecx,%eax
  40117f:	75 f5                	jne    401176 <phase_6+0x82>
  401181:	eb 05                	jmp    401188 <phase_6+0x94>
  401183:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  401188:	48 89 54 74 20       	mov    %rdx,0x20(%rsp,%rsi,2)
  40118d:	48 83 c6 04          	add    $0x4,%rsi
  401191:	48 83 fe 18          	cmp    $0x18,%rsi
  401195:	74 14                	je     4011ab <phase_6+0xb7>
  401197:	8b 0c 34             	mov    (%rsp,%rsi,1),%ecx
  40119a:	83 f9 01             	cmp    $0x1,%ecx
  40119d:	7e e4                	jle    401183 <phase_6+0x8f>
  40119f:	b8 01 00 00 00       	mov    $0x1,%eax
  4011a4:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  4011a9:	eb cb                	jmp    401176 <phase_6+0x82>
  4011ab:	48 8b 5c 24 20       	mov    0x20(%rsp),%rbx
  4011b0:	48 8d 44 24 28       	lea    0x28(%rsp),%rax
  4011b5:	48 8d 74 24 50       	lea    0x50(%rsp),%rsi
  4011ba:	48 89 d9             	mov    %rbx,%rcx
  4011bd:	48 8b 10             	mov    (%rax),%rdx
  4011c0:	48 89 51 08          	mov    %rdx,0x8(%rcx)
  4011c4:	48 83 c0 08          	add    $0x8,%rax
  4011c8:	48 39 f0             	cmp    %rsi,%rax
  4011cb:	74 05                	je     4011d2 <phase_6+0xde>
  4011cd:	48 89 d1             	mov    %rdx,%rcx
  4011d0:	eb eb                	jmp    4011bd <phase_6+0xc9>
  4011d2:	48 c7 42 08 00 00 00 	movq   $0x0,0x8(%rdx)
  4011d9:	00
  4011da:	bd 05 00 00 00       	mov    $0x5,%ebp
  4011df:	48 8b 43 08          	mov    0x8(%rbx),%rax
  4011e3:	8b 00                	mov    (%rax),%eax
  4011e5:	39 03                	cmp    %eax,(%rbx)
  4011e7:	7d 05                	jge    4011ee <phase_6+0xfa>
  4011e9:	e8 4c 02 00 00       	callq  40143a <explode_bomb>
  4011ee:	48 8b 5b 08          	mov    0x8(%rbx),%rbx
  4011f2:	83 ed 01             	sub    $0x1,%ebp
  4011f5:	75 e8                	jne    4011df <phase_6+0xeb>
  4011f7:	48 83 c4 50          	add    $0x50,%rsp
  4011fb:	5b                   	pop    %rbx
  4011fc:	5d                   	pop    %rbp
  4011fd:	41 5c                	pop    %r12
  4011ff:	41 5d                	pop    %r13
  401201:	41 5e                	pop    %r14
  401203:	c3                   	retq
```

---

分段分析：

---

```assembly
  4010f4:	41 56                	push   %r14
  4010f6:	41 55                	push   %r13
  4010f8:	41 54                	push   %r12
  4010fa:	55                   	push   %rbp
  4010fb:	53                   	push   %rbx
  4010fc:	48 83 ec 50          	sub    $0x50,%rsp
  401100:	49 89 e5             	mov    %rsp,%r13
  401103:	48 89 e6             	mov    %rsp,%rsi
  401106:	e8 51 03 00 00       	callq  40145c <read_six_numbers>
  40110b:	49 89 e6             	mov    %rsp,%r14
```

401100将`%rsp`赋给`%r13`，401103将`%rsp`赋给`%rsi`，40110b将`%rsp`赋给`%r14`，401106调用`read_six_numbers`函数读取6个数字，输入的6个整数依次存放于`0(%rsp)`、`4(%rsp)`、`8(%rsp)`、`12(%rsp)`、`16(%rsp)`、`20(%rsp)`中

---

```assembly
  40110e:	41 bc 00 00 00 00    	mov    $0x0,%r12d
  401114:	4c 89 ed             	mov    %r13,%rbp
  401117:	41 8b 45 00          	mov    0x0(%r13),%eax
  40111b:	83 e8 01             	sub    $0x1,%eax
  40111e:	83 f8 05             	cmp    $0x5,%eax
  401121:	76 05                	jbe    401128 <phase_6+0x34>
  401123:	e8 12 03 00 00       	callq  40143a <explode_bomb>
  401128:	41 83 c4 01          	add    $0x1,%r12d
  40112c:	41 83 fc 06          	cmp    $0x6,%r12d
  401130:	74 21                	je     401153 <phase_6+0x5f>
  401132:	44 89 e3             	mov    %r12d,%ebx
  401135:	48 63 c3             	movslq %ebx,%rax
  401138:	8b 04 84             	mov    (%rsp,%rax,4),%eax
  40113b:	39 45 00             	cmp    %eax,0x0(%rbp)
  40113e:	75 05                	jne    401145 <phase_6+0x51>
  401140:	e8 f5 02 00 00       	callq  40143a <explode_bomb>
  401145:	83 c3 01             	add    $0x1,%ebx
  401148:	83 fb 05             	cmp    $0x5,%ebx
  40114b:	7e e8                	jle    401135 <phase_6+0x41>
  40114d:	49 83 c5 04          	add    $0x4,%r13
  401151:	eb c1                	jmp    401114 <phase_6+0x20>
```

40110e将`%r12d`赋为0，401114将`%r13`赋给`%rbp`，401117将`%r13`指向的值赋给`%eax`，40111b将`%eax`减1，40111e和401121判断`%eax`是否不超过5:

- 若`%eax`超过5，则继续执行401123使炸弹爆炸；
- 若`%eax`不超过5，则跳到401128（4010f4+0x34）将`%r12d`加1，40112c和401130判断`%r12d`是否为6:
  - 若为6则跳到401153（4010f4+0x5f）；
  - 若不为6则继续执行401132将`%r12d`赋给`%ebx`，401135再将`%ebx`赋给`%rax`，401138再将`%rsp+%rax+4`指向的值（下一个整数）赋给`%eax`，40113b和40113e判断`%eax`的值是否等于`%rbp`指向的值：
    - 若相等则继续运行401140使炸弹爆炸；
    - 若不等则跳到401145（4010f4+0x51）将`%ebx`加1，401148和40114b判断`%ebx`是否不超过5：
      - 若不超过5则跳到401135（4010f4+0x41）循环；
      - 若超过5则继续执行40114d将`%r13`加4，401151跳到401114（4010f4+0x20）循环

可以看出此处的循环等价于：

```c
next == 0;// %r12d
while (1) {
  num = *p_loc;// %r13->%rbp, (%r13)->%eax
  if (--num > 5)// %eax
    explode_bomb();
  next++;// %r12d
  if (next == 6)
    break;
  for (i = next; i <= 5; i++) {// %r12d->%ebx; %ebx <= 5
    num_next = input[i];// %ebx->%rax, (%rsp + %rax + 4)->%eax
    if (num_next == *p_loc)// %eax == (%rbp)
      explode_bomb();
  }
  p_loc++;// %e13
}
```

可以看出，这6个无符号整数减1后不能超过5（第4、5行），即在1～6之间；且这6个数互不相同（第9～13行），即这6个整数的可能序列是**1～6的全排列**。

---

```assembly
  401153:	48 8d 74 24 18       	lea    0x18(%rsp),%rsi
  401158:	4c 89 f0             	mov    %r14,%rax
  40115b:	b9 07 00 00 00       	mov    $0x7,%ecx
  401160:	89 ca                	mov    %ecx,%edx
  401162:	2b 10                	sub    (%rax),%edx
  401164:	89 10                	mov    %edx,(%rax)
  401166:	48 83 c0 04          	add    $0x4,%rax
  40116a:	48 39 f0             	cmp    %rsi,%rax
  40116d:	75 f1                	jne    401160 <phase_6+0x6c>
```

上一循环跳出后即执行400153将`%rsp+24`的地址（6个整数末尾的位置）给`%rsi`，401158将`%r14`（6个整数开头的位置）赋给`%rax`，40115b将`%ecx`赋为7，401160将`%ecx`（7）赋给`%edx`，401162将`%edx`（7）减去`%rax`指向的值（第“`%rax+1`”个整数），401164将`%edx`（7）赋给`%rax`指向的值（第“`%rax+1`”个整数），401166将`%rax`加4（指向下一个整数），40116a和40116d判断`%rsi`（6个整数末尾的位置）和`%rax`（逐渐向后移动的指针）是否相等：

- 若不等则跳到401160（4010f4+0x6c）循环；
- 若相等则继续执行下一部分；

可以看出此处的循环等价于：

```c
p_end = rsp + 6;// %rsp + 24->%rsi
ecx = 7;// 7->%ecx
for (i = p_begin; i != p_end; i++) {// %r14->%rax; %rax != %rsi; %rax += 4
  num_new = ecx - *i;// %ecx->%edx, %edx - (%rax)->%edx
  *i = num_new;// %edx->(%rax)
}
```

可以看出，原来输入到栈中的第`i`个整数被重写为`7-i`，将重写后的6个整数视为数组`nums[6]`

---

```assembly
  40116f:	be 00 00 00 00       	mov    $0x0,%esi
  401174:	eb 21                	jmp    401197 <phase_6+0xa3>
  401176:	48 8b 52 08          	mov    0x8(%rdx),%rdx
  40117a:	83 c0 01             	add    $0x1,%eax
  40117d:	39 c8                	cmp    %ecx,%eax
  40117f:	75 f5                	jne    401176 <phase_6+0x82>
  401181:	eb 05                	jmp    401188 <phase_6+0x94>
  401183:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  401188:	48 89 54 74 20       	mov    %rdx,0x20(%rsp,%rsi,2)
  40118d:	48 83 c6 04          	add    $0x4,%rsi
  401191:	48 83 fe 18          	cmp    $0x18,%rsi
  401195:	74 14                	je     4011ab <phase_6+0xb7>
  401197:	8b 0c 34             	mov    (%rsp,%rsi,1),%ecx
  40119a:	83 f9 01             	cmp    $0x1,%ecx
  40119d:	7e e4                	jle    401183 <phase_6+0x8f>
  40119f:	b8 01 00 00 00       	mov    $0x1,%eax
  4011a4:	ba d0 32 60 00       	mov    $0x6032d0,%edx
  4011a9:	eb cb                	jmp    401176 <phase_6+0x82>
```

40116f将`%esi`（外层循环的索引）赋为0，401174跳到401197（4010f4+0xa3）将`%rsp`指向的值（“数组基地址”）和`%rsi`指向的值（“偏移量”）相加赋给`%ecx`（依次取6个整数），40119a和40119d判断`%ecx`（当前整数）是否不超过1：

- 若不超过1则跳到401183（4010f4+0x8f）将地址`0x6032d0`赋给`%edx`（某指针），401188将`%rdx`（某指针，存放地址`0x6032d0`）赋给`32+%rsp+%rsi*2`指向的值，40118d将`%rsi`加4（外层循环索引向后移动），401191和401195判断`%rsi`是否等于24（是否走到第6个整数）：
  - 若等于24则跳到4011ab（4010f4+0xb7）继续执行下一部分；
  - 若不等于24则继续执行401197（外层）循环
- 若超过1则继续执行40119f将`%eax`（内层循环的索引）赋为1，4011a4将地址`0x6032d0`赋给`%edx`（某指针），4011a9跳到401176（4010f4+0x82）将`%rdx+8`指向的值（下一个指针的值）赋给`%rdx`，40117a将`%eax`（内层循环的索引）加1，40117d和40117f判断`%ecx`（当前整数）和`%eax`（内层循环的索引）是否相等：
  - 若不等则跳到401176（4010f4+0x82）（内层）循环；
  - 若相等则继续执行401181跳到401188（4010f4+0x94）继续外层循环

可以看出此处的两层循环等价于：

```c
for (i = 0; i != 6; i++) {// %esi
  num = nums[i];// (%rsp + i)->%ecx
  if (num <= 1) {
    p = 0x6032d0;// 0x6032d0->%edx
    new_nums[i] = p;// %rdx->(%rsp + 2i + 32)
    goto 0x401191;
  } else {
    for (j = 1; j != num; j++) {// %eax
      p = 0x6032d0;// 0x6032d0->%edx
      p = *(++p);// (%rdx + 8)->%rdx
      new_nums[i] = p;// %rdx->(%rsp + 2i + 32)
      goto 0x401191;
    }
  }
}
```

在意义不改变的基础上再次优化：

```c
for (i = 0; i != 6; i++) {// %esi
  num = nums[i];// (%rsp + i)->%ecx
  p = 0x6032d0;// 0x6032d0->%edx
  if (num > 1) {
    for (j = 1; j != num; j++)// %eax
      p = *(++p);// (%rdx + 8)->%rdx
  }
  new_nums[i] = p;// %rdx->(%rsp + 8i + 32)
}
```

可以看出，此处再次生成了一个新的**地址**数组`new_nums[6]`（存放的是链表各个结点的**地址**，新数组起始地址是`%rsp+32`，依次加8），它是以`nums[6]`数组（上面重写后的）为索引，将（以`0x6032d0`为头指针的）**链表**的第`nums[i]`个数的**地址**赋给`new_nums[i]`。构造这个地址数组的目的就是排序。

由于输入的6个数是1～6的全排列，`7-i`后依然是1～6的全排列，所以新生成的数组的6个元素可以通过`0x6032d0+i`（i=1,2,3,4,5,6）来获得（顺序未知），直接`gdb`：

```shell
(gdb) x/24 0x6032d0
```

得到输出如下：

```shell
0x6032d0 <node1>:       332     1       6304480 0
0x6032e0 <node2>:       168     2       6304496 0
0x6032f0 <node3>:       924     3       6304512 0
0x603300 <node4>:       691     4       6304528 0
0x603310 <node5>:       477     5       6304544 0
0x603320 <node6>:       443     6       0       0
```

左侧为链表6个结点的地址（新数组中的元素），右侧第1列则是6个结点的数据域，第3列是6个结点的指针域（十进制）

---

```assembly
  4011ab:	48 8b 5c 24 20       	mov    0x20(%rsp),%rbx
  4011b0:	48 8d 44 24 28       	lea    0x28(%rsp),%rax
  4011b5:	48 8d 74 24 50       	lea    0x50(%rsp),%rsi
  4011ba:	48 89 d9             	mov    %rbx,%rcx
  4011bd:	48 8b 10             	mov    (%rax),%rdx
  4011c0:	48 89 51 08          	mov    %rdx,0x8(%rcx)
  4011c4:	48 83 c0 08          	add    $0x8,%rax
  4011c8:	48 39 f0             	cmp    %rsi,%rax
  4011cb:	74 05                	je     4011d2 <phase_6+0xde>
  4011cd:	48 89 d1             	mov    %rdx,%rcx
  4011d0:	eb eb                	jmp    4011bd <phase_6+0xc9>
  4011d2:	48 c7 42 08 00 00 00 	movq   $0x0,0x8(%rdx)
  4011d9:	00
```

上一部分的循环跳出之后即执行4011ab将`%rsp+32`指向的值（`new_nums[0]`）赋给`%rbx`，4011b0将`%rsp+40`的地址（新数组第2个元素的地址）赋给`%rax`（索引），4011b5将`%rsp+80`的地址（新数组末尾的地址）赋给`%rsi`，4011ba将`%rbx`（新数组第1个元素）赋给`%rcx`，4011bd将`%rax`指向的值（新数组第`%rax`个元素）赋给`%rdx`，4011c0将`%rdx`（新数组第`%rax`个元素）赋给`%rcx+8`指向的值，4011c4将`%rax`加8，即移动到下一个元素的地址，4011c8和4011cb比较`%rax`和`%rsi`：

- 若不等则继续执行4011cd将`%rdx`赋给`%rcx`，4011d0跳到4011bd（4010f4+0xc9）循环；
- 若相等则跳到4011d2（4010f4+0xde）将`%rdx+8`指向的值赋为0

可以看出此处的循环等价于：

```c
num_begin = *p_new_nums;// (%rsp + 32)=new_nums[0]->%rbx
p_next = p_new_nums + 1;// %rsp + 40=&new_nums[1]->%rax
p_end = p_new_nums + 6;// %rsp + 80->%rsi
for (num = num_begin; p_next != p_end; num = num_next) {// %rbx->%rcx; %rax != %rsi; %rdx->%rcx
  num_next = *p_next;// (%rax)->%rdx
  *(num + 1) = num_next;// %rdx->(%rcx + 8)
  p_next++;// %rax += 8
}
*(num_next + 1) = 0;// 0->(%rdx + 8)
```

可以看出，此处的循环是为新数组`new_nums[6]`的各个元素添加了指针域，即第`i`个元素（i=0,1,2,3,4）的指针域用`*(new_nums[i] + 1)`表示，指向第`i+1`个元素`new_nums[i+1]`，第`5`个元素（第6个地址）的指针域赋为0。这样做的好处是可以直接用元素的值表示下一个元素的值，便于重复赋值。

---

```assembly
  4011da:	bd 05 00 00 00       	mov    $0x5,%ebp
  4011df:	48 8b 43 08          	mov    0x8(%rbx),%rax
  4011e3:	8b 00                	mov    (%rax),%eax
  4011e5:	39 03                	cmp    %eax,(%rbx)
  4011e7:	7d 05                	jge    4011ee <phase_6+0xfa>
  4011e9:	e8 4c 02 00 00       	callq  40143a <explode_bomb>
  4011ee:	48 8b 5b 08          	mov    0x8(%rbx),%rbx
  4011f2:	83 ed 01             	sub    $0x1,%ebp
  4011f5:	75 e8                	jne    4011df <phase_6+0xeb>
  4011f7:	48 83 c4 50          	add    $0x50,%rsp
  4011fb:	5b                   	pop    %rbx
  4011fc:	5d                   	pop    %rbp
  4011fd:	41 5c                	pop    %r12
  4011ff:	41 5d                	pop    %r13
  401201:	41 5e                	pop    %r14
  401203:	c3                   	retq
```

上一部分加完指针域后继续执行4011da将`%ebp`（索引）赋为5，4011df将`%rbx+8`指向的值（开始时`%rbx`是`new_nums[0]`，此时`%rbx+8`即为新数组第`0`个元素的指针域，指向的值就是下一个元素）赋给`%rax`（即当前元素的下一个元素），4011e3将`%rax`指向的值（新数组下一个元素作为地址指向的值）赋给`%eax`，4011e5和4011e7将`%eax`和`%rbx`指向的值（新数组当前元素作为地址指向的值）比较：

- 若`%rbx`指向的值（当前元素指向值）小于`%eax`（下一元素指向值），则继续执行4011e9使炸弹爆炸（由此得知**当前元素指向值必须大于等于下一元素指向值**）；
- 若`%rbx`指向的值（当前元素指向值）不小于`%eax`（下一元素指向值），则跳到4011ee（4010f4+0xfa）将`%rbx+8`指向的值（下一个元素）赋给`%rbx`（好处体现），4011f2将`%ebp`（索引）减1，4011f5判断减1后是否为0:
  - 若不为0则跳到4011df（4010f4+0xeb）循环；
  - 若为0则继续执行并弹栈**返回**

可以看出此处的循环等价于：

```c
for (i = 5; i != 0; i--) {// %ebp
  num_next = *(num + 1);// (%rbx + 8)->%rax
  number_next = *num_next;// (%rax)->%eax
  if (*num < number_next)// (%rbx) < %eax
    explode_bomb();
  num = *(num + 1);// (%rbx + 8)->%rbx
}
return num_next;// %rax
```

可以看出，需要满足`*num >= **(num+1)`即对于新数组存放的6个地址值，它们对应指向的数据应该按顺序递减。

---

整理一下，我们需要输入6个整数，是1～6的排列，每个数再对7求补，得到新的序列。新的序列一一对应了一个链表的6个结点，即：

```shell
1:       332
2:       168
3:       924
4:       691
5:       477
6:       443
```

而这6个结点的数据域（上面右侧一列）需要递减排列：

```shell
3:       924
4:       691
5:       477
6:       443
1:       332
2:       168
```

此时便得到了新的序列为`3`，`4`，`5`，`6`，`1`，`2`。

输入序列为新的序列关于7的补，即为`4`，`3`，`2`，`1`，`6`，`5`。

至此，得到`phase_6`的password为

```shell
4 3 2 1 6 5
```

# Secret phase: *binary search tree*

虽然已经解决了6个phase，但在`bomb.c`的最后却有这样一句话：

```c
/* Wow, they got it!  But isn't something... missing?  Perhaps
 * something they overlooked?  Mua ha ha ha ha! */
```

行百里者，半于九十。游戏尚未结束。

---

在`objdump`反汇编生成的文件中可以看到，6个`phase`函数的后面存在`secret_phase`函数，此函数应该是拆除隐藏`phase`的线索。

但是当6个`phase`的password输入完成后，程序自动终止。如何进入此`secret_phase`呢？

可以在`objdump`反汇编生成的文件中查找`secret_phase`，看看在哪里调用了此函数，沿此线索可能找到入口。

```assembly
00000000004015c4 <phase_defused>:
	...
  401630:	e8 0d fc ff ff       	callq  401242 <secret_phase>
  ...
```

不出所料，在`phase_defused`函数的401630处调用了`secret_phase`函数，而根据本文开头对`main`函数的分析可知在每个`phase`通过后都会运行`phase_defused`函数。

下面开始分析`phase_defused`函数，它很可能就是开启`secret_phase`大门的**入口**。

## Enter

`phase_defused`对应的汇编语句如下：

```assembly
00000000004015c4 <phase_defused>:
  4015c4:	48 83 ec 78          	sub    $0x78,%rsp
  4015c8:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  4015cf:	00 00
  4015d1:	48 89 44 24 68       	mov    %rax,0x68(%rsp)
  4015d6:	31 c0                	xor    %eax,%eax
  4015d8:	83 3d 81 21 20 00 06 	cmpl   $0x6,0x202181(%rip)        # 603760 <num_input_strings>
  4015df:	75 5e                	jne    40163f <phase_defused+0x7b>
  4015e1:	4c 8d 44 24 10       	lea    0x10(%rsp),%r8
  4015e6:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  4015eb:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  4015f0:	be 19 26 40 00       	mov    $0x402619,%esi
  4015f5:	bf 70 38 60 00       	mov    $0x603870,%edi
  4015fa:	e8 f1 f5 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  4015ff:	83 f8 03             	cmp    $0x3,%eax
  401602:	75 31                	jne    401635 <phase_defused+0x71>
  401604:	be 22 26 40 00       	mov    $0x402622,%esi
  401609:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  40160e:	e8 25 fd ff ff       	callq  401338 <strings_not_equal>
  401613:	85 c0                	test   %eax,%eax
  401615:	75 1e                	jne    401635 <phase_defused+0x71>
  401617:	bf f8 24 40 00       	mov    $0x4024f8,%edi
  40161c:	e8 ef f4 ff ff       	callq  400b10 <puts@plt>
  401621:	bf 20 25 40 00       	mov    $0x402520,%edi
  401626:	e8 e5 f4 ff ff       	callq  400b10 <puts@plt>
  40162b:	b8 00 00 00 00       	mov    $0x0,%eax
  401630:	e8 0d fc ff ff       	callq  401242 <secret_phase>
  401635:	bf 58 25 40 00       	mov    $0x402558,%edi
  40163a:	e8 d1 f4 ff ff       	callq  400b10 <puts@plt>
  40163f:	48 8b 44 24 68       	mov    0x68(%rsp),%rax
  401644:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  40164b:	00 00
  40164d:	74 05                	je     401654 <phase_defused+0x90>
  40164f:	e8 dc f4 ff ff       	callq  400b30 <__stack_chk_fail@plt>
  401654:	48 83 c4 78          	add    $0x78,%rsp
  401658:	c3                   	retq
  401659:	90                   	nop
  40165a:	90                   	nop
  40165b:	90                   	nop
  40165c:	90                   	nop
  40165d:	90                   	nop
  40165e:	90                   	nop
  40165f:	90                   	nop
```

---

分段分析：

---

```assembly
  4015c4:	48 83 ec 78          	sub    $0x78,%rsp
  4015c8:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
  4015cf:	00 00
  4015d1:	48 89 44 24 68       	mov    %rax,0x68(%rsp)
  4015d6:	31 c0                	xor    %eax,%eax
  4015d8:	83 3d 81 21 20 00 06 	cmpl   $0x6,0x202181(%rip)        # 603760 <num_input_strings>
  4015df:	75 5e                	jne    40163f <phase_defused+0x7b>
  ...
  40163f:	48 8b 44 24 68       	mov    0x68(%rsp),%rax
  401644:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
  40164b:	00 00
  40164d:	74 05                	je     401654 <phase_defused+0x90>
  40164f:	e8 dc f4 ff ff       	callq  400b30 <__stack_chk_fail@plt>
  401654:	48 83 c4 78          	add    $0x78,%rsp
  401658:	c3                   	retq
	...
```

4015c4～4015d1和40163f～401658的含义参见[`phase_5`起始部分](#stackCanacy)。

4015d6将`%eax`赋为0，4015d8和4015df判断`%rip+0x202181`指向的值是否为6:

- 若不为6则跳到40163f（4015c4+0x7b）继续执行并**返回**；
- 若为6则继续执行

可以看出，只有当`%rip+0x202181`指向的值为6时才会继续执行。

结合4015d8右侧的注释可以发现，`%rip+0x202181`的地址为`0x603760`，再次`gdb`：

```shell
(gdb) x/d 0x603760
```

得到输出如下：

```shell
0x603760 <num_input_strings>:   0
```

发现`0x603760`指向的值为0，而结合`<num_input_strings>`名称可以**推测**此地址指向的值是一个**计数器**，用于记录我们通过了几个`phase`。

下面进行验证，利用`gdb`打断点，查看每次通过`phase`后`0x603760`指向的值的变化（输入和输出一同列出，省略了文件路径）：

```shell
(gdb) x/d 0x603760
0x603760 <num_input_strings>:   0
(gdb) b explode_bomb
Breakpoint 1 at 0x40143a
(gdb) b phase_1
Breakpoint 2 at 0x400ee0
(gdb) b phase_2
Breakpoint 3 at 0x400efc
(gdb) b phase_3
Breakpoint 4 at 0x400f43
(gdb) b phase_4
Breakpoint 5 at 0x40100c
(gdb) b phase_5
Breakpoint 6 at 0x401062
(gdb) b phase_6
Breakpoint 7 at 0x4010f4
(gdb) r
Starting program: /.../bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.

Breakpoint 1, 0x0000000000400ee0 in phase_1 ()
(gdb) x/d 0x603760
0x603760 <num_input_strings>:   1
(gdb) c
Continuing.
Phase 1 defused. How about the next one?
1 2 4 8 16 32

Breakpoint 2, 0x0000000000400efc in phase_2 ()
(gdb) x/d 0x603760
0x603760 <num_input_strings>:   2
(gdb) c
Continuing.
That's number 2.  Keep going!
0 207

Breakpoint 3, 0x0000000000400f43 in phase_3 ()
(gdb) x/d 0x603760
0x603760 <num_input_strings>:   3
(gdb) c
Continuing.
Halfway there!
0 0

Breakpoint 4, 0x000000000040100c in phase_4 ()
(gdb) x/d 0x603760
0x603760 <num_input_strings>:   4
(gdb) c
Continuing.
So you got that one.  Try this one.
9/N567

Breakpoint 5, 0x0000000000401062 in phase_5 ()
(gdb) x/d 0x603760
0x603760 <num_input_strings>:   5
(gdb) c
Continuing.
Good work!  On to the next...
4 3 2 1 6 5

Breakpoint 6, 0x00000000004010f4 in phase_6 ()
(gdb) x/d 0x603760
0x603760 <num_input_strings>:   6
(gdb) c
Continuing.
Congratulations! You've defused the bomb!
[Inferior 1 (process 8462) exited normally]
```

可以发现，每次成功通过一个`phase`后，`0x603760`指向的值都会加1。而此处判断当`0x603760`指向的值为6时才会进行，即只有成功通过6个`phase`后，`phase_defused`函数才会继续执行4015e1及后续部分

---

```assembly
  4015e1:	4c 8d 44 24 10       	lea    0x10(%rsp),%r8
  4015e6:	48 8d 4c 24 0c       	lea    0xc(%rsp),%rcx
  4015eb:	48 8d 54 24 08       	lea    0x8(%rsp),%rdx
  4015f0:	be 19 26 40 00       	mov    $0x402619,%esi
  4015f5:	bf 70 38 60 00       	mov    $0x603870,%edi
  4015fa:	e8 f1 f5 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
```

`0x603760`指向的值为6时，继续执行4015e1将`%rsp+16`的地址给`%r8`，4015e6将`%rsp+12`的地址给`%rcx`，4015eb将`%rsp+8`的地址给`%rdx`，4015f0将地址`0x402619`给`%esi`，4015f5将地址`0x603870`给`%edi`，4015fa又调用了`sscanf`函数，**推测**地址`0x402619`指向的可能是一个格式化字符串，`gdb`：

```shell
(gdb) x/s 0x402619
```

得到输出如下：

```shell
0x402619:       "%d %d %s"
```

不出所料，我们应该在某处输入两个整数（存放于`%rsp+8`和`%rsp+12`中），再输入一个字符串（存放于`%rsp+16`中）。

回忆6个`phase`，发现没有一个`phase`的密码符合这种格式，但是`phase_3`和`phase_4`的密码均为两个整数。**推测**可能在`phase_3`或`phase_4`输入两个整数后，再输入一个字符串，可能就会开启`secret_phase`。

注意此时还有一个地址`0x603870`被传给了`%edi`，**推测**可能指向一个字符串用于和输入的字符串进行比对。

直接`gdb`查看`0x603870`指向的值：

```shell
(gdb) x/s 0x603870
```

得到输出如下：

```shell
0x603870 <input_strings+240>:   ""
```

发现是空字符串，说明推测错误。再次推测，`0x603870`指向的值不可能一直是空字符串，**推测**它可能会随着我们输入密码而发生变化。使用`gdb`，并在`phase_defused`入口处打一个断点（输入和输出一同列出，省略了文件路径）：

```shell
(gdb) b phase_defused
Breakpoint 8 at 0x4015c4
(gdb) r
Starting program: /.../bomb
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Border relations with Canada have never been better.

Breakpoint 1, 0x0000000000400ee0 in phase_1 ()
(gdb) x/s 0x603870
0x603870 <input_strings+240>:   ""
(gdb) c
Continuing.

Breakpoint 8, 0x00000000004015c4 in phase_defused ()
(gdb) c
Continuing.
Phase 1 defused. How about the next one?
1 2 4 8 16 32

Breakpoint 2, 0x0000000000400efc in phase_2 ()
(gdb) x/s 0x603870
0x603870 <input_strings+240>:   ""
(gdb) c
Continuing.

Breakpoint 8, 0x00000000004015c4 in phase_defused ()
(gdb) c
Continuing.
That's number 2.  Keep going!
0 207

Breakpoint 3, 0x0000000000400f43 in phase_3 ()
(gdb) x/s 0x603870
0x603870 <input_strings+240>:   ""
(gdb) c
Continuing.

Breakpoint 8, 0x00000000004015c4 in phase_defused ()
(gdb) c
Continuing.
Halfway there!
0 0

Breakpoint 4, 0x000000000040100c in phase_4 ()
(gdb) x/s 0x603870
0x603870 <input_strings+240>:   "0 0"
(gdb) c
Continuing.

Breakpoint 8, 0x00000000004015c4 in phase_defused ()
(gdb) c
Continuing.
So you got that one.  Try this one.
9/N567

Breakpoint 5, 0x0000000000401062 in phase_5 ()
(gdb) x/s 0x603870
0x603870 <input_strings+240>:   "0 0"
(gdb) c
Continuing.

Breakpoint 8, 0x00000000004015c4 in phase_defused ()
(gdb) c
Continuing.
Good work!  On to the next...
4 3 2 1 6 5

Breakpoint 6, 0x00000000004010f4 in phase_6 ()
(gdb) x/s 0x603870
0x603870 <input_strings+240>:   "0 0"
(gdb) c
Continuing.

Breakpoint 8, 0x00000000004015c4 in phase_defused ()
(gdb) c
Continuing.
Congratulations! You've defused the bomb!
[Inferior 1 (process 13412) exited normally]
```

可以发现，在通过`phase_4`后，`0x603870`指向的值发生了变化——变成了我们输入的`"0 0"`（不应认为此时`0x603870`指向的值就是`"0 0"`，而是会因我们输入的`phase_4`密码的不同而不同，此处可以自行验证当输入不同的的`phase_4`密码时`0x603870`指向的值，同时也可以理解`phase_4`密码存在多种可能性的原因）

---

```assembly
  4015ff:	83 f8 03             	cmp    $0x3,%eax
  401602:	75 31                	jne    401635 <phase_defused+0x71>
  ...
  401630:	e8 0d fc ff ff       	callq  401242 <secret_phase>
  401635:	bf 58 25 40 00       	mov    $0x402558,%edi
```

继续执行4015ff和401602判断`%eax`（`sscanf`函数的返回值）是否为3：

- 若为3则继续执行；
- 若不为3则跳到401635（4015c4+0x71）继续执行输出提示信息

结合后续语句可以看出，`secret_phase`在401630处被调用，若跳到401635则跳过了`secret_phase`的调用，且不会再跳回来。即要想进入`secret_phase`，则`sscanf`函数的返回值必须为3，也就是说`0x603870`指向的值（**输入的`phase_4`的密码**）**除了两个整数，必须还要有一个字符串**

这个额外输入的字符串是什么？继续分析～

---

```assembly
  401604:	be 22 26 40 00       	mov    $0x402622,%esi
  401609:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
  40160e:	e8 25 fd ff ff       	callq  401338 <strings_not_equal>
  401613:	85 c0                	test   %eax,%eax
  401615:	75 1e                	jne    401635 <phase_defused+0x71>
  ...
  401630:	e8 0d fc ff ff       	callq  401242 <secret_phase>
  401635:	bf 58 25 40 00       	mov    $0x402558,%edi
```

401604将地址`0x402622`给`%esi`，401609把`%rsp+16`的地址（指向在两个整数后输入的字符串）给`%rdi`，40160e调用`strings_not_equal`函数判断两个字符串是否相等（相等返回0，不等返回1），401613和401615判断`%eax`（`strings_not_equal`函数的返回值）是否为0：

- 若为0则继续执行；
- 若不为0则跳到401635（4015c4+0x71）继续执行输出提示信息

和上述分析相同，地址`0x402622`指向的应该是一个字符串，且要想进入`secret_phase`，在两个整数后输入的字符串必须和此字符串相等。利用`gdb`查看地址`0x402622`：

```shell
(gdb) x/s 0x402622
```

 得到输出如下：

```shell
0x402622:       "DrEvil"
```

即`bomb.c`文件开头注释部分提到的“犯罪者”的名字

至此，我们已经找到了开启`secret_phase`大门的入口——**在`phase_4`输入密码时额外输入字符串`"DrEvil"`**。

## Defuse

`secret_phase`对应的汇编语句如下：

```assembly
0000000000401242 <secret_phase>:
  401242:	53                   	push   %rbx
  401243:	e8 56 02 00 00       	callq  40149e <read_line>
  401248:	ba 0a 00 00 00       	mov    $0xa,%edx
  40124d:	be 00 00 00 00       	mov    $0x0,%esi
  401252:	48 89 c7             	mov    %rax,%rdi
  401255:	e8 76 f9 ff ff       	callq  400bd0 <strtol@plt>
  40125a:	48 89 c3             	mov    %rax,%rbx
  40125d:	8d 40 ff             	lea    -0x1(%rax),%eax
  401260:	3d e8 03 00 00       	cmp    $0x3e8,%eax
  401265:	76 05                	jbe    40126c <secret_phase+0x2a>
  401267:	e8 ce 01 00 00       	callq  40143a <explode_bomb>
  40126c:	89 de                	mov    %ebx,%esi
  40126e:	bf f0 30 60 00       	mov    $0x6030f0,%edi
  401273:	e8 8c ff ff ff       	callq  401204 <fun7>
  401278:	83 f8 02             	cmp    $0x2,%eax
  40127b:	74 05                	je     401282 <secret_phase+0x40>
  40127d:	e8 b8 01 00 00       	callq  40143a <explode_bomb>
  401282:	bf 38 24 40 00       	mov    $0x402438,%edi
  401287:	e8 84 f8 ff ff       	callq  400b10 <puts@plt>
  40128c:	e8 33 03 00 00       	callq  4015c4 <phase_defused>
  401291:	5b                   	pop    %rbx
  401292:	c3                   	retq
  401293:	90                   	nop
  401294:	90                   	nop
  401295:	90                   	nop
  401296:	90                   	nop
  401297:	90                   	nop
  401298:	90                   	nop
  401299:	90                   	nop
  40129a:	90                   	nop
  40129b:	90                   	nop
  40129c:	90                   	nop
  40129d:	90                   	nop
  40129e:	90                   	nop
  40129f:	90                   	nop
```

---

分段分析：

---

```assembly
  401242:	53                   	push   %rbx
  401243:	e8 56 02 00 00       	callq  40149e <read_line>
  401248:	ba 0a 00 00 00       	mov    $0xa,%edx
  40124d:	be 00 00 00 00       	mov    $0x0,%esi
  401252:	48 89 c7             	mov    %rax,%rdi
  401255:	e8 76 f9 ff ff       	callq  400bd0 <strtol@plt>
```

401243调用`read_line`函数读取一行字符串（返回值存放于`%rax`），401248将`%edx`赋为10，40124d将`%esi`赋为0，401252将`%rax`（`read_line`函数返回值）赋给`%rdi`，401255调用`strtol`函数（[什么是`strtol`函数？](http://www.cplusplus.com/reference/cstdlib/strtol/)）。

`strtol`函数将字符串转化为`long`型整数：`%rdi`作为传入的第1个参数，作为用来解析的字符串（输入串）；`%esi`作为传入的第2个参数，作为需要解析部分的结束地址（`NULL`）；`%edx`作为整数的进制（`10`代表十进制）。函数的返回值存放于`%rax`中

---

```assembly
  40125a:	48 89 c3             	mov    %rax,%rbx
  40125d:	8d 40 ff             	lea    -0x1(%rax),%eax
  401260:	3d e8 03 00 00       	cmp    $0x3e8,%eax
  401265:	76 05                	jbe    40126c <secret_phase+0x2a>
  401267:	e8 ce 01 00 00       	callq  40143a <explode_bomb>
  40126c:	89 de                	mov    %ebx,%esi
  40126e:	bf f0 30 60 00       	mov    $0x6030f0,%edi
  401273:	e8 8c ff ff ff       	callq  401204 <fun7>
```

40125a将`%rax`（`strtol`函数的返回值）赋给`%rbx`，40125d将`%rax-1`的地址给`%eax`，401260和401265判断`%eax`是否不超过1000：

- 若超过1000则继续执行401267使炸弹爆炸（由此得知**应使`strtol`函数的返回值不超过1001（无符号数）**）；

- 若不超过1000则跳到40126c（401242+0x2a）将`%ebx`（`strtol`函数的返回值）赋给`%esi`，40126e将地址`0x6030f0`赋给`%edi`，401273<span id = "backToSec">调用函数`fun7`</span>，[点此分析`fun7`函数](#fun7)

  `gdb`查看地址`0x6030f0`（`fun7`的第一个参数）（多次查看可知，只有前60个连续内存是相关的，后面均为无关数据）：

  ```shell
  (gdb) x/60a 0x6030f0
  ```

  得到输出如下：

  ```shell
  0x6030f0 <n1>:  0x24    0x603110 <n21>
  0x603100 <n1+16>:       0x603130 <n22>  0x0
  0x603110 <n21>: 0x8     0x603190 <n31>
  0x603120 <n21+16>:      0x603150 <n32>  0x0
  0x603130 <n22>: 0x32    0x603170 <n33>
  0x603140 <n22+16>:      0x6031b0 <n34>  0x0
  0x603150 <n32>: 0x16    0x603270 <n43>
  0x603160 <n32+16>:      0x603230 <n44>  0x0
  0x603170 <n33>: 0x2d    0x6031d0 <n45>
  0x603180 <n33+16>:      0x603290 <n46>  0x0
  0x603190 <n31>: 0x6     0x6031f0 <n41>
  0x6031a0 <n31+16>:      0x603250 <n42>  0x0
  0x6031b0 <n34>: 0x6b    0x603210 <n47>
  0x6031c0 <n34+16>:      0x6032b0 <n48>  0x0
  0x6031d0 <n45>: 0x28    0x0
  0x6031e0 <n45+16>:      0x0     0x0
  0x6031f0 <n41>: 0x1     0x0
  0x603200 <n41+16>:      0x0     0x0
  0x603210 <n47>: 0x63    0x0
  0x603220 <n47+16>:      0x0     0x0
  0x603230 <n44>: 0x23    0x0
  0x603240 <n44+16>:      0x0     0x0
  0x603250 <n42>: 0x7     0x0
  0x603260 <n42+16>:      0x0     0x0
  0x603270 <n43>: 0x14    0x0
  0x603280 <n43+16>:      0x0     0x0
  0x603290 <n46>: 0x2f    0x0
  0x6032a0 <n46+16>:      0x0     0x0
  0x6032b0 <n48>: 0x3e9   0x0
  0x6032c0 <n48+16>:      0x0     0x0
  ```

  可以发现这是一棵二叉树，其中`<nab>`表示第`a`层从左向右数的第`b`个结点，且数值依次为结点数据、左子树地址、右子树地址。画出二叉树（十六进制）：

  ![hex](https://i0.hdslb.com/bfs/album/0ec2d2de58042325ee5f72ebceaa7012110b8af0.jpg)

  转换成十进制：

  ![dec](https://i0.hdslb.com/bfs/album/ac2cff619185a779c4256f036fe0b80a2ff973f1.jpg)

  即传入`fun7`函数的第一个参数就是此二叉树根结点的地址。结合`fun7`函数改写的C语句：

  ```c
  int fun7(int* p_node, int target) {
  	if (p_node == 0)
      return 0xffffffff;
    num = *p_node;// current node
    if (num <= target) {
      result = 0;
      if (num == target)
        return result;// 0
      else {
      	p_node = *(p_node + 2);// right child
      	result = fun7(p_node, target);
      	return result * 2 + 1;
      }
    } else {
      p_node = *(p_node + 1);// left child
      result = fun7(p_node, target);
    return result * 2;
    }
  }
  ```

  可以发现`fun7`函数的功能是：

  - 若当前结点为空，则返回`0xffffffff`；
  - 若当前结点数据为`target`，则返回`0`；
  - 若当前结点数据小于`target`，则继续搜索右子树，返回右子树搜索返回值的2倍+1；
  - 若当前结点数据大于`target`，则继续搜索左子树，返回左子树搜索返回值的2倍

---

```assembly
  401278:	83 f8 02             	cmp    $0x2,%eax
  40127b:	74 05                	je     401282 <secret_phase+0x40>
  40127d:	e8 b8 01 00 00       	callq  40143a <explode_bomb>
  401282:	bf 38 24 40 00       	mov    $0x402438,%edi
  401287:	e8 84 f8 ff ff       	callq  400b10 <puts@plt>
  40128c:	e8 33 03 00 00       	callq  4015c4 <phase_defused>
  401291:	5b                   	pop    %rbx
  401292:	c3                   	retq
```

401278判断`%eax`（`fun7`函数的返回值）是否为2：

- 若不为2则继续执行40127d使炸弹爆炸（由此得知**`fun7`函数的返回值须为2**）；
- 若为2则跳到401282（401242+0x40）将地址`0x402438`赋给`%edi`，401287调用`puts`函数，40128c调用`phase_defused`函数，401291和401292弹栈**返回**

`gdb`查看地址`0x402438`指向的值：

```shell
(gdb) x/s 0x402438
```

得到输出如下：

```shell
0x402438:       "Wow! You've defused the secret stage!"
```

可以看出之后调用函数即输出文本，至此拆弹结束。

---

综合以上分析，`secret_phase`的密码就是**使`fun7`函数的返回值为2的`target`值**。

什么时候`fun7`函数的返回值为2？再次观察二叉树的值：

![dec](https://i0.hdslb.com/bfs/album/ac2cff619185a779c4256f036fe0b80a2ff973f1.jpg)

发现左子树各结点数值均小于根结点、右子树各结点数值均大于根结点——这是一棵**二叉排序树（Binary Search Tree，BST）**。

观察`fun7`函数：

> - 若当前结点为空，则返回`0xffffffff`；
> - 若当前结点数据为`target`，则返回`0`；
> - 若当前结点数据小于`target`，则继续搜索右子树，返回右子树搜索返回值的2倍+1；
> - 若当前结点数据大于`target`，则继续搜索左子树，返回左子树搜索返回值的2倍。

分析：

- 顺推（从根结点开始）：

  最终一定找到数值等于`target`的结点（即`target`一定在树中）；

- 逆推（从值为`target`的结点开始）：

  - 开始可以有任意次`0`，即可以先沿右路（顺推时的左子树）返回任意次（`2*0=0`）；
  - 倒数第二次一定沿左路（顺推时的右子树）返回（`2*(2*0)+1=2*0+1=1`）；
  - 最后一次一定沿右路（顺推时的左子树）返回到根结点（`2*(2*(2*0)+1)=2*(2*0+1)=2*1=2`）。

得出结论：

- 若逆推开始沿右路返回0次，则为`22`；
- 若逆推开始沿右路返回1次，则为`20`。

至此，得到`secret_phase`的password为

```shell
22
```

或

```shell
20
```

# Functions

下面是拆弹过程中遇到的一些函数：

---



## explode_bomb

`explode_bomb`函数对应的汇编语句如下：

```assembly
000000000040143a <explode_bomb>:
  40143a:	48 83 ec 08          	sub    $0x8,%rsp
  40143e:	bf a3 25 40 00       	mov    $0x4025a3,%edi
  401443:	e8 c8 f6 ff ff       	callq  400b10 <puts@plt>
  401448:	bf ac 25 40 00       	mov    $0x4025ac,%edi
  40144d:	e8 be f6 ff ff       	callq  400b10 <puts@plt>
  401452:	bf 08 00 00 00       	mov    $0x8,%edi
  401457:	e8 c4 f7 ff ff       	callq  400c20 <exit@plt>
```

`gdb`查看`0x4025a3`和`0x4025ac`指向的值：

```shell
(gdb) x/s 0x4025a3
(gdb) x/s 0x4025ac
```

可以得到输出如下：

```shell
0x4025a3:       "\nBOOM!!!"
0x4025ac:       "The bomb has blown up."
```

可以看出`explode_bomb`函数的功能就是**引爆炸弹**。

## strings_not_equal

<span id = "equal">`strings_not_equal`函数</span>对应的汇编语句如下：

```assembly
0000000000401338 <strings_not_equal>:
  401338:	41 54                	push   %r12
  40133a:	55                   	push   %rbp
  40133b:	53                   	push   %rbx
  40133c:	48 89 fb             	mov    %rdi,%rbx
  40133f:	48 89 f5             	mov    %rsi,%rbp
  401342:	e8 d4 ff ff ff       	callq  40131b <string_length>
  401347:	41 89 c4             	mov    %eax,%r12d
  40134a:	48 89 ef             	mov    %rbp,%rdi
  40134d:	e8 c9 ff ff ff       	callq  40131b <string_length>
  401352:	ba 01 00 00 00       	mov    $0x1,%edx
  401357:	41 39 c4             	cmp    %eax,%r12d
  40135a:	75 3f                	jne    40139b <strings_not_equal+0x63>
  40135c:	0f b6 03             	movzbl (%rbx),%eax
  40135f:	84 c0                	test   %al,%al
  401361:	74 25                	je     401388 <strings_not_equal+0x50>
  401363:	3a 45 00             	cmp    0x0(%rbp),%al
  401366:	74 0a                	je     401372 <strings_not_equal+0x3a>
  401368:	eb 25                	jmp    40138f <strings_not_equal+0x57>
  40136a:	3a 45 00             	cmp    0x0(%rbp),%al
  40136d:	0f 1f 00             	nopl   (%rax)
  401370:	75 24                	jne    401396 <strings_not_equal+0x5e>
  401372:	48 83 c3 01          	add    $0x1,%rbx
  401376:	48 83 c5 01          	add    $0x1,%rbp
  40137a:	0f b6 03             	movzbl (%rbx),%eax
  40137d:	84 c0                	test   %al,%al
  40137f:	75 e9                	jne    40136a <strings_not_equal+0x32>
  401381:	ba 00 00 00 00       	mov    $0x0,%edx
  401386:	eb 13                	jmp    40139b <strings_not_equal+0x63>
  401388:	ba 00 00 00 00       	mov    $0x0,%edx
  40138d:	eb 0c                	jmp    40139b <strings_not_equal+0x63>
  40138f:	ba 01 00 00 00       	mov    $0x1,%edx
  401394:	eb 05                	jmp    40139b <strings_not_equal+0x63>
  401396:	ba 01 00 00 00       	mov    $0x1,%edx
  40139b:	89 d0                	mov    %edx,%eax
  40139d:	5b                   	pop    %rbx
  40139e:	5d                   	pop    %rbp
  40139f:	41 5c                	pop    %r12
  4013a1:	c3                   	retq
```

分段分析：

---

```assembly
  401338:	41 54                	push   %r12
  40133a:	55                   	push   %rbp
  40133b:	53                   	push   %rbx
  40133c:	48 89 fb             	mov    %rdi,%rbx
  40133f:	48 89 f5             	mov    %rsi,%rbp
```

40133c和40133f将`%rdi`和`%rsi`寄存器的值分别给了`%rbx`和`%rbp`（注意`%rdi`存的是`input`字符串；`%rsi`存的是疑似密码的字符串）

---

```assembly
  401342:	e8 d4 ff ff ff       	callq  40131b <string_length>
```

401342调用`string_length`函数，推测可能返回某字符串的长度，<span id = "backToEqual">[点此分析`string_length`函数](#length)</span>

---

```assembly
  401347:	41 89 c4             	mov    %eax,%r12d
```

401347将`%eax`的值（即`string_length`函数返回值，`input`字符串的长度）给`%r12d`

---

```assembly
  40134a:	48 89 ef             	mov    %rbp,%rdi
```

40134a将`%rbp`的值（疑似密码的字符串）给了`%rdi`

---

```assembly
  40134d:	e8 c9 ff ff ff       	callq  40131b <string_length>
```

40134d再次调用`string_length`函数，由于`%rdi`在上一步被重新赋值为疑似密码的字符串，此处求的就是密码串的长度，存储在`%eax`中

---

```assembly
  401352:	ba 01 00 00 00       	mov    $0x1,%edx
```

401352将`%edx`的值赋为1

---

```assembly
  401357:	41 39 c4             	cmp    %eax,%r12d
  40135a:	75 3f                	jne    40139b <strings_not_equal+0x63>
  ...
  40139b:	89 d0                	mov    %edx,%eax
```

401357和40135a比较`%eax`和`%r12d`处的值，即密码串和输入串的长度：

- 若二者不相等则跳到40139b（401338+0x63）将`%eax`的值赋为1（见401352）（可以看出**密码串和输入串的长度不相等时函数返回`1`**）；
- 若二者相等则继续执行

---

```assembly
  40135c:	0f b6 03             	movzbl (%rbx),%eax
```

40135c将`%rbx`指向的内容（`input`字符串第1个字符）传给`%eax`寄存器（零扩展，见CS:APP中文第三版123页）

---

```assembly
  40135f:	84 c0                	test   %al,%al
  401361:	74 25                	je     401388 <strings_not_equal+0x50>
  ...
  401388:	ba 00 00 00 00       	mov    $0x0,%edx
  40138d:	eb 0c                	jmp    40139b <strings_not_equal+0x63>
  ...
  40139b:	89 d0                	mov    %edx,%eax
```

40135f和401361判断`%al`寄存器的值（`input`字符串第1个字符）是否为`'\0'`：

- 若为`'\0'`则跳到401388（401338+0x50）将`%edx`赋为0（继续执行40138d和40139b（401338+0x63）将`%edx`的0给`%eax`并**返回**）；
- 若不为`'\0'`则继续执行

---

```assembly
  401363:	3a 45 00             	cmp    0x0(%rbp),%al
  401366:	74 0a                	je     401372 <strings_not_equal+0x3a>
  401368:	eb 25                	jmp    40138f <strings_not_equal+0x57>
  ...
  401372:	48 83 c3 01          	add    $0x1,%rbx
  ...
  40138f:	ba 01 00 00 00       	mov    $0x1,%edx
  401394:	eb 05                	jmp    40139b <strings_not_equal+0x63>
  ...
  40139b:	89 d0                	mov    %edx,%eax
```

401363、401366和401368比较`%rbp`指向的值（密码串第1个字符）和`%al`（输入串第1个字符）：

- 若二者相等则跳到401372（401338+0x3a）；
- 若不等则继续执行跳到40138f（401338+0x57）将`%edx`赋为1（继续执行401394和40139b（401338+0x63）将`%edx`的1给`%eax`并**返回**）

---

```assembly
  40136a:	3a 45 00             	cmp    0x0(%rbp),%al
  40136d:	0f 1f 00             	nopl   (%rax)
  401370:	75 24                	jne    401396 <strings_not_equal+0x5e>
  401372:	48 83 c3 01          	add    $0x1,%rbx
  401376:	48 83 c5 01          	add    $0x1,%rbp
  40137a:	0f b6 03             	movzbl (%rbx),%eax
  40137d:	84 c0                	test   %al,%al
  40137f:	75 e9                	jne    40136a <strings_not_equal+0x32>
  401381:	ba 00 00 00 00       	mov    $0x0,%edx
  401386:	eb 13                	jmp    40139b <strings_not_equal+0x63>
  ...
  401396:	ba 01 00 00 00       	mov    $0x1,%edx
  40139b:	89 d0                	mov    %edx,%eax
```

40136a和401370比较`%rbp`指向的值（密码串第1个字符）和`%al`（输入串第1个字符）：

- 若二者不等则跳到401396（401338+0x5e）把1给`%edx`再给`%eax`（即**返回1**）；
- 若相等则继续执行401372将`%rbx`（输入串指针，见40133c）加1、401376将`%rbp`（密码串指针）加1。40137a将`%rbx`指向的内容（`input`字符串第2个字符）传给`%eax`寄存器，40137d和40137f判断`%al`的值是否为`'\0'`：
  - 若不为`'\0'`则跳到40136a（401338+0x32）循环；
  - 若为`'\0'`则继续执行401381和401386将`%edx`赋为0并跳到40139b把`%edx`的0再给`%eax`并返回（即**返回0**）

可以看出此部分的循环等价于：

```c
do {
	if (*p_passwd != locChar)// %rbp != %al
		return 1;// %eax
	p_input++;// %rbx
	p_passwd++;// %rbp
	locChar = *p_input;// %rbx->%eax
} while (locChar != '\0');// %al
return 0;// %eax
```

可以看出此部分用于判断输入串和密码串是否相等，相等则返回0；不等则返回1。

---

综合以上分析可以看出`strings_not_equal`函数的功能就是**比较`%rdi`和`%rsi`指向的字符串（即输入串和密码串），相等返回0、不等返回1，返回值存入`%eax`**。

[点此返回到`phase_1`调用此函数处](#p1)。

## string_length

<span id = "length">`string_length`函数</span>对应的汇编语句如下：

```assembly
000000000040131b <string_length>:
  40131b:	80 3f 00             	cmpb   $0x0,(%rdi)
  40131e:	74 12                	je     401332 <string_length+0x17>
  401320:	48 89 fa             	mov    %rdi,%rdx
  401323:	48 83 c2 01          	add    $0x1,%rdx
  401327:	89 d0                	mov    %edx,%eax
  401329:	29 f8                	sub    %edi,%eax
  40132b:	80 3a 00             	cmpb   $0x0,(%rdx)
  40132e:	75 f3                	jne    401323 <string_length+0x8>
  401330:	f3 c3                	repz retq
  401332:	b8 00 00 00 00       	mov    $0x0,%eax
  401337:	c3                   	retq
```

分段分析：

---

```assembly
  40131b:	80 3f 00             	cmpb   $0x0,(%rdi)
  40131e:	74 12                	je     401332 <string_length+0x17>
  ...
  401332:	b8 00 00 00 00       	mov    $0x0,%eax
```

40131b和40131e判断`%rdi`寄存器指向的值（记住`%rdi`指向的是`input`）是否为`'\0'`：

- 若为`'\0'`则跳到401332（40131b+0x17）将`%eax`寄存器的值赋为`0`后返回（可以看出**输入串为空串时函数返回`0`**）
- 若不为`'\0'`则继续执行

---

```assembly
  401320:	48 89 fa             	mov    %rdi,%rdx
  401323:	48 83 c2 01          	add    $0x1,%rdx
  401327:	89 d0                	mov    %edx,%eax
  401329:	29 f8                	sub    %edi,%eax
  40132b:	80 3a 00             	cmpb   $0x0,(%rdx)
  40132e:	75 f3                	jne    401323 <string_length+0x8>
  401330:	f3 c3                	repz retq
```

401320先将`%rdi`寄存器的值给`%rdx`（记住`%rdi`指向的是`input`），401323再将`%rdx`的值加1，401327再把`%edx`的值给`%eax`，401329`%eax`再减去`%edi`的值，40132b和40132e判断此时`%rdx`指向的值是否为`'\0'`：

- 若不为`'\0'`则跳到401323（40131b+0x8）循环；
- 若为`'\0'`则返回，此时`%eax`存储的便是`%rdi`中字符串的长度

可以看出此部分的循环等价于：

```c
p = head;// %edi->%rdx
do {
	p++;// %rdx
	result = p;// %rdx->eax
	result -= head;// %eax - %edi
} while (*p != '\0');// %rdx
return result;// %eax
```

`p`指针从字符串头`head`开始向后移动，一直移动到末尾`'\0'`处，此时`p`与`head`之差即字符串的长度

---

综合以上分析可以看出`string_length`函数的功能就是**返回`%rdi`处字符串的长度，并存入`%eax`**。

[点此返回到未分析完的`strings_not_equal`函数](#backToEqual)。

## read_six_numbers

<span id = "6numbers">`read_six_numbers`函数</span>对应的汇编语句如下：

```assembly
000000000040145c <read_six_numbers>:
  40145c:	48 83 ec 18          	sub    $0x18,%rsp
  401460:	48 89 f2             	mov    %rsi,%rdx
  401463:	48 8d 4e 04          	lea    0x4(%rsi),%rcx
  401467:	48 8d 46 14          	lea    0x14(%rsi),%rax
  40146b:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
  401470:	48 8d 46 10          	lea    0x10(%rsi),%rax
  401474:	48 89 04 24          	mov    %rax,(%rsp)
  401478:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
  40147c:	4c 8d 46 08          	lea    0x8(%rsi),%r8
  401480:	be c3 25 40 00       	mov    $0x4025c3,%esi
  401485:	b8 00 00 00 00       	mov    $0x0,%eax
  40148a:	e8 61 f7 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
  40148f:	83 f8 05             	cmp    $0x5,%eax
  401492:	7f 05                	jg     401499 <read_six_numbers+0x3d>
  401494:	e8 a1 ff ff ff       	callq  40143a <explode_bomb>
  401499:	48 83 c4 18          	add    $0x18,%rsp
  40149d:	c3                   	retq
```

分段分析：

---

```assembly
  40145c:	48 83 ec 18          	sub    $0x18,%rsp
  401460:	48 89 f2             	mov    %rsi,%rdx
  401463:	48 8d 4e 04          	lea    0x4(%rsi),%rcx
  401467:	48 8d 46 14          	lea    0x14(%rsi),%rax
  40146b:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
  401470:	48 8d 46 10          	lea    0x10(%rsi),%rax
  401474:	48 89 04 24          	mov    %rax,(%rsp)
  401478:	4c 8d 4e 0c          	lea    0xc(%rsi),%r9
  40147c:	4c 8d 46 08          	lea    0x8(%rsi),%r8
```

40145c在`%rsp`中分配24字节空间压栈，401460把`%rsi`的地址给`%rdx`，401463把`%rsi+4`的地址给`%rcx`，401467把`%rsi+20`的地址给`%rax`，40146b把`%rax`的地址给`%rsp+8`，401470把`%rsi+16`的地址给`%rax`，401474把`%rax`的地址给`%rsp+0`，401478把`%rsi+12`的地址给`%r9`，40147c把`%rsi+8`的地址给`%r8`。

这一连串的赋值可以写成：

```assembly
%rdx = %rsi
%rcx = %rsi + 4
0(%rsp) = %rsi + 16
8(%rsp) = %rsi + 20
%r9 = %rsi + 12
%r8 = %rsi + 8
```

按顺序排好：

```assembly
%rdx = %rsi
%rcx = %rsi + 4
%r8 = %rsi + 8
%r9 = %rsi + 12
0x0(%rsp) = %rsi + 16
0x8(%rsp) = %rsi + 20
```

由前面的分析可知，`%rsi`寄存器存放的是`%rsp`的内容，可知此6个地址对应了`%rsp`栈中6个`int`的地址

---

```assembly
  401480:	be c3 25 40 00       	mov    $0x4025c3,%esi
  401485:	b8 00 00 00 00       	mov    $0x0,%eax
  40148a:	e8 61 f7 ff ff       	callq  400bf0 <__isoc99_sscanf@plt>
```

401480将地址`0x4025c3`传入`%esi`寄存器，直接`gdb`查看此处内容：

```shell
(gdb) x/s 0x4025c3
```

得到输出如下：

```shell
0x4025c3:       "%d %d %d %d %d %d"
```

可以看出这就是C格式化字符串，之后的401485将`%eax`赋为0，40148a调用`sscanf`函数（[什么是`sscanf`函数？](https://www.ibm.com/support/knowledgecenter/en/ssw_ibm_i_72/rtref/sscanf.htm)），**推测**此处读取了6个整数并对应存放在上述6个地址中。另外从此格式化字符串可以**推测**，输入6个整数时中间须以1个空格隔开

---

```assembly
  40148f:	83 f8 05             	cmp    $0x5,%eax
  401492:	7f 05                	jg     401499 <read_six_numbers+0x3d>
  401494:	e8 a1 ff ff ff       	callq  40143a <explode_bomb>
  401499:	48 83 c4 18          	add    $0x18,%rsp
  40149d:	c3                   	retq
```

40148f和401492判断`%eax`的值是否大于5：

- 若大于5则跳到401499（40145c+0x3d）弹出栈并返回；
- 若不大于5则继续执行401494使炸弹爆炸

---

综合以上分析可以看出`read_six_numbers`函数的功能就是**判断是否输入了5个以上整数，若输入整数未超过5个则引爆炸弹**。

[点此返回到`phase_2`调用此函数处](#p2)。

## func4

<span id = "func4">`func4`函数</span>对应的汇编语句如下：

```assembly
0000000000400fce <func4>:
  400fce:	48 83 ec 08          	sub    $0x8,%rsp
  400fd2:	89 d0                	mov    %edx,%eax
  400fd4:	29 f0                	sub    %esi,%eax
  400fd6:	89 c1                	mov    %eax,%ecx
  400fd8:	c1 e9 1f             	shr    $0x1f,%ecx
  400fdb:	01 c8                	add    %ecx,%eax
  400fdd:	d1 f8                	sar    %eax
  400fdf:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx
  400fe2:	39 f9                	cmp    %edi,%ecx
  400fe4:	7e 0c                	jle    400ff2 <func4+0x24>
  400fe6:	8d 51 ff             	lea    -0x1(%rcx),%edx
  400fe9:	e8 e0 ff ff ff       	callq  400fce <func4>
  400fee:	01 c0                	add    %eax,%eax
  400ff0:	eb 15                	jmp    401007 <func4+0x39>
  400ff2:	b8 00 00 00 00       	mov    $0x0,%eax
  400ff7:	39 f9                	cmp    %edi,%ecx
  400ff9:	7d 0c                	jge    401007 <func4+0x39>
  400ffb:	8d 71 01             	lea    0x1(%rcx),%esi
  400ffe:	e8 cb ff ff ff       	callq  400fce <func4>
  401003:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  401007:	48 83 c4 08          	add    $0x8,%rsp
  40100b:	c3                   	retq
```

分段分析：

---

```assembly
  400fce:	48 83 ec 08          	sub    $0x8,%rsp
  400fd2:	89 d0                	mov    %edx,%eax
  400fd4:	29 f0                	sub    %esi,%eax
  400fd6:	89 c1                	mov    %eax,%ecx
  400fd8:	c1 e9 1f             	shr    $0x1f,%ecx
  400fdb:	01 c8                	add    %ecx,%eax
  400fdd:	d1 f8                	sar    %eax
```

400fce分配8字节空间压栈，400fd2将`%edx`的值（14）赋给`%eax`，400fd4将`%eax`减去`%esi`（0），400fd6将`%eax`的值（差值）赋给`%ecx`，400fd8再将`%ecx`（差值）逻辑右移31位，400fdb再将`%eax`的值（差值）加上`%ecx`（右移后的值），400fdd将二者之和算术右移1位，结果存入`%eax`

---

```assembly
  400fdf:	8d 0c 30             	lea    (%rax,%rsi,1),%ecx
  400fe2:	39 f9                	cmp    %edi,%ecx
  400fe4:	7e 0c                	jle    400ff2 <func4+0x24>
  400fe6:	8d 51 ff             	lea    -0x1(%rcx),%edx
  400fe9:	e8 e0 ff ff ff       	callq  400fce <func4>
  400fee:	01 c0                	add    %eax,%eax
  400ff0:	eb 15                	jmp    401007 <func4+0x39>
  400ff2:	b8 00 00 00 00       	mov    $0x0,%eax
  400ff7:	39 f9                	cmp    %edi,%ecx
  400ff9:	7d 0c                	jge    401007 <func4+0x39>
  400ffb:	8d 71 01             	lea    0x1(%rcx),%esi
  400ffe:	e8 cb ff ff ff       	callq  400fce <func4>
  401003:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  401007:	48 83 c4 08          	add    $0x8,%rsp
  40100b:	c3                   	retq
```

400fdf将`%rax`的值（右移1位后的值）和`%rsi`的值（0）加到`%ecx`中，400fe2和400fe4将`%ecx`（二者之和）和`%edi`（输入的第一个整数）对比：

- 若二者之和不超过输入的第1个整数，则跳到400ff2（400fce+0x24）将`%eax`（先前存放右移1位后的值）赋为0，400ff7和400ff9比较`%edi`（输入的第1个整数）和`%ecx`（二者之和）：
  - 若二者之和不小于输入的第1个整数（即二者之和等于输入的第1个整数），则跳到401007（400fce+0x39）弹出栈并返回（**此时返回值`%eax`为0**）
  - 若二者之和小于输入的第1个整数，则继续执行400ffb将`%rcx`（二者之和）加1赋给`%esi`（先前存放的是0），400ffe递归调用`func4`函数，401003将递归调用的返回值`%rax`乘2再加1，401007**返回**
- 若二者之和超过了输入的第1个整数，则继续执行400fe6将`%rcx`（二者之和）减1赋给`%edx`（先前存放的是14），400fe9递归调用`func4`函数，400fee将递归调用的返回值`%eax`乘2，400ff0跳到401007（400fce+0x39）弹出栈并**返回**

结合两部分的分析，可以看出此部分的递归调用等价于：

```c
int func4(int passwd, int min, int max) {// %edi, %esi, %edx->%eax
	int len = max - min;// %ecx
  unsigned sign = len >> 31;// %ecx
  int half_len = (len + sign) >> 1;// %eax

  int mid = half_len + min;// %ecx
  if (mid <= passwd) {
    half_len = 0;// %eax
    if (mid >= passwd)// mid == passwd
      return half_len;
    else {// mid < passwd
      min = mid + 1;
      half_len = func4(passwd, min, max) * 2 + 1;
      return half_len;
    }
  } else {// mid > passwd
    max = mid - 1;
    half_len = func4(passwd, min, max) * 2;
    return half_len;
  }
}
```

---

奇怪的判断与二分法使我们暂不能看出此函数的功能，需要继续分析原`phase`函数。

[点此返回到`phase_4`调用此函数处](#p4)。

## fun7

<span id = "fun7">`fun7`函数</span>对应的汇编语句如下：

```assembly
0000000000401204 <fun7>:
  401204:	48 83 ec 08          	sub    $0x8,%rsp
  401208:	48 85 ff             	test   %rdi,%rdi
  40120b:	74 2b                	je     401238 <fun7+0x34>
  40120d:	8b 17                	mov    (%rdi),%edx
  40120f:	39 f2                	cmp    %esi,%edx
  401211:	7e 0d                	jle    401220 <fun7+0x1c>
  401213:	48 8b 7f 08          	mov    0x8(%rdi),%rdi
  401217:	e8 e8 ff ff ff       	callq  401204 <fun7>
  40121c:	01 c0                	add    %eax,%eax
  40121e:	eb 1d                	jmp    40123d <fun7+0x39>
  401220:	b8 00 00 00 00       	mov    $0x0,%eax
  401225:	39 f2                	cmp    %esi,%edx
  401227:	74 14                	je     40123d <fun7+0x39>
  401229:	48 8b 7f 10          	mov    0x10(%rdi),%rdi
  40122d:	e8 d2 ff ff ff       	callq  401204 <fun7>
  401232:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
  401236:	eb 05                	jmp    40123d <fun7+0x39>
  401238:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
  40123d:	48 83 c4 08          	add    $0x8,%rsp
  401241:	c3                   	retq
```

401208和40120b判断`%rdi`（地址`0x6030F0`）是否为0：

- 若为0则跳到401238（401204+0x34）将值`0xffffffff`赋给`%eax`并**返回**
- 若不为0则继续执行40120d将`%rdi`指向的值（地址`0x6030F0`指向的值）赋给`%edx`，40120f和401211比较`%edx`（地址`0x6030F0`指向的值）是否不超过`%esi`（`strtol`函数的返回值）：
  - 若`%edx`不超过`%esi`，则跳到401220（401204+0x1c）将`%eax`赋为0，401225和401227比较`%edx`和`%esi`是否相等：
    - 若相等则跳到40123d（401204+0x39）弹栈**返回**；
    - 若不等则继续执行401229将`%rdi+16`指向的值赋给`%rdi`，40122d递归调用`fun7`函数，401232将`%rax`的值乘2加1赋给`%eax`，401236跳到40123d（401204+0x39）弹栈**返回**
  - 若`%edx`超过`%esi`，则继续执行401213将`%rdi+8`指向的值赋给`%rdi`，401217递归调用`fun7`函数，40121c将返回值乘2，40121e跳到40123d（401204+0x39）弹栈**返回**

可以看出此部分的递归调用等价于：

```c
int fun7(int* p_node, int target) {// %rdi %esi
	if (p_node == 0)// %rdi
    return 0xffffffff;// %eax
  num = *p_node;// (%rdi)->%edx
  if (num <= target) {// %edx <= %esi
    result = 0;// %eax
    if (num == target)
      return result;// %eax=0
    else {// %edx == %esi
    	p_node = *(p_node + 2);// %rdi
    	result = fun7(p_node, target);// %eax
    	return result * 2 + 1;// %eax
    }
  } else {// %edx > %esi
    p_node = *(p_node + 1);// %rdi
    result = fun7(p_node, target);// %eax
    return result * 2;// %eax
  }
}
```

---

奇怪的条件判断与递归调用使我们暂不能看出此函数的功能，需要继续分析原`phase`函数。

[点此返回到`secret_phase`调用此函数处](#backToSec)。

# 拆弹结束

将以上得到的各个`phase`的password存入`passed.txt`中：

```shell
Border relations with Canada have never been better.
1 2 4 8 16 32
0 207
0 0 DrEvil
9/N567
4 3 2 1 6 5
22

```

注意：

1. `phase_3`、`phase_4`、`phase_5`和`secret_phase`的password均有多种情况，请只选择一种输入；
2. 若想进入并拆除`secret_phase`，请在第4行后添加空格和`DrEvil`，并在第6行后添加一行，输入`secret_phase`的password；
3. **文件最后一定要另起一空行，以代表输入结束。**

若password全部正确，则会看到输出为：

```shell
Welcome to my fiendish little bomb. You have 6 phases with
which to blow yourself up. Have a nice day!
Phase 1 defused. How about the next one?
That's number 2.  Keep going!
Halfway there!
So you got that one.  Try this one.
Good work!  On to the next...
Curses, you've found the secret phase!
But finding it and solving it are quite different...
Wow! You've defused the secret stage!
Congratulations! You've defused the bomb!
```

*特别鸣谢：本实验部分思路受[Hakula](https://hakula.xyz/csapp/bomblab.html)启发，在此表示感谢。*
