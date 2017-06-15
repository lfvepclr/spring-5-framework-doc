# **14. DAO支持**

## **14.1 介绍**

在Spring中数据访问对象\(DAO\)旨在使JDBC,Hibernate，JPA或JDO等数据访问技术有一致的处理方法，并且方法尽可能简单。  
这样就可以很容易地切换上述持久化技术，并且切换过程无需担心每种技术的特有异常。

## **14.2 一致的异常层级**

Spring为技术特定异常提供了一个适当的转化，例如：SQLException 所属异常类层级用DataAccessException 作为根异常，这些异常包括了原来的异常，所以不会有丢失可能出错信息的风险。

除了JDBC异常，Spring也包含了Hibernate特定异常，将它们转换为一组集中的运行时异常\(对JDO 和 JPA 异常也是如此\)，在合适的层次上处理多数不可恢复的持久化异常，而不会在dao上产生繁琐的catch-throw块和异常声明（仍然可以在认为合适的地方捕获和处理异常）。向上面提到的一样，JDBC异常（包括数据方言）也都转化为相同的层级结构，意味着在一个统一的项目模型中你也可以执行一些JDBC操作。

以上列举的Spring的各种模板类支持各种ORM框架。如果使用基于拦截器的类，那么我们的程序必须关心并处理HibernateExceptions和JDOExceptions本身，最好是通过分别授权给SessionFactoryUtils’ \`convertHibernateAccessException\(..\)或 convertJdoAccessException\(\)方法。这些方法将这些异常转化为与org.springframework.dao中异常层级兼容的异常。由于JDOExceptions 没有被检查，它可以被简单的抛出，这也牺牲了DAO在异常上的抽象。

下图展示了Spring提供的异常层级，（请注意：在这张图上显示出来的类层级仅仅是整个DataAccessException 的一个子集）

## ![](/assets/DataAccessException.gif)**14.3 配置 DAO 或 Repository 类的注解**

使用@Repository注解是数据访问对象（DAOs）或库能提供异常转换的最好方式，这个注解还允许组件扫描，查找并配置你的 DAOs 和库，并且不需要为它们提供 XML 配置文件。

```
@Repository
public class SomeMovieFinder implements MovieFinder {
// ...
}
```

任何DAO或库实现都需要访问持久的源，依赖于持久化技术的使用；例如：一个基于JDBC的库需要访问一个JDBC DataSource，一个基于JPA的库需要访问一个 EntityManager，最简单的方式就是使用 @Autowired, @Inject, @Resource 或@PersistenceContext 这些注解中的一个完成资源的依赖注入，这是一个JPA库的例子：

```
@Repository
public class JpaMovieFinder implements MovieFinder {

@PersistenceContext
private EntityManager entityManager;

// ...

}
```

如果你使用传统的Hibernate API，你可以注入SessionFactory：

```
@Repository
public class HibernateMovieFinder implements MovieFinder {

private SessionFactory sessionFactory;

@Autowired
public void setSessionFactory(SessionFactory sessionFactory) {
this.sessionFactory = sessionFactory;
}

// ...

}
```

最后一个例子我们将在这里展示典型的JDBC支持，你将会在初始化方法中注入 DataSource ，在初始化方法中，你将使用这个DataSource创建一个JdbcTemplate 和其他与SimpleJdbcCall相似的数据访问支持类。

```
@Repository
public class JdbcMovieFinder implements MovieFinder {

private JdbcTemplate jdbcTemplate;

@Autowired
public void init(DataSource dataSource) {
this.jdbcTemplate = new JdbcTemplate(dataSource);
}

// ...

}
```



