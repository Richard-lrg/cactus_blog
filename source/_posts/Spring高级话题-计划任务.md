---
title: Spring高级话题-计划任务
date: 2020-01-10 11:26:29
tags:
---
#### 一、什么是计划任务
> 就相当于一个定时器，可以使代码在固定的日期时间执行

#### 二、在Spring中如何使用计划任务
> 使用@EnableScheduling开启对计划任务的支持
> 使用@Scheduled声明一个计划任务 （支持多类型，包括cron, fixDelay, fixRate）

#### 三、scheduleDemo

**配置类**
```java
package com.cactus.demo.schedule;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * Created by liruigao
 * Date: 2019-12-05 15:48
 * Description:
 */

@Configuration
@ComponentScan("com.cactus.demo.schedule")
@EnableScheduling
public class ScheduleConfig {
}

```
**执行类**
```java
package com.cactus.demo.schedule;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

import java.util.Date;

/**
 * Created by liruigao
 * Date: 2019-12-05 15:45
 * Description:
 */

@Service
public class ScheduleDemo {
    @Scheduled(fixedDelay = 3000)
    public void taskOne() {
        System.out.println("taskOne - " + new Date());
    }

    @Scheduled(cron = "0 53 15 ? * *")
    public void taskTwo() {
        System.out.println("taskTwo - " + new Date());
    }
}

```

**Main**
```java
package com.cactus.demo.schedule;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-12-05 15:49
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ScheduleConfig.class);
        ScheduleDemo scheduleDemo = context.getBean(ScheduleDemo.class);
    }
}
```

**Result**
```text
taskOne - Mon Dec 09 15:04:53 CST 2019
taskOne - Mon Dec 09 15:04:56 CST 2019
taskOne - Mon Dec 09 15:04:59 CST 2019
taskTwo - Mon Dec 09 15:05:00 CST 2019
taskOne - Mon Dec 09 15:05:02 CST 2019
taskOne - Mon Dec 09 15:05:05 CST 2019
```
