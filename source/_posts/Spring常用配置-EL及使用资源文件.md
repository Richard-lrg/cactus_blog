---
title: Spring常用配置-EL及使用资源文件
date: 2020-01-10 11:21:41
tags:
---
#### 一、Spring EL是什么？
> Spring表达式语言，支持在xml和注解中使用表达式，类似于JSP的el表达式语言。


#### 二、怎么使用？
> Spring主要在 @Value注解 使用表达式， 实现资源的注入。
> 可以注入包括以下内容：
> * 普通字符
> * 操作系统属性
> * 表达式运算结果
> * 其他Bean的属性
> * 文件内容
> * 网址内容
> * 属性文件

#### 三、ELDemo

**一个憨憨的Bean**
```java
package com.cactus.demo.el;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
 * Created by liruigao
 * Date: 2019-10-11 20:52
 * Description:
 * 通过@Value注入内容
 */

@Component
public class Demo {
    @Value("乱七八糟其他的")
    public String another;

    public String getAnother() {
        return another;
    }

    public void setAnother(String another) {
        this.another = another;
    }
}

```

**一个傻傻的Config文件**
```java
package com.cactus.demo.el;

import org.apache.commons.io.IOUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.env.Environment;
import org.springframework.core.io.Resource;

import java.io.IOException;
import java.util.ArrayList;

/**
 * Created by liruigao
 * Date: 2019-10-11 20:56
 * Description:
 */

@Configuration
@ComponentScan("com.cactus.demo.el")
// 注意配置文件和文本文件要放到resources目录下，才可以获取到该文件
@PropertySource("classpath:eldemo/test.properties")
public class ELConfig {

    @Value("hi 647")
    private String normal;

    @Value("#{systemProperties['os.name']}")
    private String osName;

    // bean名字若未命名，则默认首字母小写
    @Value("#{demo.another}")
    private String demoAnother;

    @Value("classpath:eldemo/test.txt")
    private Resource testFile;

    @Value("http://www.baidu.com")
    private Resource testUrl;

    @Value("${book.name}")
    private String bookName;

    // 配置文件的数据同样可以通过Environment获取
    @Autowired
    private Environment environment;

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyConfigure() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    public void output() {
        try {
            System.out.println(normal);
            System.out.println(osName);
            System.out.println(demoAnother);
            System.out.println(IOUtils.toString(testFile.getInputStream()));
            System.out.println(IOUtils.toString(testUrl.getInputStream()));
            System.out.println(bookName);
            //通过environment获取配置文件数据
            System.out.println("env : " + environment.getProperty("book.author"));
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}

```

**Main**
```java
package com.cactus.demo.el;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-10-12 10:29
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ELConfig.class);
        ELConfig elConfig = context.getBean(ELConfig.class);
        elConfig.output();
        context.close();
    }
}

```

**result**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191015161013551.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
