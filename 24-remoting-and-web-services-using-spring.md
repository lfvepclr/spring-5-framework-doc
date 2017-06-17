# 24. 使用Spring提供远程和WEB服务

# 24.1 介绍

Spring提供了使用多种技术实现远程访问支持的集成类。远程访问支持使得具有远程访问功能的服务开发变得相当简单，而这些服务由普通的 \(Spring\) POJO实现。目前，Spring支持以下几种远程技术：

* 远程方法调用（RMI）。通过使用RmiProxyFactoryBean和RmiServiceExporter，Spring同时支持传统的RMI（与java.rmi.Remote接口和java.rmi.RemoteException配合使用）和通过RMI调用器的透明远程调用（透明远程调用可以使用任何Java接口）。
* Spring的HTTP调用器。Spring提供了一个特殊的远程处理策略，允许通过HTTP进行Java序列化，支持任何Java接口（就像RMI调用器）。相应的支持类是HttpInvokerProxyFactoryBean和HttpInvokerServiceExporter。
* Hessian。通过HessianProxyFactoryBean和HessianServiceExporter，可以使用Caucho提供的基于HTTP的轻量级二进制协议来透明地暴露服务。
* JAX-WS。Spring通过JAX-WS为web服务提供远程访问支持。（JAX-WS: 从Java EE 5 和 Java 6开始引入，作为JAX-RPC的继承者\)
* JMS。通过JmsInvokerServiceExporter和JmsInvokerProxyFacotryBean类，使用JMS作为底层协议来提供远程服务。
* AMQP。Spring AMQP项目支持AMQP作为底层协议来提供远程服务。

在讨论Spring的远程服务功能时，我们将使用以下的域模型和对应的服务：

```
public class Account implements Serializable{
    private String name;

    public String getName(){
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}

public interface AccountService {

    public void insertAccount(Account account);

    public List<Account> getAccounts(String name);

}

// the implementation doing nothing at the moment
public class AccountServiceImpl implements AccountService {

    public void insertAccount(Account acc) {
        // do something...
    }

    public List<Account> getAccounts(String name) {
        // do something...
    }

}
```

我们将从使用RMI把服务暴露给远程客户端开始，同时讨论使用RMI的一些缺点。然后我们将继续演示一个使用Hessian的例子。

## 24.2 使用RMI暴露服务

使用Spring的RMI支持，你可以通过RMI基础架构透明地暴露你的服务。完成Spring的RMI设置后，你基本上具有类似于远程EJB配 置，除了没有对安全上下文传递和远程事务传递的标准支持。当使用RMI调用器时，Spring对这些额外的调用上下文提供了钩子，你可以在此插入安全框架 或者自定义的安全凭证。

### 24.2.1 使用RmiServiceExporter导出服务

使用RmiServiceExporter，我们可以把AccountService对象的接口暴露成RMI对象。可以使用RmiProxyFactoryBean或者在传统RMI服务中使用普通RMI来访问该接口。RmiServiceExporter明确支持使用RMI调用器暴露任何非RMI的服务。

当然，我们首先需要在Spring容器中设置我们的服务：

```
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>
```

下一步我们需要使用RmiServiceExporter来暴露我们的服务：

```
<bean class="org.springframework.remoting.rmi.RmiServiceExporter">
    <!-- does not necessarily have to be the same name as the bean to be exported -->
    <property name="serviceName" value="AccountService"/>
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
    <!-- defaults to 1099 -->
    <property name="registryPort" value="1199"/>
</bean>
```

正如你所见，我们覆盖了RMI注册的端口号。通常你的应用服务器还维护一个RMI注册表，明智的做法是不要和它冲突。此外，服务名是用来绑定服务的。现在服务绑定在‘rmi://HOST:1199/AccountService’。我们将在客户端使用这个URL来链接到服务。

> ```
> Note:servicePort属性被省略了(默认值为0).这表示在与服务通信时将使用匿名端口.
> ```

### 24.2.2 在客户端链接服务

我们的客户端是一个使用AccountService来管理account的简单对象：

```
public class SimpleObject {

    private AccountService accountService;

    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }

    // additional methods using the accountService

}
```

为了把服务链接到客户端上，我们将创建一个单独的Spring容器，包含这个简单对象和链接配置位的服务：

