---
title: Spring常用配置-Event
date: 2020-01-10 11:24:07
tags:
---
#### 一、什么是Event？
> Spring的Event(事件)为Bean与Bean之间的消息通信提供了支持。
> 通俗来说，当BeanA处理完事情，我们希望BeanB知道BeanA处理了这件事情并作出相应处理，这个时候我们就要用到Event了


#### 二、如何使用Event？
> 1. 自定义事件，继承ApplicationEvent
> 2. 定义事件监听器，实现ApplicationListener
> 3. 使用容器发布事件


#### 三、EventDemo
**事件类**
```java
package com.cactus.demo.event;

import org.springframework.context.ApplicationEvent;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:44
 * Description:
 */


public class DemoEvent extends ApplicationEvent {

    private String msg;

    public DemoEvent(Object source, String msg) {
        super(source);
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}

```

**监听器类**
```java
package com.cactus.demo.event;

import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:45
 * Description:
 */

@Component
public class EventListener implements ApplicationListener<DemoEvent> {

    public void onApplicationEvent(DemoEvent demoEvent) {
        String msg = demoEvent.getMsg();
        System.out.println("I had recived some msgs from publisher : " + msg);
    }
}

```

**容器发布类**
```java
package com.cactus.demo.event;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:48
 * Description:
 */

@Component
public class DemoPublisher {
    @Autowired
    private ApplicationContext applicationContext;

    public void publish(String msg) {
        applicationContext.publishEvent(new DemoEvent(this, msg));
    }
}

```

**配置类**
```java
package com.cactus.demo.event;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:51
 * Description:
 */

@Configuration
@ComponentScan("com.cactus.demo.event")
public class EventConfig {
}

```

**Main**
```java
package com.cactus.demo.event;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-10-15 14:50
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(EventConfig.class);
        DemoPublisher publisher = context.getBean(DemoPublisher.class);
        publisher.publish("hi honey~");
        context.close();
    }
}

```

**result**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191015161321785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
