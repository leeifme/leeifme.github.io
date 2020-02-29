---
title: Git 实践 - 版本库控制
date: 2017-09-15 15:36:25
tags:
  - Git
categories:
  - notes
---

>## 创建版本库
>**把文件添加到版本库：**
>- 通过`git init`命令把这个目录变成Git可以管理的仓库
>  ```json
>  $ git init
>  Initialized empty Git repository in E:/git/test/.git/
>  ```
>- 用命令`git add`告诉Git，把文件添加到仓库
>  ```nginx
>  $ git add README.md
>  ```
>- 用命令`git commit`告诉Git，把文件提交到仓库
>  ```nginx
>  $ git commit -m "initial commit"
>  [master (root-commit) 6010c7e] initial commit
>   1 file changed, 7 insertions(+)
>   create mode 100644 README.md
>  ```
>  **另外：**
>  初始创建一个 `github`仓库时，`github`会给一些命令你去创建git本地项目.
>
>```nginx
>git remote add origin https://github.com/leeifme/test.git
>```
>这里的origin仅仅是一个名字，你可以把 `origin`命名为 `test` `->`git remote add test https://github.com/leeifme/test.git`, 以后就可以用`git push text master`了
>
>`git push -u origin master` , 这里就是把 master（默认 git 分支）推送到 origin， `-u`也就是`--set-upstream`, 代表的是更新默认推送的地方，这里就是默认以后`git pull`和`git push`时，都是推送和拉自 origin 。

<!-- more -->

## 版本回退

- `git status`命令可以让我们时刻掌握仓库当前的状态
- `git diff`顾名思义就是查看difference(修改内容)
- 如果`git status`告诉你有文件被修改过，用`git diff`可以查看修改内容。

**如何回退和撤回操作：**

- 先用`git log`命令显示从最近到最远的`git commit`提交日志

  ```nginx
  $ git log
  commit f78f60960e2d59bd6fdc1a069e840c6d1f1c2566
  Author: **********************
  Date:   Thu Jun 15 01:24:55 2017 +0800

      add txt

  commit 01ce8dd6733f7f78e7611ef72f2a092417de0bca
  Author: **********************
  Date:   Thu Jun 15 01:23:51 2017 +0800

      updata homepage

  commit 4db229de25fb4d0c325c98df9263b2c3d4d4607f
  Author: **********************
  Date:   Thu Jun 15 01:14:33 2017 +0800

      add homepage

  commit 6010c7e80796b40d89e1135bc2c86ac217274015
  Author: **********************
  Date:   Thu Jun 15 01:03:15 2017 +0800

      initial commit
  ```

  **注： **这样输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数

  ```nginx
  $ git log --pretty=oneline
  9152d920e48515d79a32824efcbe14b3665a3e6d add distributed
  f78f60960e2d59bd6fdc1a069e840c6d1f1c2566 add txt
  01ce8dd6733f7f78e7611ef72f2a092417de0bca updata homepage
  4db229de25fb4d0c325c98df9263b2c3d4d4607f add homepage
  6010c7e80796b40d89e1135bc2c86ac217274015 initial commit
  ```



- 回退版本，使用`git reset`命令。在Git中，用`HEAD`表示当前版本，也就是最新的提交`3628164...882e1e0`（注意我的提交ID和你的肯定不一样），上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

  ```nginx
  $ git reset --hard HEAD^
  HEAD is now at f78f609 add txt //注意看commitID
  ```

  ​

- 后悔回退，想恢复版本，怎么办？只要知道`commitID`就好，怎么找？

  Git很人性化的提供了一个命令`git reflog`用来记录你的每一次命令：

  ```json
  $ git reflog
  f78f609 HEAD@{0}: reset: moving to HEAD^
  9152d92 HEAD@{1}: commit: add distributed
  f78f609 HEAD@{2}: commit: add txt
  01ce8dd HEAD@{3}: commit: updata homepage
  4db229d HEAD@{4}: commit: add homepage
  6010c7e HEAD@{5}: commit (initial): initial commit
  ```

- 知道`commitID`后,由于`HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`

  ```nginx
  $ git reset --hard 9152d92
  HEAD is now at 9152d92 add distributed
  ```

  我们可以再看下版本日志，查看是否回到`commitID = 9152d92`的版本：

  ```json
  $ git log --pretty=oneline
  9152d920e48515d79a32824efcbe14b3665a3e6d add distributed
  f78f60960e2d59bd6fdc1a069e840c6d1f1c2566 add txt
  01ce8dd6733f7f78e7611ef72f2a092417de0bca updata homepage
  4db229de25fb4d0c325c98df9263b2c3d4d4607f add homepage
  6010c7e80796b40d89e1135bc2c86ac217274015 initial commit
  ```

