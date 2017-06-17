## 32.1 介绍
Spring 框架从 3.1 开始，对 Spring 应用程序提供了透明式添加缓存的支持。和事务支持一样，抽象缓存允许一致地使用各种缓存解决方案，并对代码的影响最小。

从 4.1 版本开始，缓存抽象支持了 JSR-107 注释和更多自定义选项，从而得到了显著的改进。

## 32.2 缓存抽象

```
缓存（Cache） vs 缓冲区（Buffer）

缓存和缓冲区两个术语往往可以互换着使用。但注意，它们代表着不同的东西。
缓冲区是作用于快和慢速实体之间的中间临时存储。
一块缓冲区必须等待其他并影响性能，通过允许一次性移动整个数据块而不是小块来缓解。数据从缓冲区读写只有一次。因此缓冲区对至少一方是可见的。
另一方面，缓存根据定义是隐性的，双方不会知道缓存的发生。它提高了性能，但允许以快速的方式多次读取相同的数据。

想了解更多这二者之间的差异，见：https://en.wikipedia.org/wiki/Cache_(computing)#The_difference_between_buffer_and_cache

```

核心上，抽象将缓存作用于 Java 方法上，基于缓存中的可用信息，可以减少方法的执行次数。也就是说，每次目标方法的调用时，抽象使用缓存行为来检查执行方法，检查执行方法是否给定了缓存的执行参数：如果有，则返回缓存结果，不执行具体方法；如果没有，则执行方法，并将结果缓存后，返回给用户。以便于下次调用方法时，直接返回缓存的结果。这样，只要给定缓存执行参数，在复杂的方法（无论是 CPU 或者 IO 相关）只需要执行一次，就可以得到结果，并利用缓存可重复使用结果，而不必再次执行该方法。另外，缓存逻辑可以被透明地调用，不会对调用者造成任何的困扰。

> 显然，这种方法只适用于为某个给定输入（或参数）返回相同输出（结果），无论执行多少次。

抽象提供的其他缓存相关操作，比如更新缓存内容或者删除其中一条缓存。如果在应用程序过程中，发生了变化的数据需要缓存，那这些功能会很有用。

比如 Spring 框架其他服务一样，缓存服务是一种抽象（不是缓存的实现），并且需要使用实际的存储器来存储缓存数据。也就是说，抽象能够使开发人员不必编写缓存逻辑，但它没有提供缓存的存储器。这个抽象是由 `org.springframework.cache.Cache` 和 `org.springframework.cache.CacheManager` 接口实现的。

有些抽象的实现是开箱即用的：基于 JDK `java.util.concurrent.ConcurrentMap` 缓存实现，Ehcache 2.x，Gemfire cache, Caffeine 和 JSR-107 缓存（例如 Ehcache 3.x）。有关缓存存储/提供的更多信息，请参见 32.7 节 《Plugging-in different back-end caches》。

> 缓存抽象没有特别处理多线程和多进程环境，因为这些功能由缓存实现来处理...


如果是多进程环境（即部署在多个节点上的应用程序），则需要相应的配置程序提供缓存。根据使用情况，几个节点上相同数据的副本可能足够多了，但如果在应用程序过程中更改了数据，则需要启动其他传播机制，进行同步缓存数据。

缓存一个特定的对象是典型的缓存交互 get-if-not-found-then-proceed-and-put-finally 代码块：不需应用任何锁，并且几个线程同时尝试加载相同的对象。同样适用于回收，如果多个线程同时更新或者回收数据，则可能会使用过时的数据。某些缓存提供者在该领域提供高级功能，请参考您正在使用的缓存提供者的更多高级功能的详细信息。

要使用缓存抽象，开发人员需要注意两个方面：
- 缓存声明 - 标志缓存的方法及缓存策略
- 缓存配置 - 数据读写的缓存数据库

# 32.3 基于声明式注解的缓存

对于缓存声明，抽象提供了一组 Java 注解：

- `@Cacheable` 触发缓存机制
- `@CacheEvict` 触发缓存回收
- `@CachePut` 更新缓存，而不会影响方法的执行
- `@Caching` 组合多个缓存操作到一个方法
- `@CacheConfig` 类级别共享系诶常见的缓存相关配置

下面，让我们仔细看看每个注释。

