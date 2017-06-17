# 3. IoC容器

## 3.1 Spring IoC容器和beans的介绍

本章涵盖了Spring框架实现控制反转（IoC）[\[1\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#ftn.d5e685)的原理。IoC又叫依赖注入（DI）。它描述了对象的定义和依赖的一个过程，也就是说，依赖的对象通过构造参数、工厂方法参数或者属性注入，当对象实例化后依赖的对象才被创建，当创建bean后容器注入这些依赖对象。这个过程基本上是反向的，因此命名为控制反转（IoC），它通过直接使用构造类来控制实例化，或者定义它们之间的依赖关系，或者类似于服务定位模式的一种机制。

`org.springframework.beans`和`org.springframework.context`是Spring框架中IoC容器的基础，[`BeanFactory`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/beans/factory/BeanFactory.html)接口提供一种高级的配置机制能够管理任何类型的对象。\[`ApplicationContext`\]\(\[\[\[\[[http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\)\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\)\)\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\)\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\]\(http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html\)是\`BeanFactory\`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的\`WebApplicationContext。\`\)\)\)\](http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`%29]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`%29%29]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`%29]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`]%28http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html%29是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`%29%29%29\)\)

总之，`BeanFactory`提供了配置框架和基本方法，`ApplicationContext`添加更多的企业特定的功能。`ApplicationContext`是`BeanFactory`的一个子接口，在本章它被专门用于Spring的IoC容器描述。更多关于使用`BeanFactory`替代`ApplicationContext`的信息请参考[章节 3.16, “The BeanFactory”](http://ifeve.com/spring-ioc-1-2/beans.html#beans-beanfactory)。

在Spring中，由Spring IoC容器管理的对象叫做beans。 bean就是由Spring IoC容器实例化、组装和以其他方式管理的对象。此外bean只是你应用中许多对象中的一个。Beans以及他们之间的依赖关系是通过容器配置元数据反映出来。

## 3.2容器概述

`org.springframework.context.ApplicationContext`接口代表了Spring Ioc容器，它负责实例化、配置、组装之前的beans。容器通过读取配置元数据获取对象的实例化、配置和组装的描述信息。它配置的0元数据用xml、Java注解或Java代码表示。它允许你表示组成你应用的对象以及这些对象之间丰富的内部依赖关系。

Spring提供几个开箱即用的`ApplicationContext`接口的实现类。在独立应用程序中通常创建一个[`ClassPathXmlApplicationContext`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html)或[`FileSystemXmlApplicationContext`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html)实例对象。虽然XML是用于定义配置元数据的传统格式，你也可以指示容器使用Java注解或代码作为元数据格式，但要通过提供少量XML配置来声明启用对这些附加元数据格式的支持。

在大多数应用场景中，显示用户代码不需要实例化一个或多个Spring IoC容器的实例。比如在web应用场景中，在web.xml中简单的8行（或多点）样板式的xml配置文件就可以搞定（参见第3.15.4节“Web应用程序的便利的ApplicationContext实例化”）。如果你正在使用Eclipse开发环境中的[Spring Tool Suite](https://spring.io/tools/sts)插件，你只需要鼠标点点或者键盘敲敲就能轻松搞定这几行配置。

下图是Spring如何工作的高级展示。你应用中所有的类都由元数据组装到一起  
所以当`ApplicationContext`创建和实例化后，你就有了一个完全可配置和可执行的系统或应用。

_Figure 5.1. Spring IoC容器_

![](/assets/container-magic.png)

### 3.2.1 配置元数据

如上图所示，Spring IoC容器使用了一种_配置元数据_的形式。此配置元数据表示应用程序的开发人员告诉Spring容器怎样去实例化、配置和装备你应用中的对象。

配置元数据传统上以简单直观的XML格式提供，本章大部分都使用这种格式来表达Spring IoC容器核心概念和特性。

> 基于XML的元数据不是允许配置元数据的唯一形式，Spring IoC容器与实际写入配置元数据的格式是分离的。这些天许多的开发者在他们的Spring应用中选择基于[Java配置](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-java)。

更多关于Spring容器使用其他形式的元数据信息，请查看：

* [基于注解配置：](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-annotation-config)
  在Spring2.5中有过介绍支持基于注解的配置元数据
* [基于Java配置：](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-java)
  从Spring3.0开始，由Spring JavaConfig提供的许多功能已经成为Spring框架中的核心部分。这样你可以使用Java程序而不是XML文件定义外部应用程序中的bean类。使用这些新功能，可以查看`@Configuration`,`@Bean`,`@Import`和`@DependsOn`这些注解

Spring配置由必须容器管理的一个或通常多个定义好的bean组成。基于XML配置的元数据中，这些bean通过标签定义在顶级标签内部。在Java配置中通常在使用`@Configuration`注解的类中使用`@Bean`注解方法。

这些bean的定义所对应的实际对象就组成了你的应用。通常你会定义服务层对象，数据访问层对象\(DAO\)，展现层对象比如Struts的`Action`实例，底层对象比如Hibernate的`SessionFactories`，JMS`Queues`等等。通常在容器中不定义细粒度的域对象，因为一般是由DAO层或者业务逻辑处理层负责创建和加载这些域对象。但是，你可以使用Spring集成Aspectj来配置IoC容器管理之外所创建的对象。详情请查看[Spring使用AspectJ依赖注入域对象](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-atconfigurable)

接下来这个例子展示了基于XML配置元数据的基本结构

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- 在这里写 bean 的配置和相关引用 -->
    </bean>

    <bean id="..." class="...">
        <!-- 在这里写 bean 的配置和相关引用 -->
    </bean>

    <!-- 在这里配置更多的bean -->

</beans>
```

id属性用来使用标识每个独立的bean定义的字符串。`class`属性定义了bean的类型，这个类型必须使用全路径类名（必须是包路径+类名）。id属性值可以被依赖对象引用。该例中没有体现XML引用其他依赖对象。更多请查看[bean的依赖](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-dependencies)。

## 3.11 使用JSR 330标准注解 {#toc_1}

从Spring3.0开始，Spring提供了对JSR-330标准注解\(依赖注入\)的支持。这些注解和Spring的注解以相同的方式进行扫描。你只需要在你的classpath中添加有关的jar包。

> 如果你使用maven，javax.inject 存在标准的maven库中\([http://repo1.maven.org/maven2/javax/inject/javax.inject/1/](http://repo1.maven.org/maven2/javax/inject/javax.inject/1/)\)，你可以将下面的依赖加入到你的pom.xml:
>
> ```
> <dependency>
>     <groupId>javax.inject</groupId>
>     <artifactId>javax.inject</artifactId>
>     <version>1</version>
> </dependency>
> ```

### 3.11.1 使用@Inject and @Named依赖注入 {#toc_2}

和@Autowired不同的是，@javax.inject.Inject的使用如下：

```
import javax.inject.Inject;import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        ...
    }
}
```

和@Autowired一样，@Inject被用在字段级别、方法级别和构造器参数级别。更进一步的讲，你可以把注入点声明为一个提供者，允许通过Provider.get\(\)调用按需访问较短范围的bean或者延迟访问其他bean。上面例子的变形：

```
private Provider<MovieFinder> movieFinder;
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {


@Inject
public void setMovieFinder(Provider<MovieFinder> movieFinder) {
    this.movieFinder = movieFinder;
}

public void listMovies() {
    this.movieFinder.get().findMovies(...);
    ...
}
}
```

如果你需要使用限定名作为依赖的注入想，那么你应该使用@Named注解，如下所示：

```
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

