---
title: springboot分布式事务atomikos
date: 2019-12-09 14:56:21
toc: true
category: Learn
tags:
    - springboot
    - atomikos
---



场景：现有两个不同的数据库，一个叫db_user，一个叫db_account。一个操作，要同时更新db_user的user表和db_account的account表。失败，则两个表一起回滚。



<!--more-->

### 项目目录：

![项目目录结构](1.png)

- *com.example.atomikos.config*  数据源配置信息
- *com.example.atomikos.db1*  数据库db_user的业务和对象
- *com.example.atomikos.db2*  数据库db_account的业务和对象
- *resources/mapper/user*  db_user数据库的映射文件
- *resources/mapper/account*  db_account数据库的映射文件



### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.11.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>atomikos</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>atomikos</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- mybatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>

        <!-- mysql数据库连接包，需指定版本，不然会使用8.0的jar包-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>
        <!-- alibaba的druid数据库连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.9</version>
        </dependency>

        <!-- jta-atomikos 分布式事务管理 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jta-atomikos</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>jakarta.persistence</groupId>
            <artifactId>jakarta.persistence-api</artifactId>
            <version>2.2.3</version>
        </dependency>
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
            <version>3.5.3</version>
        </dependency>
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-autoconfigure</artifactId>
            <version>1.2.4</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <configurationFile>${basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
            </plugin>
            
            <!--在application.yml文件中使用@占位符-->
            <plugin>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.7</version>
                <configuration>
                    <delimiters>
                        <delimiter>@</delimiter>
                    </delimiters>
                    <useDefaultDelimiters>false</useDefaultDelimiters>
                    <!--防止ico二进制文件损坏-->
                    <nonFilteredFileExtensions>
                        <nonFilteredFileExtension>ico</nonFilteredFileExtension>
                    </nonFilteredFileExtensions>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <!-- 开发环境 -->
        <profile>
            <id>dev</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <env>dev</env>
            </properties>
        </profile>
        <!-- 测试环境 -->
        <profile>
            <id>test</id>
            <properties>
                <env>test</env>
            </properties>
        </profile>
        <!-- 生产环境 -->
        <profile>
            <id>pro</id>
            <properties>
                <env>pro</env>
            </properties>
        </profile>
    </profiles>
</project>
```

> 连接的mysql数据库是5.7的，因此使用 `mysql-connector-java`的是5.1的版本，而mysql 6以上的数据库则需要使用 `mysql-connector-java`6.0以上，对应的驱动为 `com.mysql.cj.jdbc.Driver`



### application.yml

```yaml
server:
  port: 8091

spring:
  profiles:
    active: @env@

## 该配置节点为独立的节点，不是在在spring的节点下
mybatis:
  mapper-locations: classpath:mapping/*/*.xml  #注意：一定要对应mapper映射xml文件的所在路径
  type-aliases-package: com.example.atomikos.model  # 注意：对应实体类的路径
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl #控制台打印sql
```



### application-dev.yml

```yaml

spring:
  # 开发环境配置
  profile: dev
  datasource:
    druid:
      one:  #数据源1
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://116.62.14.208:3306/db_user?useUnicode=true&amp;characterEncoding=UTF-8
        username: root
        password: 123456
        #初始化时建立物理连接的个数
        initialSize: 1
        #池中最大连接数
        maxActive: 20
        #最小空闲连接
        minIdle: 1
        #获取连接时最大等待时间，单位毫秒
        maxWait: 60000
        #有两个含义：
        #1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。
        #2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明
        timeBetweenEvictionRunsMillis: 60000
        #连接保持空闲而不被驱逐的最小时间，单位是毫秒
        minEvictableIdleTimeMillis: 300000
        #使用该SQL语句检查链接是否可用。如果validationQuery=null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。
        validationQuery: SELECT 1 FROM DUAL
        #建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
        testWhileIdle: true
        #申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
        testOnBorrow: false
        #归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
        testOnReturn: false
        # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
        filters: stat,wall,slf4j
        # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
        #connectionProperties.druid.stat.mergeSql: true
        #connectionProperties.druid.stat.slowSqlMillis: 5000
        # 合并多个DruidDataSource的监控数据
        #useGlobalDataSourceStat: true
        #default-auto-commit: true 默认
        #default-auto-commit: false
      two: #数据源2
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://116.62.14.208:3306/db_account?useUnicode=true&amp;characterEncoding=UTF-8
        username: root
        password: 123456
        #初始化时建立物理连接的个数
        initialSize: 1
        #池中最大连接数
        maxActive: 20
        #最小空闲连接
        minIdle: 1
        #获取连接时最大等待时间，单位毫秒
        maxWait: 60000
        #有两个含义：
        #1) Destroy线程会检测连接的间隔时间，如果连接空闲时间大于等于minEvictableIdleTimeMillis则关闭物理连接。
        #2) testWhileIdle的判断依据，详细看testWhileIdle属性的说明
        timeBetweenEvictionRunsMillis: 60000
        #连接保持空闲而不被驱逐的最小时间，单位是毫秒
        minEvictableIdleTimeMillis: 300000
        #使用该SQL语句检查链接是否可用。如果validationQuery=null，testOnBorrow、testOnReturn、testWhileIdle都不会起作用。
        validationQuery: SELECT 1 FROM DUAL
        #建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
        testWhileIdle: true
        #申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
        testOnBorrow: false
        #归还连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能。
        testOnReturn: false
        # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
        filters: stat,wall,slf4j
        # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
        #connectionProperties.druid.stat.mergeSql: true
        #connectionProperties.druid.stat.slowSqlMillis: 5000
        # 合并多个DruidDataSource的监控数据
        #useGlobalDataSourceStat: true
        #default-auto-commit: true 默认
        #default-auto-commit: false

```



### 启动类

```java
package com.example.atomikos;

import tk.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan({"com.example.atomikos.db1.dao","com.example.atomikos.db2.dao"})
public class AtomikosApplication {

    public static void main(String[] args) {
        SpringApplication.run(AtomikosApplication.class, args);
    }

}
```



### 第一个数据源配置Properties

```java
package com.example.atomikos.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

// @Data注解 提供类所有属性的 get 和 set 方法，此外还提供了equals、canEqual、hashCode、toString 方法。
@Data
@Component
@ConfigurationProperties(prefix = "spring.datasource.druid.one")
public class OneDataSourceProperties {
    private String driverClassName;
    private String url;
    private String username;
    private String password;
    private Integer initialSize;
    private Integer maxActive;
    private Integer minIdle;
    private Integer maxWait;
    private Integer timeBetweenEvictionRunsMillis;
    private Integer minEvictableIdleTimeMillis;
    private String validationQuery;
    private Boolean testWhileIdle;
    private Boolean testOnBorrow;
    private Boolean testOnReturn;
    private String filters;
}
```



### 第二个数据源配置Properties

```java
package com.example.atomikos.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Data
@Component
@ConfigurationProperties(prefix = "spring.datasource.druid.two")
public class TwoDataSourceProperties {
    private String driverClassName;
    private String url;
    private String username;
    private String password;
    private Integer initialSize;
    private Integer maxActive;
    private Integer minIdle;
    private Integer maxWait;
    private Integer timeBetweenEvictionRunsMillis;
    private Integer minEvictableIdleTimeMillis;
    private String validationQuery;
    private Boolean testWhileIdle;
    private Boolean testOnBorrow;
    private Boolean testOnReturn;
    private String filters;
}
```

### 第一个数据源配置

```java
package com.example.atomikos.config;

import com.alibaba.druid.pool.xa.DruidXADataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import tk.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.jta.atomikos.AtomikosDataSourceBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;

import javax.sql.DataSource;

@Configuration
//这里要指明这个数据适用于哪些mapper，和这个数据源的sqlsessionFactory
@MapperScan(basePackages = "com.example.atomikos.db1.dao", sqlSessionFactoryRef = "oneSqlSessionFactory")
public class OneDataSourceConfiguration {
    @Autowired
    public OneDataSourceProperties oneDataSourceProperties;

    //配置第一个数据源
    @Primary
    @Bean(name = "oneDataSource")
    public DataSource oneDataSource() {
        // 这里datasource要使用阿里的支持XA的DruidXADataSource
        DruidXADataSource datasource = new DruidXADataSource();
        BeanUtils.copyProperties(oneDataSourceProperties,datasource);
        AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
        xaDataSource.setXaDataSource(datasource);
        xaDataSource.setUniqueResourceName("oneDataSource");
        return xaDataSource;
    }

    //配置第一个sqlsessionFactory
    @Primary
    @Bean(name = "oneSqlSessionFactory")
    public SqlSessionFactory oneSqlSessionFactory(@Qualifier("oneDataSource") DataSource oneDataSource)
            throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(oneDataSource);
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
	bean.setMapperLocations(resolver.getResources("classpath:mapper/user/*.xml"));
        return bean.getObject();
    }
}
```



### 第二个数据源配置

```java
package com.example.atomikos.config;

import com.alibaba.druid.pool.xa.DruidXADataSource;
import com.example.atomikos.config.TwoDataSourceProperties;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import tk.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.jta.atomikos.AtomikosDataSourceBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.core.io.support.ResourcePatternResolver;

import javax.sql.DataSource;

@Configuration
@MapperScan(basePackages = "com.example.atomikos.db2.dao", sqlSessionFactoryRef = "twoSqlSessionFactory")
public class TwoDataSourceConfiguration {
    @Autowired
    public TwoDataSourceProperties twoDataSourceProperties;

    @Bean(name = "twoDataSource")
    public DataSource twoDataSource() {
        DruidXADataSource datasource = new DruidXADataSource();
        BeanUtils.copyProperties(twoDataSourceProperties,datasource);
        AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
        xaDataSource.setXaDataSource(datasource);
        xaDataSource.setUniqueResourceName("twoDataSource");
        return xaDataSource;
    }

    @Bean(name = "twoSqlSessionFactory")
    public SqlSessionFactory twoSqlSessionFactory(@Qualifier("twoDataSource") DataSource twoDataSource)
            throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(twoDataSource);
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        bean.setMapperLocations(resolver.getResources("classpath:mapper/account/*.xml"));
        return bean.getObject();
    }
}
```

### service层使用事务回滚演示

```java
package com.example.atomikos.db1.service.user;
import com.example.atomikos.db1.dao.user.UserMapper;
import com.example.atomikos.db1.model.user.User;
import com.example.atomikos.db2.model.account.Account;
import com.example.atomikos.db2.service.account.AccountService;
import org.springframework.util.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import javax.transaction.Transactional;
import java.util.List;

/**
* @ClassName: UserService
* @Description:
* @author tangliangdong
* @date 2019-12-6
*/
@Service("userService")
public class UserService{
    @Autowired
    private UserMapper userMapper;

    @Autowired
    private AccountService accountService;

    //---------增删改查基础部分S--------
    //保存
    public Integer save(User user){
        return userMapper.insert(user);
    }

    @Transactional
    public String testAtomikos(String name){
        Account account = new Account();
        account.setName(name);
        accountService.save(account);
        User user = new User();
        user.setName(name);
        save(user);
        int i = 1 / 0;
        return "done";
    }
}
```

> 注意@Transactional 引入的包是`javax.transaction.Transactional`



源码地址 [springboot-atomikos](https://github.com/tangliangdong/springboot-atomikos)

参考自：[【十九】Spring Boot之分布式事务(JTA、Atomikos、Druid、Mybatis)](https://blog.csdn.net/jy02268879/article/details/84398657)