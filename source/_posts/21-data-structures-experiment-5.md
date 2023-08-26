---
url: data-structures-experiment-5
title: 队列
date: 2020-05-07 09:59:59
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #5 - 设计并实现队列类。

<!--more-->

> - `MyQueue();`
>  构造函数。
> - `~MyQueue();`
>   析构函数。
> - `void push_back(int data);`
>   将数据data插入到队列的队尾。
> - `void pop_front();`
>   队首元素出队。
> - `int front();`
>   询问队首元素的值。

# SeqQueue

## 0x00 数据域封装

“队尾指针”、“队首指针”、队列和最大元素个数。

```cpp
private:
    int rear, fore;
    int* SeqList;
    const static int MAX_ELEMENTS = 100;
```

## 0x01 构造函数

分配`MAX_ELEMENTS`大小的内存空间。

“队尾指针”和“队首指针”均初始化为0。

```cpp
SeqQueue::SeqQueue(){
    SeqList = new int[MAX_ELEMENTS];
    rear = fore = 0;
}
```

## 0x02 push_back

判断队列满否。

队列不满则在队尾入队。

```cpp
void SeqQueue::push_back(int data){
    if (rear > MAX_ELEMENTS - 1)
        return;
    SeqList[rear] = data;
    rear++;
}
```

## 0x03 pop_front

判断队列空否。

队列不空则在队首出队。

```cpp
void SeqQueue::pop_front(){
    if (rear == 0)
        return;
    fore++;
}
```

## 0x04 front

判断队列空否。

队列不空则返回队首元素。

```cpp
int SeqQueue::front() const{
    if (rear == 0)
        return 0;
    return SeqList[fore];
}
```

## 0x05 析构函数

释放内存。

```cpp
SeqQueue::~SeqQueue(){
    delete[] SeqList;
}
```

# LinkQueue

## 0x00 数据域封装

定义队列结点结构体（数据域和指针域）、队首指针和队尾指针。

```cpp
private:
    struct QueueNode{
        QueueNode* next;
        int value;
    };
    QueueNode* fore;
    QueueNode* rear;
```

## 0x01 构造函数

队首指针和队尾指针均初始化为空。

```cpp
LinkQueue::LinkQueue(){
    fore = rear = NULL;
}
```

## 0x02 push_back

分情况：

- 若队列为空则直接链入队；
- 若队列不为空则在队尾链入队。

```cpp
void LinkQueue::push_back(int data){
    QueueNode *newNode = new QueueNode;
    newNode->value = data;
    newNode->next = NULL;

    if (fore == NULL) {
        fore = rear = newNode;
    } else {
        rear->next = newNode;
        rear = newNode;
    }
}
```

## 0x03 pop_front

判断队列空否。

队列不空则队首指针后移并释放内存。

```cpp
void LinkQueue::pop_front(){
    if (rear == NULL)
        return;
    QueueNode *temp = fore;
    fore =  fore->next;
    delete temp;
    temp = NULL;
}
```

## 0x04 front

返回队首处的值。

```cpp
int LinkQueue::front() const{
    return fore->value;
}
```

## 0x05 析构函数

逐结点释放内存（**同链表**）。

```cpp
LinkQueue::~LinkQueue(){
    while (fore != NULL) {
        QueueNode* temp = fore;
        fore = fore->next;
        delete temp;
        temp = NULL;
    }
    rear = NULL;
}
```
