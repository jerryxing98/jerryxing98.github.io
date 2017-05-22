---
layout:     keynote
title:      "SpringBoot-最佳实践-1.2Spring父子容器"
subtitle:   "Slides: Spring父子容器及作用"
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


> 为啥Spring和SpringMVC包要分开扫描包？


---
### 背景

经常在项目看到这样的代码,为什么一个Bean 要注入两次？
``` xml
<!-- spring 配置文件-->  
<context:component-scan base-package="com.xxx.xxx.account.front">  
     <context:exclude-filter type="annotation"   
         expression="org.springframework.stereotype.Controller" />  
</context:component-scan>  
  
<!-- spring mvc -->     
<context:component-scan base-package="com.xxx.xxx.account.front.web" use-default-filters="false">  
    <context:include-filter type="annotation"   
        expression="org.springframework.stereotype.Controller" />  
</context:component-scan>  

```

测试Bean
``` java
@Service  
public class TestService implements InitializingBean {   
  
    @Autowired  
    private PersonalAddressAjaxController personalAddressAjaxController;  
  
    @Override  
    public void afterPropertiesSet() throws Exception {  
        System.out.println("--------------------------------");  
    }  
}  
```



---
### 相关测试
* I.测试1 - Spring加载全部bean,MVC加载Controller
``` xml
<!-- spring 配置文件-->  
<context:component-scan base-package="com.xxx.xxx.account.front">  
</context:component-scan>  
  
<!-- spring mvc -->     
<context:component-scan base-package="com.xxx.xxx.account.front.web" use-default-filters="false">  
    <context:include-filter type="annotation"   
        expression="org.springframework.stereotype.Controller" />  
</context:component-scan>  
```

   测试结果：TestService通过，界面显示正常。
    原因：父容器加载了全部bean,所以Service 能访问到Controller。MVC容器默认查找当前容器，能查到有转发的Controller规则所以界面正常跳转。



* II.测试2 - Spring加载全部Bean,MVC容器啥也不加载
``` xml
<!-- spring 配置文件-->  
<context:component-scan base-package="com.xxx.xxx.account.front">  
</context:component-scan>  
<!-- spring mvc -->   
```

  测试结果：TestService通过，界面显示404。
    原因：父容器加载了全部bean,所以Service 能访问到Controller。MVC容器默认查找当前容器的Controller，找不到所以界面出现404。


* III.测试3 -  Spring加载所有除了Controller的bean，MVC只加载Controller

``` xml
<!-- spring 配置文件-->  
<context:component-scan base-package="com.xxx.xxx.account.front">  
     <context:exclude-filter type="annotation"   
         expression="org.springframework.stereotype.Controller" />  
</context:component-scan>  
  
<!-- spring mvc -->     
<context:component-scan base-package="com.xxx.xxx.account.front.web" use-default-filters="false">  
    <context:include-filter type="annotation"   
        expression="org.springframework.stereotype.Controller" />  
</context:component-scan>  
```

    测试结果：TestService初始化失败，如果注释掉该bean，界面正常。
    原因：父容器不能访问子容器的bean。


* IIII.测试4 - Spring不加载bean，MVC加载所有的bean
``` xml
<!-- spring 配置文件-->  
无  
  
<!-- spring mvc -->     
<context:component-scan base-package="com.xxx.xxx.account.front.web" use-default-filters="true">  
</context:component-scan>  
```
    测试结果：TestService通过，界面正常。
    原因：因为所有的bean都在子容器中，也能查到当前容器中的Controller,所以没啥问题。



### 原理
Spring 是父容器， Spring MVC是子容器， 子容器可以访问父容器的bean,父容器不能访问子容器的bean。 


### 相关应用
通过HierarchicalBeanFactory接口，Spring的IoC容器可以建立父子层级关联的容器体系，子容器可以访问父容器中的Bean，但父容器不能访问子容器的Bean。在容器内，Bean的id必须是唯一的，但子容器可以拥有一个和父容器id相同的Bean。父子容器层级体系增强了Spring容器架构的扩展性和灵活性，因为第三方可以通过编程的方式，为一个已经存在的容器添加一个或多个特殊用途的子容器，以提供一些额外的功能。

Spring使用父子容器实现了很多功能，比如在Spring MVC中，展现层Bean位于一个子容器中，而业务层和持久层的Bean位于父容器中。这样，展现层Bean就可以引用业务层和持久层的Bean，而业务层和持久层的Bean则看不到展现层的Bean。