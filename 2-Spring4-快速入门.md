---
layout:

title: 2-Spring4-快速入门

date: 2017-02-02

updated: 2017-02-02

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


# 2-Spring4-快速入门

[TOC]

## 2.1 Spring4-对象装配

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.clsaa.edu.spring</groupId>
    <artifactId>spring</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.6.RELEASE</version>
        </dependency>


    </dependencies>

</project>
```
### 2.1.1 基本对象装配

- 容器中只有一个对象时(单例)才能使用XXX.class的形式获取对象.

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by Egg on 2017/2/19.
 */
public class App {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        System.out.println(context.getBean(MyBean.class));
        System.out.println(context.getBean("MyBean"));
    }
}

```

```
package com.clsaa.edu.spring;

/**
 * Created by Egg on 2017/2/19.
 */
public class MyBean {

}

```


```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

/**
 * Created by Egg on 2017/2/19.
 */

/**
 * 配置类
 */
@Configuration
public class MyConfig {
    @Bean(name="MyBean")
    @Scope("prototype")
    public MyBean createMyBean(){
        return new MyBean();
    }


}

```

### 2.1.2 使用FactoryBean进行对象装配

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by Egg on 2017/2/19.
 */
public class App {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        System.out.println(context.getBean(MyBean.class));
        System.out.println(context.getBean("MyBean"));
        System.out.println(context.getBean(Jeep.class));
        System.out.println(context.getBean("createRunnableFactoryBean"));
        System.out.println(context.getBean(JeepFactoryBean.class));
        System.out.println(context.getBean("&createRunnableFactoryBean"));

    }
}

```

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

/**
 * Created by Egg on 2017/2/19.
 */

/**
 * 配置类
 */
@Configuration
public class MyConfig {
    @Bean(name="MyBean")
    @Scope("prototype")
    public MyBean createMyBean(){
        return new MyBean();
    }

    @Bean
    public JeepFactoryBean createRunnableFactoryBean(){
        return new JeepFactoryBean();
    }


}
```

```
package com.clsaa.edu.spring;

/**
 * Created by Egg on 2017/2/19.
 */
public class Jeep {
}

```

```
package com.clsaa.edu.spring;

import org.springframework.beans.factory.FactoryBean;

/**
 * Created by Egg on 2017/2/19.
 */
public class JeepFactoryBean implements FactoryBean<Jeep> {


    @Override
    public Jeep getObject() throws Exception {
        return new Jeep();
    }

    @Override
    public Class<?> getObjectType() {
        return Jeep.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}

```

### 2.1.3 使用普通的Factory

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

/**
 * Created by Egg on 2017/2/19.
 */

/**
 * 配置类
 */
@Configuration
public class MyConfig {
    @Bean(name="MyBean")
    @Scope("prototype")
    public MyBean createMyBean(){
        return new MyBean();
    }

    @Bean
    public JeepFactoryBean createRunnableFactoryBean(){
        return new JeepFactoryBean();
    }


    @Bean
    public CarFactory CarFactory(){
        return new CarFactory();
    }

    @Bean Car createCar(CarFactory carFactory){
        return carFactory.create();
    }


}

```

```
package com.clsaa.edu.spring;

/**
 * Created by Egg on 2017/2/19.
 */
public class CarFactory {
    public Car create(){
        return new Car();
    }

}

```

```
package com.clsaa.edu.spring;

/**
 * Created by Egg on 2017/2/19.
 */
public class Car {
}

```

### 2.1.4 对象初始化前后的操作-1

```
package com.clsaa.edu.spring;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

/**
 * Created by Egg on 2017/2/19.
 */
public class Cat implements InitializingBean,DisposableBean{


    /**
     * 属性设置之后执行
     * @throws Exception
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("afterPropertiesSet");
    }

    /**
     * 销毁之后执行
     * @throws Exception
     */
    @Override
    public void destroy() throws Exception {
        System.out.println("destroy");
    }
}

```

### 2.1.5 对象初始化前后的操作-2

```
    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Dog createDog(){
        return new Dog();
    }
```

```
package com.clsaa.edu.spring;

/**
 * Created by Egg on 2017/2/19.
 */
public class Dog {
    public void init(){
        System.out.println("init");
    }

    public void destroy(){
        System.out.println("destroy");
    }
}

```

### 2.1.6 ### 2.1.5 对象初始化前后的操作-3

```
package com.clsaa.edu.spring;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

/**
 * Created by Egg on 2017/2/19.
 */
public class Animal {

    @PostConstruct
    public void init(){
        System.out.println("init");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("destroy");
    }
}

