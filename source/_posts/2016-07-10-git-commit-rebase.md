---
layout:     post
title:      "git 合并commit的步骤"
date:       2016-07-10 11:00:00
author:     "Tang"
header-img: "img/git.jpg"
catalog:    false
tags:
    - git
    - 备忘录
---
# 1.关于合并commit的步骤

如果是别人的项目，那就首先frok项目，然后克隆到本地，如果是自己的就直接clone到本地。

    $ git clone <版本库的网址>

![image](../../../../img/git-rebase-img/1.jpg)

若要合并`second commit`和`third commit`，则

    $ git rebase -i <first commit 的SHA-1>

其中，-i 的参数是不需要合并的 commit 的 hash 值，这里指的是first commit， 接着我们就进入 编辑模式
例如此例就是a的SHA-1 (可用git log查看该commit的版本号)

    $ git rebase -i 768d796670519c00600d99a90fcfad31182a0773

![image](../../../../img/git-rebase-img/2.jpg)

pick是会执行该commit，而squash会把这个版本的commit合并到前一个commit。
应该改成这样，会合并`third commit` 和 `second commit`

![image](../../../../img/git-rebase-img/3.jpg)

进入编辑模式，命名 合并后的新commit的信息，

![image](../../../../img/git-rebase-img/4.jpg)

一开始的三个commit,最后只剩下合并后的`first commit` 和 `second and third commits`，如图

![image](../../../../img/git-rebase-img/5.jpg)

### 最后

    $ git push origin develop --force
    
    --force 强制合并远程仓库，防止出现错误（因为本地合并之后的commit和远程还没有合并的commit之间会有冲突）

再发出合并请求