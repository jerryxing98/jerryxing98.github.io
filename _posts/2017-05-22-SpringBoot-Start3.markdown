
---
layout:     keynote
title:      "SpringBoot-最佳实践-1.3 Springfox原理解析"
subtitle:   "Slides: Spring父子容器及作用"
header-img: "img/post-bg-js-version.jpg"
navcolor:   "invert"
date:       2017-05-21
author:     "JerryMinds"
tags:
    - Java
    - SpringBoot
    - microservice
    - Springfox
    - 云端
---

> Swagger在同前端进行联调时，确实是个好东西,问题是在SAAS环境中面向公众的平台好像貌似没有人用吧

### Springfox简介

签于swagger的强大功能，java开源界大牛spring框架迅速跟上，它充分利用自已的优势，把swagger集成到自己的项目里，整了一个spring-swagger，后来便演变成springfox。springfox本身只是利用自身的aop的特点，通过plug的方式把swagger集成了进来，它本身对业务api的生成，还是依靠swagger来实现。


### Springfox大致原理
    在项目启动过程中,Spring项目在初始化的过程中会自动根据配置加载一些swagger相关的bean到当前的上下文当中,并自动扫描系统中可能需要生成api文档那些类，并生成相应的缓存,缓存起来。如果项目MVC控制层用的是springmvc那么会自动扫描所有的controller类，根据这些controller类中的方法生成相应的api文档

### 集成步骤

``` xml
<!-- sring mvc依赖 -->
      <dependency>
         <groupId>org.springframework</groupId
         <artifactId>spring-webmvc</artifactId>
      </dependency>
      <!-- swagger2核心依赖 -->
      <dependency>
         <groupId>io.springfox</groupId>
         <artifactId>springfox-swagger2</artifactId>
      </dependency>
      <!-- swagger-ui为项目提供api展示及测试的界面 -->
      <dependency>
         <groupId>io.springfox</groupId>
         <artifactId>springfox-swagger-ui</artifactId>
      </dependency>
```
**三个依赖** 第一个是springmvc依赖,第二个为swagger依赖,第三个为ui依赖,重点说下swagger，其依赖结构见下图

![img](/img/in-post/post_springfox_01.png)


### springfox的简单使用
如果只用springfox的默认的配置的话，与springmvc集成起来非常简单，只要写一个类似于以下代码的类放到你的项目里就行了，代码如下:

''' java
@Configuration
@EnableWebMvc
@EnableSwagger2
public class ApiConfig {
}
'''

简单介绍这三个注解
**@Configuration**是Spring框架中本身就有的,被**@Component**元注解标识的注解,这个注解自动将Bean注入到Spring的上下文中

**@EnableWebMvc** 启用Springmvc,其是通过元注解**@Import(DelegatingWebMvcConfiguration.class)**,其是向Springcontext中塞入了DelegatingWebMvcConfiguration类型的bean。

**@EnableSwagger2**看这个名字，用来集成Swagger2,通过元注解**@Import({Swagger2DocumentationConfiguration.class})**


Swagger2DocumentationConfiguration.class的核心配置如下：
``` java
@Configuration
@Import({ SpringfoxWebMvcConfiguration.class, SwaggerCommonConfiguration.class })
@ComponentScan(basePackages = {
 "springfox.documentation.swagger2.readers.parameter",
    "springfox.documentation.swagger2.web",
    "springfox.documentation.swagger2.mappers"
})
public class Swagger2DocumentationConfiguration {
  @Bean
  public JacksonModuleRegistrar swagger2Module() {
    return new Swagger2JacksonModule();
  }
}

```
这个类做了事 
@EnablePluginRegistries 中加入了很多插件，
其中ApiListingBuilderPlugin ，有两个实现类ApiListingReader，SwaggerApiListingReader。其中ApiListingReader会自动跟据Controller类型生成api列表，而SwaggerApiListingReader会跟据有@Api注解标识的类生成api列表。

@ComponentScan中的DocumentationPluginsManager和DocumentationPluginsBootstrapper。对于第一个DocumentationPluginsManager，它是一个没有实现任何接口的bean,管理所有的Plugin.

DocumentationPluginsBootstrapper 是实现了SmartLifecycle的接口。SmartLifecycle这个组件被实例化为一个bean纳入spring context中被管理起来的时候，会自动调用start()方法.
在这个里面的scanDocumentation(buildContext(each));  会缓存起来，DocumentationCache。