```

### 2.1.7 注册组件对象

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by Egg on 2017/2/19.
 */
public class App {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class,User.class,UserDao.class,UserService.class,UserController.class);
        System.out.println(context.getBean(MyBean.class));
        System.out.println(context.getBean("MyBean"));
        System.out.println(context.getBean(Jeep.class));
        System.out.println(context.getBean("createRunnableFactoryBean"));
        System.out.println(context.getBean(JeepFactoryBean.class));
        System.out.println(context.getBean("&createRunnableFactoryBean"));
        System.out.println(context.getBean(Car.class));
        System.out.println(context.getBean(Cat.class));
        System.out.println(context.getBean(User.class));
        context.close();

    }
}


```

```
package com.clsaa.edu.spring;

import org.springframework.stereotype.Controller;

/**
 * Created by Egg on 2017/2/19.
 */
@Controller
public class UserController {
}


```

```
package com.clsaa.edu.spring;

import org.springframework.stereotype.Service;

/**
 * Created by Egg on 2017/2/19.
 */
@Service
public class UserService {
}

```

```
package com.clsaa.edu.spring;

import org.springframework.stereotype.Repository;

/**
 * Created by Egg on 2017/2/19.
 */
@Repository
public class UserDao {
}

```

## 2.2 Spring4-依赖注入

```
    User user = context.getBean("MyUser",User.class);
    user.show();

```

```
package com.clsaa.edu.spring;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/19.
 */

@Component("MyUser")
public class User {

    //JSR250 @Resource 
    
    @Autowired
    UserDao userDao ;

    public void show(){
        System.out.println("autowired"+userDao);
    }
}

```

- spring4.3新特性:当对象构造函数中有参数时,spring会自动初始化对应的参数.
    - 构造函数只能有一个,如果有多个的话就必须有一个无参的构造函数,此时spring会调用无参的构造函数
    


### 2.2.1 指定装配某一个对象

- 方法1-接收方设置

```
@Bean
public UserDao createUserDao(){
    return new UserDao();
}
```

```
@Component("MyUser")
public class User {

    @Autowired
    @Qualifier("createUserDao")
    private UserDao userDao ;

    public void show(){
        System.out.println("autowired"+userDao);
    }
}

```

- 方法2-提供方设置

```
    @Bean
    @Primary
    public UserDao createUserDao(){
        return new UserDao();
    }
```

## 2.3 使用自动扫描的方式进行对象装载

### 2.3.1 自动扫描方法1

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by Egg on 2017/2/19.
 */
public class AnnotationClient {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("com.clsaa.edu.spring");
        System.out.println(context.getBean(MyBean.class));
        System.out.println(context.getBean("MyBean"));
        context.close();
    }
}

```

### 2.3.2 自动扫描方法2

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by Egg on 2017/2/19.
 */

@ComponentScan("com.clsaa.edu.spring")
@Configuration
public class AnnotationScan {
}

```

### 2.3.3 自动扫描的过滤

- 配置类方式

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

/**
 * Created by Egg on 2017/2/19.
 */

@ComponentScan(basePackages = "com.clsaa.edu.spring",excludeFilters = @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes = DogConfig.class))
@Configuration
public class AnnotationScan {
}

```

```
package com.clsaa.edu.spring;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created by Egg on 2017/2/19.
 */
@Configuration
public class DogConfig {
    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Dog createDog(){
        return new Dog();
    }
}

```

## 2.4 每个bean初始化的时候调用

```
package com.clsaa.edu.spring;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/20.
 * BeanPostProcessor会在每个bean初始化的时候,调用一次
 */
@Component
public class EchoBeanPostProcessor implements BeanPostProcessor {
    /**
     * 在bean依赖装配完成之后触发(属性设置完之后触发)
     * @param o
     * @param s
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        System.out.println("----postProcessBeforeInitialization----"+o.getClass());
        return o;
    }

    /**
     * 执行玩init方法完成之后触发
     * @param o
     * @param s
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("----postProcessAfterInitialization----"+o.getClass());
        return o;
    }
}


```
- 如果返回null,spring容器会销毁对应对象.
- 上述代码输出如下

```
----postProcessBeforeInitialization----class com.clsaa.edu.spring.User
user init
----postProcessAfterInitialization----class com.clsaa.edu.spring.User
```

- 上述方法可用于对指定的bean做一些处理,比如返回代理模式

## 2.5 容器初始化的时候调用

- 在spring容器初始化之后调用,且只会调用一次.

```
package com.clsaa.edu.spring;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/20.
 */

@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println("==================="+ configurableListableBeanFactory.getBeanDefinitionCount());
    }
}

```

### 2.5.1 在容器中动态注册bean



```
package com.clsaa.edu.spring;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/20.
 */
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry beanDefinitionRegistry) throws BeansException {
        for (int i = 0 ; i < 10 ; i++){
            BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.rootBeanDefinition(Person.class);
            beanDefinitionBuilder.addPropertyValue("name","admin"+i);
            beanDefinitionRegistry.registerBeanDefinition("person"+i,beanDefinitionBuilder.getBeanDefinition());
        }
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {

    }
}

```