---
title: "Refinitiv Elektron SDK - Java 使用"
layout: post
date: 2020-07-28 9:07
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Refinitv
- Elektron
- Data
- Finance
star: true
category: blog
author: joseph
description: Refinitiv Elektron SDK - Java
---

# 安装Gradle Build Tool
先安装openjdk
{% highlight shell %}
$ sudo yum install java-1.8.0-openjdk-devel
...
Complete!
$ java -version
openjdk version "1.8.0_262"
OpenJDK Runtime Environment (build 1.8.0_262-b10)
OpenJDK 64-Bit Server VM (build 25.262-b10, mixed mode)
{% endhighlight %}

去https://gradle.org/releases/查询gradle的最新版本，本文写作时最新版本是v6.5.1，这里我们用稳定版本v6.5

{% highlight shell %}
$ ls | grep gradle
gradle-6.5-bin.zip
$ sudo unzip -d /opt/gradle gradle-6.5-bin.zip # 将当前zip文件解压至/opt/gradle
$ ls /opt/gradle/gradle-6.5
LICENSE NOTICE README bin init.d lib
{% endhighlight %}

设置环境变量，将path环境变量设置为包括Gradle bin目录。在/etc/profile.d/目录下创建一个名为gradle.sh的新文件：

{% highlight shell %}
$ sudo touch /etc/profile.d/gradle.sh
$ sudo vi /etc/profile.d/gradle.sh

# 写入如下配置
export GRADLE_HOME=/opt/gradle/gradle-6.5
export PATH=${GRADLE_HOME}/bin:${PATH}

wq

# 通过chmod命令使脚本可执行
$ sudo chmod +x /etc/profile.d/gradle.sh

# 使用source命令加载环境变量
$ source /etc/profile.d/gradle.sh

# 验证gradle已经成功安装
$ gradle -v
Welcome to Gradle 6.5!
{% endhighlight %}

# ESDK Build System


