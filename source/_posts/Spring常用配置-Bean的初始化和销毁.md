---
title: Spring常用配置-Bean的初始化和销毁
date: 2020-01-10 11:22:46
tags:
---
#### 一、如何Bean的生命周期进行操作
> Spring对Bean的生命周期操作提供了支持
> java配置方式： 使用Bean的initMethod和destoryMethod进行配置

#### 二、BeanWayDemo
**Bean**
```java
package com.cactus.demo.beanway;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:19
 * Description:
 */


public class BeanWayService {
    public void init() {
        System.out.println("bean init!");
    }

    public void destory() {
        System.out.println("bean destory!");
    }
}

```

**配置类**
```java
package com.cactus.demo.beanway;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:21
 * Description:
 */

@Configuration
@ComponentScan("com.cactus.demo.beanway")
public class BeanWayConfig {
    @Bean(initMethod = "init", destroyMethod = "destory")
    public BeanWayService getBeanWayService() {
        return new BeanWayService();
    }
}

```

**Main**
```java
package com.cactus.demo.beanway;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:23
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeanWayConfig.class);
        BeanWayService beanWayService = context.getBean(BeanWayService.class);
        context.close();
    }
}

```

**result**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019101516111231.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
