title: Bounded Knapsack Problem
date: 2014-12-28 10:52:23
categories:
- Study
- Algorithm
tags:
- Knapsack
- AtCoder
---

有界背包问题 例子说明
===

最近遇到[一道背包问题的变种](http://abc015.contest.atcoder.jp/tasks/abc015_4)，是有界背包问题，一下把我给卡住了。对于背包问题的算法还没有达到灵活运用的程度，于是重新看Wiki和网上有名的[背包问题九讲](http://love-oriented.com/pack/pack2rc.pdf)，将它弄懂了。

原始的背包问题只有1个限制——背包的容量，在此容量之内求得最大价值的物品集合。
其中又有0/1背包、完全背包之分；而对于背包容量又有是否“恰好装满”之分。

本题中的背包还有一个个数的限制，即N个图片中只能选取K个。这样的问题称为有界背包问题。
有界背包问题又是多维背包问题的一种，因此可以增加一个维度来解决问题。

对于背包问题的动态规划过程，使我卡住的就是状态的定义，只要能将这个弄懂了，那么状态转移方程也就自然出来了。

本题的状态定义为DP[i][j][k]。代表的意思是 **在前i个图片中，选取了k张图片，将它们贴到宽度为j的表格中所能产生的最大价值。**
<!-- more -->

有了它，分分钟列出状态转移方程：

```
A[i - 1]为第i张图片的宽度，B[i - 1]为第i张图片的的价值。
对于DP[i][j][k]，
DP[i - 1][j - A[i - 1]][k - 1] + B[i - 1] 为选择第i张图片
DP[i - 1][j][k])                          为不选择第i张图片
```

这样初步代码就出来了！
最外侧遍历N张图片，里面的2层循环遍历每种状态。最后找N张图片下选取任意张在任意宽度下的最大价值。
``` cpp original algorithm
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>

using namespace std;

int W, N, K;
int dp[51][10001][51];

int main(int argc, const char * argv[]) {
    cin >> W >> N >> K;
    int A[50], B[50];
    for (int i = 0; i < N; ++i) {
        cin >> A[i] >> B[i];
    }
    
    memset(dp, 0, sizeof(dp));
    for (int i = 1; i <= N; ++i) {
        for (int j = 0; j <= W; ++j) {
            for (int k = 0; k <= K; ++k) {
                dp[i][j][k] = dp[i - 1][j][k];
                if (j >= A[i - 1] && k - 1 >= 0) {
                    dp[i][j][k] = max(dp[i][j][k], dp[i - 1][j - A[i - 1]][k - 1] + B[i - 1]);
                }
            }
        }
    }

    int res = 0;
    for (int i = 0; i <= K; ++i) {
        for (int j = 0; j <= W; ++j) {
            res = max(res, dp[N][j][i]);
        }
    }
    
    cout << res << endl;
    
    return 0;
}
```

当然这样是比较朴素的解法，还有进一步的空间优化可做。
这个三维数组中第一维的i（即遍历每张图片）可以省略。因为它只会取上一张图片（即i-1）的状态数据。因此空间上可以压缩至2维，但是要采取从后向前的遍历方法。（为了避免覆盖数据，详见算法九讲）

这样代码就变成了如下形式：
```cpp save memory space

int dp[10001][51]; //2维
...
    for (int i = 1; i <= N; ++i) {
        for (int j = W; j >= 0; --j) {
            for (int k = K; k >= 0; --k) {
                if (j >= A[i - 1] && k - 1 >= 0) {
                    dp[j][k] = max(dp[j][k], dp[j - A[i - 1]][k - 1] + B[i - 1]);
                }
            }
        }
    }
...
```
当然代码还是可以优化的，我参考了排名榜上前几名的写法，进行了一下优化，比如判断j >= A[i - 1]之类的也可以放到for循环里面，最终代码如下：
```cpp final resolution https://gist.github.com/tecton/50c860275d867d835924 gist
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>
#include <vector>

using namespace std;

int W, N, K;

int main(int argc, const char * argv[]) {
    cin >> W >> N >> K;
    int A[50], B[50];
    vector<vector<int> > dp(W + 1, vector<int>(K + 1, 0));
    for (int i = 0; i < N; ++i) {
        cin >> A[i] >> B[i];
        for (int j = W; j >= A[i]; --j) {
            for (int k = K; k - 1 >= 0; --k) {
                dp[j][k] = max(dp[j][k], dp[j - A[i]][k - 1] + B[i]);
            }
        }
    }

    int res = 0;
    for (int i = 0; i <= K; ++i) {
        for (int j = 0; j <= W; ++j) {
            res = max(res, dp[j][i]);
        }
    }
    
    cout << res << endl;
    
    return 0;
}
```
至此完成！