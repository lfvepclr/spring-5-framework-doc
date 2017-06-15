# 2.介绍Spring框架

Spring 框架是一个Java平台，它为开发Java应用程序提供全面的基础架构支持。Spring负责基础架构，因此您可以专注于应用程序的开发。

Spring可以让您从“plain old Java objects”（POJO）中构建应用程序和通过非侵入性的POJO实现企业应用服务。此功能适用于Java SE的编程模型，全部的或部分的适应Java EE模型。

这些例子告诉你，作为一个应用程序开发人员，如何从Spring平台中受益：

* 写一个Java方法执行数据库事务，而无需处理具体事务的APIs。
* 写一个本地Java方法去远程调用，而不必处理远程调用的APIs。
* 写一个本地Java方法实现管理操作，而不必处理JMX APIs。
* 写一个本地Java方法实现消息处理，而不必处理JMS APIs。

## 2.1依赖注入和控制反转

Java应用程序-这是一个宽松的术语，它包括的范围从受限的嵌入式应用程序到n层的服务器端企业应用程序-通常组成程序的对象互相协作而构成正确的应用程序。因此，在一个应用程序中的对象彼此具有_依赖关系（dependencies）。_

虽然Java平台提供了丰富的应用程序开发功能，但它缺乏将基本的模块组织成一个整体的方法，而将该任务留给了架构师和开发人员。虽然你可以使用如_工厂_，_抽象工厂_，_Builder_，_装饰器_和_Service Locator等_ 设计模式来构建各种类和对象实例，使他们组合成应用程序，但这些模式无非只是：最佳实践赋予的一个名字，以及这是什么样的模式，应用于哪里，它能解决的问题等等。 模式是您必须在应用程序中自己实现的形式化的最佳实践。

Spring框架_控制反转_（IOC）组件通过提供一系列的标准化的方法把完全不同的组件组合成一个能够使用的应用程序来解决这个问题。Spring框架把形式化的设计模式编写为优秀的对象，你可以容易的集成到自己的应用程序中。许多组织和机构使用Spring框架，以这种方式\(使用Spring的模式对象\)来设计健壮的，_可维护的_应用程序。

**背景**

