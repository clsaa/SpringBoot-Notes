---
layout:

title: 13-SpringBoot开发一个starter-集成redis

date: 2017-02-13

updated: 22017-02-13

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- Redis

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---



# 13-SpringBoot集成redis

[TOC]

1. 新建一个项目
2. 新建一个配置类,需要一个配置类,配置类里面需要装配好提供出去的类
3. 使用EnableXXX注解或者spring.factory配置,将提供的类加入spring容器的管理

```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import redis.clients.jedis.Jedis;

        @SpringBootApplication
        public class App {
            public static void main(String[] args) throws Exception {
                ConfigurableApplicationContext context = SpringApplication.run(App.class,args);
                Jedis jedis = context.getBean(Jedis.class);
                jedis.set("cd","51CTO");
        System.out.println(jedis.get("id"));
    }
}

```
```
package com.clsaa.edu.springboot;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import redis.clients.jedis.Jedis;

/**
 * Created by Egg on 2017/2/27.
 */
@Configuration
@ConditionalOnClass(Jedis.class)//在存在此类时才进行装配
@EnableConfigurationProperties(RedisProperties.class)
public class RedisAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean//没有被装配过才进行装配
    public Jedis jedis(RedisProperties redisProperties){
        return new Jedis(redisProperties.getHost(),redisProperties.getPort());
    }
}

```
```
package com.clsaa.edu.springboot;

import org.springframework.boot.context.properties.ConfigurationProperties;


@ConfigurationProperties(prefix = "redis")
public class RedisProperties {
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
}

```

- 若使用自己的配置文件需要配置

```
redis.host=123.206.175.47
redis.port=6379
```