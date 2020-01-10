---
title: Spring基础配置
date: 2020-01-10 11:19:27
tags:
---
>Spring框架的四大原则：
>1. 使用POJO进行轻量级和最小侵入式开发
>2. 通过依赖注入和基于接口编程实现松耦合
>3. 通过AOP和默认习惯进行声明式编程
>4. 使用AOP和模板（template）减少模式化代码

[对四大原则的理解可以看这里](http://www.west999.com/info/html/chengxusheji/Javajishu/20180719/4347703.html)

#### 一、依赖注入
##### 1.依赖注入的概念：
>依赖注入是指容器负责创建对象和维护对象间的依赖关系，而非通过对象本身负责自己的创建和解决自己的依赖。

##### 2.依赖注入的主要目的：
>主要目的就是解耦，所体现的是java组合的理念。毫无疑问组合相对于继承，耦合度会大大降低。

##### 3.具体实现
>Spring IoC容器（ApplicationContext）负责创建Bean，并通过容器将功能类Bean注入到你所需要的Bean中。
>Spring提供xml、注解、Java配置，这三种方式都被称为配置元数据。即描述数据的数据，本身不具备任何执行能力，只能通过外界代码对元数据解析后进行一些有意义的操作。

##### 4.涉及到的注解（推荐注解的方式，xml效率低且复杂）
**声明式Bean的注解：**
* @Component   没有明确角色
* @Service     业务逻辑层使用
* @Repository  数据访问层使用
* @Controller  展现层使用（提供接口的方法类） 

**注入Bean的注解**
* @Autowired    Spring提供的注解（通常使用这个）
* @Inject       JSR-330提供的注解
* @Resource     JSR-250提供的注解
（注入Bean的注解 可注解到set方法上或者属性上，具体根据实际开发场景选择）

##### 5.Ioc&Di Demo
**方法bean1**
```java
package com.cactus.iocdidemo;

import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-10-11 11:02
 * Description:
 * 1. @Service 声明FuncOneService为bean
 */

@Service
public class FuncOneService {
    public String show(String word) {
        return "Function one show : " + word;
    }
}

```

**方法bean2**
```java
package com.cactus.iocdidemo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-10-11 11:03
 * Description:
 * 1. @Service 声明 FuncTwoService 为bean
 * 2. @Autowired 将 FuncOneService 注入到 FuncTwoService
 */

@Service
public class FuncTwoService {
    @Autowired
    private FuncOneService funcOneService;

    public String show(String word) {
        return funcOneService.show(word);
    }
}

```

**配置类**
```java
package com.cactus.iocdidemo;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by liruigao
 * Date: 2019-10-11 11:07
 * Description:
 * 1. @Configuration 声明注册类
 * 2. @ComponentScan扫描指定包，并注册为Bean
 */

@Configuration
@ComponentScan("com.cactus.iocdidemo")
public class BeanConfig {
}

```

**Main**
```java
package com.cactus.iocdidemo;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-10-11 11:05
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        // 使用AnnotationConfigApplicationContext容器，并选择BeanConfig为配置类
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BeanConfig.class);

        // bean已有容器创建，从容器中获取方法bean
        FuncTwoService funcTwoService = context.getBean(FuncTwoService.class);

        String showWord = funcTwoService.show("hi ioc&di");
        System.out.println(showWord);
        
        context.close();

    }
}

```

**result：**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019101116211179.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)


#### 二、Java配置

##### 1.Java配置简介
>Java配置是Spring 4.x和Spring Boot推荐的配置方式，可以完全替代xml配置。
>通过@Configuration和@Bean来实现：
> * @Configuration 声明当前类是个配置类，相当于一个xml文件
> * @Bean         注解到方法上，声明当前方法返回值是个Bean

##### 2.什么时候使用Java配置
> 全局配置使用java配置，如数据库相关配置、MVC相关配置
> 业务Bean的配置使用注解配置，如@Service、@Component、@Repository、@Controller


