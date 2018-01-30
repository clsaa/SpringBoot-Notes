---
layout:

title: 14-SpringBoot日志

date: 2017-02-14

updated: 2017-02-14

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- 日志

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---


# 14-SpringBoot日志

[TOC]

- SpringBoot默认有日志输出

- 2017-02-27 16:04:06.644  INFO 15384 --- [           main] com.clsaa.edu.springboot.App             : Starting App on eggyer with PID 15384 (D:\Data\MyCode\codeMaven\learn_springboot004\target\classes started by Egg in D:\Data\MyCode\codeMaven\learn_springboot004)

## 14.1 使用日志

```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class App {
    public static void main(String[] args) throws Exception {
        ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
        context.getBean(UserDao.class).log();
        context.close();
    }
}

```

```
package com.clsaa.edu.springboot;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/27.
 */
@Component
public class UserDao {

    private Logger logger = LoggerFactory.getLogger(UserDao.class);
    public void log(){
        logger.debug("user dao debug log");
        logger.info("user dao info log");
        logger.warn("user dao warn log");
        logger.error("user dao error log");
    }
}

```


```
2017-02-27 16:13:14.429  INFO 15168 --- [           main] com.clsaa.edu.springboot.App             : Started App in 14.288 seconds (JVM running for 15.169)
2017-02-27 16:13:14.430  INFO 15168 --- [           main] com.clsaa.edu.springboot.UserDao         : user dao info log
2017-02-27 16:13:14.430  WARN 15168 --- [           main] com.clsaa.edu.springboot.UserDao         : user dao warn log
2017-02-27 16:13:14.430 ERROR 15168 --- [           main] com.clsaa.edu.springboot.UserDao         : user dao error log

```

## 14.2 日志配置

### 14.2.1 日志输出级别配置

- 在application.properties添加logging.level.*=debug
    - *可以是一个包名也可以是一个类名
    - 如果填为root则为所有
- 日志级别分别有:TRACE,DEBUG,INFO,WARN,ERROR,FATAL,OFF(关闭日志输出).

### 14.2.2 文件输出配置

- 指定日志文件在application.properties添加logging.file=d:/temp/my.log
- 指定日志文件所在目录ogging.path=d:/temp/logs
- 日志文件大小超过10M就会分割

### 14.2.3 指定输出格式

- logging.pattern.console=
- logging.pattern.file=

### 14.2.4 使用logback定制化日志

- 在classpath下添加logback.xml

## 14.3 更换为log4j2

- 添加配置文件log4j2.xml
- 修改pom文件,删除默认的logging配置
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>clsaa</groupId>
    <artifactId>springboot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--spring boot 父节点依赖,引入这个之后相关的引入就不需要添加version配置，spring boot会自动选择最合适的版本进行添加。-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>

    </dependencies>

</project>
```