---
title: Git 实践 - 修改操作
date: 2017-09-11 20:36:25
tags:
  - Git
categories:
  - git
---

>## 工作区和版本库
>
>- **工作区：**
>
>  就是你电脑文件夹能看到的目录，就如我创建的test文件夹就是一个工作区
>
>  ```nginx
>  leeif@leeif MINGW64 /e/git/test (master)
>  $ ls
>  git.txt  index.html  README.md
>  ```
>- **版本库(Repository)：**
>
>  工作区有一个隐藏的目录文件`.git`，这个虽然在工作区目录下，但不算在工作区，而是`Git`的版本库。
>
>  `Git`的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支`master`，以及指向`master`的一个指针叫`HEAD`。
>
>  <img src = "http://orj2jcr7i.bkt.clouddn.com/%E5%B7%A5%E4%BD%9C%E5%8C%BA%E5%92%8C%E7%89%88%E6%9C%AC%E5%BA%93.png" alt = "flex-axis">
>
>
>  ​
>
>上一篇讲了我们把文件往Git版本库里添加的时候，是分两步执行的：
>
>1. 用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；
>2. 用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。
>
>因为我们创建Git版本库时，Git自动为我们创建了唯一一个`master`分支，所以，现在，`git commit`就是往`master`分支上提交更改。
>
>你可以简单理解为，需要提交的文件修改通通放到暂存区，然后，一次性提交暂存区的所有修改。
>
>**ok，我们实践一下：**

<!-- more -->

- 首先新建一个文件`test.txt`，然后对`git.exe`进行修改，

  ```nginx
  $ git diff git.txt
  diff --git a/git.txt b/git.txt
  index 97fe984..a537f08 100644
  --- a/git.txt
  +++ b/git.txt
  @@ -2,4 +2,6 @@ Git is a version control system.
   Git is free software.

   Git is a distributed version control system.
  -Git is free software.
  \ No newline at end of file
  +Git is free software.
  +
  +add test add test
  \ No newline at end of file
  ```

  ​

- 然后我们用`git status`查看一下状态：

  ```js
  $ git status
  On branch master
  Your branch is ahead of 'origin/master' by 3 commits.
    (use "git push" to publish your local commits)

  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)

          modified:   git.txt

  Untracked files:
    (use "git add <file>..." to include in what will be committed)

          test.txt
  ```

  Git非常清楚地告诉我们，`git.txt`被修改了，而`test.txt`还从来没有被添加过，所以它的状态是`Untracked`

- 现在，使用两次命令`git add`，把`git.txt`和`test.txt`都添加后，用`git status`再查看一下：

  ```nginx
  $ git status
  On branch master
  Your branch is ahead of 'origin/master' by 3 commits.
    (use "git push" to publish your local commits)
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

          modified:   git.txt
          new file:   test.txt
  ```

  所以，暂存区的状态就变成这样了：

  <img src = "http://orj2jcr7i.bkt.clouddn.com/git%20add.png" alt = "flex-axis">

- `git add`命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支。

  ```nginx
  $ git commit -m "updata&test "
  [master 0fbc057] updata&test
   3 files changed, 5 insertions(+), 1 deletion(-)
   create mode 100644 test.txt
  ```

  在检查下状态，如果你没在对工作区文件进行修改的话，工作区返回的状态就是`clean`

  ```nginx
  $ git status
  On branch master
  Your branch is ahead of 'origin/master' by 4 commits.
    (use "git push" to publish your local commits)
  nothing to commit, working tree clean
  ```

  现在版本库变成了这样，暂存区就没有任何内容了：

  <img src = "http://orj2jcr7i.bkt.clouddn.com/git%20commit.png" alt = "flex-axis">

<div class="tip">

**小结：**
暂存区是Git非常重要的概念，弄明白了暂存区，就弄明白了Git的很多操作到底干了什么!

</div>

## 管理修改

为什么Git比其他版本控制系统设计得优秀，因为Git跟踪并管理的是修改，而非文件。

每次修改，如果不`add`到暂存区，那就不会加入到`commit`中。

**例如：**

- 我们对`git.txt`进行两次修改，并在修改之间进行如下指令操作：

  第一次修改 -> `git add` -> 第二次修改 -> `git commit`；

  没错，我们会发现第二次修改没有被提交。我们前面讲了，Git管理的是修改，当你用`git add`命令后，在工作区的第一次修改被放入暂存区，准备提交，但是，在工作区的第二次修改并没有放入暂存区，所以，`git commit`只负责把暂存区的修改提交了，也就是第一次的修改被提交了，第二次的修改不会被提交。

  ​

- 那怎么提交第二次修改呢？你可以继续`git add`再`git commit`，也可以别着急提交第一次修改，先`git add`第二次修改，再`git commit`，就相当于把两次修改合并后一块提交了：

  第一次修改 -> `git add` -> 第二次修改 -> `git add` -> `git commit`

  <br />

