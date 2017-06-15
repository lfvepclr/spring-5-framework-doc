## 16.1 介绍一下Spring中的ORM

Spring框架在实现资源管理、数据访问对象（DAO）层，和事务策略等方面，支持对Java持久化API（JPA）以及原生Hibernate的集成。以Hibernate举例来说，Spring有非常赞的IoC功能，可以解决许多典型的Hibernate配置和集成问题。开发者可以通过依赖注入来配置O-R（对象关系）映射组件支持的特性。Hibernate的这些特性可以参与Spring的资源和事务管理，并且符合Spring的通用事务和DAO层的异常体系。因此，Spring团队推荐开发者使用Spring集成的方式来开发DAO层，而不是使用原生的Hibernate或者JPA的API。老版本的Spring DAO模板现在不推荐使用了，想了解这部分内容可以参考[经典ORM使用](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/classic-spring.html#classic-spring-orm)一节。

当开发者创建数据访问应用程序时，Spring会为开发者选择的ORM层对应功能进行优化。而且，开发者可以根据需要来利用Spring对集成ORM的支持，开发者应该将此集成工作与维护内部类似的基础架构服务的成本和风险进行权衡。同时，开发者在使用Spring集成的时候可以很大程度上不用考虑技术，将ORM的支持当做一个库来使用，因为所有的组件都被设计为可重用的JavaBean组件了。Spring IoC容器中的ORM十分易于配置和部署。本节中的大多数示例都是讲解在Spring容器中的来如何配置。

开发者使用Spring框架来中创建自己的ORM DAO的好处如下：

* *易于测试*。Spring IoC的模式使得开发者可以轻易的替换Hibernate的`SessionFactory`实例，JDBC的`DataSource`
实例，事务管理器，以及映射对象（如果有必要）的配置和实现。这一特点十分利于开发者对每个模块进行独立的测试。
* *泛化数据访问异常*。Spring可以将ORM工具的异常封装起来，将所有异常（可以是受检异常）封装成运行时的`DataAccessException`体系。这一特性可以令开发者在合适的逻辑层上处理绝大多数不可修复的持久化异常，避免了大量的`catch`,`throw`和异常的声明。开发者还可以按需来处理这些异常。其中，JDBC异常（包括一些特定DB语言）都会被封装为相同的体系，意味着开发者即使使用不同的JDBC操作，基于不同的DB，也可以保证一致的编程模型。
* *通用的资源管理*。Spring的应用上下文可以通过处理配置源的位置来灵活配置Hibernate的`SessionFactory`实例，JPA的`EntityManagerFactory`实例，JDBC的`DataSource`实例以及其他类似的资源。Spring的这一特性使得这些实例的配置十分易于管理和修改。同时，Spring还为处理持久化资源的配置提供了高效，易用和安全的处理方式。举个例子，有些代码使用了Hibernate需要使用相同的`Session`来确保高效性和正确的事务处理。Spring通过Hibernate的`SessionFactory`来获取当前的`Session`，来透明的将`Session`绑定到当前的线程。Srping为任何本地或者JTA事务环境解决了在使用Hibernate时碰到的一些常见问题。
* *集成事务管理*。开发者可以通过`@Transactional`注解或在XML配置文件中显式配置事务AOP Advise拦截，将ORM代码封装在声明式的AOP方法拦截器中。事务的语义和异常处理（回滚等）都可以根据开发者自己的需求来定制。在后面的章节中，*资源和事务管理*中，开发者可以在不影响ORM相关代码的情况下替换使用不同的事务管理器。例如，开发者可以在本地事务和JTA之间进行交换，并在两种情况下具有相同的完整服务（如声明式事务）。而且，JDBC相关的代码在事务上完全和处理ORM部分的代码集成。这对于不适用于ORM的数据访问非常有用，例如批处理和BLOB流式传输，仍然需要与ORM操作共享常见事务。

> 为了更全面的ORM支持，包括支持其他类型的数据库技术（如MongoDB），开发者可能需要查看[Spring Data](http://projects.spring.io/spring-data/)系列项目。如果开发者是JPA用户，则可以从https://spring.io的查阅[开始使用JPA访问数据指南](https://spring.io/guides/gs/accessing-data-jpa/)一文进行简单了解。

## 16.2 集成ORM的注意事项

本节重点介绍适用于所有集成ORM技术的注意事项。在16.3[Hibernate]()一节中提供了很多关于如何配置和使用这些特性提的信息。

Spring对ORM集成的主要目的是使应用层次化，可以任意选择数据访问和事务管理技术，并且为应用对象提供松耦合结构。不再将业务逻辑依赖于数据访问或者事务策略上，不再使用基于硬编码的资源查找，不再使用难以替代的单例，不再自定义服务的注册。同时，为应用提供一个简单和一致的方法来装载对象，保证他们的重用并且尽可能不依赖于容器。所有单独的数据访问功能都可以自己使用，也可以很好地与Spring的`ApplicationContext`集成，提供基于XML的配置和不需要Spring感知的普通`JavaBean`实例。在典型的Spring应用程序中，许多重要的对象都是`JavaBean`：数据访问模板，数据访问对象，事务管理器，使用数据访问对象和事务管理器的业务服务，Web视图解析器，使用业务服务的Web控制器等等。

### 16.2.1 资源和事务管理

通常企业应用都会包含很多重复的的资源管理代码。很多项目总是尝试去创造自己的解决方案，有时会为了开发的方便而牺牲对错误的处理。Spring为资源的配置管理提供了简单易用的解决方案，在JDBC上使用模板技术，在ORM上使用AOP拦截技术。

Spring的基础设施提供了合适的资源处理，同时Spring引入了DAO层的异常体系，可以适用于任何数据访问策略。对于JDBC直连来说，前面提及到的`JdbcTemplate`类提供了包括连接处理，对`SQLException`到`DataAccessException`的异常封装，同时还包含对于一些特定数据库SQL错误代码的转换。对于ORM技术来说，可以参考下一节来了解异常封装的优点。

当谈到事务管理时，`JdbcTemplate`类通过Spring事务管理器挂接到Spring事务支持，并支持JTA和JDBC事务。Spring通过Hibernate，JPA事务管理器和JTA的支持来提供Hibernate和JPA这类ORM技术的支持。想了解更多关于事务的描述，可以参考第13章，[事务管理](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html)。

### 16.2.2 异常转义

当在DAO层中使用Hibernate或者JPA的时候，开发者必须决定该如何处理持久化技术的一些原生异常。DAO层会根据选择技术的不同而抛出`HibernateException`或者`PersistenceException`。这些异常都属于运行时异常，所以无需显式声明和捕捉。同时，开发者同时还需要处理`IllegalArgumentException`和`IllegalStateException`这类异常。一般情况下，调用方通常只能将这一类异常视为致命的异常，除非他们想要自己的应用依赖于持久性技术原生的异常体系。如果需要捕获一些特定的错误，比如乐观锁获取失败一类的错误，只能选择调用方和实现策略耦合到一起。对于那些只基于某种特定ORM技术或者不需要特殊异常处理的应用来说，使用ORM本身的异常体系的代价是可以接受的。但是，Spring可以通过`@Repository`注解透明地应用异常转换，以解耦调用方和ORM技术的耦合：

```
@Repository
public class ProductDaoImpl implements ProductDao {

	// class body here...

}
```
```
<beans>

	<!-- Exception translation bean post processor -->
	<bean class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"/>

	<bean id="myProductDao" class="product.ProductDaoImpl"/>

</beans>
```

上面的后置处理器`PersistenceExceptionTranslationPostProcessor`，会自动查找所有的异常转义器（实现`PersistenceExceptionTranslator`接口的Bean），并且拦截所有标记为`@Repository`注解的Bean，通过代理来拦截异常，然后通过`PersistenceExceptionTranslator`将DAO层异常转义后的异常抛出。

总而言之：开发者可以既基于简单的持久化技术的API和注解来实现DAO，同时还受益于Spring管理的事务，依赖注入和透明异常转换（如果需要）到Spring的自定义异常层次结构。