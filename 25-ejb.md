# 25. 整合EJB

## 25.1 介绍 {#toc_1}

作为一个轻量级的容器，Spring通常被认为是EJB的替代品。我们相信对域大多数就算不是最多的应用和用例来说，Spring作为一个容器结合其丰富的在事物，ORM和JDBC访问方面的支持功能，是比通过一个EJB容器和EJBs来实现同等的功能更好的选择。



然后，需要注意的是使用Spring并不会阻止你使用EJBs。实际上，Spring使访问EJBs，实现EJBs以及其中的功能更加容易。另外的，使用Spring来获取EJBs提供的服务可以允许这些服务透明的在本地EJB，远程EJB或者POJO\(Plain Old Java Object\)变量中切换，而不需要客户端代码做改变。

在这一章，我们会看到Spring使如何帮助你获取和实现EJBs。Spring提供特定的值在访问无状态会话组件\(SLSBs\)，所以我们从这里开始讨论。

## 25.2 获取EJBs {#toc_2}

### 25.2.1 概念 {#toc_3}

调用一个本地或者远程的无状态会话组件中的方法，客户端代码必须正常执行JNDI查找来获取本地或者远程的EJB Home Object，然后使用’create’ 方法来获取一个真正的（本地或者远程的）EJB对象。然后一个或者多个方法会在EJB中被调用。  
为了避免重复底层级的代码，很多EJB应用使用服务定位器和业务代表模式。它们比起在客户端代码中到处使用JNDI lookup要好，但是也有很明显不好的地方，比如：

* 通常使用EJBs的代码依赖于服务定位器或者业务代理单例，所以很难测试。
* 在使用了服务定位器模式而不是用一个业务代理的情况下，应用代码仍然需要结束时在一个EJB home上调用create\(\)方法，并且处理结果中的异常。所以它仍然和EJB的API绑在一起，还会有EJB编程模型的复杂性。
* 实现业务代理模式常常会带来明显的代码重复，我们不得不写很多的方法只是简单的调用EJB中相同的方法。

### 25.2.2 访问本地无状态会话组件\(SLSBs\) {#toc_4}

假设我们有一个web容器需要使用到本地的EJB。我们将会遵循最好的实践并且使用EJB业务方法接口模式，所以就是EJB的本地接口扩展一个不是EJB-specific的业务方法接口。让我们叫这个接口为`MyComponent`。

```
public interface MyComponent {
    ...
}
```

一个主要使用业务方法接口模式的原因是为了确保本地接口中的方法签名和bean实现类同步是自动的。另外的一个原因是它让我们可以在需要的时候更加容易转换为POJO来实现这个服务。当然我们也会需要去实现本地的home接口并且提供一个实现类来实现`SeesionBean`和`MyComponent`业务方法接口。现在我们唯一需要的Java编码是暴露一个控制器中setter方法进行设置`MyComponent`从而将我们的web层控制器连接到这个EJB实现。这将会在容器中保存引用作为一个实例变量：

```
private MyComponent myComponent;

public void setMyComponent(MyComponent myComponent) {
    this.myComponent = myComponent;
}
```

我们之后可以使用这个实例变量在这个控制器中的任何一个业务方法中。现在假设我们在一个Spring控制器以外获取了我们的控制器对象，我们可以在\(相同上下文中\)配置一个`LocalStatelessSessionProxyFactoryBean`实例，这是一个EJB代理对象。这个代理的配置以及控制器的`myComponet`属性值是在一个配置项完成的，如下：

```
<bean id="myComponent"
        class="org.springframework.ejb.access.LocalStatelessSessionProxyFactoryBean">
    <property name="jndiName" value="ejb/myBean"/>
    <property name="businessInterface" value="com.mycom.MyComponent"/>
</bean>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
```

这个配置的生效背后Spring AOP框架做了很多的工作，尽管你并不强制性需要使用AOP观念就能享受到这个结果了。这个`myComponet`bean定义了一个EJB的代理实现了业务接口。这个EJB local home在启动的时候会被缓存，所以这只有一个单独的JNDI lookup。每当这个EJB被调用的时候，这个代理就会调用本地EJB的`classname`方法并且调用EJB上相应的业务方法。  
`myController`bean定义为这个EJB代理设置了控制类的`myComponet`的属性。  
可选的\(最好是在许多这样代理定义的情况下\)，考虑使用`<jee:local-slsb>`在Spring”jee”命名空间中配置元素：

