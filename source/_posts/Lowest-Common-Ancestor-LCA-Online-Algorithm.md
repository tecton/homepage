title: Lowest Common Ancestor(LCA) Online Algorithm
date: 2015-01-02 16:06:01
categories:
- Study
- Algorithm
tags:
- LCA
- AtCoder
- 倍增法
---
LCA在线算法——倍增法
===

在AtCoder继续做题的过程中，又学习了一个新的算法。虽说是初学者的试题，不过最后一题大都是我没有见过的算法题。[这次我碰到的题目](http://abc014.contest.atcoder.jp/tasks/abc014_4)的关键点是LCA算法，即找到一棵树中的两个结点的最近父结点。

我起初的做法是用DFS遍历两个目标结点，同时记录下遍历过程中结点的深度。
当遍历到一个已经访问过的结点时，就输出它们的深度之和。可惜这么直观的算法运行结果势必超时，只能通过小数据集。

在看了解答后我仍然一头雾水，作者之提到了LCA算法，而我搜索到的LCA算法要么是离线处理大量请求的Tarjan算法、要么是在线的递归算法。前者实现起来太麻烦（我也没看懂。。）后者效率不高。

而我看了高手们提交的代码后，发现他们用的是清一色的方法。(也是醉了。。）
我也尝试了提交类似的算法（依照我自己的理解）。不过结果是MLE……

这让我好生苦恼！
终于在又搜索了一遍时发现了一个名字——倍增法。原来他们用的都是这个玩意儿！
<!-- more -->

这下终于弄懂那些奇妙的公式了。

倍增法的精妙之处在于每个结点都保存了它的父结点，爷爷结点，爷爷的爷爷结点……即`2^k`倍的祖先结点。
这使得每个结点的祖先结点数组只要开到`log(n)`就够了，基于这一点可以使得程序不会MLE。

算法步骤如下：

1. 生成邻接矩阵来保存每对边。
2. 选取一个顶点，通过DFS遍历来记录每个顶点的深度与父结点。
3. 遍历并记录每个结点`1...log(n)`层的祖先结点。</br>递推公式为`ancestor[a][b] = ancestor[ancestor[a][b - 1]][b - 1]`
4. 找到两个结点的最近公共祖先p
5. 得出结果为`depth[a] - depth[p] + depth[b] - depth[p] + 1`(+1是新加的边)

而计算LCA的过程与`2^k`倍祖先有着很大的关系。
假设a的深度是10，b的深度是15。
首先需要找到b在a同一层的结点。深度差为5，所以通过找b的父结点（深度变为14）再找4倍祖先就可以了。
它利用的特点是深度差一定用2进制数表示，所以每个结点保存的`2^k`倍祖先就够了。
代码如下：
```
if (depth[a] > depth[b]) {
    int tmp = a;
    a = b;
    b = tmp;
}
for (int i = 0; i < 20; ++i) {
    if (depth[b] - depth[a]>>i&1) {
        b = ancestor[b][i];
    }
}
```
a与b同层之后就直接向上找到共同的祖先结点即可。
加了注释的代码如下（参看了AtCoder上提交者的源码)

```cpp complete algorithm https://gist.github.com/tecton/73ef5f76a91c9717bce4 gist
//
//  main.cpp
//  14
//
//  Created by TangNing on 12/29/14.
//  Copyright (c) 2014 TangNing. All rights reserved.
//

#include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>
#include <vector>
#include <queue>
#include <map>
#include <math.h>

using namespace std;

// record adjacent matrix of graph
vector<vector<int> > graph;
// record 2^k ancestor of each node in tree
vector<vector<int> > ancestor;
// record the depth of each node from root (root has depth 0)
vector<int> depth;
int N, x, y, Q, a, b;

void DFS(int curNode, int parent, int d) {
    depth[curNode] = d;
    for (int i = 0; i < graph[curNode].size(); ++i) {
        int node = graph[curNode][i];
        if (node == parent) {
            continue;
        }
        // record father node( = 2^0 ancestor node)
        ancestor[node][0] = curNode;
        DFS(node, curNode, d + 1);
    }
}

int LCA(int a, int b) {
    if (depth[a] > depth[b]) {
        int tmp = a;
        a = b;
        b = tmp;
    }
    // move b to same depth as a
    for (int i = 0; i < 20; ++i) {
        if ((depth[b] - depth[a])>>i&1) {
            b = ancestor[b][i];
        }
    }
    if (b == a) {
        return a;
    }
    // move to LCA's child position
    for (int i = 19; i >= 0; --i) {
        if (ancestor[a][i] != ancestor[b][i]) {
            a = ancestor[a][i];
            b = ancestor[b][i];
        }
    }
    return ancestor[a][0];
}

int main(int argc, const char * argv[]) {
    // insert code here...
    cin >> N;
    graph = vector<vector<int> >(N, vector<int>(0));
    ancestor = vector<vector<int> >(N, vector<int>(20, -1));
    depth = vector<int>(N, 0);
    for (int i = 0; i < N - 1; ++i) {
        cin >> x >> y;
        x--;
        y--;
        graph[x].push_back(y);
        graph[y].push_back(x);
    }
    DFS(0, -1, 0);
    
    // calculate 2^k ancestor
    for (int j = 0; j < 20; ++j) {
        for (int i = 0; i < N; ++i) {
            if (ancestor[i][j] != -1) {
                ancestor[i][j + 1] = ancestor[ancestor[i][j]][j];
            }
        }
    }
    
    cin >> Q;
    for (int i = 0; i < Q; ++i) {
        cin >> a >> b;
        a--;
        b--;
        int p = LCA(a, b);
        cout << depth[a] + depth[b] - 2 * depth[p] + 1 << endl;
    }
    
    return 0;
}
```