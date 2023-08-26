---
url: data-structures-experiment-10
title: 有向图
date: 2020-04-17 17:39:40
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #10 - 使用**邻接表**来完成有向图的存储，并实现相应的辅助函数用于检测图存储的是否正确。

> *根据要求，不可使用顺序表，故对算法进行了改进。*

<!--more-->

> 实现Graph的成员函数
>
> - `Graph(int v);`
>
>   构造一个顶点最多为`v`的图(编号从`1`到`v`)
>
> - `~Graph();`
>
>   析构函数，将分配的空间都释放
>
> - `void addedge(int s, int t, int w);`
>
>   向图中加入从`s`到`t`权重为`w`的边
>
> - `int getOutDegree(int v);`
>
>   获得顶点`v`的出度
>
> - `int getInDegree(int v);`
>
>   获得顶点`v`的入度
>
> - `int access(int s, int t);`
>
>   检查从`s`到`t`是否存在一个直接连接到边，如果存在，则返回边的权重，如果不存在，则返回`-1`

# 0x01 数据域封装

定义边与顶点结构体。

用邻接表的方式，边需要包括终点和权值，以及用于指向下一条边的指针域。点需要包括入度和出度，以及链接边的头指针。

将顶点创建为图。定义`num`为点的个数。

```cpp
private:
    struct EdgeNode {
        int dest;
        int cost;
        EdgeNode* link;
    };
    struct VertexNode {
        int inDegree, outDegree;
        EdgeNode* firstAdj;
    };

    VertexNode* VexList;
    int num;
```

# 0x02 构造函数

为顶点分配内存，注意顶点从1开始计数。

将顶点的度初始化为0，头指针初始化为空。

```cpp
Graph::Graph(int v){
    num = v;
    VexList = new VertexNode[v + 1];
    for (int i = 1; i <= v; i++) {
        VexList[i].inDegree = 0;
        VexList[i].outDegree = 0;
        VexList[i].firstAdj = NULL;
    }
}
```

# 0x03 addedge

用尾插法加边，首先存储边的终点和权值，指针域赋空。

寻找起点链接的最后一条边，将新边链接到它的后面。注意第`9`行不可写为`p = newEdge;`，要改变的是头指针，而不是`p`的指向。

对相应的顶点增加入度或出度。

```cpp
void Graph::addedge(int s, int t, int w){
    EdgeNode* newEdge = new EdgeNode;
    newEdge->cost = w;
    newEdge->dest = t;

    newEdge->link = NULL;
    EdgeNode* p = VexList[s].firstAdj;
    if (!p)
        VexList[s].firstAdj = newEdge;
    else {
        while (p->link)
            p = p->link;
        p->link = newEdge;
    }

    VexList[s].outDegree++;
    VexList[t].inDegree++;
}
```

采用头插法会更加简洁，可以将`6`～`14`行替换为

```cpp
    newEdge->link = VexList[s].firstAdj;
    VexList[s].firstAdj = newEdge;
```

# 0x04 getInDegree

返回对应顶点的入度。

```cpp
int Graph::getInDegree(int v){
    return VexList[v].inDegree;
}
```

# 0x05 getOutDegree

返回对应顶点的出度。

```cpp
int Graph::getOutDegree(int v){
    return VexList[v].outDegree;
}
```

# 0x06 access

若起点的头指针为空，则说明不存在以此顶点为起点的边。

若起点的头指针不为空，则从起点的头指针开始，遍历所有边，寻找终点满足条件的边并返回其权值。

找不到则返回`-1`。

```cpp
int Graph::access(int s, int t){
    EdgeNode* p = VexList[s].firstAdj;
    if (!p)
        return -1;
    while (p) {
        if (p->dest == t)
            return p->cost;
        p = p->link;
    }
    return -1;
}
```

# 0x07 析构函数

遍历并释放内存。

*Visual Studio可能会在第`5`行出现不同的问题导致不可运行。*

```cpp
Graph::~Graph(){
    for (int i = 1; i <= num; i++) {
        while (VexList[i].firstAdj) {
            EdgeNode* temp = VexList[i].firstAdj;
            VexList[i].firstAdj = temp->link;
            delete temp;
        }
    }
    delete[] VexList;
}
```
