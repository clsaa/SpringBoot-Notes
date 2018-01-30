---
layout:

title: 3-SpringBoot快速入门

date: 2017-02-03

updated: 2017-02-03

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- 快速入门

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---



# 3-SpringBoot快速入门

[TOC]

## 3.1 SpringBoot Quik Start

- 配置pom
    - 继承一个父配置

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.clsaa.edu.spring</groupId>
    <artifactId>springboot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>springboot001</name>
    <url>http://maven.apache.org</url>

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
            <!--
                <version></version>
                由于我们在上面指定了 parent(spring boot)
             -->
        </dependency>
    </dependencies>

</project>
```

- 编写代码


```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;

/**
 * Created by Egg on 2017/2/21.
 */
@SpringBootApplication
public class App {
    @Bean
    public Runnable createRunnable(){
        return ()->{System.out.println("spring boot is running");};
    }
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
        context.getBean(Runnable.class).run();
    }

}

```

### 3.1.1 SpringBootApplication源码解析

```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.context.TypeExcludeFilter;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    Class<?>[] exclude() default {};

    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}

```

```
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.Configuration;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}

```

- 可以看到SpringBootApplication是一个注解的集合,其中包括
    - @Target({ElementType.TYPE})
    - @Retention(RetentionPolicy.RUNTIME)
    - @Documented
    - @Inherited
    - @SpringBootConfiguration:是继承自configuration(和configuration用法相同),表明当前类是一个配置类,因此可以在这个类中发布bean
        - @Target({ElementType.TYPE})
        - @Retention(RetentionPolicy.RUNTIME)
        - @Documented
        - @Configuration
    - @EnableAutoConfiguration:自动配置
    - @ComponentScan:自动扫描(默认从当前包开始)

### 3.1.2 run方法源码解析

```
    public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
        return (new SpringApplication(sources)).run(args);
    }
```

- 被传入的类会被设置为配置类
- 一般情况下第二个参数通常为Java主函数参数
- 还可以使用以下方式创建context
```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.Bean;

/**
 * Created by Egg on 2017/2/21.
 */
@SpringBootApplication
public class App {
    @Bean
    public Runnable createRunnable(){
        return ()->{System.out.println("spring boot is running");};
    }
    public static void main(String[] args) {
        //ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
        SpringApplication app = new SpringApplication();
        ConfigurableApplicationContext context = app.run(args);
        context.getBean(Runnable.class).run();
    }

}
```

## 3.2 指定多个源

### 3.2.1 通过set后期设置

```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

import java.util.HashSet;
import java.util.Set;

/**
 * Created by Egg on 2017/2/21.
 */
@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication();
        Set<Object> sets = new HashSet<>();
        sets.add(App2.class);
        app.setSources(sets);
        ConfigurableApplicationContext context = app.run(args);
        context.getBean(Runnable.class).run();
    }

}

```

```
package com.clsaa.springboot;

import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/21.
 */
@Component
public class App2 {
    @Bean
    public Runnable createRunnable(){
        return ()->{System.out.println("spring boot is running");};
    }
}

```

### 3.2.2 在构造函数中设置

```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;



/**
 * Created by Egg on 2017/2/21.
 */
@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App2.class);
        ConfigurableApplicationContext context = app.run(args);
        context.getBean(Runnable.class).run();
    }

}

```

## 3.3 Spring Boot 配置分析

- springBoot默认配置文件的名字application.properties,在D:\Data\MyCode\codeMaven\learn_springboot001\src\main\resources下创建application.properties文件
- 默认的位置为classpath根目录或者是classpath/config目录下
### 3.3.1 获取配置

- 在application.properties中添加属性local.ip=127.0.0.1

```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;



/**
 * Created by Egg on 2017/2/21.
 */
@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        ConfigurableApplicationContext context = app.run(args);
        System.out.println(context.getEnvironment().getProperty("local.ip"));
        context.getBean(Runnable.class).run();
    }
}

```

### 3.3.2. 注入Environment获取配置

- 方法1

```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;



/**
 * Created by Egg on 2017/2/21.
 */
@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        ConfigurableApplicationContext context = app.run(args);
        context.getBean(UserConfig.class).show();
    }
}


```

```
package com.clsaa.springboot;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/21.
 */
@Component
public class UserConfig {
    @Autowired
    private Environment env ;

    public void show(){
        System.out.println("local.ip"+env.getProperty("local.ip"));
    }
}

```

- 方法2

```
@Component
public class UserConfig {

    @Value("${local.port}")
    private String localPort;

    @Autowired
    private Environment env ;

    public void show(){
        System.out.println("local.ip:port"+env.getProperty("local.ip")+":"+localPort);
    }
}

```

- 可以进行自动类型转换

```
package com.clsaa.springboot;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Component
public class UserConfig {

    @Value("${local.port}")
    private String localPort;

    @Value("${local.port}")
    private Integer localPort2;


    @Autowired
    private Environment env ;

    public void show(){
        System.out.println("local.ip:port"+env.getProperty("local.ip")+":"+localPort2);
    }
}
```

### 3.3.3 使用properties文件的引用

```
name=springboot
app.name=${name}
```

```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        ConfigurableApplicationContext context = app.run(args);
        System.out.println(context.getEnvironment().getProperty("app.name"));
        System.out.println(context.getEnvironment().getProperty("name"));
    }
}


