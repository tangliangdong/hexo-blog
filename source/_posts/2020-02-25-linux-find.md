---
title: linux find命令常用操作
toc: true
date: 2020-02-25 21:49:39
categories: 后端
tags:
    - linux
---



### 查找指定名称的档案

```bash
find / -name nginx
```

之后会列出所有包含nginx的文件或文件夹的路径

>   查找不区分字母大小写

```bash
find / -iname nginx
```

<!-- more -->

### 仅查找目录或文件

`find `的`-type`参数可以指定所要查找的档案的类型，常用：

-   `-type d`：只查找目录类型。**d** 是directory的首字母，表示 “**目录**”。
-   `-type f`：只查找文件类型。**f** 是file的首字母，表示 “**文件**”。

```bash
# 只查找名为nginx的文件夹
find / -type d -name nginx 

# 只查找名为nginx的文件
find / -type f -name nginx 
```



### 操作查找结果

>   find 命令会默认显示每个查找到文件。

```bash
find / -name "*.jpg"
```

等价于

```bash
find / -name "*.jpg" -print
```

格式化打印结果：

```bash
find / -name "*.jpg" -printf "%p - %u\n"
```

-   %p：文件名
-   %u：文件的所有者

#### 调用命令

`-exec`可以将搜索出来的结果，输入到其他命令中进行再处理。

比如在查找出来的文件中搜索字符串

```bash
# 在所有的nginx.conf文件中搜索8888的字符串，并将包含的文件列出来。
find ./ -name "nginx.conf" -exec grep -l "8888" {} \;

# 在所有的nginx.conf文件中搜索8888的字符串，并将包含的文件和所在行的内容 列出来。
find ./ -name "nginx.conf" -exec grep -n "8888" ./ {} \;
```

-   这个操作不必用双引号括起来
-   **{} **会用查找到的每个文件来替换
-   **\ **用来转义**; 字符**
-   **; **是必须的结尾

{% note info %}
因为测试环境装了很多的nginx，find搜索出来的nginx太多，端口号为nginx的配置文件找不到，通过`ps`和`netstat`命令也无法确定该nginx的目录。因此就将 find 和 grep命令结合起来使用。
{% endnote %}



### 文件的修改与存取时间

-   -atime （access time）在过去n天内被读取过的文件
-   -ctime 在过去n天内被修改过的文件 
-   -amin 在过去 n 分钟内被读取过
-   -cmin 在过去 n 分钟内被修改过

```bash
# 查找7天内被修改过的文件
find / -ctime 7
```



### 根据文件大小查找

使用 `-size`参数可以指定文件的大小

```bash
# 查找大小刚好是50MB的文件
find / -size 50M

# 查找大于50KB的文件
find . -size +50K

# 查找小于1G的文件
finmd . -size -1G
```



---

>   参考自 [[Unix/Linux 的 find 指令使用教學、技巧與範例整理](https://blog.gtwang.org/linux/unix-linux-find-command-examples/)](https://blog.gtwang.org/linux/unix-linux-find-command-examples/)