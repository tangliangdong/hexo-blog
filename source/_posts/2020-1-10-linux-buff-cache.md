---
title: linux-buff-cache
toc: true
date: 2020-01-10 09:57:43
categories: 服务器
tags:
    - linux
---

linux 硬盘缓存对执行性能的影响，linux不只是把你的内存吃了，

<!-- more -->

## 清理内存

我们可以使用特殊的文件 `/proc/sys/vm/drop_caches`，将 3 写入，我们就能清除硬盘缓存

```bash
$ free -m
             total       used       free     shared    buffers     cached
Mem:          1504       1471         33          0         36        801
-/+ buffers/cache:        633        871
Swap:         2047          6       2041

$ echo 3 | sudo tee /proc/sys/vm/drop_caches 
3

$ free -m
             total       used       free     shared    buffers     cached
Mem:          1504        763        741          0          0        134
-/+ buffers/cache:        629        875
Swap:         2047          6       2041
```

## 硬盘缓存对程序加载的影响

让我们制作两个测试程序，一个用 Python，一个用 Java。 Python 和 Java都带有相当大的运行时，必须加载这些运行时才能运行该应用程序。 这是磁盘缓存发挥其魔力的完美方案。

```bash
$ cat Hello.java
class Hello {
    public static void main(String[] args) throws Exception {
        System.out.println("Hello World! Regards, Java");
    }
}

$ javac Hello.java

$ java Hello
Hello World! Regards, Java
```

我们先清除缓存，然后观察运行这两个程序需要多久

```bash
$ echo 3 | sudo tee /proc/sys/vm/drop_caches
3

$ free -h
              total        used        free      shared  buff/cache   available
Mem:           1.8G        1.1G        559M        768K        106M        538M
Swap:            0B          0B          0B

$ time java Hello
Hello World! Regards, Java

real	0m0.350s
user	0m0.056s
sys	0m0.021s

$ free -h
              total        used        free      shared  buff/cache   available
Mem:           1.8G        1.1G        549M        768K        115M        534M
Swap:            0B          0B          0B

$ time java Hello
Hello World! Regards, Java

real	0m0.202s
user	0m0.047s
sys	0m0.015s
```

{% note info%}
明显，第二次执行该程序的时间明显缩短了，而且 buff/cache 也变大了
{% endnote %}



## 硬盘缓存对文件读取速度的影响

我们先创建一个大文件，看看磁盘缓存如何影响我们读取它的速度。我们创建了一个200MB的文件，文件的大小可以自己进行判断。

```bash
$ echo 3 | sudo tee /proc/sys/vm/drop_caches
3

$ free -m
              total        used        free      shared  buff/cache   available
Mem:           1.8G        1.1G        568M        768K        103M        546M
Swap:            0B          0B          0B

$ dd if=/dev/zero of=bigfile bs=1M count=200
200+0 records in
200+0 records out
209715200 bytes (210 MB) copied, 0.483083 s, 434 MB/s

$ ls -lh bigfile
-rw-r--r-- 1 root root 200M Jan 10 10:39 bigfile

$ free -m
              total        used        free      shared  buff/cache   available
Mem:           1.8G        1.1G        362M        768K        310M        529M
Swap:            0B          0B          0B
```

由于文件刚刚被写入，因此它将进入磁盘缓存。 200MB的文件在“缓存”中引起了200MB的增加。 让我们读它，清除缓存，然后再次读以查看它有多快：

```bash
$ time cat bigfile > /dev/null

real	0m0.164s
user	0m0.001s
sys	0m0.053s

$ echo 3 | sudo tee /proc/sys/vm/drop_caches
3

$ time cat bigfile > /dev/null

real	0m1.831s
user	0m0.002s
sys	0m0.079s

```

{% note danger %}
读取的速度快了也有11倍了。
{% endnote %}

> Linux磁盘缓存非常简单。 它使用备用内存来大大提高磁盘访问速度，而不会占用应用程序任何内存。 在Linux上完全使用的ram存储是有效的硬件使用，而不是警告信号。



{% post_link 2020-1-10-file-descriptor 对 > /dev/null 的延伸认识 %}

---



## 参考链接

- 部分翻译自 [Experiments and fun with the Linux disk cache](https://www.linuxatemyram.com/play.html)

