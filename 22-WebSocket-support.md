# 22. WebSocket 支持

参考文档的这一部分涵盖了Spring框架对Web应用程序中`WebSocket`风格消息传递的支持，包括使用STOMP作为应用程序级`WebSocket`子协议。

[Section 22.1, “Introduction”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/websocket.html#websocket-intro) 建立一个`WebSocket`的大致框架，涵盖应用挑战，设计考虑以及何时适合的想法。

[Section 22.2,“WebSocket API”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/websocket.html#websocket-server) 介绍了服务端的`Spring WebSocket API`,[Section 22.3,“SockJS Fallback Options”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/websocket.html#websocket-fallback) 介绍了SockJS 协议，并且展示如何配置和使用它.

[Section 22.4.1, “Overview of STOMP”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/websocket.html#websocket-stomp-overview) 介绍 STOMP 信息协议. [Section 22.4.2, “Enable STOMP over WebSocket”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/websocket.html#websocket-stomp-enable) 展示如何在Spring配置STOMP. [Section 22.4.4, “Annotation Message Handling”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/websocket.html#websocket-stomp-handle-annotations) 以下部分说明如何编写注释消息处理方法，发送消息，选择消息代理选项，以及与特殊“用户”目的地的工作. 最后, [Section 22.4.18,“Testing Annotated Controller Methods”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/websocket.html#websocket-stomp-testing) 列出了测试STOMP / WebSocket应用程序的三种方法.

## 22.1 介绍

对于web应用程序，WebSocket 协议[RFC 6455](https://tools.ietf.org/html/rfc6455)定义了一个很重要的功能：全双工，客户端与服务器之间的双向通信. 这是一个令人兴奋的新功能，在漫长的技术历史上，使Web更具交互性，包括Java Applet，XMLHttpRequest，Adobe
Flash，ActiveXObject，各种Comet 技术，服务器发送的事件等.

WebSocket协议的介绍超出了本文档的范围。但是，至少要了解，HTTP仅用于初始握手，这依赖于内置于HTTP中的机制来请求协议升级（或在这种情况下为协议交换机），服务器可以使用HTTP状态101对其进行响应 （切换协议）如果它同意。
假设握手成功，HTTP升级请求下面的TCP套接字保持打开，客户端和服务器都可以使用它来彼此发送消息.

Spring Framework 4包括一个全新的WebSocket支持的spring-websocket模块。它与Java WebSocket API标准（JSR-356）兼容，并且还提供额外的附加值，如在其余介绍中所述。

### 22.1.1 WebSocket 后备选项

采用WebSocket 一个重要的挑战是在一些浏览器中缺乏对其的支持，值得注意的是， IE 第一个支持WebSocket 的版本是10 (详情请参照[http://caniuse.com/websockets](http://caniuse.com/websockets) ). 更多,一些限制性代理可以配置为阻止尝试执行HTTP升级或在一段时间后断开连接，因为它已经打开了太久. InfoQ的文章e“[How HTML5 Web Sockets Interact With Proxy Servers](http://www.infoq.com/articles/Web-Sockets-Proxy-Servers)”中提供了Peter Lubbers对此主题的一个很好的概述。

因此，为了今天构建一个WebSocket应用程序，需要后备选项才能在必要时模拟WebSocket API。 Spring Framework提供了基于SockJS协议的透明后备选项。 这些选项可以通过配置启用，不需要修改应用程序.

### 22.1.2 消息架构

除了中短期面临的挑战之外，使用WebSocket可以提出重要的设计注意事项，这对于早期的认识至关重要，特别是与我们今天建立Web应用程序相关的知识。

今天REST风格在Web应用中广受欢迎，这是一种依赖于许多URL（资源），少数HTTP方法（动词）以及诸如使用超媒体（链接），以及无状态架构。.

相比之下，WebSocket应用程序可能仅使用单个URL进行初始HTTP握手。此后，所有消息共享并在相同的TCP连接上流动。这指向一个完全不同的，异步的，事件驱动的消息架构。更接近于传统消息传递应用 (如：JMS,AMQP).

Spring Framework 4包括一个新的 spring-messaging 模块 ，其中包含[Spring Integration](http://projects.spring.io/spring-integration/) 项目的的关键抽象，例如 Message, MessageChannel, MessageHandler以及其他可以座位消息架构的基础, 该模块还包括一组用于将消息映射到方法的注释，类似于基于Spring MVC注释的编程模型。

### 22.1.3 WebSocket中的子协议支持

WebSocket确实建立了消息架构，但并不要求使用任何特定的消息协议。它是一个非常窄的TCP层，将字节流转换为消息流（文本或二进制），而不是更多。应用来解释消息的含义。

不同于HTTP（它是应用程序级协议），在WebSocket协议中，框架或容器的传入消息中没有足够的信息来知道如何路由或处理它。因此，WebSocket可以说是太低级别，只是一个非常简单的应用程序。可以做到这一点，但它可能会导致在顶部创建一个框架。这与目前使用Web框架而不是单独使用Servlet
API的大多数Web应用程序相当。

为此，WebSocket RFC定义了子协议的使用。在握手期间，客户端和服务器可以使用头部Sec-WebSocket协议来同意子协议，即较高的应用级协议使用。不需要使用子协议，即使不使用子协议，应用程序仍然需要选择客户端和服务器可以理解的消息格式。该格式可以是自定义，框架特定或标准消息传递协议。

Spring框架支持使用STOMP – 一种简单的消息传递协议，最初创建用于脚本语言，并由HTTP启发的框架。 STOMP被广泛支持，非常适合在WebSocket和Web上使用。

### 22.1.4 我应该使用WebSocket?

有关使用WebSocket的所有设计考虑，思考“什么时候使用？”是合理的。

WebSocket最适合在Web应用程序中，客户端和服务器需要以高频率和低延迟交换事件。优选的项目类别包括但不限于在金融，游戏，合作等方面的应用。这种应用对时间延迟非常敏感，并且还需要以高频率交换各种各样的消息。

但是，对于其他应用程序类型，可能并非如此。例如，一个新闻或社交软件显示突发新闻，因为它可用可能是完全可以的简单的轮询一次每隔几分钟。这里的延迟很重要，但是如果新闻需要几分钟的时间就可以接受。

即使在延迟至关重要的情况下，如果消息量相对较低（例如监控网络故障），则长时间轮询的使用应被视为一种相对简单的替代方案，其可靠性可靠，并且在效率方面是可比较的的消息相对较低）。

低延迟和高频率的消息可以使WebSocket协议的使用成为关键。即使在这样的应用程序中，选择仍然是所有客户端 –
服务器通信是否应该通过WebSocket消息完成，而不是使用HTTP和REST。答案将因应用而异;然而，有可能某些功能可以通过WebSocket和REST API来暴露，以便为客户提供替代方案。此外，REST
API调用可能需要向通过WebSocket连接的感兴趣的客户端广播消息。

Spring Framework允许`@Controller`和`@RestController`类具有HTTP请求处理和WebSocket消息处理方法。此外，Spring MVC请求处理方法或任何应用方法可以轻松地向所有感兴趣的WebSocket客户端或特定用户广播消息。

## 22.2 WebSocket API

The Spring架构提供的WebSocket API 被设计成应用于各类WebSocket 引擎. 当前这个列表包括WebSocket 运行时 ，例如 Tomcat 7.0.47+, Jetty 9.1+, GlassFish 4.1+, WebLogic 12.1.3+, and Undertow 1.0+ (and WildFly 8.0+). 随着更多的WebSocket运行时可用，可能会添加额外的支持。

### 22.2.1 创建和配置一个 WebSocketHandler

创建WebSocket服务器与实现`WebSocketHandler`一样简单，或者更有可能扩展`TextWebSocketHandler`或`BinaryWebSocketHandler`：
```
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.TextMessage;

public class MyHandler extends TextWebSocketHandler {

	@Override
	public void handleTextMessage(WebSocketSession session, TextMessage message) {
		// ...
	}

}
```

有专门的`WebSocket Java-config`和`XML`命名空间支持将上述WebSocket处理程序映射到特定的URL：

```
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
		registry.addHandler(myHandler(), "/myHandler");
	}

	@Bean
	public WebSocketHandler myHandler() {
		return new MyHandler();
	}

}
```

等价的XML配置:

```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:websocket="http://www.springframework.org/schema/websocket"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/websocket
		http://www.springframework.org/schema/websocket/spring-websocket.xsd">

	<websocket:handlers>
		<websocket:mapping path="/myHandler" handler="myHandler"/>
	</websocket:handlers>

	<bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

以上是用于Spring MVC应用程序，应该包含在[DispatcherServlet](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-servlet) 的配置中.但是Spring的WebSocket支持不依赖于Spring MVC。 在WebSocketHttpRequestHandler的帮助下 ，将[WebSocketHttpRequestHandler](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/socket/server/support/WebSocketHttpRequestHandler.html)集成到其他HTTP服务环境中相对简单.


### 22.2.2 自定义he WebSocket 握手

自定义初始HTTP WebSocket握手请求的最简单的方法是通过`HandshakeInterceptor`，它将握手方法之前的“before”和“after”。 这样的拦截器可以用于阻止握手或使任何属性可用于`WebSocketSession`。

例如，有一个内置拦截器将HTTP会话属性传递给WebSocket会话：

```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
		registry.addHandler(new MyHandler(), "/myHandler")
			.addInterceptors(new HttpSessionHandshakeInterceptor());
	}

}
```

XML等价配置:

```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:websocket="http://www.springframework.org/schema/websocket"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/websocket
		http://www.springframework.org/schema/websocket/spring-websocket.xsd">

	<websocket:handlers>
		<websocket:mapping path="/myHandler" handler="myHandler"/>
		<websocket:handshake-interceptors>
			<bean class="org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor"/>
		</websocket:handshake-interceptors>
	</websocket:handlers>

	<bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

更高级的选项是扩展执行WebSocket握手步骤的`DefaultHandshakeHandler`，包括验证客户端，协商子协议等。
如果需要配置自定义`RequestUpgradeStrategy`以适应不支持的WebSocket服务器引擎和版本，应用程序也可能需要使用此选项（有关此主题的更多信息，请参见第22.2.4节“部署注意事项”））。
Java-config和XML命名空间都可以配置自定义HandshakeHandler。

### 22.2.3 WebSocketHandler 装饰

Spring提供了一个`WebSocketHandlerDecorator`基类，可用于使用附加行为来装饰`WebSocketHandler`。

在使用WebSocket Java-config或XML命名空间时，默认情况下提供并添加了日志记录和异常处理实现。

`ExceptionWebSocketHandlerDecorator`捕获从任何`WebSocketHandler`方法引发的所有未捕获的异常，并关闭表示服务器错误的状态1011的WebSocket会话。

### 22.2.4 部署注意事项

Spring WebSocket API易于集成到Spring MVC应用程序中，其中`DispatcherServlet`用于HTTP
WebSocket握手以及其他HTTP请求。通过调用`WebSocketHttpRequestHandler`也很容易集成到其他HTTP处理场景中。这是方便和容易理解。但是，JSR-356运行时可以考虑特殊的考虑因素。

Java WebSocket API（JSR-356）提供了两个部署机制。第一个涉及启动时的Servlet容器类路径扫描（Servlet 3功能）;另一个是在Servlet容器初始化时使用的注册API。这些机制都不可能对所有HTTP处理（包括WebSocket握手和所有其他HTTP请求）使用单个“前端控制器”，例如Spring MVC的`DispatcherServlet`。

即使在JSR-356运行时运行时，通过提供特定于服务器的`RequestUpgradeStrategy`，Spring的WebSocket支持的JSR-356的一个重大限制。

第二个考虑因素是具有JSR-356支持的Servlet容器预计将执行`ServletContainerInitializer（SCI）`扫描，这可能会减慢应用程序启动速度，在某些情况下会显着降低。

如果在升级到支持`JSR-356`的Servlet容器版本之后观察到重大影响，则可以通过使用Web.XML中的<absolute-ordering />元素来选择性地启用或禁用Web片段（和SCI扫描）：

```
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
		http://java.sun.com/xml/ns/javaee
		http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">

	<absolute-ordering/>

</web-app>
```

然后，您可以通过名称有选择地启用Web片段，例如Spring自己的`SpringServletContainerInitialize`r，如果需要，可以提供对Servlet 3 Java初始化API的支持:

```
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
		http://java.sun.com/xml/ns/javaee
		http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">

	<absolute-ordering>
		<name>spring_web</name>
	</absolute-ordering>

</web-app>
```

### 22.2.5 配置WebSocket 引擎

每个底层WebSocket引擎都会公开控制运行时特性的配置属性，例如消息缓冲区大小，空闲超时等。

对于Tomcat，WildFly和GlassFish，在您的WebSocket Java配置中添加一个`ServletServerContainerFactoryBean`：

```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

	@Bean
	public ServletServerContainerFactoryBean createWebSocketContainer() {
		ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
		container.setMaxTextMessageBufferSize(8192);
		container.setMaxBinaryMessageBufferSize(8192);
		return container;
	}

}
```

或者WebSocket XML 命名空间:

```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:websocket="http://www.springframework.org/schema/websocket"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/websocket
		http://www.springframework.org/schema/websocket/spring-websocket.xsd">

	<bean class="org.springframework...ServletServerContainerFactoryBean">
		<property name="maxTextMessageBufferSize" value="8192"/>
		<property name="maxBinaryMessageBufferSize" value="8192"/>
	</bean>

</beans>
```

对于Jetty, 你需要通过WebSocket Java config预先配置`Jetty WebSocketServerFactory` 和插件注入到`Spring’s DefaultHandshakeHandler`

```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
		registry.addHandler(echoWebSocketHandler(),
			"/echo").setHandshakeHandler(handshakeHandler());
	}

	@Bean
	public DefaultHandshakeHandler handshakeHandler() {

		WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
		policy.setInputBufferSize(8192);
		policy.setIdleTimeout(600000);

		return new DefaultHandshakeHandler(
				new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
	}

}
```

或者WebSocket XML 命名空间:

```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:websocket="http://www.springframework.org/schema/websocket"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/websocket
		http://www.springframework.org/schema/websocket/spring-websocket.xsd">

	<websocket:handlers>
		<websocket:mapping path="/echo" handler="echoHandler"/>
		<websocket:handshake-handler ref="handshakeHandler"/>
	</websocket:handlers>

	<bean id="handshakeHandler" class="org.springframework...DefaultHandshakeHandler">
		<constructor-arg ref="upgradeStrategy"/>
	</bean>

	<bean id="upgradeStrategy" class="org.springframework...JettyRequestUpgradeStrategy">
		<constructor-arg ref="serverFactory"/>
	</bean>

	<bean id="serverFactory" class="org.eclipse.jetty...WebSocketServerFactory">
		<constructor-arg>
			<bean class="org.eclipse.jetty...WebSocketPolicy">
				<constructor-arg value="SERVER"/>
				<property name="inputBufferSize" value="8092"/>
				<property name="idleTimeout" value="600000"/>
			</bean>
		</constructor-arg>
	</bean>

</beans>
```

### 22.2.6允许域名配置
作为 Spring Framework 4.1.5, the default behavior for WebSocket 和SockJS默认的行为仅仅是接收相同的域名请求 . 也可以允许所有或指定的域名列表。 此检查主要是为浏览器客户端设计的。

没有什么可以阻止其他类型的客户端修改Origin标头值(需要查看更多详情：[RFC 6454: The Web Origin Concept](https://tools.ietf.org/html/rfc6454)).

三种可能的行为:

- 只允许相同的域名请求（默认）：在此模式下，当启用SockJS时，Iframe
HTTP响应头X-Frame-Options设置为SAMEORIGIN，并且禁用JSONP传输，因为它不允许检查请求的来源。 因此，当启用此模式时，不支持IE6和IE7。

- 允许指定的域名列表：每个提供的允许来源必须以http：//或https：//开头。 在此模式下，当启用SockJS时，基于IFrame和JSONP的传输均被禁用。因此，启用此模式时，不支持IE6至IE9。

- 允许所有域名：启用此模式，您应该提供*作为允许的原始值。 在这种模式下，所有的运输都可用
WebSocket 和SockJS 允许的域名能够如下配置:

```
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
		registry.addHandler(myHandler(), "/myHandler").setAllowedOrigins("http://mydomain.com");
	}

	@Bean
	public WebSocketHandler myHandler() {
		return new MyHandler();
	}

}
```

等价的XML 配置:

```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:websocket="http://www.springframework.org/schema/websocket"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/websocket
		http://www.springframework.org/schema/websocket/spring-websocket.xsd">

	<websocket:handlers allowed-origins="http://mydomain.com">
		<websocket:mapping path="/myHandler" handler="myHandler" />
	</websocket:handlers>

	<bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```


### 22.3 SockJS后备选项

正如简介所说，As explained in the [introduction](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/websocket.html#websocket-into-fallback-options),WebSocket is not supported in all browsers yet and may be precluded by restrictive network proxies. This is why
Spring provides fallback options that emulate the WebSocket API as close as possible based on the [SockJS protocol](https://github.com/sockjs/sockjs-protocol) (version 0.3.3).
