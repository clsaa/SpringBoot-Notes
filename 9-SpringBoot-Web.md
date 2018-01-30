---
layout:

title: 9-SpringBoot-Web

date: 2017-02-09

updated: 2017-02-09

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



# 9-SpringBoot-Web


[TOC]

## 9.1 快速启动一个SpringBootWeb

- @RequestMapping(value = "/user/home")表明URL,默认设置下不限制请求方式,可以使用method方法设置请求方式
- @ResponseBody表明返回数据
- application.properties中server.port=设置端口


```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Created by Egg on 2017/2/25.
 */
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class,args);
    }
}

```

```
package com.clsaa.edu.springboot;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * Created by Egg on 2017/2/25.
 */
@Controller
public class UserController {

    @RequestMapping(value = "/user/home")
    @ResponseBody
    public String home(){
        return "user home";
    }
}

```

- 在4.3.1后还支持以下控制限制请求方式

```
    @ResponseBody
    @GetMapping(value = "/user/show")
    public String show(){
        return "user show";
    }

    @ResponseBody
    @PostMapping
    public String create(){
        return "user create";
    }
```



## 9.2 参数传递

### 9.2.1 @RequestParam获取表单中的参数

- 默认必须提供值,可设置required为false
- 可设置defaultValue默认值

```
    @ResponseBody
    @PostMapping(value = "user/create")
    public String create(@RequestParam("username") String username
    ,@RequestParam("password") String password){
        return "create " + username+":"+password;
    }
```

### 9.2.2 @PathVariable获取url中的参数

```
    @ResponseBody
    @GetMapping("/user/{id}")
    public String display(@PathVariable("id") String id){
        return "user display" + id ;
    }
```

### 9.2.3 通过注入的HttpServletRequest

```
    @ResponseBody
    @GetMapping("/user/host")
    public String host(HttpServletRequest request){
        return "user display " + request.getRemoteHost() ;
    }
```

### 9.2.4 @RestController

- 表示当前controller的方法的返回值可以直接用于body输出

## 9.3 在SpringBoot中使用JSP

- 创建目录src/main/webapp
- 在WEBAPP下创建如下目录
    - src/main/webapp/WEB-INF/jsp/fail.jsp
    - src/main/webapp/WEB-INF/jsp/login.jsp
    - src/main/webapp/WEB-INF/jsp/ok.jsp
- 在application.properties中做如下配置
    - spring.mvc.view.prefix=/WEB-INF/jsp/
    - spring.mvc.view.suffix=.jsp
- 在maven中加入如下依赖

```
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-jasper</artifactId>
        </dependency>

```

```
package com.clsaa.edu.springboot;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * Created by Egg on 2017/2/26.
 */
@Controller
public class LoginController {
    /**
     * 跳转到JSP页面
     * @param username
     * @param password
     * @return
     */
    @PostMapping("/login")
    public String login(@RequestParam("username") String username
            ,@RequestParam("password") String password){
        if (username.equals(password)){
            return "ok";
        }
        return "fail";
    }

    /**
     * 给JSP传参数
     * @param model
     * @return
     */
    @GetMapping("/login")
    public String loginIndex(Model model){
        model.addAttribute("username","root");
        return "login";
    }
}

```

## 9.4 更换web容器

- springboot默认使用tomcat作为web容器
- 可以配置为Jetty
    - 排除tomcat
    - 添加jetty

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>clsaa</groupId>
    <artifactId>springboot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--spring boot 父节点依赖,引入这个之后相关的引入就不需要添加version配置，spring boot会自动选择最合适的版本进行添加。-->
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
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
            <!--
                <version></version>
                由于我们在上面指定了 parent(spring boot)
             -->
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
    </dependencies>