## 32.3.1 @Cacheable 注解
顾名思义，`@Cacheable` 用于标识可缓存的方法 - 即需要将结果存储到缓存中的方法，以便于再一次调用（具有相同的输入参数）时返回缓存结果，而无需执行该方法。在最简单的形式中，注解声明需要定义与注解方法相关联的缓存名称：

```
@Cacheable("books")
public Book findBook(ISBN isbn) {...}
```

在上面的代码中，`findBook` 与名为 `books` 的缓存相关。每次调用该方法时，都会检查缓存以查看调用是否已经被执行，并且不必重复。而在大多数情况下，只有一个缓存被声明，注解允许指定多个名称，以便使用多个缓存。在这种情况下，执行该方法之前将检查每个高速缓存 - 如果至少有一个缓存被命中，则返回缓存相关的值：

> 注意：即使没有实际执行缓存方法，所有其他不包含该值的缓存也将被更新。

```
@Cacheable({"books", "isbns"})
public Book findBook(ISBN isbn) {...}
```

### 默认键生成

缓存的本质是键值对存储，所以每次调用缓存方法都会转换作用于缓存访问合适的键。开箱即用，缓存抽象使用基于以下算法的简单 `KeyGenerator`:

- 如果没有参数，返回 `SimpleKey.EMPTY`
- 如果只有一个参数，返回该实例
- 如果大于一个参数，返回一个包含所有参数的 `SimpleKey`

这种算法对大多数用例很适用，只要参数具有自然键并实现了有效的 `hashCode()` 和 `equals()` 方法。如果不是这样，策略就需要改变。
要提供不同的默认密钥生成器，需要实现`org.springframework.cache.interceptor.KeyGenerator`接口。


> Spring 4.0 的发布，默认键生成策略发生了变化。Spring 早期版本使用的键生成策略对于多个关键参数，只考虑了参数的 `hashCode()` ，而没有考虑 `equals()` 。这可能会导键碰撞（参见 SPR-10237 资料）。新的 `SimpleKeyGenerator` 对这种场景使用了复合键。
如果要继续使用以前的键策略，可以配置不推荐使用的 `org.springframework.cache.interceptor.DefaultKeyGenerator` 类或者创建基于哈希的自定义 `KeyGenerator` 的实现

### 自定义键生成声明

缓存是通用的，因此目标方法可能有不能简单映射到缓存结构之上的签名。当目标方法具有多个参数时，这一点往往变得很明显，其中只有一些参数适合于缓存（其他的仅由方法逻辑使用）。例如：

```
@Cacheable("books")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

第一眼看代码，虽然两个 `boolean` 参数影响了该 `findBook` 方法，但对缓存没有任何用处。更进一步，如果两者之中只有一个是重要的，另一个不重要呢？

这种情况下，`@Cacheable` 注解只允许用户通过键属性指定 `key` 的生成方式。开发人员可以使用 SpEL 来选择需要的参数（或其嵌套属性），执行参数设置调用任何方法，无需编写任何代码或者实现人任何接口。这是默认键生成器推荐的方法，因为方法在代码库的增长下，会有完全不同的方法实现。而默认策略可能适用于某些方法，并不是适用于所有的方法。

这里是 SpEL 声明的一些示例 - 如果你不熟悉它，查阅[Chapter 6, Spring Expression Language (SpEL)](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/expressions.html)：

```
@Cacheable(cacheNames="books", key="#isbn")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)

@Cacheable(cacheNames="books", key="#isbn.rawNumber")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)

@Cacheable(cacheNames="books", key="T(someType).hash(#isbn)")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

上面代码片段显示了选择某个参数，或参数的某个属性值或任意（静态）方法，如此方便操作。

如果生成键的算法太具体或者需要共享，可以操作中定义一个自定定义的 `keyGenerator`。为此，请指定要使用的 `KeyGenerator` Bean 实现的名称：

