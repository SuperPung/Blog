---
url: data-structures-experiment-4
title: 栈
date: 2020-05-07 09:59:54
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #4 - 设计并实现栈类。

<!--more-->

> 栈可以由顺序表或者链表的方式去实现。通常，栈的操作是固定的，但是在编程过程中有时希望可以由自己控制到底使用链栈还是顺序栈。所以通常使用[**多态**](https://www.cnblogs.com/alinh/p/9636352.html)（[什么是多态？](https://blog.csdn.net/qq_39412582/article/details/81628254)）去编写链栈与顺序栈的代码。
>
> - `void push_back(int data);`
>   将`data`入栈。
> - `int top() const;`
>   询问栈顶元素的值并返回。
> - `void pop();`
>   弹出栈顶元素。

# SeqStack

## 0x01 构造函数

分配`MAX_ELEMENTS` 大小的内存空间。

“栈顶指针”初始化为0。

```cpp
SeqStack::SeqStack(){
    SeqList = new int[MAX_ELEMENTS];
    data_top = 0;
}
```

## 0x02 push_back

判断栈满否。

栈未满则压入栈，“栈顶指针”上移。

```cpp
void SeqStack::push_back(int data){
    if (data_top > MAX_ELEMENTS - 1)
        return;
    else {
        SeqList[data_top] = data;
        data_top++;
    }
}
```

## 0x03 top

判断栈空否。

栈不空则返回“栈顶指针”下一位的数值。

```cpp
int SeqStack::top() const{
    if (data_top == 0)
        return 0;
    else
        return SeqList[data_top - 1];
}
```

## 0x04 pop

判断栈空否。

栈不空则“栈顶指针”下移。

```cpp
void SeqStack::pop(){
    if (data_top == 0)
        return;
    else
        data_top--;
}
```

## 0x05 析构函数

释放内存。

```cpp
SeqStack::~SeqStack(){
    delete[] SeqList;
    SeqList = NULL;
}
```

# LinkStack

## 0x01 构造函数

栈顶指针初始化为空。

```cpp
LinkStack::LinkStack(){
    head = NULL;
    length = 0;
}
```

## 0x02 push_back

链入栈。

```cpp
void LinkStack::push_back(int data){
    StackNode *newNode = new StackNode;
    newNode->value = data;
    newNode->next = head;
    head = newNode;
    length++;
}
```

## 0x03 top

判断栈空否。

栈不空则返回栈顶指针处的数值。

```cpp
int LinkStack::top() const{
    if (length == 0)
        return 0;
    else
        return head->value;
}
```

## 0x04 pop

判断栈空否。

栈不空则将栈顶元素弹出栈并释放内存。

```cpp
void LinkStack::pop(){
    if (length == 0)
        return;
    else {
        StackNode *temp = head;
        head = head->next;
        delete temp;
        temp = NULL;
        length--;
    }
}
```

## 0x05 析构函数

逐结点释放内存。

```cpp
LinkStack::~LinkStack(){
    for (int i = 0; i < length; i++) {
        StackNode *temp;
        temp = head;
        head = head->next;
        delete temp;
        temp = NULL;
    }
    delete head;
    head = NULL;
}
```
