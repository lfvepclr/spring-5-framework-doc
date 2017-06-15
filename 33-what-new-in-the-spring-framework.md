# **Spring框架的新功能**

这一章主要提供Spring框架新的功能和变更。

升级到新版本的框架可以参考。[Spring git](https://github.com/spring-projects/spring-framework/wiki/Migrating-from-earlier-versions-of-the-Spring-Framework)。

内容列表

[Spring 5.x框架新的功能](https://github.com/spring-projects/spring-framework/wiki/What's-New-in-the-Spring-Framework#whats-new-in-spring-framework-5x)

[Spring 4.x框架新的功能](https://github.com/spring-projects/spring-framework/wiki/What's-New-in-the-Spring-Framework#whats-new-in-spring-framework-4x)

[Spring 3.x框架新的功能](https://github.com/spring-projects/spring-framework/wiki/What's-New-in-the-Spring-Framework#whats-new-in-spring-framework-3x)

## **Spring FrameWork 5.0新的功能**

### **JDK 8+和Java EE7+以上版本**

#### 整个框架的代码基于java8

* 通过使用泛型等特性提高可读性
* 对java8提高直接的代码支撑
* 运行时兼容JDK9
* Java EE 7API需要Spring相关的模块支持
* 运行时兼容Java EE8 API
* 取消的包,类和方法
* 包 beans.factory.access
* 包 dbc.support.nativejdbc
* 从spring-aspects 模块移除了包mock.staicmock,不在提AnnotationDrivenStaticEntityMockingControl支持
* 许多不建议使用的类和方法在代码库中删除

#### **核心特性**

JDK8的增强：

* 访问Resuouce时提供getFile或和isFile防御式抽象
* 有效的方法参数访问基于java 8反射增强
* 在Spring核心接口中增加了声明default方法的支持一贯使用JDK7 Charset和StandardCharsets的增强
* 兼容JDK9
* Spring 5.0框架自带了通用的日志封装
* 持续实例化via构造函数\(修改了异常处理\)
* Spring 5.0框架自带了通用的日志封装
* spring-jcl替代了通用的日志，仍然支持可重写
* 自动检测log4j 2.x, SLF4J, JUL（java.util.Logging）而不是其他的支持
* 访问Resuouce时提供getFile或和isFile防御式抽象
* 基于NIO的readableChannel也提供了这个新特性

#### **核心容器**

* 支持候选组件索引\(也可以支持环境变量扫描\)
* 支持@Nullable注解
* 函数式风格GenericApplicationContext/AnnotationConfigApplicationContext
* 基本支持bean API注册
* 在接口层面使用CGLIB动态代理的时候，提供事物，缓存，异步注解检测
* XML配置作用域流式
* Spring WebMVC
* 全部的Servlet 3.1 签名支持在Spring-provied Filter实现
* 在Spring MVC Controller方法里支持Servlet4.0 PushBuilder参数
* 多个不可变对象的数据绑定\(Kotlin/Lombok/@ConstructorPorties\)
* 支持jackson2.9
* 支持JSON绑定API
* 支持protobuf3
* 支持Reactor3.1 Flux和Mono

#### **SpringWebFlux**

* 新的spring-webflux模块，一个基于reactive的spring-webmvc，完全的异步非阻塞，旨在使用enent-loop执行模型和传统的线程池模型。
* Reactive说明在spring-core比如编码和解码
* spring-core相关的基础设施，比如Encode 和Decoder可以用来编码和解码数据流；DataBuffer 可以使用java ByteBuffer或者Netty ByteBuf;ReactiveAdapterRegistry可以对相关的库提供传输层支持。
* 在spring-web包里包含HttpMessageReade和HttpMessageWrite

#### **测试方面的改进**

* 完成了对JUnit 5’s Juptier编程和拓展模块在Spring TestContext框架
* SpringExtension:是JUnit多个可拓展API的一个实现，提供了对现存Spring TestContext Framework的支持，使用@ExtendWith\(SpringExtension.class\)注解引用。
* @SpringJunitConfig:一个复合注解
* @ExtendWith\(SpringExtension.class\) 来源于Junit Jupit
* @ContextConfiguration 来源于Srping TestContext框架
* @DisabledIf 如果提供的该属性值为true的表达或占位符，信号：注解的测试类或测试方法被禁用
* 在Spring TestContext框架中支持并行测试
* 具体细节查看Test 章节 通过SpringRunner在Sring TestContext框架中支持TestNG, Junit5,新的执行之前和之后测试回调。
* 在testexecutionlistener API和testcontextmanager新beforetestexecution\(\)和aftertestexecution\(\)回调。MockHttpServletRequest新增了getContentAsByteArray\(\)和getContentAsString\(\)方法来访问请求体
* 如果字符编码被设置为mock请求，在print\(\)和log\(\)方法中可以打印Spring MVC Test的redirectedUrl\(\)和forwardedUrl\(\)方法支持带变量表达式URL模板。
* XMLUnit 升级到了2.3版本。



