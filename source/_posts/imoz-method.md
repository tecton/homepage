title: imoz Algorithm Method
tags:
  - いもす法
  - AtCoder
date: 2015-03-07 19:56:20
categories:
  - Study
  - Algorithm
---

いもす法
===

##介绍
本文介绍算法练习中用到的一种方法。</br>
相关题目: [1D](http://abc001.contest.atcoder.jp/tasks/abc001_4)

我找了一圈没有看到中文对应的翻译，但找到一个专门介绍的[日文网站](http://imoz.jp/algorithms/imos_method.html)。
它是一种可以用累加操作代替填表的快速方法，比较适应于合并区间之类的题目。

<!-- more -->

##举个栗子：
有m个区间需要合并，取值范围1-n。</br>
假如1-4， 2-5， 6-8这三个区间需要合并。
###简单的方法:

用一个数组记录区间覆盖的情况。
第一遍填1-4, 第二遍填2-5， 第三遍填6-8。最后得到区间1-8。
这样如果区间很长的话会浪费很多时间在填重复的格子上，时间复杂度O(m*n)。
![简单方法步骤示意图](http://tecton.qiniudn.com/imoz-naive.png)
###いもす法
采用いもす法的话就能快很多，达到O(m + n)。</br>
它的优点在于填格子的扫描操作通过累加合并到了最后的一次操作。</br>
每遍将区间的开头位置计数+1，结尾位置+1计数-1。(可根据需要变化，也有结尾位置的计数-1，如1D)
![いもす法步骤示意图](http://tecton.qiniudn.com/imoz-illustrate.png)