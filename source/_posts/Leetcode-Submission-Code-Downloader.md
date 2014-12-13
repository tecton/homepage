title: Leetcode Submission Code Downloader
date: 2014-12-13 17:43:38
categories:
- Study
- Tool
tags:
- leetcode
- 爬虫
---
下载Leetcode提交代码的Python爬虫程序
===

在leetcode做题后我想将代码保存下来，每次手动操作的话容易遗漏等等，也挺麻烦的。</br>
于是写了一个Python的小程序，原理是模拟用户登录并将Sumission页面下的所有Accept的提交下载下来。</br>这样所有问题最后一次成功的提交代码会保存到本地，文件名为Problem的名字，如果有同名文件就会忽略。
<!-- more -->

##用到的Python库:

####模拟登录: Requests
####解析HTML: BeatifulSoup4
####密码输入: getpass
####代码匹配: re

##用法:
```python
./leetcodeDownloader.py <username>
```

##演示:
![](http://tecton.qiniudn.com/leetcode-downloader-demo.gif)

Github项目[地址在此](https://github.com/tecton/Leetcode-Submission-Downloader)