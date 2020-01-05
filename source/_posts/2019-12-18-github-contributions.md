---
title: Github的Contributions没有增加的问题
date: 2019-12-18 16:10:05
category: Learn
tags:
    - github
---

向Github提交代码的时候，发现自己的Contributions 并没有增加，原来是commit的邮件地址必须与github相一致，这也是为什么我的Commit没有被记入Contributions 和不显示头像的原因。

<!-- more -->

我们进入到git仓库中，使用 `git log` 查看每次提交的日志，就会发现用户名和邮箱并没有和Github上保持一致。我们先将commit 时的用户名和邮箱地址改为与Github注册时的一致，防止以后再出现这样的问题，我们可以通过如下的命令：

```bash
git config --global user.name <your username for Github>
git config --global user.email <your email address for Github>
```

修改后，你再向Github提交代码时，就会看到该commit被记入Contributions了。

对于已经提交过的代码，也还能修改提交的用户名和邮箱地址。

- 进入仓库地址（如果本地没有，就从Github clone下来）
- 在git仓库根目录新建一个脚本文件 `script.sh`，并将下面的内容复制进去，修改掉 **<>** 里面的内容，

```bash
#!/bin/bash
git filter-branch --env-filter '
OLD_EMAIL=<旧邮箱地址>
CORRECT_NAME=<正确用户名>
CORRECT_EMAIL=<正确邮箱地址>
if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
export GIT_COMMITTER_NAME="$CORRECT_NAME"
export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
export GIT_AUTHOR_NAME="$CORRECT_NAME"
export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi' --tag-name-filter cat -- --branches --tags
```

然后执行script.sh

```bash
sh script.sh
```

会有一条条commit修改的进度，等待修改完就行了。 

再输入 `git log` 查看commit的用户名和邮箱地址是否已经变更了，

![sh script.sh](1.png)

若是已经成功变更，则再提交到Github上，以前尘封的commit便会重新出现了。

```bash
git push --force --tags origin master
```



参考自 [解决Github的Contribution没有增加的问题](https://blog.csdn.net/Liven_Zhu/article/details/80800162)，感谢！！！