```
<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.rmi.RmiProxyFactoryBean">
    <property name="serviceUrl" value="rmi://HOST:1199/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

这就是我们为支持远程account服务在客户端所需要做的。Spring将透明地创建一个调用器并且通过RmiServiceExporter使得account服务支持远程服务。在客户端，我们用RmiProxyFactoryBean连接它。

## 24.3 使用Hessian通过HTTP远程调用服务

Hessian提供一种基于HTTP的二进制远程协议。它由Caucho开发的，可以在 [http://www.caucho.com](http://www.caucho.com) 找到更多有关Hessian的信息。

### 24.3.1 为Hessian和co.配置DispatcherServlet

Hessian使用一个自定义Servlet通过HTTP进行通讯。使用Spring的DispatcherServlet原理，从Spring Web MVC使用中可以看出，可以很容易的配置这样一个Servlet来暴露你的服务。首先我们要在你的应用里创建一个新的Servlet（以下摘录自web.xml）：

```
<servlet>
    <servlet-name>remoting</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>remoting</servlet-name>
    <url-pattern>/remoting/*</url-pattern>
</servlet-mapping>
```

你可能对Spring的DispatcherServlet很熟悉，这样你将需要在’WEB-INF’目录中创建一个名为’remoting-servlet.xml'\(在你的servlet名称后\) 的Spring容器配置上下文。这个应用上下文将在下一节中里使用。

或者，可以考虑使用Spring中更简单的HttpRequestHandlerServlet。这允许你在根应用上下文（默认是’WEB-INF/applicationContext.xml’）中嵌入远程exporter定义。每个servlet定义指向特定的exporter bean。在这种情况下，每个servlet的名称需要和目标exporter bean的名称相匹配。

### 24.3.2 使用HessianServiceExporter暴露你的bean

在新创建的remoting-servlet.xml应用上下文里，我们将创建一个HessianServiceExporter来暴露你的服务：

```
<bean id="accountService" class="example.AccountServiceImpl">
    <!-- any additional properties, maybe a DAO? -->
</bean>

<bean name="/AccountService" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

现在我们准备好在客户端连接服务了。不必显示指定处理器的映射，所以使用BeanNameUrlHandlerMapping把URL请求映射到服务上：因此，服务将通过其包含的bean名称指定的URL导出 DispatcherServlet’s mapping \(as defined above\): ’[http://HOST:8080/remoting/AccountService’](http://HOST:8080/remoting/AccountService’) 或者, 在你的根应用上下文中创建一个HessianServiceExporter\(比如在’WEB-INF/applicationContext.xml’中\):

```
<bean name="accountExporter" class="org.springframework.remoting.caucho.HessianServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

在后一情况下, 在’web.xml’中为这个导出器定义一个相应的servlet，也能得到同样的结果：这个导出器映射到request路径/remoting/AccountService。注意这个servlet名称需要与目标导出器bean的名称相匹配。

```
<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
```

### 24.3.3 在客户端上链接服务

使用HessianProxyFactoryBean，我们可以在客户端链接服务。与RMI示例一样也适用相同的原理。我们将创建一个单独的bean工厂或者应用上下文，并指明SimpleObject使用AccountService来管理accounts的以下bean：

```
<bean class="example.SimpleObject">
    <property name="accountService" ref="accountService"/>
</bean>

<bean id="accountService" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
    <property name="serviceUrl" value="http://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

### 24.3.4 对通过Hessian暴露的服务使用HTTP基本认证

Hessian的优点之一是，我们可以轻松应用HTTP基本身份验证，因为这两种协议都是基于HTTP的。你的正常HTTP 服务器安全机制可以通过使用web.xml安全功能来应用。通常，你不会为每个用户都建立不同的安全证书，而是在Hessian/BurlapProxyFactoryBean级别共享安全证书（类似一个JDBCDataSource）。

```
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    <property name="interceptors" ref="authorizationInterceptor"/>
</bean>

<bean id="authorizationInterceptor"
        class="org.springframework.web.servlet.handler.UserRoleAuthorizationInterceptor">
    <property name="authorizedRoles" value="administrator,operator"/>
</bean>
```

这个是我们显式使用了BeanNameUrlHandlerMapping的例子，并设置了一个拦截器，只允许管理员和操作员调用这个应用上下文中提及的bean。

> Note: 当然，这个例子并不表现出灵活的安全架构。有关安全性方面的更多选项，请查看Spring Security项目[http://projects.spring.io/spring-security/。](http://projects.spring.io/spring-security/。)

## 24.4 使用HTTP调用器暴露服务

与使用自身序列化机制的轻量级协议Hessian相反，Spring HTTP调用器使用标准Java序列化机制通过HTTP暴露业务。如果你的参数或返回值是复杂类型，并且不能通过Hessian的序列化机制进行序列化，HTTP调用器就很有优势（请参阅下一节，以便在选择远程处理技术时进行更多考虑）。

在底层，Spring使用JDK提供的标准工具或Commons的HttpComponents来实现HTTP调用。如果你需要更先进和更易用的功能，请使用后者。你可以参考 hc.apache.org/httpcomponents-client-ga/ 以获取更多信息。

### 24.4.1 暴露服务对象

为服务对象设置HTTP调用器基础架构类似于使用Hessian进行相同操作的方式。就象为Hessian支持提供的HessianServiceExporter，Spring的HTTP调用器提供了org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter。 为了在Spring Web MVC的DispatcherServlet中暴露AccountService\(之前章节提及过\)， 需要在调度程序的应用程序上下文中使用以下配置：

```
<bean name="/AccountService" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

如Hessian章节部分所述，这个导出器定义将通过DispatcherServlet的标准映射工具暴露出来。 或者， 在你的根应用上下文中\(比如’WEB-INF/applicationContext.xml’\)创建一个HttpInvokerServiceExporter:

```
<bean name="accountExporter" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

此外，在’web.xml’中为该导出器定义相应的servlet ，其中servlet名称与目标导出器的bean名称相匹配：

```
<servlet>
    <servlet-name>accountExporter</servlet-name>
    <servlet-class>org.springframework.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>

<servlet-mapping>
    <servlet-name>accountExporter</servlet-name>
    <url-pattern>/remoting/AccountService</url-pattern>
</servlet-mapping>
```

如果你在一个servlet容器之外运行程序和使用Oracle的Java6, 那么你可以使用内置的HTTP服务器实现。你可以配置SimpleHttpServerFactoryBean和SimpleHttpInvokerServiceExporter在一起，像下面这个例子一样：

```
<bean name="accountExporter"
        class="org.springframework.remoting.httpinvoker.SimpleHttpInvokerServiceExporter">
    <property name="service" ref="accountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>

<bean id="httpServer"
        class="org.springframework.remoting.support.SimpleHttpServerFactoryBean">
    <property name="contexts">
        <util:map>
            <entry key="/remoting/AccountService" value-ref="accountExporter"/>
        </util:map>
    </property>
    <property name="port" value="8080" />
</bean>
```

### 24.4.2 在客户端连接服务

同样，从客户端连接业务与你使用Hessian所做的很相似。使用代理，Spring可以将你的HTTP POST调用请求转换成被暴露服务的URL。

```
<bean id="httpInvokerProxy" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
    <property name="serviceUrl" value="http://remotehost:8080/remoting/AccountService"/>
    <property name="serviceInterface" value="example.AccountService"/>
</bean>
```

如前所述，你可以选择要使用的HTTP客户端。默认情况下，HttpInvokerProxy使用JDK的HTTP功能，但你也可以通过设置httpInvokerRequestExecutor属性来使用ApacheHttpComponents客户端：

```
<property name="httpInvokerRequestExecutor">
    <bean class="org.springframework.remoting.httpinvoker.HttpComponentsHttpInvokerRequestExecutor"/>
</property>
```

## 24.5 Web 服务

Spring提供了对标准Java Web服务API的全面支持：

* 使用JAX-WS暴露Web服务
* 使用JAX-WS访问Web服务

除了在Spring Core中支持 JAX-WS，Spring portfolio也提供了一种特性Spring Web Services，一种为契约优先和文档驱动的web服务所提供的方案，强烈建议用来创建现代化的，面向未来的web服务。

### 24.5.1使用JAX- WS暴露基于servlet的web服务

Spring为JAX-WS servlet的端点实现提供了一个方便的基类 – SpringBeanAutowiringSupport. 为了暴露我们的AccountService，我们扩展Spring的SpringBeanAutowiringSupport类并在这里实现了我们的业务逻辑，通常委派调用业务层。我们在Spring管理的bean里面简单地使用Spring的@Autowired 注解来表达这样的依赖关系。

```
/**
 * JAX-WS compliant AccountService implementation that simply delegates
 * to the AccountService implementation in the root web application context.
 *
 * This wrapper class is necessary because JAX-WS requires working with dedicated
 * endpoint classes. If an existing service needs to be exported, a wrapper that
 * extends SpringBeanAutowiringSupport for simple Spring bean autowiring (through
 * the @Autowired annotation) is the simplest JAX-WS compliant way.
 *
 * This is the class registered with the server-side JAX-WS implementation.
 * In the case of a Java EE 5 server, this would simply be defined as a servlet
 * in web.xml, with the server detecting that this is a JAX-WS endpoint and reacting
 * accordingly. The servlet name usually needs to match the specified WS service name.
 *
 * The web service engine manages the lifecycle of instances of this class.
 * Spring bean references will just be wired in here.
 */
