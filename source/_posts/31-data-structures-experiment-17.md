---
url: data-structures-experiment-17
title: 归并排序、基数排序
date: 2020-06-03 15:09:27
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #17 - 排序2

<!--more-->

> 在`MySort.cpp`中完成两个排序方法：
>
> - `mergeSort(int* arr, int len);`
>
>   实现归并排序，需要排序的数组为`arr`，数组长度为`len`
>
> - `cardSort(int* arr, int len);`
>
>   实现基数排序，需要排序的数组为`arr`，数组长度为`len`
>
>   整数基数排序的一个方法：可以逐位取`&`获得不同位的信息。例如，第一趟基数排序时，将序列中的数字(`a & 0xf0000000`)，就得到了最高四位的数值，可以首先对最高位比较大小并进行排序。

# mergeSort

归并排序仍然是一种“分治”的思想，即将数组分成两部分，分别对其排序，再依次取两个有序数组中较小的元素，将两个数组“归并”成一个数组。只要不断地将数组一分为二，再分别归并，就能完成对这个数组的排序。

可以看出此处为递归函数：

```cpp
void mergeSort(int *arr, int len){
    if (len < 2)
        return;
    int mid = len / 2;
    int* left = new int[mid];
    int* right = new int[len - mid];
    int i;

    for (i = 0; i < mid; i++)
        left[i] = arr[i];
    for (i = 0; i < len - mid; i++)
        right[i] = arr[i + mid];

    mergeSort(left, mid);
    mergeSort(right, len - mid);

    merge(left, mid, right, len - mid, arr);

    delete[] left;
    delete[] right;
}
```

先一分为二（若此时数组大小已经小于2，即停止递归），左半部分存入`left`，右半部分存入`right` ，再进行递归，最后`merge`函数实现归并：

```cpp
void merge(int* left, int leftCount, int* right, int rightCount, int* result) {
    int i = 0, j = 0, k = 0;

    while (i < leftCount && j < rightCount) {
        if (left[i] < right[j])
            result[k++] = left[i++];
        else
            result[k++] = right[j++];
    }

    while (i < leftCount)
        result[k++] = left[i++];
    while (j < rightCount)
        result[k++] = right[j++];
}
```

传入参数分别为：左半数组以及它的大小、右半数组以及它的大小、归并存入的数组。

递归的过程想象成二叉树，从叶结点开始。因为第一层递归时两边数组大小均为1，所以只需判断并找出左侧和右侧中较小的一个存入第一位，剩下那个存入第二位即可。第二层递归时两边的数组都是经过第一层之后的，自然都是有序的，后续递归也是如此。

注意，下面两个`while`循环只会执行一个。

# cardSort

基数排序和归并排序的思想相同，仍进行“分治”，只不过“分”的不是数组，而是每个数值。

假设对一个值全是两位数的数组排序，直接排序较为困难，我们可以先大致排一下序，即将0～9的分到一组、将10～19的分到一组……这样先按照十位数排好序，再分别对每一组的数值按个位数排序，就可以完成整体的排序。这种方法就是**最高位优先法**。

也可以反过来，即先按个位数排好序，再按十位数排序。这种方法就是**最低位优先法**，它和最高位优先法相比，优点是无需分组，因为最终的有序序列中，十位数相同的数值的个位数一定是按升序排列的。

所以我选择最低位优先法。下面看一下基数排序具体是如何实现的。

---

首先定义一些变量：

- `maxBits`是整个数组最大元素的位数（十进制），用于决定我们要循环多少次。

  因为一次循环是对当前位的排序。第一次对个位数排好序后，第二次再对十位数排序时，两个十位数相同的元素，较后的一定比较前的元素大。

- `curBits`是当前元素的位数（十进制），`curNum`是当前元素。

  这两个变量只在寻找`maxBits`时用到。

- `curBit`是当前循环正在比较的位（十进制），如个位是`1`、十位是`10`、百位是`100`等等。配合下面两个变量效果更佳。

- `MSBits`是当前元素的最高若干位（十进制），`LSBit`是`MSBits`中最低的一位。

  显而易见，这两个变量在循环中比较某一位时会用到。用当前元素除以`curBit`即为`MSBits`，再`% 10`即为`LSBit`。

  其实也可以不另取变量，此处只为提高可读性。

