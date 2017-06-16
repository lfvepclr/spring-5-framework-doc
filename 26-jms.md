# 26. JMS

## 26.1 介绍

Spring 提供了一个 JMS 的集成框架，简化了 JMS API 的使用，就像 Spring 对 JDBC API 的集成一样。

JMS 大致可分为两块功能，即消息的生产与消费。`JmsTemplate`类用于消息生产和消息的同步接收。 对于类似 Java EE 的消息驱动 Bean 形式的异步接收，Spring 提供了大量用于创建消息驱动 POJOs（MDPs）的消息监听器。Spring 还提供了一种创建消息侦听器的声明式方法。  


`org.springframework.jms.core`包提供了使用 JMS 的核心功能。它包含了 JMS 模板类，用来处理资源的创建与释放，从而简化 JMS 的使用，就像`JdbcTemplate`对 JDBC 做的一样。像其他大多数 Spring 模板类一样，JMS 模板类提供了执行公共操作的 helper 方法。在需要更复杂应用的情况下，类把处理任务的核心委托给用户实现的回调接口。JMS 类提供了方便的方法，用来发送消息、同步地使用消息以及向用户公开 JMS 会话和消息的生产者。

`org.springframework.jms.support`包提供了转换 JMSException 的功能。  
转换代码将受检查的`JMSException`层转换到不受检查异常的镜像层。如果有一个提供者指定的受检查的`javax.jms.JMSException`类的子类，这个子类异常被封装到了不受检查的`UncategorizedJmsException`异常中。

`org.springframework.jms.support.converter`包提供了 MessageConverter 抽象，进行 Java 对象和 JMS 消息的互相转换。

`org.springframework.jms.support.destination`包提供了管理 JMS 目的地的不同策略，比如针对 JNDI 中保存的目标的服务定位器。

`org.springframework.jms.annotation`包提供了支持注解驱动监听端点的必要基础架构，通过使用`@JmsListener`实现。

`org.springframework.jms.config`包提供了 JMS 命名空间的解析实现，以及配置监听容器和创建监听端点的 java 配置支持。

最后，`org.springframework.jms.connection`包提供了适用于独立应用程序的`ConnectionFactory`实现。 它还包含 Spring 对 JMS 的`PlatformTransactionManager`实现（即`JmsTransactionManager`）。这将允许 JMS 作为事务性资源无缝集成到 Spring 的事务管理机制中。

## 26.2 Spring JMS的使用

### 26.2.1 JmsTemplate

`JmsTemplate`类是JMS核心包中的中心类。它简化了 JMS 的使用，因为在发送或同步接收消息时它帮我们处理了资源的创建和释放。

使用`JmsTemplate`的代码只需要实现规范中定义的回调接口。在`JmsTemplate`中通过调用代码让`MessageCreator`回调接口用所提供的会话\(`Session`\)创建消息。然而，为了顾及更复杂的 JMS API 应用，回调接口`SessionCallback`将 JMS 会话提供给用户，回调接口`ProducerCallback`则公开了`Session`和`MessageProducer`的组合。

JMS API 公开了发送方法的两种类型，一种接受交付模式、优先级和存活时间作为服务质量（QOS）参数，另一种则使用缺省值作为 QOS 参数（无需参数）方式。由于 JmsTemplate 中有很多发送方法，QOS 参数用 bean 属性进行暴露设置，从而避免在一系列发送方法中的重复。同样地，使用`setReceiveTimeout`属性设置用于同步接收调用的超时值。

一些 JMS 提供者通过配置`ConnectionFactory`，管理方式上允许默认的 QOS值 的设置。`MessageProducer`的发送方法`send(Destination destination, Message message)`在那些专有的 JMS 中将会使用不一样的 QOS 默认值。 所以，为了提供对 QOS 值域、的统一管理，`JmsTemplate`必须通过设置布尔值属性`isExplicitQosEnabled`为true，使它能够使用自己的QOS值。

为了方便起见，`JmsTemplate`还暴露了一个基本的请求-回复操作，允许在一个作为操作一部分而被创建的临时队列上，进行消息的发送与等待回复。

> 配置的`JmsTemplate`类的实例是线程安全的。这很重要，因为这意味着你可以配置一个`JmsTemplate`单例，然后安全地将这个共享引用注入给多个协作者。 要清楚，保持对`ConnectionFactory`引用的`JmsTemplate`是有状态的，但该状态不是会话状态。

从 Spring Framework 4.1开始，`JmsMessagingTemplate`构建在`JmsTemplate`之上，并提供与消息抽象层（即`org.springframework.messaging.Message`）的集成。 这允许你以通用的方式来创建要发送的消息。

### 26.2.2 Connections

`JmsTemplate`需要一个对`ConnectionFactory`的引用。`ConnectionFactory`是 JMS 规范的一部分，并被作为使用 JMS 的入口。客户端应用通常作为一个工厂配合 JMS 提供者去创建连接，并封装一系列的配置参数，其中一些是和供应商相关的，例如 SSL 的配置选项。

当在 EJB 内使用 JMS 时，供应商提供 JMS 接口的实现，以至于可以参与声明式事务的管理，提供连接池和会话池。为了使用这个实现，J2EE 容器一般要求你在 EJB 或 servlet 部署描述符中将 JMS 连接工厂声明为 resource-ref。为确保可以在 EJB 内使用`JmsTemplate`的这些特性，客户端应当确保它能引用其中的`ConnectionFactory`实现。

#### 缓存消息资源

标准的API涉及创建许多中间对象。要发送消息，将执行以下“API”步骤

```
ConnectionFactory->Connection->Session->MessageProducer->send
```

在`ConnectionFactory`和`Send`操作之间，有三个中间对象被创建和销毁。 为了优化资源使用并提高性能，提供了两个`ConnectionFactory`的实现。

#### SingleConnectionFactory

Spring 提供`ConnectionFactory`接口的一个实现，`SingleConnectionFactory`，它将在所有的`createConnection`调用中返回同一个的连接，并忽略`close`的调用。这在测试和独立的环境中相当有用，因为同一个连接可以被用于多个`JmsTemplate`调用以跨越多个事务。`SingleConnectionFactory`接受一个通常来自 JNDI 的标准`ConnectionFactory`的引用。

