---
title: Spring高级话题-元注解与组合注解
date: 2020-01-10 11:27:54
tags:
---
#### 一、什么是元注解、组合注解
> 元注解：可以注解到别的注解上的注解
> 组合注解： 被注解的注解


#### 二、annotationCombineDemo

**组合注解**
```java
package com.cactus.demo.annotation_combine;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Created by liruigao
 * Date: 2019-12-09 11:16
 * Description:
 * 组合@Configuration与@ComponentScan
 * 一篇比较好的属性覆盖讲解文章（https://www.cnblogs.com/goodAndyxublog/p/11181200.html）
 */
// 注解使用范围，TYPE ： 类型上面  用于描述类、接口(包括注解类型) 或enum声明
@Target(ElementType.TYPE)
// 注解生命周期，RUNTIME ：表示 一个注解可以在源码、字节码、及运行时期该注解都会存在
@Retention(RetentionPolicy.RUNTIME)
// documented by javadoc
@Documented
@Configuration
@ComponentScan
public @interface BriefConfiguration {
    // 同名属性隐式覆盖
    String[] value() default {};
}

```

**方法bean**
```java
package com.cactus.demo.annotation_combine;

import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-12-09 11:24
 * Description:
 */

@Service
public class ACDemo {
    public void show() {
        System.out.println("I am created by @BriefConfiguration");
    }
}

```

**配置类**
```java
package com.cactus.demo.annotation_combine;

/**
 * Created by liruigao
 * Date: 2019-12-09 11:25
 * Description:
 */

@BriefConfiguration("com.cactus.demo.annotation_combine")
public class ACConfig {
}

```

**Main**
```java
package com.cactus.demo.annotation_combine;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-12-09 11:26
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ACConfig.class);
        ACDemo acDemo = context.getBean(ACDemo.class);
        acDemo.show();
        context.close();
    }
}

```

**Result**
```text
I am created by @BriefConfiguration
```
