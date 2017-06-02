
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



### 总结一下
   以上就是，srpingMvc整合springfox的大致原理。它主要是通过EnableSwagger2注解，向srping context注入了一系列bean，并在系统启动的时候自动扫描系统的Controller类，生成相应的api信息并缓存起来。此外，它还注入了一些被@Controller注解标识的Controller类，作为ui模块访问api列表的入口。比如springfox-swagger2-2.6.1.jar包中的Swagger2Controller类。这个Controller就是ui模块中用来访问api列表的界面地址。在访问http://127.0.0.1:8080/jadDemo/swagger-ui.html这个地址查看api列表时，通过浏览器抓包就可以看到，它是通过类似于http://127.0.0.1:8080/jadDemo/v2/api-docs?group=sysGroup这样的地址异步获得api信息（Json格式）并显示到界面上，这个地址后台对应的Controller入口就是上文的Swagger2Controller类，这个类收到请求后，直接从事先初始化好的缓存中的取出api信息生成json字符串返回。


### 坑1：配置类生成的bean必须与spring mvc共用同一个上下文
   前文描述了，在springmvc项目中，集成springfox是只要在项目写一个如下的没有任何业务代码的简单配置类就可以了。

``` java
@Configuration
@EnableWebMvc
@EnableSwagger2
public class ApiConfig {
}

```

> 因为@Configuration注解的作用，spring会自动把它实例化成一个bean注入到上下文。但切记要注意的一个坑就是：这个bean所在的上下文必须跟spring mvc为同一个上下文。怎么解理呢？因为在实际的spring mvc项目中，通常有两个上下文，一个是跟上下文，另一个是spring mvc（它是跟上下文的子上下文）。其中跟上下文是就是web.xml文件中跟spring相关的那个org.springframework.web.context.request.RequestContextListener监听器，加载起来的上下文，通常我们会写一个叫spring-contet.xml的配置文件，这里面的bean最终会初始化到跟上下文中，它主要包括系统里面的service,dao等bean，也包括数据源、事物等等。而另一个上下文是就是spring mvc了，它通过web.xml中跟spring mvc相关的那个org.springframework.web.servlet.DispatcherServlet加载起来，他通常有一个配置文件叫spring-mvc.xml。我们在写ApiConfig这个类时，如果决定用@Configuration注解来加载，那么就必须保证这个类所在的路径刚好在springmvc的component-scan的配置的base-package范围内。因为在ApiConfig在被spring加载时，会注入一列系列的bean，而这些bean中，为了能自动扫描出所有Controller类，有些bean需要依赖于SpringMvc中的一些bean，如果项目把Srpingmvc的上下文与跟上下文分开来，作为跟上下文的子上下文的话。如果不小心让这个ApiConfig类型的bean被跟上文加载到，因为root context中没有spring mvc的context中的那些配置类时就会报错。

> 实事上，我并不赞成通过@Configuration注解来配置Swagger，因为我认为，Swagger的api功能对于生产项目来说是可有可无的。我们Swagger往往是用于测试环境供项目前端团队开发或供别的系统作接口集成使上。系统上线后，很可能在生产系统上隐藏这些api列表。 但如果配置是通过@Configuration注解写死在java代码里的话，那么上线的时候想去掉这个功能的时候，那就尴尬了，不得不修改java代码重新编译。基于此，我推荐的一个方法，通过spring最传统的xml文件配置方式。具体做法就是去掉@Configuration注解，然后它写一个类似于<bean class="com.jad.web.mvc.swagger.conf.ApiConfig"/ >这样的bean配置到spring的xml配置文件中。在root context与mvc的context分开的项目中，直接配置到spring-mvc.xml中，这样就保证了它跟springmvc 的context一定处于同一个context中。


### 坑2：Controller类的参数，注意防止出现无限递归的情况

Spring mvc有强大的参数绑定机制，可以自动把请求参数绑定为一个自定义的命令对像。所以，很多开发人员在写Controller时，为了偷懒，直接把一个实体对像作为Controller方法的一个参数。比如下面这个示例代码：

``` java 
@RequestMapping(value = "update")
public String update(MenuVo menuVo, Model model){
}
```
> 这是大部分程序员喜欢在Controller中写的修改某个实体的代码。在跟swagger集成的时候，这里有一个大坑。如果MenuVo这个类中所有的属性都是基本类型，那还好，不会出什么问题。但如果这个类里面有一些其它的自定义类型的属性，而且这个属性又直接或间接的存在它自身类型的属性，那就会出问题。例如：假如MenuVo这个类是菜单类，在这个类时又含有MenuVo类型的一个属性parent代表它的父级菜单。这样的话，系统启动时swagger模块就因无法加载这个api而直接报错。报错的原因就是，在加载这个方法的过程中会解析这个update方法的参数，发现参数MenuVo不是简单类型，则会自动以递归的方式解释它所有的类属性。这样就很容易陷入无限递归的死循环。

