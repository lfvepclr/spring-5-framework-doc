## 21. 与其他Web框架集成

## 21.1 简介

### **Spring Web Flow**

Spring Web Flow \(SWF\) 旨在成为管理Web应用程序页面流的最佳解决方案。

SWF与Servlet和Portlet环境中的Spring MVC和JSF等现有框架集成。 如果您有一个业务流程（或流程）将受益于会话模型而不是纯粹的请求模型，则SWF可能是解决方案。

SWF允许您将逻辑页面流作为在不同情况下可重用的自包含模块捕获，因此非常适合构建引导用户通过驱动业务流程的受控导航的Web应用程序模块。

有关SWF的更多信息，请参阅[Spring Web Flow website](http://projects.spring.io/spring-webflow/).

本章详细介绍了Spring与第三方Web框架的集成，如 [JSF](http://www.oracle.com/technetwork/java/javaee/javaserverfaces-139869.html).

 Spring框架的核心价值主张之一是自由选择。在一般意义上，Spring并不强迫某人使用或购买任何特定的架构，技术或方法（尽管它肯定建议其他人使用）。这种选择与开发人员及其开发团队最相关的架构，技术或方法的自由可以说是在Web领域最为明显的一个领域，Spring领域同时提供了自己的Web框架（Spring MVC）提供与许多受欢迎的第三方网络框架的集成。这允许人们继续利用在特定的Web框架（如JSF）中获得的任何和所有技能，同时能够享受Spring在其他领域（如数据访问，声明式交易）管理，灵活配置和应用程序组装。

放弃了the woolly sales patter（c.f.前一段），本章的其余部分将集中在Web框架与Spring集成的细节。开发人员从其他语言开始Java经常评论的一件事就是Java中可用的Web框架的丰富程度。 Java中确实有大量的Web框架;事实上，在一个章节中有太多的东西可以覆盖任何细节的外表。本章从Java中选出了四个更受欢迎的Web框架，从所有支持的Web框架通用的Spring配置开始，然后详细说明每个支持的Web框架的特定集成选项。

> 请注意，本章不会尝试解释如何使用任何受支持的Web框架。例如，如果要将JSF用于Web应用程序的表示层，那么假设您已经熟悉了JSF本身。如果您需要有关任何支持的Web框架本身的更多详细信息，请参阅本章末尾 [Section 21.6, “Further Resources”](https://www.gitbook.com/book/lfvepclr/spring-framework-5-doc-cn/edit#) .

## 21.2 普通配置

在介绍每个支持的Web框架的集成细节之前，我们首先来看看Spring配置，这些配置不是特定于任何一个Web框架的。 （本节同样适用于Spring自己的Web框架，Spring MVC。）

（Spring’s）轻量级应用程序模型支持的一个概念（为了想要更好的词）就是分层架构。请记住，在“经典”分层架构中，网络层只是多层次之一;它作为服务器端应用程序的入口点之一，它委托服务层定义的服务对象（外观），以满足业务特定（和表示技术不可知）用例。在Spring中，这些服务对象，任何其他特定于业务的对象，数据访问对象等存在于不同的“业务上下文”中，其不包含web或表示层对象（诸如Spring MVC控制器之类的呈现对象通常在独特的“演示文稿”）。本节详细介绍了如何在一个应用程序中配置包含所有“business bean”的Spring容器（WebApplicationContext）。

具体细节：所有需要做的是在一个人的Web应用程序的标准Java EE servlet web.xml文件中声明一个ContextLoaderListener，并添加一个contextConfigLocation &lt;context-param/&gt;（在同一个文件中）来定义哪个集合的Spring XML配置文件加载。

在&lt;listener /&gt;配置下方查找：

```
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

在 &lt;context-param/&gt; 配置下方查找：

```
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>/WEB-INF/applicationContext*.xml</param-value>
</context-param>
```

如果不指定contextConfigLocation上下文参数，ContextLoaderListener将查找一个名为/WEB-INF/applicationContext.xml的文件加载。一旦加载了上下文文件，Spring将基于bean定义创建一个WebApplicationContext对象，并将其存储在Web应用程序的ServletContext中。

所有Java Web框架都构建在Servlet API之上，因此可以使用以下代码片段来访问由ContextLoaderListener创建的“business context”ApplicationContext。

```
WebApplicationContext ctx = WebApplicationContextUtils.getWebApplicationContext(servletContext);
```

The[`WebApplicationContextUtils`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/context/support/WebApplicationContextUtils.html)类是为了方便起见，因此您不必记住ServletContext属性的名称。如果WebApplicationContext.ROOT\_WEB\_APPLICATION\_CONTEXT\_ATTRIBUTE键下不存在一个对象，它的getWebApplicationContext（）方法将返回null。而不是在应用程序中获得NullPointerExceptions的风险，最好使用getRequiredWebApplicationContext（）方法。当ApplicationContext丢失时，此方法抛出异常。

一旦你引用了WebApplicationContext，你可以通过它们的名字或类型检索bean。大多数开发人员通过名称检索bean，然后将其转换为其实现的接口之一.

幸运的是，本节中的大部分框架具有更简单的查找bean的方法。它不仅使得从Spring容器获取bean变得容易，而且还允许您在其控制器上使用依赖注入。每个Web框架部分都有其具体整合策略的更多细节。

## 21.3 JavaServer Faces 1.2

JavaServer Faces（JSF）是JCP的标准组件，事件驱动的Web用户界面框架。 从Java EE 5开始，它是Java EE umbrella的官方部分.

对于受欢迎的JSF运行时以及流行的JSF组件库，请查看 [Apache MyFaces project](https://myfaces.apache.org/). MyFaces项目还提供常见的JSF扩展 ，比如[MyFaces Orchestra](https://myfaces.apache.org/orchestra/): 一个基于Spring的JSF扩展，可以提供丰富的会话范围支持。

> Spring Web Flow 2.0通过其新建立的Spring Faces模块提供丰富的JSF支持，无论是以JSF为中心的用途（如本节所述）和Spring中心使用（在Spring MVC调度程序中使用JSF视图）。 查看 [Spring Web Flow website](http://projects.spring.io/spring-webflow) 了解详情！

Spring的JSF集成中的关键要素是JSF ELResolver机制。

### 21.3.1 SpringBeanFacesELResolver \(JSF 1.2+\)

`SpringBeanFacesELResolver`是符合JSF 1.2的ELResolver实现, 与JSF 1.2和JSP 2.1使用的标准Unified EL集成像SpringBeanVariableResolver一样，它首先将Spring的“business context”WebApplicationContext委托给基于JSF实现的默认解析器.

在配置方面，只需在JSF 1.2 faces-context.xml文件中定义SpringBeanFacesELResolver：

```
<faces-config>
	<application>
		<el-resolver>org.springframework.web.jsf.el.SpringBeanFacesELResolver</el-resolver>
		...
	</application>
</faces-config>
```

### 21.3.2 FacesContextUtils

当将属性映射到faces-config.xml中的bean时，自定义VariableResolver可以正常工作，但有时可能需要明确地获取一个bean，The[`FacesContextUtils`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/jsf/FacesContextUtils.html)类使这个变得容易. 它类似于WebApplicationContextUtils，除了它需要一个FacesContext参数，而不是一个ServletContext参数.

```
ApplicationContext ctx = FacesContextUtils.getWebApplicationContext(FacesContext.getCurrentInstance());
```

## 21.4 Apache Struts 2.x

Craig McClanahan创建,[Struts](https://struts.apache.org/)是由Apache Software Foundation托管的一个开源项目。 当时，它大大简化了JSP / Servlet编程范例，并赢得了许多正在使用专有框架的开发人员。 它简化了编程模式，它是开放源代码（因此像啤酒一样自由），它拥有一个大型社区，这使得该项目在Java Web开发人员中成长和流行.

查看Strut [Spring Plugin](https://struts.apache.org/release/2.3.x/docs/spring-plugin.html) ，用于Struts附带的内置Spring集成。

## 21.5 Tapestry 5.x

查看 [Tapestry homepage](https://tapestry.apache.org/):

Tapestry是一个“面向组件的框架，用于在Java中创建动态，强大，高度可扩展的Web应用程序”。

虽然Spring拥有自已强大的 [powerful web layer](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html), 但是通过组合Tapestry用于Web用户界面和用于较低层的Spring容器来构建企业Java应用程序，有许多独特的优势。

有关更多信息，请查看 [integration module for Spring](https://tapestry.apache.org/integrating-with-spring-framework.html).

## 21.6 Further Resources

关于这章节各种Web框架的详解介绍，请点击下面的链接.

* The [JSF](http://www.oracle.com/technetwork/java/javaee/javaserverfaces-139869.html) homepage
* The [Struts](https://struts.apache.org/) homepage
* The [Tapestry](https://tapestry.apache.org/) homepage



