---
url: data-lab-2
title: 位运算实验（下）
date: 2020-04-11 09:44:20
categories: [技术]
tags: [计算机系统实验]
mathjax: true
---
Data Lab: Manipulating Bits #2 - 对于实验一（Data Lab）的一些想法（下）。

<!--more-->

> 这个实验要求学生实现简单的逻辑和算术运算函数，但是只能使用一个非常有限的C语言子集。比如，只能用位级操作来计算一个数字的绝对值。这个实验可帮助学生了解 C语言数据类型的位级表示，以及数据操作的位级行为。

# Puzzle 1 - conditional

> conditional - same as x ? y : z
> Example: conditional(2,4,5) = 4
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 16
> Rating: 3

利用位操作来实现三目运算符表达式`x ? y : z`的功能，只需判断`x`是否为`0`：若不为`0`则返回`y`；反之则返回`z`。

同样可以利用算术右移的特点制作一个**掩码（mask）**，类似[#1](/data-lab-1)中提到的“模版”，以便于操作。

```
int conditional(int x, int y, int z)
{
    int msk = (!!x) << 31 >> 31;
    return (msk & y) | (~msk & z);
}
```

# Puzzle 2 - isNonNegative

> isNonNegative - return 1 if x >= 0, return 0 otherwise
> Example: isNonNegative(-1) = 0.  isNonNegative(0) = 1.
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 6
> Rating: 3

判断符号位即可。

```
int isNonNegative(int x)
{
    return !(x >> 31);
}
```

# Puzzle 3 - isGreater

> isGreater - if x > y  then return 1, else return 0
> Example: isGreater(4,5) = 0, isGreater(5,4) = 1
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 24
>  Rating: 3

比较大小，显然当`x`为正且`y`为负时，一定有`x > y`成立；当`x`为负且`y`为正时，上式一定不成立。

当`x`与`y`同号时，判断二者之差是否为正即可。注意题目要求，当二者相等时返回`0`，故此处作差时减`1`。

判断正负依然利用符号位。

```
int isGreater(int x, int y)
{
    int _x, _y, sub;
    _x = x >> 31;
    _y = y >> 31;
    sub = (x + ~y) >> 31;
    return (!_x & _y) | (!(_x ^ _y) & !sub);
}
```

# Puzzle 4 - absVal

> absVal - absolute value of x
> Example: absVal(-1) = 1.
> You may assume -TMax <= x <= TMax
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 10
> Rating: 4

正数的绝对值为其本身，负数的绝对值为其相反数。

位全0取反为-1（`~0 + 1 = 0`），全1取反为0。

依然可以利用符号位右移作为掩码，当`x`为正时掩码全0，异或后仍为原值；当`x`为负时掩码全1，异或后相当于取反的效果，再加1即得到其相反数。

```
int absVal(int x)
{
    int _x = x >> 31;
    return (x ^ _x) + (~_x + 1);
}
```

# Puzzle 5 - isPower2

> isPower2 - returns 1 if x is a power of 2, and 0 otherwise
> Examples: isPower2(5) = 0, isPower2(8) = 1, isPower2(0) = 0
> Note that no negative number is a power of 2.
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 20
> Rating: 4

一个数为2的幂次，即其二进制位有且仅有1位为1。

不妨设其中1所在最低位为第`n`位，则减1后其`0～n-1`位均变为1，而第`n`位变为0——`0～n`为与原值相反，`n`位以上与原值相同。此时将减1后的值与原值取与，可知取与结果的低`n+1`位必为0。若原值在`n`位以上仍存在1，则取与结果必不为0，由上段推论知，此时原值必不为2的幂次。若原值在`n`位以上全0，则取与结果必为0，此时原值必为2的幂次。

显然非正值必不为2的幂次。

```
int isPower2(int x)
{
    int msk = x + ~0;
    return !(x & msk) & !(x >> 31) & !!x;
}
```

# Puzzle 6 - float_neg

> float_neg - Return bit-level equivalent of expression -f for floating point argument f. Both the argument and result are passed as unsigned int's, but they are to be interpreted as the bit-level representations of single-precision floating point values.
> When argument is NaN, return argument.
> Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
> Max ops: 10
> Rating: 2

本题需要对浮点数的构成有基本的了解，鉴于下题需要对浮点数有极为全面的认识，此处建议提前阅读[IEEE754与单精度浮点数](/ieee754-float)，注意文章中对`NaN`的定义并思考相反数的求法。

0x7fffffff = 0111 1111 1111 1111 1111 1111 1111 1111
0x7f800000 = 0111 1111 1000 0000 0000 0000 0000 0000
0x80000000 = 1000 0000 0000 0000 0000 0000 0000 0000

本题只需判断参数`uf`是否为`NaN`，是则返回其本身，否则返回其相反数。

制作低`31`位全1的掩码，同原值取与以保留参数`uf`除去符号位的值，取与结果同0x7f800000相比可判断原值是否为`NaN`。

求相反数只需变换符号位，同掩码0x80000000异或即可。

```
unsigned float_neg(unsigned uf)
{
    int msk = 0x7fffffff;
    unsigned low = uf & msk;
    if (low > 0x7f800000)
        return uf;
    else {
        msk = 0x80000000;
        return uf ^ msk;
    }
}
```

# Puzzle 7 - float_i2f

> float_i2f - Return bit-level equivalent of expression (float) x
> Result is returned as unsigned int, but it is to be interpreted as the bit-level representation of a single-precision floating point values.
> Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
> Max ops: 30
> Rating: 4

本题需将`int`型参数`x`转化为单精度浮点数`float`型数值，即实现类型转换`(float)x`的功能。如果你对浮点数没有基本的了解，可能会认为`int`型`x`是整数，转化为`float`型就不存在小数了。为避免类似一些误解，需要对浮点数有极为全面的认识，此处建议<span id = "IEEE754">提前阅读</span>[IEEE754与单精度浮点数](/ieee754-float)。

整数转化为浮点数，首先将整数的二进制编码转化为科学计数法$m\times 2^{e}$的形式，其中$m$的小数部分便对应浮点数的`fraction`位，$e$对应浮点数的`exponent`位，符号位互相对应。例如$78=1001110_{2}=1.00111\times 2^{6}\_{2}$，故`sign`位为0，`exponent`位为$6+127=133=100 00101_{2}$，`fraction`位为$001 1100 0000 0000 0000 0000$。

由于浮点数由`sign`、`exponent`、`fraction`三部分组成，可以分三部分来进行转换。

## Declare Variables

`sig`、`exp`、`frc`分别代表浮点数的三部分；`frcMsk`为`fraction`位全1的掩码，用于存放`fraction`位；`eBit`指向`x`中1的最高位。

```
int sig = (x >> 31) & 1;
int exp, frc;
int frcMsk = 0x7fffff;
int eBit = 1;
```

## Judge Special Value

特殊值只需返回特定值。

当`x`的值为0x80000000时，即$-2^{31}$，是有符号`int`型最小值。为避免操作可能导致的溢出错误，此处直接返回其`float`值，即`exponent`部分为$31+127=158$，其他位均为0。

```
if (x == 0)
    return x;
else if (x == 0x80000000)
    return 0xcf000000;
```

## Judge Sign

负数转化为其相反数进行运算。

```
if (sig)
    x = -x;
```

## Get Exponent Bits

当右移`eBit`位后为0，而右移`eBit-1`位不为0时，说明`eBit-1`位1所在最高位。例如$78=1001110_{2}$，右移`6`位不为0，而右移`7`位为0，故1所在最高位为第`6`位。

```
while (x >> eBit)
    eBit++;
exp = --eBit + 127;
```

## Get Fraction Bits

左移`31-eBit`位，将1所在最高位移动到第`31`位。当`x`为$78=1001110_{2}$时，移动后即$$78=1001 1100 0000 0000 0000 0000 0000 0000_{2}.$$再右移到fraction的位最高位后$$0000 0000 1001 1100 0000 0000 0000 0000$$同掩码$$0000 0000 0111 1111 1111 1111 1111 1111$$取与，即可得到`fraction`位。

至于为什么将最高位的1舍弃，请[返回](#IEEE754)并继续阅读。

```
x = x << (31 - eBit);
frc = (x >> 8) & frcMsk;
```

## Round to Even

上面的`fraction`是将`x`右移`8`位后取得的，应该考虑被移出去的`8`位是否满足进位的条件，若符合应该进行进位。

先取`x`的低`8`位，若大于一半或者等于一半，应该进位。注意此处进位为向**偶数**进位，即若等于一半且被进位的位置为0时不进位。

进位之后`fraction`位会出现变化，若超出`23`位长，应向`exponent`进位，且进位后`fraction`与掩码取与重置。

```
x &= 0xff;
if ((x > 0x80) || ((x == 0x80) && (frc & 1)))
    frc++;
if (frc >> 23) {
    exp++;
    frc &= frcMsk;
}
```

## Return Result

将各部分左移到相应位置取或即可。

```
return (sig << 31) | (exp << 23) | frc;
```

## Source Code

```
unsigned float_i2f(int x)
{
    int sig = (x >> 31) & 1;
    int exp, frc;
    int frcMsk = 0x7fffff;
    int eBit = 1;

    if (x == 0)
        return x;
    else if (x == 0x80000000)
        return 0xcf000000;
    else {
        if (sig)
            x = -x;

        while (x >> eBit)
            eBit++;
        exp = --eBit + 127;

        x = x << (31 - eBit);
        frc = (x >> 8) & frcMsk;
        x &= 0xff;
        if ((x > 0x80) || ((x == 0x80) && (frc & 1)))
            frc++;
        if (frc >> 23) {
            exp++;
            frc &= frcMsk;
        }
    }

    return (sig << 31) | (exp << 23) | frc;
}
```

# Puzzle ...

同时也要注意代码的规范问题，利用辅助工具可以协助提高代码的质量。

例如，运行`./btest`以检查函数功能的正确性：
![btest2](https://i0.hdslb.com/bfs/album/9b3671230c974cb7078fd70860e2105c709c9e76.png)
运行`./dlc -e bits.c`以检查每个函数的运算符使用情况：
![dlc2](https://i0.hdslb.com/bfs/album/a571177e516ab110fe055a9e119faa7e77747810.png)
运行`./driver.pl`以同时输出`dlc`和`BDD checker`的结果：
![driver2](https://i0.hdslb.com/bfs/album/ae5cacdfb06c686fbd9b4c88d560b3105634a902.png)
