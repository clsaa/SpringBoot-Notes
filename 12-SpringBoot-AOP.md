---
layout:

title: 12-SpringBoot-AOP

date: 2017-02-12

updated: 2017-02-12

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- AOP

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---


# 12-SpringBoot-AOP

[TOC]

- 日志
- 权限处理
- 异常处理
- 监控
- 性能分析

## 12.1 使用AOP

- 添加maven依赖

```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
```
- 编写切面代码
    - 确定通知(前置后置)
    - 确定切入点

```
package com.clsaa.edu.springboot;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/26.
 */
@Aspect
@Component
public class LogAspect {

    @Before("execution(* com.clsaa.edu.springboot.dao..*.*(..))")
    public void log(){
        System.out.println("method log done");
    }
}

```

## 12.2 AOP配置

- spring.aop.auto配置决定是否启用AOP
- 默认使用基于JDK的动态代理实现AOP
- spring.aop.proxy-target-class-false或者不配置表示使用JDK的动态代理=true表示使用cglib
- 如果配置了false,而类没有接口,则依赖使用CGLIB

## 12.3 在AOP中获取参数

```
package com.clsaa.edu.springboot;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

import java.util.Arrays;

/**
 * Created by Egg on 2017/2/26.
 */
@Aspect
@Component
public class LogAspect {

    @Before("execution(* com.clsaa.edu.springboot.dao..*.*(..))")
    public void log(){
        System.out.println("before method log done");
    }
    @After("execution(* com.clsaa.edu.springboot.dao..*.*(..))")
    public void logAfter(JoinPoint point){
        System.out.println("after method log done " + point.getTarget().getClass() + point.getSignature().getName() +Arrays.toString(point.getArgs()));
    }
}

```

- 在App上添加EnableAspectJautoProxy(exposeProxy=true),让我们可以在切面中获得代理对象使用AOPContext.currentProxy