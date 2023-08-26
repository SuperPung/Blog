---
url: data-structures-experiment-8
title: 二叉树
date: 2020-05-07 10:00:07
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #8 - 实现一个二叉树类。

<!--more-->

> -  `BinTree();`
>   构造函数，构造一个空的二叉排序树。
> - ` ~BinTree();`
>   析构函数，将二叉排序树销毁。
> - `void insert(int val, int parent, int flg);`
>   将值为`val`的元素插入到二叉树中。此节点的父节点编号为`parent`，如果`parent`项为0，表示插入元素为根结点。如果`flg`为-1，则此节点为父节点的左子，如果`flg`为1，则此节点为父节点的右子。
>   二叉树的节点编号按照插入顺序逐一递增，编号从1开始。
>   例如：
>   - `tree.insert(10, 0, 0);` 插入编号为1的根结点；
>   - `tree.insert(20, 1, 1);` 插入编号为2的节点，节点是根结点的右子。
> - `int* p_traversal() const;`
>   前序遍历二叉树，并将结果保存到int*中返回。*
> - `int* m_traversal() const;`
>   中序遍历二叉树，并将结果保存到int*中返回
> - `int height() const;`
>   获得二叉树高度。
> - `int countNode() const;`
>   获得二叉树的节点总个数。

# 0x00 数据域封装

定义二叉树结点结构体，包括数值、序号、左子树和右子树。

创建二叉树，定义计数器。

定义一些后续会用到的递归函数。

```cpp
private:
    struct BinNode {
      int data, number;
      BinNode* lChild;
      BinNode* rChild;
    };

    BinNode* root;
    int count;

    void destroyTree(BinNode* Tree);
    BinNode* search_number(BinNode* Tree, int num) const;
    void p_order(BinNode* Tree, int* &pointer, int &index) const;
    void m_order(BinNode* Tree, int* &pointer, int &index) const;
    int getHeight(BinNode* Tree) const;
```

# 0x01 构造函数

二叉树初始化为空。

```cpp
BinTree::BinTree(){
    root = NULL;
    count = 0;
}
```

# 0x02 insert

插入结点需要先找到父结点，所以先写一个递归函数用于找到某个结点。

```cpp
BinNode* BinTree::search_number(BinNode* Tree, int num) const {
    while (Tree) {
        if (Tree->number == num)
            return Tree;
        else {
            search_number(Tree->lChild, num);
            search_number(Tree->rChild, num);
        }
    }
    return NULL;
}
```

新建一个结点，各属性赋值。

- 如果是根结点，则直接链入；

- 如果不是根结点，则对应链入父结点的左子或右子，其中父结点通过上一函数找到。

```cpp
void BinTree::insert(int val, int parent, int flg) {
    BinNode* newNode = new BinNode;
    newNode->data = val;
    newNode->number = count + 1;
    newNode->lChild = newNode->rChild = NULL;
    if (parent == 0)
        root = newNode;
    else {
        if (flg == -1)
            search_number(root, parent)->lChild = newNode;
        else if (flg == 1)
            search_number(root, parent)->rChild = newNode;
    }
    count++;
}
```

# 0x03 p_traversal

前序遍历的递归函数表示（注意此处要**引用传递**）：

```cpp
void BinTree::p_order(BinNode* Tree, int* &pointer, int &index) const {
    if (Tree) {
        pointer[index++] = Tree->data;
        p_order(Tree->lChild, pointer, index);
        p_order(Tree->rChild, pointer, index);
    }
}
```

在递归的基础上：

```cpp
int* BinTree::p_traversal() const {
    int* p = new int[count];
    int i = 0;
    p_order(root, p, i);
    return p;
}
```

# 0x04 m_traversal

中序遍历的递归函数表示（注意此处要**引用传递**）：

```cpp
void BinTree::m_order(BinNode* Tree, int* &pointer, int &index) const {
    if (Tree) {
        m_order(Tree->lChild, pointer, index);
        pointer[index++] = Tree->data;
        m_order(Tree->rChild, pointer, index);
    }
}
```

在递归的基础上：

```cpp
int* BinTree::m_traversal() const {
    int* p = new int[count];
    int i = 0;
    m_order(root, p, i);
    return p;
}
```

# 0x05 countNode

返回结点个数。

```cpp
int BinTree::countNode() const {
    return count;
}
```

# 0x06 height

递归遍历求二叉树高度。

```cpp
int BinTree::getHeight(BinNode* Tree) const {
    if (Tree == NULL)
        return 0;
    else {
        int m = getHeight(Tree->lChild);
        int n = getHeight(Tree->rChild);
        return (m > n) ? m + 1 : n + 1;
    }
}
```

直接调用：

```cpp
int BinTree::height() const {
    return getHeight(root);
}
```

# 0x07 析构函数

递归销毁二叉树：

```cpp
void BinTree::destroyTree(BinNode* Tree) {
    while (Tree) {
        BinNode* left = Tree->lChild;
        BinNode* right = Tree->rChild;
        delete Tree;
        Tree = NULL;
        if (left)
            destroyTree(left);
        if (right)
            destroyTree(right);
    }
}
```

直接调用：

```cpp
BinTree::~BinTree(){
    destroyTree(root);
}
```
