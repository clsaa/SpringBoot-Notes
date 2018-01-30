---
layout:

title: 15-SpringBoot监控和度量

date: 2017-02-15

updated: 2017-02-15

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- 监控和度量

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---

# 15-SpringBoot监控和度量


[TOC]

```
- Mapped "{[/env/{name:.*}],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EnvironmentMvcEndpoint.value(java.lang
- Mapped "{[/env || /env.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke():监控环境
- Mapped "{[/beans || /beans.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke():监控bean
- Mapped "{[/trace || /trace.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
- Mapped "{[/info || /info.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
- Mapped "{[/mappings || /mappings.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke
- Mapped "{[/configprops || /configprops.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.:监控url映射
- Mapped "{[/dump || /dump.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
- Mapped "{[/autoconfig || /autoconfig.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.in
- Mapped "{[/health || /health.json],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.HealthMvcEndpoint.invoke(java.security.Prin:健康检查
- Mapped "{[/heapdump || /heapdump.json],methods=[GET],produces=[application/octet-stream]}" onto public void org.springframework.boot.actuate.endpoint.mvc.HeapdumpMvcEndpoint.invoke(bo
- Mapped "{[/metrics/{name:.*}],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.MetricsMvcEndpoint.value(java.lang
- Mapped "{[/metrics || /metrics.json],methods=[GET],produces=[application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke():度量
```

## 15.1 设计自己的状态检查

```
package com.clsaa.edu.springboot;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/27.
 */
@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        return Health.down().withDetail("1","2").build();
    }
}

```

## 15.2 监控某一个URL被用了多少次

-使用CounterService进行计数
- counter.user.home.request.count:2 
```
package com.clsaa.edu.springboot;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.metrics.CounterService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created by Egg on 2017/2/27.
 */
@RestController
public class UserController {
    @Autowired
    private CounterService counterService;
    @GetMapping("/user/home")
    public String home(){
        counterService.increment("user.home.request.count");
        return "home";
    }
}

```

## 15.3 为某个监控参数设置值

```
package com.clsaa.edu.springboot;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.actuate.metrics.CounterService;
import org.springframework.boot.actuate.metrics.GaugeService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created by Egg on 2017/2/27.
 */
@RestController
public class UserController {

    @Autowired
    private GaugeService gaugeService;
    @Autowired
    private CounterService counterService;
    @GetMapping("/user/home")
    public String home(){
        counterService.increment("user.home.request.count");
        return "home";
    }

    @GetMapping("/mp3/price")
    public String price(@RequestParam("price") double price){
        gaugeService.submit("mp3.price",price);
        return "mp3 price";
    }
}

```