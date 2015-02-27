title: VPS Debian Configuration Scripts
date: 2015-02-26 17:22:43
categories:
	- Study
tags:
	- VPS
	- OpenConnect
	- Shadowsocks
	- Hexo
	- Debian
	- Bash Script
---

VPS上Debian自动配置脚本
===

##介绍
本文记录了自己在Debian7系统的VPS上进行脚本自动配置的过程。
方便以后新装系统的配置工作。
通过在线脚本的执行，能够快速搭建VPN+Shadowsocks+Hexo的环境。

<!-- more -->

我在去年11月第一次尝试在VPS上部署Hexo网站以及配置Shadowsocks并[记录了下来](http://www.tecton69.com/2014/12/08/Deploy-hexo-site-on-VPS/)。
而由于手机没法方便地使用Shadowsock翻墙，需要配置VPN。我按照步骤在CentOS下配置VPN，结果还是连接失败后就搁置了…

有个学弟也需要VPN翻墙，我想到的是换到熟悉一点的Debian系统，但是好不容易把帐号，博客，翻墙等系统弄好了，对于我来说再去配置一遍环境是一个不小的工作量，于是一直没动手。

正好寒假里比较有空，于是上网搜了一下，通过比较我觉得目前用OpenConnect作VPN是一个不错的选择。它是思科的AnyConnect的开源实现，我对思科的产品印象还不错，因此就选它了。

##配置过程
VPS是在Bandwagon上购买的，它有方便的保存镜像功能，所以第一步是将原来的VPS保存为镜像。然后重装了`Debian 7 x86`的系统。

在重装的过程中，我将命令用脚本的形式保存下来，这样以后就方便很多了！对于修改文件的部分，采用了一个很简单的方法，就是把修改好的文件放在[Github项目](https://github.com/tecton/VPS-Debian7-Configuration)上。在脚本中下载下来替换掉，这样的好处是方便，而缺点就是会覆盖原有的配置。由于是自己用，而且这些脚本都是运行在新装的系统上的，所以暂不考虑通用性。

> **所以目标是装完系统后在root帐号下跑下面几个脚本就OK了！</br>**
根据功能将它们分割开来了，这样可以根据需要按照。

###帐号配置
`wget -q -O- https://raw.githubusercontent.com/tecton/VPS-Debian7-Configuration/master/user-config.sh | sh`

简要说明：</br>
1. 添加了tecton用户并加入sudoer
2. 下载ssh用的public key（自己电脑上的）
3. 配置sshd使得只能tecton用户通过public key登录，禁用root和密码登录。

###VPN配置
`wget -q -O- https://raw.githubusercontent.com/tecton/VPS-Debian7-Configuration/master/openconnect-config.sh | sh`

简要说明：</br>
1. 下载编译openconnect并配置证书、密码等等。
2. 配置iptables firewall。

参考教程：
http://luoqkk.com/openconnect-vpn-server-anyconnect-ocserv.html </br>
https://blog.phoenixlzx.com/2014/07/21/setup-openconnect-server-on-ubuntu/ </br>
http://bitinn.net/11084/

###Shadowsocks配置
由于涉及密码，只能手动配置= =
```
echo "deb http://shadowsocks.org/debian wheezy main" >> /etc/apt/sources.list;
apt-get update;
apt-get install shadowsocks-libev;
vim /etc/shadowsocks-libev/config.json #填入密码，端口等等
service shadowsocks-libev restart;
```

###个人网站配置
####第一部分
`wget -q -O- https://raw.githubusercontent.com/tecton/VPS-Debian7-Configuration/master/website-config.sh | bash`

简要说明：</br>
1. 安装nginx并配置网站目录。
2. 安装gitolite及hexo。
3. 重启。

####第二部分
gitolite需要clone admin的repo并进行配置。
最后通过添加hook使得提交上来的文件能够自动编译并部署。

**用git帐号登录**
```
git clone repositories/tecton-homepage.git/ homepage;
wget -q https://gist.githubusercontent.com/tecton/c98f564763704d24cf0e/raw/ae405515e5916555f7c498089854394a9521f921/post-receive.sh -O /home/git/repositories/tecton-homepage.git/hooks/post-receive;
chmod +x /home/git/repositories/tecton-homepage.git/hooks/post-receive;
```

##总结
对于Linux配置不太熟练的我来说有很多情况是Google查命令的，配置过程也会碰到很多小问题。以前总是配置完一遍就结束了，因此没法重复。这次通过脚本化将过程记录下来，看似简单的脚本却花了我不少时间…所幸有了开头以后就叠加就简单了。这次重装也体验到博客记录的好处，我从原来的文章回想起当时的过程，便于理清思路。

很多网上教程是指导如何一步一步做的，但跟下来很耗时，也会遇到各种问题。一些一键脚本也有失灵的时候，所以我才会选择做一个针对自己的脚本。虽然这只是自用的脚本，也希望给看到的人带来一些帮助。

PS：本文是上述脚本运行后成功上传的哈哈^.^