```

### 3.3.4 获取配置默认值

```
    /**
     * value默认必须有配置项
     * 配置项可以为空
     * 如果没有配置项则可以加默认值
     */
    @Value("${tomcat.port:9090}")
    private Integer tomcat_port;

```
- 或使用下面重载的方法
```
    System.out.println(context.getEnvironment().getProperty("app.name","12"));
```

### 3.3.5 修改默认配置文件名字和目录

- 若改文件名为app 配置program arguments :--spring.config.name=app
- 若改为classpath下conf下的app.properties 则配置program arguments :--spring.config.location=classpath:conf/app.properties
- 若指定多个:则配置program arguments :--spring.config.location=classpath:conf/app.properties,file:/temp/xxx.properties

### 3.3.6 添加新的properties

```
package com.clsaa.springboot;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

/**
 * Created by Egg on 2017/2/21.
 */
@Configuration
@PropertySource("classpath:jdbc.properties")
@PropertySource("file:/d:/test.properties")
//@PropertySources({@PropertySource("classpath:jdbc.properties"),@PropertySource("file:/d:/test.properties")})

public class FileConfig {

}

```

```
package com.clsaa.springboot;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/21.
 */
@Component
public class JDBCConfig {
    @Value("${url}")
    private String url;

    public void show(){
        System.out.println("url : " + url);
    }
}

```

```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        ConfigurableApplicationContext context = app.run(args);
        context.getBean(JDBCConfig.class).show();
    }
}
```

### 3.3.7 配置文件注入集合

```
ds.hosts[0]=192.168.154.112
ds.hosts[1]=192.168.154.112
ds.hosts[2]=192.168.154.112
```

```
package com.clsaa.springboot;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

/**
 * Created by Egg on 2017/2/21.
 */
@Component
@ConfigurationProperties("ds")
public class TomcatProperties {
    private List<String> hosts = new ArrayList<>();

    public List<String> getHosts() {
        return hosts;
    }

    public void setHosts(List<String> hosts) {
        this.hosts = hosts;
    }

    @Override
    public String toString() {
        return "TomcatProperties{" +
                "hosts=" + hosts +
                '}';
    }
}

```

## 3.4 动态引入配置

- 创建文件D:\Data\MyCode\codeMaven\learn_springboot001\src\main\resources\META-INF\spring.factories 内容:org.springframework.boot.env.EnvironmentPostProcessor=com.clsaa.springboot.MyEnvironmentPostProcessor

```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        ConfigurableApplicationContext context = app.run(args);
        System.out.println(context.getEnvironment().getProperty("springboot.name"));
        context.close();
    }
}


```
```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.env.EnvironmentPostProcessor;
import org.springframework.core.env.ConfigurableEnvironment;
import org.springframework.core.env.PropertiesPropertySource;
import org.springframework.stereotype.Component;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;


@Component
public class MyEnvironmentPostProcessor implements EnvironmentPostProcessor{
    @Override
    public void postProcessEnvironment(ConfigurableEnvironment configurableEnvironment, SpringApplication springApplication) {
        try (InputStream inputStream = new FileInputStream("D:/test.properties")){
            Properties source = new Properties();
            source.load(inputStream);
            System.out.println("=============="+source.getProperty("springboot.name"));
            PropertiesPropertySource propertySource = new PropertiesPropertySource("my",source);
            configurableEnvironment.getPropertySources().addLast(propertySource);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}

```

## 3.5 在不同生产环境中引入不同配置文件

### 3.5.1 使用代码方式
- 文件名以application-xxx.properties格式,且默认配置文件会加入其中
- 创建配置文件
    - D:\Data\MyCode\codeMaven\learn_springboot001\src\main\resources\application.properties
    - D:\Data\MyCode\codeMaven\learn_springboot001\src\main\resources\application-dev.properties
    - D:\Data\MyCode\codeMaven\learn_springboot001\src\main\resources\application-test.properties
    - D:\Data\MyCode\codeMaven\learn_springboot001\src\main\resources\application-x1.properties
    - D:\Data\MyCode\codeMaven\learn_springboot001\src\main\resources\application_test.properties
```
package com.clsaa.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        app.setAdditionalProfiles("x1");
        ConfigurableApplicationContext context = app.run(args);
        System.out.println(context.getEnvironment().getProperty("name"));
        context.close();
    }
}


```

### 3.5.2 使用启动参数的方式

- --spring.profile.active=test,dev(可以同时激活多个properties)

- 在加载某些配置文件后加载某些bean
```
package com.clsaa.springboot;

import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Profile;

/**
 * Created by Egg on 2017/2/21.
 */
@SpringBootConfiguration
public class MyConfig {

    @Bean
    public Runnable createRunnable(){
        System.out.println("createRunnable");
        return ()->{};
    }


    @Bean
    @Profile("dev")
    public Runnable createRunnable_dev(){
        System.out.println("createRunnable_dev");
        return ()->{};
    }

    @Bean
    @Profile("test")
    public Runnable createRunnable_test(){
        System.out.println("createRunnable_test");
        return ()->{};
    }



}

```

