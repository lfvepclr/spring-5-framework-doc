# 35. Spring注解编程模型

## 介绍

这篇文档是以Spring Framework 4.2作为框架基础编写的，但是，这篇文档是一份还在进行的工作。所以随着时间推移，你会看到这份文档还在更新。

**目录**

* [概要](http://ifeve.com/annotation-programming-model/#1)
* [术语](http://ifeve.com/annotation-programming-model/#2)
* [例子](http://ifeve.com/annotation-programming-model/#3)
* [FAQ](http://ifeve.com/annotation-programming-model/#4)
* [附录](http://ifeve.com/annotation-programming-model/#5)

## 概要

这些年，Spring Framework已经频繁的升级它可以支持的注解、元注解和组合注解。这篇文档旨在帮助开发者（Spring框架使用者、Spring核心框架开发者和Spring全家桶的成员项目开发者）开发和运用Spring注解。

### 这些是文档目标

这篇文档的主要目的包含以下内容：

* 怎么使用Spring注解。
* 怎么自定义组合注解。
* 怎么查找Spring注解（或者说，理解Spring注解搜索算法的原理）。

### 这些并非文档目标

这篇文档并不介绍spring经典注解的语义和配置方法。如果想学习经典注解的细节，建议开发者查阅对应的Javadoc或者参考书籍的相应章节。

## 术语

### 元注解

元注解是一种标注在别的注解之上的注解。如果一个注解可以标注在别的注解上，那么这个注解已然是元注解。例如，任何需要被文档化的注解，都应该被`java.lang.annotation`包中的元注解`@Documented`标注。

### Stereotype注解

_`译者注：保留Stereotype原生词汇；可理解为模式化注解、角色类注解。`_

Stereotype注解是一种在应用中，常被用于声明要扮演某种职责或者角色的注解。例如，`@Repository`注解用于标注任何履行了repository职责角色的类（这种职责角色通常也会被称为Data Access Object或者DAO）。

`@Component`是被Spring管理的组件的对应注解。任何标注了`@Component`的组件都会在spring组件扫描时被扫描到。同样的，任何标注了 被元注解`@Component`标注过的注解 的组件，也会在Spring组件扫描时被扫描到。例如，`@Service`就是一种被元注解`@Component`标注过的注解。

Spring核心框架提供了一系列可直接使用的stereotype注解，包括但不限于`@Component`,`@Service`,`@Repository`,`@Controller`,`@RestController`, and`@Configuration`。`@Repository`,`@Service`等等注解都基于`@Component`的更精细化的组件注解。

### 组合注解

组合注解是一种被一个或者多个元注解标注过的注解，用以撮合多个元注解的特性到新的注解。例如，叫作`@TransactionalService`的注解就是被spring的`@Transactional`和`@Service`共同标注过，而且它把`@Transactional`和`@Service`的特性结合到了一起。另外，从技术上上讲，`@TransactionalService`也是一个常见的Stereotype注解。

### 注解标注形式

一个注解无论是直接标注还是间接标注一个bean，这个注解在java8的`java.lang.reflect.AnnotatedElement`类注释中所约定的含义和特性都不会有任何改变。  
在Spring中，如果一个元注解标注了其它注解，其它注解标注了一个bean，那么我们就说这个元注解_meta-present_（间接标注了）这个bean。以上文提到的`@TransactionalService`为例，我们可以说，`@Transactional`间接标注了任何一个标注过`@TransactionalService`的bean。

### 成员别名和覆盖

_**Attribute Alias**_\(成员别名\)将注解的一个成员名变为另一个。一个成员有多个别名时，这些别名是平等可交换的。成员别名可以分为以下几种。

* 1.**Explicit Aliases**（明确的别名）：如果一个注解中的两个成员通过`@AliasFor`声明后互为别名，那么它们是明确的别名。
* 2.**Implicit Aliases**（隐含别名）：如果一个注解中的两个或者更多成员通过`@AliasFor`声明去覆盖同一个元注解的成员值，它们就是隐含别名。
* 3.**Transitive Implicit Aliases**（传递的隐含别名）：如果一个注解中的两个或者更多成员通过`@AliasFor`
  声明去覆盖元注解中的不同成员，但是实际上因为[覆盖的传递性](https://en.wikipedia.org/wiki/Transitive_relation)导致最终覆盖的是元注解中的同一个成员，那么它们就是可传递的隐含别名。

_**Attribute Override**_（成员覆盖）是注解的一个成员覆盖另一个成员。成员覆盖可以分为以下几种。

* 1.**Implicit Overrides**（隐含的覆盖）：注解`@One`有成员A，注解`@Two`有也有成员A，如果`@One`本身是被元注解`@Two`标注的，那么按照命名约定，注解`@One`中的成员A实际会覆盖注解`@Two`中的成员A（用另一种方式说，两个看似不来自不同注解的成员A指向了同一个成员A）。
* 2.**Explicit Overrides**（明确的覆盖）：如果元注解的成员B通过`@AliasFor`声明其别名为成员A，那么成员A就是明确的覆盖了成员B。
* 3.**Transitive Explicit Overrides**（传递的明确覆盖）如果注解`@One`中的成员A明确覆盖了注解`@Two`中的成员B，而且成员B实际覆盖了注解`@Three`中的成员C，那么因为[覆盖的传递性](https://en.wikipedia.org/wiki/Transitive_relation)，所以成员A实际覆盖了成员C。

## 例子

### Spring Composed

_`译者注：Spring Composed项目是spring的一个社区项目，内含一些有特色的组合注解。可点击下文中的链接下载源码`_

[Spring Composed](https://github.com/sbrannen/spring-composed)项目是可以在在spring4.2.1和更高版本中使用的一系列组合注解。你可以在Spring MVC使用像`@Get`,`@Post`,`@Put`, 和`@Delet`这样的注解，也可以在Spring MVC REST应用中使用`@GetJson`,`@PostJson`等等注解。

如果你确信已经学习了足够深入的spring-composed例子，汲取了足够多的灵感，然后你可以为spring-Composed项目贡献出由你自定义的组合注解！

### 声明了`@AliasFor`的成员

Spring4.2引入了用以支持声明和查找注解成员别名的第一个版本。`@AliasFor`注解可以用来在一个注解中为一对成员互相声明别名，也可以在组合注解中为一个成员声明一个指向另一个元注解成员的别名。

例如，来自spring-test模块的`@ContextConfiguration`注解现在是这样定义的：

```
public @interface ContextConfiguration {

    @AliasFor("locations")
    String[] value() default {};

    @AliasFor("value")
    String[] locations() default {};

    // ...
}
```

同样的，组合注解可以使用`@AliasFor`来覆盖元注解的成员，这样可以在组合注解中进行精细的操作，而且注解与元注解之间的关系更有层次。实际上，这也提供了为元注解成员起别名的可操作方案。

例如，我们可以使用自己想要的成员名称去开发组合注解，然后再覆盖元注解的成员。如下。

```
@ContextConfiguration
public @interface MyTestConfig {
    @AliasFor(annotation = ContextConfiguration.class, attribute = "value")
    String[] xmlFiles();

    // ...
}
```

## FAQ

### 1\)`@AliasFor可以用于标注`@Component`和`@Qualifier\`的成员变量吗？

答案是不可以。

在`@Qualifier`和stereotype注解（例如`@Component`,`@Repository`,`@Controller`,和其它定制的stereotype注解）不能被`@AliasFor`影响。原因是这些注解的成员变量在`@AliasFor`被发明之前数年就已经存在了。因此，出于向后的兼容性考虑，这些注解的成员变量就不能使用`@AliasFor`。

## 附录

### 使用`@AliasFor`的注解

在Spring4.2版本中，以下注解使用了`@AliasFor`来为它们的成员声明别名。

* `org.springframework.cache.annotation.Cacheable`
* `org.springframework.cache.annotation.CacheEvict`
* `org.springframework.cache.annotation.CachePut`
* `org.springframework.context.annotation.ComponentScan.Filter`
* `org.springframework.context.annotation.ComponentScan`
  \`
* `org.springframework.context.annotation.ImportResource`
* `org.springframework.context.annotation.Scope`
* `org.springframework.context.event.EventListener`
* `org.springframework.jmx.export.annotation.ManagedResource`
* `org.springframework.messaging.handler.annotation.Header`
* `org.springframework.messaging.handler.annotation.Payload`
* `org.springframework.messaging.simp.annotation.SendToUser`
* `org.springframework.test.context.ActiveProfiles`
* `org.springframework.test.context.ContextConfiguration`
* `org.springframework.test.context.jdbc.Sql`
* `org.springframework.test.context.TestExecutionListeners`
* `org.springframework.test.context.TestPropertySource`
* `org.springframework.transaction.annotation.Transactional`
* `org.springframework.transaction.event.TransactionalEventListener`
* `org.springframework.web.bind.annotation.ControllerAdvice`
* `org.springframework.web.bind.annotation.CookieValue`
* `org.springframework.web.bind.annotation.CrossOrigin`
* `org.springframework.web.bind.annotation.MatrixVariable`
* `org.springframework.web.bind.annotation.RequestHeader`
* `org.springframework.web.bind.annotation.RequestMapping`
* `org.springframework.web.bind.annotation.RequestParam`
* `org.springframework.web.bind.annotation.RequestPart`
* `org.springframework.web.bind.annotation.ResponseStatus`
* `org.springframework.web.bind.annotation.SessionAttributes`
* `org.springframework.web.portlet.bind.annotation.ActionMapping`
* `org.springframework.web.portlet.bind.annotation.RenderMapping`

## 待发掘主题_`(译者注：课后习题-_-!)`_

* 记述 注解和标注了注解和元注解的类、接口、成员方法、成员变量、参数的通用搜索算法。
  * 如果一个注解既是以注解又是以元注解的方式标注了一个元素会发生什么呢？
  * 一个注解标注了`@Inherited`（包括自定义组合注解）后，是如何对搜索算法产生影响呢？
* 记述 通过`@AliasFor`配置注解成员别名的技术原理。
  * 如果一个成员和它的别名都声明在一个注解实例（成员和别名值相同或者不同时）中在技术上会发生什么？
    * 较有代表性的一种情况是，一个`AnnotationConfigurationException`将会被抛出。
* 记述_组合注解_的技术原理。
* 记述 组合注解成员覆盖元注解成员的原理。
  * 详细记述 查找成员的算法原理：
    * 基于命名约定的间接覆盖（换句话说，组合注解中有明确名字和类型的成员去覆盖元注解的成员）
    * 使用@AliasFor来直接覆盖
  * 如果一个成员和它的众多别名中的一个在注解继承的某一层级中被重新声明了会发生什么？哪个会生效？
  * 总之，成员变量声明时的冲突是怎么被解决的？



