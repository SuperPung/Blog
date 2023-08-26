---
url: data-structures-experiment-7
title: 稀疏矩阵
copyright: true
date: 2020-03-26 11:59:33
categories: [技术]
tags: [数据结构实验]
---
Data Structures Experiment #7 - 实现稀疏矩阵的三元组形式。

<!--more-->

> 通常认为矩阵中非零元素的总数比上矩阵所有元素总数的值小于等于0.05时，则称该矩阵为稀疏矩阵。
>
> 本次实验要求以三元组的形式实现稀疏矩阵。需要SPMartix类的接口有:
>
> - `SPMatrix::SPMatrix(int r, int c);`
>   构造函数，构造一个r行c列的稀疏矩阵。
>
> - `SPMatrix::SPMatrix(int r, int c, int max_element);`
>   构造函数，构造一个r行c列，元素最多为max_element个的稀疏矩阵。
>
> - `int SPMatrix::get(int i, int j);`
>   获取矩阵中i行j列的值。如果没有元素，则返回0
>
> - `void SPMatrix::rotate();`
>   实现稀疏矩阵的转置操作。
>
> - `int SPMatrix::set(int i, int j, int val);`
>   将i行j列的值设置为val。
>
> - `SPMatrix SPMatrix::operator+(const SPMatrix& b);`
>   重载矩阵的加法操作。
>
> - `SPMatrix SPMatrix::operator-(const SPMatrix& b);`
>   重载稀疏矩阵的减法操作。
>
> - `SPMatrix SPMatrix::operator*(const SPMatrix& b);`
>   重载稀疏矩阵中的乘法操作。

# 0x00 数据域封装

对于矩阵来说，其属性值包括总行数和总列数，而稀疏矩阵还应包括非零元素的个数。

稀疏矩阵是用三元组存取的，所以定义三元组每个结点的结构，包括行标、列标和元素数值。最后将每个结点存到三元组中。

```cpp
private:
    int mu, nu, tu;
    Triple* data;
    struct Triple {
        int row, col;
        int value;
    };
```

# 0x01 构造函数

构造一个稀疏矩阵，当给出最大元素个数时，动态分配`max_element`个内存空间，未给出时则分配`r * c`个（有无更好的方法？）。

```cpp
SPMatrix::SPMatrix(int r, int c){
    data = new Triple[r * c];
    mu = r;
    nu = c;
    tu = 0;
}

SPMatrix::SPMatrix(int r, int c, int max_element){
    data = new Triple[max_element];
    mu = r;
    nu = c;
    tu = 0;
}
```

# 0x02 set函数

直接将参数存入三元组，存完后非零元素个数增1。

```cpp
void SPMatrix::set(int i, int j, int val){
    data[tu].row = i;
    data[tu].col = j;
    data[tu].value = val;
    tu++;
}
```

# 0x03 get函数

将三元组从头遍历，找对应行和列的元素值，未找到则返回0。

```cpp
int SPMatrix::get(int i, int j)const{
    for (int k = 0; k < tu; k++) {
        if (data[k].row == i && data[k].col == j)
            return data[k].value;
    }
    return 0;
}
```

# 0x04 转置函数

首先将矩阵的型变换，即行数变列数、列数变行数。

其次将三元组的每个结点行标变列标、列标变行标。

> 注：稀疏矩阵的转置还需对三元组重新排序，由于本算法在读取时并未对其排序，故此处从略。“一个稀疏矩阵A<sub>m\*n</sub>采用三元组形式表示，若把三元组中有关行下标与列下标的值互换，并把m和n的值互换，则就完成了A<sub>m\*n</sub>的转置运算。”这一说法是错误的。

```cpp
void SPMatrix::rotate(){
    int temp = mu;
    mu = nu;
    nu = temp;

    for (int k = 0; k < tu; k++) {
        int temp = data[k].row;
        data[k].row = data[k].col;
        data[k].col = temp;
    }
}
```

# 0x05 重载加法运算符

另构造一和矩阵来存储两矩阵之和。

两矩阵可加的前提是两矩阵同型，应该先作判断。

矩阵的加法即对应位置元素相加，将矩阵视为数组以便于运算。

两矩阵对应位置不一定都存在元素，即数组的`loc`不一定相同，应分别判断。

每次加完后，和矩阵的非零元素数增1。

最后考虑某一个矩阵的非零元素已经全部遍历完成，此时将另一矩阵剩余元素加到和矩阵里。