### 3.11.2 和标准@Component注解等价@Named和@ManagedBean {#toc_3}

和@Component注解不同，@javax.inject.Named or javax.annotation.ManagedBean使用如下：

```
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener") // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

对于@Component而言不指定名称是非常常见的。@Named也可以使用类似的方式：

```
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

当使用@Named或@ManagedBean时，可以与使用Spring注解以完全相同的方式扫描组件：

```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig {
...
}
```

> 和@Component不同，the JSR-330 @Named注解和JSR-250 ManagedBean注解它们之间不能组合。请使用Spring的原型模式来构建自定义组件注解。

### 3.11.3 JSR-330标准注解的局限性 {#toc_4}

当你使用标准注解时，务必要知道一些重要的不可用的特性，如下表所示：

表3.6 Spring组件模型元素 vs JSR-330 变体

| Spring | javax.inject.\* | javax.inject restrictions / comments |
| :--- | :--- | :--- |
| @Autowired | @Inject | @Inject 没有’required’属性;可以使用Java 8 的Optional 代替 |
| @Component | @Named / @ManagedBean | JSR-330 不提供组合模型，仅仅是识别组件命名的途径 |
| @Scope\(“singleton”\) | @Singleton | JSR-330默认范围类似Spring的原型。但是，为了使其与Spring常规默认值保持一致，默认情况下，在Spring容器中声明的JSR-330 bean是单例。为了使用一个不是单例的范围，您应该使用Spring的@Scope注解。javax.inject还提供了@Scope注解。然而，这只是用于创建自己的注解。 |
| @Qualifier | @Qualifier / @Named | javax.inject.Qualifier只是构建自定义限定符的元注解。具体的字符串限定符（像Spring的@Qualifier值）可以通过javax.inject.Named关联 |
| @Value | – | 没有对应的 |
| @Required | – | 没有对应的 |
| @Lazy | – | 没有对应的 |
| ObjectFactory | Provider | javax.inject.Provider是Spring ObjectFactory 的替代品，只需要一个简短的get\(\)方法名称。它也可以和Spring @Autowired或非注解的构造器和setter方法结合使用。 |

## 3.12 基于Java的容器配置 {#toc_5}

### 3.12.1 基本概念：@Bean 和 @Configuration {#toc_6}

最核心的是Spring支持全新的Java配置，例如@Configuration注解的类和@Bean注解的方法。  
@Bean注解用来说明通过Spring IoC容器来管理时一个新对象的实例化，配置和初始化的方法。这对于熟悉Spring以XML配置的方式，@Bean和 element元素扮演了相同的角色。你可以在任何使用@Componen的地方使用@Bean，但是更常用的是在配置@Configuration的类中使用。  
一个用@Configuration注解的类说明这个类的主要是作为一个bean定义的资源文件。进一步的讲，被@Configuration注解的类通过简单地在调用同一个类中其他的@Bean方法来定义bean之间的依赖关系。简单的@Configuration 配置类如下所示：

```
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }

}
```

上面的AppConfig类和Spring XML 的配置是等价的：

