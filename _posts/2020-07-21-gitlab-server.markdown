---
title: "GitLab服务器搭建"
layout: post
date: 2020-07-21 12:41
image: /assets/images/markdown.jpg
headerImage: false
tag:
- GitLab
- server
star: true
category: blog
author: joseph
description: Install Gitlab server in Centos 7
---

我们上一篇介绍了搭建Git服务器。要使用一个更现代、功能更全的Git服务器，我们可以试试GitLab
# 准备工作

1. 安装基础依赖

{% highlight shell %}
# 安装技术依赖
$ yum install -y curl policycoreutils-python openssh-server

# 启动ssh服务 & 设置为开机启动
$ systemctl enabale sshd
$ systemctl start sshd
{% endhighlight %}

2. 安装Postfix

Postfix是一个邮件服务器，GitLab发送邮件需要用到

{% highlight shell %}
# 安装postfix
$ yum install -y postfix

# 启动postfix并设置为开机启动
$ systemctl enable postfix
$ systemctl start postfix
{% endhighlight %}

3. 开放ssh以及http服务（80端口）

{% highlight shell %}
# 开放ssh、http服务
$ sudo firewall-cmd --add-service=ssh --permanent
$ sudo firewall-cmd --add-service=http --permanent

# 重载防火墙规则
$ sudo firewall-cmd --reload
{% endhighlight %}

# 部署过程

我们这里部署社区版gitlab-ce，如果要部署商业版可以把关键字替换成gitlab-ee

1. yum安装gitlab

{% highlight shell %}
$ curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash 
$ yum install -y gitlab-ce
{% endhighlight %}

2. 配置GitLab站点Url

将external_url修改为ip地址或域名：

{% highlight shell %}
# 修改配置文件
$ vi /etc/gitlab/gitlab.rb

# 配置首页地址 (注意有个等号=)
external_url='10.10.10.82'
{% endhighlight %}

3. 启动并访问GitLab

{% highlight shell %}
# 重新配置并启动
$ gitlab-ctl reconfigure

# 完成后将会看到如下输出
Running handlers complete
Chef Client finished, 556/1522 resources updated in 05 minutes 34 seconds
gitlab Reconfigured!
{% endhighlight %}

直接在浏览器输入ip地址"10.10.10.82"，可以访问gitlab。也可以通过修改本地host将域名指向服务器ip：

{% highlight shell %}
$ vi /etc/hosts

10.10.10.82 www.algo-git.com
{% endhighlight %}

以上修改后，重启network
{% highlight shell %}
$ systemctl restart network

# 或者通过nscd清除dns缓存
$ yum -y install nscd
$ systemctl status nscd
$ systemctl start nscd

$ nscd -i hosts
{% endhighlight %}

注意：以上修改/etc/hosts 只能对修改的本机生效。如果要对所有机器生效，还是要老老实实地去注册一个域名，然后通过dns解析吧。