```
@Cacheable(cacheNames="books", keyGenerator="myKeyGenerator")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

> key 和 keyGenerator 的参数是互斥的，指定两者的同样的操作将导致异常。

### 默认缓存解析
开箱即用，缓存抽象使用一个简单的 `CacheResolver`,在 `CacheManager` 可以配置操作级别来检索缓存。

需要不同的默认缓存解析器，需要实现接口`org.springframework.cache.interceptor.CacheResolver` 。

### 自定义缓存解析
默认缓存解析适用于使用单个 `CacheManager` 并且应用在不需要复杂缓存解析的应用程序。

对于使用多个缓存管理器的应用，可以为每个操作设置一个 `cacheManager`:

```
@Cacheable(cacheNames="books", cacheManager="anotherCacheManager")
public Book findBook(ISBN isbn) {...}
```

也可以完全以与键生成类似的方式来替换 `CacheResolver`。每个缓存操作都要求缓存解析，基于运行时参数的缓存解析:

```
@Cacheable(cacheResolver="runtimeCacheResolver")
public Book findBook(ISBN isbn) {...}
```

> 自 Spring 4.1 以后，缓存注解的属性值是不必要的，因为 CacheResolver 可以提供该特定的信息，无论注解的内容是什么。与 key 和 keyGenerator 类似，cacheManager 和 cacheResolver 参数是互斥的，并且指定两者同样的操作会导致异常，因为 CacheResolver 的实现将忽略自定义的 CacheManager。这是你不希望的。


### 同步缓存
在多线程环境中，某些操作可能会导致同时引用相同的参数（通常在启动时）。默认情况下，缓存抽象不会锁定任何对象，同样的对象可能会被计算好几次，从而达不到缓存的目的。

对于这些特熟情况，`sync` 属性可用于指示底层缓存提供程序在计算该值时锁定缓存对象。因此，只有一个线程将忙于计算值，而其他线程会被阻塞，直到该缓存对象被更新为止。

```
@Cacheable(cacheNames="foos", sync="true")
public Foo executeExpensiveOperation(String id) {...}
```

> 这是可选功能，可能你是用的缓存库不支持它。由核心框架提供的所有 CacheManager 都会实现并支持它。翻阅缓存提供商文档可以了解更多详细信息。

### 条件缓存
有时，一种方法可能不适合缓存（例如，它可能取决于给定的参数）。缓存注解通过条件参数支持这样的功能，采用使用 `SpEL` 表达式的 `condition` 参数表示。如果是 true，则缓存方法，如果是 false，则不缓存，无论缓存中有什么值或者使用了哪些参数都严格按照规则执行该方法。一个快速入门的例子 - 只有当参数名称长度小于 32 的时候，才会缓存下面方法：

```
@Cacheable(cacheNames="book", condition="#name.length < 32")
public Book findBook(String name)
```

另外，`unless` 参数用于是否向缓存添加值。不同于 `condition` 参数，`unless` 参数在方法被调用后评估表达式。扩展上一个例子 - 我们只想缓存平装书：

```
@Cacheable(cacheNames="book", condition="#name.length < 32", unless="#result.hardback")
public Book findBook(String name)
```

缓存抽象支持 `java.util.Optional`,只有当它作为缓存值的时候才可使用。`#result` 指向的是业务实体，不是在包装类上。上面的例子可以重写如下：

```
@Cacheable(cacheNames="book", condition="#name.length < 32", unless="#result.hardback")
public Optional<Book> findBook(String name)
```
注意：结果仍然是 `Book` 不是 `Optional`

### 缓存 SpEL 上下文
每个 SpEL 表达式都会有求值上下文。除了构建参数，框架提供了专门的缓存相关元数据，比如参数名称。下表列出了可用于上下文的项目，因此可以使用他们进行键和条件的计算：

表 32.1 缓存 SpEL 元数据

|参数名|用处|描述|例子|
|:----:|:----:|:----:|:----:|
| methodName |root object|被调用的方法名|`#root.methodName`|
| method |root object|被调用的方法|`#root.method.name`|
| target |root object|被调用的对象|`#root.target`|
| targetClass |root object|被调用的类|`#root.targetClass`|
| args |root object|被调用的的类目标参数|`#root.args[0]`|
|caches|root object|当前方法执行的缓存列表|`#root.caches[0].name`|
|argument name|	evaluation context|任何方法参数的名称|`#iban` 或 `#a0` (也可以使用 `#p0` 或者 `#p<#arg>`)|
|result|	evaluation context|方法调用的结果（缓存的值）|`#result`|

### 32.3.2 @CachePut 注解

需要缓存更新但不影响方法执行的情况，可以使用 `@CachePut` 注解。也就是说，该方法始终执行，并将其结果放入缓存中（根据 `@CachePut` 选项）。它支持与 `@Cacheable` 相同的选项，适用于缓存而不是方法流程优化：

