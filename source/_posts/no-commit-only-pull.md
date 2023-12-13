---
title: 不 commit 仅仅 pull
date: 2023-12-13 14:12:44
tags:
  - git
---

**工作中可能有一种情况是他人提交了代码，我们需要在此基础上修改，但是有部分自己写的代码未完成，这个时候我们不想提交只想更新一下代码。**

- 暂存改动（暂存改动的文件，注意：新增的文件不会到暂存区）

```bash
git stash
```

- 获取远程分支（获取当前分支时，可以只写 git fetch）

```bash
git fetch <remote>
```

- 合并远程分支（同上）

```bash
git merge <remote>
```

- 还原暂存的修改

```bash
git stash pop
```

**此时即可达到不 commit 直接拉下来代码的效果**

```bash
git stash               # 暂存当前改动且未commit的文件
git fetch <remote>      # 获取远程分支
git merge <remote>      # 合并远程分支到本地分支
git stash pop           # 还原被暂存的问题
```
