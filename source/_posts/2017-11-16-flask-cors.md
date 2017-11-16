---
title: flask-cors实现CORS跨域请求
date: 2017-11-16 15:13:23
category: Learn
tags: 
    - flask
    - python
---

在做ionic登录的时候碰到的跨域请求的问题，因为缺少 `Access-Control-Allow-Origin`，

可以通过安装 `flask-cors` 来解决问题

```
pip3 install flask-cors
```

在出现 `app = Flask(__name__)` 这段的地方加上如下代码。即可实现跨域请求

```
from flask_cors import CORS
app = Flask(__name__)
CORS(app)
```