```
@CachePut(cacheNames="book", key="#isbn")
public Book updateBook(ISBN isbn, BookDescriptor descriptor)
```

> 对同一个方法同时使用 `@CachePut` 和 `@Cacheable` 注解通常是不推荐的。因为它们有不同的行为。后者通过使用缓存并跳过方法执行，前者强制执行方法并进行缓存更新。这会导致意想不到的行为，除了具体的场景（例如需要排除条件的注解），应该避免这样的声明方式。还要注意，这样的条件不应该依赖于结果对象（#result 对象），因为结果对象应该在前面被验证后排除。

### 32.3.3 @CacheEvict 注解
缓存抽象不仅仅缓存更多数据，还可以回收缓存。这个过程对于从缓存中删除旧数据或者未使用的数据非常有用。与 `@Cacheable` 相反, 注解 `@CacheEvict` 划分了回收缓存的方法，即作为从缓存中删除数据的触发器方法。`@CacheEvict` 需要指定一个（或者多个）执行动作，并且允许自定义缓存和键解析或条件被指定。但有一个额外的参数 `allEntries`,可以指示是否所有对象都要被收回，还是一个对象（取决于key）。

```
@CacheEvict(cacheNames="books", allEntries=true)
public void loadBooks(InputStream batch)
```

当整个缓存区需要被收回时，这个选项就会派上用场 - 而不是回收每个对象（当不起作用时会需要很长的响应时间），所以对象会在一个操作中被删除，如上所示。注意，框架将忽略此场景下指定的任何key，因为它不适用（整个缓存收回，不仅仅是一个条目被收回）。

还可以指出发生缓存回收是在默认之后还是在通过 `beforeInvocation` 属性执行方法后。前者提供与其他注解相同的语义 - 一旦方法成功完成，就执行缓存上的动作（这种情况是回收）。如果方法不执行（因为它可能会被缓存）或者抛出异常，则不会发生回收。后者（`beforeInvocation = ture`）会导致在调用该方法之前发生回收 - 在驱逐不需要与方法结果相关的情况下，这是有用的。

重要的是，`void` 方法可以和 `@CacheEvict` 一起使用 - 由于方法作为触发器，返回值会被忽略（因为他们不与缓存交互） - `@Cacheable` 就不是这样的，它添加 / 更新数据到缓存时，需要一个结果。

### 32.3.4 @Caching 注解

另外情况下，需要指定想同类型的多个注解，例如 `@CacheEvict` 或 `@CachePut` 需要被指定。比如因为不同缓存之间的条件或者键表达式不同。@Caching 允许在同一个方法上使用多个嵌套的 `@Cacheable` 、`@CachePut` 和 `@CacheEvict` ：

```
@Caching(evict = { @CacheEvict("primary"), @CacheEvict(cacheNames="secondary", key="#p0") })
public Book importBooks(String deposit, Date date)
``` 

### 32.3.5 @CacheConfig 注解

到目前为止，我们已经看到缓存操作提供了许多定制选项，这些选项可以在操作的基础上进行设置。 但是，一些自定义选项可能很麻烦，可以配置是否适用于该类的所有操作。 例如，指定用于该类的每个缓存操作的高速缓存的名称可以被单个类级别定义所替代。 这是`@CacheConfig`发挥作用的地方。

```
@CacheConfig("books")
public class BookRepositoryImpl implements BookRepository {

	@Cacheable
	public Book findBook(ISBN isbn) {...}
}
```

`@CacheConfig` 是一个类级别的注解，可以共享缓存名称。自定义 `KeyGenerator`，自定义 `CacheManager` 和自定义 `CacheResolver`。该注解在类上不会操作任何缓存。

操作级别上的自定义会覆盖 `@CacheConfig` 的自定义。因此，每个缓存操作都会有三个级别的定义：

- 全局配置，可用于 `CacheManager` 和 `KeyGenerator`
- 类级别，用 `@CacheConfig`
- 操作级别层面

### @EnableCaching 注解

重点注意的是，即使声明缓存注解也不会主动触发动作 - Spring 中很多东西都这样，该特性必须声明式的启用（意味着如果缓存出现问题，则通过删除某一个配置就可以验证，而不是所有代码的注释）。

启动缓存注解，将 `@EnableCaching` 注解加入到 `@Configuration` 类中：

