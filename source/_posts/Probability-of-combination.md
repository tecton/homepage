title: Probability of combination
date: 2015-01-14 22:49:15
categories:
- Study
- Algorithm
tags:
- Combination
- AtCoder
---

一道关于概率的算法题
===

在刷了这么多套AtCoder的题目之后已经发现规律了-。-就是前2题是刚入门编程的同学也能完成的超简单题目。</br>
而3、4两题难度飙升，考验智力。

这次的题目来自于初学者的[第11次比赛](http://abc011.contest.atcoder.jp)。

第三题感觉没有以前做的难，所以没看解答就能完成；而第四题的概率计算问题又被卡住了。
所幸解答过程非常详细，看完之后对于杨辉三角形（帕斯卡三角形）刮目相看。以前知道它有很多特性，却不知道可以这么用！

<!-- more -->

经过理解后，题目可以很容易地抽象成下面的问题：
> 在二维的平面上，从原点出发。每次可以沿着x或y轴移动1。</br>
> 假如共移动N次，最后停留在(X, Y)的概率是多少。

这道题是求概率的，那么最快的方法必然是数学的方法。
由于左右走会抵消，上下走也是同样的道理，所以如果要落在(X, Y)的话最终一定符合这样的等式:
> rightStep - leftStep = X</br>
> upStep - downStep = Y</br>
> leftStep + rightStep + upStep + downStep = N

这样的话概率的计算就容易的，利用排列组合的知识可以求解:
> 一共有4^N种可能</br>
> 到达(X, Y)的情况是</br>
> C(N, leftStep + rightStep) * C(leftStep + rightStep, leftStep) * C(upStep + downStep, upStep)</br>
> C(N, M)代表N种选M个的组合数

而它精妙的地方在于利用杨辉三角形计算C(N, M)的概率。形状如下:</br>
1</br>
0.5 0.5</br>
0.25 0.5 0.25</br>
...

计算方法为(上元素+左上元素) / 2

代码如下：
```cpp resolution https://gist.github.com/tecton/773bf67ad8db61f8017a gist
int main(int argc, const char * argv[]) {
    // insert code here...
#ifdef DEBUG
    freopen("/Users/tangning/in","r",stdin);
#endif
    int N;
    int D, X, Y;
    double combinationRate[1001][1000];
    cin >> N >> D >> X >> Y;
    X = fabs(X);
    Y = fabs(Y);
    int xnum = X / D;
    int ynum = Y / D;
    if (xnum * D != X || ynum * D != Y) {
        cout << "0.0" << endl;
        return 0;
    }
    memset(combinationRate, 0, sizeof(combinationRate));
    combinationRate[0][0] = 1;
    for (int i = 1; i <= N; ++i) {
        for (int j = 0; j <= i; ++j) {
            if (j - 1 >= 0) {
                combinationRate[i][j] += combinationRate[i - 1][j - 1];
            }
            combinationRate[i][j] += combinationRate[i - 1][j];
            combinationRate[i][j] /= 2.0;
        }
    }
    double result = 0;
    for (int i = 0; i <= N; ++i) {
        if (i < xnum || (N - i) < ynum)
            continue;
        if ((i + xnum) % 2 != 0)
            continue;
        if ((N - i + ynum) % 2 != 0)
            continue;
        int l = (i + xnum) / 2;
        int u = (N - i + ynum) / 2;
        result += combinationRate[N][i] * combinationRate[i][l] * combinationRate[N - i][u];
    }
    printf("%.15f\n", result);
    
    return 0;
}
```