import org.springframework.web.context.support.SpringBeanAutowiringSupport;

@WebService(serviceName="AccountService")
public class AccountServiceEndpoint extends SpringBeanAutowiringSupport {

    @Autowired
    private AccountService biz;

    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }

    @WebMethod
    public Account[] getAccounts(String name) {
        return biz.getAccounts(name);
    }

}
```

我们的AccountServletEndpoint需要和Spring在同一个上下文的web应用里运行，以允许访问Spring的功能。为JAX-WS servlet端点部署使用标准规约是Java EE 5 环境下的默认情况。

### 24.5.2 使用JAX-WS暴露单独web服务

Oracle JDK 1.6附带的内置JAX-WS provider 使用内置的HTTP服务器来暴露web服务。Spring的SimpleJaxWsServiceExporter类检测所有在Spring应用上下文中配置有@WebService注解的bean，然后通过默认的JAX-WS服务器（JDK 1.6 HTTP服务器）导出。

在这种场景下，端点实例将被作为Spring bean来定义和管理。它们将使用JAX-WS引擎来注册，但其生命周期将由Spring应用程序上下文决定。这意味着Spring的显示依赖注入可用于端点实例。当然通过@Autowired来进行注解驱动的注入也会起作用。

```
<bean class="org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
    <property name="baseAddress" value="http://localhost:8080/"/>
</bean>

