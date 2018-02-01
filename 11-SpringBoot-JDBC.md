---
layout:

title: 11-SpringBoot-JDBC

date: 2017-02-11

updated: 2017-02-11

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- JDBC

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---

# 11-SpringBoot-JDBC

[TOC]

## 11.1 装配DataSource

1. 加入数据库驱动
2. 配置文件中加入如下配置,springboot会自动装配.
3. spring在给我们装配好datasource的同时,会给我们装配一个JDBCTEMPLATE

```yaml
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/springboot
spring.datasource.username=root
spring.datasource.password=xiaojie1996
```

```java
package com.clsaa.edu.springboot;

import org.apache.tomcat.jdbc.pool.DataSource;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

import java.sql.Connection;
import java.sql.SQLException;


@SpringBootApplication
public class App {
    public static void main(String[] args) throws SQLException {
        ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
        DataSource dataSource = context.getBean(DataSource.class);
        Connection connection = dataSource.getConnection();
        System.out.println(connection.getCatalog());
        connection.close();
    }
}
```

## 11.2 使用JdbcTemplate

### 11.2.1 插入数据

```java
public void addProduct(String name){
    String sql = "insert into product (pname)values('"+name+"')";
    jdbcTemplate.execute(sql);
}
```

## 11.3 使用其他数据源

- 默认支持四种

```java
@Configuration
@Conditional(PooledDataSourceCondition.class)
@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
@Import({ DataSourceConfiguration.Tomcat.class, DataSourceConfiguration.Hikari.class,
        DataSourceConfiguration.Dbcp.class, DataSourceConfiguration.Dbcp2.class,
        DataSourceConfiguration.Generic.class })
protected static class PooledDataSourceConfiguration {
}
```

- springboot默认使用org.apache.tomcat.jdbc.pool.DataSource
- 在配置文件中添加spring.datasource.type=com.zaxxer.hikari.HikariDataSource
- 添加maven依赖

```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
</dependency>
```

### 11.3.1 引入默认不支持的数据源

- 引入maven依赖
- 编写配置类

```java
package com.clsaa.edu.springboot;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.core.env.Environment;

import javax.sql.DataSource;

/**
 * Created by Egg on 2017/2/26.
 */
@SpringBootConfiguration
public class DBConfiguration {

    @Autowired
    private Environment environment;
    public DataSource createDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUrl(environment.getProperty("spring.datasource.url"));
        druidDataSource.setUsername(environment.getProperty("spring.datasource.username"));
        druidDataSource.setPassword(environment.getProperty("spring.datasource.password"));
        druidDataSource.setDriverClassName(environment.getProperty("spring.datasource.driverClassName"));
        return druidDataSource;
    }

}

```

## 11.4 事务

- 使用@EnableTransactionManagement开启事务
- 使用@Transactional标记需要使用事务的方法
- spring默认只对runtimeException做回滚,当事务中直接抛出Exception时不会回滚事务
- 可以通过@Transactional(rollbackFor = Exception.class)回滚所有异常或不回滚哪些异常
- 要想使用事务生效,必须在直接调用的方法上有@Transactional注解,而间接调用的方法无所谓

```java
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;

@SpringBootApplication
@EnableTransactionManagement
public class App {
    public static void main(String[] args) throws Exception {
        ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
        context.getBean(ProductDao.class).addProductBatch("TV","MP4");
        System.out.println(context.getBean(DataSource.class));
    }
}

```

```
    @Transactional
    public void addProductBatch(String ... names) throws Exception {
        for (String name : names){
            String sql = "insert into product (pname)values('"+name+"')";
            jdbcTemplate.execute(sql);
            throw new RuntimeException();
        }
    }
```