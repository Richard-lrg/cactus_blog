---
title: Spring高级话题-多线程
date: 2020-01-10 11:25:32
tags:
---
#### 一、Spring中的多线程如何使用
> * Spring通过TaskExecutor(任务执行器)来实现多线程和并发编程，通过ThreadPoolTaskExecutor实现以基于线程池的TaskExecutor。
> * 在实际使用中，我们需要通过@EnableAsync来开启对异步任务的支持，通过@Async来声明一个异步任务


#### 二、AsyncDemo
> 实现一个基于线程池的异步任务demo

**配置类**
```java
package com.cactus.demo.async;

import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

/**
 * Created by liruigao
 * Date: 2019-12-05 15:21
 * Description:
 * @EnableAsync 声明开启异步支持
 * 实现AsyncConfigurer接口并重写对应方法，获得需要的任务执行器
 */

@Configuration
@ComponentScan("com.cactus.demo.async")
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(10);
        threadPoolTaskExecutor.setMaxPoolSize(20);
        threadPoolTaskExecutor.setQueueCapacity(30);
        threadPoolTaskExecutor.initialize();
        return threadPoolTaskExecutor;
    }

    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return null;
    }
}

```

**方法bean**
```java
package com.cactus.demo.async;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-12-05 15:28
 * Description:
 * @Async 声明funcTwo为异步方法，此方法自动注入使用配置类获得任务执行器
 * @Async若置于类上，则此类所有方法均为异步执行
 */

@Service
public class AsyncDemo {
    public void funcOne(Integer i) {
        System.out.println(Thread.currentThread().getName() + " - funcOne : " + i);
    }

    @Async
    public void funcTwo(Integer i) {
        try {
            Thread.sleep(50 - i);
            System.out.println(Thread.currentThread().getName() + " - funcTwo : " + i);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

**Main**
```java
package com.cactus.demo.async;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-12-05 15:30
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AsyncConfig.class);
        AsyncDemo asyncDemo = context.getBean(AsyncDemo.class);
        for (int i = 0; i < 5; i++) {
            asyncDemo.funcOne(i);
        }
        for (int i = 0; i < 5; i++) {
            asyncDemo.funcTwo(i);
        }
        context.close();
    }
}

```

**Result**
```text
main - funcOne : 0
main - funcOne : 1
main - funcOne : 2
main - funcOne : 3
main - funcOne : 4
ThreadPoolTaskExecutor-5 - funcTwo : 4
ThreadPoolTaskExecutor-4 - funcTwo : 3
ThreadPoolTaskExecutor-3 - funcTwo : 2
ThreadPoolTaskExecutor-2 - funcTwo : 1
ThreadPoolTaskExecutor-1 - funcTwo : 0
```