```
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

> Full @Configuration vs 'lite' @Beans mode?
>
> 当@Bean方法在没有使用@Configuration注解的类中声明时，它们被称为以“lite”进行处理。例如，用@Component修饰的类或者简单的类中都被认为是“lite”模式。
>
> 不同于full @Configuration，lite @Bean 方法不能简单的在类内部定义依赖关系。通常，在“lite”模式下一个@Bean方法不应该调用其他的@Bean方法。
>
> 只有在@Configuration注解的类中使用@Bean方法是确保使用“full”模式的推荐方法。这也可以防止同样的@Bean方法被意外的调用很多次，并有助于减少在'lite'模式下难以被追踪的细小bug。

这个模块下我们深入的讨论了@Configuration和@Beans注解，首先我们将介绍基于Java配置的各种Spring容器的创建。

### 3.12.2 使用AnnotationConfigApplicationContext实例化Spring容器 {#toc_7}

下面的部分介绍Spring的AnnotationConfigApplicationContext，Spring 3.0的新内容。这个通用的ApplicationContext实现不仅可以接受@Configuration注解类为输入，还可以接受使用JSR-330元数据注解的简单类和@Component类。

当@Configuration注解的类作为输入时，@Configuration类本身会被注册为一个bean，在这个类中所有用@Bean注解的方法都会被定义为一个bean。

当使用@Component和JSR-330类时，它们被注册为bean的定义，并且假设在有必要时使用这些类内部诸如@Autowired或@Inject之类的DI元数据。

**简单构造**

实例化使用@Configuration类作为输入实例化AnnotationConfigApplicationContext和实例化ClassPathXmlApplicationContext时使用Spring的XML文件作为输入的方式大致相同。这在无XML配置的Spring容器时使用：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如上所述，AnnotationConfigApplicationContext不限于仅使用@Configuration类。任何@Component或JSR-330注解的类都可以作为输入提供给构造函数。例如：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

上面假设MyServiceImpl、Dependency1和Dependency2都用了Spring的依赖注入的注解，例如@Autowired。

**使用register\(Class&lt;?&gt;…​\)的方式构建容器**

也可以使用无参构造函数实例化AnnotationConfigApplicationContext，然后使用register\(\)方法配置。当使用编程方式构建AnnotationConfigApplicationContext时，这种方法特别有用。

**使用scan（String …）组件扫描**

启用组件扫描，只需要在你的@Configuration类中做如下配置：

```
@Configuration
@ComponentScan(basePackages = "com.acme")
public class AppConfig  {
    ...
}
```

有Spring使用经验的用户，对Spring XML的context的声明非常熟悉：

```
<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
```

在上面的例子中，com.acme将会被扫描，它会寻找任何@Component注解的类，这些类将会在Spring的容器中被注册成为一个bean。AnnotationConfigApplicationContext暴露的scan\(String…​\)方法以达到相同组件扫描的功能：

```
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

记住：使用@Configuration注解的类是使用@Component进行元注解，所以它们也是组件扫描的候选，假设AppConfig被定义在com.acme这个包下（或者它下面的任何包），它们都会在调用scan\(\)方法期间被找出来，然后在refresh\(\)方法中它们所有的@Bean方法都会被处理，在容器中注册成为bean。

**AnnotationConfigWebApplicationContext对于web应用的支持**

AnnotationConfigApplicationContext在WebApplicationContext中的变体为  
AnnotationConfigWebApplicationContext。当配置Spring ContextLoaderListener servlet 监听器、Spring MVC DispatcherServlet的时候，可以用此实现。下面为配置典型的Spring MVC DispatcherServlet的web.xml代码段。注意contextClass上下文参数和init-param的使用：

```
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

### 3.12.3 使用@Bean注解 {#toc_8}

@Bean是XML元素方法级注解的直接模拟。它支持由提供的一些属性，例如：[init-method](http://ifeve.com/spring-beans-standard-annotations/beans.html#beans-factory-lifecycle-initializingbean)，[destroy-method](http://ifeve.com/spring-beans-standard-annotations/beans.html#beans-factory-lifecycle-disposablebean)，[autowiring](http://ifeve.com/spring-beans-standard-annotations/beans.html#beans-factory-autowire)和name。

您可以在@Configuration或@Component注解的类中使用@Bean注解。

**定义一个bean**

要定义一个bean，只需在一个方法上使用@Bean注解。您可以使用此方法在指定方法返回值类型的ApplicationContext中注册bean定义。默认情况下，bean名称与方法名称相同。以下是@Bean方法声明的简单示例：

```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

这种配置完全和下面的Spring XML配置等价：

```
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

两种声明都可以使得一个名为transferService的bean在ApplicationContext可用，绑定到TransferServiceImpl类型的对象实例上：

transferService -&gt; com.acme.TransferServiceImpl

**Bean 依赖**

@Bean注解方法可以具有描述构建该bean所需依赖关系的任意数量的参数。例如，如果我们的TransferService需要一个AccountRepository，我们可以通过一个方法参数实现该依赖：

```
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

这种解决原理和基于构造函数的依赖注入几乎相同，请参考相关章节，查看详细信息。

**生命周期回调**

任何使用了@Bean定义了的类都支持常规生命周期回调，并且可以使用JSR-250中的@PostConstruct和@PreDestroy注解，详细信息，参考JSR-250注解。

完全支持常规的Spring生命周期回调。如果一个bean实现了InitializingBean，DisposableBean或Lifecycle接口，它们的相关方法就会被容器调用。

完全支持\*Aware系列的接口，例如：BeanFactoryAware，BeanNameAware，MessageSourceAware，ApplicationContextAware等。

@Bean注解支持任意的初始化和销毁回调方法，这与Spring XML 中bean元素上的init方法和destroy-method属性非常相似：

```
public class Foo {
    public void init() {
        // initialization logic
    }
}

public class Bar {
    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }

    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }

}
```

默认情况下，使用Java config定义的具有公开关闭或停止方法的bean将自动加入销毁回调。如果你有一个公开的关闭或停止方法，但是你不希望在容器关闭时被调用，只需将@Bean（destroyMethod =””）添加到你的bean定义中即可禁用默认（推测）模式。 默认情况下，您可能希望通过JNDI获取资源，因为它的生命周期在应用程序之外进行管理。特别地，请确保始终为DataSource执行此操作，因为它已知在Java EE应用程序服务器上有问题。

```
@Bean(destroyMethod="")
public DataSource dataSource() throws NamingException {
    return (DataSource) jndiTemplate.lookup("MyDS");
}
```

