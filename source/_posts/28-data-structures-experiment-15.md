---
url: data-structures-experiment-15
title: 哈希
date: 2020-05-20 19:28:46
categories: [技术]
tags: [数据结构实验]
mathjax: true
---

Data Structures Experiment #15 - 使用哈希算法完成`MyHash`类，实现简单的哈希类。

<!--more-->

> 添加`private`属性之后，完善相应的`public`函数。
> 自选哈希函数，自选解决哈希碰撞的方法。
>
> - `MyHash(int max_element);`
>   构造函数，构造一个元素最多为`max_element`的哈希表。
> - `~MyHash();`
>   析构函数。
> - `void setvalue(int key, int value);`
>   将哈希表中键值为`key`的元素设定值改为`value`。（哈希表中没有被`setvalue`的键值默认值为`0`）
> - `int getvalue(int key);`
>   获得哈希表中键值为`key`的元素的对应`value`值。

[什么是哈希表？](https://zh.wikipedia.org/wiki/哈希表)

此处选取哈希函数的构造方法为“除留余数法”，解决哈希碰撞的方法为“链地址法”。

众所周知，数组查找容易，但插入和删除困难；链表查找困难，但插入和删除容易。哈希表作为二者的结合，查找容易，插入和删除也容易。

创建哈希表为一个数组。为加快检索，将`key`值对某一固定值`p`取余后余数相同的归为一类（即数论中的“同余类/剩余类”），以链表的形式存放到数组中对应第“余数”个元素处，构成“完全剩余系”。

# 0x00 数据域封装

将数组的每个元素加上一个头指针，用于指向链表头。链表每个结点包括`key`值和`value`值。

```cpp
private:
    struct node {
        int value;
        int key;
        node* next;
    };
    struct table {
        node* head;
    };

    table* HashTable;
```

# 0x01 构造函数

根据最大元素数为1000，可以定义固定的模`p`为997（小于1000的最大素数）。

若取模为合数，即存在小于自身的因子，则每个含有此因子的`key`值都会映射到数组的相同元素，从而造成局部链表加长，这显然不利于我们快速查找。

由于取模为997，则所有`key`值映射后的范围是0～996，所以数组大小取`p-1`。

```cpp
#define p 997

MyHash::MyHash(int max_element){
    HashTable = new table[p - 1];
    for (int i = 0; i < p - 1; i++)
        HashTable[i].head = NULL;
}
```

# 0x02 setValue

插入链表，类似栈的LIFO存储方式。

```cpp
void MyHash::setvalue(int key, int value){
    node* newNode = new node;
    newNode->key = key;
    newNode->value = value;
    newNode->next = HashTable[key % p].head;
    HashTable[key % p].head = newNode;
}
```

# 0x03 getValue

先确定余数的值，即其所在链表位于数组中的位置，然后遍历链表查找。

注意“哈希表中没有被`setvalue`的键值默认值为`0`”。

```cpp
int MyHash::getvalue(int key){
    node* temp = HashTable[key % p].head;
    while (temp) {
        if (temp->key == key)
            return temp->value;
        temp = temp->next;
    }
    return 0;
}
```

# 0x04 析构函数

逐个链表释放内存。

```cpp
MyHash::~MyHash(){
    for (int i = 0; i < p - 1; i++) {
        node* temp = HashTable[i].head;
        while (temp) {
            node* q = temp;
            temp = temp->next;
            delete q;
        }
    }
    delete[] HashTable;
}
```