##### 3.JavaConfig demo
**FuncOneService**
```java
package com.cactus.javaconfigdemo;

/**
 * Created by liruigao
 * Date: 2019-10-11 11:33
 * Description: 普通方法类，不声明为Bean
 */


public class FuncOneService {
    public String show(String word) {
        return "java config, Function one show : " + word;
    }
}

```
**FuncTwoService**
```java
package com.cactus.javaconfigdemo;

/**
 * Created by liruigao
 * Date: 2019-10-11 11:34
 * Description:普通方法类， 不声明为bean， 不依赖注入
 */


public class FuncTwoService {
    private FuncOneService funcOneService;

    public void setFuncOneService(FuncOneService funcOneService) {
        this.funcOneService = funcOneService;
    }

    public String show(String word) {
        return funcOneService.show(word);
    }
}

```
**JavaConfig**
```java
package com.cactus.javaconfigdemo;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by liruigao
 * Date: 2019-10-11 11:36
 * Description:
 * 1. 声明为配置类
 * 2. 无需进行包扫描，因为所有的bean都在这里定义
 * 3. 通过使用@Bean注解， 可以更加灵活地创建管理Bean
 */

@Configuration
public class JavaConfig {

    @Bean
    public FuncOneService funcOneService() {
        return new FuncOneService();
    }

    @Bean
    public FuncTwoService funcTwoService() {
        FuncTwoService funcTwoService = new FuncTwoService();
        funcTwoService.setFuncOneService(funcOneService());
        return funcTwoService;
    }
}

```
**Main**
```java
package com.cactus.javaconfigdemo;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-10-11 11:43
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(JavaConfig.class);
        FuncTwoService funcTwoService = context.getBean(FuncTwoService.class);

        System.out.println(funcTwoService.show("hi javaConfig"));

        context.close();
    }
}

```
**result**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011162129763.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)


#### 三、AOP

##### 1.AOP的由来
> **AOP: 面向切面编程**

>OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。当我们需 要为分散的对象引入公共行为的时候，OOP则显得无能为力。也就是说，OOP允许你定义从上到下的关系，但并不适合定义从左到右的关系。
>AOP技术利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为 “Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低 模块间的耦合度，并有利于未来的可操作性和可维护性。
>AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为； 那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天功的妙手 将这些剖开的切面复原，不留痕迹。

##### 2.AOP的目的
> Spring AOP存在的目的就是为了解耦。 让一组类共享相同的行为

##### 3.涉及到的注解

**切面编程涉及到的注解：**
* @Aspect 声明一个切面
* @PointCut 声明一个切点
* @Before, @After, @Around 定义建言， 可直接将切点（或者方法拦截规则）作为参数

**创建一个注解所涉及到的注解：**

* @interface 声明一个注解
* @Target 用于描述注解的使用范围
* @Retention 用于描述一个注解存在的生命周期


##### 4.AOP Demo

**pom文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.cactus</groupId>
    <artifactId>springdemo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
<!--        Spring容器-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
<!--        Aop-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
<!--        aspectj-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.8.13</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.13</version>
        </dependency>
    </dependencies>


</project>
```

**拦截注解**
```java
package com.cactus.aopdemo;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Created by liruigao
 * Date: 2019-10-11 15:01
 * Description: 拦截规则的注解
 * 1. @Target
 * 用于描述注解的使用范围
 * ElementType枚举类型，元注解中的枚举值决定了一个注解可以标记的范围
 *      TYPE ： 类型上面  用于描述类、接口(包括注解类型) 或enum声明
 *      FIELD ： 用于描述字段
 *      METHOD ：方法
 *      PARAMETER ： 参数 【参数名】
 *      CONSTRUCTOR ： 构造方法
 *      LOCAL_VARIABLE ： 局部变量
 *      ANNOTATION_TYPE ： 可以打在注解上面
 *      PACKAGE ：可以打在包上面
 *      TYPE_PARAMETER ： 参数类型【形式参数类型】
 *2. @Retention
 * 用于描述一个注解存在的生命周期【源码，字节码文件，运行时】
 * 枚举值RetentionPolicy：几个值决定了几个状态：
 * 		SOURCE ：表示一个注解可以存在于源码中==>java的源码中
 *      CLASS ：表示 一个注解可以在源码中，并且可以在字节码文件中
 *      RUNTIME ：表示 一个注解可以在源码、字节码、及运行时期该注解都会存在
 *
 */

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Action {
    String name();
}

