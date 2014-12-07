title: Install Yosemite
tags:
  - mac
  - yosemite
categories:
  - Study
date: 2014-11-29 21:43:00
---

Yosemite系统出来有一段时间了，它还有个美丽的译名“优山美地”。我之前曾经尝试安装过，不过报了磁盘空间不够的错误。
因此我怀疑是Fusion Disk导致SSD上面的空间不够了，而它又没有这么智能将文件转移到HDD上。
 
目前网易音乐、印象笔记的界面也变为扁平化设计了，加之系统操作时已经不如以往流畅了，所以正好升级一把。
 
虽然Fusion Disk能够提供高速和大容量两大优点，但是使用久了之后仍然会有卡顿现象，尤其是在打开大图片的时候。
于是这次在备份了系统之后去除了Fusion Disk，恢复SSD + HDD。之前很少使用的资料手动搬运到HDD上。

步骤很方便:

1. 备份系统

2. 制作U盘安装盘

<!-- more -->
3. 进入Yosemite安装系统, 启动命令行。
删除Fusion逻辑磁盘组命令如下:
{% code lang:objc %}
diskutil cs list
diskutil coreStorage delete lvgUUID
{% endcode %}
lvgUUID是上条命令列出的logic volume group的UUID。

4. 重新格式化SSD与HDD。

5. 安装Yosemite至SSD。
 
在恢复软件的过程中发现有几个新的软件很好用：（另：app store已购软件的下载速度飞快）

* ShadowsocksX
之前用过cow，不过当时不知怎么的匹配规则有问题，所以以前用的是后台cow全部走代理，而chrome中用switchysharp进行URL规则判断。
现在想用原生safari浏览器，因此需要配置系统的pac文件。
此次ShadowsocksX会自动配置pac，上网也变得很方便了。

* 鼠须管
刚看到时感觉名字有些奇特，其实是开源的mac下输入法。
以前一直用搜狗输入法，但苦于双拼的键位方案自定义失败。看到这个输入法定制自由度很高，应该能成功。

* Chameleon SSD Optimizer
改装的SSD用的打开Trim的软件，同时会关闭系统的Kext。
