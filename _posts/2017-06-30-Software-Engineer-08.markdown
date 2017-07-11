---
layout:     post
title:      "软件工程系列-工程模块"
subtitle:   ""
date:       2017-06-02 12:00:00
author:     "JerryMinds"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 云端
    - 软件工程
    - 日志
    - 调度
---


#### 日志处理
### 异常分类
* 异常的继承结构 ,基类为Throwable,Error和Exception继承Throwable，RuntimeException和IOException等继承Exception，具体的RuntimeException继承RuntimeException
Error和RuntimeException及其子类成为未检查异常（unchecked），其它异常成为已检查异常（checked）。

* Error体系 ：
Error类体系描述了Java运行系统中的内部错误以及资源耗尽的情形。应用程序不应该抛出这种类型的对象（一般是由虚拟机抛出）。如果出现这种错误，除了尽力使程序安全退出外，在其他方面是无能为力的。所以，在进行程序设计时，应该更关注Exception体系。




* 异常处理的流程：
① 遇到错误，方法立即结束，并不返回一个值；同时，抛出一个异常对象 。
② 调用该方法的程序也不会继续执行下去，而是搜索一个可以处理该异常的异常处理器，并执行其中的代码 。


* SpringMVC的异常处理
HandlerExceptionResolver 
 detectAllHandlerExceptionResolvers

Step1:
Spring MVC在找到你的handler之后，会通过反射调用handler的方法：如果Handler方法抛异常，会被catch住，然后执行相应的异常处理逻辑。
handler的异常会在processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);被处理。
mv = processHandlerException(request, response, handler, exception); 这个方法会处理异常，并且返回一个error ModelAndView。如果mv不为空，还会渲染这个mv。


Step2:
HandlerExceptionResolver
然后，调用每个HandlerExceptionResolver的resolveException方法。如果该方法返回的不是null，表示已经处理结束。否则表示交个下一次异常处理器处理。

可以看到默认会查找Spring上下文中所有实现了HandlerExceptionResolver接口的类:

HandlerExceptionResolver
AbstractHandlerExceptionResolver
AbstractHandlerMethodExceptionResolver
ExceptionHandlerExceptionResolver
AnnotationMethodHandlerExceptionResolver：默认开启，配合@ExceptionHandler注解。@deprecated in favor of ExceptionHandlerExceptionResolver
DefaultHandlerExceptionResolver：默认开启，处理标准的Spring异常并且将他们转换为相应的HTTP状态码。
ResponseStatusExceptionResolver：默认开启，配合@ResponseStatus注解将异常映射为HTTP状态码。
SimpleMappingExceptionResolver
HandlerExceptionResolverComposite
经过debug发现，Spring会找到如下异常处理器：

org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver
org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver
org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver
me.arganzheng.study.springmvc.common.RestHandlerExceptionResolver

特别需要注意的是HandlerExceptionResolver是有顺序的（实现了Ordered接口），如果没有指定，业务自定义的HandlerExceptionResolver会排在最后。

lab.s2jh.core.web.exception.AnnotationHandlerMethodExceptionResolver，全局统一拦截处理功能异常；

其中注入contentNegotiationManager，判断根据不同请求类型构造对应的数据格式响应，如JSON或JSP页面；

根据不同异常类型，做一定的错误消息友好转义处理，区分控制不同异常是否需要进行logger日志记录；

logger记录时，生成全局唯一的错误编号并合理的传递到前端界面显示以便用户反馈，把相关请求数据基于MDC方式记录下来，以便问题排查；

Step3:
Step4: 


### 捕获错误日志并持久化
* 定义错误码 根据具体业务进行错误码的定义，分类越细阅好方便排查
* 错误日志分类 运行日志，数据库日志，服务日志
* 异步持久化，典型的需要持久化的属性有，错误码，抛出来的class,操作的用户,错误栈,修复标签,创建时间,修改时间，模块负责人 等


### 日志告警并进行修复标识
* 将错误日志告警发送给相关责任人
* 跟踪相关修复细节
* 对错误日志进行按 天，周粒度拿出来进行Review ，找出潜在的系统风险



### 用户端优化

##错误信息分类
* 成功消息 - 成功消息
    使用前端一般是绿色的短暂气泡提示
* 告警消息 - 实例：无操作权限
    偶尔用于标识业务处理基本完成，但是其中存在一些需要注意放在message或data中的提示信息。前端一般是黄色的气泡提示
* 失败消息 - 
    操作处理失败。前端一般是红色的长时间或需要用户主动关闭的气泡提示
* 确认    - 高危操作
    本次提交中止，反馈用户进行确认。前端一般会弹出一个供用户'确认'操作的对话框，然后用户主动确认之后会自动再次发起请求并跳过确认检查进行后续业务处理


