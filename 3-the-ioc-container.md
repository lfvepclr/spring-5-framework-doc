# 3. IoC容器

## 3.1 Spring IoC容器和beans的介绍

本章涵盖了Spring框架实现控制反转（IoC）[\[1\]](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#ftn.d5e685)的原理。IoC又叫依赖注入（DI）。它描述了对象的定义和依赖的一个过程，也就是说，依赖的对象通过构造参数、工厂方法参数或者属性注入，当对象实例化后依赖的对象才被创建，当创建bean后容器注入这些依赖对象。这个过程基本上是反向的，因此命名为控制反转（IoC），它通过直接使用构造类来控制实例化，或者定义它们之间的依赖关系，或者类似于服务定位模式的一种机制。

`org.springframework.beans`和`org.springframework.context`是Spring框架中IoC容器的基础，[`BeanFactory`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/beans/factory/BeanFactory.html)接口提供一种高级的配置机制能够管理任何类型的对象。[`ApplicationContext`](http://ifeve.com/spring-ioc-1-2/href="http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/context/ApplicationContext.html)是`BeanFactory`的子接口。它能更容易集成Spring的AOP功能、消息资源处理（比如在国际化中使用）、事件发布和特定的上下文应用层比如在网站应用中的`WebApplicationContext。`

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



