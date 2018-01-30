---
layout:

title: 17-SpringBoot构建微服务实战

date: 2017-02-17

updated: 2017-02-17

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---


# 17-SpringBoot构建微服务实战

[TOC]

## 17.1 编写服务启动入口

```
package com.clsaa.edu.springboot;

import com.clsaa.edu.springboot.bean.Product;
import com.clsaa.edu.springboot.mapper.ProductMapper;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

/**
 * Created by Egg on 2017/2/28.
 */
@SpringBootApplication
public class App {
    public static void main(String[] args) {

        ConfigurableApplicationContext context = SpringApplication.run(App.class,args);

    }
}

```

## 17.2 服务配置

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.clsaa.edu.spring</groupId>
    <artifactId>springboot_ms_1</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.1.RELEASE</version>
    </parent>

    <parent>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-samples</artifactId>
        <version>1.2.0</version>
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.2</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
</project>
```

```
spring.datasource.url=jdbc:mysql:///db_products
spring.datasource.username=root
spring.datasource.password=xiaojie1996
spring.datasource.driver=com.mysql.jdbc.Driver
```

```
CREATE database db_products DEFAULT charset utf-8;
CREATE TABLE  products(pid INT NOT NULL PRIMARY KEY auto_increment,
pname VARCHAR(200) , type VARCHAR(50 , price DOUBLE ,createTime TIMESTAMP ) )
```

## 17.3 controller

```
package com.clsaa.edu.springboot.controller;

import com.clsaa.edu.springboot.bean.Product;
import com.clsaa.edu.springboot.mapper.ProductMapper;
import com.clsaa.edu.springboot.web.Response;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * Created by Egg on 2017/2/28.
 */
@RestController
public class ProductController {

    @Autowired
    private ProductMapper productMapper;
    @PostMapping("/soa/product/add")
    public Response add(Product product){
        Integer res = productMapper.addProduct(product);
        if (res == 1){
            return new Response("200","ok");
        }
        return new Response("500","Fail");
    }

    @GetMapping("/soa/product/{id}")
    public Object get(@PathVariable("id") Integer id){
        Product product = productMapper.getById(id);
        return new Response("200","ok",product);
    }

    @GetMapping("/soa/products")
    public Object list(){
        return new Response("200","ok",productMapper.getByList());
    }

    @DeleteMapping("/soa/product/{id}")
    public Object delete(@PathVariable("id") Integer id){
        Integer res = productMapper.deleteById(id);
        return res==1?new Response("200","OK"):new Response("500","fail");
    }

    @PutMapping("/soa/product/update")
    public Object update(Product product){
        Integer res = productMapper.update(product);
        return res==1?new Response("200","OK"):new Response("500","fail");
    }
}

```

## 17.4 数据库

```
package com.clsaa.edu.springboot.bean;

import java.sql.Timestamp;

/**
 * Created by Egg on 2017/2/28.
 */
public class Product {
    private Integer pid;
    private String pname;
    private String type;
    private Double price;
    private Timestamp timestamp;

    public Integer getPid() {
        return pid;
    }

    public void setPid(Integer pid) {
        this.pid = pid;
    }

    public String getPname() {
        return pname;
    }

    public void setPname(String pname) {
        this.pname = pname;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    public Timestamp getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Timestamp timestamp) {
        this.timestamp = timestamp;
    }

    @Override
    public String toString() {
        return "Product{" +
                "pid=" + pid +
                ", pname``='" + pname + '\'' +
                ", type='" + type + '\'' +
                ", price=" + price +
                ", timestamp=" + timestamp +
                '}';
    }
}

```

```
package com.clsaa.edu.springboot.mapper;

import com.clsaa.edu.springboot.bean.Product;
import org.apache.ibatis.annotations.*;

import java.util.List;

/**
 * Created by Egg on 2017/2/28.
 */
@Mapper
public interface ProductMapper {

    @Insert("insert into products (pname,type,price)values(#{pname},#{type},#{price})")
    public Integer addProduct(Product product);

    @Delete("delete from products where pid=#{arg1}")
    public Integer deleteById(Integer id);

    @Update("update products set pname=#{pname},type=#{type},price=#{price} where pid=${arg1}")
    public Integer update(Product product);

    @Select("select * from products where pid=#{arg1}")
    public Product getById(Integer id);

    @Select("select * from products order by pid desc")
    public List<Product> getByList();

}

```