</project>
```

## 9.5 在springboot中获取静态资源

### 9.5.1 将静态资源放置在webapp下

- 如:src/main/webapp/user.html
- 此时我们可以直接在url中获取静态资源:http://localhost:8080/user.html

### 9.5.2 使用ResourceProperties

- org.springframework.boot.autoconfigure.web.ResourceProperties
- 可以看到默认资源访问路径是
    - "classpath:/META-INF/resources/"
    - "classpath:/resources/"
    - "classpath:/static/"
    - "classpath:/public/" 
可以通过spring.resources.staticLocations=classpath:/html/进行配置D:\Data\MyCode\codeMaven\learn_springboot003\src\main\resources\html
```
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties implements ResourceLoaderAware {

	private static final String[] SERVLET_RESOURCE_LOCATIONS = { "/" };

	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
			"classpath:/META-INF/resources/", "classpath:/resources/",
			"classpath:/static/", "classpath:/public/" };

	private static final String[] RESOURCE_LOCATIONS;

	static {
		RESOURCE_LOCATIONS = new String[CLASSPATH_RESOURCE_LOCATIONS.length
				+ SERVLET_RESOURCE_LOCATIONS.length];
		System.arraycopy(SERVLET_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS, 0,
				SERVLET_RESOURCE_LOCATIONS.length);
		System.arraycopy(CLASSPATH_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS,
				SERVLET_RESOURCE_LOCATIONS.length, CLASSPATH_RESOURCE_LOCATIONS.length);
	}

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/] plus context:/ (the root of the servlet context).
	 */
	private String[] staticLocations = RESOURCE_LOCATIONS;

	/**
	 * Cache period for the resources served by the resource handler, in seconds.
	 */
	private Integer cachePeriod;

	/**
	 * Enable default resource handling.
	 */
	private boolean addMappings = true;

	private final Chain chain = new Chain();

	private ResourceLoader resourceLoader;

```

## 9.6 拦截器

- preHandle:请求之前调用,如果返回false则终止执行请求.
- postHandle:请求之后,页面渲染之前执行.
- afterCompletion:请求完成之后执行,一般用于资源清理.

```
/*
 * Copyright 2002-2016 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.web.servlet;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.method.HandlerMethod;

/**
 * Workflow interface that allows for customized handler execution chains.
 * Applications can register any number of existing or custom interceptors
 * for certain groups of handlers, to add common preprocessing behavior
 * without needing to modify each handler implementation.
 *
 * <p>A HandlerInterceptor gets called before the appropriate HandlerAdapter
 * triggers the execution of the handler itself. This mechanism can be used
 * for a large field of preprocessing aspects, e.g. for authorization checks,
 * or common handler behavior like locale or theme changes. Its main purpose
 * is to allow for factoring out repetitive handler code.
 *
 * <p>In an asynchronous processing scenario, the handler may be executed in a
 * separate thread while the main thread exits without rendering or invoking the
 * {@code postHandle} and {@code afterCompletion} callbacks. When concurrent
 * handler execution completes, the request is dispatched back in order to
 * proceed with rendering the model and all methods of this contract are invoked
 * again. For further options and details see
 * {@code org.springframework.web.servlet.AsyncHandlerInterceptor}
 *
 * <p>Typically an interceptor chain is defined per HandlerMapping bean,
 * sharing its granularity. To be able to apply a certain interceptor chain
 * to a group of handlers, one needs to map the desired handlers via one
 * HandlerMapping bean. The interceptors themselves are defined as beans
 * in the application context, referenced by the mapping bean definition
 * via its "interceptors" property (in XML: a &lt;list&gt; of &lt;ref&gt;).
 *
 * <p>HandlerInterceptor is basically similar to a Servlet Filter, but in
 * contrast to the latter it just allows custom pre-processing with the option
 * of prohibiting the execution of the handler itself, and custom post-processing.
 * Filters are more powerful, for example they allow for exchanging the request
 * and response objects that are handed down the chain. Note that a filter
 * gets configured in web.xml, a HandlerInterceptor in the application context.
 *
 * <p>As a basic guideline, fine-grained handler-related preprocessing tasks are
 * candidates for HandlerInterceptor implementations, especially factored-out
 * common handler code and authorization checks. On the other hand, a Filter
 * is well-suited for request content and view content handling, like multipart
 * forms and GZIP compression. This typically shows when one needs to map the
 * filter to certain content types (e.g. images), or to all requests.
 *
 * @author Juergen Hoeller
 * @since 20.06.2003
 * @see HandlerExecutionChain#getInterceptors
 * @see org.springframework.web.servlet.handler.HandlerInterceptorAdapter
 * @see org.springframework.web.servlet.handler.AbstractHandlerMapping#setInterceptors
 * @see org.springframework.web.servlet.handler.UserRoleAuthorizationInterceptor
 * @see org.springframework.web.servlet.i18n.LocaleChangeInterceptor
 * @see org.springframework.web.servlet.theme.ThemeChangeInterceptor
 * @see javax.servlet.Filter
 */
