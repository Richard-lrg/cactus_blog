---
title: Spring高级话题-测试
date: 2020-01-10 11:29:39
tags:
---
#### 一、关于Spring中的测试
> 在开发过程中开发人员会经常使用到两种测试方式，单元测试和集成测试
> * 单元测试：只针对单一的类或方法，对运行环境没有依赖
> * 集成测试：需要来自不同层的不同对象的交互，如数据库，网络连接，ioc容器等
> Spring通过Spring TestContext Framework对集成测试提供了顶级的支持


> 补充： 基于Maven构建的项目结构默认有关于测试的目录：
    > 测试代码：src/test/java
    > 测试资源：src/text/resources

#### 二、如何使用Spring提供的测试功能
> 1. 使用@RunWith注解，让代码运行于Spring测试环境(@RunWith就是一个运行器，SpringJUnit4ClassRunner.class提供了Spring TestContext Framework的功能)
> 2. 使用@ContextConfiguration注解来配置应用容器
> 3. 使用@ActiveProfiles来确定profile

#### 三、testDemo

**实体类**
```java
package com.cactus.demo.test;

/**
 * Created by liruigao
 * Date: 2019-12-09 14:04
 * Description:
 */


public class TestDemo {
    private String content;

    public TestDemo() {
    }

    public TestDemo(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}

```

**配置类**
```java
package com.cactus.demo.test;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

/**
 * Created by liruigao
 * Date: 2019-12-09 14:05
 * Description:
 */

@Configuration
public class TestConfig {
    @Bean
    @Profile("dev")
    public TestDemo getDevBean() {
        return new TestDemo("I am dev testDemo");
    }

    @Bean
    @Profile("prod")
    public TestDemo getProdBean() {
        return new TestDemo("I am prod testDemo");
    }
}

```

**测试运行类**
```java
package testdemo;

import com.cactus.demo.test.TestConfig;
import com.cactus.demo.test.TestDemo;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/**
 * Created by liruigao
 * Date: 2019-12-09 14:10
 * Description:
 */

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TestConfig.class})
@ActiveProfiles("prod")
public class DemoTest {
    @Autowired
    private TestDemo testBean;

    @Test
    public void test() {
        String content = testBean.getContent();
        System.out.println(content);
    }
}

```

**Result**
```text
I am prod testDemo
```
