---
title: "RHEL 7.5（一）替换yum源以及安装GUI"
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
description: Redhat系统替换成CentOS的yum源，以及安装GUI
---

## 准备

要在新安装好的RHEL7.5上安装Vmware Workstation，需要提前做好如下准备：

0. 设置上网

1. 替换yum源（Redhat的yum需要付费，切换成CentOS的源）

2. 安装GUI

### 设置上网

{% highlight shell %}
>>>$ ls /etc/sysconfig/network-scripts/
ifcfg-enp0s31f6 # 这个就是我们网卡的名字
ifcfg-lo
...
>>>$ ls -l /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
-rw-r--r--
>>>$ sudo chmod o+w /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
>>>$ ls -l /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
-rw-r--rw-
{% endhighlight %}


修改ifcfg-enp0s31f6:
{% highlight shell %}
>>>$ vi /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
TYPE=Ethernet
...
# 以下为修改项
BOOTPROTO=static     # dhcp为动态分配ip
ONBOOT=yes           # 设置电脑启动时开启
IPADDR0=192.168.1.20 # fixed局域网ip，0可省略
GATEWAY0=192.168.1.1 # 网关，0可省略
PREFIX0=24           # 子网掩码，0可省略（也可以是NETMASK=255.255.255.0，换算成2进制是24个1）
DNS1=192.168.1.1     # DNS解释地址（通常与网关一致），不要忘记DNS后面有个1
ZONE=public          # 可能是optional的
{% endhighlight %}

按wq保存之后，记得将权限改回来：

{% highlight shell %}
>>>$ sudo chmod o-w /etc/sysconfig/network-scripts/ifcfg-enp0s31f6
{% endhighlight %}

之后reboot电脑，让ip生效。重启电脑后

{% highlight shell %}
>>>$ ip address   # 查看ip地址，看到刚刚的配置已经生效
1: lo: ...
2: enp0s31f6: ...
   ...
   inet 192.168.1.20/24 ...
>>>$ ping www.baidu.com # 试一试连外网，收到数据的话即ok
{% endhighlight %}

### 替换yum源

Redhat自带的yum需要付费，而CentOS的yum是免费使用的，Redhat和CentOS的内核是非常接近的，我们可以将其替换成CentOS的源（*注意*：这一步操作有一点危险，操作不当可能导致不能开机）

{% highlight shell %}
>>>$ rpm -qa | grep yum # 查询rpm库中的yum文件
yum-3.4.3-158.e17.noarch
yum-metadata-parser-1.1.4-10.e17.x86_64
yum-rhn-plugin-2.0.1-10.e17.noarch
>>>$ rpm -qa | grep yum | xargs rpm -e --nodeps # 切换到root用户，通过以下命令删除yum文件
{% endhighlight %}

去mirrors.aliyun.com/centos/7/os/x86_64/Packages/ 下载以下几个文件
* yum-3.4.3-167.el7.centos.noarch.rpm
* yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
* yum-plugin-fastestmirror-1.1.31-53.el7.noarch.rpm
* yum-utils-1.1.31-53.el7.noarch.rpm
* yum-rhn-plugin-2.0.1-10.el7.noarch.rpm
* python-urlgrabber-3.10-10.el7.noarch.rpm
* python-iniparse-0.4-9.el7.noarch.rpm

上传至RHEL服务器，按顺序进行安装
{% highlight shell %}
>>>$ sudo rpm -ivh python-iniparse-0.4-9.el7.noarch.rpm
>>>$ sudo rpm -ivh python-urlgrabber-3.10-10.el7.noarch.rpm
>>>$ sudo rpm -ivh yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
>>>$ sudo rpm -ivh yum-plugin-fastestmirror-1.1.31-53.el7.noarch.rpm (--force --nodeps)
>>>$ sudo rpm -ivh yum-utils-1.1.31-53.el7.noarch.rpm (--force --nodeps)
>>>$ sudo rpm -ivh yum-rhn-plugin-2.0.1-10.el7.noarch.rpm (--force --nodeps)
>>>$ sudo rpm -ivh yum-3.4.3-167.el7.centos.noarch.rpm (--force --nodeps)
{% endhighlight %}

通常直接这样会遇到报错"**.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY ..."，可以在上述命令后面加上 `--force --nodeps` 进行强制安装.

另一种方法是[用CentOS的签名去注册][1]
{% highlight shell %}
>>>$ cd /etc/pki/rpm-gpg/
>>>$ wget http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
>>>$ rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
{% endhighlight %}

修改yum源配置文件。先看看现有的配置：

{% highlight shell %}
>>>$ ls /etc/yum.repos.d/  # 空白目录，说明未有配置
{% endhighlight %}

接着下载阿里镜像到/etc/yum.repos.d/ 目录下：

{% highlight shell %}
>>>$wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
{% endhighlight %}

如果wget命令不可用，则可以在另外的机子下载好，然后通过sftp上载至服务器上。修改刚刚下载的文件 /etc/yum.repos.d/CentOS-Base.repo，将所有的$releasever 替换成7:

{% highlight shell %}
>>> vi /etc/yum.repos.d/CentOS-Base.repo

:%s/$releasever/7/g
{% endhighlight %}

最后，运行以下命令进行更新源
{% highlight shell %}
>>>$ yum clean all
>>>$ yum makecache
>>>$ yum update # 【注意】不要跑这句，yum update之后会mess up /boot/efi/EFI 里面的启动项
{% endhighlight %}

### 安装GUI

首先，经过上面更换yum源之后，应该已经可以通过以下命令查看可用的package groups：
{% highlight shell %}
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

{% highlight shell %}
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

现在，需要对grub进行修复（注意：在确认修复前，不要随便reboot，否则可能开不了机）
{% highlight shell %}
>>>$ yum update
>>>$ grubby --bootloader-probe
error opening /boot/grub/grub.cfg for read: no such file or directory
>>>$ ls /usr/lib/grub
i386-pc
>>>$ grub2-install --boot-directory=/boot --efi-directory=/boot/efi /dev/sda --target=i386-pc
...
grub2-install: error: will not proceed with blocklists.
>>>$ ls /boot/grub
splash.xpm.gz
>>>$ grub2-mkconfig -o /boot/grub/grub.cfg
....
done
>>>$ ls /boot/grub/
grub.cfg splash.xpm.gz
>>>$ groupby --bootloader-probe
grub2
>>>$ grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
{% endhighlight %}

如果幸运的话，就试试重启电脑吧：

{% highlight raw %}
>>>$ reboot
{% endhighlight %}


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
grub> linuxefi /vmlinuz-3.10.0-1127.13.1.el7.x86_64 root=/dev/mapper/rhel-root # 通过tab键补全
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

Reboot之后如果能够正常开机，显示图形界面的登陆界面，就说明一切已经ok了。下一篇将讲如何安装Vmware Workstation。

[1]: https://www.cnblogs.com/code1992/p/10735725.html
