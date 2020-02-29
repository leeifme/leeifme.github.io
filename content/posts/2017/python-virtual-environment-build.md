---
title: Python 虚拟坏境的搭建和配置
date: 2017-09-23 17:12:00
tags:
  - Python
categories:
  - notes
---

Python易用，但用好却不易，其中比较头疼的就是包管理和Python不同版本的问题，特别是当你使用Windows的时候。为了解决这些问题，有不少发行版的[Python]()，比如WinPython、Anaconda等，这些发行版将python和许多常用的package打包，方便pythoners直接使用，此外，还有virtualenv、pyenv等工具管理虚拟环境。

这篇文章主要就是记录了不使用发行版，和最终使用发行版Anaconda，因为其强大而方便的包管理与环境管理的功能。

## Python虚拟环境的搭建和配置

**系统环境：**

- Win10
- Python 3.6.0
- Anaconda3

## 不使用发行版

**借助`virtualenvwrapper`工具实现虚拟环境**

首先，安装：

```nginx
pip install virtualenvwrapper
```

**注：** 可以使用豆瓣源`https://pypi.douban.com/simple/`

但是不幸的是，在我的系统上报错了


```nginx
Collecting virtualenvwrapper
  Using cached virtualenvwrapper-4.7.2.tar.gz
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "D:\python\Anaconda3\lib\urllib\request.py", line 1318, in do_open
        encode_chunked=req.has_header('Transfer-encoding'))
      File "D:\python\Anaconda3\lib\http\client.py", line 1239, in request
        self._send_request(method, url, body, headers, encode_chunked)
      File "D:\python\Anaconda3\lib\http\client.py", line 1285, in _send_request
        self.endheaders(body, encode_chunked=encode_chunked)
      File "D:\python\Anaconda3\lib\http\client.py", line 1234, in endheaders
        self._send_output(message_body, encode_chunked=encode_chunked)
      File "D:\python\Anaconda3\lib\http\client.py", line 1026, in _send_output
        self.send(msg)
      File "D:\python\Anaconda3\lib\http\client.py", line 964, in send
        self.connect()
      File "D:\python\Anaconda3\lib\http\client.py", line 1400, in connect
        server_hostname=server_hostname)
      File "D:\python\Anaconda3\lib\ssl.py", line 401, in wrap_socket
        _context=self, _session=session)
      File "D:\python\Anaconda3\lib\ssl.py", line 808, in __init__
        self.do_handshake()
      File "D:\python\Anaconda3\lib\ssl.py", line 1061, in do_handshake
        self._sslobj.do_handshake()
      File "D:\python\Anaconda3\lib\ssl.py", line 683, in do_handshake
        self._sslobj.do_handshake()
    socket.timeout: _ssl.c:733: The handshake operation timed out

    During handling of the above exception, another exception occurred:

    Traceback (most recent call last):
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 746, in open_url
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 947, in _socket_timeout
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 1065, in open_with_auth
      File "D:\python\Anaconda3\lib\urllib\request.py", line 223, in urlopen
        return opener.open(url, data, timeout)
      File "D:\python\Anaconda3\lib\urllib\request.py", line 526, in open
        response = self._open(req, data)
      File "D:\python\Anaconda3\lib\urllib\request.py", line 544, in _open
        '_open', req)
      File "D:\python\Anaconda3\lib\urllib\request.py", line 504, in _call_chain
        result = func(*args)
      File "D:\python\Anaconda3\lib\urllib\request.py", line 1361, in https_open
        context=self._context, check_hostname=self._check_hostname)
      File "D:\python\Anaconda3\lib\urllib\request.py", line 1320, in do_open
        raise URLError(err)
    urllib.error.URLError: <urlopen error _ssl.c:733: The handshake operation timed out>

    During handling of the above exception, another exception occurred:

    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "C:\Users\leeif\AppData\Local\Temp\pip-build-somfu2_x\virtualenvwrapper\setup.py", line 7, in <module>
        pbr=True,
      File "D:\python\Anaconda3\lib\distutils\core.py", line 108, in setup
        _setup_distribution = dist = klass(attrs)
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\dist.py", line 315, in __init__
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\dist.py", line 361, in fetch_build_eggs
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\pkg_resources\__init__.py", line 851, in resolve
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\pkg_resources\__init__.py", line 1123, in best_match
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\pkg_resources\__init__.py", line 1135, in obtain
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\dist.py", line 428, in fetch_build_egg
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\command\easy_install.py", line 652, in easy_install
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 635, in fetch_distribution
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 616, in find
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 559, in download
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 804, in _download_url
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 810, in _attempt_download
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 709, in _download_to
      File "D:\python\Anaconda3\lib\site-packages\setuptools-27.2.0-py3.6.egg\setuptools\package_index.py", line 760, in open_url
    distutils.errors.DistutilsError: Download error for https://pypi.python.org/packages/d5/d6/f2bf137d71e4f213b575faa9eb426a8775732432edb67588a8ee836ecb80/pbr-3.1.1.tar.gz#md5=4e82c2e07af544c56a5b71c801525b00: _ssl.c:733: The handshake operation timed out

    ----------------------------------------
Command "python setup.py egg_info" failed with error code 1 in .....\Local\Temp\pip-build-somfu2_x\virtualenvwrapper\
```

