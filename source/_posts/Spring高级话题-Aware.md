---
title: Spring高级话题-Aware
date: 2020-01-10 11:24:58
tags:
---
#### 一、Aware是什么？
> Spring Aware就是一些定义了Spring容器本身功能资源的接口

##### Spring提供的Aware接口
|接口|备注|
| --- | --- |
|BeanNameAware| 获得到容器中Bean的名称 |
|BeanFactoryAware|获得当前 bean factory,这样可以调用容器的服务|
|ApplicationContextaware*|当前的 application context,这样可以调用容器的服务|
|MessageSourceAware|获得 message source,这样可以获得文本信息|
|ApplicationEventPublisherAware|应用事件发布器,可以发布事件|
|ResourceLoaderAware|获得资源加载器,可以获得外部资源文件|

#### 二、什么时候用Aware

> 当某一个Bean需要获得Spring容器的服务时，可以实现对应的Aware接口。
> 注意: 这样会造成Bean与Spring框架的耦合性增加

#### 三、AwareDemo

**bean**

```java
package com.cactus.demo.aware;

import org.apache.commons.io.IOUtils;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-12-05 14:35
 * Description:
 */

@Service
public class AwareDemo implements BeanNameAware, ResourceLoaderAware {
    private String beanName;
    private ResourceLoader resourceLoader;
    public void setBeanName(String s) {
        this.beanName = s;
    }

    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    public void show() {
        System.out.println("beanName : " + beanName);
        Resource resource = resourceLoader.getResource("classpath:awaredemo/awaredemo.txt");
        String desc = resource.getDescription();
        System.out.println("resource desc : " + desc);
        try {
            String streamStr = IOUtils.toString(resource.getInputStream(), "utf-8");
            System.out.println("resource streamStr : " + streamStr);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

**配置类**
```java
package com.cactus.demo.aware;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by liruigao
 * Date: 2019-12-05 14:43
 * Description:
 */

@Configuration
@ComponentScan("com.cactus.demo.aware")
public class AwareConfig {
}

```

**awaredemo.txt**
```txt
this is a awaredemo!
```

**Main**
```java
package com.cactus.demo.aware;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-12-05 14:45
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AwareConfig.class);
        AwareDemo awareDemo = context.getBean(AwareDemo.class);
        awareDemo.show();
        context.close();
    }
}
```

**Result**
```
beanName : awareDemo
resource desc : class path resource [awaredemo/awaredemo.txt]
resource streamStr : this is a awaredemo!
```