```cpp
SPMatrix SPMatrix::operator+(const SPMatrix& b){
    SPMatrix add(mu, nu);
    if (b.mu != mu || b.nu != nu)
        return add;
    add.mu = mu;
    add.nu = nu;
    add.tu = 0;

    int i = 0, j = 0, aLoc, bLoc;
    while (i < tu && j < b.tu) {
        aLoc = data[i].row * nu + data[i].col;
        bLoc = b.data[j].row * b.nu + b.data[j].col;

        if (aLoc < bLoc) {
            add.data[add.tu].row = data[i].row;
            add.data[add.tu].col = data[i].col;
            add.data[add.tu].value = data[i].value;
            i++;
        }

        if (aLoc > bLoc) {
            add.data[add.tu].row = b.data[j].row;
            add.data[add.tu].col = b.data[j].col;
            add.data[add.tu].value = b.data[j].value;
            j++;
        }

        if (aLoc == bLoc) {
            if (data[i].value + b.data[j].value) {
                add.data[add.tu].row = data[i].row;
                add.data[add.tu].col = data[i].col;
                add.data[add.tu].value = data[i].value + b.data[j].value;
                i++;
                j++;
            }
        }

        add.tu++;
    }

    while (i < tu) {
        add.data[add.tu].row = data[i].row;
        add.data[add.tu].col = data[i].col;
        add.data[add.tu].value = data[i].value;
        i++;
        add.tu++;
    }
    while (j < b.tu) {
        add.data[add.tu].row = b.data[j].row;
        add.data[add.tu].col = b.data[j].col;
        add.data[add.tu].value = b.data[j].value;
        j++;
        add.tu++;
    }

    return add;
}
```

# 0x06 重载减法运算符

与加法类似。

# 0x07 重载乘法运算符

另构造一积矩阵来存储两矩阵之积。

两矩阵可乘的前提是左矩阵的列数等于右矩阵的行数，应该先作判断。

矩阵的乘法即左矩阵的行与右矩阵的列对应相乘再相加，将右矩阵视为数组以便于运算。

以左矩阵为基准，将右矩阵的各个属性分别存储以便于进一步运算，即重排。包括右矩阵的每行非零元素个数以及每行第一个非零元素在数组中对应的位置。

按左矩阵的行进行运算，每计算完一行将结果存入一个数组中，再赋值给积矩阵。

最后释放`new`所分配的内存。

```cpp
SPMatrix SPMatrix::operator*(const SPMatrix& b){
    SPMatrix mul(mu, nu);
    if (b.mu != nu)
        return mul;

    int* tuOfEachBRow = new int[b.mu];
    int* firstIndexOfEachBRow = new int[b.mu + 1];
    int* rowPOfResult = new int[b.nu];
    int aLoc, aRowLoc, aColLoc, bColLoc;

    for (int i = 0; i < b.mu; i++)
        tuOfEachBRow[i] = 0;
    for (int i = 0; i < b.tu; i++)
        tuOfEachBRow[b.data[i].row]++;
    firstIndexOfEachBRow[0] = 0;
    for (int i = 1; i < b.mu; i++)
        firstIndexOfEachBRow[i] = firstIndexOfEachBRow[i - 1] + tuOfEachBRow[i - 1];

    aLoc = 0;
    mul.tu = -1;
    while (aLoc < tu) {
        aRowLoc = data[aLoc].row;
        for (int i = 0; i < b.nu; i++)
            rowPOfResult[i] = 0;
        while (aLoc < tu && data[aLoc].row == aRowLoc) {
            aColLoc = data[aLoc].col;
            for (int i = firstIndexOfEachBRow[aColLoc]; i < firstIndexOfEachBRow[aColLoc + 1]; i++) {
                bColLoc = b.data[i].col;
                rowPOfResult[bColLoc] += data[aLoc].value * b.data[i].value;
            }
            aLoc++;
        }
        for (int i = 0; i < b.nu; i++) {
            if (rowPOfResult[i]) {
                mul.tu++;
                mul.data[mul.tu].row = aRowLoc;
                mul.data[mul.tu].value = rowPOfResult[i];
                mul.data[mul.tu].col = i;
            }
        }
    }

    mul.mu = mu;
    mul.nu = b.nu;
    mul.tu++;

    delete[] tuOfEachBRow;
    delete[] firstIndexOfEachBRow;
    delete[] rowPOfResult;
    return mul;
}
```

# 0x08 析构函数

释放内存。

```cpp
SPMatrix::~SPMatrix(){
    delete[] data;
}
```

# 0x...

如果你发现了我的错误或者有更好的解决方案，欢迎一起交流。
