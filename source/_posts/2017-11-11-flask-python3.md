---
title: 安装python3版本的flask
date: 2017-11-11 23:09:52
category: Learn
tags: 
    - flask
    - python
---

Mac 默认安装的是 python2.7的版本，我额外安装了python，通过 `python3` 命令执行python3的。

<!-- more -->

我安装的 python3 默认安装的是 `easy_install-3.6`

通过 `easy_install` 安装 pip

```
sudo easy_install-3.6 pip
```

我安装好后支持 `python3` 的是 `pip3`

安装 flask

```
virtualenv flask1

cd flask1

source bin/activate

pip3 install flask
```

在根目录创建一个最简单的应用 `index.py`，里面输入如下代码：

```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

终端执行命令

```
python3 index.py
```




