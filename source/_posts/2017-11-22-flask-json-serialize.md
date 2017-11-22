---
title: flask对象列表序列化
date: 2017-11-22 18:25:09
category: Learn
toc: true
tags: 
    - flask
    - json
---

之前不管是java还是php，从数据库取出数据库对象之后，转换成json字符串总是很方便的。

但是在python的 flask框架却是没这么简单

<!-- more -->

## flask对象列表序列化

就拿根据用户id取出订单为例：

```py
# database.py
class Indent(db.Model):

    # 表名
    __tablename__ = 'x_indent'

    # 列对象
    id = db.Column(db.Integer,primary_key=True)
    number = db.Column(db.String(255))
    user_id = db.Column(db.Integer)
    price = db.Column(db.String(255))
    address = db.Column(db.String(255))
    add_time = db.Column(db.Integer)

    # 初始化对象
    def __init__(self, number=None,user_id=None,price=None,address=None,add_time=None):
        self.number = number
        self.user_id = user_id
        self.price = price
        self.address = address
        self.add_time = add_time
    
    # 插入记录
    def insert(self, indent):
        db.session.add(indent)
        db.session.commit()
```

```py
# app_url.py
# 订单接口
@mod.route('/indent',methods=['POST'])
def indent():
    # 从url里取出参数
    userId = request.args.get('userId')
    # 从数据库取出订单数据
    indent = Indent.query.filter_by(user_id=userId).all()
    return json.dumps(indent)
```

* json.dumps()  编码json数据
* json.loads()  解码json数据

但如果简单这样写的话，是会报错的，

![](2.png)

Flask还有个内置的 `jsonify()`，

> 其中，jsonify的作用是，把dict或list转换为string(类似于json.dumps())。

在 `database.py`的 `Indent`类中加入这个方法

```py
def serialize(self):
    return {
        'number': self.number,
        'user_id': self.user_id,
        'price': self.price,
        'address': self.address,
        'add_time': self.add_time,
    }
```

然后使用 ***列表推导*** 将对象列表转换为可序列化值列表:

```py
# app_url.py
@mod.route('/indent',methods=['POST'])
def indent():
    userId = request.args.get('userId')
    indent = Indent.query.filter_by(user_id=userId).all()
    
    return jsonify(
        indent = [i.serialize() for i in indent])
```

获得的json字符串

![](1.png)

## 列表推导式

> 列表推导式可以使用非常简洁的方式对列表或其他可迭代对象的元素进行遍历和过滤，快速生成满足特定需求的列表，代码具有非常强的可读性，是Python程序开发时应用最多的技术之一。Python的内部实现对列表推导式做了大量优化，可以保证很快的运行速度，也是推荐使用的一种技术。列表推导式的语法形式为：

[表达式 for 变量 in 序列或迭代对象 if 条件表达式]

列表推导式在逻辑上等价于一个循环语句，只是形式上更加简洁。例如，

```py
aList = [x*x for x in range(10)]

# 相当于

aList = []
for x in range(10):
aList.append(x*x)
```

## 借鉴

* [Flask jsonify一个对象列表](http://www.developerq.com/article/1494141355)
* [详解Python列表推导式](http://qkxue.net/info/115754/Python)