public interface HandlerInterceptor {

	/**
	 * Intercept the execution of a handler. Called after HandlerMapping determined
	 * an appropriate handler object, but before HandlerAdapter invokes the handler.
	 * <p>DispatcherServlet processes a handler in an execution chain, consisting
	 * of any number of interceptors, with the handler itself at the end.
	 * With this method, each interceptor can decide to abort the execution chain,
	 * typically sending a HTTP error or writing a custom response.
	 * <p><strong>Note:</strong> special considerations apply for asynchronous
	 * request processing. For more details see
	 * {@link org.springframework.web.servlet.AsyncHandlerInterceptor}.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @param handler chosen handler to execute, for type and/or instance evaluation
	 * @return {@code true} if the execution chain should proceed with the
	 * next interceptor or the handler itself. Else, DispatcherServlet assumes
	 * that this interceptor has already dealt with the response itself.
	 * @throws Exception in case of errors
	 */
	boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception;

	/**
	 * Intercept the execution of a handler. Called after HandlerAdapter actually
	 * invoked the handler, but before the DispatcherServlet renders the view.
	 * Can expose additional model objects to the view via the given ModelAndView.
	 * <p>DispatcherServlet processes a handler in an execution chain, consisting
	 * of any number of interceptors, with the handler itself at the end.
	 * With this method, each interceptor can post-process an execution,
	 * getting applied in inverse order of the execution chain.
	 * <p><strong>Note:</strong> special considerations apply for asynchronous
	 * request processing. For more details see
	 * {@link org.springframework.web.servlet.AsyncHandlerInterceptor}.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @param handler handler (or {@link HandlerMethod}) that started asynchronous
	 * execution, for type and/or instance examination
	 * @param modelAndView the {@code ModelAndView} that the handler returned
	 * (can also be {@code null})
	 * @throws Exception in case of errors
	 */
	void postHandle(
			HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
			throws Exception;

	/**
	 * Callback after completion of request processing, that is, after rendering
	 * the view. Will be called on any outcome of handler execution, thus allows
	 * for proper resource cleanup.
	 * <p>Note: Will only be called if this interceptor's {@code preHandle}
	 * method has successfully completed and returned {@code true}!
	 * <p>As with the {@code postHandle} method, the method will be invoked on each
	 * interceptor in the chain in reverse order, so the first interceptor will be
	 * the last to be invoked.
	 * <p><strong>Note:</strong> special considerations apply for asynchronous
	 * request processing. For more details see
	 * {@link org.springframework.web.servlet.AsyncHandlerInterceptor}.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @param handler handler (or {@link HandlerMethod}) that started asynchronous
	 * execution, for type and/or instance examination
	 * @param ex exception thrown on handler execution, if any
	 * @throws Exception in case of errors
	 */
	void afterCompletion(
			HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception;

}

```

### 9.5.1 基本使用示例

- 创建handler

```
package com.clsaa.edu.springboot;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created by Egg on 2017/2/26.
 */
@RestController
public class UserController {
    @GetMapping("/user/home")
    public String home(){
        System.out.println("user home");
        return "user home";
    }
}

```

- 实现HanlerInterceptor接口

```
package com.clsaa.edu.springboot;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Created by Egg on 2017/2/26.
 */
public class LogHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandler:"+handler.getClass());
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle:"+handler.getClass());

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion:"+handler.getClass());
    }
}

