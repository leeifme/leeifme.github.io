---
title: Windows 利器 - babun
categories: 
- collect
tags:
- 百宝箱
- 安利系列
date: 2018-02-10 20:36:25
---

## 什么是 babun

babun 是 windows 上的一个第三方 shell，在这个 shell 上面你可以使用几乎所有 linux，unix 上面的命令，他几乎可以取代 windows 的 shell。用官方的题目说就是 A Windows shell you will love!

> ### babun 的几个特点
>
> 使用 babun 无需管理员权限
> 先进的安装包管理器 (类似于 linux 上面的 apt-get 或 yum)
> 预先配置了 Cygwin 和很多插件
> 拥有 256 色的兼容控制台
> HTTP(S) 的代理支持
> 面向插件的体系结构
> 可以使用它来配置你的 git
> 集成了 oh-my-zsh
> 自动升级
> 支持 shell 编程，内置 VIM 等
>
> babun 官网链接：<<http://babun.github.io/>>

## 什么是 cmder

cmder 是 window 下的多标签命令行工具，可以方便的新建 cmd、cmd admin、powershell、powershell admin 多种命令行，设置很多，功能强大。

## 安装

### cmder 安装

下载：<http://cmder.net/>

cmder 是开箱即用的软件就不在详述了，具体使用可参考官网说明。

### babun 安装

下载：<http://babun.github.io/>

##### 默认安装

下载完成之后解压 babun，直接双击目录中 install.bat 脚本 (需管理员权限) 进行安装。几分钟之后自动安装完成，默认会被安装在`%userprofile%\.babun`目录下。

##### 自定义安装位置

通过 cmd 命令行在执行 install.bat 时指定参数 / t 或 / target 指定安装的目录。

执行：`babun.bat /t c:\babun`

安装好之后会在 c:\babun 目录下生成一个. babun 的目录，babun 所有文件都在这个目录中。注意安装目录最好不要有空格，这是 cygwin 要求的。

##### 测试安装成功

安装完毕后，一般需要以下两个命令检查

```
babun check(用于判断环境是否正确)
babun update(用于判断是否有新的更新包)
```

## babun 主要配置

### 主题配置

- 字体下载（不安装会乱码）

  ```shell
  # clone(克隆git)
  git clone https://github.com/powerline/fonts.git --depth=1
  # install(运行安装脚本)
  cd fonts
  ./install.sh
  # clean-up a bit(清楚克隆git文件)_
  cd ..
  rm -rf fonts
  ```

  安装  [在`~\.local\share\fonts`目录下找到`DejaVu Sans Mono for Powerline`字体]

- 把`ZSH_THEME=”babun”`设置为`ZSH_THEME=”agnoster”`（详情参考`.zshrc`配置）

- 修改`~/.minttyrc`

  ```shell
  # open ~/.minttyrc
  vi ~/.minttyrc

  # 把下面这段复制粘贴进去
  BoldAsFont=no
  Columns=90
  Rows=55
  Font=DejaVu Sans Mono for Powerline
  FontHeight=10
  Transparency=low

  ForegroundColour=248,248,242
  BackgroundColour=27,29,30
  CursorColour=160,160,160
  Black=249,38,114
  Red=249,38,114
  Green=130,180,20
  Yellow=253,151,31
  Blue=38,139,210
  Magenta=140,84,254
  Cyan=86,194,214
  White=204,204,198
  BoldRed=255,89,149
  BoldBlack=80,83,84
  BoldGreen=183,235,70
  BoldYellow=254,237,108
  BoldBlue=98,173,227
  BoldMagenta=191,160,254
  BoldCyan=148,216,229
  BoldWhite=248,248,242
  Scrollbar=none
  ```

### babun 插件安装

#### [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) && [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

```shell
cd ~/.oh-my-zsh/custom/plugins
git clone git://github.com/zsh-users/zsh-syntax-highlighting.git
git clone git://github.com/zsh-users/zsh-autosuggestions
vi ~/.zshrc
```

```shell
# add below to ~/.zshrc
plugins=(zsh-syntax-highlighting zsh-autosuggestions)
```

```shell
source ~/.zshrc or src
```

