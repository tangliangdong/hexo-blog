---
title: springcloud+nacos+seata 实现分布式事务管理
toc: true
date: 2020-01-10 10:00:43
categories: springcloud
tags:
    - seata
---



<!-- more -->

- Seata的整合样例 [Seata Samples](https://github.com/seata/seata-samples.git)

如果​有​疑惑​的​可以去​看​看官方样例是如何使用的​，​依样画葫芦总是会简单​很多:smile:



## 下载启动seata

在 [Seata](https://github.com/seata/seata) 克隆代码，然后用idea打开启动

![Seata Samples](1.png)

进入其中的server模块



```text /server/src/main/resources/register.conf
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    serverAddr = "localhost"
    namespace = "public"
    cluster = "default"
  }
}

config {
  # file、nacos、apollo、zk、consul、etcd3
  # 配置中心的选择
  type = "file"

  nacos {
    serverAddr = "localhost"
    namespace = "public"
  }
  
  file {
    name = "file.conf"
  }
}
```

{% note info %}
我这里先不nacos动态配置的方式，而是用 file.conf 配置的形式
{% endnote %}

```text /server/src/main/resources/file.conf
service {
  #transaction service group mapping
  vgroup_mapping.my_test_tx_group = "default"
  # 这里的是之后对每个微服务进行配置所需要用到的
  service.vgroup_mapping.order-service-fescar-service-group="default"
  service.vgroup_mapping.account-service-fescar-service-group="default"
  # only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "127.0.0.1:8091"
  # disable seata
  disableGlobalTransaction = false
}

## transaction log store, only used in seata-server
store {
  ## store mode: file、db
  mode = "db"

  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://116.62.14.208:3306/seata"
    user = "root"
    password = "123456"
  }
}
```

再要创建seata所需的数据库，对应的sql的文件在根目录下的 `/script/server/db/mysql.sql`

![seata sql](2.png)

然后就可以启动 seata-server，当然在此之前已经将nacos启动了，`localhost;8848`



## 配置微服务