```
<jee:local-slsb id="myComponent" jndi-name="ejb/myBean"
        business-interface="com.mycom.MyComponent"/>

<bean id="myController" class="com.mycom.myController">
    <property name="myComponent" ref="myComponent"/>
</bean>
```

这种EJB获取机制极大的简化了应用代码：web层的代码\(或者其他EJB客户端代码\)对使用EJB没有依赖。如果我们想要使用一个POJO或者一个模拟对象获取其他的测试桩来替代这个EJB的引用，我们可以简单的修改这个`myComponet`bean的定义而不需要修改任何Java代码。另外的，我们也不需要在我们应用中写任何一行JNDI lookup或者其他EJB样板代码。  
Benchmarks和其他在实际应用中的经验表明这种方法\(涉及目标EJB的反射调用\)的性能开销是很小的，通常在使用中是不可检测到的。记住我们并不想要细密度的导出调用EJBs，因为在应用服务器中有使用EJB基础架构的开销。  
这里有一个关于JNDI lookup的警告。在一个bean容器中，类最好作为一个单例来使用\(似乎没有道理把它作为原型使用\).然而，如果那个bean容器预先实例化了单例你也许会遇到问题如果这个Bean容器在EJB容器加载目标EJB之前加载了。因为这个JNDI lookup会在这个类的init\(\)方法中调用并且被缓存，但是这个EJB还没有绑定到对应的目标地址。这个解决的方法是不要预先实例化这个工程对象，而是在第一次使用的时候再创建它。在XML容器中，它被`lazy-init`属性控制。

尽管这不会是大多是大多数Spring用户的兴趣所在，那些使用EJB做AOP编程的用户也许想要看看`LocalSlsbInvokerInterceptor`

### 25.2.3 获取远程的无状态会话组件\(SLSBs\) {#toc_5}

获取远程EJBs基本上和获取本地EJBs相同，除了`SimpleRemoteStatelessSessionProxyFactoryBean`或者`<jee:remote-slsb>`配置元素会被用到。当然不论用不用到Spring，远程调用语义应用；一个调用在其他VM其他电脑上的对象的方法有时需要被不同的对待根据应用场景和失败的句柄。  
Spring的EJB客户端再一次比非Spring方法更加有优势。通常EJB客户端代码要轻松的在调用EJBs本地和远程之间切换是有问题的。因为远程的接口方法必须声明它们可能抛出`RemoteException`，而客户端代码必须处理它，而本地的接口没有这个异常。客户端处理本地EJBs的代码如果要改为处理远程EJBs通常需要修改增加处理远程异常，并且客户端如果本来是处理远程EJBs的需要改为处理本地EJBs的代码可以保持不变但是做了很多不必要的处理远程异常的事情，或者就需要移除这部分代码。使用Spring 远程EJB代理你可以不用声明任何抛出`RemoteException`的业务方法接口和实现EJB代码，这里有一个远程接口除了抛出异常以外完全相同，而是依靠代理来动态的处理这两个接口使它们表现的一样。这样客户端代码无需处理  
`RemoteException`类。任何在调用这个EJB过程中抛出的`RemoteException`将会被再次抛出作为没有没检查的`RemoteAccessException`类，这个类是`RuntimeException`的子类。这个目标服务可以在不需要客户端代码知晓的情况下在本地EJB或者远程EJB\(或者甚至是简单的Java对象\)实现中切换。当然，这个是可选择的；你也可以在你的业务接口中声明`RemoteException`。

### 25.2.4 获取EJB 2.x SLSBs和EJB 3 SLSBs的比较 {#toc_6}

通过Spring获取EJB 2.x会话组件和EJB3会话组件基本上是透明的。Spring的EJB获取器包含了`<jee:local-slsb>`和`<jee:remote-slsb>`，会在运行时透明的采用实际的组件。它们如果找到了home interface（EJB 2.x 风格）就会获取，反之就直接运行组件调用\(EJB 3风格\)。  
Note：对于EJB 3会话组件，你也可以直接的使用`JndiObjectFactoryBean`／`jee:jndi-lookup`因为完全有效的组件引用在这里被暴露给了JNDI loolups。定义显示的`<jee:local-slsb>`／`<jee:remote-slsb>`lookups只是提供了一致和更加显示的EJB访问配置。

