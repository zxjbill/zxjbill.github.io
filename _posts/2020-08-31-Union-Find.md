---
layout: post
title:  "Union Find 查并集"
date:   2020-08-31 15:34:00
categories: Data-Structure-and-Algorithm
tags: 算法 Union Find
mathjax: true
mermaid: false
excerpt_separator: <!--more-->
---

* content
{:toc}
并查集（Union-Find）是集合元素根据相交与否进行划分的数据结构。本文简单介绍原理，`C++` 代码实现和少量案例。
<!--more-->

## 基本逻辑
* 等价关系：定义在集合 S 上的元素，描述两元素之间的关系 R，且此关系具有自反性、对称性、传递性，称 R 为等价关系。
* 等价关系的两种常见存放方式：
  * 布尔矩阵：集合 S 有 N 个元素，则利用 $N \times N$ 存放这种所有关系，真则相关，假则无关。
  * 等价类（不相交集）：$x \in S$ 的等价类包含所有与 x 相关的元素。这样任意两个等价类不相交。所有等价类的并集为整个集合。
* 不相交集合主要实现两个操作，find 和 union。
  * find：查找某个元素属于哪个子集，返回包含给定元素的子集的名称。
  * union：添加等价关系。涉及到判断对应的不相交集合是否存在，不相交集合的是否相同，合并不相交集合。

## 实现思路
* 采用树的结构进行存储，每个结点只需要记录父节点即可，双亲表示法。所以可以利用整数数组来存储。由于可能有多个不相交集合，所以整个数组是一颗树。
* find 和 union 性能受到形成的树的高度所影响。
* 改进措施：
  * union 按照高度、规模并。
  * find 路径压缩
  * 其中路径压缩和按照规模并是兼容的，两者可以同时使用。

```cpp
class DisjointSet{
private:
    int size;
    int *parent;

public:
    DisjoinSet(int s);
    ~DisjointSet(){
        delet[] parent;
    }
    void Union(int root1, int root2);
    int Find(int x);
};

DisjoinSet::DisjoinSet(int n){
    size = n;
    parent = new int[size];

    for (int i = 0; i < size; ++i){
        parent[i] = -1;
    }
}

int DisjoinSet::Find(int x){
    if (parent[x] < 0]){
        return x;
    }
    // 此处有路径压缩
    return parent[x] = Find(parent[x]);
}

void DisjoinSet::Union(int root1, int root2){
    if (root1 == root2){
        return;
    }

    // 按照规模并
    if (parent[root1] > parent[root2]){
        parent[root2] += parent[root1];
        parent[root1] = root2;
    }
    else{
        parent[root1] += parent[root2];
        parent[root2] = root1;
    }
}
```

## 例题
* leetcode 952 题中采用了 Find Union 的求解。但此题难点并不是采用并查集，而并查集的元素是什么，元素应该是具体数字，而并非序号，有利于减少除法运算量。采用 static 的数组存每个数的最大质因素，有利于减少除法运算数量。