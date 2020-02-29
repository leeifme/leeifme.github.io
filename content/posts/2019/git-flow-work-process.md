---
title: git-flow 的工作流程
date: 2019-01-06 14:15:15
categories: 
tags:
- Git
- 规范建议
- 安利系列
---

## 前言

> 当在团队开发中使用版本控制系统时，商定一个统一的工作流程是至关重要的。`Git` 的确可以在各个方面做很多事情，然而，如果在你的团队中还没有能形成一个特定有效的工作流程，那么混乱就将是不可避免的。
>
> `git-flow` 就是当前非常流行且行之有效的工作流程， 是一个 `Git` 扩展集，按 Vincent Driessen 的分支模型提供高层次的库操作，并且提供了极其出色的命令帮忙以及输出提示，本文中有演示操作，可以仔细阅读并观察发生了什么事情

## git-flow 安装

**安装指南：**https://github.com/petervanderdoes/gitflow-avh/wiki/Installation

### 准备工作（Windows）

按照官方给出的安装方法，依次点击三个链接：

- [util-linux package](http://gnuwin32.sourceforge.net/packages/util-linux-ng.htm)
- [libintl](http://gnuwin32.sourceforge.net/packages/libintl.htm)  
- [libiconv](http://gnuwin32.sourceforge.net/packages/libiconv.htm)

三个要下载的都是`Binaries` 的 zip 格式文件，下载完后只需要把各自 bin 目录下的对应文 (`getopt.exe`、`libintl3.dll`、`libiconv2.dll` ) 件复制到 git 安装目录的 bin 目录下即可, 其他的都不需要，版本和路径根据自己的情况而定

### Clone the git-flow 

打开 `Git Bash` 执行：

```shell
$ git config --global url."https://github".insteadOf git://github
```

如果不执行这条命令，直接执行下面命令的话，clone 会出现卡住现象

```shell
$ git clone --recursive git://github.com/nvie/gitflow.git
```

打开`powershell`（以管理员身份运行）， cd 到下载 `gitflow` 的目录下

```powershell
PS C:\Users\Administrator> cd .\gitflow\
PS C:\Users\Administrator\gitflow> .\contrib\msysgit-install.cmd "C:\Program Files\Software\Git"

# "C:\Program Files\Software\Git" 根据安装 git 的安装目录进行调整
```

测试是否安装成功，打开 `Git Bash`

```shell
Administrator at 14:56:56 / $ git flow help
usage: git flow <subcommand>

Available subcommands are:
   init      Initialize a new git repo with support for the branching model.
   feature   Manage your feature branches.
   bugfix    Manage your bugfix branches.
   release   Manage your release branches.
   hotfix    Manage your hotfix branches.
   support   Manage your support branches.
   version   Shows version information.
   config    Manage your git-flow configuration.
   log       Show log deviating from base branch.

Try 'git flow <subcommand> help' for details.
```

## git-flow 使用

### 开发新功能 ( **Feature** )

**需求：**添加一个 `README.md` 文件，并添加内容

**创建 `my-feature` 分支**

```shell
Administrator at 16:04:03 Go-fun (develop) $ git flow feature start my-feature
Switched to a new branch 'feature/my-feature'

Summary of actions:
- A new branch 'feature/my-feature' was created, based on 'develop'
- You are now on branch 'feature/my-feature'

Now, start committing on your feature. When done, use:

     git flow feature finish my-feature
     
Administrator at 16:04:51 Go-fun (feature/my-feature) $
```

根据提示，我们知道一个新的分支 `feature／my-feature` 基于 `develop` 分支创建好了, 并且目前我们所在的分支为 `feature／my-feature`，可以查看下分支情况

```shell
 $ git branch
  develop
* feature/my-feature
  master
```

**在 `feature/my-feature` 分支新建一个 README.md 并提交**

```shell
$ vi README.md
$ git add README.md
$ git commit -m "create README.md"
[feature/my-feature e783fdf] create README.md
1 file changed, 1 insertion(+)
create mode 100644 README.md
```

**完成 `feature` 分支合并**

根据之前创建分支的提示可知，在完成新功能的开发后，可使用 ` git flow feature finish <name> ` 提交合并

```shell
$ git flow feature finish my-feature
Switched to branch 'develop'
Updating bde3a64..e783fdf
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
Deleted branch feature/my-feature (was e783fdf).

Summary of actions:
- The feature branch 'feature/my-feature' was merged into 'develop'
- Feature branch 'feature/my-feature' has been locally deleted
- You are now on branch 'develop'
```

根据提示，我们看到了 git flow feature finish my-feature 这条语句的作用。

1. `feature/my-feature` 分支被 merge 到了 `develop` 分支
2. 本地 `feature/my-feature` 分支被删除，现在我们处于 `develop` 分支

参看分支情况：

```shell
$ git branch
* develop
  master
```

**不使用 `git-flow`**

在不使用 `git-flow` 工具的情况下，怎么实现 

1. 新建一个分支

   ```shell
    $ git checkout -b my-feature develop
   ```

2. 当开发完成后，合并分支。

   ```shell
    $ git merge --no-ff my-feature
    Already up-to-date.
   ```

3. 删除本地 my-feature 分支

   ```shell
    $ git branch -d my-feature
   ```

4. 提交更改到远程 develop 分支

   ```shell
    $ git push origin develop
   ```

比较两种方式就会发现，git flow 方便了分支的管理, 使用上更为简单

### 发布版本 ( **Release** )

**需求：**添加一个 `README.md` 文件，并添加版本号与日期

**创建 `release 1.0` 分支**

```shell
Administrator at 17:37:59 Go-fun (develop) $ git flow release start 1.0
Switched to a new branch 'release/1.0'

Summary of actions:
- A new branch 'release/1.0' was created, based on 'develop'
- You are now on branch 'release/1.0'

Follow-up actions:
- Bump the version number now!
- Start committing last-minute fixes in preparing your release
- When done, run:

     git flow release finish '1.0'


Administrator at 17:38:07 Go-fun (release/1.0) $
```

根据提示，流程和创建 `feature` 分支差不多，查看分支情况

```shell
$ git branch
  develop
  master
* release/1.0
```

**添加修改文件到暂存区**

```shell
$ git add README.md
$ git commit -m "update version to 1.0"
```

**完成 `release 1.0` 分支**

```shell
Administrator at 19:00:28 Go-fun (release/1.0) $ git flow release finish 1.0
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
Merge made by the 'recursive' strategy.
 README.md | 5 ++++-
 1 file changed, 4 insertions(+), 1 deletion(-)
Already on 'master'
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
Switched to branch 'develop'
Merge made by the 'recursive' strategy.
 README.md | 5 ++++-
 1 file changed, 4 insertions(+), 1 deletion(-)
Deleted branch release/1.0 (was 922692f).

Summary of actions:
- Release branch 'release/1.0' has been merged into 'master'
- The release was tagged '1.0'
- Release tag '1.0' has been back-merged into 'develop'
- Release branch 'release/1.0' has been locally deleted
- You are now on branch 'develop'
```

根据上图的提示，发现这一步做了如下的事情。

1. `release/1.0` 分支已经被 merge 到了 master 分支
2. 版本被打上 `tag 1.0` 
3. `elease/1.0` 被 merge 到了 `develop` 分支
4. 本地 `release/1.0` 被删除
5. 切换分支到 `develop`

**更新远程分支**

```shell
$ git push origin master
$ git push origin develop
$ git push --tags
```

**不使用 `git-flow`**

1. 创建 release-1.0 分支

    ```shell
    $ git checkout -b release-1.0 develop
    Switched to a new branch 'release-1.0'
    ```

2. 提交到分支 release-1.0

    ```shell
    $ git commit -am "Bumped version number to 1.0"
    ```

3. 切换分支到 master

    ```shell
    $ git checkout master
    ```

4. 合并 release-1.0 到 develop 分支

    ```shell
    $ git merge --no-ff release-1.0
    ```

5. 打上标签 (tag)

    ```shell
    $ git tag -a 1.0 -am "message"
    ```

6. 切换到 develop 分支，并合并到 develop 分支。

    ````shell
    $ git checkout develop
    $ git merge --no-ff release-1.0
    ````

7. 删除本地分支 realse-1.0

    ```shell
    $ git branch -d release-1.0
    ```

8. 更新远程分支

   ```shell
   $ git push origin master
   $ git push origin develop
   $ git push --tags
   ```

如果说在开发新功能的时候你没体会到 `git flow` 的优点，那么在 release 版本的时候，就能很明显地感受到 `git flow` 的优势

### 紧急修复 ( **Hotfix** )

git flow 基于 master 分支，用于紧急修复，修改完成后要 merge 到 master，develop 分支

**使用 `git-flow`**

```shell
$ git flow hotfix start 1.0.1
```

**不使用 `git-flow`**

```shell
$ git checkout -b hotfix-1.2.1 master
```

其他步骤与在 release 开发相同，同样不要忘了在 master 分支打上 tag

