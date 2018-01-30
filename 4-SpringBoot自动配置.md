---
layout:

title: 4-SpringBoot自动配置

date: 2017-02-04

updated: 2017-02-04

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



# 4-SpringBoot自动配置

[TOC]
## 4.1 根据条件的自动配置

- @conditional是基于条件的自动配置,一般配合Condition接口一起使用,只有接口实现类返回true,才装配,否则不装配.
- 用实现了Condition接口的类传入@Conditional中
- @Conditional可以标记在配置类的方法中,也可以标记在配置类上.标记的位置不同,作用域不同.
- @Conditional可以传入多个实现了condition接口的类,只有多个类对应的函数都返true了才进行装配.

```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class App {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
        System.out.println(context.getBeansOfType(EncodingConvert.class));

        context.close();
    }
}

```
```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringBootConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;


@SpringBootConfiguration
public class EncodingConvertConfiguration {
    @Bean
    @Conditional(UTF8Condition.class)
    public EncodingConvert createUTF8EncodingConvert(){
        return new UTF8EncodingConvert();
    }

    @Bean
    @Conditional(GBKCondition.class)
    public EncodingConvert createGBKEncodingConvert(){
        return new GBKEncodingConvert();
    }
}

```

```
package com.clsaa.edu.springboot;

/**
 * Created by Egg on 2017/2/22.
 */
public interface EncodingConvert {
}

```

```
package com.clsaa.edu.springboot;

/**
 * Created by Egg on 2017/2/22.
 */
public class GBKEncodingConvert implements EncodingConvert {
}

package com.clsaa.edu.springboot;

/**
 * Created by Egg on 2017/2/22.
 */
public class UTF8EncodingConvert implements EncodingConvert {
}


```
```
package com.clsaa.edu.springboot;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * Created by Egg on 2017/2/22.
 */
public class GBKCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String encoding = System.getProperty("file.encoding");
        if (encoding != null){
            return "gbk".equals(encoding.toLowerCase());
        }
        return false;
    }
}

package com.clsaa.edu.springboot;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * Created by Egg on 2017/2/22.
 */
public class UTF8Condition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String encoding = System.getProperty("file.encoding");
        if (encoding != null){
            return "utf-8".equals(encoding.toLowerCase());
        }
        return false;
    }
}

```

### 4.1.1 Spring提供的ConditionOnXXX

- spring框架还提供了很多@Condition给我们用，当然总结用语哪种好理解，看给位读者喽
    - @ConditionalOnBean（仅仅在当前上下文中存在某个对象时，才会实例化一个Bean）
    - @ConditionalOnClass（某个class位于类路径上，才会实例化一个Bean）
    - @ConditionalOnExpression（当表达式为true的时候，才会实例化一个Bean）
    - @ConditionalOnMissingBean（仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean）
    - @ConditionalOnMissingClass（某个class类路径上不存在的时候，才会实例化一个Bean）
    - @ConditionalOnNotWebApplication（不是web应用）
    - @ConditionalOnClass：该注解的参数对应的类必须存在，否则不解析该注解修饰的配置类；
    - @ConditionalOnMissingBean：该注解表示，如果存在它修饰的类的bean，则不需要再创建这个bean；可以给该注解传入参数例如@ConditionOnMissingBean(name = "example")，这个表示如果name为“example”的bean存在，这该注解修饰的代码块不执行。
```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;

/**
 * Created by Egg on 2017/2/22.
 */
@SpringBootConfiguration
public class UserConfiguration {

    @Bean
    @ConditionalOnProperty(name ="runnable.enabled",havingValue = "true",matchIfMissing=true)
    public Runnable createRunnable(){
        return ()->{};
    }
}

```

- matchIfMissing代表值不存在的时候函数返回什么(需不需要装载bean)