另外，通过@Bean方法，通常会选择使用编程来进行JNDI查找：要么使用Spring的JndiTemplate/JndiLocatorDelegate帮助类，要么直接使用JNDI InitialContext，但不能使用JndiObjectFactoryBean变体来强制将返回类型声明为FactoryBean类型以代替目标的实际类型，它将使得在其他@Bean方法中更难用于交叉引用调用这些在此引用提供资源的方法。  
当然上面的Foo例子中，在构造期间直接调用init（）方法同样有效：

```
@Configuration
public class AppConfig {
    @Bean
    public Foo foo() {
        Foo foo = new Foo();
        foo.init();
        return foo;
    }

    // ...

}
```

当您直接在Java中工作时，您可以对对象执行任何您喜欢的操作，并不总是需要依赖容器生命周期！

**指定bean的作用域**

**使用@Scope注解**

你可以指定@Bean注解定义的bean应具有的特定作用域。你可以使用Bean作用域章节中的任何标准作用域。

默认的作用域是单例，但是你可以用@Scope注解重写作用域。

```
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }

}
```

**@Scope和scope 代理**

Spring提供了一个通过范围代理来处理范围依赖的便捷方法。使用XML配置创建此类代理的最简单方法是元素。使用@Scope注解配置Java中的bean提供了与proxyMode属性相似的支持。默认是没有代理（ScopedProxyMode.NO），但您可以指定ScopedProxyMode.TARGET\_CLASS或ScopedProxyMode.INTERFACES。

如果你使用Java将XML参考文档（请参阅上述链接）到范围的@Bean中移植范围限定的代理示例，则它将如下所示  
如果你将XML 参考文档的scoped代理示例转化为Java @Bean，如下所示：

```
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

**自定义Bean命名**

默认情况下，配置类使用@Bean方法的名称作为生成的bean的名称。但是，可以使用name属性来重写此功能。

```
@Configuration
public class AppConfig {

    @Bean(name = "myFoo")
    public Foo foo() {
        return new Foo();
    }

}
```

**Bean别名**

如3.3.1节”bean 命名”中所讨论的，有时一个单一的bean需要给出多个名称，称为bean别名。 为了实现这个目标，@Bean注解的name属性接受一个String数组。

```
@Configuration
public class AppConfig {

    @Bean(name = { "dataSource", "subsystemA-dataSource", "subsystemB-dataSource" })
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }

}
```

**Bean描述**

有时候需要提供一个详细的bean描述文本是非常有用的。当对bean暴露（可能通过JMX）进行监控使，特别有用。

可以使用@Description注解对Bean添加描述：

```
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Foo foo() {
        return new Foo();
    }

}
```

### 3.12.4 使用@Configuration注解 {#toc_9}

@Configuration是一个类级别的注解，指明此对象是bean定义的源。@Configuration类通过public @Bean注解的方法来声明bean。在@Configuration类上对@Bean方法的调用也可以用于定义bean之间的依赖。概述，请参考第3.12.1节”基本概念：@Bean和@Configuration”。

**bean依赖注入**

当@Beans相互依赖时，表示依赖关系就像一个bean方法调用另一个方法一样简单：

```
@Configuration
public class AppConfig {

    @Bean
    public Foo foo() {
        return new Foo(bar());
    }

    @Bean
    public Bar bar() {
        return new Bar();
    }

}
```

上面的例子，foo接受一个bar的引用来进行构造器注入：

> 这种方法声明的bean的依赖关系只有在@Configuration类的@Bean方法中有效。你不能在@Component类中来声明bean的依赖关系。

**方法查找注入**

如前所述，方法查找注入是一个你很少用用到的高级特性。在单例的bean对原型的bean有依赖性的情况下，它非常有用。这种类型的配置使用，Java提供了实现此模式的自然方法。

```
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

使用Java支持配置，您可以创建一个CommandManager的子类，覆盖它抽象的createCommand\(\)方法，以便它查找一个新的（原型）命令对象：

```
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with command() overridden
    // to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

**有关基于Java配置内部如何工作的更多信息**

下面的例子展示了一个@Bean注解的方法被调用两次：

```
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }

}
```

clientDao\(\)方法clientService1\(\)和clientService2\(\)被各自调用了一次。因为这个方法创建并返回了一个新的ClientDaoImpl实例，你通常期望会有2个实例（每个服务各一个）。这有一个明显的问题：在Spring中，bean实例默认情况下是单例。神奇的地方在于：所有的@Configuration类在启动时都使用CGLIB进行子类实例化。在子类中，子方法在调用父方法创建一个新的实例之前会首先检查任何缓存\(作用域\)的bean。注意，从Spring 3.2开始，不再需要将CGLIB添加到类路径中，因为CGLIB类已经被打包在org.springframework.cglib下，直接包含在spring-core JAR中。

> 根据不同的bean作用域，它们的行为也是不同的。我们这里讨论的都是单例模式。
>
> 这里有一些限制是由于CGLIB在启动时动态添加的特性，特别是配置类都不能是final类型。然而从4.3开始，配置类中允许使用任何构造函数，包含@Autowired使用或单个非默认构造函数声明进行默认注入。如果你希望避免CGLIB带来的任何限制，那么可以考虑子在非@Configuration注解类中声明@Bean注解方法。例如，使用@Component注解类。在 @Bean方法之间的交叉调用不会被拦截，所以你需要在构造器或者方法级别上排除依赖注入。

### 3.12.5 基于Java组合配置 {#toc_10}

**使用@Import注解**

和Spring XML文件中使用元素来帮助模块化配置类似，@Import注解允许从另一个配置类加载@Bean定义：

```
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }

}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }

}
```

现在，在实例化上下文时不是同时指明ConfigA.class和ConfigB.class，而是仅仅需要明确提供ConfigB：

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

这种方法简化了容器实例化，因为只需要处理一个类，而不是需要开发人员在构建期间记住大量的@Configuration注解类。

> 从Spring Framework 4.2开始，@Import注解也支持对常规组件类的引用，类似AnnotationConfigApplicationContext.register方法。如果你希望避免组件扫描，使用一些配置类作为所有组件定义的入口，这个方法特别有用。

**引入的@Bean定义中注入依赖**

上面的类中可以运行，但是太过简单。在大多数实际场景中，bean在配置类之间相互依赖。当使用XML时，这没有问题，因为没有编译器参与，一个bean可以简单的声明为ref=”someBean”并且相信Spring在容器初始化过程处理它。当然，当使用@Configuration注解类，Java编译器会对配置模型放置约束，以便其他对其他引用的bean进行Java语法校验。  
幸运的是，解决这个问题也很简单。正如我们讨论过的，@Bean方法可以有任意数量的参数来描述bean的依赖。让我们考虑一下真实场景和一系列@Configuration类，每个bean都依赖了其他配置中声明的bean：

```
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }

}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }

}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