**解决办法：**

```
pip install -i https://pypi.douban.com/simple/ pbr

pip install -i https://pypi.douban.com/simple/ stevedore

pip install -i https://pypi.douban.com/simple/ virtualenvwrapper
```

然后使用`mkvirutualenv --python=(python's path) name`新建虚拟坏境

## 使用Anaconda发行版

> [Anaconda](https://www.continuum.io/why-anaconda) 是一个用于科学计算的 Python 发行版，支持 Linux, Mac, Windows 系统，提供了包管理与环境管理的功能，可以很方便地解决多版本 python 并存、切换以及各种第三方包安装问题。Anaconda 利用工具 / 命令`conda`来进行 package 和 environment 的管理，并且已经包含了 Python 和相关的配套工具。
>
> `conda`可以理解为一个工具，也是一个可执行命令，其核心功能是**包管理**与**环境管理**。包管理与 pip 的使用类似，环境管理则允许用户方便地安装不同版本的 python 并可以快速切换。Anaconda 则是一个打包的集合，里面预装好了 conda、某个版本的 python、众多 packages、科学计算工具等等，所以也称为 Python 的一种发行版。

## Anaconda 的安装

Anaconda 的下载页参见[官网下载](https://www.continuum.io/downloads)，Linux、Mac、Windows 均支持。

安装时，会发现有两个不同版本的 Anaconda，分别对应 Python 2.7 和 Python 3.5，两个版本其实除了这点区别外其他都一样。后面我们会看到，安装哪个版本并不本质，因为通过环境管理，我们可以很方便地切换运行时的 Python 版本。（由于我常用的 Python 是 2.7 和 3.4，因此倾向于直接安装 Python 2.7 对应的 Anaconda）

下载后直接按照说明安装即可。这里想提醒一点：尽量按照 Anaconda 默认的行为安装——不使用 root 权限，仅为个人安装，安装目录设置在个人主目录下（Windows 就无所谓了）。这样的好处是，同一台机器上的不同用户完全可以安装、配置自己的 Anaconda，不会互相影响。

对于 Mac、Linux 系统，Anaconda 安装好后，实际上就是在主目录下多了个文件夹（`~/anaconda`）而已，Windows 会写入注册表。安装时，安装程序会把 bin 目录加入 PATH（Linux/Mac 写入`~/.bashrc`，Windows 添加到系统变量 PATH），这些操作也完全可以自己完成。以 Linux/Mac 为例，安装完成后设置 PATH 的操作是

```
# 将 anaconda 的 bin 目录加入 PATH，根据版本不同，也可能是~/anaconda3/bin
echo 'export PATH="~/anaconda2/bin:$PATH"' >> ~/.bashrc
# 更新 bashrc 以立即生效
source ~/.bashrc
```

配置好 PATH 后，可以通过`which conda`或`conda --version`命令检查是否正确。假如安装的是 Python 2.7 对应的版本，运行`python --version`或`python -V`可以得到 Python 2.7.12 :: Anaconda 4.1.1 (64-bit)，也说明该发行版默认的环境是 Python 2.7。

## Conda 的环境管理

Conda 的环境管理功能允许我们同时安装若干不同版本的 Python，并能自由切换。对于上述安装过程，假设我们采用的是 Python 2.7 对应的安装包，那么 Python 2.7 就是默认的环境（默认名字是`root`，注意这个 root 不是超级管理员的意思）。

假设我们需要安装 Python 3.4，此时，我们需要做的操作如下：

```
# 创建一个名为 python34 的环境，指定 Python 版本是 3.4（不用管是 3.4.x，conda 会为我们自动寻找 3.4.x 中的最新版本）
conda create --name python34 python=3.4
 
# 安装好后，使用 activate 激活某个环境
activate python34 # for Windows
source activate python34 # for Linux & Mac
# 激活后，会发现 terminal 输入的地方多了 python34 的字样，实际上，此时系统做的事情就是把默认 2.7 环境从 PATH 中去除，再把 3.4 对应的命令加入 PATH
 
# 此时，再次输入
python --version
# 可以得到 `Python 3.4.5 :: Anaconda 4.1.1 (64-bit)`，即系统已经切换到了 3.4 的环境
 
# 如果想返回默认的 python 2.7 环境，运行
deactivate python34 # for Windows
source deactivate python34 # for Linux & Mac
 
# 删除一个已有的环境
conda remove --name python34 --all
```

用户安装的不同 python 环境都会被放在目录`~/anaconda/envs`下，可以在命令中运行`conda info -e`查看已安装的环境，当前被激活的环境会显示有一个星号或者括号。

说明：有些用户可能经常使用 python 3.4 环境，因此直接把`~/anaconda/envs/python34`下面的 bin 或者 Scripts 加入 PATH，去除 anaconda 对应的那个 bin 目录。这个办法，怎么说呢，也是可以的，但总觉得不是那么 elegant……

如果直接按上面说的这么改 PATH，你会发现 conda 命令又找不到了（当然找不到啦，因为 conda 在`~/anaconda/bin`里呢），这时候怎么办呢？方法有二：1. 显式地给出 conda 的绝对地址 2. 在 python34 环境中也安装 conda 工具（推荐）。

## Conda 的包管理

Conda 的包管理就比较好理解了，这部分功能与`pip`类似。

例如，如果需要安装 scipy：

```
# 安装 scipy
conda install scipy
# conda 会从从远程搜索 scipy 的相关信息和依赖项目，对于 python 3.4，conda 会同时安装 numpy 和 mkl（运算加速的库）
 
# 查看已经安装的 packages
conda list
# 最新版的 conda 是从 site-packages 文件夹中搜索已经安装的包，不依赖于 pip，因此可以显示出通过各种方式安装的包
```

conda 的一些常用操作如下：

```nginx
conda list
 
# 查看某个指定环境的已安装包
conda list -n python34
 
# 查找 package 信息
conda search numpy
 
# 安装 package
conda install -n python34 numpy
# 如果不用 - n 指定环境名称，则被安装在当前活跃环境
# 也可以通过 - c 指定通过某个 channel 安装
 
# 更新 package
conda update -n python34 numpy
 
# 删除 package
conda remove -n python34 numpy
```

前面已经提到，conda 将 conda、python 等都视为 package，因此，完全可以使用 conda 来管理 conda 和 python 的版本，例如

```python
# 更新 conda，保持 conda 最新
conda update conda
 
# 更新 anaconda
conda update anaconda
 
# 更新 python
conda update python
# 假设当前环境是 python 3.4, conda 会将 python 升级为 3.4.x 系列的当前最新版本
```

补充：如果创建新的 python 环境，比如 3.4，运行`conda create -n python34 python=3.4`之后，conda 仅安装 python 3.4 相关的必须项，如 python, pip 等，如果希望该环境像默认环境那样，安装 anaconda 集合包，只需要：

```python
# 在当前环境下安装 anaconda 包集合
conda install anaconda
 
# 结合创建环境的命令，以上操作可以合并为
conda create -n python34 python=3.4 anaconda
# 也可以不用全部安装，根据需求安装自己需要的 package 即可
```

## 设置国内镜像

如果需要安装很多 packages，你会发现 conda 下载的速度经常很慢，因为 Anaconda.org 的服务器在国外。所幸的是，清华 TUNA 镜像源有 Anaconda 仓库的镜像，我们将其加入 conda 的配置即可：

```python
# 添加 Anaconda 的 TUNA 镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
# TUNA 的 help 中镜像地址加有引号，需要去掉
 
# 设置搜索时显示通道地址
conda config --set show_channel_urls yes
```

执行完上述命令后，会生成`~/.condarc`(Linux/Mac) 或`C:UsersUSER_NAME.condarc`文件，记录着我们对 conda 的配置，直接手动创建、编辑该文件是相同的效果。

**ok,配置完成，Just Try！**