<bean id="accountServiceEndpoint" class="example.AccountServiceEndpoint">
    ...
</bean>

...
```

AccountServiceEndpoint可能来自于Spring的SpringBeanAutowiringSupport，也可能不是。因为这里的端点是由Spring完全管理的bean。这意味着端点实现可能像下面这样没有任何父类定义 – 而且Spring的@Autowired配置注解仍然能够使用：

```
@WebService(serviceName="AccountService")
public class AccountServiceEndpoint {

    @Autowired
    private AccountService biz;

    @WebMethod
    public void insertAccount(Account acc) {
        biz.insertAccount(acc);
    }

    @WebMethod
    public List<Account> getAccounts(String name) {
        return biz.getAccounts(name);
    }

}
```

### 24.5.3 使用JAX-WS RI的Spring支持来暴露服务

Oracle的JAX-WS RI被作为GlassFish项目的一部分来开发,它使用了Spring支持来作为JAX-WS Commons项目的一部分。这允许把JAX-WS端点作为Spring管理的bean来定义。这与前面章节讨论的单独模式类似 – 但这次是在Servlet环境中。注意这在Java EE 5环境中是不可迁移的，建议在没有EE的web应用环境如Tomcat中嵌入JAX-WS RI。 与标准的暴露基于servlet的端点方式不同之处在于端点实例的生命周期将被Spring管理。这里在web.xml将只有一个JAX-WS servlet定义。在标准的Java EE 5风格中\(如上所示\)，你将对每个服务端点定义一个servlet，每个服务端点都代理到Spring bean \(通过使用@Autowired，如上所示\)。 关于安装和使用详情请查阅[https://jax-ws-commons.dev.java.net/spring/](https://jax-ws-commons.java.net/spring/)。

### 24.5.4 使用JAX-WS访问web服务

Spring提供了2个工厂bean来创建JAX-WS web服务代理，它们是LocalJaxWsServiceFactoryBean和JaxWsPortProxyFactoryBean。前一个只能返回一个JAX-WS服务对象来让我们使用。后面的是可以返回我们业务服务接口的代理实现的完整版本。这个例子中我们使用后者来为AccountService端点再创建一个代理：

```
<bean id="accountWebService" class="org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
    <property name="serviceInterface" value="example.AccountService"/>
    <property name="wsdlDocumentUrl" value="http://localhost:8888/AccountServiceEndpoint?WSDL"/>
    <property name="namespaceUri" value="http://example/"/>
    <property name="serviceName" value="AccountService"/>
    <property name="portName" value="AccountServiceEndpointPort"/>
</bean>
```

serviceInterface是我们客户端将使用的远程业务接口。wsdlDocumentUrl是WSDL文件的URL. Spring需要用它作为启动点来创建JAX-WS服务。namespaceUri对应.wsdl文件中的targetNamespace。serviceName对应.wsdl文件中的服务名。portName对应.wsdl文件中的端口号。 现在我们可以很方便的访问web服务，因为我们有一个可以将它暴露为AccountService接口的bean工厂。我们可以在Spring中这样使用：

```
<bean id="client" class="example.AccountClientImpl">
    ...
    <property name="service" ref="accountWebService"/>
</bean>
```

从客户端代码上我们可以把这个web服务当成一个普通的类进行访问：

```
public class AccountClientImpl {

    private AccountService service;

    public void setService(AccountService service) {
        this.service = service;
    }

    public void foo() {
        service.insertAccount(...);
    }
}
```

> Note: 上面例子被稍微简化了，因为JAX-WS需要端点接口及实现类来使用@WebService,@SOAPBinding等注解。 这意味着你不能简单地使用普通的Java接口和实现来作为JAX-WS端点，你需要首先对它们进行相应的注解。这些需求详情请查阅JAX-WS文档。

## 24.6 JMS

使用JMS来作为底层的通信协议透明暴露服务也是可能的。Spring框架中对JMS的远程支持也很基础 – 它在同一线程和同一个非事务 Session上发送和接收，这些吞吐量将非常依赖于实现。需要注意的是这些单线程和非事务的约束仅适用于Spring的JMS远程处理支持。请参见 第26章, JMS \(Java消息服务\)，Spring对基于JMS的消息的丰富支持。 下面的接口可同时用在服务端和客户端。

```
package com.foo;

public interface CheckingAccountService {

    public void cancelAccount(Long accountId);

}
```

对于上面接口的使用在服务的端简单实现如下：

```
package com.foo;

public class SimpleCheckingAccountService implements CheckingAccountService {

    public void cancelAccount(Long accountId) {
        System.out.println("Cancelling account [" + accountId + "]");
    }

}
```

这个包含JMS设施的bean的配置文件可同时用在客户端和服务端： &lt;?xml version=”1.0″ encoding=”UTF-8″?&gt;

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="tcp://ep-t43:61616"/>
    </bean>

    <bean id="queue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg value="mmm"/>
    </bean>

</beans>
```

### 24.6.1 服务端配置

