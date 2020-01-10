---
title: Spring常用配置 - Bean的Scope
date: 2020-01-10 11:20:26
tags:
---
#### 一、Scope是什么？
>**Scope：** 描述Spring容器如何创建Bean的实例


#### 二、Scope具体内容
@Scope的value有5个，分别解释下：

| @Scope | 意义 |
| --- | --- |
| Singleton | Spring的默认配置，一个Spring容器中只有一个Bean的实例 |
| Prototype | 每次调用都会新建一个Bean的实例 |
| Request | Web项目中，会给每个http request新建一个Bean实例 |
| Session | Web项目中，会给每个http session新建一个Bean实例 |
| GlobalSession | portal应用中使用，给每一个global http session新建一个Bean实例 |


#### 三、ScopeDemo

**Bean1**
```java
package com.cactus.demo.scope;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-10-11 20:37
 * Description:
 * 声明为Bean
 * Scope默认为singleton，一个Spring容器中只会存在一个实例
 */

@Service
//@Scope("singleton")
public class BeanOne {
}

```

**Bean2**
```java
package com.cactus.demo.scope;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-10-11 20:37
 * Description:
 * 声明为Bean
 * Scope为prototype，此时每次调用都会新建一个实例
 */

@Service
@Scope("prototype")
public class BeanTwo {
}

```

**配置类**
```java
package com.cactus.demo.scope;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by liruigao
 * Date: 2019-10-11 20:41
 * Description:
 */

@Configuration
@ComponentScan("com.cactus.demo.scope")
public class ScopeConfig {
}

```

**Main**
```java
package com.cactus.demo.scope;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-10-11 20:42
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ScopeConfig.class);
        BeanOne one1 = context.getBean(BeanOne.class);
        BeanOne one2 = context.getBean(BeanOne.class);
        BeanTwo two1 = context.getBean(BeanTwo.class);
        BeanTwo two2 = context.getBean(BeanTwo.class);
        System.out.println("Is one1 and one2 the same?  " + one1.equals(one2));
        System.out.println("Is two1 and two2 the same?  " + two1.equals(two2));
        context.close();
    }
}

```

**result**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191015160903428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