这里有其他的方法实现相同的结果。记住@Configuration类最终只是容器中的另一个bean：这意味着它们可以像任何其他bean一样利用@Autowired和@Value注入等！

> 确保您注入的依赖关系是最简单的。@Configuration类在上下文初始化期间处理，强制要求依赖使用这种方式进行注入可能导致意外的早期初始化问题。如果可能，就采用如上述例子所示的基于参数的注入。
>
> 同时，也要特别小心通过@Bean的BeanPostProcessor BeanFactoryPostProcessor定义。它们应该被声明为static的@Bean方法，不会触发包含它们的配置类的实例化。否则，@Autowired和@Value将在配置类上不生效，因为它太早被创建为一个实例了。

```
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }

}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    @Autowired
    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

> 只有Spring Framework 4.3才支持@Configuration类中的构造方法注入。注意，如果目标bean只定义了一个构造函数，那么则不需要指定@Autowired；如果目标bean只定义一个构造函数，则不需要指定@Autowired;在上面的例子中，@Autowired在RepositoryConfig构造函数中是不必要的。
>
> 在上面的场景中，使用@Autowired可以很好的提供所需的模块化，但是准确的决定在哪里自动注入定义的bean还是模糊的。例如，作为开发者来看待ServiceConfig，如何准确的确定自动注入 AccountRepository 是在哪里声明的？它没有明确的出现在代码中，这可能还不错。记住，Spring Tool Suite 提供工具可以渲染图形展示对象是如何装配的，这些可能是你所需要的。同时，你的Java IDE 也可以很简单的找出所有AccountRepository类型的声明和使用，这将很快的展示出你@Bean方法的位置和返回类型。

在这种歧义不可接受的情况下，你希望从IDE中直接从一个@Configuration类导航到另一个类，可以考虑自动装配配置类的本身：

```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }

}
```

在上面的情形中，定义的AccountRepository是完全透明的。但是，ServiceConfig和RepositoryConfig是紧密耦合在一起了；这需要权衡。这种紧密耦合的可以通过基于接口或者抽象@Configuration类来缓解。可以考虑下面的代码：

```
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();

}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }

}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class}) // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

现在ServiceConfig与具体DefaultRepositoryConfig就是送耦合，IDE内置的工具也仍然有用：对开发人员来说，可以轻松的获取RepositoryConfig实现层级类型。用这种方法，定位到@Configuration类及它的依赖类和定位基于接口的代码就没什么区别。

**有条件的包括@Configuration类或@Bean方法**

通常，有条件的开启或者禁用一个完整的@Configuration类，甚至是基于有任意系统状态的单独@Bean方法。一个常见的例子就是使用@Profile注解来激活仅在Spring 环境中启用的特定的profile文件（有关详细信息，请参阅第3.13.1节“Bean definition profiles”）  
@Profile注解是用一个更加灵活的@Conditional注解实现的。@Conditional注解表示@Bean在被注册前应该查阅特定的org.springframework.context.annotation.Condition实现。  
Condition接口的实现只提供了一个返回true或者false的matches\(…\)方法。例如  
@Profile是Condition的具体实现：

```
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
        // Read the @Profile annotation attributes
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
    }
    return true;
}
```

@Conditional详细信息请参考javadocs。

**Java and XML 混合配置**

Spring 对@Configuration配置类的支持的目的不是100%来替换Spring XML配置的。一些基本特性，例如：Spring XML命名空间仍然是容器配置的一个理想方式。在 XML更便于使用或者是必须使用的情况下，要么以“XML为中心”的方式来实例化容器，比如，ClassPathXmlApplicationContext，要么以“Java为中心”的方式，使用AnnotationConfigurationApplicationContext 和@ImportResource注解来引入所需的XML。

**以XML为中心使用@Configuration类**

