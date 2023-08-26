---
url: data-structures-experiment-9
title: 哈夫曼树
date: 2020-04-09 09:46:32
categories: [技术]
tags: [数据结构实验]
mathjax: true
---
Data Structures Experiment #9 - 完成哈夫曼树类的私有属性以及接口，实现哈夫曼树类。

<!--more-->

> 在`HuffmanTree`类中给出了四个函数:
> - `HuffmanTree(const char* str);`
>   将`str`构建为哈夫曼树，要求权值较小的节点放在左子，左子编号为`0`。例如`"aaaabbc"`的树生成如下。
>
>   ![HufTree](https://i0.hdslb.com/bfs/album/25e097dfaba0636eb8eb1f1d2a121ac69c3ba89c.png)
>
> - `char* getencode(char c);`
>   获取字符`c`的编码。将结果以`char*`的形式返回。上例中的 `getencode(‘c’)`返回结果为`00`。
>
> - `int getWPL();`
>   返回哈夫曼树的WPL。
>
> - `~HuffmanTree();`
> 哈夫曼树的析构函数。

# 0x00 数据域封装

定义Huffman树结点类型`HTNode`，包括结点字符、权、父结点、左子、右子、编码等。
存储叶结点个数`leaves`和结点个数`nodes`。
存储结点转换后的编码。

最终将建成的Huffman树存储在`HTNode`型数组中，前`leaves`项存储叶结点，后`nodes - leaves`项存储根结点。

由于父结点、左子、右子可以用数组下标指向，故声明为`int`型。

> 为便于操作字符串，将结点编码声明为`string`型，需要包含`<string>`头文件。

```cpp
private:
    struct HTNode {
        char data;
        int weight;
        std::string code;
        int lChild, rChild, parent;
    };
    HTNode* HufTree;
    int leaves;
    int nodes;
    char* encode;
```

# 0x01 构造函数

由于构造Huffman树的步骤较多，为便于理解，我将构造函数分为`Reading`->`Initializing`->`Combining`->`Encoding`->`Freeing`五个阶段。

## Reading

此阶段读取`char*`型的`str`，并存储其中的字符和权。

应考虑`str`中字符未必连续，故算法进行了改进：先读取字符，再读取权。*（此处感谢@刘宇婷同学的指正）*

计算权之前，需要对权数组初始化，即全部赋0。若忽略此操作，虽然macOS与Linux均可正常运行，但Windows下会出现错误，因为未对数组初始化导致数组各项存放的值均为“垃圾值”，不可确定。这与其他变量的声明类似，声明后不初始化不是好的编程习惯。*（此处感谢@刘宇婷同学的指正）*

> 操作C字符串需要包含`<cstring>`头文件。

```cpp
int size;
char* ch = new char[strlen(str)];
int* weigh = new int[strlen(str)];
size = 0;
ch[0] = str[0];

for (int i = 1; i < strlen(str); i++) {
    bool exist = false;
    for (int j = 0; j < size + 1; j++) {
        if (str[i] == ch[j])
            exist = true;
    }
    if (!exist)
        ch[++size] = str[i];
}
leaves = size + 1;
nodes = 2 * leaves - 1;

for (int i = 0; i < leaves; i++) {
    weigh[i] = 0;
    for (int j = 0; j < strlen(str); j++) {
        if (ch[i] == str[j])
            weigh[i]++;
    }
}
```

## Initializing

此阶段用上一阶段存储的字符数组和权数组来初始化Huffman数组的前部分（叶结点），其他赋值为`-1`便于进行判断。

`char*`型结点编码`encode`在此处定义，便于[以后](#encode)使用。

> 此处若`encode`未被定义，则会出现报错`Segmentation fault: 11`。

```cpp
HufTree = new HTNode[nodes];
encode = new char[nodes];

for (int i = 0; i < nodes; i++) {
    HufTree[i].weight = 0;
    HufTree[i].parent = -1;
    HufTree[i].lChild = -1;
    HufTree[i].rChild = -1;
    if (i < leaves) {
        HufTree[i].data = ch[i];
        HufTree[i].weight = weigh[i];
        HufTree[i].code = "";
    }
}
```

## Combining

此阶段将上一阶段初始化的Huffman数组进行合并，构建Huffman树。

合并时需要寻找权最小的两个结点，故另构造`Searching`函数，此函数**须写在构造函数外部**。

### Searching

由于需要找到权最小的两个结点，可以直接赋值给参数，使用引用传递。

此处应确保最小权为正，因为权为0即结点未初始化，权为负即结点被[屏蔽](#shield)。

初始化`min`和`_min`时应确保其为正，不可以直接取第0位，可能会导致判断出现错误。*（此处感谢@刘宇婷同学的指正）*

> 不要忘记初始化后`break`。

```cpp
void HuffmanTree::mini(int& left, int& right) {
    int min, _min;
    for (int i = 0; i < nodes; i++) {
        if (HufTree[i].weight > 0) {
            min = _min = HufTree[i].weight;
            left = right = i;
            break;
        }
    }

    for (int i = 0; i < nodes; i++) {
        if (HufTree[i].weight < min && HufTree[i].weight > 0) {
            min = HufTree[i].weight;
            left = i;
        }
    }

    for (int i = 0; i < nodes; i++) {
        if (HufTree[i].weight < _min && HufTree[i].weight > min && HufTree[i].weight > 0) {
            _min = HufTree[i].weight;
            right = i;
        }
    }
}
```

### Combining

寻找权最小的两个结点，进行合并。

将所得父结点存储在Huffman数组后部分（根结点），将后部分进行初始化。

合并之后需要将两个子结点<span id = "shield">**屏蔽**</span>，不会出现在以后的`Searching`中。

> 此处若不处理已经合并的结点，则下次寻找时仍会返回此结点而出现错误。

```cpp
for (int i = leaves; i < nodes; i++) {
    int min1, min2;
    mini(min1, min2);

    HufTree[min1].parent = HufTree[min2].parent = i;
    HufTree[i].lChild = min1;
    HufTree[i].rChild = min2;

    HufTree[i].weight = HufTree[min1].weight + HufTree[min2].weight;

    HufTree[min1].weight = -HufTree[min1].weight;
    HufTree[min2].weight = -HufTree[min2].weight;
}
```

## Encoding

此阶段将上一阶段构建完成的Huffman树的叶结点进行编码，通过寻找每一个叶结点的父结点，再寻找该父结点的父结点……以此类推，每次追溯父结点时判断子结点是父结点的左子还是右子，编码字符串相应地添加`0`或`1`。最后将所得字符串反转，即为编码。

> 为实现字符串的反转，需要`#include <algorithm>`。

```cpp
for (int i = 0; i < leaves; i++) {
    int parent_index = HufTree[i].parent;
    int child_index = i;
    while (parent_index != -1) {
        if (child_index == HufTree[parent_index].lChild)
            HufTree[i].code += "0";
        else if (child_index == HufTree[parent_index].rChild)
            HufTree[i].code += "1";
        child_index = parent_index;
        parent_index = HufTree[parent_index].parent;
    }
    reverse(HufTree[i].code.begin(), HufTree[i].code.end());
}
```

## Freeing

释放字符数组和权数组内存。

```cpp
delete[] ch;
delete[] weigh;
```

*至此，构造函数已经完成。*

# 0x02 Get Encode

遍历叶结点，找到`c`字符对应的结点，将其编码转换为`char*`型，存储于构造函数的<span id = "encode">`encode`</span>中。

```cpp
char* HuffmanTree::getcode(char c){
    for (int i = 0; i < leaves; i++) {
        if (HufTree[i].data == c) {
            for (int j = 0; j < HufTree[i].code.length(); j++)
                encode[j] = HufTree[i].code[j];
            encode[HufTree[i].code.length()] = '\0';

            return encode;
        }
    }
    return NULL;
}
```

# 0x03 Get WPL

WPL：Weighted Path Length（带权路径长度），计算公式如下。$$WPL =\sum_{k=1}^nw_kl_k$$

```cpp
int HuffmanTree::getWPL() {
    int WPL;
    WPL = 0;

    for (int i = 0; i < leaves; i++)
        WPL += -HufTree[i].weight * HufTree[i].code.length();

    return WPL;
}
```

# 0x04 析构函数

释放内存。

```cpp
HuffmanTree::~HuffmanTree() {
    delete[] HufTree;
    delete[] encode;
}
```
