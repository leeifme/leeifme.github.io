---
title: Raspberry Pi — 新手入门操作
date: 2017-11-20 20:36:25
categories: 
- collect
tags:
- 玩转硬件
---

## 通过 SSH 访问树莓派

> SSH 的主要目的是用来取代传统的 telnet 和 R 系列命令远程登陆和远程执行命令的工具，实现对远程登陆和远程执行命令加密。同样通过 SSH 功能也能快速的打开树莓派得终端。

###**开启 SSH 服务**

- 树莓派系统一般都含有树莓派服务，因此不需要进行安装。在树莓派系统中打开命令行窗口，输入：sudo raspi-config 进入配置界面，将选中条选择到 “9 Advanced Options”（上下键移动选中条），回车。

![img](https://img-blog.csdn.net/20170629170451112)

- 选中 “A4 SSH”，回车，然后通过选中 “Yes”（左右键移动选中条）来开启 SSH 服务。

![img](https://img-blog.csdn.net/20170629170513134)

- SSH 服务开启完成

![img](https://img-blog.csdn.net/20170629170533361)

###**SSH 服务连接树莓派**

- 在电脑下载安装 “putty”，完成后打开软件，弹出配置界面，在主机名（Host Name）中输入树莓派对应 IP 地址、端口号，点击打开（弹出安全警告选择是）。

![img](https://img-blog.csdn.net/20170629172515159)

- 在进入登陆界面后，输入对应得树莓派账号、密码（默认账号：pi   密码：raspberry）即可连接到树莓派。

![img](https://img-blog.csdn.net/20170629172524083)

## 使用 VNC 远程登陆树莓派

> 使用树莓派的一种经典方式，是远程登陆进行操作。常用的方法有两种，一种是通过 SSH 协议，一种是使用 VNC 工具。这两种方法在 Raspbian（至迟 2017 年 3 月版）上都是集成的，比较明显的区别就是：SSH 使用终端的命令行进行操作，VNC 则具备图形界面。因此，比较而言 VNC 更适合初学者使用。本文对使用 VNC 远程登陆树莓派进行详细介绍。

###**准备**

树莓派的 Raspbian 系统，虽然集成了 VNC Server，默认却是不开启的。假设你想在你的 windows 计算机上使用 vnc 远程登陆树莓派，所需条件如下：

- 一台 windows 计算机
- 一台 Raspbian 系统的树莓派
- 局域网环境（确保 windows 计算机与树莓派可以互相连接）

###**开启 Raspbian 的 VNC Server**

在终端输入命令：

> sudo raspi-config

raspi-config 是 Raspbian 系统自带的配置工具，树莓派的很多必要功能都需要通过它进行设置。本文只对 VNC 方面的配置进行介绍。raspi-config 的界面如下： 
![1.raspi-config的界面](https://img-blog.csdn.net/20170418111459523?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk1MjgwNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
将光标移至第 5 项：Interfacing Options，并敲回车。 
![2.选择VNC选项](https://img-blog.csdn.net/20170418111659510?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk1MjgwNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
选择第 3 项：VNC，并敲回车。 
![3.确认开启](https://img-blog.csdn.net/20170418111834352?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk1MjgwNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
选择 “是”，并敲回车。 
![4.设置完成](https://img-blog.csdn.net/20170418110930894?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk1MjgwNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
设置结束，敲回车进行确认。 
通过以上操作，VNC Server 开启完毕。之后树莓派每次开机，VNC Server 都会自启动。这就确保了我们可以随时远程登陆树莓派。

###**连接网络并查询 IP 地址**

将你的树莓派连接到局域网中，有线网无线网均可。注意：并不是所有的树莓派都具备无线网卡，对于这部分树莓派，需要外接 USB 无线网卡才能连接无线网络。 
远程登陆树莓派，需要知道树莓派的 ip 地址。首先，在终端输入以下命令：

```shell
ifconfig
```

该命令用于查询或进行网络配置，与 windows 系统的 ipconfig 功能相似。以我的树莓派为例，敲回车后，输出如下： 
![ifconfig示例](https://img-blog.csdn.net/20170424092505680?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjk1MjgwNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
系统显示的信息中，左侧是网络设备类型及标号，右侧是网络设备的详细信息，其中的 inet addr 就是 ipv4 地址。我的树莓派有 3 个网络设备，其中 eth0 指的是有线网卡；lo 指的是回环接口，一般用于网络测试；wlan0 指的是无线网卡。因为我是用 wifi 连接网络的，因此查看 wlan0 右侧的 inet addr 为 192.168.155.2，这就是我的树莓派的 ip 地址。

##启用 LCD 显示屏

### 准备工作

> 1）安装好 raspbian 系统 
> 2）连接上 LCD 显示屏 
> 3）ssh 连接上树莓派

### 下载 LCD 显示屏驱动

```
wget -O LCD-show.tar.gz http://www.waveshare.net/w/upload/3/34/LCD-show-180331.tar.gz
```

### 安装显示屏驱动

```
// 1） 如果你要安装2.8inch 的屏就直接复制下面的内容：

sudo tar zxvf LCD-show.tar.gz
cd LCD-show/
sudo ./LCD28-show

// 2）如果你要安装3.5 inch 的屏就直接复制下面的内容：
sudo tar zxvf LCD-show.tar.gz
cd LCD-show/
sudo ./LCD35-show
```

###LCD 和 HDMI 相互切换

使用上面两种方法在正常使用 LCD 的情况下，外接 HDMI 是没有显示的，如需启用 HDMI 输出，需执行以下命令，树莓派会自动重启。再等待约 30 秒，HDMI 显示屏开始显示。

```
cd LCD-show/
./LCD-hdmi
```

如需切换回 LCD 显示方式，则需执行以下命令：

```
cd LCD-show/
./LCD35-show
```

###设置显示方向

安装完触摸驱动后，可以通过运行以下命令修改屏幕旋转方向。

- 旋转 0 度：

```
cd LCD-show/
./LCD35-show 0
```

- 旋转 90 度：

```
cd LCD-show/
./LCD35-show 90
```

- 旋转 180 度：

```
cd LCD-show/
./LCD35-show 180
```

- 旋转 270 度：

```
cd LCD-show/
./LCD35-show 270
```

## 文件共享 (samba)

### 首先更新源

```shell
sudo apt-get update
```

### 安装 samba 软件

```bash
sudo apt-get install samba samba-common-bin
```

###修改配置文件 /etc/samba/smb.conf

```shell
sudo nano /etc/samba/smb.conf
```

> 配置每个用户可以读写自己的 home 目录，在 “[homes]” 节中，把 “read only = yes” 改为 “read only = no”。

[![img](http://www.waveshare.net/study/data/attachment/portal/201612/21/102451jxcjzwk695he85ew.png)](http://www.waveshare.net/study/data/attachment/portal/201612/21/102451jxcjzwk695he85ew.png)[![img](http://www.waveshare.net/study/data/attachment/portal/201612/21/102451l2rpl6986uuaqta3.png)](http://www.waveshare.net/study/data/attachment/portal/201612/21/102451l2rpl6986uuaqta3.png)

###重启 samba 服务

```
sudo /etc/init.d/samba restart
```



###添加默认用户 pi 到 samba

```
sudo smbpasswd -a pi
```

[![img](http://www.waveshare.net/study/data/attachment/portal/201612/21/102452hv8v71b7dudvhx1b.png)](http://www.waveshare.net/study/data/attachment/portal/201612/21/102452hv8v71b7dudvhx1b.png)

输入密码确定即可。

[![img](http://www.waveshare.net/study/data/attachment/portal/201612/21/102453b9qo5zabps1v7bpp.png)](http://www.waveshare.net/study/data/attachment/portal/201612/21/102453b9qo5zabps1v7bpp.png)



###访问树莓派文件

> 使用文件浏览器打开 ip 地址 \\192.168.6.123\pi (ip 地址改为树莓派 IP 地址)，输入用户密码，则可以访问树莓派 home 目录。
>
> 在 windows 网络中你会发现多了台电脑 RASPBERRYPI，以后直接点击即可访问树莓派存储器, 或在windows 开始菜单中运行打开 \\RASPBERRYPI.

[![img](http://www.waveshare.net/study/data/attachment/portal/201612/21/102452pd9ja6jr4aapdi1v.png)](http://www.waveshare.net/study/data/attachment/portal/201612/21/102452pd9ja6jr4aapdi1v.png)

[![img](http://www.waveshare.net/study/data/attachment/portal/201612/21/102453s0tl0c1ltays1t0b.png)](http://www.waveshare.net/study/data/attachment/portal/201612/21/102453s0tl0c1ltays1t0b.png)