#### CachingConnectionFactory

`CachingConnectionFactory`扩展了`SingleConnectionFactory`的功能，它添加了会话、消息生产者、消息消费者的缓存。 初始缓存大小设置为1，使用`sessionCacheSize`属性来增加缓存会话的数量。请注意，实际缓存会话的数量将超过该值，因为会话的缓存是基于确认模式的，因此当设置`sessionCacheSize`为1时，缓存的会话可能达到4个，因为每个确认模式都会缓存一个。当缓存的时候，消息生产者和消息消费者被缓存在他们自己的会话中同时也考虑到生产者和消费者的唯一属性。消息生产者基于他们的目的地被缓存，消息消费者基于目的地、选择器、非本地传送标识和持久订阅名称（假设创建持久消费者）的组合键被缓存。

### 26.2.3 Destination 管理

目的地（Destination），像`ConnectionFactories`一样，是可以在 JNDI 中进行存储和提取的 JMS 管理对象。当配置一个 Spring 应用上下文，可以使用 JNDI 工厂类`JndiObjectFactoryBean / <jee:jndi-lookup>`将你的对象引用依赖注入到 JMS 目的地。然而，如果在应用中有大量的目的地，或者 JMS 供应商提供了特有的高级目的地管理特性，这个策略常常显得很笨重。高级目的地管理的例子如创建动态目的地或支持目的地的命名层次。`JmsTemplate`将目的地名称到 JMS 目的地对象的解析委派给一个`DestinationResolver`接口的实现。`DynamicDestinationResolver`是`JmsTemplate`使用的默认实现，并且提供动态目的地解析。同时`JndiDestinationResolver`作为 JNDI 包含的目的地的服务定位器，并且可选择地退回来使用`DynamicDestinationResolver`提供的行为。

相当常见的是在一个 JMS 应用中所使用的目的地只有在运行时才知道，因此，当一个应用被部署时，它不能被创建。这经常是因为交互系统组件之间的共享应用逻辑是在运行时按照已知的命名规范创建目的地。虽然动态目的地的创建不是 JMS 规范的一部分，但是许多供应商已经提供了这个功能。用户为所建的动态目的地定义名称，这样区别于临时的目的地，并且动态目的地不会被注册到 JNDI 中。创建动态目的地所使用的 API 在不同的供应商之间差别很大，因为目的地所关联的属性是供应商特有的。然而，有时由供应商作出的一个简单的实现选择是忽略 JMS 规范中的警告，并使用`TopicSession`的方法`createTopic(String topicName)`或者`QueueSession`的方法`createQueue(String queueName)`来创建一个拥有默认属性的新目的地。依赖于供应商的实现，`DynamicDestinationResolver`也可能创建一个物理上的目的地，而不是只是解析。

布尔属性`PubSubDomain`被用来配置`JmsTemplate`使用什么样的 JMS 域。这个属性的默认值是 false，使用点到点的队列。`JmsTemplate`使用该属性决定了通过`DestinationResolver`的实现来解析动态目的地的行为。

你还可以通过属性`DefaultDestination`配置一个带有默认目的地的`JmsTemplate`。默认的目的地被使用时，它的发送和接收操作不需要指定一个特定的目的地。

### 26.2.4 消息监听容器

