---
title: springcloud+nacos+seata 实现分布式事务管理
toc: true
date: 2020-01-10 10:00:43
categories: springcloud
tags:
    - seata
---

`Seata` 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。`Seata` 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。

<!-- more -->

- Seata的整合样例 [Seata Samples](https://github.com/seata/seata-samples.git)

如果​有​疑惑​的​可以去​看​看官方样例是如何使用的​，​依样画葫芦总是会简单​很多:smile:



## 下载启动seata

在 [Seata](https://github.com/seata/seata) 克隆代码，然后用idea打开启动

![Seata Samples](1.png)

### seata-server模块

```bash /server/src/main/resources/register.conf
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
这里使用 nacos 作为注册中心，并采用 file.conf 配置的形式
{% endnote %}

```bash /server/src/main/resources/file.conf
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

### 创建seata数据库

seata 选择 store.mode = "db"，则需要创建数据库。

seata 所需的三个数据库，对应的sql的文件在根目录下的 `/script/server/db/mysql.sql`

![seata mysql](4.png)

![seata sql](2.png)

然后就可以启动 seata-server，当然在此之前已经将nacos启动了，地址为`localhost;8848`



## 配置微服务 Account-Server 

 **account-server**、**order-server** 和 **storage-server** 差不多，就只展示一个的配置，

```xml pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.xiaotang</groupId>
    <artifactId>spring-account</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-account</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
        <spring-cloud-alibaba-dependencies.version>0.2.2.RELEASE</spring-cloud-alibaba-dependencies.version>
        <seata.version>1.0.0</seata.version>
        <spring-cloud-alibaba-seata.version>2.1.1.RELEASE</spring-cloud-alibaba-seata.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
         <!-- seata -->
        <dependency>
            <groupId>io.seata</groupId>
            <artifactId>seata-spring</artifactId>
            <version>${seata.version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-seata</artifactId>
            <version>${spring-cloud-alibaba-seata.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
            <scope>compile</scope>
        </dependency>
        <!-- feign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba-dependencies.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### 修改 application.yml 文件

自定义事务组的名称 `tx-service-group: account-service-group`

```yaml application.yml
server:
  port: 8021

spring:
  application:
    name: account-server
  cloud:
    nacos:
      discovery: #nacos注册地址
        server-addr: 127.0.0.1:8848
    # seata 服务分组，要与服务端file.conf中service.vgroup_mapping的后缀对应
    alibaba:
      seata:
        tx-service-group: account-service-group
  datasource:
    url: jdbc:mysql://localhost:3306/account?characterEncoding=utf8
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: 123456
mybatis:
  mapper-locations: classpath:/mapper/*.xml
  type-aliases-package: com.xiaotang.springaccount

logging:
  level:
    org:
      springframework:
        boot:
          autoconfigure: ERROR
```

### 添加并修改 registry.conf 配置文件

主要是将注册中心改为 nacos

```bash /src/main/resources/registry.conf
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    serverAddr = "localhost:8848"
    namespace = "public"
    cluster = "default"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "file"
  nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
  }
}
```

### 添加并修改 file.conf 配置文件，

主要是修改自定义事务组名称

```bash /src/main/resources/file.conf
service {
  #transaction service group mapping
  vgroup_mapping.account-service-group="default"
  # only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "127.0.0.1:8091"
  # disable seata
  disableGlobalTransaction = false
}
```

{% note danger %}
vgroup_mapping.account-service-group="default" 必须配置，不然会一直查找这个配置，找不到控制台会报ERROR，同一个事务组的名称必须一致，在这里所有微服务的事务组名称必须都是 `account-service-group`。

其他配置可以不写，找不到会使用默认配置
{% endnote %}

![ERROR](3.png)

---

### 在启动类中取消数据源的自动创建

`exclude = DataSourceAutoConfiguration.class`，不去除就会报错。

```java com.xiaotang.springaccount.SpringAccountApplication;
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
@EnableFeignClients
@EnableDiscoveryClient
@MapperScan("com.xiaotang.springaccount.dao")
public class SpringAccountApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringAccountApplication.class, args);
    }
}
```

### 配置 DataSourceProxyConfig 使用 Seata 对数据源进行代理

```java com.xiaotang.springaccount.config.DataSourceProxyConfig
import io.seata.rm.datasource.DataSourceProxy;

@Configuration
public class DataSourceProxyConfig {

    // mapper.xml 映射文件坐标
    @Value("${mybatis.mapper-locations}")
    private String mapperLocations;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource(){
        return new DruidDataSource();
    }

    @Primary
    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(mapperLocations));
        // 添加数据库拦截器
        sqlSessionFactoryBean.setPlugins(new Interceptor[]{new InsertInterceptor()});
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }
}
```

### 使用 @GlobalTransactional 注解开启全局分布式事务

```java com.xiaotang.springaccount.service.AccountService
@Service
public class AccountService {

    @Autowired
    private AccountMapper accountMapper;

    @Autowired
    private OrdersClient ordersClient;

    @Autowired
    private StorageClient storageClient;

    public Integer add(Account account){
        return accountMapper.insert(account);
    }
	// 全局事务
    @GlobalTransactional
    @Transactional(rollbackFor = Exception.class)
    public Integer addTest(Account account, Integer status){
        // 通过Feign调用order-server、storage-server接口
        ordersClient.add("banana", 2, 1, account.getUsername());
        storageClient.add("apple", 3);
        Integer index = add(account);
        if(status == 0){
            throw new RuntimeException("hello world");
        }
        return index;
    }
}
```

<<<<<<< HEAD
给每个库都创建创建 undo_log 表（日志回滚表 ）} ，文件在 **seata/script/client/at/db/mysql.sql**
=======
给每个库都创建undo_log 表（日志回滚表 ） ，文件在 **seata/script/client/at/db/mysql.sql**
>>>>>>> d8fb6bfffba2c472d2766ddf2a4e435dd3786beb

```sql
CREATE TABLE IF NOT EXISTS `undo_log`(
    `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT COMMENT 'increment id',
    `branch_id`     BIGINT(20)   NOT NULL COMMENT 'branch transaction id',
    `xid`           VARCHAR(100) NOT NULL COMMENT 'global transaction id',
    `context`       VARCHAR(128) NOT NULL COMMENT 'undo_log context,such as serialization',
    `rollback_info` LONGBLOB     NOT NULL COMMENT 'rollback info',
    `log_status`    INT(11)      NOT NULL COMMENT '0:normal status,1:defense status',
    `log_created`   DATETIME     NOT NULL COMMENT 'create datetime',
    `log_modified`  DATETIME     NOT NULL COMMENT 'modify datetime',
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8 COMMENT ='AT transaction mode undo table';
```



##  启动服务功能演示

{% tabs 选项卡 %}

<!-- tab seata-server -->
![seata-server](5-0.png)

<!-- endtab -->

<!-- tab account-server -->
![account-server](5-1.png)
<!-- endtab -->
<!-- tab order-server -->
![order-server](5-2.png)
<!-- endtab -->
<!-- tab storage-server -->
![storage-server](5-3.png)
<!-- endtab -->

{% endtabs %}

成功注册到seata

---

当我在添加完所有的数据后，最后抛出一个RuntimeException，就会看到order-server和storage-server 最后回滚了操作。

{% tabs 选项卡 2 %}

<!-- tab order-server -->
![order-server](6-1.png)

<!-- endtab -->

<!-- tab storage-server -->
![storage-server](6-2.png)
<!-- endtab -->

{% endtabs %}

## 参考链接

- [Spring Alibaba Cloud使用Seata实现分布式事务，Nacos作为配置中心(一)](https://juejin.im/post/5ddddd75e51d45330c6aec6f)

