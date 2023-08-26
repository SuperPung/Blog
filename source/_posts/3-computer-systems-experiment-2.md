---
url: computer-systems-experiment-2
title: 计算机系统基础实验二
copyright: true
date: 2020-02-25 14:21:33
categories: [技术]
tags: [计算机系统实验]
---
Computer Systems Experiment #2 - 指针、字节序、移位、数组 reverse

<!--more-->

> - 完善附件的代码：补齐缺失的代码完成以下功能
> - 根据代码的输出结果，判定一下自己的计算机的CPU是大端序还是小端序，说明理由。
> - 完善checkShift()函数中输出结果的代码，理解Logic/Arithmetic 移位操作, 理解超出变量类型bit长度时编译器和CPU的不同行为。总结对常数和变量进行移位的结果。
> - 学习使用inpswap函数，对inplaceReverseArray()函数内容进行完善，检查table1和table2的输出结果是否满足要求。
> - 如果inplaceReverseArray的结果有问题，如何修改才能正确完成数组的反序。
> - 写明自己用编译器版本、操作系统名及版本。
> - 作业提交：最终的源代码，在代码的开头用注释的方式写出以上4项的结果报告。

1. The CPU of my computer is little endian, because the output is
    localVar @ 0061FECC:d5
    localVar @ 0061FECD:dd
    localVar @ 0061FECE:00
    localVar @ 0061FECF:00.
    The least significant byte d5 comes first, and the most one 00 is stored in the end.
2. When the operation is beyond the length of signed varible type,
   the constant will be shifted by logic by compiler, while the varible will be shifted by arithmetic by CPU.
3. In the reverse_array function, the judging condition of FOR-loop shouldnot be first <= last.
   Because when they are equal, the xor result will be 0, thus making the middle element of the array be 0.
4. Compiler: gcc version 8.1.0 (i686-win32-dwarf-rev0, Built by MinGW-W64 project)
    OS: Windows 10 Professional, version 1909 (updated Jan 2020) (x86)

My code is as follows:

```
#include <stdio.h>
//functions prototype declearation
void checkShift();
void inplaceReverseArray();
void reverse_array(int a[], size_t cnt);
int gVar;

int main()
{
        int x1 = 23, y1 = 0xAA;

        //pinter test
        int B = -12345;
        int *p;
        int i;
        p = &B; //pass the address of B to p
        *p = 56789;
        char *pc;
        pc = (char *)&B;

        //check pointer addressed value
        printf("%d = %8.8x\n", p[0], p[0]);
        printf("%d = %x\n", (char)pc[0], pc[0] & 0xFF);

        //check local address & endian
        for (i = 0; i < sizeof(B); i++)
        {
                printf("localVar @ %p:%2.2x\n", pc + i, (unsigned char)*(pc + i));
        }

        //check global address & endian
        pc = (char *)&gVar;
        gVar = 0x12345678;
        for (i = 0; i < sizeof(B); i++)
        {
                printf("gVar @ %p:%2.2x\n", pc + i, (unsigned char)*(pc + i));
        }

        //check code address
        printf("main @ %p\n", main);

        //check lib code address
        printf("printf @ %p\n", printf);

        //check inplace swap
        x1 = 0x23;
        y1 = 0xAA;
        printf("before swap: %x,%x\n", x1, y1);
        inpswap(&x1, &y1);
        printf("after swap: %x,%x\n", x1, y1);
        printf("%d,%d\n", x1, y1);

        //check Logical/arithmetic shift
        checkShift();

        //use inplace swap reverse array element
        inplaceReverseArray();
}

//check shift bits>=variable size
//learn shift
void checkShift()
{
        unsigned uBig = 0x00123456;
        int varBig = (0x00123456 >> 32);

        //signed constant shift by compiler
        printf("signed constant shift by compiler\n");
        varBig = (0x87123456 >> 32);
        printf("0x87123456 >> 32: %d\n", varBig);

        varBig = (0x87123456 >> 40);
        printf("0x87123456 >> 40: %d\n", varBig);

        varBig = (0x87123456 << 32);
        printf("0x87123456 << 32: %d\n", varBig);

        varBig = (0x87123456 << 40);
        printf("0x87123456 << 40: %d\n", varBig);

        //signed varable shift by CPU
        printf("signed varible shift by CPU\n");
        varBig = (0x87123456);
        varBig <<= 32;
        printf("0x87123456 << 32: %d\n", varBig);

        varBig = (0x87123456);
        varBig <<= 40;
        printf("0x87123456 << 40: %d\n", varBig);

        varBig = (0x87123456);
        varBig >>= 32;
        printf("0x87123456 >> 32: %d\n", varBig);

        varBig = (0x87123456);
        varBig >>= 40;
        printf("0x87123456 >> 40: %d\n", varBig);

        //unsigned shift by variable
        printf("unsigned shift by variable\n");
        uBig = 0x87654321;
        uBig <<= 40;
        printf("0x87654321 << 40: %d\n", uBig);

        uBig = 0x87654321;
        uBig <<= 32;
        printf("0x87654321 << 32: %d\n", uBig);

        uBig = 0x87654321;
        uBig >>= 40;
        printf("0x87654321 >> 40: %d\n", uBig);

        uBig = 0x87654321;
        uBig >>= 32;
        printf("0x87654321 >> 32: %d\n", uBig);

        unsigned int varBig2 = (0x123456 << 40); //compiler =>0
        printf("%d\n", varBig);
}

//you can not use other array as intermidiate buffer
//参考 中文ebook  p38练习题2.11，完善代码，找出BUG
//ref  English Book p91 Practice Problem 2.11 complete the code & find bugs
void inplaceReverseArray()
{
        int table1[] = {1, 2, 3, 4, 5};
        int table2[] = {1, 2, 3, 4};

        // table1
        for (int i = 0; i < sizeof(table1) / sizeof(int); i++)
                printf("%d\n", table1[i]);

        reverse_array(table1, sizeof(table1) / sizeof(int));

        for (int i = 0; i < sizeof(table1) / sizeof(int); i++)
                printf("%d\n", table1[i]);

        // table2
        for (int i = 0; i < sizeof(table2) / sizeof(int); i++)
                printf("%d\n", table2[i]);

        reverse_array(table2, sizeof(table2) / sizeof(int));

         for (int i = 0; i < sizeof(table2) / sizeof(int); i++)
                printf("%d\n", table2[i]);
}

//reverseArray without buffer
void reverse_array(int a[], size_t cnt)
{

        size_t first, last;
        for (first = 0, last = cnt - 1; first < last; first++, last--)
        {
                inpswap(&a[first], &a[last]);
        }
}

//swap without buffer
void inpswap(int *x, int *y)
{
        *x = *x ^ *y;
        *y = *x ^ *y;
        *x = *x ^ *y;
}
```

If you have found my mistake, please tell me. Thank you.
