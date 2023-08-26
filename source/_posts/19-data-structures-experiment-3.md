---
url: data-structures-experiment-3
title: 双向链表
date: 2020-05-07 09:59:50
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #3 - 双向链表的实现。

<!--more-->

> - `DBLinkList::DBLinkList();`
>   构造函数，需要在类中初始化相应的成员变量。
> - `DBLinkList::~DBLinkList();`
>   析构函数，需要将所有动态申请的内存都释放。
> - `void DBLinkList::insert(int data, int location);`
>   在`location`位置插入`data`，测试程序保证`data`合法。
> - `void DBLinkList::remove(int location);`
>   将`location`的元素移除，测试程序保证`location`的合法性。
> - `int DBLinkList::length();`
>   获取双向链表的长度。
> - `int DBLinkList::getData(int location);`
>   获取列表中`lcation`位置的元素，测试数据保证`location`的合法。
> - `void DBLinkList::bubbleSort();`
>   将双向链表进行冒泡排序。

# 0x01 构造函数

将头、尾指针均赋为空。

```cpp
DBLinkList::DBLinkList(){
    head = NULL;
    tail = NULL;
    len = 0;
}
```

# 0x02 insert

判断位置的合法性。

分情况：

- 在表头插入，判断是否为空表；
- 在表中插入，指针沿链表移动到前一个位置，断开并重新链接；
- 在表尾插入，利用尾指针链接。

```cpp
void DBLinkList::insert(int data, int location){
    if (location < 0 || location > len)
        return;

    DBNode *newNode = new DBNode;
    newNode->value = data;
    if (location == 0) {
        newNode->nxt = head;
        newNode->pre = NULL;
        head = newNode;
        if (tail == NULL) {
            tail = newNode;
            newNode->nxt = NULL;
        } else
            newNode->nxt->pre = newNode;
    }

    else if (location != len) {
    DBNode *p = head;
    for (int i = 0; i < location - 1; i++)
        p = p->nxt;
    p->nxt->pre = newNode;
    newNode->nxt = p->nxt;
    p->nxt = newNode;
    newNode->pre = p;
    }

    else if (location == len) {
        tail->nxt = newNode;
        newNode->pre = tail;
        tail = newNode;
        newNode->nxt = NULL;
    }

    len++;
}
```

# 0x03 remove

同上。

```cpp
void DBLinkList::remove(int location){
    if (location == 0) {
        if (len == 1) {
            DBNode *temp = head;
            head = NULL;
            tail = NULL;

            delete temp;
            temp = NULL;
        }
        else
        {
            head->nxt->pre = NULL;
            DBNode *temp = head;
            head = head->nxt;

            delete temp;
            temp = NULL;
        }
    }

    else if (location != len - 1) {
        DBNode *p = head;
        for (int i = 0; i < location; i++)
            p = p->nxt;
        p->pre->nxt = p->nxt;
        p->nxt->pre = p->pre;

        delete p;
        p = NULL;
    }

    else if (location == len - 1) {
        DBNode *temp = tail;
        tail = tail->pre;
        tail->nxt = NULL;

        delete temp;
        temp = NULL;
    }

    len--;
}
```

# 0x04 length

直接返回长度。

```cpp
int DBLinkList::length(){
    return len;
}
```

# 0x05 getData

指针沿链表移动到该位置，返回其数值。

```cpp
int DBLinkList::getData(int location){
    DBNode *p = head;
    for (int i = 0; i < location; i++)
        p = p->nxt;
    return p->value;
}
```

# 0x06 bubbleSort

冒泡排序，未改变指针，仅交换数值。

```cpp
void DBLinkList::bubbleSort(){
    for (int i = 0; i < len - 1; i++) {
        DBNode *p = head;
        for (int j = 0; j < len - i - 1; j++)
        {
            p = p->nxt;
            if (p->pre->value > p->value) {
                int temp = p->pre->value;
                p->pre->value = p->value;
                p->value = temp;
            }
        }
    }
}
```

# 0x07 析构函数

逐结点释放内存。

```cpp
DBLinkList::~DBLinkList(){
    for (int i = 0; i < len; i++) {
        DBNode *temp;
        temp = head;
        head = head->nxt;
        delete temp;
    }
    delete head;
    delete tail;
}
```