```

**被拦截类1**
```java
package com.cactus.aopdemo;

import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-10-11 15:16
 * Description:
 *  被拦截类，使用拦截注解
 */

@Service
public class FuncOneService {

    @Action(name = "注解式拦截")
    public void show() {
        System.out.println("function one show time!");
    }
}

```

**被拦截类2**
```java
package com.cactus.aopdemo;

import org.springframework.stereotype.Service;

/**
 * Created by liruigao
 * Date: 2019-10-11 15:42
 * Description:
 * 被拦截类， 使用方法规则拦截
 */

@Service
public class FuncTwoService {
    public void show() {
        System.out.println("this is func2 showtime");
    }

    public void test() {
        System.out.println("this is func2 showtime test");
    }
}

```

**切面类**
```java
package com.cactus.aopdemo;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

/**
 * Created by liruigao
 * Date: 2019-10-11 15:19
 * Description: 切面类
 * @Aspect 声明切面
 * @Component 声明Bean
 * @Pointcut 声明切点
 * @After 声明一个建言，并使用@Pointcut声明的切点。
 */

@Aspect
@Component
public class FuncAspect {

    // 使用注解拦截-start

    @Pointcut("@annotation(com.cactus.aopdemo.Action)")
    public void annotationPointCut(){};

    // 可通过反射获得注解上的属性，然后做自定义操作
    @After("annotationPointCut()")
    public void after(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature)joinPoint.getSignature();
        Method method = signature.getMethod();
        Action action = method.getAnnotation(Action.class);
        System.out.println("FuncAspect after, action name : " + action.name());
    }

    @Before("annotationPointCut()")
    public void before(JoinPoint joinPoint) {
        System.out.println("FuncAspect after, no operation, 注解式拦截");
    }

    // 使用注解拦截-end

    // 使用方法规则拦截-start

    @Before("execution(* com.cactus.aopdemo.FuncTwoService.*(..))")
    public void beforef(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature)joinPoint.getSignature();
        Method method = signature.getMethod();
        System.out.println("方法规则拦截-before， methodName:" + method.getName());
    }

    @After("execution(* com.cactus.aopdemo.FuncTwoService.*(..))")
    public void afterf(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature)joinPoint.getSignature();
        Method method = signature.getMethod();
        System.out.println("方法规则拦截-after， methodName:" + method.getName());
    }
    // 使用方法规则拦截-end
}

```

**配置类**
```java
package com.cactus.aopdemo;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

/**
 * Created by liruigao
 * Date: 2019-10-11 15:38
 * Description: AOP demo 配置类
 * @EnableAspectJAutoProxy 开启Spring对AspectJ的支持
 */

@Configuration
@ComponentScan("com.cactus.aopdemo")
@EnableAspectJAutoProxy
public class AopConfig {
}

```

**Main**
```java
package com.cactus.aopdemo;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-10-11 15:37
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AopConfig.class);
        FuncOneService funcOneService = context.getBean(FuncOneService.class);
        FuncTwoService funcTwoService = context.getBean(FuncTwoService.class);
        // 注解式拦截
        funcOneService.show();
        // 方法规则式拦截
        funcTwoService.show();
        funcTwoService.test();

        context.close();
    }
}

```

**result**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011162149201.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)

