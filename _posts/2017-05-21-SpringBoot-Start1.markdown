---
layout:     keynote
title:      "SpringBoot-最佳实践-1.1迁移旧应用"
subtitle:   "Slides: Springmvc迁移至SpringBoot"
header-img: "img/post-bg-js-version.jpg"
navcolor:   "invert"
date:       2017-05-21
author:     "JerryMinds"
tags:
    - Java
    - SpringBoot
    - microservice
    - 云端
---



> 对于一个创业项目或者一个类似的项目如何快速启动？架构新项目 or 迁移旧的应用

---
### 前言

手上有个迭代了几年的项目,是迁移旧的应用还是架构新项目呢？我相信很多人都会选择最大限度的兼容旧项目。

---
### 几个步骤
* I.迁移web.xml文件
* II.兼容DispacheServlet及配置文件
* III.兼容静态文件

#### 迁移web.xml文件 - Filter迁移,Lister迁移,

spring boot提供了 ServletRegistrationBean，FilterRegistrationBean，ServletListenerRegistrationBean这3个东西来进行配置Servlet、Filter、Listener

编写配置类WebConfig.java

```java
import java.util.ArrayList;
import java.util.EventListener;
import java.util.List;
 
import org.springframework.boot.context.embedded.FilterRegistrationBean;
import org.springframework.boot.context.embedded.ServletListenerRegistrationBean;
import org.springframework.boot.context.embedded.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
 
import com.tianshouzhi.springbootstudy.web.filter.DemoFilter;
import com.tianshouzhi.springbootstudy.web.listener.DemoListener;
import com.tianshouzhi.springbootstudy.web.servlet.DemoServlet;
 
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter{
    @Bean
    public FilterRegistrationBean getDemoFilter(){
        DemoFilter demoFilter=new DemoFilter();
        FilterRegistrationBean registrationBean=new FilterRegistrationBean();
        registrationBean.setFilter(demoFilter);
        List<String> urlPatterns=new ArrayList<String>();
        urlPatterns.add("/*");//拦截路径，可以添加多个
        registrationBean.setUrlPatterns(urlPatterns);
        registrationBean.setOrder(1);
        return registrationBean;
    }
    @Bean
    public ServletRegistrationBean getDemoServlet(){
        DemoServlet demoServlet=new DemoServlet();
        ServletRegistrationBean registrationBean=new ServletRegistrationBean();
        registrationBean.setServlet(demoServlet);
        List<String> urlMappings=new ArrayList<String>();
        urlMappings.add("/demoservlet");////访问，可以添加多个
        registrationBean.setUrlMappings(urlMappings);
        registrationBean.setLoadOnStartup(1);
        return registrationBean;
    }
    @Bean
    public ServletListenerRegistrationBean<EventListener> getDemoListener(){
        ServletListenerRegistrationBean<EventListener> registrationBean
                                   =new ServletListenerRegistrationBean<>();
        registrationBean.setListener(new DemoListener());
//      registrationBean.setOrder(1);
        return registrationBean;
    }
}
```

#### 兼容DispacheServlet及配置文件 - 取消AutoConfig,进行手动load



#### 兼容静态文件 -- SpringMVC的静态文件加载

1.SpringMVC静态文件的访问 - 交给Servlet
Spring MVC 的入口是 DispatcherServlet，所有的请求都会汇集于该类，而后分发给不同的处理类。如果不做额外的配置，是无法访问静态资源的。
![img](/img/in-post/post-springboot/post-springboot-static-resource-01.png)

如果想让 Dispatcher Servlet 直接可以访问到静态资源，最简单的方法当然是交给默认的 Servlet。

![img](/img/in-post/post-springboot/post-springboot-static-resource-02.png)

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}

```
这种情况下 Spring MVC 对资源的处理与 Servlet 方式相同。

2.SpringMVC - 自定义配置

我们可以通过很简单的配置使得 Spring MVC 有能力处理对静态资源进行处理。
在 Spring MVC 中，资源的查找、处理使用的是责任链设计模式（Filter Chain）：

![img](/img/in-post/post-springboot/post-springboot-static-resource-03.png)

其思路为如果当前 resolver 找不到资源，则转交给下一个 resolver 处理。 当前 resolver 找到资源则立即返回给上级 resovler（如果存在），此时上级 resolver 又可以选择对资源进一步处理或再次返回给它的上级（如果存在）。

配置方法为重写 WebMvcConfigurerAdapter 类的 addResourceHandlers

```java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {
    @Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/webjars/**")
                .addResourceLocations(
                        "classpath:/META-INF/resources/webjars/");
}
```

通过这样的配置，就成功添加了一个 **PathResourceResolver**。
![img](/img/in-post/post-springboot/post-springboot-static-resource-04.png)

该 resolver 的作用是将 url 为 /webjars/** 的请求映射到 **classpath:/META-INF/resources/webjars/ **。


3.SpringMVC - 进阶
* 为静态资源添加版本号
* gzip 压缩
* chain cache
* 省略 webjar 版本
* Transformer
* Http 缓存

4.使用SpringBoot默认配置