---
layout:

title: 7-SpringBoot事件监听

date: 2017-02-07

updated: 2017-02-07

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


# 7-SpringBoot事件监听


[TOC]

- 当一个bean处理完后需要另一个bean继续处理,那么就需要一个bean监听另一个bean

## 7.1 事件流程

1. 自定义事件:一般是继承ApplicationEvent抽象类
2. 定义事件监听器:一般是实现ApplicationListener接口
3. 启动的时候把监听器加入到spring容器中
4. 发布事件

```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ComponentScan;

@EnableAutoConfiguration
@ComponentScan
public class App {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(App.class);
        app.addListeners(new MyApplicationLisener());
        ConfigurableApplicationContext context = app.run(args);
        context.publishEvent(new MyApplicationEvent(new Object()));
        context.close();
    }
}

```
```
package com.clsaa.edu.springboot;

import org.springframework.context.ApplicationEvent;

/**
 * Created by Egg on 2017/2/24.
 * 定义事件
 */
public class MyApplicationEvent extends ApplicationEvent {
    /**
     * Create a new ApplicationEvent.
     *
     * @param source the object on which the event initially occurred (never {@code null})
     */
    public MyApplicationEvent(Object source) {
        super(source);
    }
}

```
```
package com.clsaa.edu.springboot;

import org.springframework.context.ApplicationListener;

/**
 * Created by Egg on 2017/2/24.
 * 定义事件监听器
 */
public class MyApplicationLisener implements ApplicationListener<MyApplicationEvent> {
    @Override
    public void onApplicationEvent(MyApplicationEvent event) {
        System.out.println("================"+event.getClass());
    }
}

```

- 也可以在监听器上加component注解,那样就不用app.addListeners(new MyApplicationLisener());
- 还可以在springboot配置文件中配置context.listener.classes=com.clsaa.edu.springboot.MyApplicationLisener

```
package com.clsaa.edu.springboot;

import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/24.
 */
@Component
public class MyEventHandler {

    @EventListener
    public void event(MyApplicationEvent event){
        System.out.println("=========="+event.getClass());
    }
}

```
- 所有该参数事件,或其子事件都可以接收到