假设你可能会以XML包含@Configuration类的方式来启动一个Spring容器。例如，在一个现有使用Spring XML的大型代码库中，根据需要从已有的XML文件中创建@Configuration类是很简单的。下面你可以发现在以XML为中心的情形下使用@Configuration类的选项。谨记，@Configuration类最终只是容器中的一个bean。在这个例子中，我们会创建一个名为AppConfig的@Configuration类，它作为一个bean的定义包含在system-test-config.xml中。因为[context:annotation-config/](http://ifeve.com/spring-beans-standard-annotations/annotation-config/)是打开的，容器会识别@Configuration，并且会处理AppConfig中声明的@Bean方法。

```
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }

}
```

**system-test-config.xml:**

```
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

**jdbc.properties:**

jdbc.url=jdbc:hsqldb:hsql://localhost/xdb

jdbc.username=sa

jdbc.password=

```
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

> 在上面的system-test-config.xml中，AppConfig中的bean没有声明id的元素。然而它也可以被接受，在没有其他bean引用的情况下也没有必要给出，也不太可能通过名字的方式从容器中显式取出。像DataSource bean一样，只能通过类型自动注入，所以明确的bean id也不严格要求。

因为@Configuration是@Component的一个元注解，对于component的扫描@Configuration注解类会自动成为候选者。和上面的场景相同，利用component扫描可以重新定义system-test-config.xml。注意在个案例中，我们不需要明确的声明[context:annotation-config/](http://ifeve.com/spring-beans-standard-annotations/annotation-config/)，因为开启[context:component-scan/](http://ifeve.com/spring-beans-standard-annotations/component-scan/)，功能是相同的。

**system-test-config.xml:**

```
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

**@Configuration类为中心的 XML @ImportResource的使用**

在以@Configuration类为主要机制的配置容器的应用程序中，仍然有必要使用一些XML。在这些场景中，只需使用@ImportResource，并根据需要定义一些XML。这样实现了“以Java为中心”方式来配置容器，并将XML保持在最低限度。

```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }

}
```

**properties-config.xml**

```
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
```

**jdbc.properties**

jdbc.url=jdbc:hsqldb:hsql://localhost/xdb

jdbc.username=sa

jdbc.password=

```
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

## 3.13 环境抽象 {#toc_11}

在应用环境中，集成在容器的抽象环境模型有两个方面：profiles和properties。只有给出的profile被激活，一组逻辑命名的bean定义才会在容器中注册。无论是在XML中或者通过注解，bean都会被分配给一个profile。环境变量对象角色和profiles的关系来决定哪个profiles\(如果有\)处于当前激活状态，哪个profiles默认被激活。几乎在所有的应用中，Properties都扮演了一个重要的对象，这可能有各种来源：属性文件， JVM 系统属性文件，系统环境变量，JNDI，servlet上下文参数，属性查询对象，Maps等等。环境变量对象的角色和properties的关系用于配置属性并从中解析属性提供给用户一个便捷的服务接口。

### 3.13.1 Bean的profiles定义 {#toc_12}

Bean定义profiles是在核心容器中允许不同的bean在不同环境注册的机制。环境对于不同的用户意味着不同的东西，这个特性可以帮助许多用例，包括：

* 在开发中不使用内存中的数据源 VS 在质量测试或生产环境中从JNDI查找相同的数据源。
* 当把应用部署在可执行的环境中时注册监控基础架构
* 对于客户A注册的自定义实现VS.客户B部署

让我首先考虑在一个需要数据源的应用中使用这个例子。在测试环境中，配置可能如下：

```
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

让我现在考虑一下，如何把这个应用部署在测试环境或者生产环境中，假设应用所需的数据源将会被注册在生产应用环境中的JNDI目录。现在我们的数据源bean看起来像这样：

```
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题就是如何根据当前的环境在这两种变量之间进行切换。随着时间的推移，Spring的用户已经设计了很多种方法来实现此功能，通常依赖于系统环境变量和包含${placeholder}的XML 语句，根据环境变量的值可以解决正确的文件路径配置。Bean定义profiles是容器为了解决这个问题而提供的一个核心功能。

如果我们概括一下上面bean定义环境变量的示例，我们最终需要在特定的上下文中注册特定的bean，而不是其他的。你可以说你想要在情形A中注册一个特定的Bean定义的profile，在情形B中是另外一个。我们首先看下如何更新我们的配置以反映这种需求。

**@Profile**

当一个或者多个特定的profiles被激活，@Profile注解允许你指定一个有资格的组件来注册。使用我们上面的例子，我们可以按照下面的重写dataSource配置：

```
@Configuration
@Profile("dev")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

```
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

> 如前所述，使用@Bean方法，通常会选择使用程序话的JNDI查找：要么使用Spring的 JndiTemplate/JndiLocatorDelegate帮助要么直接使用上面展示的JNDI InitialContext，而不是强制声明返回类型为FactoryBean的JndiObjectFactoryBean变体。

@Profile可以被用作为创建一个自定义组合注解的元注解。下面的例子定义了一个@Production注解，它可以被用作替换@Profile（“production”）的注解。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

在仅仅包含一个特殊bean的配置类中，@Profile也可以被声明在方法级别：

```
@Configuration
public class AppConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean
    @Profile("production")
    public DataSource productionDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

> 如果一个@Configuration类被标记为@Profile，那么所有的@Bean方法和@Import注解相关的类都会被忽略，除非一个或多个特别的profiles被激活。如果一个@Component或@Configuration类被标记为@Profile\({“p1”, “p2”}\)，那么这个类将不会被注册/处理，除非被标记为’p1’和/或’p2’的profiles已经被激活。如果给出的profile的前缀带有取反的操作符\(!\)，那么注解的元素将会被注册，除非这个profile没有被激活。例如，给出@Profile\({“p1”, “!p2”}\)，如果profile ‘p1’是激活状态或者profile ‘p2’不是激活状态的时候才会注册。

### 3.13.2 XML bean 定义 profiles {#toc_13}

XML对应元素的profile属性。我们上面的示例配置可以重写为下面的两个XML配置：

```
<beans profile="dev"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
```

```
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

也可以避免在同一个文件中分割和嵌套元素：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="dev">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

spring-bean.xsd 约束允许这样的元素仅作为文件中的最后一个元素。这有助于XML的灵活性，且不会产生混乱。

**激活profile**

现在我们已经更新了我们的配置，我们仍然需要说明哪个profile是激活的。如果我们现在开始我们示例应用程序，我们将会看到一个NoSuchBeanDefinitionException被抛出，因为容器找不到一个名为dataSource的Spring bean。  
激活一个profile可以通过多种方式完成，但是大多数情况下，最直接的办法就是通过存在ApplicationContext当中的环境变量的API进行编程：