```
@Configuration
@EnableCaching
public class AppConfig {
}
```

对于 XML 配置，使用 `cache:annotation-driven` 元素：

```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:cache="http://www.springframework.org/schema/cache"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd">

		<cache:annotation-driven />

</beans>
```

`cache:annotation-driven` 元素和 `@EnableCaching` 注解允许指定多种选项，这些选项通过 AOP 将缓存行为添加到应用程序的方式。该配置与 `@Transactional` 注解类似：


> Java 高级自定义配置需要实现 `CachingConfigurer`。更多详细信息，参考 javadoc。


表 32.2. 缓存注解设置

| XML 属性|注解属性|默认|描述|
|:----:|:----:|:----:|:----:|
| cache-manager  | N/A（参阅 CachingConfigurer javadocs）| cacheManager | 要使用的缓存管理器的名称。 使用该缓存管理器（或“cacheManager”未设置）将在幕后初始化默认`CacheResolver`。 要更精细地管理缓存解析度，请考虑设置“缓存解析器”属性。|
| cache-resolver | N/A（参阅 CachingConfigurer javadocs） | 配置 `cacheManager ` 的 `SimpleCacheResolver ` | 要用于解析后备缓存的CacheResolver的bean名称。 此属性不是必需的，只需要指定为“cache-manager”属性的替代方法。|
| key-generator | N/A（参阅 CachingConfigurer javadocs） | `SimpleKeyGenerator ` | 要使用的自定义键生成器的名称。|
| error-handler | N/A（参阅 CachingConfigurer javadocs） | `SimpleCacheErrorHandler ` |要使用的自定义缓存错误处理程序的名称。 默认情况下，在缓存相关操作期间抛出任何异常抛出在客户端。 |
| mode | mode | proxy | 默认模式“代理”使用Spring的AOP框架处理带注释的bean（以下代理语义，如上所述，适用于仅通过代理进行的方法调用）。 替代模式“aspectj”代替使用Spring的AspectJ缓存方面编写受影响的类，修改目标类字节码以应用于任何类型的方法调用。 AspectJ编织需要类路径中的spring-aspects.jar以及启用加载时编织（或编译时编织）。 （有关如何设置加载时编织的详细信息，请参阅 [the section called “Spring configuration”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-aj-ltw-spring)）|
| proxy-target-class | proxyTargetClass | false |仅适用于代理模式。 控制为使用`@Cacheable`或`@CacheEvict`注释注释的类创建的缓存代理类型。 如果`proxy-target-class`属性设置为`true`，则会创建基于类的代理。 如果`proxy-target-class`为`false`，或者如果属性被省略，则会创建基于标准的基于JDK接口的代理。 （有关详细检查不同代理类型，请参见第[Section 7.6, “Proxying mechanisms”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-proxying)） |
| order | order | Ordered.LOWEST_PRECEDENCE |定义应用于使用`@Cacheable`或`@CacheEvict`注释的bean的缓存通知的顺序。 （有关与AOP通知的排序有关的规则的更多信息，请参阅[ the section called “Advice ordering”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/aop.html#aop-ataspectj-advice-ordering)。）没有指定的排序意味着AOP子系统确定建议的顺序。 |


> `<cache：annotation-driven />` 只会匹配相对应的应用上下文中定义的 `@Cacheable` / `@CachePut` / `@CacheEvict` / `@Cached`。如果将 `<cache:annotation-driven/>` 配置在 `DispatcherServlet` 的 `WebApplicationContext` ，它只检查控制器中的bean，而不是您的服务。参考 [18.2 章节 `DispatcherServlet`](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-servlet) 获取更多信息。  

    方法可见性和缓存注解
    使用代理后，缓存注解应用于公共可见的方法。如果对 protected、private 或 package-visible 方法进行缓存注解，不会引起错误。但注解的方法不会显示缓存配置。如果更改字节码时需要注解非公共方法，请考虑使用 AspectJ（见下文）。



> Spring 建议只使用 `@Cache*` 注解具体的类（或具体的方法），而不是注解接口。可以在接口（或接口方法）上注解 `@Cache*` ，但只有当使用基于接口的代理时，才能使用它。Java 注解不是用接口继承，如果使用基于类的代理（`proxy-target-class="true"`）或基于代理的 `weaving-based(mode="aspectj")`，缓存设置无法识别，对象也不会被包装在缓存代理中。

> 在代理模式（默认情况）中，只有通过代理进入的外部方法调用被截取。实际上，自调用目标对象中调用另一个目标对象方法的方法在运行时不会导致实际的缓存，即使被调用的方法被 `@Cacheable` 标志 - 可以考虑 aspectj 模式。此外，代理必须被完全初始化提供预期行为，因此不应该依赖于初始化的代码功能，即 `@PostConstruct`

### 32.3.7 使用自定义注解
```
自定义注解和 AspectJ
开箱即用，此功能仅使用基于代理的方法，但可使用 AspectJ 进行一些额外的工作。

spring-aspects 模块仅为标准注解定义了一个切面。如果你定义了自己的注解，那还需要为这些注解定义一个方面。检查 AnnotationCacheAspect 为例。
```
缓存抽象允许使用不同的注解识别什么方法触发缓存或者缓存回收。这是非常方便的模板机制，因为不需要重复缓存声明（特别是在指定键和条件），或者在代码库不允许使用外部的导入（`org.springframework`）。其他的注解 @Cacheable, @CachePut, @CacheEvict 和 @CacheConfig 可作为元注解，也可以被其他注解使用。换句话说，可以自定义一个通用的注解替换 `@Cacheable` 声明：

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Cacheable(cacheNames="books", key="#isbn")
public @interface SlowService {
}
```
以上我们定义的注解 `@SlowService` ，并使用 `@Cacheable` 注解 - 现在我们替换下面的代码：

```
@Cacheable(cacheNames="books", key="#isbn")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```
改为：

```
@SlowService
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```
即使  `@SlowService`不是Spring注解，容器也可以在运行时自动选择其声明并了解其含义。 请注意，如上所述，需要启用注解驱动的行为。

## 32.4 JCache (JSR-107)  注解
Spring Framework 4.1 以来，缓存抽象完全支持 JCache 标准：即 `@CacheResult`，`@CachePut`，`@CacheRemove` 和 `@CacheRemoveAll` 还有 `@CacheDefaults`，`@CacheKey` 和 `@CacheValue` 。这些注解被大家正确的使用，体现了缓存在 `JSR-107` 的实现：缓存抽象的内部实现，并提供了符合规范的默认 `CacheResolver` 和 `KeyGenerator` 的实现。换句话说，如果你已经使用了 Spring 缓存抽象，那可以平滑切换到这些标准注解，无需更改缓存存储（或者配置）。


### 32.4.1 特征总结

对于熟悉 Spring 缓存注解，下面描述了 Spring 注解和 JSR-107 对应的主要区别：

表 32.3. Spring vs JSR-107 缓存注解

| Spring |JSR-107|备注|
|:----:|:----:|:----:|
|`@Cacheable`|`@CacheResult`|相似，`@CacheResult` 可缓存特定的异常，并强制执行该方法，不管缓存的结果。 |
|`@CachePut`|`@CachePut`|Spring 调用方法之后获取结果去更新缓存，但 JCache 将其作为使用 `@CacheValue` 的参数传递。所以 JCache 允许实际方法调用之前或者之后更新缓存。 |
|`@CacheEvict`|`@CacheRemove`|相似， `@CacheRemove` 支持条件回收，以防止方法调用导致异常|
|`@CacheEvict(allEntries=true)`| `@CacheRemoveAll` |查阅 `@CacheRemove`|
|`@CacheConfig`|`@CacheDefaults`|允许类似的方式配置相同的属性|

JCache 的 `javax.cache.annotation.CacheResolver` 概念和 Spring 的  `CacheResolver` 是一样的，除了 JCache 只支持单个缓存。默认下，根据注解声明的名称检索要使用的缓存。如果没有指定缓存的名称，则会自动生成默认值。更多信息可以查看 javadoc 的 `@CacheResult#cacheName()`。

`CacheResolverFactory` 检索出 `CacheResolver` 实例。每个缓存操作都可以自定义工厂： 

```
@CacheResult(cacheNames="books", cacheResolverFactory=MyCacheResolverFactory.class)
public Book findBook(ISBN isbn)
```
```
注意
对于所有引用的类，Spring 会找到具体指定类型的 Bean。如果存在多个匹配，会创建一个新实例，并可以正常的 Bean 生命周期回调（如依赖注入）
```

键由 javax.cache.annotation.CacheKeyGenerator 生成，和 Spring 的 KeyGenerator 能达到相同的目标。默认情况下，所有方法参数都会被考虑，除非至少有一个参数被注解为 @CacheKey。这和 Spring 的自定义键生成类似。例如，下面代码操作是相同的，一个使用 Spring 抽象，另一个使用 JCache：

```
@Cacheable(cacheNames="books", key="#isbn")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)

@CacheResult(cacheName="books")
public Book findBook(@CacheKey ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

CacheKeyResolver 可以在操作中指定，类似方式是 CacheResolverFactory。

JCache 管理注解方法抛出的异常：可以防止缓存更新，但将异常作为故障的标志，而不是再次调用该方法。假设抛出 InvalidIsbnNotFoundException 异常，那么 ISBN 的结构是无效的。如果这是一个永久的失败，没有任何书会被查询出来。以下的缓存异常，以使具有无效的 ISBN 进一步直接抛出缓存的异常，而不是再次调用该方法。

```
@CacheResult(cacheName="books", exceptionCacheName="failures"
             cachedExceptions = InvalidIsbnNotFoundException.class)
public Book findBook(ISBN isbn)
```

### 32.4.2 启用 JSR-107 支持

不需要特殊处理，去支持 JSR-107 和 Spring 的声明式注解。JSR-17 API 和 `spring-context-support` 模块在类路径下，`@EnableCaching` 和 `cache:annotation-driven` 两者会被启用。

> 根据你的具体案例选择需要的。你还可以使用 JSR-107 API 和其他使用 Spring 自己的注解来匹配服务。注意，如果这些服务影响相同的缓存，则使用一致的键生成实现。

## 32.5 缓存声明式 XML 配置

如果不想使用注解，可以使用 XML 进行声明式配置缓存。所以不用注解方法的形式，而从外部指定目标方法和缓存指令（类似于声明式事务管理）。以前的例子可以转化为：

```
<!-- the service we want to make cacheable -->
<bean id="bookService" class="x.y.service.DefaultBookService"/>

<!-- cache definitions -->
<cache:advice id="cacheAdvice" cache-manager="cacheManager">
	<cache:caching cache="books">
		<cache:cacheable method="findBook" key="#isbn"/>
		<cache:cache-evict method="loadBooks" all-entries="true"/>
	</cache:caching>
</cache:advice>

<!-- apply the cacheable behavior to all BookService interfaces -->
<aop:config>
	<aop:advisor advice-ref="cacheAdvice" pointcut="execution(* x.y.BookService.*(..))"/>
</aop:config>

<!-- cache manager definition omitted -->
```

上面的配置中，`bookService` 是可配缓存的服务。在 `cache:advice` 指定方法 `findBooks`

###  32.6 配置缓存存储
开箱即用，缓存抽象提供了多种存储集成。要使用它们，需要简单地声明一个适当的`CacheManager` - 一个控制和管理`Cache`s，可用于检索这些存储。

32.6.1 JDK ConcurrentMap-based Cache
-------
基于JDK的`Cache`实现位于`org.springframework.cache.concurrent`包下。它允许使用`ConcurrentHashMap`作为后备缓存存储。

```
<!-- simple cache manager -->
<bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
	<property name="caches">
		<set>
			<bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="default"/>
			<bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" p:name="books"/>
		</set>
	</property>
</bean>
```

上面的代码片段使用`SimpleCacheManager`创建一个`CacheManager`,为两个嵌套的`ConcurrentMapCache`实例命名为*default*和*books*。请注意，这些名称是为每个缓存直接配置的。

由于缓存是由应用程序创建的，因此它必须与其生命周期一致，使其适用于基本用例，测试或简单应用程序。缓存规模好，速度快，但不提供任何管理、持久化能力或驱逐合同。

32.6.2 Ehcache-based Cache（基于Ehcache的缓存）
-------

> Ehcache 3.x完全符合JSR-107标准，不需要专门的支持。

Ehcache 2.x的实现位于org.springframework.cache.ehcache包下。要使用它，只需要声明适当的`CacheManager`：
```
<bean id="cacheManager"
      class="org.springframework.cache.ehcache.EhCacheCacheManager" p:cache-manager-ref="ehcache"/>

<!-- EhCache library setup -->
<bean id="ehcache"
      class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean" p:config-location="ehcache.xml"/>
```

此设置引导Spring IoC（通过ehcache bean）中的`ehcache`库，然后将其连接到专用的`CacheManager`实现中。请注意，整个ehcache特定的配置是从`ehcache.xml`读取的。

32.6.3 Caffeine Cache
----------
Caffeine是Java 8的重写Guava缓存，其实现位于`org.springframework.cache.caffeine`包下，并提供了对Caffeine的几项功能的访问。

根据需要，配置创建缓存的`CacheManager`很简单：
```
<bean id="cacheManager"
      class="org.springframework.cache.caffeine.CaffeineCacheManager"/>
```

还可以明确提供使用的缓存。在这种情况下，只有manager才能提供：
```
<bean id="cacheManager" class="org.springframework.cache.caffeine.CaffeineCacheManager">
	<property name="caches">
		<set>
			<value>default</value>
			<value>books</value>
		</set>
	</property>
</bean>
```
Caffeine `CacheManager`也支持自定义的`Caffeine` 和 `CacheLoader`。查看[Caffeine documentation](https://github.com/ben-manes/caffeine/wiki)获取更多信息。

32.6.4 GemFire-based Cache（基于GemFire的缓存）
--------
GemFire是面向内存/磁盘支持，弹性可扩展，持续可用，主动（内置基于模式的订阅通知），全局复制数据库，并提供功能齐全的边缘缓存。有关如何使用GemFire作为CacheManager（及更多）的更多信息，请参考 [Spring Data GemFire reference documentation](http://docs.spring.io/spring-gemfire/docs/current/reference/html/)。

32.6.5 JSR-107 Cache
-----
Spring的缓存抽象也可以使用兼容JSR-107的缓存。JCache实现位于`org.springframework.cache.jcache`包下。
要使用它，只需要声明适当的`CacheManager`：
```
<bean id="cacheManager"
      class="org.springframework.cache.jcache.JCacheCacheManager"
      p:cache-manager-ref="jCacheManager"/>

<!-- JSR-107 cache manager setup  -->
<bean id="jCacheManager" .../>
```
32.6.6 Dealing with caches without a backing store（处理没有后端存储的缓存）
---------
有时在切换环境或进行测试时，可能会有缓存声明，而不配置实际的后备缓存。由于这是一个无效的配置，因此在运行时将会抛出异常，因为缓存基础结构无法找到合适的存储。在这种情况下，而不是删除缓存声明（这可以证明是乏味的），可以连接一个不执行缓存的简单的虚拟缓存，也就是强制每次执行缓存的方法：
```
<bean id="cacheManager" class="org.springframework.cache.support.CompositeCacheManager">
	<property name="cacheManagers">
		<list>
			<ref bean="jdkCache"/>
			<ref bean="gemfireCache"/>
		</list>
	</property>
	<property name="fallbackToNoOpCache" value="true"/>
</bean>
```

上面的`CompositeCacheManager`链接多个`CacheManager`s，另外，通过`fallbackToNoOpCache`标志，添加了一个*no op*缓存，用于所有不被配置的缓存管理器处理的定义。也就是说，在`jdkCache`或`gemfireCache`（上面配置）中找不到的每个缓存定义都将由*no op*缓存来处理，这不会存储任何导致目标方法每次执行的信息。

### 32.7 插入不同的后端缓存
-------------

显然，有很多缓存产品可以用作后备存储。要插入它们，需要提供一个`CacheManager`和`Cache`实现，因为不幸的是没有可用的标准供我们使用。这听起来比实际上更难听，这些类往往是简单的适配器，将缓存抽象框架映射到存储API的顶部，就像ehcache类可以显示一样。大多数`CacheManager`类都可以使用`org.springframework.cache.support`包中的类，例如`AbstractCacheManager`，其中只需要完成实际的映射即可完成代码。我们希望及时提供与Spring集成的库可以填补这个小的配置差距。

### 32.8 如何设置 TTL/TTI/Eviction policy/XXX 功能?
-----
直接通过您的缓存提供商。缓存抽象是一个抽象而不是缓存实现。您正在使用的解决方案可能支持各种数据策略和其他解决方案不同的拓扑（例如JDK `ConcurrentHashMap`），因为缓存中的提取将无济于事，因为不需要后台支持。这样的功能应该通过后台缓存，配置它或通过其本机API直接控制。