[![zsh-syntax-highlighting](https://raw.githubusercontent.com/zsh-users/zsh-syntax-highlighting/master/images/preview.png)](https://raw.githubusercontent.com/zsh-users/zsh-syntax-highlighting/master/images/preview.png)

#### [autojump](https://github.com/wting/autojump)

```shell
cd ~/.oh-my-zsh/custom/plugins
git clone git://github.com/joelthelion/autojump.git
cd autojump
./install.py
vi ~/.zshrc
```

``` shell
# add below to ~/.zshrc
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
autoload -U compinit && compinit -u
```

然后定位到 plugins=(git) 的位置，将其修改成：

```shell
plugins=(git autojump)
```

### .zshrc配置

```shell
# Path to your oh-my-zsh installation.
export ZSH=$HOME/.oh-my-zsh

# Set name of the theme to load.
# Look in ~/.oh-my-zsh/themes/
ZSH_THEME="agnoster"

# DEFAULT_USER="$USER"

# Uncomment the following line if you want to disable marking untracked files
# under VCS as dirty. This makes repository status check for large repositories
# much, much faster.
# DISABLE_UNTRACKED_FILES_DIRTY="true"

# The optional three formats: "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
HIST_STAMPS="yyyy-mm-dd"

# Would you like to use another custom folder than $ZSH/custom?
# ZSH_CUSTOM=/path/to/new-custom-folder

# Which plugins would you like to load? (plugins can be found in ~/.oh-my-zsh/plugins/*)
# Custom plugins may be added to ~/.oh-my-zsh/custom/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(git autojump sh-autosuggestions zsh-syntax-highlighting wd web-search history history-substring-search)

[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
autoload -U compinit && compinit -u

source ~/.bash_profile
# Load zsh-syntax-highlighting.
source ~/.oh-my-zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
#
# Load zsh-autosuggestions.
source ~/.oh-my-zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh

# User configuration
export PATH=$HOME/bin:/usr/local/bin:$PATH
# export MANPATH="/usr/local/man:$MANPATH"

source $ZSH/oh-my-zsh.sh

# Compilation flags
# export ARCHFLAGS="-arch x86_64"

# ssh
# export SSH_KEY_PATH="~/.ssh/dsa_id"

# Example aliases
alias zshconfig="mate ~/.zshrc"
alias ohmyzsh="mate ~/.oh-my-zsh"
alias cls='clear'
alias mysql='/usr/local/opt/mysql/bin/mysql'
alias mysqladmin='/usr/local/opt/mysql/bin/mysqladmin'
alias rake='noglob rake'

alias code="/Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code"
alias vi="vim"
alias vim="vim"
alias tmux="tmux -2"
alias ssh="ssh -X"
alias s="ssh -X"
alias md="mkdir -p"
alias rd="rmdir"
alias df="df -h"
alias mv="mv -i"
alias slink="link -s"
alias sed="sed -E"
alias l="ls -l"
alias la="ls -a"
alias ll="ls -la"
alias lt="ls -lhtrF"
alias l.="ls -lhtrdF .*"
alias grep="grep --color=auto"
alias cd..="cd .."
alias cd...="cd ../.."
alias cd....="cd ../../.."
alias ..="cd .."
alias ...="cd ../.."
alias ....="cd ../../.."
alias zb="cat /dev/urandom | hexdump -C | grep --color=auto \"ca fe\""
alias mtr="/usr/local/sbin/mtr"
alias gs="git status"
alias gsm="git summary"
alias ga='git add'
alias gd='git diff'
alias gf='git fetch'
alias grv='git remote -v'
alias grb='git rebase'
alias gbr='git branch'
alias gpl="git pull"
alias gps="git push"
alias gco="git checkout"
alias gl="git log"
alias gc="git commit -m"
alias gm="git merge"
alias pro="proxychains4"
alias gb="go build"

alias -s go=vi
alias -s html=vi
alias -s rb=vi
alias -s py=vi
alias -s txt=vi
alias -s ex=vi
alias -s exs=vi
alias -s js=vi
alias -s json=vi


alias qas01='ssh root@123.59.56.118 -p38390'
alias rdp01='ssh lei.li@rdp01.xiaoenai.net -p38390'
alias rdp='ssh lei.li@rdp01.xiaoenai.net -p38390'

_COLUMNS=$(tput cols)
_MESSAGE=" FBI Warining "
y=$(( ( $_COLUMNS - ${#_MESSAGE} )  / 2 ))
spaces=$(printf "%-${y}s" " ")

echo " "
echo -e "${spaces}\033[41;37;5m FBI WARNING \033[0m"
echo " "
_COLUMNS=$(tput cols)
_MESSAGE="Ferderal Law provides severe civil and criminal penalties for"
y=$(( ( $_COLUMNS - ${#_MESSAGE} )  / 2 ))
spaces=$(printf "%-${y}s" " ")
echo -e "${spaces}${_MESSAGE}"

_COLUMNS=$(tput cols)
_MESSAGE="the unauthorized reproduction, distribution, or exhibition of"
y=$(( ( $_COLUMNS - ${#_MESSAGE} )  / 2 ))
spaces=$(printf "%-${y}s" " ")
echo -e "${spaces}${_MESSAGE}"

_COLUMNS=$(tput cols)
_MESSAGE="copyrighted motion pictures (Title 17, United States Code,"
y=$(( ( $_COLUMNS - ${#_MESSAGE} )  / 2 ))
spaces=$(printf "%-${y}s" " ")
echo -e "${spaces}${_MESSAGE}"

_COLUMNS=$(tput cols)
_MESSAGE="Sections 501 and 508). The Federal Bureau of Investigation"
y=$(( ( $_COLUMNS - ${#_MESSAGE} )  / 2 ))
spaces=$(printf "%-${y}s" " ")
echo -e "${spaces}${_MESSAGE}"

_COLUMNS=$(tput cols)
_MESSAGE="investigates allegations of criminal copyright infringement"
y=$(( ( $_COLUMNS - ${#_MESSAGE} )  / 2 ))
spaces=$(printf "%-${y}s" " ")
echo -e "${spaces}${_MESSAGE}"

_COLUMNS=$(tput cols)
_MESSAGE="(Title 17, United States Code, Section 506)."
y=$(( ( $_COLUMNS - ${#_MESSAGE} )  / 2 ))
spaces=$(printf "%-${y}s" " ")
echo -e "${spaces}${_MESSAGE}"

echo " "
```

