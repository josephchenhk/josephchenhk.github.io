---
title: "Git服务器搭建"
layout: post
date: 2020-07-21 9:41
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Git
- server
star: true
category: blog
author: joseph
description: Install Git server in Centos 7
---

# 安装Git

{% highlight shell %}
$ yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
$ yum install git
{% endhighlight %}

接下来我们创建一个git用户组和git用户，用来运行git服务

{% highlight shell %}
$ groupadd git
$ useradd git -g git
{% endhighlight %}

# 创建证书登陆

收集需要登陆的用户的公钥，公钥位于id_rsa.pub文件中，把我们的公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个。如果没有该文件就创建一个：

{% highlight shell %}
$ cd /home/git/
$ mkdir .ssh
$ chmod 755 .ssh
$ touch .ssh/authorized_keys
$ chomod 644 .ssh/authorized_keys
{% endhighlight %}

# 初始化Git仓库

首先我们选定一个目录作为Git仓库，假定是/home/git/gitrepo/algo.git，在/home/git/gitrepo目录下输入命令

{% highlight shell %}
$ cd /home/git/
$ mkdir gitrepo
$ chown git:git gitrepo/
$ cd gitrepo

$ git init --bare algo.git
{% endhighlight %}

以上命令创建了一个空仓库，服务器上的仓库通常以.git结尾。然后，把仓库所属用户改为git：

{% highlight shell %}
$ chown -R git:git algo.git
{% endhighlight %}

# 克隆仓库

{% highlight shell %}
$ git clone git@192.168.2.24:/home/git/gitrepo/algo.git
{% endhighlight %}

192.168.2.24为Git所在服务器ip，需要将其改为你自己的服务器ip。这样我们的Git服务器就安装完成了。
