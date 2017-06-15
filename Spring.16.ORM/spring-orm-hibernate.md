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