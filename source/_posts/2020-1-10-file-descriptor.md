---
title: 文件描述符 与 /dev/null
toc: true
date: 2020-01-10 10:55:21
categories: 服务器
tags:
    - linux
---

> 文件描述符是与文件输入、输出关联的整数。它们用来跟踪已打开的文件。最常见的文件描述符是stidin、stdout、和stderr。我们可以将某个文件描述符的内容重定向到另外一个文件描述符中。
>                                                                                         			——*《linux shell脚本攻略》*

<!-- more -->

## 文件名描述符

- 0 —— stdin（标准输入）
- 1 —— stdout （标准输出）
- 2 —— stderr （标准错误输出）

> 重定向操作是shell命令对应的文件描述符输出的文本信息重新输入到另外一个文件中。

**重定向标准输出stdout：**

```bash
$ ls 123.txt 1>stdout.txt
$ cat stdout.txt
123.txt
```

{% note info %}
标准输出的 **1** 可以省略，默认就是标准输出
{% endnote %}

{% note danger%}
1>stdout.txt，中间不能有任何的空格
{% endnote %}

**重定向标准错误stderr**

```shell
$ ls abc.txt 2>stderr.txt
$ cat stderr.txt
ls: cannot access abc.txt: No such file or directory
```

标准错误的重定向到了 stderr.txt 文件中。



```bash
# 你可以将stderr单独定向到一个文件，将stdout重定向到另一个文件：
$ cmd 2>stderr.txt 1>stdout.txt
# stderr 和 stdout 都重定向到一个文件中
$ cmd > output.txt 2>&1
```



## linux特殊文件

> /dev/null是一个特殊的设备文件，这个文件接收到的任何数据都会被丢弃。因此，null这个设备通常也被成为位桶（bit bucket）或黑洞。
>
> ​															——《linux shell脚本攻略》

重定向操作给这个 /dev/null 文件的所有东西都会被丢弃。因此如果不想让输出结果打印到控制台，就可以将输出重定向到 */dev/null* 文件中。（可以标准错误重定向到 /dev/null 文件中）

## 参考链接

- [shell程序中 2> /dev/null 代表什么意思？ - 裕用ID的回答 - 知乎](https://www.zhihu.com/question/53295083/answer/135258024)