```

- 写一个类,集成WebMvcConfigurerAdapter抽象类,然后重写addInterceptors把上一步的拦截器加进去

```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringBootConfiguration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

/**
 * Created by Egg on 2017/2/26.
 */
@SpringBootConfiguration
public class WebConfiguration extends WebMvcConfigurerAdapter{
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogHandlerInterceptor());
    }
}

```

- 输出

```
preHandler:class org.springframework.web.method.HandlerMethod
user home
postHandle:class org.springframework.web.method.HandlerMethod
afterCompletion:class org.springframework.web.method.HandlerMethod

```

## 9.6 异常处理

- 默认会使用org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration进行异常的处理
- 下列代码访问url会显示
```
Whitelabel Error Page

This application has no explicit mapping for /error, so you are seeing this as a fallback.

Sun Feb 26 15:45:49 CST 2017
There was an unexpected error (type=Internal Server Error, status=500).
org.springframework.web.util.NestedServletException: Request processing failed; nested exception is java.lang.IllegalArgumentException: args is empty
```

```
package com.clsaa.edu.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * Created by Egg on 2017/2/25.
 */
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class,args);
    }
}

```

```
package com.clsaa.edu.springboot;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created by Egg on 2017/2/26.
 */
@RestController
public class UserController {
    @GetMapping("/user/help")
    public String help(){
        throw new IllegalArgumentException("args is empty");
    }
}

```

- 我们可以通过@SpringBootApplication(exclude = ErrorMvcAutoConfiguration.class)取消默认的异常处理类

### 9.6.1 加入自己的异常处理逻辑

- 实现以下接口
```
public interface ErrorPageRegistrar {

	/**
	 * Register pages as required with the given registry.
	 * @param registry the error page registry
	 */
	void registerErrorPages(ErrorPageRegistry registry);

}
```

- 通过调用registry.addErrorPages(),调用之前我们要先构造ErrorPage,其构造可根据http状态码对应页面也可以根据具体的异常对应页面.
```
	public ErrorPage(HttpStatus status, String path) {
		this.status = status;
		this.exception = null;
		this.path = path;
	}

	public ErrorPage(Class<? extends Throwable> exception, String path) {
		this.status = null;
		this.exception = exception;
		this.path = path;
	}
```

- 代码实现

```
package com.clsaa.edu.springboot;

import org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created by Egg on 2017/2/26.
 */
@RestController
public class UserController {

    @GetMapping("/user/help")
    public String help(){
        throw new IllegalArgumentException("args is empty");
    }
}

```
```
package com.clsaa.edu.springboot;

import org.springframework.boot.web.servlet.ErrorPage;
import org.springframework.boot.web.servlet.ErrorPageRegistrar;
import org.springframework.boot.web.servlet.ErrorPageRegistry;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;

/**
 * Created by Egg on 2017/2/26.
 */

@Component
public class CommonErrorPageRegistry implements ErrorPageRegistrar {

    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND,"/404.html");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR,"/500.html");
        registry.addErrorPages(errorPage404,errorPage500);
    }
}

```

### 9.6.2 SpringMVC当前Controller异常处理

```
package com.clsaa.edu.springboot;

import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.FileNotFoundException;

/**
 * Created by Egg on 2017/2/26.
 */
@RestController
public class BookController  {

    @ExceptionHandler(value = {FileNotFoundException.class,ClassNotFoundException.class})
    public String errorHandlerList(){
        return "file not found exception";
    }

    @GetMapping("/book/list")
    public String list()throws FileNotFoundException{
        throw new FileNotFoundException("file not found");
    }

    @GetMapping("/book/error")
    public String error()throws ClassNotFoundException{
        throw new ClassNotFoundException("class not found");
    }
}

```

### 9.6.3 全局异常处理

```
package com.clsaa.edu.springboot;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * Created by Egg on 2017/2/26.
 */
@ControllerAdvice
public class GlobeExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Demo errorHandler(Exception e){
        Demo demo = new Demo();
        demo.setName("name");
        return demo;
    }
}

```