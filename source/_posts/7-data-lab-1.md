---
url: data-lab-1
title: 位运算实验（上）
date: 2020-04-02 18:34:18
categories: [技术]
tags: [计算机系统实验]
mathjax: true
---
Data Lab: Manipulating Bits #1 - 对于实验一（Data Lab）的一些想法（上）。

<!--more-->

> 这个实验要求学生实现简单的逻辑和算术运算函数，但是只能使用一个非常有限的C语言子集。比如，只能用位级操作来计算一个数字的绝对值。这个实验可帮助学生了解 C语言数据类型的位级表示，以及数据操作的位级行为。

# Puzzle 1 - isAsciiDigit

> isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
> Examples: isAsciiDigit(0x35) = 1
>           isAsciiDigit(0x3a) = 0
>           isAsciiDigit(0x05) = 0
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 15
> Rating: 3

本题应首先判断参数`x`是否介于`0x30`与`0x39`之间，但实验要求不可使用`if`等语句，应该考虑位运算的方法。

我们知道，参数`x`介于`0x30`与`0x39`之间，等价于`x - 0x30`的结果为非负，且`x - 0x3a`的结果为负。从而可以利用正数与负数的符号位的区别进行进一步运算。

同时注意实验要求不可使用减法运算符，可以用“取反加一”的方法代替之。

```
int isAsciiDigit(int x)
{
    int lowCmp = x + ~0x30 + 1;
    int highCmp = x + ~0x3a + 1;
    lowCmp >>= 31;
    highCmp >>= 31;
    return ((!lowCmp) & !!(highCmp));
}
```

# Puzzle 2 - anyEvenBit

> anyEvenBit - return 1 if any even-numbered bit in word set to 1
> Examples: anyEvenBit(0xA) = 0, anyEvenBit(0xE) = 1
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 12
> Rating: 2

本题需要对参数`x`的偶数位进行判断，故可以利用偶数位全为1、奇数位全为0的“模版”辅助判断。

注意实验要求使用的常数应保证在0～255之间，故可以多次左移来将模版扩大为需要的长度。

同时注意要求中的`any even-numbered bit in word set to 1`意为只要存在某一偶数位为1即可，并非要求所有偶数位全部为1。所以其反义为“所有偶数位全为0”。

```
int anyEvenBit(int x)
{
    int a = 0x55;
    int b = (a << 8) + a;
    int c = (b << 8) + a;
    int d = (c << 8) + a;
    return !!(x & d);
}
```

# Puzzle 3 - copyLSB

> copyLSB - set all bits of result to least significant bit of x
> Examples: copyLSB(5) = 0xFFFFFFFF, copyLSB(6) = 0x00000000
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 5
> Rating: 2

本题可以利用算术右移时高位填充与符号位有关的特点来解。

```
int copyLSB(int x)
{
    return x << 31 >> 31;
}
```

# Puzzle 4 - leastBitPos

> leastBitPos - return a mask that marks the position of the least significant 1 bit. If x == 0, return 0
> Example: leastBitPos(0x60) = 0x20
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 6
> Rating: 2

一个数的编码中，“1”所在的位权最小的位的低位全为0，即为“...100...0”的形式（记作`1`）。按位取反，变为“...011...1”的形式（记作`2`），再加1得到“...100...0”的形式（记作`3`）。其中形式`3`中的“1”的高位与形式`2`的对应位相同，即与形式`1`（原数）相反。

形式`3`即原数的相反数，故可以将参数`x`与其相反数按位与，较高位均变为0，只保留了“1”所在位权最小位。

```
int leastBitPos(int x)
{
    int _x = ~x + 1;
    return x & _x;
}
```

# Puzzle 5 - divpwr2

> divpwr2 - Compute x/(2^n), for 0 <= n <= 30
> Round toward zero
> Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 15
> Rating: 2

首先可能想到直接对参数`x`进行右移，但注意右移为算术右移，负数的右移可能会得到错误的结果。

