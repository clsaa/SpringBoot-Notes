---
layout:

title: 16-Springboot-测试

date: 2017-02-16

updated: 2017-02-16

tags:
- SpringBoot
- SpringBoot与微服务
- SpringBoot实战与原理
- Spring
- 测试

categories: SpringBoot实战与原理分析

permalink:

thumbnail:

toc: true

comment: true

notag: false

top: false

---


# 16-Springboot-测试

[TOC]

## 16.1 基本使用

```
package com.clsaa.edu.springboot;

import org.springframework.stereotype.Repository;

/**
 * Created by Egg on 2017/2/27.
 */
@Repository
public class UserDao {
    public Integer addUser(String username){
        System.out.println("user dao adduser");
        if (username == null){
            return 0;
        }
        return 1;
    }
}

```

```
package com.clsaa.edu.springboot;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import static org.junit.Assert.*;

/**
 * Created by Egg on 2017/2/27.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserDaoTest {
    @Autowired
    private  UserDao userDao;
    @Test
    public void addUser() throws Exception {
        Assert.assertEquals(Integer.valueOf(1),userDao.addUser("root"));
        Assert.assertEquals(Integer.valueOf(0),userDao.addUser(null));
    }

}
```
```
package com.clsaa.edu.springboot;

import com.clsaa.edu.springboot.bean.User;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * Created by Egg on 2017/2/27.
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationContextTest {
    @Autowired
    private ApplicationContext context;

    @Test
    public void testNull(){
        Assert.assertNotNull(context.getBean(User.class));
    }
}

```

## 16.2 在测试环境中部署一些特殊的bean

```
package com.clsaa.edu.springboot;

import com.clsaa.edu.springboot.bean.User;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * Created by Egg on 2017/2/27.
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {TestBeanConfiguration.class})
public class ApplicationContextTest {
    @Autowired
    private ApplicationContext context;

    @Test
    public void testNull(){
        Assert.assertNotNull(context.getBean(Runnable.class));
    }
}

```
```
package com.clsaa.edu.springboot;

import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;

/**
 * Created by Egg on 2017/2/27.
 */
@TestConfiguration
public class TestBeanConfiguration {

    @Bean
    public Runnable createRunnable(){
        return ()->{};
    }
}

```

## 16.3 测试环境中的配置文件

- D:\Data\MyCode\codeMaven\learn_springboot004\src\test\resources 在test下创建resources目录与生产环境中resources目录等价
- springboot优先在测试环境中取D:\Data\MyCode\codeMaven\learn_springboot004\src\test\resources下的配置文件,测试环境下没有才会加载正式环境下.

## 16.4 测试接口

```
package com.clsaa.edu.springboot;

/**
 * Created by Egg on 2017/2/27.
 */
public interface UserMapper {

    Integer createUser(String name);

}

```

```
package com.clsaa.edu.springboot;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.BDDMockito;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;
@RunWith(SpringRunner.class)
public class UserMapperTest {

    @MockBean
    private UserMapper userMapper;

    @Test
    public void testCreateUser(){
        BDDMockito.given(userMapper.createUser("admin")).willReturn(Integer.valueOf(1));
        BDDMockito.given(userMapper.createUser("")).willReturn(Integer.valueOf(0));
        BDDMockito.given(userMapper.createUser(null)).willThrow(NullPointerException.class);
        Assert.assertEquals(Integer.valueOf(1),userMapper.createUser("admin"));
        Assert.assertEquals(Integer.valueOf(0),userMapper.createUser(""));
    }
}
```

## 16.5 测试controller

- @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)注意使用这个注解设置测试时的web环境

```
package com.clsaa.edu.springboot;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class BookController {
    @GetMapping("/book/home")
    public String home(){
        return "book home";
    }
}

```

```
package com.clsaa.edu.springboot;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import static org.junit.Assert.*;

/**
 * Created by Egg on 2017/2/28.
 */

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class BookControllerTest {
    @Autowired
    private TestRestTemplate restTemplate;
    @Test
    public void home() throws Exception {
        String body = restTemplate.getForObject("/book/home",String.class);
        Assert.assertEquals("book home",body);
    }

}
```