注：提交后，用`git diff HEAD -- readme.txt`命令可以查看工作区和版本库里面最新版本的区别

## 撤销修改

每个人都会犯错，但重点在于你能不能及时改正，Git当然也知道这一点，所以它给了我们撤销修改这个机会。

- **第一种情况，在没`git add`提交到暂存区时，撤销修改**

  我对`git.txt`增加了一条错误信息，使用`git diff`查看一下：

  ```nginx
   $ git diff HEAD -- git.txt
  diff --git a/git.txt b/git.txt
  index a537f08..246dfe2 100644
  --- a/git.txt
  +++ b/git.txt
  @@ -4,4 +4,6 @@ Git is free software.
   Git is a distributed version control system.
   Git is free software.
   add test add test

  +
  +Add an error message
  \ No newline at end of file
  ```

  我们再用`git status`查看一下状态：

  ```nginx
  $ git status
  On branch master
  Your branch is ahead of 'origin/master' by 4 commits.
    (use "git push" to publish your local commits)
  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)

          modified:   git.txt

  no changes added to commit (use "git add" and/or "git commit -a")
  ```

  可以发现，Git会告诉你，`git checkout -- file`可以丢弃工作区的修改

  ok， 我们试一下，执行：

  ```nginx
  $ git checkout -- git.txt
  ```

  然后，查看文件`git.txt`:

  ```nginx
  $ cat git.txt
  Git is a version control system.
  Git is free software.

  Git is a distributed version control system.
  Git is free software.

  add test add test
  ```

  嗯，最后一行`Add an error message`果然撤销了

  ​

- **第二种情况，你把修改文件提交到了暂存区，但在`git commit`之前**

  我们给`git.txt`添加`Add a second error message`，并执行`git add`提交到暂存区，用`git staus`参看以下状态：

  ```nginx
  $ $ git status
    (use "git push" to publish your local commits)
  Changes to be committed:
    (use "git reset HEAD <file>..." to unstage)

          modified:   git.txt
  bash: $: command not found
  ```

  可以看到，Git同样告诉我们，用命令`git reset HEAD file`可以把暂存区的修改撤销掉（unstage），重新放回工作区：

  ```nginx
  $ git reset HEAD git.txt
  Unstaged changes after reset:
  M       git.txt
  ```

  `git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用`HEAD`时，表示最新的版本。

  现在，我们可以执行`git status`查看状态发现，暂存区是干净的，而工作区有修改，其实，现在就回到了我们上面`第一种情况`了，执行`git checkout -- git.txt`后，查看状态，发现工作区也干净了，也就达到撤销修改的目的了。

  ```nginx
  $ cat git.txt
  Git is a version control system.
  Git is free software.

  Git is a distributed version control system.
  Git is free software.

  add test add test
  ```

<div class = "tip">

**小结:**

- 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。
- 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。
- 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考上一篇，不过前提是没有推送到远程库。

</div>

## 删除文件

在Git中，删除也是一个修改操作，我们实验一下，之前添加一个新文件test.txt到Git并且进行了`git commit`提交操作，可以看下`log`：

```nginx
$ git log --pretty=oneline
0fbc057e11b4ba59f3700a79d760e5af927a6a65 updata&test  //添加text.txt文件
9152d920e48515d79a32824efcbe14b3665a3e6d add distributed
f78f60960e2d59bd6fdc1a069e840c6d1f1c2566 add txt
01ce8dd6733f7f78e7611ef72f2a092417de0bca updata homepage
4db229de25fb4d0c325c98df9263b2c3d4d4607f add homepage
6010c7e80796b40d89e1135bc2c86ac217274015 initial commit
```

一般情况下，我们通常直接在文件管理器中把没用的文件删了，或者用`rm`命令删了：

```java
$ rm test.txt

# 查看目录 已经没了test,txt文件
$ ls
git.txt  index.html  README.md
```

习惯性查看下状态：

```json
$ git status
On branch master
Your branch is ahead of 'origin/master' by 4 commits.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    test.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

发现Git已经知道我们进行了删除操作。

**现在会出现两种情况：**

- 一种情况就是执行`rm`命令后，发现文件删除错误了，但好在版本库里还有，所以可以很轻松的把误删的文件恢复到最新版本：

  ```json
  $ git checkout -- test.txt
  ```

  `git checkout`其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。

  ​


- 另一种情况确定进行删除操作，并需要从版本库中删除该文件。因此，需要使用`git rm`命令，并执行`git commit`提交操作：

  ```nginx
  $ git rm test.txt
  rm 'test.txt'

  $ git commit -m "deleted test"
  [master a15cd98] deleted test
   1 file changed, 0 insertions(+), 0 deletions(-)
   delete mode 100644 test.txt
  ```

  现在，文件就从版本库中也删除了，就此消失不见~~~

<div class = "tip">

命令git rm用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。

</div>
