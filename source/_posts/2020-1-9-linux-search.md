---
title: 日志文件查看搜索
toc: true
date: 2020-01-09 16:47:23
categories: 后端
tags:
    - linux
---



筛查日志这件事，根据情况不同，采用的方法也会有所不同。比如日志很小，至多几千行这样的情况，我们完全可以使用一般的文本编辑器打开，直接查找所需内容即可。而像一些“大型”日志，尤其是长时间稳定性测试所产生的日志，动辄可能会有几个g，几十g，再用编辑器打开显然不够现实。

<!-- more -->

## grep命令

>  grep命令是linux下最好用的命令之一。grep用来筛选内容的速度应该是最快的，这点没有之一，大到几个g的文件，几秒就可以完成对单一关键词的筛取，可谓是查找大文件的“神器”，而且grep命令格式十分简单，常用的搜索功能只需三个参数即可完成。



```bash
grep [option] patten file
```

| 缩写          | 全称                               | 备注                                                       |
| ------------- | ---------------------------------- | ---------------------------------------------------------- |
| -a            | --text                             | 不要忽略二进制的数据。                                     |
| -A <显示行数> | --after-context=<显示行数>         | 显示匹配行之{% label danger@后 %}的n行的内容               |
| -b            | --byte-offset                      | 在匹配行之前，标示出该行第一个字符的编号。                 |
| -B <显示行数> | --before-context=<显示行数>        | 显示匹配行之{% label danger@前 %}的n行的内容               |
| -c            | --count                            | 统计符合样式的列数                                         |
| -C <显示行数> | --context=<显示行数>或 -<显示行数> | 显示匹配行{% label danger@前后 %}前后的n行的内容           |
| -i            | --ignore-case                      | 忽略字符大小写差异                                         |
| -n            | --line-number                      | 显示符合样式的行号                                         |
| -r            | --recursive                        | 遍历文件夹查找                                             |
| -l            | --file-with-matches                | 列出文件内容符合指定的样式的文件名称                       |
| -L            | --files-without-match              | 列出文件内容{% label danger@不符合%}指定的样式的文件名称。 |
| -v            | --revert-match                     | 显示不包含匹配文本的所有行                                 |



{% note danger %}
如果 grep 命令打印 Binary file catalina.out matches，则需要加上 -a 参数
{% endnote %}



| 优点                                         | 缺点                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| *快速，可批量筛选出含有关键词的全部文本行。* | *如果关键词在文本中出现较多，无法快速定位至某一次关键词出现的位置，依然会出现刷屏效果。* |



## more / less 命令

> more和less命令都是用分页查看文本的方式，每次只显示一定行的文本，避免像cat那样被大量的文字快速刷屏，同时支持搜索，可以在文件中搜索某个关键词并实现定位。

- **less** 似乎更适合对于日志的筛查，可以进行向前或向后**双方向的搜索**，并且可以按方向键**逐行前后滚动**
- **more** 只支持向后查找和向后翻页或滚动。

```bash
more/less file
```

**搜索关键字：**输入命令后按"/"，输入关键词后回车即可定位至关键词第一次出现的位置，此时按 **n键** 可切换至下一次出现的位置，使用less时，按 **N键** 返回上一次出现的位置。

| 优点                                                         | 缺点                                               |
| ------------------------------------------------------------ | -------------------------------------------------- |
| *可以自动定位关键词出现的位置，并显示关键词前后的文本内容，使用起来比较方便。* | *搜索速度较慢，文件特别大的话要等很久才能搜索到。* |



## **head / tail 命令**

> head和tail命令是功能近似而作用位置相反的两个命令，head命令用来从开头读取文本，tail命令则是从尾部读取文本。当我们不关注日志中间的一大坨内容，只关注开头或结尾的部分内容时，head和tail命令可以说是最好的解决方案。

```bash
# 显示开头到第n行
head -n file

# 从倒数第n行开始显示到末尾
tail -n file

# 从文件开头第n行开始显示到末尾
tail +n file
```

**使用方法：**设置需要从文件开头 / 结尾查找的行数（n），即可显示对应结果。

| 优点                                         | 缺点                                                |
| -------------------------------------------- | --------------------------------------------------- |
| *方便实用，尤其是tail，可以从尾部读取文件。* | *单独使用不能查找关键词，需用管道结合 grep进行查找* |

{% note info%}
**head / tail和 grep 结合使用**

{% endnote %}

```bash
head/tail -n file | grep pattern
```



## 参考链接

- [大日志，看我如何对付你](https://cloud.tencent.com/developer/article/1484955)