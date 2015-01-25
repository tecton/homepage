title: Recursive Expression
date: 2015-01-25 18:30:29
categories:
- Study
- Algorithm
tags:
- 递归
- Matrix
- AtCoder
---

递推表达式的快速计算
===

[题目在此](http://abc009.contest.atcoder.jp/tasks/abc009_4)，看完解答之后恍然大悟。这道题其实并没有很难，但是没有掌握要领就很难做。一开始我就朝着简化表达式去想，然后卡住了。

这是一题根据递推公式求解第M项值的问题，因为每项计算都是与前k项有关，所以直接模拟计算肯定会来不及。

<!-- more -->

比较著名的递推式就是Fibonacci数列，每项是前两项只和。
通过这道题的讲解可以提炼出两点通用的解法：
1. 递归式的求解可以写成矩阵的形式，这样可以利用矩阵计算简化模拟过程。
2. 矩阵的运算可以利用2进制的特定进行中间项的累乘，将M阶的乘积降到O(log(M))时间计算。

对于位运算来说，“与操作”和“异或操作”与加、乘类似，因此适用矩阵的相乘。由于是位操作，单位矩阵上对角线的值都是bit全1的值，即UINT_MAX。

代码如下：
```cpp resolution https://gist.github.com/tecton/d0f0905ce9f05eff9219 gist
typedef vector<vector<unsigned int> > vvui;
 
// just like the matrix multiplication, & and ^ are similar
vvui AndXor(vvui a, vvui b) {
    int n = (int)a.size();
    vvui c(n, vector<unsigned int>(n, 0));
    for (int i = 0; i < n; ++i) {
        for (int j = 0; j < n; ++j) {
            for (int k = 0; k < n; ++k) {
                c[i][j] ^= (a[i][k] & b[k][j]);
            }
        }
    }
    return c;
}
 
int main(int argc, const char * argv[]) {
    int K, M;
    cin >> K >> M;
    vector<unsigned int> A(K);
    for (int i = 0; i < K; ++i) {
        cin >> A[i];
    }
    vvui C(K, vector<unsigned int>(K, 0));
    // build matrix
    for (int i = 0; i < K; ++i) {
        cin >> C[0][i];
    }
    for (int i = 1; i < K; ++i) {
        C[i][i - 1] = -1; // -1 is max of unsigned int
    }
    
    vvui Tr(K, vector<unsigned int>(K, 0));
    for (int i = 0; i < K; ++i) {
        Tr[i][i] = -1;
    }
    if (M <= K) {
        cout << A[M - 1] << endl;
        return 0;
    }
    M -= K;
    int m = 1;
    // shorten time to log(m)
    while (m <= M) {
        if (m & M) {
            Tr = AndXor(C, Tr);
        }
        m <<= 1;
        C = AndXor(C, C);
    }
    unsigned result = 0;
    for (int i = 0; i < K; ++i) {
        result ^= (Tr[0][i] & A[K - 1 - i]);
    }
    
    cout << result << endl;
    
    return 0;
}
```