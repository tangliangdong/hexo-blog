---
title: nginx部署flask
date: 2017-11-20 18:56:29
category: Learn
toc: true
tags: 
    - nginx
    - centos7
    - flask
    - python3
---

> 以前用的一直都是apache的服务器，加上之前apache部署flask一直失败，就想着用nginx试试，网上也建议用nginx，所有就试试 nginx 来搭建服务器。

<!-- more -->

先把项目传到 `/usr/share/nginx` 目录下

### 安装uwsgi

```bash
pip3 install uwsgin
```

如果报了 致命错误：Python.h：没有那个文件或目录，就去安装 `python-devel`

```bash
yum install python-devel
```

在项目根目录下创建 `uwsgin.ini`，里面写入如下配置

```
[uwsgi]
socket = 127.0.0.1:8001
pythonpath = /usr/share/flask1
socket-timeout=10
module = index
callable = app
processes = 4
chmod-socket = 666
threads = 2
```

* socket: 通讯端口，外界可以通过127.0.0.1:8001访问
* pythonpath: 项目目录
* socket-timeout: 
* module: 启动文件的文件名，我们可以在本地用python3 index.py启动flask项目。
* callable: 程序内启用的application变量名。
* processes: 处理器个数。
* chmod-socket: 权限
* threads: 线程数。

### 配置 Nginx 

打开 nginx 配置文件

```bash
vim /etc/nginx/conf.d/api.conf
```

加上如下的配置代码：

```bash
server {
    listen 80;
    server_name  flask.tangliangdong.me; # 外部域名
    root         /usr/share/nginx/flask; # 项目根目录
    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8001; # uwsgin.ini 配置的 socket 值
        uwsgi_param UWSGI_PYHOME /usr/share/flask1/index; # 项目启动文件 index.py
        uwsgi_param UWSGI_CHDIR /usr/share/flask1; # 项目根目录
        uwsgi_param UWSGI_SCRIPT run:app; 
        # proxy_pass http://127.0.0.1:80; # 反向代理 Gunicorn 本地的服务地址
        # proxy_set_header Host $host;
        # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

> uwsgi_pass需要和上面的socket保持一致。

-----

重启 nginx 

```bash
systemctl restart nginx
```

进入项目根目录执行

```bash
uwsgi uwsgi.ini
```

这样运行起来后，如果访问flask里的请求，会报这个错误 `KeyError: 'REQUEST_METHOD'`

👉👉👉 [Nginx/wsgi部署Django：避开502KeyError: u’REQUEST_METHOD’错误](http://www.h5w3.com/?p=831)

因为上面 `api.conf` 里有 `uwsgi_params`: `include uwsgi_params;`

因此需要在 `/etc/nginx/conf.d` 下创建 `uwsgi_params` 文件，里面加上如下的内容。

```bash
uwsgi_param QUERY_STRING        $query_string;
uwsgi_param REQUEST_METHOD      $request_method;
uwsgi_param CONTENT_TYPE        $content_type;
uwsgi_param CONTENT_LENGTH      $content_length;

uwsgi_param REQUEST_URI         $request_uri;
uwsgi_param PATH_INFO           $document_uri;
uwsgi_param DOCUMENT_ROOT       $document_root;
uwsgi_param SERVER_PROTOCOL     $server_protocol;
uwsgi_param UWSGI_SCHEME        $scheme;

uwsgi_param REMOTE_ADDR         $remote_addr;
uwsgi_param REMOTE_PORT         $remote_port;
uwsgi_param SERVER_PORT         $server_port;
uwsgi_param SERVER_NAME         $server_name;
```

再执行，就可以了。

```bash
systemctl restart nginx

uwsgi uwsgi.ini
```



