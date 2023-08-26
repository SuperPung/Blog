---
url: data-structures-experiment-2
title: 链表
date: 2020-05-07 09:59:46
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #2 - 使用给出的模板，实现链表的基本操作。

<!--more-->

> - `LinkList();`
>   构造函数，构造一个`LinkList`，初始化内部的成员变量。
>
> - `void insert(const int& data, const int& location);`
>   将`data`插入`location`中。其中`location`的计数从`0`开始，并且测试时会保证`location`小于等于元素的个数。
>
> - `int length();`
>   返回链表中元素的个数。
>
> - bool remove(const int& location);
>   删除位置为`location`的元素，如果`location`不合法，则返回`false`。删除成功则返回`true`。
>
> - `int getData(const int& location);`
>   返回位置为`location`的元素。
>
> - `void converse();`
>   要求可以对链表实现翻转
>   例如：
>
>   - 链表中内容为 1 2 3 4 5
>   - 倒置后为        5 4 3 2 1
>
> - `void append(LinkList& append_list);`
>   要求可以将`append_list`扩充到链表中，需要对`append_list`中的元素实现深拷贝。
>
>   例如：
>
>   - 原始的list : 1 2 3     append_list:  2 3 4
>   - 调用`append(append_list)`后: 1 2 3 2 3 4
>
> - `～LinkList();`
>   析构函数。

# 0x01 构造函数

将头指针赋为空。

```cpp
LinkList::LinkList(){
    head = NULL;
    len = 0;
}
```

# 0x02 length

直接返回长度。

```cpp
int LinkList::length()
{
    return len;
}
```

# 0x03 getData

沿链表遍历到对应位置。

```cpp
int LinkList::getData(const int &location)
{
    node *p = head;
    for (int i = 0; i < location; i++)
        p = p->next;
    return p->value;
}
```

# 0x04 insert

尾插法分情况：

- 当表为空时，直接插入到`head`处；

- 当表不空时，指针沿链表移动到前一个位置，断开并重新链接。

长度加1。

```cpp
void LinkList::insert(const int& data, const int& location){
    node *newNode = (node*) malloc(sizeof(node));
    newNode->value = data;
    if (head == NULL || location == 0) {
        newNode->next = head;
        head = newNode;
        len++;
    } else {
    node *p = head;
    for (int i = 0; i < location - 1; i++)
        p = p->next;
    newNode->next = p->next;
    p->next = newNode;
    len++;
    }
}
```

# 0x05 remove

分情况：

- 当删除头结点时，直接**释放头结点内存**，头指针后移；
- 当删除结点不为头结点时，判断位置合法性后，指针移动到前一个位置，断开并重新链接，**释放内存**。

长度减1。

```cpp
bool LinkList::remove(const int& location){
    if (location == 0) {
        node *temp = head;
        head = head->next;

        delete temp;
        temp = NULL;
        len--;
        return true;
    }
    else
    {
        node *p = head;
        for (int i = 0; i < location - 1; i++)
            p = p->next;
        if (p == NULL || p->next == NULL)
            return false;
        node *temp = p->next;
        p->next = temp->next;
        len--;
        free(temp);
        return true;
    }
}
```

# 0x06 converse

不改变指针域的情况下，交换数值。

```cpp
void LinkList::converse(){
    for (int i = 0; i < len / 2; i++) {
        node *p = head;
        for (int j = 0; j < i; j++)
            p = p->next;
        node *q = head;
        for (int j = 0; j < len - i - 1; j++)
            q = q->next;
        int temp = p->value;
        p->value = q->value;
        q->value = temp;
    }
}
```

# 0x07 append

在表尾链接新表，长度增加。

```cpp
void LinkList::append(const LinkList& append_list){
    node *p = head;
    for (int i = 0; i < len - 1; i++)
        p = p->next;
    p->next = append_list.head;
    len += append_list.len;
}
```

# 0x08 析构函数

**逐结点**释放内存。

```cpp
LinkList::~LinkList(){
    for (int i = 0; i < len; i++) {
        node *temp;
        temp = head;
        head = head->next;
        delete temp;
    }
    delete head;
}
```
