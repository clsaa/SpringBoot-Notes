---
layout:

title: 5-Enable注解工作原理

date: 2017-02-05

updated: 2017-02-05

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- 原理

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---


# 5-Enable注解工作原理

[TOC]

## 5.1 Enable注解用法

- 把配置文件里的属性注入到bean中
```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ComponentScan;

/**
 * EnableConfigurationProperties是用来启用一个特性的
 * 这个特性可以把配置文件的属性注入到bean里面去
 */
@EnableConfigurationProperties
@ComponentScan
public class App {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
        System.out.println(context.getBeansOfType(TomcatProperties.class));
        context.close();
    }
}
```
```
package com.clsaa.edu.springboot;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/23.
 */
@Component
@ConfigurationProperties(prefix = "tomcat")
public class TomcatProperties {
    private String host;
    private Integer port;

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public Integer getPort() {
        return port;
    }

    public void setPort(Integer port) {
        this.port = port;
    }

    @Override
    public String toString() {
        return "TomcatProperties{" +
                "host='" + host + '\'' +
                ", port=" + port +
                '}';
    }
}

```
- 使用spring启用一个异步
    - @EnableAsync启用异步
    - @Async标记异步
```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.scheduling.annotation.EnableAsync;


@EnableConfigurationProperties
@EnableAsync
@ComponentScan
public class App {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
        System.out.println("====start====");
        context.getBean(Jeep.class).run();
        System.out.println("====end====");
        context.close();
    }
}

```

```
package com.clsaa.edu.springboot;

import ch.qos.logback.core.util.TimeUtil;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

import java.util.concurrent.TimeUnit;

/**
 * Created by Egg on 2017/2/23.
 */
@Component
public class Jeep implements Runnable {

    @Override
    @Async
    public void run() {
        try {
            for (int i = 0 ; i < 10 ; i++) {
            System.out.println("================="+i);
                TimeUnit.SECONDS.sleep(1);
            }
        }catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

## 5.2 Enable注解源码分析

```
/**
 * Enable support for {@link ConfigurationProperties} annotated beans.
 * {@link ConfigurationProperties} beans can be registered in the standard way (for
 * example using {@link Bean @Bean} methods) or, for convenience, can be specified
 * directly on this annotation.
 *
 * @author Dave Syer
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EnableConfigurationPropertiesImportSelector.class)
public @interface EnableConfigurationProperties {

	/**
	 * Convenient way to quickly register {@link ConfigurationProperties} annotated beans
	 * with Spring. Standard Spring Beans will also be scanned regardless of this value.
	 * @return {@link ConfigurationProperties} annotated beans to register
	 */
	Class<?>[] value() default {};

}
```

- Enable注解的核心在于@Import

```
package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Indicates one or more {@link Configuration @Configuration} classes to import.
 *
 * <p>Provides functionality equivalent to the {@code <import/>} element in Spring XML.
 * Allows for importing {@code @Configuration} classes, {@link ImportSelector} and
 * {@link ImportBeanDefinitionRegistrar} implementations, as well as regular component
 * classes (as of 4.2; analogous to {@link AnnotationConfigApplicationContext#register}).
 *
 * <p>{@code @Bean} definitions declared in imported {@code @Configuration} classes should be
 * accessed by using {@link org.springframework.beans.factory.annotation.Autowired @Autowired}
 * injection. Either the bean itself can be autowired, or the configuration class instance
 * declaring the bean can be autowired. The latter approach allows for explicit, IDE-friendly
 * navigation between {@code @Configuration} class methods.
 *
 * <p>May be declared at the class level or as a meta-annotation.
 *
 * <p>If XML or other non-{@code @Configuration} bean definition resources need to be
 * imported, use the {@link ImportResource @ImportResource} annotation instead.
 *
 * @author Chris Beams
 * @author Juergen Hoeller
 * @since 3.0
 * @see Configuration
 * @see ImportSelector
 * @see ImportResource
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();

}

```

- @Import的用法
    - 用来导入一个或者多个类(会被spring容器托管)
    - 或者导入配置类(配置类里的bean都会被spring容器托管)

```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Import;

/**
 * Created by Egg on 2017/2/23.
 */
@ComponentScan
@Import({User.class,Role.class,MyConfiguration.class})
public class Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Application.class,args);
        System.out.println(context.getBean(User.class));
        System.out.println(context.getBean(Role.class));
        System.out.println(context.getBean(Runnable.class));
        context.close();
    }
}

```
```
package com.clsaa.edu.springboot;

import org.springframework.context.annotation.Bean;

/**
 * Created by Egg on 2017/2/23.
 */
public class MyConfiguration {
    @Bean
    public Runnable createRunnable(){
        return ()->{};
    }

}
package com.clsaa.edu.springboot;

/**
 * Created by Egg on 2017/2/23.
 */
public class User {
}

```

- @Import的核心在于ImportSelector
    - 能根据业务逻辑(配置的注解)动态的返回一个字符串数组
```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Import;


@ComponentScan
//@Import({User.class,Role.class,MyConfiguration.class})
@Import(MyImportSelector.class)
public class Application {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(Application.class,args);
        System.out.println(context.getBean(User.class));
        System.out.println(context.getBean(Role.class));
        context.close();
    }
}

```

```
package com.clsaa.edu.springboot;

import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;

public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.clsaa.edu.springboot.User","com.clsaa.edu.springboot.MyConfiguration","com.clsaa.edu.springboot.Role"};
    }
}

```

- 另外还要注意ImportBeanDefinitionRegistrar这个接口
    - 与之前的类似我们可以用这个接口提供的函数中的registry参数往容器中注入bean
```
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}

```