> 为了解决这个问题，我目前只是自己写了一个OperationParameterReader插件实现类以及它依赖的ModelAttributeParameterExpander工具类，通过配置的方式替换掉到srpingfox原来的那两个类，偷梁换柱般的把参数解析这个逻辑替换掉，并避开无限递归。当然，这相当于是一种修改源码级别的方式。我目前还没有找到解决这个问题的更完美的方法，所以，只能建议大家在用spring-fox Swagger的时候尽量避免这种无限递归的情况。毕竟，这不符合springmvc命令对像的规范，springmvc参数的命令对像中最好只含有简单的基本类型属性。


### 坑3:api分组相关，Docket实例不能延迟加载
springfox默认会把所有api分成一组，这样通过类似于http://127.0.0.1:8080/jadDemo/swagger-ui.html这样的地址访问时，会在同一个页面里加载所有api列表。这样，如果系统稍大一点，api稍微多一点，页面就会出现假死的情况，所以很有必要对api进行分组。api分组，是通过在ApiConf这个配置文件中，通过@Bean注解定义一些Docket实例，网上常见的配置如下:

``` java
@EnableWebMvc
@EnableSwagger2
public class ApiConfig {
@Bean
 public Docket customDocket() {
       return new Docket(DocumentationType.SWAGGER_2).apiInfo(apiInfo());
    }
}
```

> 上述代码中通过@Bean注入一个Docket，这个配置并不是必须的，如果没有这个配置，框架会自己生成一个默认的Docket实例。这个Docket实例的作用就是指定所有它能管理的api的公共信息，比如api版本、作者等等基本信息，以及指定只列出哪些api（通过api地址或注解过滤）。

> Docket实例可以有多个，比如如下代码：

``` java
@EnableWebMvc
@EnableSwagger2
public class ApiConfig {
@Bean
 public Docket customDocket1() {
       return new Docket(DocumentationType.SWAGGER_2)
    .groupName("apiGroup1").apiInfo(apiInfo()).select()
    .paths(PathSelectors.ant("/sys/**"));
    }
@Bean
 public Docket customDocket2() {
       return new Docket(DocumentationType.SWAGGER_2)
    .groupName("apiGroup2").apiInfo(apiInfo())
    .select()
    .paths(PathSelectors.ant("/shop/**"));
    }
}
```


>   当在项目中配置了多个Docket实例时，也就可以对api进行分组了，比如上面代码将api分为了两组。在这种情况下，必须给每一组指定一个不同的名称，比如上面代码中的"apiGroup1"和"apiGroup2"，每一组可以用paths通过ant风格的地址表达式来指定哪一组管理哪些api。比如上面配置中，第一组管理地址为/sys/开头的api第二组管理/shop/开头的api。当然，还有很多其它的过滤方式，比如跟据类注解、方法注解、地址正则表达式等等。分组后，在api 列表界面右上角的下拉选项中就可以选择不同的api组。这样就把项目的api列表分散到不同的页面了。这样，即方便管理，又不致于页面因需要加载太多api而假死。

>   然而，同使用@Configuration一样，我并不赞成使用@Bean来配置Docket实例给api分组。因为这样，同样会把代码写死。所以，我推荐在xml文件中自己配置Docket实例实现这些类似的功能。当然，考虑到Docket中的众多属性，直接配置bean比较麻烦，可以自己为Docket写一个FactoryBean，然后在xml文件中配置FactoryBean就行了。然而将Docket配置到xml中时。又会遇到一个大坑，就那是，spring对bean的加载方式默认是延迟加载的，在xml中直接配置这些Docket实例Bean后。你会发现，没有一点效果，页面左上角的下拉列表中跟本没有你的分组项。


>   这个问题曾困扰过我好几个小时，后来凭经验推测出可能是因为sping bean默认延迟加载，这个Docket实例还没加载到spring context中。实事证明，我的猜测是对的。我不知道这算是springfox的一个bug，还是因为我跟本不该把对Docket的配置从原来的java代码中搬到xml配置文件中来。


### 其他坑，
springfox还有些其它的坑，比如@ApiOperation注解中，如果不指定httpMethod属性具体为某个get或post方法时，api列表中，会它get,post,delete,put等所有方法都列出来，搞到api列表重复的太多，很难看。另外，还有在测试时，遇到登录权限问题，等等。这一堆堆的比较容易解决的小坑，因为篇幅有限，我就不多说了。还有比如@Api、@ApiOperation及@ApiParam等等注解的用法，网上很多这方面的文档，我就不重复了。