---
layout:     post
title:      "git创建项目"
subtitle:   "关于如何创建项目，并书写readme.md(可用来介绍项目，以及一些技术工具的介绍分享)"
date:       2016-07-10 11:00:00
author:     "Tang"
header-img: "img/git.jpg"
catalog:    false
tags:
    - git
    - 备忘录
---

## 1.首先我们需要创建一个项目，有两种方式：
![image](../../../../img/git-create-project-img/1.jpg)
![image](../../../../img/git-create-project-img/2.jpg)

之后，我们会进入项目的创建页面

![image](../../../../img/git-create-project-img/3.jpg)

这就是我们项目的首页，

![image](../../../../img/git-create-project-img/4.jpg)

然后我们就需要clone到本地，进行项目的更新，修改。

当然在此之前，我们需要在电脑上下载 [git](https://git-scm.com/),还有一些需要配置的，可以参考 [git初始配置](https://git-scm.com/book/zh/v1/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git);

![image](../../../../img/git-create-project-img/5.jpg)

然后在你本地找一个文件夹当做存放git项目的仓库，在该文件夹下，单击右键选择 `Git Bash Here`，输入 `git clone git@github.com:tangliangdong/git-book.git`

![image](../../../../img/git-create-project-img/6.jpg)

这样就算克隆成功了。这就算刚克隆下来的项目的根目录。

![image](../../../../img/git-create-project-img/7.jpg)

用编辑器打开后就能用[markdown](http://www.jianshu.com/p/q81RER)语法进行书写，markdown可以去学习下，不难

![image](../../../../img/git-create-project-img/8.jpg)

在我们完成后，就可以向仓库提交文件。

在我们的项目的根目录下，打开git(就是之前提到的打开步骤)，输入`git add .` (意思是将所有更新过的文件放到暂存库，等待提交)。

![image](../../../../img/git-create-project-img/9.jpg)

再输入`git commit -m 'first'` (first代表的是我们给这个commit标注的信息)提交到本地仓库，我们可以通过输入`git log` 来查看我们提交的历史记录。

![image](../../../../img/git-create-project-img/10.jpg)

然后我们可以将其提交到github远程仓库，输入`git push origin master`

![image](../../../../img/git-create-project-img/11.jpg)

这样我们就成功的同步到了github上，我们可以去github上就会发现我们多了一次commit，

![image](../../../../img/git-create-project-img/12.jpg)

这里讲的步骤比较简单，具体的可以看[git教程](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%85%B3%E4%BA%8E%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6)