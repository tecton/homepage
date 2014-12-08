title: Deploy hexo site on VPS
date: 2014-12-08 12:41:41
categories:
  - Study
tags:
  - VPS
  - Gitolite
  - git hook
---

在VPS上自动部署Hexo网站
=====================

##介绍
- - -
关于博客的框架我在几经尝试之后选了Hexo，用Markdown来写文章确实很方便。
我在最近买了搬瓦工的VPS，虽然内存不大，但是Hexo生成的是静态的页面，VPS足够host一个静态网站了。

然而对于写的新文章如何部署到服务器上是一个普遍会遇到的问题。
当然最简单的方法是手动拷贝静态文件到服务器上，但是这样操作麻烦又容易出错。

我想到的关键词叫“持续集成”，但是它的范围很大，这里还是叫自动部署吧。
通过搜索与尝试成功地完成了这个过程。（嗯，这样以后就可以专注于博客内容了……）

知识点 | 效果
------|-----
Git command|exp+3
Gitolite|exp+5
Nginx|exp+5
Linux用户权限管理|exp+10
Linux ssh登录|exp+10

<!-- more -->
关于整个过程我的**目标**是这样的：
1. 本地写文章
2. git push到服务器
3. Github上有备份，网站能自动更新。

没有直接用Github的Page是因为它有时候访问不了，而VPS可以用来科学上网顺带host一下。

##解决方案
- - -
那么这个过程最终解决方案是这样的：
* VPS上Nginx用作静态网页HTTP服务器
* VPS建立一个私有的Git仓库
* 本地同时向私有Git仓库及Github提交
* VPS上的Git仓库利用hook生成页面文件及部署

##具体说明
- - -
由于涉及的东西比较多，故依次序分别说明。
###Linux ssh登录
远程登录用密码有两个缺点：麻烦和不安全，别人也可以尝试输入密码。
而ssh key的方式就比较方便了，将本地的RSA公钥放在服务器上就行了。

在失败了几次之后我总结了以下几点：
1. 在/etc/ssh/sshd_config可以配置：
 * PermitRooLogin no/yes #是不是使得root能登录，我设成no
 * AllowUsers 这里本来是admin，我改成了自己建的用户名
2. 用户目录下~/.ssh/authorized_keys里面记录了你的[公钥](https://help.github.com/articles/generating-ssh-keys/)，有了这个才能登录
3. ~/.ssh/authorized_keys的权限问题
 ```sh
 chmod 700 ~/.ssh
 chmod 600 ~/.ssh/authorized_keys
 ```

###Nginx 目录配置
Nginx同样有目录权限的问题，Nginx启动之后我有1个master，2个worker。worker的用户为nginx。

因此我将网站的根目录所在组设为一个新的组www-data，且组的权限为读和执行
```sh
chown tecton:www-data /path/of/nginx/website
chmod 750 /path/of/nginx/website
```
将nginx和自己2个用户添加到www-data组
```sh
usermod -a -G www-data nginx
usermod -a -G www-data tecton
```
这些操作意味着Nginx进程有权限读取网站文件，否则出现的就是403 Forbidden页面。

###Gitolite建立私有Git仓库
[Gitolite的文档](http://gitolite.com/gitolite/install.html)非常详细。
我遵循了它一个非常方便的配置方法，新增了一个用户git。
在ssh配置AllowUsers中加入git的授权登录。
而authorized_keys文件需要由Gitolite生成。

比较独特的是新建一个Git repo并不是手动创建的，而是需要在gitolite-admin这个项目中配置，这样权限分配等都有一个统一的管理。

我在conf/gitolite.conf中加入了新的repo。

在本地可以通过`git clone`复制服务器上的项目。
####同时向Github提交
在代码提交的时候，需要同时向VPS及Github提交，一个简单的方法就是在.git/config中新增Github的地址:
```
[remote "origin"]
     url = git@github:xxxx.git
     url = git@vps:xxxx.git
     ...
```
###利用Git hook自动部署
Git hook可以在Git操作前后执行一些命令，我修改了网上的[一份优秀的代码](http://icyleaf.com/2012/03/apps-auto-deploy-with-git/)，加入了`hexo generate`命令及将生成的页面文件复制到网站目录的操作。（VPS上需要安装nodejs以及hexo，我使用了nvm）

整个流程是：
1. 本地执行`git push origin remote`
2. 检测到commit的信息里面有关键字`[deploy]`
3. 本地的项目执行pull（因为服务器上是bare repo，没有项目文件）
4. 运行hexo generate生成静态页面
5. 修改静态页面权限
6. 将静态页面复制到Nginx网站目录

为了方便起见，生成的文件对所有用户都有R+X的权限。
[代码在此](https://gist.github.com/tecton/c98f564763704d24cf0e)：
```sh
#!/bin/sh
#
# git autodeploy script when it matches the string "[deploy]"
#
# @author    icyleaf <icyleaf.cn@gmail.com>
# @link      http://icyleaf.com
# @version   0.1
#
# @modifier  tecton <tecton69@gmail.com>
# @link      http://tecton69.com
#
# Usage:
#       1. put this into the post-receive hook file itself below
#       2. change path to your directory.
#       3. `chmod +x post-recive` 
#       4. Done!
 
# Check the remote git repository whether it is bare
IS_BARE=$(git rev-parse --is-bare-repository)
if [ -z "$IS_BARE" ]; then
	echo >&2 "fatal: post-receive: IS_NOT_BARE"
	exit 1
fi
 
# Get the latest commit subject
SUBJECT=$(git log -1 --pretty=format:"%s")
 
# Deploy the HEAD sources to publish
IS_PULL=$(echo "$SUBJECT" | grep "\[deploy\]")
if [ -z "$IS_PULL" ]; then
	echo >&2 "tips: post-receive: IS_NOT_PULL"
	exit 1
fi
 
# Check the deploy dir whether it exists
DEPLOY_DIR=/home/git/homepage
if [ ! -d $DEPLOY_DIR ] ; then
	echo >&2 "fatal: post-receive: DEPLOY_DIR_NOT_EXIST: \"$DEPLOY_DIR\""
	exit 1
fi
 
# Check the deploy dir whether it is git repository
#
#IS_GIT=$(git rev-parse --git-dir 2>/dev/null)
#if [ -z "$IS_GIT" ]; then
#	echo >&2 "fatal: post-receive: IS_NOT_GIT"
#	exit 1
#fi
 
# Goto the deploy dir and pull the latest sources
cd $DEPLOY_DIR
#env -i git reset --hard
env -i git pull

# generate static site using hexo and copy it to destination directory.
SITE_DIR=/home/tecton/www.tecton69.com/public
source /home/git/.nvm/nvm.sh
nvm use 0.10
hexo generate
chmod -R 755 public/*
cp -rf public/* $SITE_DIR
```