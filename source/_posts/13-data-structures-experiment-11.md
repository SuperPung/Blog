---
url: data-structures-experiment-11
title: Prim 和 Kruskal 算法
date: 2020-04-23 19:16:56
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #11 - 完成最小生成树Prim算法和Kruskal算法

<!--more-->

> 在上机样例中给出了图类的基础实现。最小生成树算法针对的是无向图，注意与上机10的区别。
>
> - `Graph();`
>   构造函数
> - `~Graph();`
>   析构函数
> - `void addedge(int s, int t, int w);`
>   向图中添加一条从s到t，权重为w的边
> - `int prim();`
>   使用prim算法求得图的最小生成树，并将结果返回
> - `int kruskal();`
>   使用kruskal算法求得图的最小生成树，并将结果返回

经历了多次修改与完善，终于完成了两个算法。

可以使用邻接矩阵或邻接表的方式来表示，本算法采用邻接表。

# 0x01 数据域封装

基本思路与[Experiment #10](https://superpung.com/data-structures-experiment-10/)类似。

在边结点中，增加了“起点”`vAdj`，便于后续算法的实现。

在点结点中，增加了`flag`标志，用于区分此顶点是否被访问过，同样便于后续算法的实现。

对点与边分别计数，便于控制循环次数。

```cpp
private:
    struct EdgeNode {
        int vAdj, vertex;
        int weight;
        EdgeNode* next;
    };
    struct VertexNode {
        bool flag;
        EdgeNode* firstAdj;
    };

    VertexNode* VexList;
    int num_v, num_e;
```

# 0x02 构造函数

构造一个只有顶点没有边的图，注意顶点依然从`1`开始排序。

初始化所有顶点均未被访问过。

```cpp
Graph::Graph(int max_v){
    num_v = max_v;
    num_e = 0;
    VexList = new VertexNode[num_v + 1];
    for (int i = 1; i <= num_v; i++) {
        VexList[i].firstAdj = NULL;
        VexList[i].flag = false;
    }
}
```

# 0x03 addedge

两种算法针对的是无向图，故两顶点之间双向连通。

边数只记一次。

```cpp
void Graph::addedge(int s, int t, int w){
    EdgeNode* newEdge1 = new EdgeNode;
    newEdge1->weight = w;
    newEdge1->vAdj = s;
    newEdge1->vertex = t;

    newEdge1->next = VexList[s].firstAdj;
    VexList[s].firstAdj = newEdge1;

    EdgeNode* newEdge2 = new EdgeNode;
    newEdge2->weight = w;
    newEdge2->vAdj = t;
    newEdge2->vertex = s;

    newEdge2->next = VexList[t].firstAdj;
    VexList[t].firstAdj = newEdge2;

    num_e++;
}
```

# 0x04 Prim's algorithm

Prim算法的核心思想是**加点**。

从某一顶点开始，寻找与它相连的且相连边权最小的顶点，将它们加入已经访问的顶点集合。再次寻找与这两个顶点相连的所有顶点中相连边权最小的顶点，并加入顶点集合。重复此步骤，直到访问了所有顶点，即全部顶点都已加入已经访问的顶点集合。

## 1. Initializing

初始化所有顶点均未被访问，声明`sum`以存储权之和、`min_w`以存储每次循环找到的最小权、`visited`数组以存储已经访问过的顶点、`index`记录目前已经访问的顶点个数。

```cpp
for (int i = 1; i <= num_v; i++)
        VexList[i].flag = false;

int sum = 0;
int min_w;
int* visited = new int[num_v];
int index = 0;
```

## 2. Starting

选择一个顶点作为起始顶点，存储到已访问的顶点集合，**并记录为已访问**。

```cpp
for (int i = 1; i <= num_v; i++) {
    if (VexList[i].firstAdj) {
        visited[index++] = i;
        VexList[i].flag = true;
        break;
    }
}
```

## 3. Calculating

当所有顶点未被完全访问时，不断循环，直至顶点全部访问。

`2`～`14`行：对最小权（**及其顶点**）进行初始化，任取当前顶点与**未访问顶点**所连接边的权（不可随机设定），便于后续进行比较。不可将此步骤置于下一循环体中，否则每当切换顶点后都会对`min_w`重新赋值。`7`行要进行判断，若到达链表末尾且未找到尚未访问的顶点，则停止并开始从下一顶点的链表中开始寻找。

`16`～`24`行：在已经访问的顶点中，**逐个顶点**寻找其与未访问过顶点所连接的边（链表）中最小的权，并将对应顶点存入已访问的顶点集合。（最小权一定要对应顶点）

`26`～`28`行：**当最小权找到后**，将对应顶点记录为已访问，已访问的顶点个数加一，并把最小权加入`sum`中。

```cpp
while (index < num_v) {
    for (int i = 0; i < index; i++) {
        EdgeNode* p = VexList[visited[i]].firstAdj;
        while (VexList[p->vertex].flag && p->next) {
            p = p->next;
        }
        if (VexList[p->vertex].flag)
            continue;
        else {
            min_w = p->weight;
            visited[index] = p->vertex;
            break;
        }
    }

    for (int i = 0; i < index; i++) {
        EdgeNode* p = VexList[visited[i]].firstAdj;
        while (p) {
            if (p->weight <= min_w && !VexList[p->vertex].flag) {
                min_w = p->weight;
                visited[index] = p->vertex;
            }
            p = p->next;
        }
    }
    VexList[visited[index]].flag = true;
    index++;
    sum += min_w;
}
```



## 4. Freeing and Returning

**释放内存**，并返回结果。

```cpp
delete[] visited;
return sum;
```

## 5. 完整的Prim算法

```cpp
int Graph::prim(){
    for (int i = 1; i <= num_v; i++)
        VexList[i].flag = false;

    int sum = 0;
    int min_w;
    int* visited = new int[num_v];
    int index = 0;

    for (int i = 1; i <= num_v; i++) {
        if (VexList[i].firstAdj) {
            visited[index++] = i;
            VexList[i].flag = true;
            break;
        }
    }

    while (index < num_v) {
        for (int i = 0; i < index; i++) {
            EdgeNode* p = VexList[visited[i]].firstAdj;
            while (VexList[p->vertex].flag && p->next) {
                p = p->next;
            }
            if (VexList[p->vertex].flag)
                continue;
            else {
                min_w = p->weight;
                visited[index] = p->vertex;
                break;
            }
        }

        for (int i = 0; i < index; i++) {
            EdgeNode* p = VexList[visited[i]].firstAdj;
            while (p) {
                if (p->weight <= min_w && !VexList[p->vertex].flag) {
                    min_w = p->weight;
                    visited[index] = p->vertex;
                }
                if (!p->next)
                    break;
                p = p->next;
            }
        }

        VexList[visited[index]].flag = true;
        index++;
        sum += min_w;
    }

    delete[] visited;
    return sum;
}
```

# 0x05 Kruskal's algorithm

Kruskal算法的核心思想是**加边**。

将图中所有边按权值大小进行排序，从权最小的边开始，逐条边加入生成树中，注意加边后不可与已加入的边构成环，直至所有顶点均连通。

## 1. Initializing

声明`sum`以存储权之和、`edges`数组以存储边、`index`记录目前已经访问的边数。

声明`begin`和`end`便于判断加边后是否会构成环，<span id = "set">`set`数组</span>存储每个顶点的[**状态**](#state)，即所在连通分支，起到函数的作用（`set`值）。

```cpp
int sum = 0;
EdgeNode* edges = new EdgeNode[num_e];
int index = 0;
int begin, end;
int* set = new int[num_v + 1];
for (int i = 1; i <= num_v; i++)
    set[i] = i;
```

## 2. Storing

从第一个顶点开始，将所有边存入`edges`数组中。

为避免将同一条边“双向”存储，只存储从小顶点到大顶点方向的边，故在`4`行进行了判断。

注意存储时还要包括边的“起点”，这是与以往不同的地方，以便于后续对连通分支的判断。

```cpp
for (int i = 1; i <= num_v; i++) {
    EdgeNode* p = VexList[i].firstAdj;
    while (p) {
        if (p->vertex > i) {
            edges[index].vAdj = i;
            edges[index].vertex = p->vertex;
            edges[index].weight = p->weight;
            index++;
        }
        p = p->next;
    }
}
```

## 3. Sorting

将存储的边按权从小到大进行排序，采用冒泡算法。

```cpp
for (int i = 0; i < num_e; i++) {
    for (int j = i + 1; j < num_e; j++) {
        if (edges[i].weight > edges[j].weight) {
            EdgeNode temp = edges[i];
            edges[i] = edges[j];
            edges[j] = temp;
        }
    }
}
```

## 4. Calculating

以排好序的边数组为基准，从头开始挑选边，并将其加入生成树。当挑选出来的边数达到`顶点数-1`时，说明已经将所有顶点连通，停止。当所有顶点均已挑选过，停止。

[初始化`set`数组](#set)时，<span id = "state">将第`i`位赋值为`i`</span>，说明每个顶点一开始均自己为一连通分支，即若两顶点的`set`值不等，则这两个顶点必位于不同的连通分支，此时可以将连接这两个顶点的边加入生成树中。

加入以后，这两个顶点便连通了，也即这两个顶点所在的两个连通分支便连通了。记这两个顶点分别为`begin`和`end`。此时将与`end`在同一连通分支的所有顶点（包括它自己）全部加入到`begin`的连通分支中（也可以从另一个方向），也即将与`end`的`set`值相等的所有顶点的`set`值全部改为`begin`的`set`值。

注意此处必须单独赋值给`begin`和`end`，否则随着`set`值的改变，判断条件中的值也会随即改变，需要让它定下来。

若新加入一条边，则已加入的边数加一。

```cpp
for (int i = 0, j = 0; i < num_e && j < num_v; i++) {
    if (set[edges[i].vAdj] != set[edges[i].vertex]) {
        sum += edges[i].weight;
        begin = set[edges[i].vAdj];
        end = set[edges[i].vertex];
        for (int k = 1; k <= num_v; k++) {
            if (set[k] == end)
                set[k] = begin;
        }
        j++;
    }
}
```

## 5. Freeing and Returning

**释放内存**，并返回结果。

```cpp
delete[] set;
delete[] edges;
return sum;
```

## 6. 完整的Kruskal算法

```cpp
int Graph::kruskal(){
    int* set = new int[num_v + 1];
    int sum = 0;
    EdgeNode* edges = new EdgeNode[num_e];
    int index = 0;
    int begin, end;
    for (int i = 1; i <= num_v; i++)
        set[i] = i;

    for (int i = 1; i <= num_v; i++) {
        EdgeNode* p = VexList[i].firstAdj;

        while (p) {
            if (p->vertex > i) {
                edges[index].vAdj = i;
                edges[index].vertex = p->vertex;
                edges[index].weight = p->weight;
                index++;
            }
            p = p->next;
        }
    }

    for (int i = 0; i < num_e; i++) {
        for (int j = i + 1; j < num_e; j++) {
            if (edges[i].weight > edges[j].weight) {
                EdgeNode temp = edges[i];
                edges[i] = edges[j];
                edges[j] = temp;
            }
        }
    }

    for (int i = 0, j = 0; i < num_e && j < num_v; i++) {
        if (set[edges[i].vAdj] != set[edges[i].vertex]) {
            sum += edges[i].weight;
            begin = set[edges[i].vAdj];
            end = set[edges[i].vertex];
            for (int k = 1; k <= num_v; k++) {
                if (set[k] == end)
                    set[k] = begin;
            }
            j++;
        }
    }

    delete[] set;
    delete[] edges;
    return sum;
}
```

# 0x06 析构函数

逐个结点释放内存。

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
}
```

# 0x07 补充

由于题目给出的样例图极其特殊，包括但不限于边数过少、边权值大量重复等，以至于你很容易`pass check point!`而不能发现自身算法的缺陷与不足。

所以我**强烈建议**自行增加测试样例，下面有三个样例可供参考。

## g1

![11g1](https://i0.hdslb.com/bfs/album/fc32ff3b8089381e9ae15f27a01b8f63f0ee07eb.png)

（顶点`A`～`G`分别对应`1`～`7`）

```cpp
Graph g1(7);

g1.addedge(1, 2, 7);
g1.addedge(1, 4, 5);
g1.addedge(2, 3, 8);
g1.addedge(2, 4, 9);
g1.addedge(2, 5, 7);
g1.addedge(3, 5, 5);
g1.addedge(4, 5, 15);
g1.addedge(4, 6, 6);
g1.addedge(5, 6, 8);
g1.addedge(5, 7, 9);
g1.addedge(6, 7, 11);

if(g1.prim() == 39) cout << "pass check point 3!" << endl;
if(g1.kruskal() == 39) cout << "pass check point 4!" << endl;
```

## g2

![11g2](https://i0.hdslb.com/bfs/album/eb24c019395607ba1022c2a79e44d32753548a95.png)

```cpp
Graph g2(6);

g2.addedge(1, 2, 6);
g2.addedge(1, 3, 1);
g2.addedge(1, 4, 5);
g2.addedge(2, 3, 5);
g2.addedge(2, 5, 3);
g2.addedge(3, 4, 5);
g2.addedge(3, 5, 6);
g2.addedge(3, 6, 4);
g2.addedge(4, 6, 2);
g2.addedge(5, 6, 6);

if(g2.prim() == 15) cout << "pass check point 5!" << endl;
if(g2.kruskal() == 15) cout << "pass check point 6!" << endl;
```

## g3

![11g3](https://i0.hdslb.com/bfs/album/e9ee363b1c9836c2d50fee1dd3049e6d6fb54c13.png)

```cpp
Graph g3(10);

g3.addedge(1, 2, 3);
g3.addedge(1, 4, 6);
g3.addedge(1, 5, 9);
g3.addedge(2, 3, 2);
g3.addedge(2, 4, 4);
g3.addedge(2, 6, 9);
g3.addedge(3, 4, 2);
g3.addedge(3, 6, 8);
g3.addedge(3, 7, 9);
g3.addedge(4, 7, 9);
g3.addedge(5, 6, 8);
g3.addedge(5, 10, 18);
g3.addedge(6, 7, 7);
g3.addedge(6, 9, 9);
g3.addedge(6, 10, 10);
g3.addedge(7, 8, 4);
g3.addedge(7, 9, 5);
g3.addedge(8, 9, 1);
g3.addedge(8, 10, 4);
g3.addedge(9, 10, 3);

if(g3.prim() == 38) cout << "pass check point 7!" << endl;
if(g3.kruskal() == 38) cout << "pass check point 8!" << endl;
```
