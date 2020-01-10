---
title: Spring 概述
date: 2020-01-10 11:14:53
tags:
---
##### 一、Spring概述
>The Spring Framework is an application framework and inversion of control container for the Java platform. The framework's core features can be used by any Java application, but there are extensions for building web applications on top of the Java EE (Enterprise Edition) platform. Although the framework does not impose any specific programming model, it has become popular in the Java community as an addition to, or even replacement for the Enterprise JavaBeans (EJB) model. The Spring Framework is open source.


>Spring框架是 Java 平台的一个开源的全栈（Full-stack）应用程序框架和控制反转容器实现，一般被直接称为 Spring。该框架的一些核心功能理论上可用于任何 Java 应用，但 Spring 还为基于Java企业版平台构建的 Web 应用提供了大量的拓展支持。虽然 Spring 没有直接实现任何的编程模型，但它已经在 Java 社区中广为流行，基本上完全代替了企业级JavaBeans（EJB）模型。且Spring框架是开源的。


##### 二、Spring的模块
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011161754478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)

Spring是模块化的，这意味着你可以只使用你需要的Spring模块。

| Core Container |  | AOP |  | Messaging |  |
| --- | --- | --- | --- | --- | --- |
| Spring-Core | 核心工具类，其他模块大量使用Spring-Core | Spring-AOP | 基于代理的AOP支持 | Spring-Messaging | 对消息架构和协议的支持 |
| Spring-Beans | 定义对Bean的支持 | Spring-Aspects | 基于AspectJ的AOP支持 |
| Spring-Context | 运行时的Spring容器 |
| Spring-Context-Suppot | Spring容器对第三方包的集成支持 |
| Spring-Expression | 使用表达式语言在运行时查询和操作对象 |


| Web |  | Data Access |  | Test |  |
| --- | --- | --- | --- | --- | --- |
| Spring-web | 提供基础的web集成的功能 | Spring-JDBC | 提供以JDBC访问数据库的支持 |Test模块主要是针对spring的各个模块做各种各样的测试，包括单元测试、集成测试等等。||
| Spring-Webmvc | 提供基于Servlet的SpringMVC | Spring-TX | 提供编程式和声明式的事务支持 |
| Spring-WebSocket | 提供WebSocket功能 | Spring-ORM | 提供对对象/关系映射技术的支持 |
| Spring-Webmvc-Portlet | 提供Portlet环境支持 | Spring-OXM | 提供对对象/xml映射技术的支持 |
||| Spring-JMS | 提供对JMS的支持 |

##### 三、Spring的生态
>###### 1.Spring Boot
>Spring Boot是一个开发基于Spring的脚手架项目，它默认集成了嵌入式Tomcat，配置注解化，支持快速集成第三方开发组件（如MyBatis），大大降低了使用Spring的门槛，而且内置了许多可以直接用于生产环境的功能，是目前用于开发微服务架构项目的不二选择。
>
>###### 2.Spring Cloud
>
>Spring Cloud为开发基于微服务架构的软件系统提供了一整套工具集合，其中包含了开发各个微服务组件的具体项目，如：Spring Cloud Config（配置中心），Spring Cloud Netflix（服务注册中心），Spring Cloud Sleuth（服务调用监控），Spring Cloud Gateway（服务网关）等等。
>Spring Cloud的基础是Spring Boot，基于Spring Boot可以大大简化开发各微服务组件的流程。
>
>###### 3.Spring Cloud Data Flow
>
>Spring Cloud Data Flow用于构建在云环境或K8S中基于微服务的实时或批数据处理架构，具体来讲就是支持一系列需要进行数据处理的场景，如：ETL，数据导入/导出，事件流，预测分析等等。
>
>###### 4.Spring Data
>
>Spring Data旨在提供一套基于Spring编程模型的数据访问API，是一个数据访问框架集合，其中包含了多个具体的支持不同方式访问特定数据库类型的子模块，如：Spring Data JDBC（使用JDBC方式访问关系型数据库），Spring Data MongoDB（访问MongoDB数据库）等。
>
>###### 5.Spring Integration
>
>Spring Integration的目的是提供一个简单的模型，用于构建企业级应用集成解决方案。
>
>###### 6.Spring Batch
>
>Spring Batch是一个轻量级的批处理框架，旨在开发对企业系统日常运营至关重要的强大批处理应用程序。
>支持事务管理，提供了基于Web的管理接口。
>
>###### 7.Spring Security
>
>Spring Security是用于实现认证和授权，以及访问控制的安全框架，在Java生态与之提供类似的功能还有一个框架：Apache Shiro。
>Spring Security依赖于Spring Framework，也就是说如果要Spring Security，那么应用架构也必须是基于Spring Framework的，这大大限制了Spring Security的使用场景；反之，Shiro就没有这样限制，而且从项目架构上Shiro更加简洁。当然，Spring Security提供了非常丰富的安全控制的功能，在某些方面甚至比Shiro更加完善，与之对应的是掌握的Spring Security的复杂度比较大。因此，对于在应用中是否选择Spring Security需要根据实际需求来决定。
>
>###### 8.Spring HATEOAS
>
>如果Web应用基于Spring框架（即：使用了Spring MVC）开发，那么可以直接使用Spring HATEOAS来开发满足HATEOAS约束的RESTFul服务。
>这里需要理解一个单词简写：“HATEOAS”。HATEOAS（Hypermedia as the engine of application state）是REST架构风格中最复杂的约束，也是构建成熟REST服务的核心。它的重要性在于打破了客户端和服务器之间严格的契约，使得客户端可以更加智能和自适应，而 REST 服务本身的演化和更新也变得更加容易。
>
>###### 9.Spring REST Docs
>
>Spring REST Docs是一个文档工具，用于为REST架构风格的Web服务自动生成相应的文档，这样可以解放开发者专门撰写API文档的工作。
>
>###### 10.Spring AMQP
>
>Spring AMQP项目旨在将核心的Spring概念应用于基于AMQP的消息传递解决方案的开发中，它提供了一个“模板”的抽象用于发送和接收消息。
>
>###### 11.Spring Mobile
>
>Spring Mobile是对Spring MVC的扩展，旨在简化移动Web应用的开发。
>Spring Mobile可以检测出当前请求使用的设备是PC、还是手机或者是平板以及用户设备是安卓平台还是iOS平台，然后根据请求设备的不同，返回适合该设备的视图。
>
>###### 12.Spring For Android
>
>官方的说法LSpring For Android旨在简化原声Android应用的开发,Spring For Android提供了2个对原生Android应用开发的支持：
>(1)提供了一个REST客户端
>(2)支持访问安全API时的认证
>
>###### 13.Spring Web Flow
>
>Spring Web Flow主要应用于需要在Web页面上创建引导用户执行类似“下一步”这样的基于流程的应用场景，该框架构建于Spring MVC之上。
>
>###### 14.Spring Web Services
>
>Spring Web Services用于开发WebService服务，类似的框架如：Apache CXF，Apache Axis2。
>
>###### 15.Spring LDAP
>
>Spring LDAP是一个工具，用于为基于Spring的应用程序使用LDAP（Lightweight Directory Access Protocol）协议。
>
>###### 16.Spring Session
>
>Spring Session提供了管理用户Session信息的API和对应实现，Spring Session使得支持集群会话变得简单，而不依赖于特定于应用程序容器的解决方案。
>
>**and so on......**
>更多请看 https://spring.io/
