---
title: Flask-SQLAlchemy 操作
date: 2017-11-22 22:22:57
category: Learn
toc: true
tags: 
    - flask
---

> SQLAlchemy是Python最广泛使用的一个ORM(对象关系映射，简单地说就是把数据库的表即各种操作映射到Python对象上面来)工具。它支持操作PostgreSQL、MySQL、Oracle、Microsoft SQL Server、SQLite等支持SQL的数据库。 👉👉👉 [文档地址](http://docs.sqlalchemy.org/en/latest/contents.html)

<!-- more -->

# Model/数据表定义

## 表定义

```py
# 订单类
class Indent(db.Model):

    # 表名
    __tablename__ = 'x_indent'

    # 列对象
    id = db.Column(db.Integer,primary_key=True)
    number = db.Column(db.String(255))
    user_id = db.Column(db.Integer)
    address = db.Column(db.String(255))

    def __init__(self,number=None,user_id=None,address=None):
        self.number = number
        self.user_id = user_id
        self.address = address
```

## 列定义

```py
# 列类型
## 数字
BigInteger	# 长整型
Boolean		# 布尔值
Enum		# 枚举值，例如class MyEnum(enum.Enum): one=1 two =2. 定义时候Enum(MyEnum)
Float
SmallInteger
Integer(unsigned=False)		# 整型
Interval
Numeric

## 字符
JSON
LargeBinary(length=None)	# 二进制
PickleType	# pickle类型
SchemaType
String(50)	# 字符串类型，括号里表示长度
Text(length=None)
Unicode
UnicodeText

## 时间
Date
DateTime	# daatetime.datetime()对象
Time		# datetime.time()对象
TIMESTAMP	# 时间戳

# 关联列属性
fullname = column_property(firstname + ' ' + lastname)	# 表示这一列的值由指定的列值确定

# 列属性
primary_key=True	# 是否是主键
```

# 查询

## 普通查询

```py
# 查询表
query = session.query(User)
query		# 打印sql语句
query.count()
query.statement	# 同上
query.all()		# 获取所有数据
session.query(Indent.id).distinct().all()
query.limit(2).all()
query.offset(2).all()
query.first()
query.get(2)	# 根据主键获取
query.filter(Indent.id==2, address=='杭州').first().name
query.filter('id = 2').first()	# 复杂的filter
query.order_by('number').all()		# 排序
query(func.count('*')).all()
# 查询列
session.query(Indent.address)	# 去除指定列
session.query(Indent.id, Indent.number)
# 拼接
query2 = query.filter(Indent.id > 10)	# 拼接相当于AND
query2.filter(or_(Indent.id == 1))	# or操作
```

## 连表查询

```py
data = db.session.query(IndentProduct.product_id,Indent.address).\
            join(Indent, Indent.id == IndentProduct.indent_id).\
            filter(IndentProduct.indent_id==indentId).\
            all()
print(jsonify(data))
```

> 在连表查询的时候如果不做处理，最后返回的不是带键值对的 `dict`。

<img src="1.png" width="50%" />

列表中的项并不是标准的 Python tuple，`<class 'sqlalchemy.util._collections.result'>`，它是一个 [AbstractKeyedTuple](https://github.com/zzzeek/sqlalchemy/blob/master/lib/sqlalchemy/util/_collections.py#L22) 对象，拥有一个 keys() 方法，这样可以很容易将其转换成 dict ：

```py
list = [dict(zip(result.keys(), result)) for result in data]
print(jsonify(list))
```

<img src="2.png" width="50%" />

## 筛选字段

> 除了一开始在 `query()` 指定需要的字段，还可以使用 `SQLAlchemy` 提供的  [with_eitities()](http://docs.sqlalchemy.org/en/latest/orm/query.html#sqlalchemy.orm.query.Query.with_entities) 方法

```py
data = db.session.query(IndentProduct).\
            join(Indent, Indent.id == IndentProduct.indent_id).\
            filter(IndentProduct.indent_id==indentId).\
            with_entities(IndentProduct.product_id,Indent.address).\
            all()
print(jsonify(list))
```

==注意filter()里面的表示两个相等的是 \==，不是 \===

# 插入数据

```py
def insert(self, indent):
    db.session.add(indent)
    db.session.commit()
```

# 删除数据

```py
def delete(id):
    db.session.query.filter(Indent.id==id).first()
    db.session.delete(indent)
    db.session.commit()

def deleteById(id):
    db.session.query(Indent).filter(Indent.id == id).delete()
    db.session.commit()
```

# 修改

```py
def update(self,IndentId,address):
    db.session.query(Indent).\
        filter(Indent.id==IndentId).\
        update({Indent.address: address})
        
    session.flush()	# 写数据库，不提交
```

# 其他

```py
session.rollback()	# 回滚：把未提交的数据都回滚
session.commit()	# 提交
```

# 借鉴

* [SQLAlchemy手册 | 豪翔天下](https://haofly.net/sqlalchemy/)
* [在 Flask-SQLAlchemy 中联表查询](https://blog.zengrong.net/post/2656.html)




