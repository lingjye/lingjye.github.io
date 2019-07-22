---
layout: post
title: "Git常用命令（备忘）"
subtitle: ''
author: "lingjye"
header-style: text
tags:
  - Git
---

克隆

```
git clone git_respository_url
```

创建git仓库

```
git init 
```

本地提交

```
// 先添加文件，‘.’指所有文件，指定选中文件，在后边加上文件名字
git add .

// 提交
git commit -m '更新内容'
```

删除文件

```
git mv 文件名
```

添加git仓库地址

```
git remote add origin git_respository_url
```

拉取远程分支

```
git pull origin master
```

提交本地代码到远程仓库

```
git push origin master
```

添加标签

```
git tag 0.1.0
```

查看标签

```
git tag
```

推送标签到远程仓库

```
git push --tags
```

删除本地标签

```
git tag -d 0.1.0
```

删除远程仓库标签

```
git push origin :refs/tags/0.1.0
```

创建分支

```
git branch dev
```

检出分支

```
git checkout dev
```

创建并检出新分支

```
git checkout -b dev
```

查看分支

```
git branch
```

删除分支

```
git branch -d dev
```

合并

```
git merge dev
```

撤销合并

```
git revert -m '内容描述' 合并操作的sha-1
```

reset

```
git reset --hart 重置到某一操作的sha-1
```