在服务端你只需要使用JmsInvokerServiceExporter来暴露服务对象。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerServiceExporter">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="service">
            <bean class="com.foo.SimpleCheckingAccountService"/>
        </property>
    </bean>

    <bean class="org.springframework.jms.listener.SimpleMessageListenerContainer">
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="destination" ref="queue"/>
        <property name="concurrentConsumers" value="3"/>
        <property name="messageListener" ref="checkingAccountService"/>
    </bean>

</beans>
```

```
package com.foo;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Server {

    public static void main(String[] args) throws Exception {
        new ClassPathXmlApplicationContext(new String[]{"com/foo/server.xml", "com/foo/jms.xml"});
    }

}
```

### 24.6.2 客户端配置

客户端只需要创建一个客户端代理来实现上面的接口\(CheckingAccountService\)。根据后面的bean定义创建的结果对象可以被注入到其它客户端对象中，而这个代理会负责通过JMS将调用转发到服务端。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="checkingAccountService"
            class="org.springframework.jms.remoting.JmsInvokerProxyFactoryBean">
        <property name="serviceInterface" value="com.foo.CheckingAccountService"/>
        <property name="connectionFactory" ref="connectionFactory"/>
        <property name="queue" ref="queue"/>
    </bean>

</beans>
```

```
package com.foo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Client {

    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext(
                new String[] {"com/foo/client.xml", "com/foo/jms.xml"});
        CheckingAccountService service = (CheckingAccountService) ctx.getBean("checkingAccountService");
        service.cancelAccount(new Long(10));
    }

}
```

## 24.7 AMQP