在 EJB 世界里，JMS 消息最常用的功能之一是用于实现消息驱动 Bean（MDB）。Spring 提供了一个方法来创建消息驱动的 POJO（MDP），并且不会把用户绑定在某个 EJB 容器上。（[参见第26.4.2节“异步接收 – 消息驱动的 POJO”，详细介绍了 Spring 的 MDP 支持](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jms.html#jms-asynchronousMessageReception)）。从 Spring Framework 4.1开始，端点方法可以简单使用 @JmsListener 注解，[参见第26.6节“注释驱动的侦听器端点 “ 更多细节。](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jms.html#jms-annotated)

用消息监听容器从 JMS 消息队列接收消息，并驱动被注入该消息的消息监听器。监听容器负责消息接收和分发到对应的监听器的所有线程。消息监听容器是 MDP 和消息提供者之间的一个中介，负责处理消息接收的注册、事务管理、资源获取与释放和异常转换等。这使得应用开发人员可以专注于开发和接收消息（可能的响应）相关的（复杂）业务逻辑，把和 JMS 基础框架有关的样板化的部分委托给框架处理。

有两个标准的 JMS 消息监听容器包含在 Spring 中，每一个都有它特殊的功能集。

#### SimpleMessageListenerContainer

这个消息监听容器是两种标准风格中比较简单的一个，它在启动时创建固定数量的 JMS 会话和消费者，使用标准的 JMS 方法`MessageConsumer.setMessageListener()`注册监听，并且让 JMS 提供者做监听回调。它不适于动态运行要求或者参与额外管理事务。兼容上，它与标准的 JMS 规范很近，但它通常情况下不兼容 Java EE 的 JMS 限制条件。

> 虽然`SimpleMessageListenerContainer`不允许参与外部管理的事务，但它确实支持原生 JMS 事务：只需将`sessionTransacted`标志切换为 true，或者在命名空间中将`acknowledge`属性设置为 transacted：监听器抛出的异常将会导致回滚，然后消息被重新传递。或者，考虑使用`CLIENT_ACKNOWLEDGE`模式，在异常的情况下提供重新传递，但没有使用事务会话，因此在事务协议中不包括任何其他会话操作（例如发送响应消息）。

#### DefaultMessageListenerContainer

这个消息监听容器用于大部分的案例中。与`SimpleMessageListenerContainer`相反的是，这个容器适于动态运行要求并且能参与额外管理事务。 在配置`JtaTransactionManager`的时候，每一个被接收的消息使用 XA 事务注册，因此可能利用 XA 事务语法处理。该监听容器在 JMS 提供者低要求、高级功能（如外部管理事务的参与）以及与 Java EE 环境的兼容性之间取得了良好的平衡。

容器缓存等级可以定制，注意当缓存不可用的时候，每一次消息接收，一个新的连接和新的会话就会被创建。使用高负载的非持久化订阅可能导致消息丢失，在这种情况下，确保使用合适的缓存等级。

当代理挂掉时，此容器也具备可恢复的能力。默认情况下，一个简单的`BackOff`实现会每5秒重试一次。可以为更细粒度的恢复选项指定自定义的`BackOff`实现，请参见`ExponentialBackOff`示例。

> 与同级的`SimpleMessageListenerContainer`一样，`DefaultMessageListenerContainer`支持原生 JMS 事务，并允许自定义确认模式。如果可行的话，强烈建议您使用外部管理的事务：即，如果你可以忍受在 JVM 挂掉的情况下偶尔会重复发送消息。业务逻辑中的自定义重复消息检测步骤可能涵盖这些情况，例如以业务实体的形式存在的检查或协议表检查。这样的安排将比任何其他方式显着更有效：用 XA 事务（通过使用`JtaTransactionManager`配置你的`DefaultMessageListenerContainer`）来包裹整个过程，覆盖了 JMS 消息的接收以及消息监听器中业务逻辑的执行（包括数据库操作等）。

### 26.2.5 事务管理

Spring 提供了一个`JmsTransactionManager`用于对 JMS`ConnectionFactory`做事务管理。这将允许 JMS 应用利用 Spring 的事务管理特性。[第13章事务管理中所述的 Spring 的托管事务功能](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html)。`JmsTransactionManager`在执行本地资源事务管理时将从指定的`ConnectionFactory`绑定一个`ConnectionFactory/Session`这样的配对到线程中。`JmsTemplate`会自动检测这样的事务资源，并对它们进行相应操作。

在 Java EE 环境中，`ConnectionFactory`会池化连接和会话，这样这些资源将会在整个事务中被有效地重复利用。在一个独立的环境中，使用 Spring 的`SingleConnectionFactory`时所有的事务将公用一个JMS 连接，但是每个事务将保留自己独立的会话。或者，请考虑使用具体提供者的池适配器，如 ActiveMQ 的`PooledConnectionFactory`类。

`JmsTemplate`也利用`JtaTransactionManager`和支持 XA 的 JMS`ConnectionFactory`一起来执行分布式事务。请注意，这需要使用 JTA 事务管理器以及正确的 XA 配置的`ConnectionFactory`！（检查您的 Java EE 服务/ JMS 提供者的文档。）

在使用 JMS API 从连接中创建会话时，通过托管和非托管事务环境重用代码可能会令人困惑。 这是因为 JMS API 只有一种工厂方法来创建会话，它需要对事务和确认模式赋值。在托管环境中，设置这些值是环境事务性基础架构的责任，因此供应商对 JMS 连接的包装器将忽略这些值。 在非托管环境中使用`JmsTemplate`时，可以通过使用属性`sessionTransacted`和`sessionAcknowledgeMode`来指定这些值。 当与`JmsTemplate`一起使用`PlatformTransactionManager`时，模板将始终被赋予一个事务性 JMS 会话。

## 26.3 发送消息

`JmsTemplate`包含许多方便的方法来发送消息。有些发送方法可以使用`javax.jms.Destination`对象指定目的地，也可以使用字符串在 JNDI 中查找目的地。没有目的地参数的发送方法使用默认的目的地。

```
import javax.jms.ConnectionFactory;
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.Queue;
import javax.jms.Session;

import org.springframework.jms.core.MessageCreator;
import org.springframework.jms.core.JmsTemplate;

public class JmsQueueSender {

	private JmsTemplate jmsTemplate;
	private Queue queue;

	public void setConnectionFactory(ConnectionFactory cf) {
		this.jmsTemplate = new JmsTemplate(cf);
	}

	public void setQueue(Queue queue) {
		this.queue = queue;
	}

	public void simpleSend() {
		this.jmsTemplate.send(this.queue, new MessageCreator() {
			public Message createMessage(Session session) throws JMSException {
				return session.createTextMessage("hello queue world");
			}
		});
	}
}
```

这个例子使用`MessageCreator`回调接口从提供的会话对象中创建一个文本消息，并且通过一个`ConnectionFactory`的引用来构造`JmsTemplate`。或者，提供了一个无参数的构造方法和`connectionFactory`，并可用于以 JavaBean 方式构建实例（使用 BeanFactory 或纯 Java 代码）。或者考虑从 Spring 的基类`JmsGatewaySupport`派生，它对 JMS 配置具有内置的 bean 属性。

方法`send(String destinationName，MessageCreator creator)`让你利用目的地的字符串名称发送消息。如果这些名称在 JNDI 中注册，则应将模板的`destinationResolver`属性设置为`JndiDestinationResolver`的一个实例。

如果创建了`JmsTemplate`并指定一个默认的目的地，那么`send(MessageCreator c)`会向该目的地发送消息。

### 26.3.1 使用消息转换器

为便于发送领域模型对象，`JmsTemplate`有多种以一个 Java 对象为参数并做为发送消息的数据内容。`JmsTemplate`里可重载的方法`convertAndSend`和`receiveAndConvert`将转换的过程委托给接口`MessageConverter`的一个实例。这个接口定义了一个简单的合约用来在 Java 对象和 JMS 消息间进行转换。缺省的实现`SimpleMessageConverter`支持`String`和`TextMessage`，`byte[]`和`BytesMesssage`,以及`java.util.Map`和`MapMessage`之间的转换。使用转换器，可以使你和你的应用关注于通过 JMS 接收和发送的业务对象而不用操心它是具体如何表达成 JMS 消息的。

目前的沙箱模型包括一个`MapMessageConverter`，它使用反射转换 JavaBean 和`MapMessage`。其他流行可选的实现方式包括使用已存在的 XML 编组的包（如 JAXB，Castor 或 XStream）来创建一个表示对象的`TextMessage`。

为方便那些不能以通用方式封装在转换类里的消息属性、消息头和消息体的设置，通过`MessagePostProcessor`接口，你可以在消息被转换后并且在发送前访问该消息。下例展示了如何在`java.util.Map`已经转换成一个消息后更改消息头和属性。

```
public void sendWithConversion() {
	Map map = new HashMap();
	map.put("Name", "Mark");
	map.put("Age", new Integer(47));
	jmsTemplate.convertAndSend("testQueue", map, new MessagePostProcessor() {
		public Message postProcessMessage(Message message) throws JMSException {
			message.setIntProperty("AccountID", 1234);
			message.setJMSCorrelationID("123-00001");
			return message;
		}
	});
}
```

这将产生一个如下的消息格式:

```
MapMessage={
	Header={
		... standard headers ...
		CorrelationID={123-00001}
	}
	Properties={
		AccountID={Integer:1234}
	}
	Fields={
		Name={String:Mark}
		Age={Integer:47}
	}
}
```

### 26.3.2 SessionCallback和ProducerCallback

虽然 send 操作适用于许多常见的使用场景，但是有时你需要在一个 JMS 会话（`Session`） 或者`MessageProducer`上执行多个操作。接口`SessionCallback`和`ProducerCallback`分别提供了 JMS Session 和 Session / MessageProducer 对。在`JmsTemplate`上的`execute()`方法执行这些回调方法。

## 26.4 接收消息

### 26.4.1 同步接收

虽然 JMS 通常与异步处理相关，但它也可以同步地消费消息。可重载的`receive(..)`方法提供了这个功能。在同步接收期间，调用线程阻塞，直到接收到消息。这可能是一个危险的操作，因为调用线程可能无限期地被阻塞。`receiveTimeout`属性指定了接收者等待消息的超时时间。

### 26.4.2 异步接收 – 消息驱动的 POJOs

> Spring 还可以通过使用`@JmsListener`注解来支持监听注解端点，并提供了一种以编程方式注册端点的开放式基础架构。 这是设置异步接收器的最方便的方法，有关详细信息，[请参见第26.6.1节“启用监听端点注解”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jms.html#jms-annotated-support)。

类似于 EJB 世界里流行的消息驱动 bean\(MDB\)，消息驱动 POJO\(MDP\) 作为 JMS 消息的接收器。MDP 的一个约束\(请看下面的有关`javax.jms.MessageListener`类的讨论\)是它必须实现`javax.jms.MessageListener`接口。另外当你的 POJO 将以多线程的方式接收消息时必须确保你的代码是线程安全的。

下面是 MDP 的一个简单实现:

```
import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.TextMessage;

public class ExampleListener implements MessageListener {

	public void onMessage(Message message) {
		if (message instanceof TextMessage) {
			try {
				System.out.println(((TextMessage) message).getText());
			}
			catch (JMSException ex) {
				throw new RuntimeException(ex);
			}
		}
		else {
			throw new IllegalArgumentException("Message must be of type TextMessage");
		}
	}

}
```

一旦你实现了`MessageListener`接口,下面该创建一个消息监听容器了。

请看下面例子是如何定义和配置一个随 Sping 发行的消息侦听容器的\(这个例子用`DefaultMessageListenerContainer`\)。

```
<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="jmsexample.ExampleListener" />

<!-- and this is the message listener container -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
	<property name="connectionFactory" ref="connectionFactory"/>
	<property name="destination" ref="destination"/>
	<property name="messageListener" ref="messageListener" />
</bean>
```

请参阅各种消息监听容器的 Spring javadocs，以了解每个实现所支持功能的完整描述。

### 26.4.3 SessionAwareMessageListener 接口

`SessionAwareMessageListener`接口是一个 Spring 专门用来提供类似于 JMS`MessageListener`的接口，也提供了从接收`Message`来访问 JMS`Session`的消息处理方法。

```
package org.springframework.jms.listener;

public interface SessionAwareMessageListener {

	void onMessage(Message message, Session session) throws JMSException;

}
```

如果你希望你的 MDP 可以响应所有接收到的消息（使用`onMessage(Message, Session)`方法提供的`Session`）那么你可以选择让你的 MDP 实现这个接口（优先于标准的 JMS`MessageListener`接口）。所有随 Spring 发行的支持 MDP 的消息监听容器都支持`MessageListener`或`SessionAwareMessageListener`接口的实现。要注意的是实现了`SessionAwareMessageListener`接口的类通过接口与 Spring 有了耦合。是否选择使用它完全取决于开发者或架构师。

请注意`SessionAwareMessageListener`接口的`onMessage(..)`方法会抛出`JMSException`异常。和标准 JMS`MessageListener`接口相反，当使用`SessionAwareMessageListener`接口时，客户端代码负责处理所有抛出的异常。

### 26.4.4 MessageListenerAdapter

`MessageListenerAdapter`类是 Spring 的异步支持消息类中的最后一个组建：简而言之，它允许您将几乎任何类都暴露为MDP（当然有一些限制）。

请考虑以下接口定义。请注意，虽然该接口既不继承`MessageListener`，也不继承`SessionAwareMessageListener`接口，但通过`MessageListenerAdapter`类依然可以当作一个 MDP 使用。还要注意，各种消息处理方法是如何根据可以接收和处理的各种消息的内容进行强类型匹配的。

```
public interface MessageDelegate {

	void handleMessage(String message);

	void handleMessage(Map message);

	void handleMessage(byte[] message);

	void handleMessage(Serializable message);

}
```

```
public class DefaultMessageDelegate implements MessageDelegate {
	// implementation elided for clarity...
}
```

尤其要注意的是，上述`MessageDelegate`接口的实现（上述`DefaultMessageDelegate`类）完全不依赖于 JMS。它是一个真正的 POJO，我们可以通过如下配置把它设置成 MDP。

```
<!-- this is the Message Driven POJO (MDP) -->
<bean id="messageListener" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
	<constructor-arg>
		<bean class="jmsexample.DefaultMessageDelegate"/>
	</constructor-arg>
</bean>

<!-- and this is the message listener container... -->
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
	<property name="connectionFactory" ref="connectionFactory"/>
	<property name="destination" ref="destination"/>
	<property name="messageListener" ref="messageListener" />
</bean>
```

以下是另一个只能接收 JMS`TextMessage`消息的 MDP 示例。注意消息处理方法是如何实际调用`receive`\(在`MessageListenerAdapter`中默认的消息处理方法的名字是`handleMessage`\)的，但是它是可配置的\(从下面可以看到\)。注意`receive(..)`方法是如何使用强制类型来只接收和处理JMS`TextMessage`消息的。

```
public interface TextMessageDelegate {

	void receive(TextMessage message);

}
```

```
public class DefaultTextMessageDelegate implements TextMessageDelegate {
	// implementation elided for clarity...
}
```

辅助的`MessageListenerAdapter`类配置文件类似如下：

```
<bean id="messageListener" class="org.springframework.jms.listener.adapter.MessageListenerAdapter">
	<constructor-arg>
		<bean class="jmsexample.DefaultTextMessageDelegate"/>
	</constructor-arg>
	<property name="defaultListenerMethod" value="receive"/>
	<!-- we don't want automatic message context extraction -->
	<property name="messageConverter">
		<null/>
	</property>
</bean>
```

请注意，如果上述`messageListener`接收到不是`TextMessage`类型的 JMS 消息，则会抛出`IllegalStateException`（随之产生的其他异常只被捕获而不处理）。`MessageListenerAdapter`还有一个功能就是如果处理方法返回一个非空值，它将自动返回一个响应消息。请看下面的接口及其实现：

```
public interface ResponsiveTextMessageDelegate {

	// notice the return type...
	String receive(TextMessage message);

}
```

```
public class DefaultResponsiveTextMessageDelegate implements ResponsiveTextMessageDelegate {
	// implementation elided for clarity...
}
```

如果将上述`DefaultResponsiveTextMessageDelegate`与`MessageListenerAdapter`联合使用，那么从执行`receive(..)`方法返回的任何非空值都将（缺省情况下）转换为`TextMessage`。这个返回的`TextMessage`将被发送到原来的`Message`中 JMS Reply-To 属性定义的目的地（如果存在），或者是`MessageListenerAdapter`设置（如果配置了）的缺省目的地；如果没有定义目的地，那么将产生一个`InvalidDestinationException`异常（此异常将不会只被捕获而不处理，它将沿着调用堆栈上传）。

### 26.4.5 事务中的消息处理

在事务中调用消息监听器只需要重新配置监听容器。

本地资源事务可以通过监听容器上定义的`sessionTransacted`标志进行简单地激活。 然后，每个消息监听器调用将在激活的 JMS 事务中进行操作，并在监听器执行失败的情况下进行消息回滚。 发送响应消息（通过`SessionAwareMessageListener`）将成为同一本地事务的一部分，但任何其他资源操作（如数据库访问）将独立运行。 在监听器的实现中通常需要进行重复消息的检测，覆盖数据库处理已经提交但消息处理提交失败的情况。

```
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
	<property name="connectionFactory" ref="connectionFactory"/>
	<property name="destination" ref="destination"/>
	<property name="messageListener" ref="messageListener"/>
	<property name="sessionTransacted" value="true"/>
</bean>
```

对于参与外部管理的事务，你将需要配置一个事务管理器并使用支持外部管理事务的监听容器：通常为`DefaultMessageListenerContainer`。

要配置 XA 事务参与的消息监听容器，您需要配置一个`JtaTransactionManager`（默认情况下，它将委托给 Java EE 服务器的事务子系统）。请注意，底层的 JMS`ConnectionFactory`需要具有 XA 能力并且正确地注册到你的 JTA 事务协调器上！（检查你的 Java EE 服务的 JNDI 资源配置。）这允许消息接收以及例如同一事务下的数据库访问（具有统一提交语义，以 XA 事务日志开销为代价）。

```
<bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

然后，你只需要将它添加到我们之前的容器配置中。其余的交给容器处理。

```
<bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
	<property name="connectionFactory" ref="connectionFactory"/>
	<property name="destination" ref="destination"/>
	<property name="messageListener" ref="messageListener"/>
	<property name="transactionManager" ref="transactionManager"/>
</bean>
```

## 26.5 支持 JCA 消息端点

从 Spring2.5 版本开始，Spring 也提供了基于 JCA`MessageListener`容器的支持。`JmsMessageEndpointManager`将根据提供者`ResourceAdapter`的类名自动地决定`ActivationSpec`类名。因此，通常它只提供如下例所示的 Spring 的通用`JmsActivationSpecConfig`。

```
<bean class="org.springframework.jms.listener.endpoint.JmsMessageEndpointManager">
	<property name="resourceAdapter" ref="resourceAdapter"/>
	<property name="activationSpecConfig">
		<bean class="org.springframework.jms.listener.endpoint.JmsActivationSpecConfig">
			<property name="destinationName" value="myQueue"/>
		</bean>
	</property>
	<property name="messageListener" ref="myMessageListener"/>
</bean>
```

或者，您可以使用给定的`ActivationSpec`对象设置`JmsMessageEndpointManager`。`ActivationSpec`对象也可能来自 JNDI 查找（使用`<jee：jndi-lookup>`）。

```
<bean class="org.springframework.jms.listener.endpoint.JmsMessageEndpointManager">
	<property name="resourceAdapter" ref="resourceAdapter"/>
	<property name="activationSpec">
		<bean class="org.apache.activemq.ra.ActiveMQActivationSpec">
			<property name="destination" value="myQueue"/>
			<property name="destinationType" value="javax.jms.Queue"/>
		</bean>
	</property>
	<property name="messageListener" ref="myMessageListener"/>
</bean>
```

使用 Spring 的`ResourceAdapterFactoryBean`，目标`ResourceAdapter`可以在本地配置，如以下示例所示。

```
<bean id="resourceAdapter" class="org.springframework.jca.support.ResourceAdapterFactoryBean">
	<property name="resourceAdapter">
		<bean class="org.apache.activemq.ra.ActiveMQResourceAdapter">
			<property name="serverUrl" value="tcp://localhost:61616"/>
		</bean>
	</property>
	<property name="workManager">
		<bean class="org.springframework.jca.work.SimpleTaskWorkManager"/>
	</property>
</bean>
```

指定的`WorkManager`也可能指向环境特定的线程池 – 通常通过`SimpleTaskWorkManager`的`asyncTaskExecutor`属性。如果，你恰好考虑使用多个适配器，为你的所有`ResourceAdapter`实例定义一个共享线程池。

在某些环境（例如 WebLogic 9或更高版本）中，可以从 JNDI 中获取整个`ResourceAdapter`对象（使用`<jee：jndi-lookup>`）。然后，基于Spring 的消息监听器可以与服务器托管的`ResourceAdapter`进行交互，也可以使用服务内置的`WorkManager`。

有关更多详细信息，请参阅`JMSMessageEndpointManager`、`JmsActivationSpecConfig`和“\`ResourceAdapterFactoryBean“的 JavaDoc。

Spring 还提供了一个通用的 JCA 消息端点管理器，它不绑定到 JMS ：`org.springframework.jca.endpoint.GenericMessageEndpointManager`。 它允许使用任何消息监听器类型（例如 CCI`MessageListener`）和任何提供者特定的`ActivationSpec`对象。从所涉及 JCA 提供者的文档可以找到这个连接器的实际能力，并参考“`GenericMessageEndpointManager`的 JavaDoc ”来了解 Spring 特有的配置详细信息。

> 基于 JCA 的消息端点管理器与 EJB 2.1的消息驱动 Bean 很相似；它使用了提供者们约定的相同底层资源。 与 EJB 2.1 MDB 一样，任何被 JCA 提供者支持的消息监听器接口都可以在 Spring 上下文中使用。尽管如此，Spring 仍为 JMS 提供了显式的“方便的”支持，很显然是因为 JMS 是 JCA 端点管理约定中最通用的端点 API。

## 26.6 注解驱动的监听端点

异步接收消息的最简单的方法是使用注解监听端点的基础架构。简而言之，它允许你暴露托管一个 bean 的方法作为一个 JMS 的监听端点。

```
@Component
public class MyService {

	@JmsListener(destination = "myDestination")
	public void processOrder(String data) { ... }
}
```

上述示例的想法是，每当`javax.jms.Destination`“myDestination” 上有消息可用时，就调用相应地`processOrder`方法（在这种情况下，JMS消息的内容类似于`MessageListenerAdapter`提供的内容）。

注解端点的基础架构使用`JmsListenerContainerFactory`为每个注解方法创建一个消息监听容器。这样的容器没有针对应用上下文进行注册，但是可以使用`JmsListenerEndpointRegistry`bean 进行简单的管理。

> `@JmsListener`是 Java 8上的可重复注解，因此可以通过向其添加额外的`@JmsListener`声明将多个JMS目的地关联到同一个方法。 在 Java 6和7上，你可以使用`@JmsListeners`注解。

### 26.6.1 启用监听端点的注解

要启用对`@JmsListener`注解的支持，请将`@EnableJms`添加到你的一个`@Configuration`类上。

```
@Configuration
@EnableJms
public class AppConfig {

	@Bean
	public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
		DefaultJmsListenerContainerFactory factory =
				new DefaultJmsListenerContainerFactory();
		factory.setConnectionFactory(connectionFactory());
		factory.setDestinationResolver(destinationResolver());
		factory.setConcurrency("3-10");
		return factory;
	}
}
```

默认情况下，基础架构将查找名为`jmsListenerContainerFactory`的 bean 作为用于创建消息监听容器的工厂源。 在这种情况下，忽略 JMS 基础架构的设置，`processOrder`方法可以在一个线程池中被调用，线程池核心数为3，最大线程数为10。

对于使用的每个注解都可以自定义监听容器工厂，或通过实现`JmsListenerConfigurer`接口来显示的配置默认值。仅在存在没有指定容器工厂的端点时，默认值才是必须的。有关详细信息和示例，请参阅 javadoc。

如果您喜欢XML配置，请使用`<jms：annotation-driven>`元素。

```
<jms:annotation-driven/>

   <bean id="jmsListenerContainerFactory"
           class="org.springframework.jms.config.DefaultJmsListenerContainerFactory">
       <property name="connectionFactory" ref="connectionFactory"/>
       <property name="destinationResolver" ref="destinationResolver"/>
       <property name="concurrency" value="3-10"/>
   </bean>
```

### 26.6.2 编程式端点注册

`JmsListenerEndpoint`提供了 JMS 端点的模型，并负责为该模型配置容器。除了使用注解之外，基础架构也允许你用编程的方式来配置端点。

```
@Configuration
@EnableJms
public class AppConfig implements JmsListenerConfigurer {

	@Override
	public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
		SimpleJmsListenerEndpoint endpoint = new SimpleJmsListenerEndpoint();
		endpoint.setId("myJmsEndpoint");
		endpoint.setDestination("anotherQueue");
		endpoint.setMessageListener(message -> {
			// processing
		});
		registrar.registerEndpoint(endpoint);
	}
}
```

本例中我们使用`SimpleJmsListenerEndpoint`来提供`MessageListener`，你也可以建立自己的端点变体并自定义调用机制。

应该注意的是，你完全可以不使用`@JmsListener`，而仅通过`JmsListenerConfigurer`来注册所有端点。

### 26.6.3 注解式端点方式签名

到目前为止，我们已经在我们的端点注入了一个简单的`String`，但实际上它可以有一个非常灵活的方法签名。现在让我们重写它并注入一个带有自定义头部的`Order`：

```
@Component
public class MyService {