```
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("dev");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

除此之外，profiles也可以通过声明spring.profiles.active属性来激活，这个可以通过在系统环境变量，JVM系统属性，web.xml中的servlet上下文环境参数，甚至JNDI的入口（请参考 3.13.3, “PropertySource abstraction”）。在集成测试中，激活profiles可以通过在Spring-test模块中的@ActiveProfiles注解来声明（参见“使用profiles来配置上下文环境”章节）。

注意，profiles不是“二者选一”的命题；它可以一次激活多个profiles。以编程的方式来看，简单的传递多个profile名字给接受String 可变变量参数的setActiveProfiles\(\)方法：

```
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

在声明式中，spring.profiles.active可以接受以逗号分隔的profile 名称列表：

```
-Dspring.profiles.active="profile1,profile2"
```

**默认的profile**

默认配置文件表示默认启用的配置文件。考虑以下几点：

```
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有profile是激活状态，上面的dataSource将会被创建；这种方式可以被看做是对一个或者多个bean提供了一种默认的定义方式。如果启用任何的profile，那么默认的profile都不会被应用。  
在上下文环境可以使用setDefaultProfiles\(\)或者spring.profiles.default属性来修改默认的profile名字。

### 3.13.3 属性源抽象 {#toc_14}

Spring 环境抽象提供了可配置的属性源层次结构的搜索操作。为了充分的解释，请考虑下面的例子：

```
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsFoo = env.containsProperty("foo");
System.out.println("Does my environment contain the 'foo' property? " + containsFoo);
```

在上面的代码段中，我们看到了一个高级别的方法来要求Spring是否为当前环境定义foo属性。为了回答这个问题，环境对象对一组PropertySource对象执行搜索。一个PropertySource是对任何key-value资源的简单抽象，并且Spring 的标准环境是由两个PropertySource配置的，一个表示一系列的JVM 系统属性\(System.getProperties\(\)\),一个表示一系列的系统环境变量\(System.getenv\(\)\)。

> 这些默认的属性资源存在于StandardEnvironment，可以在应用中独立使用。StandardServletEnvironment包含其他默认的属性资源，包括servlet配置和servlet上下文参数。它可以选择性的启用JndiPropertySource。详细信息请查看javadocs。

具体的说，当使用StandardEnvironment时，如果在运行时系统属性或者环境变量中包括foo，那么调用env.containsProperty\(“foo”\)方法将会返回true。

搜索是按照层级执行的。默认情况，系统属性优先于环境变量，所以这两个地方同时存在属性foo的时候，调用env.getProperty\(“foo”\)将会返回系统属性中的foo值。注意，属性值不会被合并而是被之前的值覆盖。对于一个普通的StandardServletEnvironment，它完整的层次结构如下，最顶端的优先级最高：

* ServletConfig参数\(如果适用，例如DispatcherServlet上下文环境\)
* ServletContext参数\(web.xml中的context-param\)
* JNDI环境变量\(“java:comp/env/”\)
* JVM系统属性\(“-D”命令行参数\)
* JVM系统环境变量\(操作系统环境变量\)

更重要的是，整个机制都是可配置的。也许你有个自定义的属性来源，你想把它集成到这个搜到里面。这也没问题，只需简单的实现和实例化自己的PropertySource，并把它添加到当前环境的PropertySources集合中：

```
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

在上面的代码中，MyPropertySource被添加到搜索中的最高优先级。如果它包含了一个foo属性，在任何其他的PropertySource中的foo属性之前它会被检测到并返回。MutablePropertySources API暴露了很多允许精确操作该属性源集合的方法。

### 3.13.4 @PropertySource {#toc_15}

@PropertySource注解对添加一个PropertySource到Spring的环境变量中提供了一个便捷的和声明式的机制。  
给出一个名为”app.properties”的文件，它含了testbean.name=myTestBean的键值对，下面的@Configuration类使用@PropertySource的方式来调用testBean.getName\(\)，将会返回”myTestBean”。

```
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {
 @Autowired
 Environment env;

 @Bean
 public TestBean testBean() {
  TestBean testBean = new TestBean();
  testBean.setName(env.getProperty("testbean.name"));
  return testBean;
 }
}
```

任何出现在@PropertySource中的资源位置占位符都会被注册在环境变量中的资源解析。

```
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {
 @Autowired
 Environment env;

 @Bean
 public TestBean testBean() {
  TestBean testBean = new TestBean();
  testBean.setName(env.getProperty("testbean.name"));
  return testBean;
 }
}
```

假设”my.placeholder”已经在其中的一个资源中被注册，例如：系统属性或环境变量，占位符将会被正确的值解析。如果没有，”default/path”将会使用默认值。如果没有默认值，而且无法解释属性，则抛出IllegalArgumentException异常。

### 3.13.5 声明中的占位符解决方案 {#toc_15}

以前，元素中占位符的值只能被JVM系统熟悉或者环境变量解析。现在已经解决了这种情况。因为抽象的环境已经通过容器被集成了，很容易通过它来分配占位符。这意味着你可以使用任何你喜欢的方式配置：可以通过系统属性和环境变量来改变搜索优先级，或者完全删除它们；可以以适当的方式添加你自己混合属性资源。

具体来说，下面的声明无论customer属性被定义在哪里，只要它存在环境变量中就有作用：

## 3.14 注册一个加载时编织器 {#toc_16}

在类被加载进JVM时Spring使用LoadTimeWeaver进行动态转换。  
为了使的load-time weaving可用，那么你只需在配置了@Configuration的类上添加@EnableLoadTimeWeaving。

```
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {

}
```

相应的xml配置使用context:load-time-weaver元素：

```
<beans>
    <context:load-time-weaver/>
</beans>
```

