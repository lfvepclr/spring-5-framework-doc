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
## 16.3 Hibernate

我们将首先介绍Spring环境中的[Hibernate 5](http://hibernate.org)，然后通过使用Hibernate 5来演示Spring集成O-R映射器的方法。本节将详细介绍许多问题，并显示DAO实现和事务划分的不同变体。这些模式中大多数可以直接转换为所有其他支持的ORM工具。本章中的以下部分将通过简单的例子来介绍其他ORM技术。

> 从Spring 5.0开始，Spring需要Hibernate ORM对JPA的支持要基于4.3或更高的版本，甚至Hibernate ORM 5.0+可以针对本机Hibernate Session API进行编程。请注意，Hibernate团队可能不会在5.0之前维护任何版本，仅仅专注于5.2以后的版本。

### 16.3.1 在Spring容器中配置SessionFactory

开发者可以将资源如JDBC`DataSource`或Hibernate`SessionFactory`定义为Spring容器中的bean来避免将应用程序对象绑定到硬编码的资源查找上。应用对象需要访问资源的时候，都通过对应的Bean实例进行间接查找，详情可以通过下一节的DAO的定义来参考。

下面引用的应用的XML元数据定义就展示了如何配置JDBC的`DataSource`和`Hibernate`的`SessionFactory`的：

```
<beans>
	<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName" value="org.hsqldb.jdbcDriver"/>
		<property name="url" value="jdbc:hsqldb:hsql://localhost:9001"/>
		<property name="username" value="sa"/>
		<property name="password" value=""/>
	</bean>

	<bean id="mySessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
		<property name="dataSource" ref="myDataSource"/>
		<property name="mappingResources">
			<list>
				<value>product.hbm.xml</value>
			</list>
		</property>
		<property name="hibernateProperties">
			<value>
				hibernate.dialect=org.hibernate.dialect.HSQLDialect
			</value>
		</property>
	</bean>
</beans>
```

这样，从本地的Jaksrta Commons DBCP的`BasicDataSource`转换到JNDI定位的`DataSource`仅仅只需要修改配置文件。

```
<beans>
	<jee:jndi-lookup id="myDataSource" jndi-name="java:comp/env/jdbc/myds"/>
</beans>
```

开发者也可以通过Spring的`JndiObjectFactoryBean`或者`<jee:jndi-lookup>`来获取对应Bean以访问JNDI定位的`SessionFactory`。但是，JNDI定位的`SessionFactory`在EJB上下文不常见。

### 16.3.2 基于Hibernate API来实现DAO

Hibernate有一个特性称之为上下文会话，在每个Hibernate本身每个事务都管理一个当前的`Session`。这大致相当于Spring每个事务的一个Hibernate`Session`的同步。如下的DAO的实现类就是基于简单的Hibernate API实现的：

```
public class ProductDaoImpl implements ProductDao {

	private SessionFactory sessionFactory;

	public void setSessionFactory(SessionFactory sessionFactory) {
		this.sessionFactory = sessionFactory;
	}

	public Collection loadProductsByCategory(String category) {
		return this.sessionFactory.getCurrentSession()
				.createQuery("from test.Product product where product.category=?")
				.setParameter(0, category)
				.list();
	}
}
```

除了需要在实例中持有`SessionFactory`引用以外，上面的代码风格跟Hibernate文档中的例子十分相近。Spring团队强烈建议使用这种基于实例变量的实现风格，而非守旧的`static HibernateUtil`风格(总的来说，除非*绝对*必要，否则尽量不要使用`static`变量来持有资源)。

上面DAO的实现完全符合Spring依赖注入的样式：这种方式可以很好的集成Spring IoC容器，就好像Spring的`HibernateTemplate`代码一样。当然，DAO层的实现也可以通过纯Java的方式来配置（比如在UT中）。简单实例化`ProductDaoImpl`并且调用`setSessionFactory(...)`即可。当然，也可以使用Spring bean来进行注入，参考如下XML配置：

```
<beans>
	<bean id="myProductDao" class="product.ProductDaoImpl">
		<property name="sessionFactory" ref="mySessionFactory"/>
	</bean>
</beans>
```

上面的DAO实现方式的好处在于只依赖于Hibernate API，而无需引入Spring的class。这从非侵入性的角度来看当然是有吸引力的，毫无疑问，这种开发方式会令Hibernate开发人员将会更加自然。

然而，DAO层会抛出Hibernate自有异常`HibernateException`（属于非检查异常，无需显式声明和使用try-catch），但是也意味着调用方会将异常看做致命异常——除非调用方将Hibernate异常体系作为应用的异常体系来处理。而在这种情况下，除非调用方自己来实现一定的策略，否则捕获一些诸如乐观锁失败之类的特定错误是不可能的。对于强烈基于Hibernate的应用程序或不需要对特殊异常处理的应用程序，这种代价可能是可以接受的。

幸运的是，Spring的`LocalSessionFactoryBean`可以通过Hibernate的`SessionFactory.getCurrentSession()`方法为所有的Spring事务策略提供支持，使用`HibernateTransactionManager`返回当前的Spring管理的事务的`Session`。当然，该方法的标准行为仍然是返回与正在进行的JTA事务相关联的当前`Session`（如果有的话）。无论开发者是使用Spring的`JtaTransactionManager`，EJB容器管理事务（CMT）还是JTA，都会适用此行为。

总而言之：开发者可以基于纯Hibernate API来实现DAO，同时也可以集成Spring来管理事务。

### 16.3.3 声明式事务划分

Spring团队建议开发者使用Spring声明式的事务支持，这样可以通过AOP事务拦截器来替代事务API的显式调用。AOP事务拦截器可以在Spring容器中使用XML或者Java的注解来进行配置。这种事务拦截器可以令开发者的代码和重复的事务代码相解耦，而开发者可以将精力更多集中在业务逻辑上，而业务逻辑才是应用的核心。

> 在继续之前，强烈建议开发者先查阅章节[13.5 声明式事务管理](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/transaction.html#transaction-declarative)的内容。

开发者可以在服务层的代码使用注解`@Transactional`，这样可以让Spring容器找到这些注解，以对其中注解了的方法提供事务语义。

```
public class ProductServiceImpl implements ProductService {

	private ProductDao productDao;

	public void setProductDao(ProductDao productDao) {
		this.productDao = productDao;
	}

	@Transactional
	public void increasePriceOfAllProductsInCategory(final String category) {
		List productsToChange = this.productDao.loadProductsByCategory(category);
		// ...
	}

	@Transactional(readOnly = true)
	public List<Product> findAllProducts() {
		return this.productDao.findAllProducts();
	}

}
```

开发者所需要做的就是在容器中配置`PlatformTransactionManager`的实现，或者是在XML中配置`<tx:annotation-driver/>`标签，这样就可以在运行时支持`@Transactional`的处理了。参考如下XML代码：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop.xsd">

	<!-- SessionFactory, DataSource, etc. omitted -->

	<bean id="transactionManager"
			class="org.springframework.orm.hibernate5.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory"/>
	</bean>

	<tx:annotation-driven/>

	<bean id="myProductService" class="product.SimpleProductService">
		<property name="productDao" ref="myProductDao"/>
	</bean>
</beans>
```

### 16.3.4 编程式事务划分

开发者可以在应用程序的更高级别上对事务进行标定，而不用考虑低级别的数据访问执行了多少操作。这样不会对业务服务的实现进行限制；只需要定义一个Spring的`PlatformTransactionManager`即可。当然，`PlatformTransactionManager`可以从多处获取，但最好是通过`setTransactionManager(..)`方法以Bean来注入，正如`ProductDAO`应该由`setProductDao(..)`方法配置一样。下面的代码显示Spring应用程序上下文中的事务管理器和业务服务的定义，以及业务方法实现的示例：

```
<bean id="myTxManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
		<property name="sessionFactory" ref="mySessionFactory"/>
	</bean>

	<bean id="myProductService" class="product.ProductServiceImpl">
		<property name="transactionManager" ref="myTxManager"/>
		<property name="productDao" ref="myProductDao"/>
	</bean>
</beans>
```
```
public class ProductServiceImpl implements ProductService {

	private TransactionTemplate transactionTemplate;
	private ProductDao productDao;

	public void setTransactionManager(PlatformTransactionManager transactionManager) {
		this.transactionTemplate = new TransactionTemplate(transactionManager);
	}

	public void setProductDao(ProductDao productDao) {
		this.productDao = productDao;
	}

	public void increasePriceOfAllProductsInCategory(final String category) {
		this.transactionTemplate.execute(new TransactionCallbackWithoutResult() {
			public void doInTransactionWithoutResult(TransactionStatus status) {
				List productsToChange = this.productDao.loadProductsByCategory(category);
				// do the price increase...
			}
		});
	}
}
```

Spring的`TransactionInterceptor`允许任何检查的应用异常到`callback`代码中去，而`TransactionTemplate`还会非受检异常触发进行回调。`TransactionTemplate`则会因为非受检异常或者是由应用标记事务回滚(通过`TransactionStatus`)。`TransactionInterceptor`也是一样的处理逻辑，但是同时还允许基于方法配置回滚策略。

### 16.3.5 事务管理策略

无论是`TransactionTemplate`或者是`TransactionInterceptor`都将实际的事务处理代理到`PlatformTransactionManager`实例上来进行处理的，这个实例的实现可以是一个`HibernateTransactionManager`(包含一个Hibernate的`SessionFactory`通过使用`ThreadLocal`的`Session`)，也可以是`JatTransactionManager`(代理到容器的JTA子系统)。开发者甚至可以使用一个自定义的`PlatformTransactionManager`的实现。现在，如果应用有需求需要需要部署分布式事务的话，只是一个配置变化，就可以从本地Hibernate事务管理切换到JTA。简单地用Spring的JTA事务实现来替换Hibernate事务管理器即可。因为引用的`PlatformTransactionManager`的是通用事务管理API，事务管理器之间的切换是无需修改代码的。

对于那些跨越了多个Hibernate会话工厂的分布式事务，只需要将`JtaTransactionManager`和多个`LocalSessionFactoryBean`定义相结合即可。每个DAO之后会获取一个特定的`SessionFactory`引用。如果所有底层JDBC数据源都是事务性容器，那么只要使用`JtaTransactionManager`作为策略实现，业务服务就可以划分任意数量的DAO和任意数量的会话工厂的事务。

无论是`HibernateTransactionManager`还是`JtaTransactionManager`都允许使用JVM级别的缓存来处理Hibernate，无需基于容器的事务管理器查找，或者JCA连接器（如果开发者没有使用EJB来实例化事务的话）。

`HibernateTransactionManager`可以为指定的数据源的Hibernate JDBC的`Connection`转成为纯JDBC的访问代码。如果开发者仅访问一个数据库，则开发者完全可以不使用JTA，通过Hibernate和JDBC数据访问进行高级别事务划分。如果开发者已经通过`LocalSessionFactoryBean`的`dataSource`属性与`DataSource`设置了传入的`SessionFactory`，`HibernateTransactionManager`会自动将Hibernate事务公开为JDBC事务。或者，开发者可以通过`HibernateTransactionManager`的`dataSource`属性的配置以确定公开事务的类型。

### 16.3.6 对比由容器管理的和本地定义的资源

开发者可以在不修改一行代码的情况下，在容器管理的JNDI`SessionFactory`和本地定义的`SessionFactory`之间进行切换。是否将资源定义保留在容器中，还是仅仅留在应用中，都取决于开发者使用的事务策略。相对于Spring定义的本地`SessionFactory`来说，手动注册的JNDI`SessionFactory`没有什么优势。通过Hibernate的JCA连接器来发布一个`SessionFactory`只会令代码更符合J2EE服务标准，但是并不会带来任何实际的价值。

Spring对事务支持不限于容器。使用除JTA之外的任何策略配置，事务都可以在独立或测试环境中工作。特别是在单数据库事务的典型情况下，Spring的单一资源本地事务支持是一种轻量级和强大的替代JTA的解决方案。当开发者使用本地EJB无状态会话Bean来驱动事务时，即使只访问单个数据库，并且只使用无状态会话Bean来通过容器管理的事务来提供声明式事务，开发者的代码依然是依赖于EJB容器和JTA的。同时，以编程方式直接使用JTA也需要一个J2EE环境的。 JTA不涉及JTA本身和JNDI DataSource实例方面的容器依赖关系。对于非Spring，JTA驱动的Hibernate事务，开发者必须使用Hibernate JCA连接器或开发额外的Hibernate事务代码，并为JVM级缓存正确配置`TransactionManagerLookup`。

Spring驱动的事务可以与本地定义的Hibernate`SessionFactory`一样工作，就像本地JDBC DataSource访问单个数据库一样。但是，当开发者有分布式事务的要求的情况下，只能选择使用Spring JTA事务策略。JCA连接器是需要特定容器遵循一致的部署步骤的，而且显然JCA支持是需要放在第一位的。JCA的配置需要比部署本地资源定义和Spring驱动事务的简单web应用程序需要更多额外的的工作。同时，开发者还需要使用容器的企业版，比如，如果开发者使用的是WebLogic Express的非企业版，就是不支持JCA的。具有跨越单个数据库的本地资源和事务的Spring应用程序适用于任何基于J2EE的Web容器（不包括JTA，JCA或EJB），如Tomcat，Resin或甚至是Jetty。此外，开发者可以轻松地在桌面应用程序或测试套件中重用中间层代码。

综合前面的叙述，如果不使用EJB，请尽量使用本地的`SessionFactory`设置和Spring的`HibernateTransactionManager`或`JtaTransactionManager`。开发者能够得到了前面提到的所有好处，包括适当的事务性JVM级缓存和分布式事务支持，而且没有容器部署的不便。只有必须配合EJB使用的时候，JNDI通过JCA连接器来注册Hibernate`SessionFactory`才有价值。

### 16.3.7 Hibernate的虚假应用服务器警告

在某些具有非常严格的`XADataSource`实现的JTA环境（目前只有一些WebLogic Server和WebSphere版本）中，当配置Hibernate时，没有考虑到JTA的 `PlatformTransactionManager`对象，可能会在应用程序服务器日志中显示虚假警告或异常。这些警告或异常经常描述正在访问的连接不再有效，或者JDBC访问不再有效。这通常可能是因为事务不再有效。例如，这是WebLogic的一个实际异常：

```
java.sql.SQLException: The transaction is no longer active - status: 'Committed'. No
further JDBC access is allowed within this transaction.
```

开发者可以通过配置令Hibernate意识到Spring中同步的JTA`PlatformTransactionManager`实例的存在，这样即可消除掉前面所说的虚假警告信息。开发者有以下两种选择：

* 如果在应用程序上下文中，开发者已经直接获取了JTA `PlatformTransactionManager`对象（可能是从JNDI到`JndiObjectFactoryBean`或者`<jee：jndi-lookup>`标签），并将其提供给Spring的`JtaTransactionManager`（其中最简单的方法就是指定一个引用bean将此JTA `PlatformTransactionManager`实例定义为`LocalSessionFactoryBean`的`jtaTransactionManager`属性的值）。 Spring之后会令`PlatformTransactionManager`对象对Hibernate可见。
* 更有可能开发者无法获取JTA`PlatformTransactionManager`实例，因为Spring的`JtaTransactionManager`是可以自己找到该实例的。因此，开发者需要配置Hibernate令其直接查找JTA `PlatformTransactionManager`。开发者可以如Hibernate手册中所述那样通过在Hibernate配置中配置应用程序服务器特定的`TransactionManagerLookup`类来执行此操作。

本节的其余部分描述了在`PlatformTransactionManager`对Hibernate可见和`PlatformTransactionManager`对Hibernate不可见的情况下发生的事件序列:

当Hibernate未配置任何对JTA`PlatformTransactionManager`的进行查找时，JTA事务提交时会发生以下事件：

* JTA事务提交
* Spring的`JtaTransactionManager`与JTA事务同步，所以它被JTA事务管理器通过`afterCompletion`回调调用。
* 在其他活动中，此同步令Spring通过Hibernate的`afterTransactionCompletion`触发回调（用于清除Hibernate缓存），然后在Hibernate Session上调用`close()`，从而令Hibernate尝试`close()`JDBC连接。
* 在某些环境中，因为事务已经提交，应用程序服务器会认为`Connection`不可用，导致`Connection.close()`调用会触发警告或错误。

当Hibernate配置了对JTA`PlatformTransactionManager`进行查找时，JTA事务提交时会发生以下事件：

* JTA事务准备提交
* Spring的`JtaTransactionManager`与JTA事务同步，所以JTA事务管理器通过`beforeCompletion`方法来回调事务。
* Spring确定Hibernate与JTA事务同步，并且行为与前一种情况不同。假设Hibernate Session需要关闭，Spring将会关闭它。
* JTA事务提交。
* Hibernate与JTA事务同步，所以JTA事务管理器通过`afterCompletion`方法回调事务，可以正确清除其缓存。
## 16.4 JPA

Spring JPA在`org.springframework.orm.jpa`包中已经可用，Spring JPA用了Hibernate集成相似的方法来提供更易于理解的JPA支持，与此同时，了解了JPA底层实现，可以理解更多的Spring JPA特性。

### 16.4.1 Spring中JPA配置的三个选项

Spring JPA支持提供了三种配置JPA`EntityManagerFactory`的方法，之后通过`EntityManagerFactory`来获取对应的实体管理器。

#### LocalEntityManagerFactoryBean

> 通常只有在简单的部署环境中使用此选项，例如在独立应用程序或者进行集成测试时，才会使用这种方式。

`LocalEntityManagerFactoryBean`创建一个适用于应用程序且仅使用JPA进行数据访问的简单部署环境的`EntityManagerFactory`。工厂bean会使用JPA`PersistenceProvider`自动检测机制，并且在大多数情况下，仅要求开发者指定持久化单元的名称：

```
<beans>
	<bean id="myEmf" class="org.springframework.orm.jpa.LocalEntityManagerFactoryBean">
		<property name="persistenceUnitName" value="myPersistenceUnit"/>
	</bean>
</beans>
```

这种形式的JPA部署是最简单的，同时限制也很多。开发者不能引用现有的JDBC`DataSource` bean定义，并且不支持全局事务。而且，持久化类的织入(weaving)(字节码转换)是特定于提供者的，通常需要在启动时指定特定的JVM代理。该选项仅适用于符合JPA Spec的独立应用程序或测试环境。

#### 从JNDI中获取EntityManagerFactory

> 在部署到J2EE服务器时可以使用此选项。检查服务器的文档来了解如何将自定义JPA提供程序部署到服务器中，从而对服务器进行比默认更多的个性化定制。

从JNDI获取`EntityManagerFactory`(例如在Java EE环境中)，只需要在XML配置中加入配置信息即可：

```
<beans>
	<jee:jndi-lookup id="myEmf" jndi-name="persistence/myPersistenceUnit"/>
</beans>
```

此操作将采用标准J2EE引导：J2EE服务器自动检测J2EE部署描述符（例如web.xml）中persistence-unit-ref条目和持久性单元（实际上是应用程序jar中的META-INF/persistence.xml文件），并为这些持久性单元定义环境上下文位置。

在这种情况下，整个持久化单元部署（包括持久化类的织入(weaving)（字节码转换））都取决于J2EE服务器。JDBC `DataSource`通过META-INF/persistence.xml文件中的JNDI位置进行定义; 而`EntityManager`事务与服务器JTA子系统集成。 Spring仅使用获取的`EntityManagerFactory`，通过依赖注入将其传递给应用程序对象，通常通过`JtaTransactionManager`来管理持久性单元的事务。

如果在同一应用程序中使用多个持久性单元，则这种JNDI检索的持久性单元的bean名称应与应用程序用于引用它们的持久性单元名称相匹配，例如`@PersistenceUnit`和`@PersistenceContext`注释。

#### LocalContainerEntityManagerFactoryBean

> 在基于Spring的应用程序环境中使用此选项来实现完整的JPA功能。这包括诸如Tomcat的Web容器，以及具有复杂持久性要求的独立应用程序和集成测试。

`LocalContainerEntityManagerFactoryBean`可以完全控制`EntityManagerFactory`的配置，同时适用于需要细粒度定制的环境。 `LocalContainerEntityManagerFactoryBean`会基于`persistence.xml`文件，`dataSourceLookup`策略和指定的`loadTimeWeaver`来创建一个`PersistenceUnitInfo`实例。因此，可以在JNDI之外使用自定义数据源并控制织入(weaving)过程。以下示例显示`LocalContainerEntityManagerFactoryBean`的典型Bean定义：

```
<beans>
	<bean id="myEmf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="someDataSource"/>
		<property name="loadTimeWeaver">
			<bean class="org.springframework.instrument.classloading.InstrumentationLoadTimeWeaver"/>
		</property>
	</bean>
</beans>
```
下面的例子是一个典型的persistence.xml文件：

```
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="1.0">
	<persistence-unit name="myUnit" transaction-type="RESOURCE_LOCAL">
		<mapping-file>META-INF/orm.xml</mapping-file>
		<exclude-unlisted-classes/>
	</persistence-unit>
</persistence>
```

> `<exclude-unlisted-classes />`标签表示不会进行注解实体类的扫描。指定的显式`true`值 -  `<exclude-unlisted-classes>true</exclude-unlisted-classes/>`- 也意味着不进行扫描。`<exclude-unlisted-classes> false</exclude-unlisted-classes>`则会触发扫描;但是，如果开发者需要进行实体类扫描，建议开发者简单地省略`<exclude-unlisted-classes>`元素。

`LocalContainerEntityManagerFactoryBean`是最强大的JPA设置选项，允许在应用程序中进行灵活的本地配置。它支持连接到现有的JDBC`DataSource`，支持本地和全局事务等。但是，它对运行时环境施加了需求，其中之一就是如果持久性提供程序需要字节码转换，就需要有织入(weaving)能力的类加载器。

此选项可能与J2EE服务器的内置JPA功能冲突。在完整的J2EE环境中，请考虑从JNDI获取`EntityManagerFactory`。或者，在开发者的`LocalContainerEntityManagerFactoryBean`定义中指定一个自定义`persistenceXmlLocation`，例如META-INF/my-persistence.xml，并且只在应用程序jar文件中包含有该名称的描述符。因为J2EE服务器仅查找默认的META-INF/persistence.xml文件，所以它会忽略这种自定义持久性单元，从而避免了与Spring驱动的JPA设置之间发生冲突。 （例如，这适用于Resin 3.1）

> 何时需要加载时间织入？
并非所有JPA提供商都需要JVM代理。Hibernate就是一个不需要JVM代理的例子。如果开发者的提供商不需要代理或开发者有其他替代方案，例如通过定制编译器或`Ant`任务在构建时应用增强功能，则不用使用加载时间编织器。

`LoadTimeWeaver`是一个Spring提供的接口，它允许以特定方式插入JPA`ClassTransformer`实例，这取决于环境是Web容器还是应用程序服务器。 通过代理挂载`ClassTransformers`通常性能较差。[代理](https://docs.oracle.com/javase/6/docs/api/java/lang/instrument/package-summary.html)会对整个虚拟机进行操作，并检查加载的每个类，这是生产服务器环境中最不需要的额外负载。

Spring为各种环境提供了一些`LoadTimeWeaver`实现，允许`ClassTransformer`实例仅适用于每个类加载器，而不是每个VM。

有关`LoadTimeWeaver`的实现及其设置的通用或定制的各种平台（如Tomcat，WebLogic，GlassFish，Resin和JBoss）的更多了解，请参阅AOP章节中的[Spring配置](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#aop-aj-ltw-spring)一节。

如前面部分所述，开发者可以使用`@EnableLoadTimeWeaving`注解或者`load-time-weaver`XML元素来配置上下文范围的`LoadTimeWeaver`。所有JPA`LocalContainerEntityManagerFactoryBeans`都会自动拾取这样的全局织入器。这是设置加载时间织入器的首选方式，为平台（WebLogic，GlassFish，Tomcat，Resin，JBoss或VM代理）提供自动检测功能，并将织入组件自动传播到所有可以感知织入者的Bean：

```
<context:load-time-weaver/>
<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	...
</bean>
```

开发者也可以通过`LocalContainerEntityManagerFactoryBean`的`loadTimeWeaver`属性来手动指定专用的织入器：

```
<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	<property name="loadTimeWeaver">
		<bean class="org.springframework.instrument.classloading.ReflectiveLoadTimeWeaver"/>
	</property>
</bean>
```
无论LTW如何配置，使用这种技术，依赖于仪器的JPA应用程序都可以在目标平台（例如：Tomcat）中运行，而不需要代理。这尤其重要的是当主机应用程序依赖于不同的JPA实现时，因为JPA转换器仅应用于类加载器级，彼此隔离。


#### 处理多个持久化单元

例如，对于依赖存储在类路径中的各种JARS中的多个持久性单元位置的应用程序，Spring将`PersistenceUnitManager`作为中央仓库来避免可能昂贵的持久性单元发现过程。默认实现允许指定多个位置，这些位置将通过持久性单元名称进行解析并稍后检索。（默认情况下，搜索classpath下的META-INF/persistence.xml文件。）

```
<bean id="pum" class="org.springframework.orm.jpa.persistenceunit.DefaultPersistenceUnitManager">
	<property name="persistenceXmlLocations">
		<list>
			<value>org/springframework/orm/jpa/domain/persistence-multi.xml</value>
			<value>classpath:/my/package/**/custom-persistence.xml</value>
			<value>classpath*:META-INF/persistence.xml</value>
		</list>
	</property>
	<property name="dataSources">
		<map>
			<entry key="localDataSource" value-ref="local-db"/>
			<entry key="remoteDataSource" value-ref="remote-db"/>
		</map>
	</property>
	<!-- if no datasource is specified, use this one -->
	<property name="defaultDataSource" ref="remoteDataSource"/>
</bean>

<bean id="emf" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
	<property name="persistenceUnitManager" ref="pum"/>
	<property name="persistenceUnitName" value="myCustomUnit"/>
</bean>
```

在默认实现传递给JPA provider之前，是允许通过属性（影响全部持久化单元）或者通过`PersistenceUnitPostProcessor`以编程(对选择的持久化单元进行)进行对`PersistenceUnitInfo`进行自定义的。如果没有指定`PersistenceUnitManager`，则由`LocalContainerEntityManagerFactoryBean`在内部创建和使用。

### 16.4.2 基于JPA的EntityManagerFactory和EntityManager来实现DAO

> 虽然`EntityManagerFactory`实例是线程安全的，但`EntityManager`实例不是。注入的JPA `EntityManager`的行为类似于从JPA Spec中定义的应用程序服务器的JNDI环境中提取的`EntityManager`。它将所有调用委托给当前事务的`EntityManager`(如果有);否则，它每个操作返回的都是新创建的`EntityManager`，通过使用不同的`EntityManager`来保证使用时的线程安全。

通过注入的方式使用`EntityManagerFactory`或`EntityManager`来编写JPA代码，是不需要依赖任何Spring定义的类的。如果启用了`PersistenceAnnotationBeanPostProcessor`，Spring可以在实例级别和方法级别识别`@PersistenceUnit`和`@PersistenceContext`注解。使用`@PersistenceUnit`注解的纯JPA DAO实现可能如下所示：

```
public class ProductDaoImpl implements ProductDao {

	private EntityManagerFactory emf;

	@PersistenceUnit
	public void setEntityManagerFactory(EntityManagerFactory emf) {
		this.emf = emf;
	}

	public Collection loadProductsByCategory(String category) {
		EntityManager em = this.emf.createEntityManager();
		try {
			Query query = em.createQuery("from Product as p where p.category = ?1");
			query.setParameter(1, category);
			return query.getResultList();
		}
		finally {
			if (em != null) {
				em.close();
			}
		}
	}
}
```

上面的DAO对Spring的实现是没有任何依赖的，而且很适合与Spring的应用程序上下文进行集成。而且，DAO还可以通过注解来注入默认的`EntityManagerFactory`：

```
<beans>

	<!-- bean post-processor for JPA annotations -->
	<bean class="org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor"/>

	<bean id="myProductDao" class="product.ProductDaoImpl"/>
</beans>
```

如果不想明确定义`PersistenceAnnotationBeanPostProcessor`，可以考虑在应用程序上下文配置中使用Spring上下文`annotation-config`XML元素。这样做会自动注册所有Spring标准后置处理器，用于初始化基于注解的配置，包括`CommonAnnotationBeanPostProcessor`等。

```
<beans>
	<!-- post-processors for all standard config annotations -->
	<context:annotation-config/>

	<bean id="myProductDao" class="product.ProductDaoImpl"/>
</beans>
```

这样的DAO的主要问题是它总是通过工厂创建一个新的`EntityManager`。开发者可以通过请求事务性`EntityManager`（也称为*共享EntityManager*，因为它是实际的事务性EntityManager的一个共享的，线程安全的代理）来避免这种情况。

```
public class ProductDaoImpl implements ProductDao {

	@PersistenceContext
	private EntityManager em;

	public Collection loadProductsByCategory(String category) {
		Query query = em.createQuery("from Product as p where p.category = :category");
		query.setParameter("category", category);
		return query.getResultList();
	}
}
```

`@PersistenceContext`注解具有可选的属性类型，默认值为`PersistenceContextType.TRANSACTION`。此默认值是开发者所需要接收共享的`EntityManager`代理。替代方案`PersistenceContextType.EXTENDED`则完全不同：该方案会返回一个所谓扩展的`EntityManager`，该`EntityManager`不是*线程安全*的，因此不能在并发访问的组件（如Spring管理的单例Bean）中使用。扩展实体管理器仅应用于状态组件中，比如持有会话的组件，其中`EntityManager`的生命周期与当前事务无关，而是完全取决于应用程序。

> 方法和实例变量级别注入
指示依赖注入（例如`@PersistenceUnit`和`@PersistenceContext`）的注解可以应用于类中的实例变量或方法，也就是表达式方法级注入和实例变量级注入。实例变量级注释简洁易用，而方法级别允许进一步处理注入的依赖关系。在这两种情况下，成员的可见性（`public`，`protected`，`private`）并不重要。
类级注解怎么办？
在J2EE平台上，它们用于依赖关系声明，而不是资源注入。

注入的`EntityManager`是由Spring管理的（Spring可以意识到正在进行的事务）。重要的是要注意，因为通过注解进行注入，即使新的DAO实现使用通过方法注入的`EntityManager`而不是`EntityManagerFactory`的注入的，在应用程序上下文XML中不需要进行任何修改。

这种DAO风格的主要优点是它只依赖于Java Persistence API;不需要导入任何Spring的实现类。而且，Spring容器可以识别JPA注解来实现自动的注入和管理。从非侵入的角度来看，这种风格对JPA开发者来说可能更为自然。

### 16.4.3 Spring驱动的JPA事务

> 如果开发者还没有阅读[声明式事务管理](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#transaction-declarative)，强烈建议开发者先行阅读，这样可以更详细地了解Spring的对声明式事务支持。

JPA的推荐策略是通过JPA的本地事务支持的本地事务。 Spring的`JpaTransactionManager`提供了许多来自本地JDBC事务的功能，例如针对任何常规JDBC连接池（不需要XA要求）指定事务的隔离级别和资源级只读优化等。

Spring JPA还允许配置`JpaTransactionManager`将JPA事务暴露给访问同一`DataSource`的JDBC访问代码，前提是注册的`JpaDialect`支持检索底层JDBC连接。Spring为EclipseLink和Hibernate JPA实现提供了实现。有关`JpaDialect`机制的详细信息，请参阅下一节。

### 16.4.4 JpaDialect和JpaVendorAdapter

作为高级功能，`JpaTransactionManager`和`AbstractEntityManagerFactoryBean`的子类支持自定义`JpaDialect`，将其作为Bean传递给`jpaDialect`属性。`JpaDialect`实现可以以供应商特定的方式使能Spring支持的一些高级功能：

* 应用特定的事务语义，如自定义隔离级别或事务超时
* 为基于JDBC的DAO导出事务性JDBC连接
* 从`PersistenceExceptions`到Spring`DataAccessExceptions`的异常转义

这对于特殊的事务语义和异常的高级翻译特别有价值。但是Spring使用的默认实现（`DefaultJpaDialect`）是不提供任何特殊功能的。如果需要上述功能，则必须指定适当的方言才可以。

> 作为一个更广泛的供应商适应设施，主要用于Spring的全功能`LocalContainerEntityManagerFactoryBean`设置，`JpaVendorAdapter`将`JpaDialect`的功能与其他提供者特定的默认设置相结合。指定`HibernateJpaVendorAdapter`或`EclipseLinkJpaVendorAdapter`是分别为Hibernate或EclipseLink自动配置`EntityManagerFactory`设置的最简单方便的方法。但是请注意，这些提供程序适配器主要是为了与Spring驱动的事务管理一起使用而设计的，即为了与`JpaTransactionManager`配合使用的。

有关其操作的更多详细信息以及在Spring的JPA支持中如何使用，请参阅`JpaDialect`和`JpaVendorAdapter`的Javadoc。

### 16.4.5 为JPA配置JTA事务管理

作为`JpaTransactionManager`的替代方案，Spring还允许通过JTA在J2EE环境中或与独立的事务协调器（如Atomikos）进行多资源事务协调。除了用Spring的`JtaTransactionManager`替换`JpaTransactionManager`，还有需要以下一些操作：

* 底层JDBC连接池是需要具备XA功能，并与开发者的事务协调器集成的。这在J2EE环境中很简单，只需通过JNDI导出不同类型的`DataSource`即可。有关导出`DataSource`等详细信息，可以参考应用服务器文档。类似地，独立的事务协调器通常带有特殊的XA集成的`DataSource`实现。
* 需要为JTA配置JPA`EntityManagerFactory`。这是特定于提供程序的，通常通过在`LocalContainerEntityManagerFactoryBean`的特殊属性指定为"jpaProperties"。在使用Hibernate的情况下，这些属性甚至是需要基于特定的版本的;请查阅Hibernate文档以获取详细信息。
* Spring的`HibernateJpaVendorAdapter`会强制执行某些面向Spring的默认设置，例如在Hibernate 5.0中匹配Hibernate自己的默认值的连接释放模式“on-close”，但在5.1 / 5.2中不再存在。对于JTA设置，不要声明`HibernateJpaVendorAdapter`开始，或关闭其`prepareConnection`标志。或者，将Hibernate 5.2的`hibernate.connection.handling_mode`属性设置为`DELAYED_ACQUISITION_AND_RELEASE_AFTER_STATEMENT`以恢复Hibernate自己的默认值。有关WebLogic的相关说明，请参考[Hibernate的虚假应用服务器警告]()一节。
* 或者，可以考虑从应用程序服务器本身获取`EntityManagerFactory`，即通过JNDI查找而不是本地声明的`LocalContainerEntityManagerFactoryBean`。服务器提供的`EntityManagerFactory`可能需要在服务器配置中进行特殊定义，减少了部署的移植性，但是`EntityManagerFactory`将为开箱即用的服务器JTA环境设置。