   	@JmsListener(destination = "myDestination")
   	public void processOrder(Order order, @Header("order_type") String orderType) {
       	...
   	}
}
```

可以向 JMS 监听端点中注入的主要元素包括：

* 原始的`javax.jms.Message`或任意子类（当然，它与传入的消息类型相匹配）。
* 可选的`javax.jms.Session`来操作 JMS 原生 API，来发送自定义回复。
* 代表着接收消息的`org.springframework.messaging.Message`。注意此消息同时包含自定义和`JmsHeaders`定义的标准头。
* `@Header`注解的方法参数被用来提取一个特定的头部值，包括标准 JMS 头。`@Headers`
* 注解参数必须指定给一个`java.util.Map`，用来获取所有头。
* 不被支持的（如`Message`、`Session`等）、且无注解的元素被视为有效载荷。可以明确的给它们添加`@Payload`注解。也可以通过添加`@Valid`注解来开启校验。

注入 Spring 的`Message`抽象可以获取特定消息的所有信息，而无需依赖特定传输 API。

```
@JmsListener(destination = "myDestination")
public void processOrder(Message<Order> order) { ... }
```

可以自己扩展`DefaultMessageHandlerMethodFactory`来处理额外的方法参数。同时你也可以自定义转换和校验规则。

例如，如果我们需要在处理之前确保我们的`Order`有效，我们可以使用`@Valid`对有效负载进行注解，并配置必要的验证器，如下所示：

```
@Configuration
@EnableJms
public class AppConfig implements JmsListenerConfigurer {