有关更多信息，请参考[Spring AMQP Reference Document ‘Spring Remoting with AMQP’ section](http://docs.spring.io/spring-amqp/docs/current/reference/html/_reference.html#remoting)。

## 24.8 不实现远程接口自动检测

对远程接口不实现自动探测的主要原因是为了避免向远程调用者打开了太多的大门。目标对象有可能实现的是类似InitializingBean或者DisposableBean这样的内部回调接口，而这些是不希望暴露给调用者的。

提供一个所有接口都被目标实现的代理通常和本地情况无关。但是当暴露一个远程服务时，你应该只暴露特定的用于远程使用的服务接口。除了内部回调接口，目标有可能实现了多个业务接口，而往往只有一个是用于远程调用的。出于这些原因，我们要求指定这样的服务接口。

这是在配置方便性和意外暴露内部方法的危险性之间作的权衡。始终指定一个服务接口并不需要花太大代价，并可以令控制具体方法暴露更加安全。

## 24.9 选择技术时的注意事项

这里提到的每种技术都有它的缺点。你在选择一种技术时，应该仔细考虑你的需要和所暴露的服务及你在远程访问时传送的对象。

当使用RMI时，通过HTTP协议访问对象是不可能的，除非你正在HTTP通道传输RMI流量。RMI是一种重量级协议，因为它支持整个对象的序列化，当要求网络上传输复杂数据结构时这是非常重要的。然而，RMI-JRMP与Java客户端相关：它是一种Java-to-Java的远程访问解决方案。

如果你需要基于HTTP的远程访问而且还要求使用Java序列化，Spring的HTTP调用器是一个不错的选择。它和RMI调用器共享相同的基础设施，只需使用HTTP作为传输。注意HTTP调用器不仅限于Java-to-Java的远程访问，而且还限于使用Spring的客户端和服务器端。（后者也适用于Spring的RMI调用器，用于非RMI接口。）

Hessian可以在异构环境中运行时提供重要的价值，因为它们明确允许非Java客户端。然而，非Java支持仍然有限。已知问题包括将Hibernate对象与延迟初始化的集合相结合的序列化。如果您有这样的数据模型，请考虑使用RMI或HTTP调用者而不是Hessian。

在使用服务集群和需要JMS代理（JMS broker）来处理负载均衡及发现和自动-失败恢复服务时JMS是很有用的。缺省情况下，在使用JMS远程服务时使用Java序列化，但是JMS提供者也可以使用不同的机制例如XStream来让服务器用其他技术。

最后但并非最不重要的是，EJB比RMI具有优势，因为它支持标准的基于角色的身份认证和授权，以及远程事务传递。用RMI调用器或HTTP调用器来支持安全上下文的传递是可能的，虽然这不由核心core Spring提供：Spring提供了合适的钩子来插入第三方或定制的解决方案。

## 24.10 在客户端访问RESTful服务

RestTemplate是客户端访问RESTful服务的核心类。它在概念上类似于Spring中的其他模板类，例如JdbcTemplate、 JmsTemplate和其他Spring组合项目中发现的其他模板类。

RestTemplate’s behavior is customized by providing callback methods and configuring the \`HttpMessageConverter用于将对象打包到HTTP请求体中，并将任何响应解包成一个对象。通常使用XML作为消息格式，Spring提供了MarshallingHttpMessageConverter，它使用了的Object-to-XML框架，也是org.springframework.oxm包的一部分。这为你提供了各种各样的XML到对象映射技术的选择。

本节介绍如何使用RestTemplate它及其关联 的HttpMessageConverters。

### 24.10.1 RestTemplate

在Java中调用RESTful服务通常使用助手类（如Apache HttpComponents）完成HttpClient。对于常见的REST操作，此方法的级别太低，如下所示。

```
String uri = "http://example.com/hotels/1/bookings";

PostMethod post = new PostMethod(uri);
String request = // create booking request content
post.setRequestEntity(new StringRequestEntity(request));

httpClient.executeMethod(post);

if (HttpStatus.SC_CREATED == post.getStatusCode()) {
    Header location = post.getRequestHeader("Location");
    if (location != null) {
        System.out.println("Created new booking at :" + location.getValue());
    }
}
```

RestTemplate提供了更高级别的方法，这些方法与六种主要的HTTP方法中的每一种相对应，这些方法使得调用许多RESTful服务成为一个单行和执行REST的最佳实践。

Note: RestTemplate具有异步计数器部分：请参见[第24.10.3节“异步RestTemplate”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-async-resttemplate)。

Table 24.1. RestTemplate方法概述

| HTTP Method | RestTemplate Method |
| :--- | :--- |
| DELETE | [delete](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#delete%28String, Object…​%29) |
| GET | [getForObject](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#getForObject%28String, Class, Object…​%29) [getForEntity](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#getForEntity%28String, Class, Object…​%29) |
| HEAD | [headForHeaders\(String url, String…​ uriVariables\)](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#headForHeaders%28String, Object…​%29) |
| OPTIONS | [optionsForAllow\(String url, String…​ uriVariables\)](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#optionsForAllow%28String, Object…​%29) |
| POST | [postForLocation\(String url, Object request, String…​ uriVariables\)](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#postForLocation%28String, Object, Object…​%29) [postForObject\(String url, Object request, Class&lt;T&gt; responseType, String…​ uriVariables\)](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#postForObject%28java.lang.String, java.lang.Object, java.lang.Class, java.lang.String…​%29) |
| PUT | [put\(String url, Object request, String…​uriVariables\)](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#put%28String, Object, Object…​%29) |
| PATCH and others | [exchange](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#exchange%28java.lang.String, org.springframework.http.HttpMethod, org.springframework.http.HttpEntity, java.lang.Class, java.lang.Object…​%29) [execute](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/RestTemplate.html#execute%28java.lang.String, org.springframework.http.HttpMethod, org.springframework.web.client.RequestCallback, org.springframework.web.client.ResponseExtractor, java.lang.Object…​%29) |

RestTemplate方法名称遵循命名约定，第一部分指出正在调用什么HTTP方法，第二部分指出返回的内容。例如，该方法getForObject\(\)将执行GET，将HTTP响应转换为你选择的对象类型并返回该对象。方法postForLocation\(\) 将执行POST，将给定对象转换为HTTP请求，并返回可以找到新创建的对象的响应HTTP Location头。在异常处理HTTP请求的情况下，RestClientException类型的异常将被抛出; 这个行为可以在RestTemplate通过插入另一个ResponseErrorHandler实现来改变。

exchange和execute方法是上面列出的更具体的方法的广义版本，并且可以支持额外的组合和方法，例如HTTP PATCH。但是，请注意，底层HTTP库还必须支持所需的组合。JDK HttpURLConnection不支持该PATCH方法，但Apache HttpComponents HttpClient4.2或更高版本支持。他们还能够通过使用一个能够捕获和传递通用类型信息的新类ParameterizedTypeReference来使得RestTemplate能够读取通用类型的HTTP响应信息（例如List）。

对象通过HttpMessageConverter实例传递给这些方法并从这些方法返回被转换为HTTP消息。主要mime类型的转换器默认注册，但你也可以编写自己的转换器并通过messageConverters\(\)实体属性注册它 。模板默认注册的转换器实例是ByteArrayHttpMessageConverter，StringHttpMessageConverter，FormHttpMessageConverter和SourceHttpMessageConverter。如果使用MarshallingHttpMessageConverter或者MappingJackson2HttpMessageConverter，你可以使用messageConverters\(\)实体属性覆盖这些默认值。

每个方法以两种形式使用URI模板参数，作为String可变长度参数或Map&lt;String,String&gt;。例如，使用可变长参数如下：

```
String result = restTemplate.getForObject(
    "http://example.com/hotels/{hotel}/bookings/{booking}", String.class,"42", "21");
```

使用一个Map&lt;String,String&gt;如下：

```
Map<String, String> vars = Collections.singletonMap("hotel", "42");
String result = restTemplate.getForObject(
        "http://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
```

要创建一个实例，RestTemplate可以简单地调用默认的无参数构造函数。这将使用java.net包中的标准Java类作为底层实现来创建HTTP请求。这可以通过指定实现来覆盖ClientHttpRequestFactory。Spring提供了HttpComponentsClientHttpRequestFactory使用Apache HttpComponents HttpClient创建请求的实现。HttpComponentsClientHttpRequestFactory通过使用一个可以配置凭证信息或连接池功能的org.apache.http.client.HttpClient实例来配置。

> Note: HTTP请求的java.net实现可能会在访问表示错误的响应状态（例如401）时引发异常。如果这是一个问题，请切换到HttpComponentsClientHttpRequestFactory。

前面使用Apache HttpCOmponentsHttpClientdirectly的例子用RestTemplate重写如下：

```
uri = "http://example.com/hotels/{id}/bookings";

RestTemplate template = new RestTemplate();

Booking booking = // create booking object

URI location = template.postForLocation(uri, booking, "1");
```

使用Apache HttpComponents, 而不是原生的java.net功能，构造RestTemplate如下：

```
RestTemplate template = new RestTemplate(new HttpComponentsClientHttpRequestFactory());
```

> Note: Apache HttpClient 支持gzip编码，要使用这个功能，构造HttpCOmponentsClientHttpRequestFactory如下：
>
> ```
> HttpClient httpClient = HttpClientBuilder.create().build();
> ClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
> RestTemplate restTemplate = new RestTemplate(requestFactory);
> ```

当execute方法被调用，通用的回调接口是RequestCallback并且会被调用。

```
public <T> T execute(String url, HttpMethod method, RequestCallback requestCallback,
        ResponseExtractor<T> responseExtractor, String... uriVariables)

// also has an overload with uriVariables as a Map<String, String>.
```

RequestCallback接口定义如下：

```
public interface RequestCallback {
 void doWithRequest(ClientHttpRequest request) throws IOException;
}
```

允许您操作请求标头并写入请求主体。当使用execute方法时，你不必担心任何资源管理，模板将始终关闭请求并处理任何错误。有关使用execute方法及它的其他方法参数的含义的更多信息，请参阅API文档。

#### 使用URI

对于每个主要的HTTP方法，RestTemplate提供的变体使用String URI或java.net.URI作为第一个参数。  
String URI变体将模板参数视为String变长参数或者一个Map&lt;String,String&gt;。他们还假定URL字符串不被编码且需要编码。例如：

```
restTemplate.getForObject("http://example.com/hotel list", String.class);
```

将在 [http://example.com/hotel list执行一个GET请求。这意味着如果输入的URL字符串已被编码，它将被编码两次](http://example.com/hotel list执行一个GET请求。这意味着如果输入的URL字符串已被编码，它将被编码两次) – 即将 [http://example.com/hotel list变为http://example.com/hotel list。如果这不是预期的效果，则使用java.net.URI方法变体，假设URL已经被编码，如果要重复使用单个（完全扩展）URI多次，通常也是有用的。](http://example.com/hotel list变为http://example.com/hotel list。如果这不是预期的效果，则使用java.net.URI方法变体，假设URL已经被编码，如果要重复使用单个（完全扩展）URI多次，通常也是有用的。)

UriComponentsBuilder类可用于构建和编码URI包括URI模板的支持。例如，你可以从URL字符串开始：

```
UriComponents uriComponents = UriComponentsBuilder.fromUriString( "http://example.com/hotels/{hotel}/bookings/{booking}").build()
        .expand("42", "21")
        .encode();

URI uri = uriComponents.toUri();
```

或者分别制定每个URI组件:

```
UriComponents uriComponents = UriComponentsBuilder.newInstance()
        .scheme("http").host("example.com").path("/hotels/{hotel}/bookings/{booking}").build()
        .expand("42", "21")
        .encode();

URI uri = uriComponents.toUri();
```

#### 处理请求和响应头

除了上述方法之外，RestTemplate还具有exchange\(\) 方法，可以用于基于HttpEntity 类的任意HTTP方法执行。  
也许最重要的是，该exchange\(\)方法可以用来添加请求头和读响应头。例如：

```
HttpHeaders requestHeaders = new HttpHeaders();
requestHeaders.set("MyRequestHeader", "MyValue");
HttpEntity<?> requestEntity = new HttpEntity(requestHeaders);

HttpEntity<String> response = template.exchange( "http://example.com/hotels/{hotel}",
        HttpMethod.GET, requestEntity, String.class, "42");

String responseHeader = response.getHeaders().getFirst("MyResponseHeader");
String body = response.getBody();
```

在上面的例子，我们首先准备了一个包含MyRequestHeader 头的请求实体。然后我们检索返回和读取MyResponseHeader和消息体。

#### Jackson JSON 视图支持

可以指定一个 Jackson JSON视图来系列化对象属性的一部分，例如：

```
MappingJacksonValue value = new MappingJacksonValue(new User("eric", "7!jd#h23"));
value.setSerializationView(User.WithoutPasswordView.class);
HttpEntity<MappingJacksonValue> entity = new HttpEntity<MappingJacksonValue>(value);
String s = template.postForObject("http://example.com/user", entity, String.class);
```

### 24.10.2 HTTP 消息转换

通过HttpMessageConverters，对象传递到和从getForObject\(\),postForLocation\(\),和put\(\)这些方法返回被转换成HTTP请求和HTTP相应。HttpMessageConverter接口如下所示，让你更好地感受它的功能：

```
public interface HttpMessageConverter<T> {

    // Indicate whether the given class and media type can be read by this converter.
    boolean canRead(Class<?> clazz, MediaType mediaType);

    // Indicate whether the given class and media type can be written by this converter.
    boolean canWrite(Class<?> clazz, MediaType mediaType);

    // Return the list of MediaType objects supported by this converter.
    List<MediaType> getSupportedMediaTypes();

    // Read an object of the given type from the given input message, and returns it.
    T read(Class<T> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException;

    // Write an given object to the given output message.
    void write(T t, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException;

}
```

框架中提供主要媒体（mime）类型的具体实现，默认情况下，通过RestTemplate在客户端和 RequestMethodHandlerAdapter在服务器端注册。

HttpMessageConverter的实现下面章节中描述。对于所有转换器，使用默认媒体类型，但可以通过设置supportedMediaTypesbean属性来覆盖。

**StringHttpMessageConverter**

一个HttpMessageConverter的实现，实现从HTTP请求和相应中读和写Strings。默认情况下，该转换器支持所有的文本媒体类型（text/\*）,并用text/plain的Content-Type来写。

**FormHttpMessageConverter**

一个HttpMessageConverter的实现，实现从HTTP请求和响应读写表单数据。默认情况下，该转换器读写application/x-www-form-urlencoded媒体类型。表单数据被读取并写入MultiValueMap&lt;String, String&gt;。

**ByteArrayHttpMessageConverter**

一个HttpMessageConverter的实现，实现从HTTP请求和响应中读取和写入字节数组。默认情况下，此转换器支持所有媒体类型（_/_），并使用其中的一种Content-Type进行写入application/octet-stream。这可以通过设置supportedMediaTypes属性和覆盖getContentType\(byte\[\]\)来重写。

**MarshallingHttpMessageConverter**

一个HttpMessageConverter的实现，从org.springframework.oxm包中使用Spring的Marshaller和Unmarshaller抽象实现读取和写入XML。该转换器需要Marshaller和Unmarshaller才能使用它。这些可以通过构造函数或bean属性注入。默认情况下，此转换器支持（ text/xml）和（application/xml）。

**MappingJackson2HttpMessageConverter**

一个HttpMessageConverter的实现，使用Jackson XML扩展的ObjectMapper实现读写JSON。可以根据需要通过使用JAXB或Jackson提供的注释来定制XML映射。当需要进一步控制时，XmlMapper 可以通过ObjectMapper属性注入自定义，以便需要为特定类型提供自定义XML序列化器/反序列化器。默认情况下，此转换器支持（application/xml）。

**MappingJackson2XmlHttpMessageConverter**

一个HttpMessageConverter的实现，可以使用Jackson XML扩展的XmlMapper读取和写入XML。可以根据需要通过使用JAXB或Jackson提供的注释来定制XML映射。当需要进一步控制时，XmlMapper 可以通过ObjectMapper属性注入自定义，以便需要为特定类型提供自定义XML序列化器/反序列化器。默认情况下，此转换器支持（application/xml）。

**SourceHttpMessageConverter**

一个HttpMessageConverter的实现，从HTTP请求和响应中读写 javax.xml.transform.Source。仅支持DOMSource、SAXSource和StreamSource。默认情况下，此转换器支持（text/xml）和（application/xml）。

BufferedImageHttpMessageConverter

一个HttpMessageConverter的实现，从HTTP请求和响应中读写java.awt.image.BufferedImage。此转换器读写Java I/O API支持的媒体类型。

### 24.10.3 异步RestTemplate

Web应用程序通常需要查询外部REST服务。当为这些需求扩张应用程序时，HTTP和同步调用的性质带来挑战：可能会阻塞多个线程，等待远程HTTP响应。

AsyncRestTemplate和[第24.10.1节“RestTemplate”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-resttemplate)的API非常相似; 请 参见[表24.1“RestTemplate方法概述”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/remoting.html#rest-overview-of-resttemplate-methods-tbl)。这些API之间的主要区别是AsyncRestTemplate返回ListenableFuture 封装器而不是具体的结果。

前面的RestTemplate例子翻译成：

```
// async call
Future<ResponseEntity<String>> futureEntity = template.getForEntity(
    "http://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");

// get the concrete result - synchronous call
ResponseEntity<String> entity = futureEntity.get();
```

ListenableFuture 接受完成回调：

```
ListenableFuture<ResponseEntity<String>> futureEntity = template.getForEntity(
    "http://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");

// register a callback
futureEntity.addCallback(new ListenableFutureCallback<ResponseEntity<String>>() {
    @Override
    public void onSuccess(ResponseEntity<String> entity) {
        //...
    }

    @Override
    public void onFailure(Throwable t) {
        //...
    }
});
```

> Note: 默认AsyncRestTemplate构造函数为执行HTTP请求注册一个SimpleAsyncTaskExecutor 。当处理大量短命令请求时，线程池的TaskExecutor实现ThreadPoolTaskExecutor 可能是一个不错的选择。

有关更多详细信息，参考ListenableFuture的[javadoc](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/util/concurrent/ListenableFuture.html) and AsyncTestTmeplate的[javadoc](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/client/AsyncRestTemplate.html).

