title: Add dvorak double pinyin in Rime
date: 2014-12-01 09:43:27
categories:
  - Study
tags:
  - Rime
  - Double Pinyin
  - Dvorak
---

鼠须管自定义德沃夏克双拼输入方案
============================

之前用的输入法是搜狗拼音，但是尝试了修改双拼映射之后没有成功，于是Mac下一直用着标准的拼音输入。
这次装了Rime之后发现了其定制的高自由度，在看了相关文档后不仅对输入法的整体结构有了一个大致的了解，也使得Mac上也能完成我独特的输入需求。

<!-- more -->

年初的时候我转用德沃夏克布局，拼音方法也尝试了一个网上某大牛制作的基于它的[双拼方案](http://zhangshenjia.com/event/dvorak-shuangpin/)。至今为止用下来的感觉还挺习惯的，并打算继续用下去，但是由于是自定义的键位布局，所有输入法自带的默认双拼里没有。
Windows下可以用谷歌拼音输入法自定义布局，而Mac下Rime就可以，不过是刚刚了解到。

Rime的文档齐全，在看了[如何自定义输入方案](https://code.google.com/p/rimeime/wiki/RimeWithSchemata)以及独特的[拼写运算](https://code.google.com/p/rimeime/wiki/SpellingAlgebra)之后根据默认提供的几套双拼方案改了一个自己使用的输入方案。

主要由以下代码完成声母韵母对键位的映射：

``` yaml
- xform/^([aoe].*)$/U$1/  # 添上固定的零聲母U
- xform/^zh/O/            # 替換聲母鍵，用大寫以防與原有的字母混淆
- xform/^ch/A/
- xform/^sh/E/
- xform/ei$/Q/            # 替換韻母鍵
```

由于有临时英文的输入需要，反转为拼音的配置删除了。

具体代码可见[Github](https://github.com/tecton/Rime-Dvorak-Double-Pinyin/tree/master)