   	@Override
   	public void configureJmsListeners(JmsListenerEndpointRegistrar registrar) {
       	registrar.setMessageHandlerMethodFactory(myJmsHandlerMethodFactory());
   	}

   	@Bean
   	public DefaultMessageHandlerMethodFactory myHandlerMethodFactory() {
       	DefaultMessageHandlerMethodFactory factory = new DefaultMessageHandlerMethodFactory();
       	factory.setValidator(myValidator());
       	return factory;
   	}
}
```

### 26.6.4 响应管理

`MessageListenerAdapter`允许你的方法有非空返回值。此时返回值将被封装在一个`javax.jms.Message`中，发往原始消息的`JMSReplyTo`头中定义的目的地或者监听自己默认的目的地。监听默认的目的地可以通过`@SendTo`注解来设置。

假定现在我们的`processOrder`方法将返回一个`OrderStatus`，下面将展示如何自动地发送响应：

```
@JmsListener(destination = "myDestination")
@SendTo("status")
public OrderStatus processOrder(Order order) {
   	// order processing
   	return status;
}
```

> 如果你有多个`@JmsListener`注解方法，您还可以将`@SendTo`注解放在 class 上以共享默认响应目的地。

如果您需要以独立传输的方式设置额外的头信息，则可以返回一个`Message`，如下所示：

```
@JmsListener(destination = "myDestination")
@SendTo("status")
public Message<OrderStatus> processOrder(Order order) {
   	// order processing
   	return MessageBuilder
       	    .withPayload(status)
           	.setHeader("code", 1234)
           	.build();
}
```

如果响应的目的地是运行时实时计算的，可以将响应结果封装在一个`JmsResponse`中，直接指定一个目的地。前面的例子可以重写如下：

```
@JmsListener(destination = "myDestination")
public JmsResponse<Message<OrderStatus>> processOrder(Order order) {
   	// order processing
   	Message<OrderStatus> response = MessageBuilder
       	    .withPayload(status)
           	.setHeader("code", 1234)
           	.build();
	return JmsResponse.forQueue(response, "status");
}
```

## 26.7 JMS 命名空间的支持

Spring 引入了 XML 命名空间以简化 JMS 的配置。使用 JMS 命名空间元素时，需要引用如下的 JMS Schema：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:jms="http://www.springframework.org/schema/jms"
		xsi:schemaLocation="
			http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms.xsd">

	<!-- bean definitions here -->

</beans>
```

