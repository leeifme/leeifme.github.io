---
title: vscode + sftp 开发环境同步差异文件
date: 2018-09-15 14:15:15
categories: 
tags:
- 百宝箱
- 安利系列
---

## 前言

{{< alert theme="success" >}}
解决需求：
本地是 win10 系统，代码需要在 linux 下跑，又不想装虚拟机或双系统；
所以，项目用到连接远程测试服务器进行开发联调，需要安装 SFTP/FTP 的扩展插件才能同步代码；
还有其他实现方法，如，Git 工作流、winscp、rzlz 等，但大都不太灵活，甚至麻烦
{{< /alert >}}


## CentOS 7 配置使用 SFTP 服务器

### 何为 SFTP？

SFTP，即 SSH 文件传输协议（ SSH File Transfer Protocol ），或者说是安全文件传输协议（ Secure File Transfer Protocol ）。SFTP 是一个独立的 SSH 封装协议包，通过安全连接以相似的方式工作。它的优势在于可以利用安全的连接传输文件，还能浏览本地和远程系统上的文件系统。

在很多情况下，SFTP 都比 FTP 更可取，因为它具有最基本的安全特性和能利用 SSH 连接的能力，FTP 是一种不安全的协议，只能在有限的情况下或在您信任的网络上使用。

**先决条件：**

服务器 `OpenSSH-Server` 版本最低 4.8p1，因为配置权限需要版本添加的新配置项 `ChrootDirectory` 来完成。

如何查看 OpenSSH 版本，命令如下：

```shell
$ ssh -V
OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017
```

### 创建用户信息

添加用户组：

```shell
$ groupadd sftp
```

添加用户：

```shell
$ useradd -g sftp -s /sbin/nologin -M leeifme
```

参数注解：

```shell
-g              # 加入用户组
-s              # 指定用户登入后所使用的shell
/sbin/nologin   # 用户不允许登录
-M              # 不要自动建立用户的登入目录
```

设置用户密码：

```shell
$ passwd leeifme
```

### 创建用户目录并设置权限

创建 sftp 主目录：

```sh
$ mkdir /home/danaos
```

设置 sftp 主目录权限：

```sh
$ chown root:sftp /home/danaos
```

文件夹所有者必须是 root，用户组可以不是 root

```sh
$ chmod 755 /home/danaos
```

权限不能超过 755 但不包括 755，否则会导致登录报错

### 创建上传目录并设置权限

在`/home/danaos/`主目录下创建`archer`项目文件夹，并设置所有者是：`leeifme`，用户组隶属：`sftp`，这样新增的帐号才能有上传编辑的权限。

```sh
$ mkdir -p /home/danaos/archer
$ chown leeifme:sftp /home/danaos/archer
```

### 修改 sshd_config 配置文件

```shell
$ vim /etc/ssh/sshd_config
```

将此行注释掉，例如：

```sh
#Subsystem sftp /usr/libexec/openssh/sftp-server
```

在此行下面添加如下内容：

```sh
Subsystem sftp internal-sftp       # 指定使用sftp服务使用系统自带的internal-sftp
Match Group sftp                   # 匹配sftp组的用户,若要匹配多个组,可用逗号分开
ChrootDirectory /home/danaos       # 限制用户的根目录
ForceCommand internal-sftp         # 只能用于sftp登录
AllowTcpForwarding no              # 禁止用户使用端口转发
X11Forwarding no                   # 禁止用户使用端口转发
```

### 重启 SSH 服务

```shell
$ systemctl restart sshd
```

### 测试是否能够登录、上传、下载等操作

在`10.28.204.61`服务器执行以下命令登录：

```sh
$ sftp leeifme@192.168.50.124
leeifme@192.168.50.124's password:
Connected to leeifme@192.168.50.124.
sftp> ls
archer
sftp> cd archer/
sftp>
```

**上传**

```sh
sftp> put test.go
Uploading test.go to /archer/test.go
test.go             100% 3824   626.2KB/s   00:00
sftp>
```

**下载**

```sh
sftp> get test.go
Fetching /archer/test.go to test.go
/archer/oplog.go    100% 3824   475.1KB/s   00:00
sftp>
```

**删除**

```sh
sftp> rm test.go
Removing test.go
```

**更多命令请参阅**

```shell
sftp> help
```

## vscode 配置 sftp

### 下载 sftp 插件

- 在 vscode 中快捷键 `ctrl+shift+P` 打开指令窗口，输入`extension:install`，回车，左侧即打开扩展安装的界面 
- 在搜索框中输入`sftp`，第一个就是需要安装的，点安装(可以参考下载数，选择适合自己的插件，功能大都大同小异)

### 在 vscode 的工程中配置 sftp.json

然后快捷键 `ctrl+shift+P` 打开指令窗口，输入`sftp:config`，回车，就会在当前工作工程的`.vscode`文件夹下生成一个`sftp.json`文件，不知道哪天似乎是插件更新了，默认的文件非常空，我们只能手动配置文件的参数了。配置好`host, port, username, privateKeyPath, remotePath, ignore`这参数即可：

- host：工作站的 IP 地址
- port：ssh 的端口
- username：工作站自己的用户名
- privateKeyPath：存放在本地的已配置好的用于登录工作站的密钥文件。和下面的使用密码二选一（可以是 openssh 格式的，也可以是 ppk 格式的）
- password：工作站自己的用户密码。使用密钥和使用密码选用一种即可；使用密码的话工作站不用配置 ssh，但使用密钥的话工作站上需要配置好 ssh，password 就可以填 null
- protocol：协议类型，默认选`"sftp"`
- remotePath：工作站上与本地工程同步的文件夹路径，需要和本地工程文件根目录同名，且在使用 sftp 上传文件之前要手动在工作站上使用mkdir生成这个根目录，根目录下的其他子目录会自动对应生成
- ignore：指定在使用`sftp: sync to remote`的时候忽略的文件及文件夹，注意每一行后面有逗号，最后一行没有逗号
- debug： 默认是 false，如果设置为 true，可以看到通过菜单的 查看 -> 输出 打开输出界面，看到打印，怀疑自己连接有问题的可以打开看看
- uploadOnSave: 默认是 false，建议设置成 true，这样每次修改后 ctrl+s 保存后会自动同步。否则就需要手动同步

举个栗子：（**记住不能有任何注释内容**）

```json
{
    "protocol": "sftp",
    "host": "192.168.50.124",
    "port": 22,
    "username": "leeifme",
    "password": "XXXXXXXXXX",
    "remotePath": "archer/",
    "watcher": {
        "files": "**/", 
        "autoUpload": true, 
        "autoDelete": true 
    },
    "ignore": [
        "**/.git/**"
    ]
}
```

