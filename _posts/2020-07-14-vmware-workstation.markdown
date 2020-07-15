---
title: "Vmware Workstation"
layout: post
date: 2020-07-14 22:44
image: /assets/images/markdown.jpg
headerImage: false
tag:
- markdown
- elements
star: true
category: blog
author: joseph
description: Install vmware workstation in Linux
---

## 准备

在装好的RHEL7.5上安装Vmware Workstation，需要提前做好如下准备：

0. 设置上网

1. 替换yum源（Redhat的yum需要付费，切换成CentOS的源）

2. 安装GUI

### 设置上网

{% highlight raw %}
>>>$ ls /etc/sysconfig/network-scripts/
ifcfg-enp0s31f6
ifcfg-lo
...
>>>$ ls -l /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
-rw-r--r--
>>>$ sudo chmod o+w /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
>>>$ ls -l /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
-rw-r--rw-
{% endhighlight %}

修改ifcfg-enp0s31f6:
{% highlight raw %}
>>>$ vi /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
TYPE=Ethernet
...
# 以下为修改项
BOOTPROTO=static     # dhcp为动态分配ip
ONBOOT=yes
IPADDR0=192.168.1.20 # fixed局域网ip
GATEWAY0=192.168.1.1 # 网关
PREFIX0=24           # 子网掩码（也可以是NETMASK=255.255.255.0）
DNS1=192.168.1.1     # DNS解释地址（通常与网关一致）
ZONE=public          # 可能是optional的
{% endhighlight %}

### 替换yum源

{% highlight raw %}
>>>$ rpm -qa | grep yum # 查询rpm库中的yum文件
yum-3.4.3-158.e17.noarch
yum-metadata-parser-1.1.4-10.e17.x86_64
yum-rhn-plugin-2.0.1-10.e17.noarch
>>>$ rpm -qa | grep yum | xargs rpm -e --nodeps # 切换到root用户，通过以下命令删除yum文件
{% endhighlight %}


This note **demonstrates** some of what [Markdown][1] is *capable of doing*.

And that's how to do it.

{% highlight html %}
This note **demonstrates** some of what [Markdown][some/link] is *capable of doing*.
{% endhighlight %}

---

## Headings

There are six levels of headings. They correspond with the six levels of HTML headings. You've probably noticed them already in the page. Each level down uses one more hash character. But we are using just 4 of them.

# Headings can be small

## Headings can be small

### Headings can be small

#### Headings can be small

{% highlight raw %}
# Heading
## Heading
### Heading
#### Heading
{% endhighlight %}

---

## Lists

### Ordered list

1. Item 1
2. A second item
3. Number 3

{% highlight raw %}
1. Item 1
2. A second item
3. Number 3
{% endhighlight %}

### Unordered list

* An item
* Another item
* Yet another item
* And there's more...

{% highlight raw %}
* An item
* Another item
* Yet another item
* And there's more...
{% endhighlight %}

---

## Paragraph modifiers

### Quote

> Here is a quote. What this is should be self explanatory. Quotes are automatically indented when they are used.

{% highlight raw %}
> Here is a quote. What this is should be self explanatory.
{% endhighlight raw %}

---

## URLs

URLs can be made in a handful of ways:

* A named link to [Mark It Down][3].
* Another named link to [Mark It Down](https://google.com/)
* Sometimes you just want a URL like <https://google.com/>.

{% highlight raw %}
* A named link to [MarkItDown][3].
* Another named link to [MarkItDown](https://google.com/)
* Sometimes you just want a URL like <https://google.com/>.
{% endhighlight %}

---

## Horizontal rule

A horizontal rule is a line that goes across the middle of the page.
It's sometimes handy for breaking things up.

{% highlight raw %}
---
{% endhighlight %}

---

## Images

Markdown can also contain images. I'll need to add something here sometime.

{% highlight raw %}
![Markdowm Image][/image/url]
{% endhighlight %}

![Markdowm Image][5]

*Figure Caption*?

{% highlight raw %}
![Markdowm Image][/image/url]
<figcaption class="caption">Photo by John Doe</figcaption>
{% endhighlight %}

![Markdowm Image][5]
<figcaption class="caption">Photo by John Doe</figcaption>

*Bigger Images*?

{% highlight raw %}
![Markdowm Image][/image/url]{: class="bigger-image" }
{% endhighlight %}

![Markdowm Image][5]{: class="bigger-image" }

---

## Code

A HTML Example:

{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <h1>Just a test</h1>
</body>
</html>
{% endhighlight %}

A CSS Example:

{% highlight css %}
pre {
    padding: 10px;
    font-size: .8em;
    white-space: pre;
}

pre, table {
    width: 100%;
}

code, pre, tt {
    font-family: Monaco, Consolas, Inconsolata, monospace, sans-serif;
    background: rgba(0,0,0,.05);
}
{% endhighlight %}

A JS Example:

{% highlight js %}
// Sticky Header
$(window).scroll(function() {

    if ($(window).scrollTop() > 900 && !$("body").hasClass('show-menu')) {
        $('#hamburguer__open').fadeOut('fast');
    } else if (!$("body").hasClass('show-menu')) {
        $('#hamburguer__open').fadeIn('fast');
    }

});
{% endhighlight %}

[1]: https://daringfireball.net/projects/markdown/
[2]: https://www.fileformat.info/info/unicode/char/2163/index.htm
[3]: https://daringfireball.net/projects/markdown/basics
[4]: https://daringfireball.net/projects/markdown/syntax
[5]: https://kune.fr/wp-content/uploads/2013/10/ghost-blog.jpg
