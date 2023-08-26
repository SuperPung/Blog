---
url: data-structures-experiment-6
title: 字符串
date: 2020-05-07 10:00:03
categories: [技术]
tags: [数据结构实验]
---

Data Structures Experiment #6 - 封装字符串类。

<!--more-->

> - `MyString.h`中的`private`需要添加必要的成员变量。并实现`MyString.cpp`中的接口方法。
>
> - `MyString::MyString(const char* str);`
>   构造函数，使用`str`构造`MyString`实例。
>
> - `MyString::~MyString();`
>   析构函数。
>
> - `int MyString::length() const;`
>   返回储存字符串的长度。
>
> - `void MyString::replace(const char* replace, int loc);`
>   将字符串中，从`loc`开始替换为`replace`。
>
>   例如，假设`MyString`实例`msa`储存的字符串为“`hello`”,执行`msa.replace("str", 1)`后，`msa`中储存的字符串变为 “`hstr`”。
>
> - `int MyString::find(const char* str) const;`
>   在`MyString`实例中查找`str`第一次出现的位置。如果实例中不包含`str`，则返回`-1`。注意，需要使用KMP方法实现。
>   例如`MyString`实例`msa`储存的字符串为“`aaabab`”， 执行`msa.find("ab")`返回2。`msa.find("abc") `返回-1。
>
> - `const char* MyString::c_string() const;`
>   返回储存在实例中字符串。

# 0x00 数据域封装

字符数组及其长度。

```cpp
private:
    char* myStr;
    int len;
```

# 0x01 构造函数

长度即为输入串的长度，对字符指针分配内存并拷贝，末尾加`'\0'`。

```cpp
MyString::MyString(const char* str){
    len = strlen(str);
    myStr = new char[len + 1];
    strcpy(myStr, str);
    myStr[len] = '\0';
}
```

# 0x02 length

直接返回长度。

```cpp
int MyString::length() const{
    return len;
}
```

# 0x03 replace

如果替换后长度增加，则需要重新分配内存。

再从`loc`处开始拷贝。

长度改变。

```cpp
void MyString::replace(const char* rep, int loc){
    int rLen = strlen(rep);
    if ((loc + rLen) > len) {
        myStr = (char*)realloc(myStr, loc + rLen + 1);
    }
    for (int i = loc; i < loc + rLen; i++)
        myStr[i] = rep[i - loc];
    myStr[loc + rLen] = '\0';

    len = loc + rLen;
}
```

# 0x04 find

利用KMP方法。

- 首先获取`next`数组：

  子串`str`自身匹配，求最大匹配数（第12行）。

- 获取`next`数组后开始对比：

  `myStr`串不回溯，每当不匹配时`str`串索引`j`回溯到`next[j]`处。当`j == strlen(str)`时说明已找到，否则未找到。

```cpp
int MyString::find(const char* str) const{
    int next[strlen(str)];
    next[0] = -1;
    int i = 0, j = -1;

    while (i < strlen(str)) {
        if (j == -1 || str[i] == str[j]) {
            i++;
            j++;
            next[i] = j;
        } else
            j = next[j];
    }

    i = 0;
    j = 0;
    while (i < len && (j == -1 || j < strlen(str))) {
        if (j == -1 || myStr[i] == str[j]) {
            i++;
            j++;
        } else
            j = next[j];
    }
    if (j == strlen(str))
        return i - j;
    else
        return -1;
}
```

# 0x05 c_string

返回字符数组。

```cpp
const char* MyString::c_string() const{
    return myStr;
}
```

# 0x06 析构函数

释放内存。

```cpp
MyString::~MyString(){
    if (myStr) {
        delete[] myStr;
        myStr = NULL;
    }
    len = 0;
}
```
