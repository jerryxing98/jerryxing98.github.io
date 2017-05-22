---
layout:     keynote
title:      "SpringSecurity-Api安全管理"
subtitle:   "Slides: SpringMVC-Api安全管理"
header-img: "img/post-bg-js-version.jpg"
navcolor:   "invert"
date:       2017-05-21
author:     "JerryMinds"
tags:
    - Java
    - SpringSecurity
    - microservice
    - 云端
---

> 通常情况下，把API直接暴露出去是风险很大的，不说别的，直接被机器攻击就喝一壶的。那么一般来说，对API要划分出一定的权限级别，然后做一个用户的鉴权，依据鉴权结果给予用户开放对应的API。目前，比较主流的方案有几种:


* 用户名和密码鉴权，使用Session保存用户鉴权结果(不适合移动时代)
* 使用OAuth进行鉴权（其实OAuth也是一种基于Token的鉴权，只是没有规定Token的生成方式）-（不做开放平台,太过复杂）
* 自行采用Token进行鉴权 （内部产品比较合适）


---
### 解决方案（JWT+SpringSecurity）

#### JWT介绍
一般的JWT的工作流程
1 用户导航到登录页，输入用户名、密码，进行登录
2 服务器验证登录鉴权，如果改用户合法，根据用户的信息和服务器的规则生成JWT Token
3 服务器将该token以json形式返回（不一定要json形式，这里说的是一种常见的做法）
4 用户得到token，存在localStorage、cookie或其它数据存储形式中。
5 以后用户请求/protected中的API时，在请求的header中加入 Authorization: Bearer xxxx(token)。   此处注意token之前有一个7字符长度的 Bearer
6 服务器端对此token进行检验，如果合法就解析其中内容，根据其拥有的权限和自己的业务逻辑给出对应的响应结果。
7 用户取得结果

![img](/img/in-post/post-springboot/post-springboot-security-01.png)

为了更好的理解这个token是什么，我们先来看一个token生成后的样子，下面那坨乱糟糟的就是了。

```
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ3YW5nIiwiY3JlYXRlZCI6MTQ4OTA3OTk4MTM5MywiZXhwIjoxNDg5Njg0NzgxfQ.RC-BYCe_UZ2URtWddUpWXIp4NMsoeq2O6UF-8tVplqXY1-CI9u1-a-9DAAJGfNWkHE81mpnR3gXzfrBAB3WUAg

```

但仔细看到的话还是可以看到这个token分成了三部分，每部分用 . 分隔，每段都是用 Base64 编码的。如果我们用一个**Base64**的解码器的话 （ https://www.base64decode.org/ ），可以看到第一部分 **eyJhbGciOiJIUzUxMiJ9** 被解析成了:

```
{
    "alg":"HS512"
}
```
这是告诉我们HMAC采用HS512算法对JWT进行的签名。


第二部分 **eyJzdWIiOiJ3YW5nIiwiY3JlYXRlZCI6MTQ4OTA3OTk4MTM5MywiZXhwIjoxNDg5Njg0NzgxfQ** 被解码之后是

```
{
    "sub":"wang",
    "created":1489079981393,
    "exp":1489684781
}
```
这段告诉我们这个Token中含有的数据声明（Claim），这个例子里面有三个声明：**sub**, **created** 和 **exp**。在我们这个例子中，分别代表着用户名、创建时间和过期时间，当然你可以把任意数据声明在这里。


看到这里，你可能会想这是个什么鬼token，所有信息都透明啊，安全怎么保障？别急，我们看看token的第三段 RC-BYCe_UZ2URtWddUpWXIp4NMsoeq2O6UF-8tVplqXY1-CI9u1-a-9DAAJGfNWkHE81mpnR3gXzfrBAB3WUAg。同样使用Base64解码之后，咦，这是什么东东

```
D X    DmYTeȧLUZcPZ0$gZAY_7wY@
```

头中的数据通常包含两部分：一个是我们刚刚看到的 alg，这个词是 algorithm 的缩写，就是指明算法。另一个可以添加的字段是token的类型(按RFC 7519实现的token机制不只JWT一种)，但如果我们采用的是JWT的话，指定这个就多余了。

```
{
  "alg": "HS512",
  "typ": "JWT"
}
```

payload中可以放置三类数据：系统保留的、公共的和私有的：

* 系统保留的声明（Reserved claims）：这类声明不是必须的，但是是建议使用的，包括：iss (签发者), exp (过期时间),
sub (主题), aud (目标受众)等。这里我们发现都用的缩写的三个字符，这是由于JWT的目标就是尽可能小巧。

* 公共声明：这类声明需要在 IANA JSON Web Token Registry 中定义或者提供一个URI，因为要避免重名等冲突。
* 私有声明：这个就是你根据业务需要自己定义的数据了。

签名的过程是这样的：采用header中声明的算法，接受三个参数：base64编码的header、base64编码的payload和秘钥（secret）进行运算。签名这一部分如果你愿意的话，可以采用RSASHA256的方式进行公钥、私钥对的方式进行，如果安全性要求的高的话。


```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

#### SpringSecurity介绍
#### 集成及测试