一旦配置了ApplicationContext，那么在ApplicationContext中的任何bean都可以实现LoadTimeWeaverAware，从而接受对类加载时编织器实例的引用。这与Spring JPA支持相结合时非常有用，JPA类转化必须使用加载时编织。可以通过javadocs的LocalContainerEntityManagerFactoryBean获取更多详细信息，对于AspectJ加载时的编织请参考：

Section 7.8.4, “Load-time weaving with AspectJ in the Spring Framework”.

## 3.15 ApplicationContext的额外功能 {#toc_17}

正如本章开头所讨论的那样，org.springframework.beans.factory包提供基本的功能来管理和操作bean，包括以编程的方式。The org.springframework.context包增加了ApplicationContext接口，它继承了BeanFactory接口，除了以面向应用框架的风格扩展接口来提供一些额外的功能。很多人以完全声明的方式使用ApplicationContext，甚至没有以编程的方式去创建它，而是依赖诸如ContextLoader等支持类来自动的实例化ApplicationContext，作为Java EE web应用程序正常启动的一部分。  
为了增强BeanFactory在面向框架风格的功能，上下文的包还提供了以下的功能：

* 通过MessageSource接口访问i18n风格的消息
* 通过ResourceLoader接口访问类似URL和文件资源
* 通过ApplicationEventPublisher接口，即bean实现ApplicationListener接口来进行事件发布
* 通过HierarchicalBeanFactory接口实现加载多个\(分层\)上下文，允许每个上下文只关注特定的层，例如应用中的web层

### 3.15.1 使用MessageSource 国际化 {#toc_18}

ApplicationContext接口继承了一个叫做MessageSource的接口，因此它也提供了国际化\(i18n\)的功能。Spring也提供了HierarchicalMessageSource接口，它可以分层去解析信息。这些接口共同为Spring消息效应解析提供了基础。这些接口上定义的方法包括：

* String getMessage\(String code, Object\[\] args, String default, Locale loc\): 这个基础的方法用来从MessageSource检索消息。当指定的区域中没有发现消息时，将使用默认的。任何参数传递都将使用标准库提供的MessageFormat变成替换值。
* String getMessage\(String code, Object\[\] args, Locale loc\): 本质上和前面提供的方法相同，只有一个区别，就是当没有指定消息，又没有发现消息，将会抛出NoSuchMessageException 异常。
* String getMessage\(MessageSourceResolvable resolvable, Locale locale\): 所有的属性处理方法都被包装在一个名为MessageSourceResolvable的类中，你可以使用此方法。

当ApplicationContext被载入的时候，它会自动的在上下文中去搜索定义的MessageSource bean。这个bean必须有messageSource的名称。如果找到这么一个bean，所有上述方法的调用都会委托给消息源。如果没有发现消息源，ApplicationContext会尝试寻找一个同名的父消息源。如果是这样，它会将那个bean作为MessageSource。如果ApplicationContext没有找到任何的消息源，那么一个空的DelegatingMessageSource将被实例化，以便能够接受到对上述定义方法的调用。

Spring提供了ResourceBundleMessageSource和StaticMessageSource两个MessageSource实现。它们两个都实现了HierarchicalMessageSource以便处理嵌套消息。StaticMessageSource很少使用，但是它提供了通过编程的方式增加消息源。下面展示ResourceBundleMessageSource使用的例子：

```
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

在上面的例子中，假设在类路径下定义了format，exceptions和windows三个资源包。解析消息的任何请求都会通过ResourceBundles被JDK以标准方式处理。为了举例说明，假设上述两个资源包的文件内容是…

```
# in format.properties
message=Alligators rock!
```

```
# in exceptions.properties
argument.required=The {0} argument is required.
```

下面的实例展示了执行MessageSource功能的程序。记住所有的ApplicationContext的实现也是MessageSource的实现，而且它可以被强转为MessageSource接口。

```
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", null);
    System.out.println(message);
}
```

上面的程序输出的结果为：  
Alligators rock!

所以总结一下，MessageSource是定义在一个名为beans.xml，它存在类路径的跟目录下。messageSource bean定义通过basenames属性引用了很多的资源。在列表中传递给basenames属性的三个文件作为类路径下根目录中的文件存在，分别为format.properties, exceptions.properties, and windows.properties。

下一个例子展示传递给消息查找的参数；这些参数将会被转换为字符串并插入到消息查找的占位符中。

```
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.foo.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```

```
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", null);
        System.out.println(message);
    }

}
```

调用execute\(\)方法的输出结果为:  
The userDao argument is required.  
关于国际化\(i18n\)，Spring的各种MessageSource实现遵循与标准JDK ResourceBundle相同的语言环境和回退规则。简而言之，并继续用前面messageSource 为例，如果你想根据英国\(en-GB\)解析消息，你可以创建这些文件format\_en\_GB.properties，exceptions\_en\_GB.properties, and windows\_en\_GB.properties。  
通常，地域设置通过应用周围环境管理的。在此示例中，手动指定对\(英国\)区域消息进行解析。

\# in exceptions\_en\_GB.properties

argument.required=Ebagum lad, the {0} argument is required, I say, required.

```
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

上面的程序运行输出为：

`Ebagum lad, the ‘userDao’ argument is required, I say, required.`

你也可以使用MessageSourceAware接口来获取对已定义MessageSource的引用。任何在ApplicationContext定义的bean都会实现MessageSourceAware，当bean被创建或者配置的时候，它会在应用上下文的MessageSource中被被注入。

> 作为ResourceBundleMessageSource的替代方法，Spring提供了一个ReloadableResourceBundleMessageSource类。这个变体支持同样打包文件格式，但是它更灵活而不是标准JDK基于ResourceBundleMessageSource的实现。特别的，它允许从任何Spring 资源位置读取文件\(不仅仅是从类路径\)而且还支持属性文件热加载\(同时高效缓存他们\)。ReloadableResourceBundleMessageSource的详细信息参考javadocs。



