---
title: "远程连接至局域网"
layout: post
date: 2020-07-20 19:23
image: /assets/images/markdown.jpg
headerImage: false
tag:
- markdown
- elements
star: true
category: blog
author: joseph
description: Install vmware workstation in Linux Redhat 7.5
---

## 安装Vmware Workstation
* 下载：[WMware Workstation Pro 14.1.0 Linux 完美破解版 多语言版][1]
* 注册码：(适合VMware 2017 v14.x) FF31K-AHZD1-H8ETZ-8WWEZ-WUUVA / CV7T2-6WY5Q-48EWP-ZXY7X-QGUWD

上传至服务器之后，解压，得到
{% highlight shell %}
>>>$ chmod o+x VMware-Workstation-Full-14.1.0-7370693.x86_64.bundle
>>>$ ./WMware-Workstation-Full-14.1.0-7370693.x86_64.bundle
(填上上面的注册码)
# 安装好后打开，会提示安装gcc
>>>$ yum install gcc
# 装好gcc之后再次尝试打开，会提示Kernel Headers 3.10.0-1127.13.el7.x86_64 not found，查看现有kernels headers，如果没有的话安装
>>>$ ls /usr/src/kernels/
>>>$ yum install kernel-devel
# 装好kernel-devel之后再次尝试打开，会提示还有several packages need to compile，点击install，安装之后会提示失败，不要管它，点击cancel，之后再次打开，就ok了
{% endhighlight %}

下载CentOS-7-x86_64-Minimal-2003.iso并上传至服务器，打开vmware workstation，安装镜像。当安装完成之后，会提示说"Could not open /dev/vmmon: No such file or direcotry. Please make sure that the kernel module `vmmon` is loaded." 

这个错误大概是说vmmonitor和vmnet这两个模块没有经过签名认证，所以vmware出于安全的考虑，无法built。解决方案之一是重启系统，进入bios修改secure boot，将之改为disable，再重新尝试即可。

[1]: https://www.macxin.com/archives/3156.html
