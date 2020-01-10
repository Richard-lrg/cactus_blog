---
title: Spring高级话题-条件注解
date: 2020-01-10 11:27:18
tags:
---
#### 一、什么是条件注解
> 根据特定的条件来控制Bean的创建行为
> 使用到的注解 @Conditional

#### 二、如何使用条件注解
> 1. 通过实现Condition接口并重写matches方法（构造判断条件）来实现一个条件判断类
> 2. 在配置Bean时使用@Conditional注解，并指定条件判断类，实现有条件地创建Bean

#### 三、conditionDemo
> 通过判断程序在什么系统下运行，来创建对应的Bean，并输出该系统下列表展示命令

**条件判断类**
```java
package com.cactus.demo.conditional;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * Created by liruigao
 * Date: 2019-12-05 16:08
 * Description:
 */


public class MacCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String osName = context.getEnvironment().getProperty("os.name");
        System.out.println("osName : " + osName);
        return osName.contains("Mac");
    }
}

```

**条件判断类**
```java
package com.cactus.demo.conditional;


import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * Created by liruigao
 * Date: 2019-12-05 16:13
 * Description:
 */


public class WindowsCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String osName = context.getEnvironment().getProperty("os.name");
        System.out.println("osName : " + osName);
        return osName.contains("windows");
    }
}

```

**demoBean接口类**
```java
package com.cactus.demo.conditional;

/**
 * Created by liruigao
 * Date: 2019-12-05 16:15
 * Description:
 */


public interface IListService {
    public void list();
}

```
**demoBean**
```java
package com.cactus.demo.conditional;

/**
 * Created by liruigao
 * Date: 2019-12-05 16:16
 * Description:
 */


public class MacListService implements IListService {
    public void list() {
        System.out.println("mac command:  ls");
    }
}

```
**demoBean**
```java
package com.cactus.demo.conditional;

/**
 * Created by liruigao
 * Date: 2019-12-05 16:16
 * Description:
 */


public class WindowsListService implements IListService {
    public void list() {
        System.out.println("windows command : dir");
    }
}
```

**配置类**
```java
package com.cactus.demo.conditional;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

/**
 * Created by liruigao
 * Date: 2019-12-05 16:04
 * Description:
 */

@Configuration
public class ConditionalConifg {

    @Bean
    @Conditional(WindowsCondition.class)
    public IListService windowsListService() {
        return new WindowsListService();
    }

    @Bean
    @Conditional(MacCondition.class)
    public IListService macListService() {
        return new MacListService();
    }
}

```

**Main**
```java
package com.cactus.demo.conditional;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by liruigao
 * Date: 2019-12-05 16:02
 * Description:
 */


public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(ConditionalConifg.class);
        IListService listService = context.getBean(IListService.class);
        listService.list();
        context.close();
    }
}

```

**Result**
```text
WindowsCondition --> osName : Mac OS X
MacCondition --> osName : Mac OS X
mac command:  ls
```