命名空间由三个顶级元素组成：，和。可以使用[注解驱动的监听端点](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jms.html#jms-annotated)。 和定义共享监听容器的配置，并且包含了子元素。下面是一个基本配置的示例，包含两个监听器。

```
<jms:listener-container>

	<jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>

	<jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
```

上面的例子等同于在[第26.4.4节“MessageListenerAdapter”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/jms.html#jms-receiving-async-message-listener-adapter)的示例，定义两个不同的监听器容器和两个不同的`MessageListenerAdapter`。除了上面显示的属性之外，`listener`元素也包含几个可选属性。下面的表格列出了所有的属性：

##### 表26.1 JMS &lt;listner&gt;的元素属性

| 属性 | 描述 |
| :--- | :--- |
| id | 托管监听容器的 bean 名称。如果未指定，将自动生成一个 bean 名称。 |
| destination（必选） | 监听器的目的地名称，由`DestinationResolver`的策略决定。 |
| ref（必选） | 处理对象的 bean 名称。 |
| method | 处理器中被调用的方法名。如果 ref 指向`MessageListener`或者 Spring`SessionAwareMessageListener`，则这个属性可以被忽略。 |
| response-destination | 默认的响应目的地是发送响应消息抵达的目的地。 这用于请求消息没有包含`JMSReplyTo`域的情况。响应目的地类型被监听容器的`destination-type`属性决定。记住：这仅仅适用于有返回值的监听器方法，因为每个结果对象都会被转化成响应消息。 |
| subscription | 持久订阅的名称（如果需要的话）。 |
| selector | 监听器的一个可选的消息选择器。 |
| concurrency | 监听器启动的会话/消费者的并发数量。可以是表示最大数量（例如“5”）的简单数字，也可以是表示下限以及上限（例如“3-5”）的范围。 请注意，指定的最小值只是一个提示，在运行时可能会被忽略。默认值是容器提供的值。 |

&lt;listener-container /&gt;元素也接受几个可选属性。这允许自定义各种策略（例如，`taskExecutor`和`destinationResolver`）以及基本的 JMS 设置和资源引用。使用这些属性，可以定义很广泛的定制监听容器，同时仍享有命名空间的便利。

作为一个通过`factory-id`属性指定要暴露的 bean 的 id 的`JmsListenerContainerFactory`，自动暴露了这些设置。

```
jms:listener-container connection-factory="myConnectionFactory"
		task-executor="myTaskExecutor"
		destination-resolver="myDestinationResolver"
		transaction-manager="myTransactionManager"
		concurrency="10">

	<jms:listener destination="queue.orders" ref="orderService" method="placeOrder"/>

	<jms:listener destination="queue.confirmations" ref="confirmationLogger" method="log"/>

</jms:listener-container>
```

下面的表格描述了所有可用的属性。参考`AbstractMessageListenerContainer`类和具体子类的 Javadoc 来了解每个属性的细节。这部分的 Javadoc 也提高那个了事务选择和消息传输场景的讨论。

##### 表26.2 JMS &lt;listner-container&gt;的元素属性

| 属性 | 描述 |
| :--- | :--- |
| container-type | 监听容器的类型。 可用的选项有：default，simple，default102或simple102（默认值为“default”）。 |
| container-class | 自定义监听容器的实现类作为完全限定类名。默认是 Spring 的标准`DefaultMessageListenerContainer`或`SimpleMessageListenerContainer`，取决于`container-type`属性。 |
| factory-id | 通过对`JmsListenerContainerFactory`指定 id，暴露该元素的设置，以便它们可以与其他端点一起使用。 |
| connection-factory | 对 JMS`ConnectionFactory`bean 的引用（默认 bean 名称为`connectionFactory`）。 |
| task-executor | 对JMS监听器调用者的 Spring`TaskExecutor`的引用。 |
| destination-resolver | 对解决 JMS 目标的`DestinationResolver`策略的引用。 |
| message-converter | 对 JMS 消息转换为监听器方法参数的`MessageConverter`策略的引用。 默认是一个`SimpleMessageConverter`。 |
| error-handler | 对处理任何未捕获异常的`ErrorHandler`策略的引用，异常可能发生在`MessageListener`的执行期间。 |
| destination-type | 监听器的 JMS 目标类型：queue，topic，durableTopic，sharedTopic 或 sharedDurableTopic。 这样可以间接启用容器的`pubSubDomain`，`subscriptionDurable`和`subscriptionShared`属性。 默认是队列（即禁用这3个属性）。 |
| response-destination-type | 响应的JMS目标类型：queue、topic。 默认值为`destination-type`属性的值。 |
| client-id | 监听容器的 JMS 客户 ID。 使用持久订阅时需要指定。 |
| cache | JMS资源的缓存级别：none, connection, session, consumer 或者 auto。 默认情况下（auto），缓存级别有效的是“consumer”。除非已经指定了外部事务管理器，在这种情况下，有效的默认值为 none（假设 Java EE 风格的事务管理，其中给定的`ConnectionFactory`是 XA-aware 池）。 |
| acknowledge | 原生JMS确认模式：auto，client，dups-ok 或 transacted。transacted 值激活了本地交易的会话。 或者，指定下面描述的`transaction-manager`属性。默认为 auto。 |
| transaction-manager | 对外部`PlatformTransactionManager`的引用（通常是基于XA的事务协调器，例如 Spring 的`JtaTransactionManager`）。如果未指定，将使用本地确认（请参阅`acknowledge`属性）。 |
| concurrency | 每个监听器启动的会话/消费者的并发数量。 可以是表示最大数量（例如“5”）的简单数字，也可以是表示下限以及上限（例如“3-5”）的范围。 请注意，指定的最小值只是一个提示，在运行时可能会被忽略。 默认值为1；在 topic 监听器或者 queue 的次序很重要的情况下，将并发限制为1；一般的queue可以考虑提高并发数。 |
| prefetch | 要加载到单个会话的消息的最大数量。请注意，提高此数量可能会导致并发消费者的饥饿！ |
| receive-timeout | 用于接收调用的超时时间（以毫秒为单位）。 默认值为1000 ms（1秒）；-1表示没有超时限制。 |
| back-off | 指定用于计算恢复尝试间隔的`BackOff`实例。 如果`BackOffExecution`实现返回`BackOffExecution＃STOP`，监听容器将不再进一步尝试恢复。 当设置此属性时，将忽略`recovery-interval`值。 默认值为`FixedBackOff`，间隔为5000 ms，即5秒。 |
| recovery-interval | 指定恢复尝试之间的间隔（以毫秒为单位）。是以指定间隔创建`FixedBackOff`的便捷方式。 有关更多恢复选项，请考虑指定`BackOff`实例。 默认值为5000 ms，即5秒。 |
| phase | 此容器应在其中开始和停止的生命周期阶段。 值越小，容器就越早启动，并且更晚停止。 默认值为`Integer`。`MAX_VALUE`，意味着容器将尽可能晚地启动并尽快停止。 |

使用 jms Schema 支持来配置基于 JCA 的监听器容器很相似。

```
<jms:jca-listener-container resource-adapter="myResourceAdapter"
		destination-resolver="myDestinationResolver"
		transaction-manager="myTransactionManager"
		concurrency="10">

	<jms:listener destination="queue.orders" ref="myMessageListener"/>

</jms:jca-listener-container>
```

JCA 可用的配置选项描述如下表：

##### 表26.3 JMS &lt;jca-listner-container&gt;的元素属性

| 属性 | 描述 |
| :--- | :--- |
| factory-id |  |
| resource-adapter | 对 JCA`ResourceAdapter`bean 的引用（默认 bean 名称是`resourceAdapter`）。 |
| activation-spec-factory | 对`JmsActivationSpecFactory`的引用。 默认情况是自动检测 JMS 供应商及其`ActivationSpec`类（请参阅`DefaultJmsActivationSpecFactory`） |
| destination-resolver | 对解决JMS目标的`DestinationResolver`策略的引用。 |
| message-converter | 对 JMS 消息转换为监听器方法参数的`MessageConverter`策略的引用。 默认是一个`SimpleMessageConverter`。 |
| error-handler | 对处理任何未捕获异常的`ErrorHandler`策略的引用，异常可能发生在`MessageListener`的执行期间。 |
| destination-type | 监听器的JMS目标类型：queue，topic，durableTopic，sharedTopic或sharedDurableTopic。 这样可以间接启用容器的pubSubDomain，subscriptionDurable和subscriptionShared属性。 默认是队列（即禁用这3个属性）。 |
| response-destination-type | 响应的JMS目标类型：“queue”，“topic”。 默认值为“destination-type”属性的值。 |
| client-id | 监听容器的 JMS 客户 ID。 使用持久订阅时需要指定。 |
| acknowledge | 原生JMS确认模式：auto，client，dups-ok 或 transacted。transacted 值激活了本地交易的会话。 或者，指定下面描述的`transaction-manager`属性。 默认为 auto。 |
| transaction-manager | 对 Spring`JtaTransactionManager`或`javax.transaction.TransactionManager`的引用，为每个传入消息启动 XA 事务。 如果未指定，将使用本地确认（请参阅`acknowledge`属性）。 |
| concurrency | 每个监听器启动的会话/消费者的并发数量。 可以是表示最大数量（例如“5”）的简单数字，也可以是表示下限以及上限（例如“3-5”）的范围。 请注意，指定的最小值只是一个提示，在使用 JCA 监听容器的运行时可能会被忽略。 默认值为1; |
| prefetch | 要加载到单个会话的消息的最大数量。 请注意，提高此数量可能会导致并发消费者的饥饿！ |