- `group`数组作为一个函数（数学意义上），存储某一位上元素的个数（后续还会进行改进）。

  若`group[x]`的值为`y`，则意味着当前循环中，最低位为`x`的元素有`y`个。`x`的取值范围为0～9，共10个元素的大小。

- `result`数组用于存放当前循环排好序的序列，最终再copy给`arr`数组。

- `i`和`j`用于`for`循环的索引。

```cpp
    int maxBits = 1;
    int curBits, curNum, curBit;
    int MSBits, LSBit;
    int group[10];
    int result[len];
    int i, j;
```

---

其次就是第一步，找到最大位数`maxBits`：

遍历整个数组，找到最大的位数。

求位数的方法就是一直除以`10`（“右移”一位），直到结果为`0`，看它除了多少次。

```cpp
    for (int i = 0; i < len; i++) {
        curNum = arr[i];
        curBits = 0;
        while (curNum) {
            curBits++;
            curNum /= 10;
        }
        maxBits = (curBits > maxBits) ? curBits : maxBits;
    }
```

---

最后是核心的循环部分（代码已截断）：

初始时`curBit`为1，代表此次循环针对的是个位数，此后每次循环`curBit`自增十倍，代表更高位。

每次循环开始时都要初始化`group`数组为全0用于计数。

```cpp
    for (i = 0, curBit = 1; i < maxBits; i++, curBit *= 10) {
        for (j = 0; j < 10; j++)
            group[j] = 0;
```

此处的`for`循环用于给`group`数组赋值。第2～3行为固定操作，取了当前排序的位。将`group`中对应位置的计数增1即可。

```cpp
        for (j = 0; j < len; j++) {
            MSBits = arr[j] / curBit;
            LSBit = MSBits % 10;
            group[LSBit]++;
        }
```

注意，为了简化排序，后续操作中我不再对`arr`进行交换，而是直接将对应元素复制到另一个`result`数组。那么此时“应该复制到哪个位置”就是需要解决的问题。

由于上一步得到的`group`数组已经存放了对应位的个数，那么我们只需将`group[1]`加上`group[0]`，就得到了当前位为`1`的元素在数组中的终止位置，后续也是如此。

注意数组从0开始，需先将`group[0]`自减1。

```cpp
        group[0]--;
        for (j = 1; j < 10; j++)
            group[j] += group[j - 1];
```

其次是核心中的核心，将`arr`数组复制到`result`的对应位置。

上一步已经计算出了当前位为0、1、2……9的元素们各自在数组中的终止位置，即我们已经将数组分块并大致排好了顺序；且上一次循环已经将当前位的下一位排好序（第一次除外），即我们可以对每一块进行更细的排序。所以我们只需**从后向前遍历**（保证每一块的元素从大到小），并将元素存入对应的终止位置，存入后终止位置自减1，最后即可得到我们想要的排好序的`result`序列。

```cpp
        for (j = len - 1; j >= 0; j--) {
            MSBits = arr[j] / curBit;
            LSBit = MSBits % 10;
            result[group[LSBit]--] = arr[j];
        }
```

此时的`arr`数组只是作为每次循环的传递作用，故最后将得到的`result`数组拷贝到`arr`。

```cpp
      for (j = 0; j < len; j++)
            arr[j] = result[j];
    }
}
```

---

完整代码如下：

```cpp
void cardSort(int* arr, int len){
    int maxBits = 1;
    int curBits, curNum, curBit;
    int MSBits, LSBit;
    int group[10];
    int result[len];
    int i, j;

    for (int i = 0; i < len; i++) {
        curNum = arr[i];
        curBits = 0;
        while (curNum) {
            curBits++;
            curNum /= 10;
        }
        maxBits = (curBits > maxBits) ? curBits : maxBits;
    }

    for (i = 0, curBit = 1; i < maxBits; i++, curBit *= 10) {
        for (j = 0; j < 10; j++)
            group[j] = 0;
        for (j = 0; j < len; j++) {
            MSBits = arr[j] / curBit;
            LSBit = MSBits % 10;
            group[LSBit]++;
        }

        group[0]--;
        for (j = 1; j < 10; j++)
            group[j] += group[j - 1];

        for (j = len - 1; j >= 0; j--) {
            MSBits = arr[j] / curBit;
            LSBit = MSBits % 10;
            result[group[LSBit]--] = arr[j];
        }

        for (j = 0; j < len; j++)
            arr[j] = result[j];
    }
}
```