“ 现在的问题是，什么方面的控制被（他们）反转了？ ”马丁·福勒2004年在[他的网站](http://martinfowler.com/articles/injection.html)提出了这个有关控制反转（IOC）的问题 ，福勒建议重命名，使之能够自我描述，并提出了依赖注入\( _Dependency Injection_\)。

## 2.2模块

Spring框架的功能被有组织的分散到约20个模块中。这些模块分布在核心容器，数据访问/集成，Web，AOP（面向切面​​的编程），植入\(Instrumentation\)，消息传输和测试，如下面的图所示。

_图2.1 Spring框架概述_

![](/assets/spring-overview.png.pagespeed.ce.XVe1noRCMt.png)

以下部分列出了每个可用模块，以及它们的工件名称和它们支持的主要功能。工件的名字对应的是

_工件标识符，使用在_[依赖管理工具](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/overview.html#dependency-management)中。

### 2.2.1核心容器

[_核心容器_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-introduction)由以下模块组成，spring-core， spring-beans，spring-context，spring-context-support，和spring-expression （Spring表达式语言）。

spring-core和spring-beans模块[提供了框架的基础功能](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-introduction)，包括IOC和依赖注入功能。 BeanFactory是一个成熟的工厂模式的实现。你不再需要编程去实现单例模式，允许你把依赖关系的配置和描述从程序逻辑中解耦。

[_上下文_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#context-introduction)（spring-context）模块建立在由[_Core和Beans_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/beans.html#beans-introduction)模块提供的坚实的基础上：它提供一个框架式的对象访问方式，类似于一个JNDI注册表。上下文模块从Beans模块继承其功能，并添加支持国际化（使用，例如，资源集合），事件传播，资源负载，并且透明创建上下文，例如，Servlet容器。Context模块还支持Java EE的功能，如EJB，JMX和基本的远程处理。ApplicationContext接口是Context模块的焦点。 spring-context-support支持整合普通第三方库到Spring应用程序上下文，特别是用于高速缓存（ehcache，JCache）和调度（CommonJ，Quartz）的支持。

spring-expression模块提供了强大的[_表达式语言_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/expressions.html)去支持查询和操作运行时对象图。这是对JSP 2.1规范中规定的统一表达式语言（unified EL）的扩展。该语言支持设置和获取属性值，属性分配，方法调用，访问数组，集合和索引器的内容，逻辑和算术运算，变量命名以及从Spring的IoC容器中以名称检索对象。 它还支持列表投影和选择以及常见的列表聚合。

### 2.2.2 AOP和Instrumentation

spring-aop模块提供了一个符合[_AOP_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-introduction)联盟（要求）的面向方面的编程实现，例如，允许您定义方法拦截器和切入点（pointcuts），以便干净地解耦应该被分离的功能实现。 使用源级元数据\(source-level metadata\)功能，您还可以以类似于.NET属性的方式将行为信息合并到代码中。

单独的spring-aspects模块，提供了与AspectJ的集成。

spring-instrument模块提供了类植入\(instrumentation\)支持和类加载器的实现,可以应用在特定的应用服务器中。该spring-instrument-tomcat 模块包含了支持Tomcat的植入代理。

### 2.2.3消息

Spring框架4包括spring-messaging\(消息传递模块\)，其中包含来自Spring Integration的项目，例如，Message，MessageChannel，MessageHandler，和其他用来传输消息的基础应用。该模块还包括一组用于将消息映射到方法的注释\(annotations\)，类似于基于Spring MVC注释的编程模型。

### 2.2.4数据访问/集成

数据访问/集成层由JDBC，ORM，OXM，JMS和事务模块组成。

spring-jdbc模块提供了一个[JDBC](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jdbc.html#jdbc-introduction) –抽象层，消除了需要的繁琐的JDBC编码和数据库厂商特有的错误代码解析。

spring-tx模块支持用于实现特殊接口和所有POJO（普通Java对象）的类的[编程和声明式事务](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html) 管理。

spring-orm模块为流行的对象关系映射\([object-relational mapping](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/orm.html#orm-introduction) \)API提供集成层，包括JPA和Hibernate。使用spring-orm模块，您可以将这些O / R映射框架与Spring提供的所有其他功能结合使用，例如前面提到的简单声明性事务管理功能。

spring-oxm模块提供了一个支持[对象/ XML映射](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/oxm.html)实现的抽象层，如JAXB，Castor，JiBX和XStream。

spring-jms模块\([Java Messaging Service](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jms.html)\) 包含用于生产和消费消息的功能。自Spring Framework 4.1以来，它提供了与 spring-messaging模块的集成。

### 2.2.5 Web

Web层由spring-web，spring-webmvc和spring-websocket 模块组成。

spring-web模块提供基本的面向Web的集成功能，例如多部分文件上传功能，以及初始化一个使用了Servlet侦听器和面向Web的应用程序上下文的IoC容器。它还包含一个HTTP客户端和Spring的远程支持的Web相关部分。

spring-webmvc模块（也称为Web-Servlet模块）包含用于Web应用程序的Spring的模型-视图-控制器\([_MVC_](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-introduction)\)和REST Web Services实现。 Spring的MVC框架提供了领域模型代码和Web表单之间的清晰分离，并与Spring Framework的所有其他功能集成。

### 2.2.6测试

spring-test模块支持使用JUnit或TestNG对Spring组件进行[单元测试](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/unit-testing.html)和 [集成测试](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html)。它提供了Spring ApplicationContexts的一致[加载](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testcontext-ctx-management)和这些上下文的[缓存](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/integration-testing.html#testcontext-ctx-management-caching)。它还提供可用于独立测试代码的[模仿\(mock\)对象](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/unit-testing.html#mock-objects)。

## 2.3使用场景

之前描述的构建模块使Spring成为许多应用场景的理性选择，从在资源受限设备上运行的嵌入式应用程序到使用Spring的事务管理功能和Web框架集成的全面的企业应用程序。

_图2.2 典型的成熟完整的Spring Web应用程序_

![](/assets/overview-full.png.pagespeed.ce.sC26wirtWB.png)

Spring的[声明式事务管理功能](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative)使Web应用程序完全事务性，就像使用EJB容器管理的事务一样。所有您的定制业务逻辑都可以使用简单的POJO实现，并由Spring的IoC容器进行管理。附加服务包括支持发送电子邮件和独立于Web层的验证，可让您选择执行验证规则的位置。 Spring的ORM支持与JPA和Hibernate集成;例如，当使用Hibernate时，可以继续使用现有的映射文件和标准的Hibernate SessionFactory配置。表单控制器将Web层与域模型无缝集成，从而无需ActionForms或将HTTP参数转换为域模型的值的其他类。

_图2.3 使用第三方web框架的Spring中间层_

![](/assets/overview-thirdparty-web.png.pagespeed.ce.1lJso2G8WP.png)

有时情况不允许你完全切换到不同的框架。 Spring框架并不强制您使用其中的一切;这不是一个_全有或全无的_解决方案。使用Struts，Tapestry，JSF或其他UI框架构建的现有前端可以与基于Spring的中间层集成，从而允许您使用Spring事务功能。您只需要使用ApplicationContext连接您的业务逻辑，并使用WebApplicationContext来集成您的Web层。

_图2.4 远程使用场景_

![](/assets/overview-remoting.png.pagespeed.ce.HIMsJb_Xya.png)

当您需要通过Web服务访问现有代码时，你可以使用Spring的 Hessian-，Rmi-或HttpInvokerProxyFactoryBean类。启用对现有应用程序的远程访问并不困难。

_图2.5  EJBs – 包装现有的POJOs_

![](/assets/overview-ejb.png.pagespeed.ce.VN88UiKUhA.png)

Spring Framework还为Enterprise JavaBeans提供了一个[访问和抽象层](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/ejb.html)，使您能够重用现有的POJO，并将其包装在无状态会话bean中，以便在可能需要声明式安全性的, 可扩展的，故障安全的Web应用程序中使用。

### 2.3.1依赖管理和命名约定

依赖关系管理和依赖注入是不同的。为了将Spring的这些不错的功能（如依赖注入）集成到应用程序中，您需要组装所有需要的库（jar文件），并在运行时导入到类路径（classpath）中，也有可能在编译时就需要加入类路径。这些依赖关系不是注入的虚拟组件，而是文件系统中的物理资源（通常是这样）。依赖关系管理的过程包括定位这些资源，存储它们并将其添加到类路径中。依赖关系可以是直接的（例如，我的应用程序在运行时依赖于Spring）或间接的（例如我的应用程序依赖于commons-dbcp ，而commons-dbcp 又依赖于commons-pool）。间接依赖关系具有“传递性”，它们是最难识别和管理的依赖关系。

如果你要使用Spring，你需要获得一个包含你所需要的Spring模块的jar库的副本。为了使这更容易，Spring被打包为一组尽可能分离依赖关系的模块，例如，如果您不想编写Web应用程序，则不需要spring-web模块。要引用本指南中的Spring库模块，我们使用一个简写命名约定spring- \*或spring – \*.jar，其中\*表示模块的简称（例如spring-core，spring-webmvc，spring-jms等） ）。您实际使用的jar文件名通常是与版本号连接的模块名称（例如spring-core-5.0.0.M5.jar）。

Spring框架的每个版本都会发布到以下几个地方：

* Maven Central，它是Maven查询的默认存储库，不需要任何特殊配置。 Spring的许多常见的库也可以从Maven Central获得，Spring社区的大部分使用Maven进行依赖关系管理，所以这对他们来说很方便。这里的jar的名字是spring – \* – &lt;version&gt; .jar，Maven groupId是org.springframework。
* 在专门用于Spring的公共Maven存储库中。除了最终的GA版本，该存储库还承载开发快照和里程碑版本。 jar文件名与Maven Central格式相同，因此这是一个有用的地方，可以将开发中的版本的Spring与在Maven Central中部署的其他库配合使用。该存储库还包含集中分发的zip文件，其中包含所有Spring jar，捆绑在一起以便于下载。

所以你需要决定的第一件事是如何管理你的依赖关系：我们通常建议使用像Maven，Gradle或Ivy这样的自动化系统，但你也可以通过自己下载所有的jar来手动执行。

您将在下面找到Spring artifacts列表。有关每个模块的更完整的描述，[第2.2节“模块”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/overview.html#overview-modules).

_表2.1  Spring框架的Artifacts_

| GroupId | **ArtifactId** | **Description\(描述\)** |
| :--- | :--- | :--- |
| org.springframework | spring-aop | Proxy-based AOP support |
| org.springframework | spring-aspects | AspectJ based aspects |
| org.springframework | spring-beans | Beans support, including Groovy |
| org.springframework | spring-context | Application context runtime, including scheduling and remoting abstractions |
| org.springframework | spring-context-support | Support classes for integrating common third-party libraries into a Spring application context |
| org.springframework | spring-core | Core utilities, used by many other Spring modules |
| org.springframework | spring-expression | Spring Expression Language \(SpEL\) |
| org.springframework | spring-instrument | Instrumentation agent for JVM bootstrapping |
| org.springframework | spring-instrument-tomcat | Instrumentation agent for Tomcat |
| org.springframework | spring-jdbc | JDBC support package, including DataSource setup and JDBC access support |
| org.springframework | spring-jms | JMS support package, including helper classes to send and receive JMS messages |
| org.springframework | spring-messaging | Support for messaging architectures and protocols |
| org.springframework | spring-orm | Object/Relational Mapping, including JPA and Hibernate support |
| org.springframework | spring-oxm | Object/XML Mapping |
| org.springframework | spring-test | Support for unit testing and integration testing Spring components |
| org.springframework | spring-tx | Transaction infrastructure, including DAO support and JCA integration |
| org.springframework | spring-web | Web support packages, including client and web remoting |
| org.springframework | spring-webmvc | REST Web Services and model-view-controller implementation for web applications |
| org.springframework | spring-websocket | WebSocket and SockJS implementations, including STOMP support |

#### Spring的依赖和依赖于Spring

虽然Spring为大量企业和其他外部工具提供集成和支持，但它有意将其强制性的依赖保持在最低限度：在使用Spring用于简单的用例时，您不必定位和下载（甚至自动的去做）大量的jar库。对于基本依赖注入功能，只有一个强制性的外部依赖关系，也就是用于日志记录的依赖（有关日志记录选项的更详细描述，请参阅下文）。

接下来，我们概述了配置依赖于Spring的应用程序所需的基本步骤，首先是使用Maven，然后使用Gradle，最后使用Ivy。在任何情况下，如果不清楚，请参阅依赖关系管理系统的文档，或查看一些示例代码 – Spring本身在构建时使用Gradle来管理依赖关系，我们的示例主要使用Gradle或Maven。

#### Maven的依赖管理

如果您使用Maven进行依赖关系管理，则甚至不需要显式提供依赖关系。 例如，要创建应用程序上下文并使用依赖注入来配置应用程序，您的Maven依赖配置如下所示：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.0.M5</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

（依赖配置）就是这样。注意，如果您不需要针对Spring API进行编译，那么范围\(scope\)可以被声明为运行时，通常情况下这是基本的依赖注入用例。

以上示例适用于Maven Central存储库。要使用Spring Maven仓库（例如，使用里程碑或开发中的快照版本），您需要在Maven配置中指定仓库位置。完整版本：

```
<repositories>
    <repository>
        <id>io.spring.repo.maven.release</id>
        <url>http://repo.spring.io/release/</url>
        <snapshots><enabled>false</enabled></snapshots>
    </repository>
</repositories>
```

对于里程碑\(milestones\)：

```
<repositories>
    <repository>
        <id>io.spring.repo.maven.milestone</id>
        <url>http://repo.spring.io/milestone/</url>
        <snapshots><enabled>false</enabled></snapshots>
    </repository>
</repositories>
```

而对于快照（snapshots）：

```
<repositories>
    <repository>
        <id>io.spring.repo.maven.snapshot</id>
        <url>http://repo.spring.io/snapshot/</url>
        <snapshots><enabled>true</enabled></snapshots>
    </repository>
</repositories>
```

#### Maven的“材料清单”依赖

使用Maven时，可能会意外混合不同版本的Spring JAR。例如，您可能会发现第三方库或另一个Spring项目会传递依赖于旧版本的Spring JARs。如果您忘记自己明确声明直接依赖，可能会出现各种意外问题。

为了克服这些问题，Maven支持“材料清单\(bill of materials\)”（BOM）依赖的概念。您可以在dependencyManagement部分中导入spring-framework-bom，以确保所有Spring依赖（直接和传递）都是相同的版本

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>5.0.0.M5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

使用BOM的另外一个好处是，您不再需要在依赖于Spring Framework artifacts时指定&lt;version&gt;属性：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
<dependencies>
```

#### Gradle 依赖管理

要使用[Gradle](http://www.gradle.org/) 构建系统的Spring仓库，请在仓库部分中包含适当的URL：

```
repositories {
    mavenCentral()
    // and optionally...
    maven { url "http://repo.spring.io/release" }
}
```

您可以根据需要将repositoriesURL从/release更改为/milestone或/snapshot。一旦repositories被配置，你可以通常使用Gradle方式来声明依赖：

```
dependencies {
    compile("org.springframework:spring-context:5.0.0.M5")
    testCompile("org.springframework:spring-test:5.0.0.M5")
}
```

#### Ivy依赖管理

如果您喜欢使用 [Ivy](https://ant.apache.org/ivy) 来管理依赖关系，那么还有类似的配置选项。 要配置Ivy指向Spring仓库，请将以下解析器添加到您的ivysettings.xml中：

```
<resolvers>
    <ibiblio name="io.spring.repo.maven.release"    m2compatible="true" root="http://repo.spring.io/release/"/>
</resolvers>
```

您可以更改root从URL /release/到/milestone/或/snapshot/适当。

配置完成后，您可以在通常的方式添加依赖。例如（在ivy.xml）：

您可以根据需要将rootURL从 /release/更改为/milestone/或/snapshot/。 配置完成后，您可以按通常的方式添加依赖项。例如（在ivy.xml中）：

```
<dependency org="org.springframework"    name="spring-core" rev="5.0.0.M5" conf="compile->runtime"/>
```

#### Zip文件发行

虽然使用依赖关系管理的构建系统是推荐的获取Spring框架的方法，但仍然可以下载发布的zip文件。

Zip文件发布到Spring Maven存储库（这仅仅是为了我们的方便，您不需要使用Maven或任何其他构建系统才能下载它们）。

要下载发布的zip文件，打开Web浏览器到[http://repo.spring.io/release/org/springframework/spring](http://repo.spring.io/release/org/springframework/spring)，并为所需的版本选择相应的子文件夹。zip文件以-dist.zip结尾，例如spring-framework- {spring-version} -RELEASE-dist.zip。[里程碑](http://repo.spring.io/milestone/org/springframework/spring)和 [快照](http://repo.spring.io/snapshot/org/springframework/spring)也会发布在这里。

### 2.3.2 日志

日志是Spring非常重要的依赖，因为a）它是唯一的强制性外部依赖关系，b）每个人都喜欢看到他们使用的工具的一些输出，以及c）Spring集成了许多其他工具，都会具有日志依赖关系。应用程序开发人员的目标之一通常是将统一的日志配置放在整个应用程序的中央位置，包括所有外部组件。这比以前有更多的困难，因为有这么多的日志框架可以选择。

Spring中的强制性日志依赖关系是Jakarta Commons Logging API（JCL）。我们针对JCL进行编译，我们还使JCL Log 对象对于扩展了Spring Framework的类可见。对于用户来说，所有版本的Spring都使用相同的日志库很重要：迁移很简单，因为即使扩展了Spring的应用程序，但仍然保留了向后兼容性。我们这样做的方式是使Spring中的一个模块显式地依赖于commons-logging（遵循JCL规范的实现），然后在编译时使所有其他模块依赖于它。例如，如果您使用Maven，并且想知道在哪里可以获取对commons-logging的依赖，那么它来自Spring，特别是来自名为spring-core的中央模块。

commons-logging 的好处在于，您不需要任何其他操作来使您的应用程序正常工作。它具有运行时发现算法，可以在类路径中查找其他日志框架，并使用它认为合适的日志框架（或者您可以告诉它需要哪一个）。如果没有其他可用的，您可以从JDK（简称java.util.logging或JUL）获得好用的日志框架。您应该发现，在大多数情况下，您的Spring应用程序可以快乐地把日志输出到控制台，这很重要。

#### 不使用Commons Logging

不幸的是，commons-logging中的运行时发现算法虽然对最终用户很方便，但是也有很多问题。如果我们可以把时空倒回，现在开始使用Spring去开始一个新的项目，我们会使用不同的日志依赖关系。第一选择可能是Simple Logging Facade for Java（[SLF4J](http://www.slf4j.org/)），它也被许多其他使用Spring的应用工具所使用。

基本上有两种方法来关闭commons-logging：

1. 排除spring-core模块的依赖关系（因为它是明确依赖于commons-logging的唯一模块）
2. 依赖于一个特殊的commons-logging依赖关系，用空的jar代替库（更多的细节可以在
   [SLF4J FAQ](http://slf4j.org/faq.html#excludingJCL)
   中找到）

要排除commons-logging，请将以下内容添加到dependencyManagement部分：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0.M5</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

现在这个应用程序可能已经被破坏了，因为在类路径中没有实现JCL API，所以要修复它，必须提供一个新的JCL API。在下一节中，我们向您展示如何使用SLF4J提供JCL的替代实现。

#### 使用SLF4J

SLF4J是一个更清洁的依赖关系，在运行时比commons-logging更有效率，因为它使用编译时绑定，而不是其集成的其他日志框架的运行时发现。这也意味着你必须更加明确地说明你在运行时想要发生什么，并声明它或相应地进行配置。 SLF4J提供对许多常见的日志框架的绑定，因此通常可以选择一个已经使用的日志框架，并绑定到该框架进行配置和管理。

SLF4J提供了绑定到许多常见的日志框架的方法，包括JCL，它也是可以反转的：是其他日志框架和自身\(Spring\)之间的桥梁。所以要使用SLF4J与Spring，您需要使用SLF4J-JCL bridge替换commons-logging依赖关系。一旦完成，那么在Spring中日志调用将被转换为对SLF4J API的日志调用，因此如果应用程序中的其他库使用该API，那么您有一个统一的地方来配置和管理日志记录。

常见的选择可能是将Spring链接到SLF4J，然后提供从SLF4J到Log4j的显式绑定。您需要提供多个依赖关系（并排除现有的commons-logging）：the bridge，Log4j的SLF4J实现和Log4j实现本身。在Maven你会这样做：

```
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.0.0.M5</version>
        <exclusions>
            <exclusion>
                <groupId>commons-logging</groupId>
                <artifactId>commons-logging</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>1.7.22</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.7</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.7</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.7</version>
    </dependency>
</dependencies>
```

#### 使用的Log4j

> Log4j的1.x版本已经寿终正寝，以下的内容特指Log4j 2

许多人使用[Log4j](https://logging.apache.org/log4j) 作为配置和管理日志的日志框架。它是高效和成熟的，当我们构建和测试Spring，实际上它是在运行时使用的。 Spring还提供了一些用于配置和初始化Log4j的实用功能，因此它在某些模块中对Log4j具有可选的编译时依赖性。

要使用JCL和Log4j，所有你需要做的就是把Log4j加到类路径，并为其提供一个配置文件（log4j2.xml，log4j2.properties或其他 [支持的配置格式](https://logging.apache.org/log4j/2.x/manual/configuration.html)）。对于Maven用户，所需的最少依赖关系是：

```
<dependencies>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.7</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-jcl</artifactId>
        <version>2.7</version>
    </dependency>
</dependencies>
```

如果你也想使用SLF4J，还需要以下依赖关系：

```
<dependencies>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.7</version>
  </dependency>
</dependencies>
```

下面是一个例子log4j2.xml用来把日志输出到控制台：

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
  <Appenders>
    <Console name="Console" target="SYSTEM_OUT">
      <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
    </Console>
  </Appenders>
  <Loggers>
    <Logger name="org.springframework.beans.factory" level="DEBUG"/>
    <Root level="error">
      <AppenderRef ref="Console"/>
    </Root>
  </Loggers>
</Configuration>
```

#### 运行时容器和原生JCL

许多人在那些本身提供JCL实现的容器中运行他们的Spring应用程序。IBM WebSphere应用服务器（WAS）为例。这往往会导致问题，遗憾的是没有一劳永逸的解决方案; 在大多数情况下，简单地从您的应用程序排除commons-logging是不够的。

要清楚这一点：报告的问题通常不是JCL本身，甚至commons-logging：而是将commons-logging绑定到另一个框架（通常是Log4j）。这可能会失败，因为commons-logging更改了在一些容器中发现的旧版本（1.0）和大多数人现在使用的版本（1.1）之间执行运行时发现的方式。 Spring不使用JCL API的任何不寻常的部分，所以没有什么破坏，但是一旦Spring或您的应用程序尝试输出日志，您可以发现与Log4j的绑定不起作用

在这种情况下，使用WAS最简单的方法是反转类加载器层次结构（IBM将其称为”parent last”），以便应用程序控制JCL依赖关系，而不是容器。该选项并不总是开放的，但是在公共领域还有许多其他建议可供选择，您的里程\(集成程度\)可能因容器的确切版本和功能集而异。

