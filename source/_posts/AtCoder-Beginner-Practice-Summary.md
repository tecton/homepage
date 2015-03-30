title: AtCoder Beginner Practice Summary
categories:
  - Study
  - Algorithm
tags:
  - AtCoder
date: 2015-03-07 16:06:21
---

AtCoder 初级题目整理与总结
===

##简介
经过漫长的时间，前几天终于把AtCoder上面的初级的19个练习做完了。
由于最后一题大部分是看解答的，今天准备总结一下，顺便再练手、重新加深印象。(内容比较多，看来得分好几次）

总的来说上面的题目前两道是非常容易的，而第三题难度适中，有时还能做出来，第四题比较考验脑力，直观的解法都会超时。

<!-- more -->

##整理
###1. [いもす法](http://www.tecton69.com/2015/03/07/imoz-method/)
出现好多次了，比较常用。

###2. 枚举搜索 
[2D](http://abc002.contest.atcoder.jp/tasks/abc002_4)

在个数比较小的情况下可以利用位操作可以进行搜索。

```
for (int i = 0; i < 1 << n; ++i) {
	for (int j = 0; j < n; ++j) {
		if (i & (1 << j))
			// jth element is selected.
	}
}
```

###3. 杨辉三角形（pascal三角形）
[3D](http://abc003.contest.atcoder.jp/tasks/abc003_4)
[11D](http://abc011.contest.atcoder.jp/tasks/abc011_4)

第一次发现杨辉三角形还能这么用……求组合的时候，可以利用杨辉三角形来构造表格，可以将乘除（阶乘公式）运算变化为加法。同样可以[计算概率问题](http://www.tecton69.com/2015/01/14/Probability-of-combination/)。

```
int pascal[1000][1000] = {};
for (int i = 0; i < n; ++i) {
	pascal[i] = pascal[0] = 1;
	for (int j = 1; j < i; ++j) {
		pascal[i][j] = pascal[i - 1][j] + pascal[i - 1][j - 1];
	}
}
// nCm = pascal[n][m]
```

###4. 排容原理 [wiki](http://zh.wikipedia.org/wiki/排容原理)
[3D](http://abc003.contest.atcoder.jp/tasks/abc003_4)
[5D](http://abc005.contest.atcoder.jp/tasks/abc005_4)
动态规划，用在计算面积的时候比较多，理解不是很难。

###5. 最长递增子串
[6D](http://abc006.contest.atcoder.jp/tasks/abc006_4)

可以用动态规划法来求解，做优化的话可以达到O(nlogn)。
这是叫填格子么？…

```
int a[] = {data};
// 利用二分查找的方式插入位置 O(logn)
int maxLength = 0;
int s[1000];
for (int i = 0; i < n; ++i) {
	int start = 0, end = n;
	// 假如严格递增
	while (start <= end) {
		int middle = start + (end - start) / 2;
		if (s[middle] < a[i])
			start = middle + 1;
		else
			end = middle - 1;
	}
	s[start] = a[i];
	maxLength = max(maxLength, start);
}
```

最后maxLength就是最长递增子串的长度。

###6. 数位DP
[7D](http://abc007.contest.atcoder.jp/tasks/abc007_4)

对每一位进行动态规划。没有掌握透彻，有待学习…

###7. 拼数学
[8C](http://abc008.contest.atcoder.jp/tasks/abc008_3)

此类题属于没想出来就做不出来

###8. 状态重复
[4C](http://abc004.contest.atcoder.jp/tasks/abc004_3)

可能描述起来有些困难。
一般是对一组数据进行操作，n次操作后可能会回到原来的某一状态。
4C就是这个类型，不过比较简单,重复30次后就会出现循环。