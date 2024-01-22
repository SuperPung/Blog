---
url: data-structures-experiment-13
title: Dijkstra 算法
date: 2020-05-06 16:16:07
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #13 - 完成Dijkstra算法，实现单源最短路的求法（邻接表表示）

<!--more-->

> 完善给出的Graph类。
>
> - `Graph(int max_v);`
>   构造函数，max_v 是顶点的个数。
> - `～Graph();`
>   析构函数，释放分配的空间。
> - `void addedge(int s, int t, int w);`
>   添加一条从s到t，权重为w的有向边。
> - `int getV();`
>   获得图中顶点的个数。
> - `int* dijkstra();`
>   获取从1出发的单源最短路数组。如果某个点无法到达则长度为-1。

# 0x00 数据域封装

与[#12](/data-structures-experiment-12)基本相同。

增加了`inDegree`数组以存放各顶点入度，`dij`数组以存放dijkstra算法生成的单源最短路数组：`dij[i]`代表顶点`1`到顶点`i`的最短路权。

```cpp
private:
    struct EdgeNode {
        int dest;
        int cost;
        EdgeNode* next;
    };
    struct VertexNode {
        EdgeNode* firstAdj;
    };

    VertexNode* VexList;
    int* inDegree;
    int* dij;
    int num_v;
```

# 0x01 构造函数

与前三次基本相同。

初始化`dij[1]`为0，其他均为`-1`。

```cpp
Graph::Graph(int max_v){
    num_v = max_v;
    VexList = new VertexNode[num_v + 1];
    inDegree = new int[num_v + 1];
    dij = new int[num_v + 1];
    for (int i = 1; i <= num_v; i++) {
        VexList[i].firstAdj = NULL;
        inDegree[i] = 0;
        if (i == 1)
            dij[i] = 0;
        else
            dij[i] = -1;
    }
}
```

# 0x02 addedge

与[#12](/data-structures-experiment-12)基本相同。

```cpp
void Graph::addedge(int s, int t, int w){
    EdgeNode* newEdge = new EdgeNode;
    newEdge->weight = w;
    newEdge->vertex = t;

    newEdge->next = VexList[s].firstAdj;
    VexList[s].firstAdj = newEdge;

    inDegree[t]++;
}
```

# 0x03 getV

判断出度和入度不同时为零。

```cpp
int Graph::getV(){
    int sum = 0;
    for (int i = 1; i <= num_v; i++) {
        if (VexList[i].firstAdj || inDegree[i])
            sum++;
    }

    return sum;
}
```

# 0x04 dijkstra

[什么是dijkstra算法？](https://v.youku.com/v_show/id_XMjQyOTY1NDQw.html?spm=a2hbt.13141534.app.5~5!2~5!2~5~5~5!2~5~5!2~5!2~5!2~5~5!32~A)

设顶点数为`count`，则至多只需`count-1`次（即`left`次）更新`dij`数组即可。

- 更新前的准备工作：

  声明每次更新时从`dij`数组找到的未访问过的最小值以及它的下标，`visit`数组用于判断是否访问过此顶点，并将`dij`数组初始化（类似广度第一层）。

- 开始更新：

  - 先初始化`min`，找一个未访问过的且距离不为无穷的顶点。若找完后`i > count`，则说明没有符合条件的顶点，即未访问过的顶点都是无穷远的顶点，说明算法已经结束，直接`break`。

  - 初始化后，再逐个比较找到最小值，并更新最小值及其下标。成功找到最小值后将该顶点记为已访问。

  - 从最小值顶点开始遍历其链表，设指针为`p`，判断`1`->(不一定直达)->`min_index`->`p->dest`路径总权（`p->cost+min`）是否比`1`->(不一定直达)->`p->dest`（`dij[p->dest]`）小，取较小者更新`dij[p->dest]`。

- 更新后：

  - 观察`main.cpp`，发现要求dijkstra算法生成的单源最短路数组为从`0`开始，则只需将`dij`数组整体前移1位。

  - 释放内存并返回。

完整算法如下。

```cpp
int* Graph::dijkstra(){
    int min, min_index;
    int count = getV();
    int i;
    int left = count - 1;
    bool* visit = new bool[count + 1];
    EdgeNode* p;

    for (i = 2; i <= count; i++)
        visit[i] = false;
    p = VexList[1].firstAdj;
    while (p) {
        dij[p->dest] = p->cost;
        p = p->next;
    }

    while (left--) {
        for (i = 2; i <= count; i++) {
            if (dij[i] >= 0 && !visit[i]) {
                min = dij[i];
                min_index = i;
                break;
            }
        }
        if (i > count)
            break;

        for (i = 2; i <= count; i++) {
            if (dij[i] >= 0 && dij[i] < min && !visit[i]) {
                min = dij[i];
                min_index = i;
            }
        }
        visit[min_index] = true;

        p = VexList[min_index].firstAdj;
        while (p) {
            dij[p->dest] = (p->cost + min < dij[p->dest] || dij[p->dest] == -1) ?
                           (p->cost + min) : dij[p->dest];
            p = p->next;
        }
    }

    for (i = 0; i < count; i++)
        dij[i] = dij[i + 1];

    delete[] visit;
    return dij;
}
```

# 0x05 析构函数

逐结点释放内存。

若`main.cpp`末尾有以下语句：

```cpp
    delete arr;
    // 在这里可以释放 dijkstra 分配的数组
```

则无需在析构函数中释放`dij`数组内存。

```cpp
Graph::~Graph(){
    for (int i = 1; i <= num_v; i++) {
        while (VexList[i].firstAdj) {
            EdgeNode* temp = VexList[i].firstAdj;
            VexList[i].firstAdj = temp->next;
            delete temp;
        }
    }
    delete[] VexList;
    delete[] inDegree;
    delete[] dij;
}
```

# 0x06 补充

建议增加测试样例，现提供一个如下（供参考）。

![13g1](https://i0.hdslb.com/bfs/album/e6c1e1b8c4555508d90f08cc76b283c0be358fc9.gif)

```cpp
    Graph g1(7);

    g1.addedge(1, 2, 7);
    g1.addedge(1, 3, 9);
    g1.addedge(1, 6, 14);
    g1.addedge(2, 3, 10);
    g1.addedge(2, 4, 15);
    g1.addedge(3, 4, 11);
    g1.addedge(3, 6, 2);
    g1.addedge(4, 5, 6);
    g1.addedge(6, 5, 9);

    int len1 = g1.getV();
    int *arr1 = g1.dijkstra();

    int r_arr1[6] = {0, 7, 9, 20, 20, 11};

    if(len1 == 6){
        cout << "Pass check point 3!" << endl;
    }

    for(i = 0; i < len1; i++){
        if(arr1[i] != r_arr1[i]) break;
    }
    if(i == len1) cout << "Pass check point 4!" << endl;
```
