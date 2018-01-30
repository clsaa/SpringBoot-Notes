---
layout:

title: 10-SpringBoot定制和优化内嵌Tomcat

date: 2017-02-10

updated: 2017-02-10

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- SpringBoot定制与优化

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---



# 10-SpringBoot定制和优化内嵌Tomcat

[TOC]

## 10.1 配置日志

### 10.1.1 在配置文件中配置

- 在application.properties中添加
    - server.tomcat.accesslog.enabled=true
    - server.tomcat.accesslog.directory=d:/temp/logs

## 10.2 EmbeddedServletContainerCustomizer接口,通过代码配置tomcat

```
package com.clsaa.edu.springboot;

import org.apache.catalina.connector.Connector;
import org.apache.catalina.valves.AccessLogValve;
import org.apache.catalina.valves.Constants;
import org.apache.coyote.http11.Http11NioProtocol;
import org.springframework.boot.context.embedded.ConfigurableEmbeddedServletContainer;
import org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer;
import org.springframework.boot.context.embedded.tomcat.TomcatConnectorCustomizer;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.stereotype.Component;

import java.io.File;

/**
 * Created by Egg on 2017/2/26.
 */
@Component
public class MyEmbeddedServletContainerCustomizer implements EmbeddedServletContainerCustomizer {
    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        TomcatEmbeddedServletContainerFactory factory = (TomcatEmbeddedServletContainerFactory) container;
        factory.setPort(10102);
        factory.setBaseDirectory(new File("d:/temp/tomcat"));
        factory.addContextValves(getLogAccessLogValue());
        //设置自定义链接器
        factory.addConnectorCustomizers(new MyTomcatConnectorCustomizer());
    }

    private AccessLogValve getLogAccessLogValue(){
        AccessLogValve logValve = new AccessLogValve();
        logValve.setDirectory("d:/temp/logs");
        logValve.setEnabled(true);
        logValve.setPattern(Constants.AccessLog.COMMON_PATTERN);
        logValve.setPrefix("spring boot log ");
        logValve.setSuffix(".txt");
        return logValve;
    }
    class MyTomcatConnectorCustomizer implements TomcatConnectorCustomizer{

        @Override
        public void customize(Connector connector) {
            //协议的handler
            System.out.println(connector.getProtocolHandler());
            Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
            //设置最大连接数
            protocol.setMaxConnections(200);
            //设置最大线程数
            protocol.setMaxThreads(50);

        }
    }
}

```

## 10.3 使用配置类

```
package com.clsaa.edu.springboot;

import org.apache.catalina.connector.Connector;
import org.apache.catalina.valves.AccessLogValve;
import org.apache.catalina.valves.Constants;
import org.apache.coyote.http11.Http11NioProtocol;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.context.embedded.EmbeddedServletContainerFactory;
import org.springframework.boot.context.embedded.tomcat.TomcatConnectorCustomizer;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;

import java.io.File;

/**
 * Created by Egg on 2017/2/26.
 */
@SpringBootConfiguration
public class WebServerConfiguration {
    @Bean
    public EmbeddedServletContainerFactory createEmbeddedServletContainerFactory(){
        TomcatEmbeddedServletContainerFactory factory = new TomcatEmbeddedServletContainerFactory();
        factory.setPort(10102);
        factory.setBaseDirectory(new File("d:/temp/tomcat"));
        factory.addContextValves(getLogAccessLogValue());
        //设置自定义链接器
        factory.addConnectorCustomizers(new MyTomcatConnectorCustomizer());
        return factory;
    }
    private AccessLogValve getLogAccessLogValue(){
        AccessLogValve logValve = new AccessLogValve();
        logValve.setDirectory("d:/temp/logs");
        logValve.setEnabled(true);
        logValve.setPattern(Constants.AccessLog.COMMON_PATTERN);
        logValve.setPrefix("spring boot log new");
        logValve.setSuffix(".txt");
        return logValve;
    }
    class MyTomcatConnectorCustomizer implements TomcatConnectorCustomizer {

        @Override
        public void customize(Connector connector) {
            //协议的handler
            System.out.println(connector.getProtocolHandler());
            Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
            //设置最大连接数
            protocol.setMaxConnections(200);
            //设置最大线程数
            protocol.setMaxThreads(50);

        }
    }
}

```