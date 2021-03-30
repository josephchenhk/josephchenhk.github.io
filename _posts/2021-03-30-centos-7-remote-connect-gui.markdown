---
title: CentOS 7 远程连接GUI界面
---

# 安装vnc-server
有了桌面环境，一般可以通过云服务器厂商提供的web终端远程连接进入桌面。用web终端每次都需要打开浏览器，然后登陆云管理后台再连接，比较麻烦。我们采取直接从桌面客户端远程连接的方式，省去打开浏览器和登录云管理后台的操作。

远程桌面技术有多种，例如VNC, TeamViewer, RDP等，本文使用免费且广泛使用的VNC。

在服务器上先安装服务端(tigervnc是tightvnc的分支)：

{%highlight shell%}
> yum install -y tigervnc-server
{%endhighlight%}

接着复制一份VNC配置：

{%highlight shell%}
> cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
{%endhighlight%}

注意上述命令参数重的“@:1”，可以将数字1换成30000内的任意数字，“5900+数字”即为程序的显示（监听）端口，如"@:1"表示监听5901端口。

编辑该配置文件(`/etc/systemd/system/vncserver@:1.service`)，将文件内的<USER>替换成远程连接时的登录用户名（如果是root，注意将第二个<USER>所在行的"/home"移除掉）。配置示例：
	
{%highlight shell%}
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=simple

# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
ExecStart=/usr/bin/vncserver_wrapper <USER> %i
ExecStop=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'

[Install]
WantedBy=multi-user.target
{%endhighlight%}
	
接下来设置vnc连接密码：
	
{%highlight shell%}
> vncpasswd
{%endhighlight%}
	
注意连接密码与登录密码不同：连接密码用于显示远程桌面，登录密码用于用户登录系统。

设置好后，启动vnc服务：

	{%highlight shell%}
> systemctl daemon-reload
> systemctl start vncserver@:1
> systemctl enable vncserver@:1
	{%endhighlight%}

如果开启了防火墙，注意放行相应端口（阿里云的话在安全管理控制组进行设置）。

	
# 客户端连接

	服务端配置完毕，接下来用客户端连接。

vnc是免费技术，许多客户端都支持该协议。本文采用App Store上免费的“Remote Desktop - VNC”软件进行连接。

在输入框填写服务器地址：**ip:port**，其中ip是服务器的ip或域名，port是监听的端口，例如**5901**。输入后按回车，弹出密码输入框，输入vncpasswd设置的密码。密码正确的话就可以看到服务器的图形桌面。
