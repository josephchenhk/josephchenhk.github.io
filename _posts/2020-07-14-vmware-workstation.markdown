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
IPADDR0=192.168.1.20 # fixed局域网ip，0可省略
GATEWAY0=192.168.1.1 # 网关，0可省略
PREFIX0=24           # 子网掩码，0可省略（也可以是NETMASK=255.255.255.0，换算成2进制是24个1）
DNS1=192.168.1.1     # DNS解释地址（通常与网关一致），不要忘记DNS后面有个1
ZONE=public          # 可能是optional的
{% endhighlight %}

按wq保存之后，记得将权限改回来：

{% highlight raw %}
>>>$ sudo chmod o-w /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
{% endhighlight %}

之后reboot电脑，让ip生效。重启电脑后

{% highlight raw %}
>>>$ ip address   # 查看ip地址，看到刚刚的配置已经生效
1: lo: ...
2: enp0s31f6: ...
   ...
   inet 192.168.1.20/24 ...
>>>$ ping www.baidu.com # 试一试连外网，收到数据的话即ok
{% endhighlight %}

### 替换yum源

{% highlight raw %}
>>>$ rpm -qa | grep yum # 查询rpm库中的yum文件
yum-3.4.3-158.e17.noarch
yum-metadata-parser-1.1.4-10.e17.x86_64
yum-rhn-plugin-2.0.1-10.e17.noarch
>>>$ rpm -qa | grep yum | xargs rpm -e --nodeps # 切换到root用户，通过以下命令删除yum文件
{% endhighlight %}

去mirrors.aliyun.com/centos/7/os/x86_64/Packages/ 下载以下几个文件
yum-3.4.3-167.el7.centos.noarch.rpm
yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
yum-plugin-fastestmirror-1.1.31-53.el7.noarch.rpm
yum-utils-1.1.31-53.el7.noarch.rpm
yum-rhn-plugin-2.0.1-10.el7.noarch.rpm
python-urlgrabber-3.10-10.el7.noarch.rpm
python-iniparse-0.4-9.el7.noarch.rpm

上传至RHEL服务器，按顺序进行安装
{% highlight raw %}
>>>$ sudo rpm -ivh python-iniparse-0.4-9.el7.noarch.rpm
>>>$ sudo rpm -ivh python-urlgrabber-3.10-10.el7.noarch.rpm
>>>$ sudo rpm -ivh yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
>>>$ sudo rpm -ivh yum-plugin-fastestmirror-1.1.31-53.el7.noarch.rpm
>>>$ sudo rpm -ivh yum-utils-1.1.31-53.el7.noarch.rpm
>>>$ sudo rpm -ivh yum-rhn-plugin-2.0.1-10.el7.noarch.rpm
>>>$ sudo rpm -ivh yum-3.4.3-167.el7.centos.noarch.rpm
{% endhighlight %}

通常直接这样会遇到报错"**.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY ..."，可以在上述命令后面加上 `--force --nodeps` 进行强制安装.

另一种方法是[用CentOS的签名去注册][6]
{% highlight raw %}
>>>$ cd /etc/pki/rpm-gpg/
>>>$ wget http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
>>>$ rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
{% endhighlight %}

修改yum源配置文件。先看看现有的配置：

{% highlight raw %}
>>>$ ls /etc/yum.repos.d/  # 空白目录，说明未有配置
{% endhighlight %}

接着下载阿里镜像到/etc/yum.repos.d/ 目录下：

{% highlight raw %}
>>>$wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
{% endhighlight %}

如果wget命令不可用，则可以在另外的机子下载好，然后通过sftp上载至服务器上。修改刚刚下载的文件 /etc/yum.repos.d/CentOS-Base.repo，将所有的$releasever 替换成7:

{% highlight raw %}
>>> vi /etc/yum.repos.d/CentOS-Base.repo

:%s/$releasever/7/g
{% endhighlight %}

最后，运行以下命令进行更新源
{% highlight raw %}
>>>$ yum clean all
>>>$ yum makecache
>>>$ yum update # 【注意】不要跑这句，yum update之后会mess up /boot/efi/EFI 里面的启动项
{% endhighlight %}

### 安装GUI

首先，经过上面更换yum源之后，应该已经可以通过以下命令查看可用的package groups：
{% highlight raw %}
>>>$ yum group list
Loaded plugins: fastestmirror, ...
...
Available Environment Groups
 ...
 Server with GUI
 GNOME Desktop
 ...
Available Groups:
 ...
 Graphical Administration Tools
 ...
Done

# 如果是Centos 7:
>>>$ yum groupinstall "GNOME Desktop" "Graphical Adminstration Tools"

# 如果是RHEL 7:
>>>$ yum groupinstall "Server with GUI"

# 令系统在启动后启用GUI（level5是图形界面，原来文字界面应该是level3）
>>>$ ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target

# 千万不要重启电脑！
【！！】>>>$ reboot
{% endhighlight %}

重启电脑之前，先检查以下启动项是否已经被污染了：

{% highlight raw %}
>>>$ ls /etc/efi/EFI
BOOT redhat CentOS # 多了一个CentOS，grub已经mess up，如果此时reboot，会导致不能进入系统
>>>$ ls /etc/efi/EFI/redhat
fonts grub.cfg grubenv grubx64.efi
>>>$ ls /etc/efi/EFI/centos
BOOT.CSV BOOTX64.CSV fw fwupia32.efi fwupx64.efi MokManager.efi shim.efi shimx64-cemtps.efi shimx64.efi
# grub.cfg文件在redhat下，但其实系统需要到/etc/efi/EFI/centos目录下面进行启动，所以要在该目录重建grub
>>>$ grub-mkconfig -o /boot/efi/EFI/centos/grub.cfg
>>>$ ls /etc/efi/EFI/centos
... grub.cfg # 已经生成grub.cfg文件
{% endhighlight %}

现在，试一试重启




我们需要修复grub

{% highlight raw %}
# 进入grub
grub> ls (hd1,1)/efi  # 找到efi文件夹
./ ../ redhat/ boot/ centos/
grub> ls (hd1,2)/     # 找到kernel镜像所在位置
... vmlinuz-3.10.0-1127.13.1.el7.x86_64 initramfs-3.10.0-1127.13.1.el7.x86_64.img...
grub> set pager=1
grub> cat (hd1,1)/efi/redhat/grub.cfg
(找出以下两句话：)
linuxefi /vmlinuz-3.10.0-1127.13.1.el7.x86_64 root=/dev/mapper/rehel-root ro crashkernel=auto rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet LANG=en_US.UTF-8
initrdefi /initramfs-3.10.0-1127.13.1.el7.x86_64.img

grub> GRUB_PRELOAD_MODULES="lvm"
grub> set root=(hd1,2)  # kernel镜像所在位置
grub> linuxefi /vmlinuz-3.10.0-1127.13.1.el7.x86_64 # 通过tab键补全
grub> initrdefi /initramfs-3.10.0-1127.13.1.el7.x86_64.img
grub> boot
{% endhighlight %}

如果此方法不能正常启动linux，则可以找到u盘镜像所在位置，然后重装系统了：
{% highlight raw %}
grub> ls (hd0,4)/
... images/ ...
grub> GRUB_PRELOAD_MODULES="lvm"
grub> set root=(hd0,4)
grub> linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:/dev/sdb4 quiet # 注意这里要指定根目录
grub> initrdefi /images/pxeboot/initrd.img
grub> boot
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
[6]: https://www.cnblogs.com/code1992/p/10735725.html
