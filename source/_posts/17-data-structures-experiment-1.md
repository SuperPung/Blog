---
url: data-structures-experiment-1
title: 顺序表
date: 2020-05-07 09:59:41
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #1 - 搭建实验环境，并实现顺序表的基本操作

<!--more-->

> 实现一个SeqList类。
>
> - `SeqList(const int& listSize);`
>   构造函数，构建一个`SeqList`，其中`listSize`为顺序表最多存放元素的个数。
> - `bool isIn(const int& data);`
>   判断`data`是否在顺序表中。如果`data`在顺序表中，返回`true`，否则返回`false`。
> - `int length();`
>   返回顺序中加入元素的个数。
> - `int getData(const int& location);`
>   返回位置为`location`的元素。
> - `bool insert(const int& data, const int& location);`
>   将`data`插入`location`中，其中如果顺序表已经满了，则返回`false`。其中`location`的计数从`0`开始，并且测试时会保证`location` 小于等于元素的个数。
> - `bool remove(const int& location);`
>   删除位置为`location`的元素，如果`location`不合法，则返回`false`。删除成功则返回`true`。
> - `～SeqList();`
>   析构函数。

# 0x01 构造函数

直接分配`LIST_SIZE`大小的空间。

```cpp
SeqList::SeqList(const int& listSize) :LIST_SIZE(listSize)
{
    seq = new int[LIST_SIZE];
    // equal to:  seq = (int*) malloc(sizeof(int) * LIST_SIZE);
    len = 0;
}
```

# 0x02 isIn

默认为元素不在表中，顺序遍历即可。

```cpp
bool SeqList::isIn(const int& data){
    bool defaultOut = false;
    for (int i = 0; i < len; i++)
    {
        if (data == seq[i])
        {
            defaultOut = true;
            break;
        }
    }
        if (defaultOut)
            return true;
        else
            return false;
}
```

# 0x03 length

直接返回长度。

```cpp
int SeqList::length(){
    return len;
}
```

# 0x04 getData

返回对应位置的数值。

```cpp
int SeqList::getData(const int& location){
    return seq[location];
}
```

# 0x05 insert

将该位置后元素整体后移，插入即可。

```cpp
bool SeqList::insert(const int& data, const int& location){
    if (len == LIST_SIZE)
        return false;
    else
    {
        for (int i = len - 1; i >= location; i--)
            seq[i + 1] = seq[i];
        seq[location] = data;
        len++;
        return true;
    }
}
```

# 0x06 remove

首先判断位置的合法性。

将该位置后元素整体前移，删除即可。

```cpp
bool SeqList::remove(const int& location){
    if (location >= len || location < 0 || len == 0)
        return false;
    else
    {
        for (int i = location; i < len - 1; i++)
            seq[i] = seq[i + 1];
        len--;
        return true;
    }
}
```

# 0x07 析构函数

释放内存。

```cpp
SeqList::~SeqList(){
    delete[] seq;
}
```
