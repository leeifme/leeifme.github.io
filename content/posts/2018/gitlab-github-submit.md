---
title: Gitlab or Github 提交规范和建议
date: 2018-12-06 14:15:15
categories: 
tags:
- Git
- 规范建议
---

## 标签（tag）的管理规范

### 标签分类

正式标签：以发布的版本号居多，例：`v.1.2`

临时标签：版本测试期间做回溯和标记使用，例：`v1.1-st` 、 `v1.1-sit2`

### 命名规范

1. 标签前缀（大多）以发布版本号命名，示例：`v1.3.1`（后期遇到特殊情况再讨论）
2. 如果需要在版本号后加备注信息，用『“-”：连字符』连接，不再使用 “_” 下划线，且字母全部小写，强烈建议不要以中文命名。
3. 版本经历多次修改时，可以加上 release 的后缀
4. www：是带官网版本的包（目前官网和产品还没有分仓库）
5. 临时标签存在于当前版本的测试期间，版本正式发布后，可以删除相关临时标签。

### 标签一览

﻿![img](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2018/12/tag.png)﻿

### 清理远程不存在的标签﻿

删除远程不存在标签的做法：

打开命令行

```shell
#进入仓库的目录
cd ../.git 
git tag -l | xargs git tag -d 
git fetch origin --prune 
git fetch origin --tags 
```

## 分支（ branch）管理规范

### 分支规范

﻿![img](https://leeifme.oss-cn-shanghai.aliyuncs.com/blog/2018/12/branch.png)﻿

## 提交（commit）规范

### commit message

从简，只规范提交前缀（type），只允许使用下面7个标识。

- `feat`：新功能（feature）
- `fix`：修补 bug
- `docs`：文档（documentation）
- `style`： 格式（不影响代码运行的变动）
- `refactor`：重构（即不是新增功能，也不是修改 bug 的代码变动）
- `test`：增加测试
- `chore`：构建过程或辅助工具的变动﻿

### 提交常犯错误

1. 未加 type
2. type 拼写错误
3. type 后的冒号禁止使用中文冒号
4. 冒号后没有空格
5. 测试期，fix: #216 xxxx   fix bug 时未加编号



> **参考内容**
>
> [Git 提交的正确姿势：Commit message 编写指南](https://www.oschina.net/news/69705/git-commit-message-and-changelog-guide)