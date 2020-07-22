---
title: "Jira服务器搭建"
layout: post
date: 2020-07-22 15:41
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Jira
- server
star: true
category: blog
author: joseph
description: Install Jira server in Centos 7
---

# 安装jdk

去官网下载jdk-8u261-linux-x64.rpm

{% highlight shell %}
$ yum localinstall jdk-8u261-linux-x64.rpm
{% endhighlight %}

装之前，如果系统已经有open-jdk，需要先卸装

{% highlight shell %}
$ rpm -qa | grep java
$ rpm -e --nodeps [查看到的open-jdk] 
{% endhighlight %}

# 安装mysql

先检查系统是否有安装mysql

{% highlight shell %}
$ rpm -qa | grep mysql
{% endhighlight %}

下载mysql的repo源

{% highlight shell %}
$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
{% endhighlight % }

安装mysql-community-release-el7-5.noarch.rpm包：

{% highlight shell %}
$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
{% endhighlight %}

安装完这个包之后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo, /etc/yum.repos.d/mysql-community-source.repo

我们便可以运行下面命令去安装mysql

{% highlight shell %}
$ sudo yum install mysql-server
{% endhighlight %}

安装完成后，再次查看mysql

{% highlight shell %}
$ rpm -qa | grep mysql
mysql-community-libs-5.6.49-2.el7.x86_64
mysql-community-release-el7-5.noarch
mysql-community-common-5.6.49-2.el7.x86_64
mysql-community-client-5.6.49-2.el7.x86_64
mysql-community-server-5.6.49-2.el7.x86_64
{% endhighlight %}

# 启动mysqld

在进行mysql配置之前，先要启动mysql server：

{% highlight shelll %}
$ service mysqld start
{% endhighlight %}

# 设置mysql

首先要重置密码

{% highlight shell %}
$ mysql -u root
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)

# 这个错误原因是/var/lib/mysql的访问权限问题，将/var/lib/mysql的拥有者改为others
$ ls -l /var/lib/mysql
drwxr-xr-x. 2 mysql mysql 6 Jun 2 02:13 mysql
drwxr-x---. 2 mysql mysql 6 Jun 2 02:13 mysql-files

# 修改子目录的用户组
$ chown mysql:mysql /var/lib/mysql -R

# 再试一次登陆mysql
$ mysql -u root
{% endhighlight %}

192.168.2.24为Git所在服务器ip，需要将其改为你自己的服务器ip。这样我们的Git服务器就安装完成了。
