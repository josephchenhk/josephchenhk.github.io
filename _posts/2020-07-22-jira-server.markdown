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
{% endhighlight %}

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

# 这个错误原因是/var/lib/mysql的访问权限问题，将/var/lib/mysql的拥有者改为mysql:mysql
$ ls -l /var/lib/mysql
drwxr-xr-x. 2 mysql mysql 6 Jun 2 02:13 mysql
drwxr-x---. 2 mysql mysql 6 Jun 2 02:13 mysql-files

# 修改子目录的用户组
$ chown mysql:mysql /var/lib/mysql -R

# 再试一次登陆mysql
$ mysql -u root
{% endhighlight %}

成功以root用户进入mysql
{% highlight shell %}
# 创建jira数据库
mysql> CREATE DATABASE jira CHARACTER SET utf8 COLLATE utf8_bin;
Query OK, 1 row affected (0.00 sec)

# 授予jira用户权限
mysql> GRANT ALL PRIVILEGES ON jira.* TO 'jira'@'localhost' IDENTIFIED BY 'jirapass';
Query OK, 0 rows affected (0.00 sec)

# 测试jira连接mysql
$ exit
$ mysql -ujira -pjirapass
mysql>
{% endhighlight %}

# 安装jira

{% highlight shell %}
$ wget https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-7.4.1-x64.bin

$ ll atlassian-jira-software-7.4.1-x64.bin
-rw-rw-r--
$ chmod a+x atlassian-jira-software-7.4.1-x64.bin
$ sudo ./atlassian-jira-software-7.4.1-x64.bin # 记得要用sudo
Installation Directory: /opt/atlassian/jira
Home directory: /var/atlassian/application-data/jira
HTTP Port: 8080
RMI Port:8005
{% endhighlight %}

先关闭jira，将pojie文件atlassian-extras-3.2.jar 和 mysql-connector-java-5.1.39-bin.jar 两个文件复制至 /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/目录下

{% highlight shell %}
$ /etc/init.d/jira stop
$ lsof -i:8080
$ cp atlassian-extras-3.2.jar /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
$ cp mysql-connector-java-5.1.39-bin.jar /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
$ /etc/init.d/jira start
{% endhighlight %}

装好后如果在非本地机，通过ip:8080访问打不开，可能是由于防火墙的缘故，作如下处理

{% highlight shell %}
$ sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
$ service firewalld stop
$ service firewalld start
{% endhighlight %}

打开浏览器，输入ip:8080访问jira配置页面，按步骤操作如下：

![Markdown Image][1]
![Markdown Image][2]
![Markdown Image][3]
![Markdown Image][4]
![Markdown Image][5]
![Markdown Image][6]
![Markdown Image][7]
![Markdown Image][8]
![Markdown Image][9]
![Markdown Image][10]

[1]: /assets/images_in_posts/jira1.png
[1]: /assets/images_in_posts/jira2.png
[1]: /assets/images_in_posts/jira3.png
[1]: /assets/images_in_posts/jira4.png
[1]: /assets/images_in_posts/jira5.png
[1]: /assets/images_in_posts/jira6.png
[1]: /assets/images_in_posts/jira7.png
[1]: /assets/images_in_posts/jira8.png
[1]: /assets/images_in_posts/jira9.png
[1]: /assets/images_in_posts/jira10.png