负数除以$2^n$，可以看作它的相反数除以$2^n$后再取相反数。取相反数即“取反加一”，也就是说，当参数`x < 0`时，`x / `$2^n$ 与`~((~x + 1) >> n) + 1`等价。基于此，下面讨论一下此表达式具体的算法。

前提`x < 0`。首先计算`(~x + 1) >> n`，不难看出结果与`~x`的低`n`位有密切联系，而`~x`由`x`按位取反而来，有以下两种情况：
- 当`x`的低`n`位全部为0时，`~x`运算后的低`n`位全部变为1，再加1即“等价于”（对较高位来说）在第`n`位加1（整体加上`1 << n`），故`~x + 1`的**第`n`位至第31位**（认为最低位为第0位，下同）与`~x + (1 << n))`的相同。则`~((~x + 1) >> n) + 1 = ~((~x + (1 << n)) >> n) + 1 = ~(~x >> n + 1) + 1`，此即`~x >> n + 1`的相反数。而`~(~x >> n) + 1`为`~x >> n`的相反数，即满足`~(~x >> n) + 1 + ~x >> n = 0`，故所求`~x >> n + 1`的相反数为`~(~x >> n)`，即`x >> n`。
- 当`x`的低`n`位不全为0时，即至少某一位为1，`~x`运算后该位变成0，其他位为1，再加1后的进位一定进到了最低的0上，而不会影响到较高位。故`~x + 1`的**第`n`位至第31位**与`~x`的相同。则`~((~x + 1) >> n) + 1 = ~(~x >> n) + 1 = (x >> n) + 1`。

可以看出，当`x`为负数时，只有低`n`位不全为0时才需要加1。

```
int divpwr2(int x, int n)
{
    int sig = !!(x >> 31);
    int msk = (1 << n) + ~0;
    int xLowN = msk & x;
    return (x >> n) + ((!!xLowN) & sig);
}
```

# Puzzle 6 - bitCount

> bitCount - returns count of number of 1's in word
> Examples: bitCount(5) = 2, bitCount(7) = 3
> Legal ops: ! ~ & ^ | + << >>
> Max ops: 40
> Rating: 4

查找“1”一共有多少位，首先可能想到按照每一位进行查找，但此时运算符总数会超出上限。所以我们可以把参数`x`划分为8部分，用“模版”`0001 0001 0001 0001 0001 0001 0001 0001`来比较，并比较4次。

获得模版的方法同`Puzzle 2`。

设比较的结果为`sum`，比较完成之后可以发现，我们已经把参数`x`的8部分中各自“1”的总数存放在了`sum`对应的$1/8$部分中，然后将这8部分进行3次“对折”，即可将8部分各自总数加到一起。

```
int bitCount(int x)
{
    int msk, sum;

    msk = 0x11 | (0x11 << 8);
    msk = msk | (msk << 16);

    sum = x & msk;
    sum += (x >> 1) & msk;
    sum += (x >> 2) & msk;
    sum += (x >> 3) & msk;

    msk = 0xff | (0xff << 8);
    sum = (sum >> 16) + (sum & msk);

    msk = 0xf | (0xf << 8);
    sum = ((sum >> 4) & msk) + (sum & msk);

    msk = 0xff;
    sum = (sum & msk) + (sum >> 8);

    return sum;
}
```
# Puzzle ...

同时也要注意代码的规范问题，利用辅助工具可以协助提高代码的质量。

例如，运行`./btest`以检查函数功能的正确性：

![btest](https://i0.hdslb.com/bfs/album/5fea2ce5692cbcdb26e03fa47274be915e228888.png)

运行`./dlc -e bits.c`以检查每个函数的运算符使用情况：

![dlc](https://i0.hdslb.com/bfs/album/7d59b250e41cb8dd1852e18dbad4c70c1953df71.png)

运行`./driver.pl`以同时输出`dlc`和`BDD checker`的结果：

![driver](https://i0.hdslb.com/bfs/album/38e2713cf2330c14aa96e078f86442459f10bf88.png)
