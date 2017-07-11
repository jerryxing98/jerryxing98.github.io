---
layout:     post
title:      "软件工程系列-质量控制"
subtitle:   ""
date:       2017-06-02 12:00:00
author:     "JerryMinds"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 云端
    - 软件工程
    - 质量控制
---



#### 依赖管理
用maven的依赖插件的分析结果来帮助你识别项目中的依赖管理问题

maven依赖插件（Maven Dependency Plugin）的analyze 目标可以识别一个项目中关于依赖的两类问题：
* 直接就使用但未声明的依赖；（项目可以编译因为它也得到了所需的依赖）



有如下的假定:
```
A项目使用了来自B和C项目的类；
A只声明了一个B项目的依赖；
B项目也使用了C项目的类；
B项目声明了以一个C项目的依赖。
```
这种情况存在一个潜在的问题，因为A依赖B，B依赖C，A项目只有在B项目移除了对C项目的引用后才能编译。如果是SNAPSHOT版本的话问题将会更加严重。

当对A项目用mvn dependency:analyze进行分析时，maven将会有如下输出：
```
[WARNING] Used undeclared dependencies found:
[WARNING] com.avaya.ace.aaft:foundation_services_client_api:jar:6.2.0-SNAPSHOT:compile
```
A项目直接声明一个对C项目的引用就会解决这个问题。



* 声明了但没有使用的依赖
如果一个项目声明了一个依赖，但没有使用它，mvn dependency:analyze会有如下输出：
```
[WARNING] Unused declared dependencies found:
[WARNING] com.gigaspaces:gs-openspaces:jar:7.1.1-b4534:compile
```
在POM文件移除这个依赖声明就可以解决这个问题。
未使用的依赖一般是开发者没有搞清楚maven dependenceManagement的原理造成的。


* 不要写代码复制解压文件
使用maven标准的插件来替代。

* 良好的mvn依赖代码规范和分层
分为这几个层的依赖 
1.Common 通用的依赖 Log,Json,工具类
2.Web 类的依赖 前端模块依赖，API工具等 
3.Model层的依赖 Lombok之类的依赖
4.DAO层的依赖，同数据访问相关的依赖，数据库依赖包
5.Service层依赖 ，中间件的依赖，Spring等
6.业务第三包


#### CodeStyle 

#### Enforcer插件

#### Google AutoValue
最近这些年函数式编程的思维开始流行起来，很多非函数式的语言也开始在语言层面支持某些函数式的特征，比如Java 8里面的lambda, stream等等，Google AutoValue也是类似的目的，它的目的是让你的POJO编程readonly的(只有Getter), 从而实现函数式语言里面Immutable的特性。

它最大的贡献在于，它让你只需要定义你需要的字段:

```
import com.google.auto.value.AutoValue;

@AutoValue
abstract class Animal {
  static Animal create(String name, int numberOfLegs) {
    // See "How do I...?" below for nested classes.
    return new AutoValue_Animal(name, numberOfLegs);
  }

  abstract String name();
  abstract int numberOfLegs();
}
```


这里我们定义了我们需要两个字段: name和numberOfLegs以及一个用来构建Animal对象的create方法，其它的则都由AutoValue自动生成，其中AutoValue_Animal就是AutoValue自动生成的类。在AutoValue_Animal里面，它帮我们自动实现了hashCode, equals, toString等等这些重要的方法。这些方法的特征是实现的过程基本都一样，但是容易出错(由于粗心), 那么不如交给框架去自动产生。

如果你的类字段比较多，那么AutoValue还支持Builder模式:

```
import com.google.auto.value.AutoValue;

@AutoValue
abstract class Animal {
  abstract String name();
  abstract int numberOfLegs();

  static Builder builder() {
    return new AutoValue_Animal.Builder();
  }

  @AutoValue.Builder
  abstract static class Builder {
    abstract Builder setName(String value);
    abstract Builder setNumberOfLegs(int value);
    abstract Animal build();
  }
}
```
这样我们就可以一步一步渐进地把对象构造出来了。


#### Api Surface Test
所谓的API Surface是指我们一个系统暴露给另外一个系统一个SDK的时候，我们到底应该把哪些类，哪些package暴露给用户，这个问题很重要，因为考虑向后兼容性的话，暴露的package越少，将来修改的时候，破坏向后兼容性的可能性就越小，SDK就越稳定。我们平常的时候对于这种事情可能都是通过人肉分析、review，在Beam里面它直接编写成了一个单元测试:

```
  @Test
  public void testSdkApiSurface() throws Exception {

    @SuppressWarnings("unchecked")
    final Set<String> allowed =
        ImmutableSet.of(
            "org.apache.beam",
            "com.fasterxml.jackson.annotation",
            "com.fasterxml.jackson.core",
            "com.fasterxml.jackson.databind",
            "org.apache.avro",
            "org.hamcrest",
            // via DataflowMatchers
            "org.codehaus.jackson",
            // via Avro
            "org.joda.time",
            "org.junit");

    assertThat(
        ApiSurface.getSdkApiSurface(getClass().getClassLoader()), containsOnlyPackages(allowed));
  }
```
这个测试表明，Beam的SDK向外暴露的package就是上面列出的这些，下次来个新手贡献的代码如果破坏了这个约定，那么这个单元测试就直接报错，无法提交merge。还是那句话，凡是能自动化检测的东西不要依靠人肉。这里涉及到的主要技术是:

通过扫描classpath对指定包里面所有类的方法，参数，返回值进行检查。
hamcrest这个支持matcher的单元测试的库，它让你不只可以assertTrue, assertFalse, assertEquals, 而是可以自由指定需要满足的条件