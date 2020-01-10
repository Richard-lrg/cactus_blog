---
title: Spring常用配置-Profile
date: 2020-01-10 11:23:26
tags:
---
#### 一、Profile是什么？
> 在企业开发中，项目开发环境和产品环境的配置是不同的（如数据库的配置）。 
> Profile为不同环境下使用不同的配置提供了支持

#### 二、如何使用profile
> 1. 通过设定Environment的ActiceProfile来设定当前context（容器）需要使用的配置环境
>    开发中通常使用@Profile注解，达到不同情况实例化不同Bean的目的

#### 三、ProfileDemo
**demobean**
```java
package com.cactus.demo.profile;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:31
 * Description:
 */


public class DemoBean {
    private String content;

    public DemoBean() {
    }

    public DemoBean(String content) {
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
package com.cactus.demo.profile;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:32
 * Description:
 */

@Configuration
public class ProfileConfig {
    @Bean
    @Profile("dev")
    public DemoBean getDemoBeanDev() {
        return new DemoBean("this is dev env");
    }

    @Bean
    @Profile("prod")
    public DemoBean getDemoBeanProd() {
        return new DemoBean("this is prod env");
    }
}

```

**Main**
```java
package com.cactus.demo.profile;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:34
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.getEnvironment().setActiveProfiles("prod");
        // 需要后置注册配置类，否则会报error
        context.register(ProfileConfig.class);
        context.refresh();
        DemoBean bean = context.getBean(DemoBean.class);
        System.out.println(bean.getContent());
        context.close();
    }
}

```

**result**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191015161215275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
