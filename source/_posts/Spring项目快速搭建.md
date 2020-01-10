---
title: Spring项目快速搭建
date: 2020-01-10 11:16:16
tags:
---
#### 一、Maven

##### Maven简介
>Apache Maven是一个软件项目管理工具。基于项目对象模型的概念，maven可用来管理项目的依赖、编译、文档等信息。
>使用Maven管理项目时，项目依赖的jar包将不再包含在项目内，而是集中在用户目录下的.m2文件夹下


##### Maven安装

详情见https://maven.apache.org/download.cgi

##### pom.xml
Maven项目都有一个pom.xml用来管理项目的依赖以及项目的编译等功能

**pom文件的一些重要元素**

| 元素 | 作用 |
| --- | --- |
| dependencies | <dependencies></dependencies>,此元素包含多个项目依赖需要使用的<dependency> |
| dependency | <dependency></dependency>通过groupId,artifactId,version来确定唯一的依赖。（groupId: 组织的唯一标识，artifactId:项目的唯一标识,version:版本号）|
| 变量定义 | <properties></properties>可定义变量并在dependency中引用 |
| 编译插件 | <build></build> |

##### Maven运作方式

Maven会根据dependency中的依赖配置，直接通过互联网在Maven中心库下载相关依赖包到.m2文件夹下。

#### 二、Spring项目搭建

##### 推荐方式
https://start.spring.io/

##### 手动方式
基于IntelliJ Idea搭建
1. File -> New -> Project -> Maven -> next
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011161911635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
2. 填写groupId,artifactId,version
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011161930711.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
3. 选择项目路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019101116194367.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
4. 在pom文件中添加自己需要的依赖
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011161959160.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NhY3R1c19Mcmc=,size_16,color_FFFFFF,t_70)
5. 开始自己的showtime

