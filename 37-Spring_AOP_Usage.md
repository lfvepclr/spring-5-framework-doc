
## Spring AOP的经典用法
在本附录中，我们会讨论一些初级的Spring AOP接口，以及在Spring 1.2应用中所使用的AOP支持。
对于新的应用，我们推荐使用 Spring AOP 2.0来支持，在[AOP](#core.adoc)章节有介绍。
但在已有的项目中，或者阅读数据或者文章时，可能会遇到Spring AOP 1.2风格的示例。
Spring 2.0完全兼容Spring 1.2，在本附录中所有的描述都是Spring 2.0所支持的。



### Spring中的切点API
一起看一下Spring是如何处理关键切点这个概念。


#### 概念
Spring的切点模型能够让切点重（chong）用不同的独立的增强类型。
这样可以实现，针对不同的增强，使用相同的切点。

`org.springframework.aop.Pointcut`是一个核心接口，用于将增强定位到特定的类或者方法上。
完整的接口信息如下：

```java
public interface Pointcut {

    ClassFilter getClassFilter();

    MethodMatcher getMethodMatcher();

}

```

将`Pointcut`拆分成两个部分，允许重（chong）用类和方法匹配的部分，和细粒度的组合操作（例如和其他的方法匹配器执行一个“组合”操作）。

`ClassFilter`接口用于将切点限制在给定的目标类上。
如果`matches()`方法总是返回true，所有的类都会被匹配上。

```java
public interface ClassFilter {

    boolean matches(Class clazz);

}
```


`MethodMatcher`接口通常更为重要。完整的接口描述如下：

```java
public interface MethodMatcher {

    boolean matches(Method m, Class targetClass);

    boolean isRuntime();

    boolean matches(Method m, Class targetClass, Object[] args);

}

```

`matches(Method, Class)`方法用于测试切点是否匹配目标类的一个指定方法。
这个测试可以在AOP代理创建时执行，避免需要在每一个方法调用时，再测试一次。
如果对于一个给定的方法，`matches(Method, Class)`方法返回true，并且对于同MethodMatcher实例的`isRuntime()`方法也返回true，
那么在每次被匹配的方法执行时，都会调用`boolean matches(Method m, Class targetClass, Object[] args)`方法。
这样使得在目标增强执行前，一个切点可以在方法执行时立即查看入参。

大部分MethodMatcher是静态的，意味着他们`isRuntime()`方法的返回值是false。
在这种情况下，`boolean matches(Method m, Class targetClass, Object[] args)`方法是永远不会被调用的。

**提示**
> 如果可以，尽量将切点设置为静态，这样在一个AOP代理生成后，可以允许AOP框架缓存评估的结果。



#### 切点操作
Spring在切点的操作：尤其是，`组合（union）`和`交叉（intersection）`。


* 组合意味着方法只需被其中任意切点匹配。
* 交叉意味着方法需要被所有切点匹配。
* 组合通常更为有用。
* 切点可以使用`org.springframework.aop.support.Pointcuts`类或者`org.springframework.aop.support.ComposablePointcut`中的静态方法组合。
然而，使用AspectJ的切点表达式通常是一种更为简单的方式。


#### AspectJ切点表达式
自从2.0版以后，Spring所使用的最重要切点类型就是`org.springframework.aop.aspectj.AspectJExpressionPointcut`。
这个切点使用了一个AspectJ支持的库，用以解析AspectJ切点表达式的字符串。

有关原始AspectJ切点元素支持的讨论，请参阅之前章节。


#### 方便的切点实现

Spring提供了几个方便的切点具体实现。有些可以在框架外使用；其他的则为应用程序的特定切点实现所需要的子类。


##### 静态切点
静态切点是基于方法和目标类的，不能将方法参数也考虑其中。
对于大多数用法，静态切点是足够且最佳的选择。

对于Spring来说，当一个方法第一次被调用是，对静态切点仅仅评估一次是可行的：在本次评估后，再次调用该方法时，就没有必要再对切点进行评估。

我们一起看一些Spring中包含的静态切点具体实现。

###### 正则表达式切点
一个显而易见的方式是使用正则表达式来指定静态切点。几个在Spring之外的框架可以实现这部分功能。
`org.springframework.aop.support.Perl5RegexpMethodPointcut`是一个常见的正则表达式切点，使用Perl 5正则表达式语法。
`Perl5RegexpMethodPointcut`类的正则表达式匹配依赖于Jakarta ORO。
Spring也提供了`JdkRegexpMethodPointcut`类，可以在JDK 1.4版本之上使用正则表达式。


使用`Perl5RegexpMethodPointcut`类，你可以提供一个正则表达式字符串的列表。
如果与该列表的中的某个正则匹配上了，那么切点的判定就为true（判定的结果是这些切点的有效组合）。

使用方法如下所示：

```xml
<bean id="settersAndAbsquatulatePointcut"
        class="org.springframework.aop.support.Perl5RegexpMethodPointcut">
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>

```

Spring提供了一个方便的类，`RegexpMethodPointcutAdvisor`，允许我们引用一个Advice（记住一个Advice可能是一个介入增强、前置增强、或者异常抛出增强等）。
实际上，Spring会使用`JdkRegexpMethodPointcut`类。
使用`RegexpMethodPointcutAdvisor`简化配置，这个类封装了切点和增强。如下所示：

```xml
<bean id="settersAndAbsquatulateAdvisor"
        class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    <property name="advice">
        <ref bean="beanNameOfAopAllianceInterceptor"/>
    </property>
    <property name="patterns">
        <list>
            <value>.*set.*</value>
            <value>.*absquatulate</value>
        </list>
    </property>
</bean>

```

`RegexpMethodPointcutAdvisor`可以被任意类型的增强使用。

###### 属性驱动切入
一个重要的静态切点就是`metadata-driven`切点。它会使用一些元数据属性信息：通常是源码级的元数据。

##### 动态切点
动态切点的判定代价比静态切点要大。动态切点除了静态信息外，还需要考虑方法`参数`。
这意味着它们在每次方法调用时都必须进行判定；判定的结果不能被缓存，因为参数是变化的。

代表性的事例是`控制流`切点。

###### 控制流切点
Spring的控制流切点在概念上与AspectJ的`cflow`切点类似，不过功能稍弱。（目前没有方法，可以指定一个切点在其他切点匹配的连接点后执行。）
一个控制流切点匹配当前的调用栈【待定】。例如，如果一个连接点被一个在`com.mycompany.web`包中、或者`SomeCaller`类中的方法调用，就会触发。
控制流切点使用`org.springframework.aop.support.ControlFlowPointcut`类来指定。
**说明**
> 控制流切点在运行时进行评估明显代价更大，甚至是其他动态切点。在Java 1.4，大概是其他动态切点的5倍。



#### Pointcut父类
Spring提供了一些有用的切点父类，方便开发者实现自己的切点。

因为静态切点是最为实用的，你可能需要实现StaticMethodMatcherPointcut的子类，如下所示。
这里只需要实现一个抽象方法即可（虽然也可以覆盖其他方法来自定义类的行为）。

```java
class TestStaticPointcut extends StaticMethodMatcherPointcut {

    public boolean matches(Method m, Class targetClass) {
        // return true if custom criteria match
    }

}

```

Spring也有动态切点的父类。
在Spring 1.0 RC2版本之后，可以自定义任意增强类型的切点。

#### 自定义切点
由于切点在Spring AOP中都是Java类，而不是语言特征（就像在AspectJ中），可以声明自定义切点，无论静态还是动态。
自定义切点在Spring中是可以任意复杂的。然而，如果可以，推荐使用AspectJ切点表达式。

**说明**
> Spring之后的版本可能支持由JAC提供的“语义切点”。
> 例如：在目标对象中，所有修改实例变量的方法。

### Spring中的Advice接口

现在让我们看一下Spring AOP如何处理Advice（增强）。

#### Advice的生命周期
每个Advice都是一个Spring的Bean。一个Advice实例在被增强的对象间共享，或者对于每一个被增强的对象都是唯一的。
这取决于增强是类级的、还是对象级的【待定】。
Each advice is a Spring bean. An advice instance can be shared across all advised
objects, or unique to each advised object. This corresponds to `per-class` or
`per-instance` advice.

Per-class级增强最为常用。它适用于通常的增强，例如事务增强。这种增强不依赖于代理对象或者增加新的状态；它们只是对方法和参数进行增强。
Per-instance级增强适用于介绍，支持它很复杂【待定】。在本示例中，增强对被代理的对象添加了一个状态。

也可以在同一个AOP代理中，使用共享和per-instance级增强的组合。


#### Spring中的增强类型
Spring在框架层之外，支持多种方式的增强，并且支持任意增强类型的可扩展性。
我们一起了解一下标准增强类型和增强的基础概念。


##### 拦截式环绕型增强
Spring中最基本的增强类型之一就是`拦截式环绕型增强`，
通过使用方法拦截器，Spring完全符合AOP联盟的环绕型增强接口。
环绕型方法拦截器应该实现以下接口：

```java
public interface MethodInterceptor extends Interceptor {

    Object invoke(MethodInvocation invocation) throws Throwable;

}

```
`invoke()` 方法的`MethodInvocation`表示了将要被调用的方法、目标连接点、AOP代理、以及该方法的参数。
`invoke()` 方法应当返回调用结果：目标连接点的返回值。


一个简单的`方法拦截器`实现如下所示：

```java
public class DebugInterceptor implements MethodInterceptor {

    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("Before: invocation=[" + invocation + "]");
        Object rval = invocation.proceed();
        System.out.println("Invocation returned");
        return rval;
    }

}

```

注意调用MethodInvocation对象的`proceed()`方法。这个方法将拦截器链路调向连接点。
大多数拦截器会调用该方法，并返回该方法的值。但是，就像任意环绕增强一样，一个方法拦截器也可以返回一个不同的值，或者抛出一个异常，而不是调用`proceed()`方法。
但是，没有足够的理由，不要这么干！

**说明**
> 方法拦截器提供与其他AOP联盟标准的AOP实现的互通性。
> 在本章剩余部分讨论的其他类型增强，会以Spring特定的方式实现AOP的概念。
> 使用最为具体的类型增强有一定优势，但如果你想在其他AOP框架中使用切面，就需要坚持使用方法拦截器。
> 需要注意的是，切点在框架间是不通用的，AOP联盟目前没有定义切点的接口。


##### 前置增强
一个简单的增强类型是前置增强。这种增强不需要一个`MethodInvocation`对象，因为它仅仅在方法进入时被调用。

前置增强的优势是不需要调用`proceed()`方法，因此不会无故中断调用链。


`MethodBeforeAdvice`接口如下所示。【待定】
interface is shown below. (Spring's API design would allow for
field before advice, although the usual objects apply to field interception and it's
unlikely that Spring will ever implement it).

```java
public interface MethodBeforeAdvice extends BeforeAdvice {

    void before(Method m, Object[] args, Object target) throws Throwable;

}

```

需要注意的是该方法的返回类型是`void`。前置增强可以在连接点钱插入一些自定义的行为，但是不能改变返回结果。
如果一个前置增强抛出一个异常，它会中断调用链中接下来的执行步骤。这个异常将传递到调用链的上一层。
如果该异常没有被处理，或者在被调用方法中签名【待定】，这个异常会直接传递给方法调用方；否则，该异常会被AOP代理类封装到一个未经检查的异常中。


在Spring中，一个前置增强的例子：统计所有方法的执行次数：


```java
public class CountingBeforeAdvice implements MethodBeforeAdvice {

    private int count;

    public void before(Method m, Object[] args, Object target) throws Throwable {
        ++count;
    }

    public int getCount() {
        return count;
    }
}
```

**提示**
> 前置增强可以被任何切点使用。


##### 异常抛出增强
当连接点返回的结果是一个抛出的异常时，异常抛出增强会被调用。
Spring提供异常抛出增强。
需要主意的是`org.springframework.aop.ThrowsAdvice` 接口不包括任何方法：它是一个标签式接口，标识给出的对象实现了一个或多个类型的异常抛出增强。
它们的格式如下所示：

```java
afterThrowing([Method, args, target], subclassOfThrowable)
```

只有最后一个参数是必须的。这个方法可能拥有1个或者4个参数，取决于增强方法是否对被增强的方法和方法参数感兴趣。
下面的类是异常抛出增强的例子。

如果一个`RemoteException`（包括子类）被抛出，下面这个增强就会被调用：

```java

public class RemoteThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }

}

```

如果一个`ServletException`被抛出，下面这个增强就会被调用。
与上面不同的是，该方法声明了4个参数，因此它可以访问被调用的方法、方法参数和目标对象：

```java
public class ServletThrowsAdviceWithArguments implements ThrowsAdvice {

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }

}

```

最后一个示例描述了，一个类中如果声明两个方法，可以同时处理`RemoteException`和`ServletException`。
一个类中可以包含任意个异常抛出增强的处理方法。

```java
public static class CombinedThrowsAdvice implements ThrowsAdvice {

    public void afterThrowing(RemoteException ex) throws Throwable {
        // Do something with remote exception
    }

    public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
        // Do something with all arguments
    }
}

```

**说明**
> 如果一个异常抛出增强本身抛出了一个异常，它将覆盖掉原始的异常（例如，改变抛给用户的异常）。
> 这个覆盖的异常通常是一个运行时异常；这样就可以兼容任何的方法签名。
> 但是，如果一个异常抛出增强抛出了一个检查时异常，这个异常必须和该目标方法的声明匹配，以此在一定程度上与特定的目标签名相结合。

_不要抛出与目标方法签名不兼容的检查时异常！_


**提示**
> 异常抛出增强可以被任意切点使用。


##### 后置增强
后置增强必须实现`org.springframework.aop.AfterReturningAdvice`接口，如下所示：

```java
public interface AfterReturningAdvice extends Advice {

    void afterReturning(Object returnValue, Method m, Object[] args,
            Object target) throws Throwable;

}

```

一个后置增强可以访问被调用方法的返回值（不能修改）、被调用方法、方法参数、目标对象。

下面的后置增强统计了所有执行成功的方法调用，即没有抛出异常的调用：


```java
public class CountingAfterReturningAdvice implements AfterReturningAdvice {

    private int count;

    public void afterReturning(Object returnValue, Method m, Object[] args,
                    Object target) throws Throwable {
                ++count;
            }

    public int getCount() {
        return count;
    }

}

```

这个增强不会改变执行路径。如果它抛出了一个异常，该异常会抛出到拦截链，而不是返回返回值。

**提示**
> 后置增强可以被任意切点使用。



##### 引介增强
Spring将引介增强当作一个特殊的拦截式增强。

引介增强需要一个`IntroductionAdvisor`和一个`IntroductionInterceptor`实现以下接口：

```java
public interface IntroductionInterceptor extends MethodInterceptor {

    boolean implementsInterface(Class intf);

}
```
`invoke()`方法继承自AOP联盟的`MethodInterceptor`接口，必须被引介实现：
也就是说，如果被调用的方式是一个被介入的接口，该引介拦截器就会负责处理该方法的调用，不能调用`proceed()`方法。

不是所有的切点都可以使用引介增强，因为它只适用于类级，而不是方法级。
你可以通过`IntroductionAdvisor`来使用引介增强，该类有如下几个方法：

```java
public interface IntroductionAdvisor extends Advisor, IntroductionInfo {

    ClassFilter getClassFilter();

    void validateInterfaces() throws IllegalArgumentException;

}

public interface IntroductionInfo {

    Class[] getInterfaces();

}
```

没有`MethodMatcher`，因此也没有`Pointcut`与引介增强相关联。只有类过滤器是符合逻辑的。
`getInterfaces()`方法会返回被该增强器引介的接口集合。
`validateInterfaces()`会在内部被调用，用于确定被引介的接口是否可以被配置的`IntroductionInterceptor`所实现。

让我们一起看一个Spring测试套件的简单示例。
假设我们想要将以下的接口介入到一个或多个对象中：

```java
public interface Lockable {

    void lock();

    void unlock();

    boolean locked();

}

```

这里解释了一个`mixin`。
我们希望能够将被增强的对象转换成一个Lockable对象，无论它原来的类型是什么，并且调用转换后对象的lock和unlock方法。
如果调用lock()方法，我们希望所有的setter方法抛出一个`LockedException`异常。
这样我们就可以提供一个切面，使该对象不可变，而不需要对该对象有所了解：一个很好的AOP示例。

首先，我们需要一个`IntroductionInterceptor` ，这很重要。
在这种情况下，我扩展`org.springframework.aop.support.DelegatingIntroductionInterceptor`类。
我们可以直接实现IntroductionInterceptor，但是大多数情况下使用`DelegatingIntroductionInterceptor`是最合适的。

`DelegatingIntroductionInterceptor`被设计成代理一个需要被引介接口的真实实现，隐藏使用拦截器去这样做。
使用构造函数的参数，可以把代理设置为任意对象；默认的代理（使用无参构造函数时）就是引介增强【待定】。
The delegate can be set to any object using a constructor argument; the
default delegate (when the no-arg constructor is used) is this.
因此在下面的示例中，代理是`DelegatingIntroductionInterceptor`的子类`LockMixin`。
给定的代理（默认是自身），一个`DelegatingIntroductionInterceptor`对象查找所有被该代理所实现的接口结合（除了IntroductionInterceptor），
并支持代理介入它们。
像`LockMixin`的子类调用`suppressInterface(Class intf)`方法，可以禁止不能被暴露的接口被调用。
然而无论一个`IntroductionInterceptor`准备支持多少个接口，`IntroductionAdvisor`都会控制哪些接口实际是被暴露的。
一个被引介的接口会隐藏掉目标对象的所有接口的实现。

因此`DelegatingIntroductionInterceptor`的子类LockMixin，也实现了Lockable接口本身。
超类会自动获取Lockable能支持的引介，因此我们不需要为此设置。这样我们就可以引介任意数量的接口。


需要注意所使用的`locked`对象变量。它有效的增加了目标对象的附加状态。

```java
public class LockMixin extends DelegatingIntroductionInterceptor implements Lockable {

    private boolean locked;

    public void lock() {
        this.locked = true;
    }

    public void unlock() {
        this.locked = false;
    }

    public boolean locked() {
        return this.locked;
    }

    public Object invoke(MethodInvocation invocation) throws Throwable {
        if (locked() && invocation.getMethod().getName().indexOf("set") == 0) {
            throw new LockedException();
        }
        return super.invoke(invocation);
    }

}

```

通常是不需要覆盖`invoke()`方法的：如果方法被引介的话，`DelegatingIntroductionInterceptor`代理会调用方法，否则调用连接点，通常也是足够了。
在这种情况下，我们需要加入一个检查：如果处于锁住的模式，任何setter方法都是不能被调用。

所需要的引介增强器非常简单。它所需要做的仅仅是持有一个明确的`LockMixin`对象，指定需要被引介的接口（在本示例中，仅仅是`Lockable`接口）。
一个更加复杂的例子是持有一个引介拦截器的引用（被定义为一个原型）：在本示例中，没有配置和一个`LockMixin`对象相关，所有我们简单使用`new`来创建。


```java
public class LockMixinAdvisor extends DefaultIntroductionAdvisor {

    public LockMixinAdvisor() {
        super(new LockMixin(), Lockable.class);
    }

}

```

我们可以非常简单的使用这个增强器：不需要任何配置。（但是，使用`IntroductionInterceptor`的同时，不使用`IntroductionAdvisor`是不行的。）
和之前介绍的一样，Advisor是一个per-instance级的，它是有状态的。
因此对于每一个被增强的对象，就像`LockMixin` 一样，我们都需要一个不同的`LockMixinAdvisor`。
Advisor就是被增强对象状态的一部分。

我们可以使用编程的方式应用这个Advisor，使用`Advised.addAdvisor()`方法，或者在XML中配置（推荐），就像其他Advisor一样。
下面会讨论所有代理创建的选择方式，包括“自动代理创建者”正确的处理引介和有状态的mixins。


### Spring中的Advisor接口

在Spring中，一个Advisor是一个切面，仅仅包括了一个和切点表达式相关联的增强对象。

除了介绍的特殊情况，任何Advisor都可以被任意增强使用。
`org.springframework.aop.support.DefaultPointcutAdvisor` 是最为常用的advisor类。
例如，它可以被`MethodInterceptor`、`BeforeAdvice`和`ThrowsAdvice`使用。

在Spring的同一个AOP代理中，有可能会混淆Advisor和增强。
例如，在一个代理的配置中，你可能使用了一个拦截式环绕增强、异常抛出增强和前置增强：Spring会自动创建需要的拦截链。


### 使用ProxyFactoryBean创建AOP代理

如果你的业务对象使用了Spring IoC容器（一个ApplicationContext或者BeanFactory），你应该、也会希望使用一个Spring的AOP FactoryBean。
（需要注意的是，一个FactoryBean间接的引入了一层，该层可以创建不同类型的对象。）

**说明**
> Spring 2.0 AOP在内部也是用了工厂对象。

在Spring中，创建AOP代理最基础的方式是使用`org.springframework.aop.framework.ProxyFactoryBean`类。
这样可以完全控制将要使用的切点和增强，以及它们的顺序。
然而，更简单的是这是可选的，如果你不需要这样的控制。


#### 基础
`ProxyFactoryBean`就像Spring其他`FactoryBean`的实现一样，间接的引入了一个层次。
如果你定义了一个名为`foo`的`ProxyFactoryBean`，那么对象引用的`foo`，不是`ProxyFactoryBean`实例本身，
而是`ProxyFactoryBean` 对象调用`getObject()`方法的返回值。
这个方法会创建一个AOP代理来包装目标对象。

使用一个`ProxyFactoryBean`或者IoC感知类来创建AOP代理的最大好处之一是，增强和切点同样也可以被IoC管理。
这是一个强大的功能，实现的方法是其他AOP框架难以企及的。


#### JavaBean属性

与大多数Spring所提供的`FactoryBean`实现相同的是，`ProxyFactoryBean` 本身也是一个JavaBean。
它的属性用于：
* 指定你想要代理的目标
* 指定是否需要使用CGLIB（参照下面的介绍和[基于JDK和CGLIB的代理](core.adoc#aop-pfb-proxy-types),）

一些关键的属性继承自`org.springframework.aop.framework.ProxyConfig`（Spring中所有代理工厂的超类）
这些关键属性包括：
* `proxyTargetClass`: 如果目标类将被代理标志为`true`，而不是目标类的接口。
如果该属性设置为`true`，CGLIB代理就会被创建（但也需要参见[基于JDK和CGLIB的代理](core.adoc#aop-pfb-proxy-types)）
* `optimize`: 控制是否积极优化通过`CGLIB`创建的代理类。除非完全了解AOP代理相关的优化处理，否则不要使用这个设置。
这个设置当前只对CGLIB代理有效；对JDK动态代理无效。
* `frozen`: 如果一个代理配置是`frozen`，那么就不再允许对该配置进行更改。
如果你不想调用者在代理创建后操作该代理（通过被增强的接口），作为轻微的优化手段是该配置是很有用的。
该配置的默认值是`false`，因此增加附带的advice是允许的。
* `exposeProxy`: 该属性决定当前的代理是否在暴露在`ThreadLocal`中，让它可以被目标对象访问到。
如果一个目标对象需要获取该代理，`exposeProxy`就设置为`true`，目标对象可以通过`AopContext.currentProxy()`方法获取当然的代理。
* `aopProxyFactory`: 需要使用的`AopProxyFactory`实现。提供了是否使用动态代理的自定义方式，CGLIB或者其他代理模式。
该属性的默认是适当的选择动态代理或者CGLIB。没有必要使用该属性；在Spring 1.1中它的目的在于添加新的代理类型。


`ProxyFactoryBean`的其他属性：

* `proxyInterfaces`: 接口名称的字符串数组。如果没有提供该属性，会使用一个CGLIB代理参见（[基于JDK和CGLIB的代理](core.adoc#aop-pfb-proxy-types)）

* `interceptorNames`: `Advisor`字符串数组，需要使用的拦截器或者其他advice的名称。
顺序非常重要，先到的先处理。也就是列表中的第一个拦截器将会第一个处理调用。

这些名称是当前工厂的实例名称，包括从祖先工厂继承来的名称。
这里不能包括bean的引用，因为这么做的结果是`ProxyFactoryBean`忽略advice的单例设置。

你可以在一个拦截器名称后添加一个星号( `*`)。这样在应用中，所有以型号前的部分为名称开始的advisor对象，都将被应用。
这个特性的示例可以在[使用'全局'advisor](core.adoc#aop-global-advisors)中找到。

*  singleton: 是否该工厂返回一个单例对象，无论调用多少次`getObject()`方法。
某些`FactoryBean`实现提供了这样的方法。该配置的默认值是`true`。
如果你需要使用一个有状态的advice，例如有状态的mixins，使用prototype的advice，以及将该属性设置为`false`。


#### 基于JDK和CGLIB的代理
本章作为明确的文档，介绍`ProxyFactoryBean`对于一个特定的目标对象（即被代理的对象）如何选择创建一个基于JDK的还是基于CGLIB的代理。

**说明**
> `ProxyFactoryBean`创建基于JDK或基于CGLIB的代理在Spring 1.2.x和2.0版本间有所改变。
> `ProxyFactoryBean`目前与`TransactionProxyFactoryBean`类的自动检测接口所表现的语义相似。

如果被代理的目标对象的类（以下简称目标类）没有实现任何接口，那么就会创建基于CGLIB的代理。
这是一个最简单的情景，因为JDK代理是基于接口的，没有接口就意味着JDK代理类是行不通的。
即一个简单的目标类插入，通过`interceptorNames`属性指定一系列的拦截器。
需要注意的是即使`ProxyFactoryBean`的`proxyTargetClass`属性被设置为`false`，也会创建基于CGLIB的代理。
（这显然没有任何意义，而且最好从Bean定义中移除，因为它是冗余的，而且是很糟的混淆。）

如果目标类实现了一个（或者多个）接口，那么被创建代理的类型取决于`ProxyFactoryBean`的配置。
如果`ProxyFactoryBean`的`proxyTargetClass`属性被置为`true`，那么会创建基于CGLIB的代理。
这很有道理，并且符合最小惊讶原则。
即使`ProxyFactoryBean`的`proxyInterfaces`属性被设置成一个或多个全量的接口名称，只要`proxyTargetClass`属性被置为`true`，就会创建基于CGLIB的代理。

即使`ProxyFactoryBean`的`proxyInterfaces`属性被设置成一个或多个全量的接口名称，那么就会创建基于JDK的代理。
被创建的代理会实现所有`proxyInterfaces`所指定的接口；如果目标类也实现的接口多余`proxyInterfaces`所指定的，这也是可以的，但这些额外的接口不会被创建的代理所实现。

如果`ProxyFactoryBean`的`proxyInterfaces`没有被设置，但是目标类也没有实现一个（或多个）接口，
`ProxyFactoryBean`会自动检测至少一个目标类实际实现的接口，并且创建一个基于JDK的代理。
实际上被代理的接口，就是目标类所有实现的接口；事实上，这和简单的将目标类实现的每一个接口所组成的列表设置为`proxyInterfaces`属性，效果是一样的。
然而，自动检测显然减少了工作量，也不容易出现拼写错误。


#### 代理接口
我们一起看一个简单的`ProxyFactoryBean`示例。这个例子涉及：
* 一个将被代理的目标对象。在下面的示例中定义的是"personTarget"对象。
* 一个Advisor和一个Interceptor用以提供增强。
* 一个AOP代理对象指定了目标对象（"personTarget"对象）和需要代理的接口，以及应用的advice。

```xml
<bean id="personTarget" class="com.mycompany.PersonImpl">
    <property name="name"><value>Tony</value></property>
    <property name="age"><value>51</value></property>
</bean>

<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty"><value>Custom string property value</value></property>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor">
</bean>

<bean id="person" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces"><value>com.mycompany.Person</value></property>
    <property name="target"><ref bean="personTarget"/></property>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```

需要主意的是`interceptorNames`属性使用的是一个字符串列表：当前工厂的interceptor或者advisor名称。
Advisor、拦截器、前置增强、后置增强、异常抛出增强都可以被使用。Advisor的排序很重要。

**说明**

> 你可能会疑惑，为什么列表没有持有bean的引用。
> 原因是如果一个ProxyFactoryBean的singleton属性是false，它就必须返回一个独立的代理对象。
> 如果每一个advisor对象本身是一个prototype的，就应该返回一个独立的对象，因此从工厂中获得一个prototype的实例是有必要的；持有一个引用是不行的。

上面定义的"person"对象可以被一个Person实现所替代，如下所示：

```java
Person person = (Person) factory.getBean("person");
```
在同一个IoC上下文中的其他bean，也可以强类型依赖它，作为一个原生的java对象：

```xml
<bean id="personUser" class="com.mycompany.PersonUser">
    <property name="person"><ref bean="person" /></property>
</bean>
```
本示例中的`PersonUser`类暴露了一个Person类型的属性。
就此而言，AOP代理可以透明的替代一个“真实”person的实现。
然而，它的class是一个动态代理类。它也可以被强制转换为`Advised`接口（接下来会讨论）。

可以使用内部匿名bean来隐藏目标和代理的区别。
只有`ProxyFactoryBean`的定义是不一样的；包含的advice仅仅是为了示例的完整性：


```xml
<bean id="myAdvisor" class="com.mycompany.MyAdvisor">
    <property name="someProperty"><value>Custom string property value</value></property>
</bean>

<bean id="debugInterceptor" class="org.springframework.aop.interceptor.DebugInterceptor"/>

<bean id="person" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="proxyInterfaces"><value>com.mycompany.Person</value></property>
    <!-- Use inner bean, not local reference to target -->
    <property name="target">
        <bean class="com.mycompany.PersonImpl">
            <property name="name"><value>Tony</value></property>
            <property name="age"><value>51</value></property>
        </bean>
    </property>
    <property name="interceptorNames">
        <list>
            <value>myAdvisor</value>
            <value>debugInterceptor</value>
        </list>
    </property>
</bean>
```
这样做有一个好处是只会有一个`Person`类型的对象：如果我们想要阻止用户从应用上下文中获取一个没有被advise的对象是很有用的，
或者需要阻止Spring IoC容器的自动注入时的歧义。
还有一个可以作为优点的是，ProxyFactoryBean定义是独立的。
但是，有时候从工厂中可以得到一个没有被advise的目标也是一个优点：比如在特定的测试场景。


#### 代理类
如果你需要代理一个类，而不是代理一个或多个接口？

设想一下，在上面的实例中，如果没有`Person`接口，我们需要去增强一个叫`Person`的类，该类没有实现任何业务接口。
在这种情况下，你需要配置Spring，使用CGLIB代理，而不是动态代理。
只需要将ProxyFactoryBean的`proxyTargetClass`属性置为true。
虽然最好使用接口变成，而不是类，但当增强遗留的代码时，增强目标类而不是目标接口，可能更为有用。
（通常情况下，Spring不是约定俗成的。它对应用好的实践非常简单，并且避免强制使用特定的实践方式）


如果需要，你可以在任何情况下强制使用CGLIB，甚至对于接口。

CGLIB代理的的工作原理是在运行时生成目标类的子类。
Sprig将原始目标对象的方法调用委托给该生成的子类：该子类使用了装饰器模式，在增强时织入。

CGLIB代理通常对用户是透明的。然而，有一些问题需要考虑：

* `Final`方法是不能被advise的，因为它们不能被重写。
* 从Spring 3.2之后，就不再需要在项目的classpath中加入CGLIB的库，CGLIB相关的类已经被重新打包在org.springframework包下，直接包含在prig-core 的jar包中。
这样即方便使用，又不会和其他项目所依赖的CGLIB出现版本冲突。

CGLIB代理和动态代理在性能上有所差异。
自从Spring 1.0后，动态代理稍快一些。
然而，这种差异在未来可能有所改变。
在这种情况下，性能不再是考虑的关键因素。



#### 使用全局advisor

通过在拦截器的名称上添加星号，所有匹配星号前部分的名称的advisor，都将添加到advisor链路中。
如果你需要添加一个套标准的全局advisor，这可能会派上用场。0

```xml
<bean id="proxy" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target" ref="service"/>
    <property name="interceptorNames">
        <list>
            <value>global*</value>
        </list>
    </property>
</bean>

<bean id="global_debug" class="org.springframework.aop.interceptor.DebugInterceptor"/>
<bean id="global_performance" class="org.springframework.aop.interceptor.PerformanceMonitorInterceptor"/>
```




### 简明的代理定义
特别是在定义事务代理时，最终可能有许多类似的代理定义。
使用父、子bean定义，以及内部bean定义，可能会使代理的定义更加清晰和简明。

首先，创建一个代理的父模板定义。

```xml
<bean id="txProxyTemplate" abstract="true"
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributes">
        <props>
            <prop key="*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

这个定义自身永远不会实例化，所以实际上是不完整的定义。
然后每个需要被创建的代理，只需要一个子bean的定义，将目标对象包装成一个内部类定义，因为目标对象永远不会直接被使用。

```xml
<bean id="myService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MyServiceImpl">
        </bean>
    </property>
</bean>
```


当然也可以覆盖父模板的属性，例如在本示例中，事务传播的设置：


```xml
<bean id="mySpecialService" parent="txProxyTemplate">
    <property name="target">
        <bean class="org.springframework.samples.MySpecialServiceImpl">
        </bean>
    </property>
    <property name="transactionAttributes">
        <props>
            <prop key="get*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="find*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="load*">PROPAGATION_REQUIRED,readOnly</prop>
            <prop key="store*">PROPAGATION_REQUIRED</prop>
        </props>
    </property>
</bean>
```

需要主意的是，在上面的示例中，我们通过`abstract`属性明确的将父bean标记为抽象定义，
就如前面介绍的[子bean定义](#core.adoc)，因此该父bean永远不会被实例化。
应用上下文（不是简单的bean工厂）默认会预先实例化所有单例。
因此，重要的是，如果你有一个仅仅想作为模板的bean（父bean）定义，并且指定了该bean的class，
那么你必须保证该bean的`abstract`属性被置为`tue`，否则应用上下文会尝试在实际中预先实例化该bean。



### 使用ProxyFactory以编程的方式创建AOP代理
使用Spring以编程的方式创建AOP代理非常简单。
这也运行你在不依赖Spring IoC容器的情况下使用Spring的AOP。

下面的代码展示了使用一个拦截器和一个advisor创建一个目标对象的代理。

```java
ProxyFactory factory = new ProxyFactory(myBusinessInterfaceImpl);
factory.addInterceptor(myMethodInterceptor);
factory.addAdvisor(myAdvisor);
MyBusinessInterface tb = (MyBusinessInterface) factory.getProxy();
```

第一步是构造一个`org.springframework.aop.framework.ProxyFactory`对象。
像上面的示例一样，可以使用一个目标对象创建它，或者使用指定接口集的构造函数替代来创建该ProxyFactory。

你可以添加拦截器和advisor，并在ProxyFactory的生命周期中操作它们。
如果你添加一个IntroductionInterceptionAroundAdvisor，可以使得该代理实现附加的接口集合。

在ProxyFactory也有一些好用的方法（继承自`AdvisedSupport`），允许你天机其他的增强类型，比如前置增强和异常抛出增强。
AdvisedSupport是ProxyFactory和ProxyFactoryBean的超类。

**提示**
> 在IoC框架中集成AOP代理的创建在大多数应用中是最佳实践。
> 通常，我们推荐在Java代码之外配置AOP。




### 操作被增强的对象
当你创建了AOP代理，你就能使用`org.springframework.aop.framework.Advised`接口来操作他们。
任何一个AOP代理，都能强制转换成该接口，或者无论任何该代理实现的接口。
这个接口包含以下方法：


```java
Advisor[] getAdvisors();

void addAdvice(Advice advice) throws AopConfigException;

void addAdvice(int pos, Advice advice) throws AopConfigException;

void addAdvisor(Advisor advisor) throws AopConfigException;

void addAdvisor(int pos, Advisor advisor) throws AopConfigException;

int indexOf(Advisor advisor);

boolean removeAdvisor(Advisor advisor) throws AopConfigException;

void removeAdvisor(int index) throws AopConfigException;

boolean replaceAdvisor(Advisor a, Advisor b) throws AopConfigException;

boolean isFrozen();
```

`getAdvisors()`方法会返回添加到该工厂的每一个advisor、拦截器或者其它类型的增强。
如果你添加了一个Advisor，那么返回Advisor数组在该索引下的对象，就是你添加的那个。
如果你添加的是一个拦截器或者其他类型的增强，Spring将会把它包装成一个带有切点（切点判断恒为真）的Advisor。
如果你添加了`MethodInterceptor`对象，该advisor `getAdvisors()`方法返回值，该索引处会是一个`DefaultPointcutAdvisor`对象，
该对象包括了你添加的`MethodInterceptor`对象和一个匹配所有类和方法的切点。

`addAdvisor()`可以用于添加任何Advisor。
通常该advisor是一个普通的`DefaultPointcutAdvisor`对象，包括了切点和advice，可以和任何advice或切点一起使用（除了引介增强）。

默认情况下，在一个代理被创建后，也可以添加或者删除advisor和拦截器。
唯一的限制是，不能增加或者删除一个引介advisor，因为已经从工厂生成的代理不能再进行接口修改。
（你可以从工厂中重新获取一个新的代理来避免该问题）。

一个简单的例子是强制转换一个AOP代理成为`Advised` 对象，并且检验和操作它的advice：

```java
Advised advised = (Advised) myObject;
Advisor[] advisors = advised.getAdvisors();
int oldAdvisorCount = advisors.length;
System.out.println(oldAdvisorCount + " advisors");

// Add an advice like an interceptor without a pointcut
// Will match all proxied methods
// Can use for interceptors, before, after returning or throws advice
advised.addAdvice(new DebugInterceptor());

// Add selective advice using a pointcut
advised.addAdvisor(new DefaultPointcutAdvisor(mySpecialPointcut, myAdvice));

assertEquals("Added two advisors", oldAdvisorCount + 2, advised.getAdvisors().length);
```

**说明**
> 问题是，是否建议（没有一语双关）在生产环境中对一个业务对象进行修改advice，尽管这毫无疑问是一个合法的使用案例。
> 然而，在开发环境是非常有用的：例如，在测试过程中。
> 我有时发现将一个拦截器或者advice增加到测试代码中是非常有用的，进入到一个方法中，调用我想测试的部分。
> （例如，advice可以进入到一个方法的事务中：例如运行一个SQL后检查数据库是否正确更新，在该事务标记回滚之前。）

根据你创建的代理，通常你可以设置一个`frozen`标志，在这种情况下， `Advised`的`isFrozen()`方法会返回true，
并且任何通过添加或者删除方法试图修改advice都会抛出一个`AopConfigException`异常。
在一些情况下，冻结一个advise对象的状态是有用的，例如，阻止调用代码删除安全拦截器。
在Spring 1.1也用于积极优化，当运行时的修改被认为是没必要的。


### 使用“autoproxy”能力
至此我们已经考虑过使用一个`ProxyFactoryBean`或者相似的工厂类创建明确的AOP代理。

Spring允许我们使用“autoproxy”bean定义，可以自动代理选择的bean定义。
这是建立在Spring“bean后处理器（BeanPostProcessor）”机制之上，这可以允许在容器加在后修改任何bean定义。


在这个模型上，你可以在bean定义的XML文件中设置一些特殊的bean定义，用以配置自动代理机制。
这允许你只需要声明符合代理条件的目标即可：你不需要使用 `ProxyFactoryBean`。


有两种方式实现：
* 在当前上下文中，使用一个指定了目标bean定义的自动代理创建器，
* 一些特殊自动代理创建器需要分开考虑；由源码级元数据信息驱动的自动代理创建器。



#### 自动代理Bean的定义
`org.springframework.aop.framework.autoproxy`包提供了以下标准自动代理创建器


##### BeanNameAutoProxyCreator
`BeanNameAutoProxyCreator`类是一个`BeanPostProcessor`，为纯文本或者通配符匹配出的命名为目标bean自动创建AOP代理。

```xml
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames"><value>jdk*,onlyJdk</value></property>
    <property name="interceptorNames">
        <list>
            <value>myInterceptor</value>
        </list>
    </property>
</bean>
```


和`ProxyFactoryBean`一样，有一个`interceptorNames`属性，而不是一个列表拦截器，
确保原型advisor正确的访问方式。
命名为 "interceptors",可以使任何advisor或者任何类型的advice。

和通常的自动代理一样，使用`BeanNameAutoProxyCreator`的要领是，使用最小的配置量，将相同的配置一致地应用到多个对象上。

与bean定义匹配的命名，比如上面示例中的"jdkMyBean"和"onlyJdk"，就是目标类普通的原有bean定义。
一个AOP代理会被`BeanNameAutoProxyCreator`自动创建。相同的advice会被应用到所有匹配的bean上。
需要注意的是，被应用的advisor（不是上面示例中的拦截器），对不同的bean可能使用不同的切点。


##### DefaultAdvisorAutoProxyCreator
一个更一般且更强大的自动代理创建器是`DefaultAdvisorAutoProxyCreator`。
在上下文中会自动应用符合条件的advisor，不需要在自动代理创建器的bean定义中指定目标对象的bean名称。
它也提供了相同的有点，一致的配置和避免重复定义`BeanNameAutoProxyCreator`。

使用此机制涉及：
* 指定一个`DefaultAdvisorAutoProxyCreator` bean定义。
* 在相同或者相关的上下文中指定一系列的Advisor。需要注意的是，这些都必须是Advisor，而不仅仅是拦截器或者其他的advice。
这很必要，因为这里必须有评估的切点，以便检测候选bean是否符合每一个advice。


`DefaultAdvisorAutoProxyCreator`会自动的评估包含在每一个advisor中的切点，用以确定每一个业务对象需要应用的advice（就像示例中的 "businessObject1"和"businessObject2"）。

这意味着任意数量的advisor都能自动的应用到每一个业务对象。
如果所有在advisor的切点都不能匹配一个业务对象中的任何方法，这个对象就不会被代理。
由于bean的定义都是添加给创建的业务对象。如果需要，它们都会被自动代理。

通常情况下，自动代理都有使调用方或者依赖方不能获取未被advise对象的优点。

在本ApplicationContext中调用getBean("businessObject1")会返回一个AOP代理，而不是目标业务对象（之前的“内部bean”也提供了这种优点）。


```xml
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="customAdvisor" class="com.mycompany.MyAdvisor"/>

<bean id="businessObject1" class="com.mycompany.BusinessObject1">
    <!-- Properties omitted -->
</bean>

<bean id="businessObject2" class="com.mycompany.BusinessObject2"/>
```

如果你想对许多业务对象应用相同的advice，`DefaultAdvisorAutoProxyCreator`将会有所帮助。
一旦基础定义设置完成，你就可以简单添加新的业务对象，不需要特定的proxy配置。
你也可以轻松地添加其他切面。例如，使用最小的配置修改，添加跟踪或性能监控切面。

DefaultAdvisorAutoProxyCreator提供过滤（使用命名约定，以便只有特定的advisor被评估，允许在相同的工厂中使用多个、不同的被配置的AdvisorAutoProxyCreator）和排序的支持。
Advisor可以实现`org.springframework.core.Ordered`接口，当顺序是一个问题时，确保正确的顺序。
在上面示例中使用的TransactionAttributeSourceAdvisor，有一个可配置的顺序值；默认配置是无序的。


##### AbstractAdvisorAutoProxyCreator
AbstractAdvisorAutoProxyCreator是DefaultAdvisorAutoProxyCreator的超类。
你可以通过继承这个类创建自己的自动代理创建器，
虽然这种情况微乎其微，advisor定义为`DefaultAdvisorAutoProxyCreator`框架的行为提供了有限的定制。


#### 使用元数据驱动
一个特别重要的自动代理类型就是元数据驱动。这和 .NET的`ServicedComponents`编程模型类似。
事务管理和其他企业服务的配置在源码属性中保存，而不是像在EJB中一样使用XML部署描述符。

在这种情况下，你结合能够解读元数据属性的Advisor，使用`DefaultAdvisorAutoProxyCreator`。
元数据细节存放在备选advisor的切点部分，而不是自动创建器类的本身中。


这实际上是`DefaultAdvisorAutoProxyCreator`的一种特殊情况，但值得考虑（元数据感知代码在advisor切点中，而不是AOP框架自身上）。

JPetStore示例应用程序的`/attributes`目录，展示了属性驱动的使用方法。
在这种情况下，没必要使用`TransactionProxyFactoryBean`。
简单在业务对象上定义事务属性就足够了，因为使用的是元数据感知切点。
包含了下面代码的bean定义，在 `/WEB-INF/declarativeServices.xml`文件中。
需要注意的是这是通用的，也可以在JPetStore之外使用。


```xml
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="transactionInterceptor"
        class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributeSource">
        <bean class="org.springframework.transaction.interceptor.AttributesTransactionAttributeSource">
            <property name="attributes" ref="attributes"/>
        </bean>
    </property>
</bean>

<bean id="attributes" class="org.springframework.metadata.commons.CommonsAttributes"/>
```
`DefaultAdvisorAutoProxyCreator`bean（命名不是重点，甚至可以省略）定义会获取所有在当前应用上下文中符合的切点。
在这种情况下，`TransactionAttributeSourceAdvisor`类星的"transactionAdvisor" bean定义，将适用于携带了事务属性的类或者方法。
TransactionAttributeSourceAdvisor通过构造函数依赖一个TransactionInterceptor对象。
本示例中通过自动装配解决该问题。
`AttributesTransactionAttributeSource`依赖一个`org.springframework.metadata.Attributes`接口的实现。
在本代码片段中，"attributes" bean满足这一点，使用Jakart aCommons Attributes API来获取属性信息。
（这个应用代码必须使用Commons Attributes编译任务编译）

JPetStore示例应用程序的`/annotation`目录包含了一个类似自动代理的示例，需要JDK 1.5版本以上的注解支持。
下面的配置可以自动检测Spring的`Transactional`，为包含该注解的bean配置一个隐含的代理。

```xml
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>

<bean class="org.springframework.transaction.interceptor.TransactionAttributeSourceAdvisor">
    <property name="transactionInterceptor" ref="transactionInterceptor"/>
</bean>

<bean id="transactionInterceptor"
        class="org.springframework.transaction.interceptor.TransactionInterceptor">
    <property name="transactionManager" ref="transactionManager"/>
    <property name="transactionAttributeSource">
        <bean class="org.springframework.transaction.annotation.AnnotationTransactionAttributeSource"/>
    </property>
</bean>
```

这里定义的 `TransactionInterceptor`依赖一个`PlatformTransactionManager`定义，没有被包含在这个通用文件中（尽管可以包含），
因为它对应用的事务需求是定制的（通常是JTA，就像本示例，或者是Hibernate、JDBC）：

```xml
<bean id="transactionManager"
        class="org.springframework.transaction.jta.JtaTransactionManager"/>
```

**提示**

> 如果你只需要声明式事务管理，使用这些通用的XML定义会导致Spring为所有包含事务属性的类或方法创建自动代理。
> 你不需要直接使用AOP，以及.NET和ServicedComponents相似的编程模型。

这种机制是可扩展的，可以基于通用属性自动代理。你需要：
* 定义你自己的个性化属性。
* 指定包含必要advice的Advisor，包括一个切点，该切点会被一个类或方法上存在的定义属性所触发。
你也可以使用一个已有的advice，仅仅实现了获取自定义属性的一个静态切点。

对每个被advise的类，这样的advisor都可能是唯一的（例如mixins【待定】）：
它们的bean需要被定义为prototype，而不是单例。
例如，Spring测试套件中的`LockMixin`引介拦截器，可以对一个mixin目标与一个属性驱动切点一起使用。
我们使用通用的`DefaultPointcutAdvisor`，使用JavaBean属性进行配置。

```xml
<bean id="lockMixin" class="org.springframework.aop.LockMixin"
        scope="prototype"/>

<bean id="lockableAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor"
        scope="prototype">
    <property name="pointcut" ref="myAttributeAwarePointcut"/>
    <property name="advice" ref="lockMixin"/>
</bean>

<bean id="anyBean" class="anyclass" ...
```

如果该属性感知切点匹配了`anyBean`或者其他bean定义的任何方法，这个mixin都会被应用。
需要注意的是 `lockMixin`和`lockableAdvisor`都是prototype的。
`myAttributeAwarePointcut`切点可以是一个单例定义，因为它不会持有被advise对象的个性状态。


### 使用TargetSources
Spring提供了一个`TargetSource`概念，由`org.springframework.aop.TargetSource`接口所表示。
该接口负责返回实现了连接点的目标对象。
【待定】每次AOP代理处理一个方法调用时，TargetSource`实现都需要一个目标的实例。

开发人员使用Spring AOP通常不需要直接使用TargetSource，但是它提供了一个强大的供给池、热替换和其他复杂的目标。
例如，一个池化的TargetSource可以为每次调用返回不同的目标示例，通过池子来管理这些实例。

如果你没有指定一个TargetSource，默认的实现手段是使用一个包装的本地对象。
每次调用返回的是同一个目标（如你所愿）。

让我们看一个Spring提供的标准TargetSource，以及如何使用它们。

**提示**
> 当使用一个自定义的TargetSource时，你的目标通常是一个prototype bean定义，而不是单例bean定义。
> 这允许Spring在需要时创建一个新的目标实例。



#### 热替换TargetSource
`org.springframework.aop.target.HotSwappableTargetSource`的存在，允许一个AOP代理的目标进行切换，同时允许调用者持有她们的引用。

修改TargetSource的目标对象会立即生效。`HotSwappableTargetSource`是线程安全的。

你可以如下所示，通过`HotSwappableTargetSource`的`swap()`方法修改目标对象：

```java
HotSwappableTargetSource swapper = (HotSwappableTargetSource) beanFactory.getBean("swapper");
Object oldTarget = swapper.swap(newTarget);
```

需要参考的XML定义如下所示：

```xml
<bean id="initialTarget" class="mycompany.OldTarget"/>

<bean id="swapper" class="org.springframework.aop.target.HotSwappableTargetSource">
    <constructor-arg ref="initialTarget"/>
</bean>

<bean id="swappable" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="swapper"/>
</bean>
```

上面的调用的`swap()`方法，修改了`swappable`对象的目标对象。
持有这个bean引用的客户端对此更改毫无感知，但是会立即开始命中新的目标对象。

尽管这个示例没有添加任何advice，并且添加一个advice到一个使用的`TargetSource`中也是没必要的，当然任何`TargetSource`都可以和任意的advice结合使用。


#### 池化TargetSources
使用一个池化的`TargetSource`，提供了一个与无状态会话的EJB类似的编程模型，池子中维护了相同类型的实例，当方法调用时释放池子中的对象。

Spring池子和SLSB池子的关键区别在于，Spring池子可以适用于任意POJO类。通常和Spring一样，这个服务可以用非侵入性的方式使用。


Spring提供了对框架外Commons Pool 2.2的支持，Commons Pool 2.2提供了一个相当有效的池子实现。
使用该特性，需要在应用的classpath中加入commons-pool的jar包。
也可以继承`org.springframework.aop.target.AbstractPoolingTargetSource`，来支持任意其他的池子API。


**说明**
>  Commons Pool 1.5版本以上也被支持，不过在Spring Framework 4.2被弃用了。

示例配置如下所示：

```xml
<bean id="businessObjectTarget" class="com.mycompany.MyBusinessObject" scope="prototype">
    ... properties omitted
</bean>

<bean id="poolTargetSource" class="org.springframework.aop.target.CommonsPool2TargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
    <property name="maxSize" value="25"/>
</bean>

<bean id="businessObject" class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="targetSource" ref="poolTargetSource"/>
    <property name="interceptorNames" value="myInterceptor"/>
</bean>
```
需要主意的是目标对象，即本示例中的"businessObjectTarget"必须是prototype的。
这允许`PoolingTargetSource`在需要的时候创建为目标对象创建新的实例来扩张池子大小。
参考 `AbstractPoolingTargetSource`的Javadoc文档，以及你要使用的具体的子类属性信息：
“maxSize”是最基础的，需要保证它存在

在本示例中，"myInterceptor"是一个拦截器的名称，需要在同一个IoC上下文中被定义。
然而，不需要为使用的池子，指定拦截器。
如果你只需要池子，不需要任何advice，就不要设置interceptorNames属性。

也可以通过Spring配置将任意池化的对象强制转成`org.springframework.aop.target.PoolingConfig`接口，
通过一个引介，可以显示当前池子的配置和大小信息。
你需要定义一个像这样的advisor：

```xml
<bean id="poolConfigAdvisor" class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
    <property name="targetObject" ref="poolTargetSource"/>
    <property name="targetMethod" value="getPoolingConfigMixin"/>
</bean>
```

这个advisor通过调用`AbstractPoolingTargetSource`类中的一个方法方法获取，因此使用MethodInvokingFactoryBean。
这个advisor的命名（本示例中的"poolConfigAdvisor" ）必须在ProxyFactoryBean暴露的池化对象的拦截器列表中。

强制转化如下所示：

```java
PoolingConfig conf = (PoolingConfig) beanFactory.getBean("businessObject");
System.out.println("Max pool size is " + conf.getMaxSize());
```

**说明**
> 池化无状态的服务实例通常是不必要的。
> 我们不认为这是默认选择，因为大多数的无状态对象自然是县城安全的，并且如果资源被缓存，实例池会存在问题。

简单的池子也可以使用自动代理。可以使用任何自动代理创建器设置 TargetSource。


#### Prototype类型的TargetSource
设置一个"prototype"的TargetSource和池化一个TargetSource是类似的。
在本示例中，当每个方法调用时，都会创建一个目标的示例。
尽管在现代JVM中创建一个对象的成本不高，绑定一个新对象（满足IoC的依赖）可能花费更多。
因此，没有一个很好的理由，你不应该使用这个方法，。

为此，你可以修改上面定义的 `poolTargetSource`成如下所示（为了清楚起见，我也修改了命名）：

```xml
<bean id="prototypeTargetSource" class="org.springframework.aop.target.PrototypeTargetSource">
    <property name="targetBeanName" ref="businessObjectTarget"/>
</bean>
```

只有一个属性：目标bean的命名。在TargetSource实现中使用继承是为了确保命名的一致性。
与池化TargetSource一样，目标bean的定义也必须是prototype。


#### ThreadLocal的TargetSource
如果你需要为每一个进来的请求（每个线程一个那种）创建一个对象，那么`ThreadLocal`的TargetSource将会有所帮助。
JDK范畴提供`ThreadLocal`的概念是在线程上透明存储资源的能力。
建立一个`ThreadLocalTargetSource`对其他类型的TargetSource，与该概念十分相似：

```xml
<bean id="threadlocalTargetSource" class="org.springframework.aop.target.ThreadLocalTargetSource">
    <property name="targetBeanName" value="businessObjectTarget"/>
</bean>
```

**说明**

> 当在多线程和多classload环境中，不正确的使用ThreadLocal时会出现一些问题（潜在的结果是内存泄漏）。
> 应当始终考虑将 `ThreadLocal`封装在一些类（包装类除外）中，不能直接使用 `ThreadLocal` 本身。
> 同样的，应当始终记得为线程中的资源正确的使用set和unset（后者只涉及到调用一个`ThreadLocal.set(null)`方法）。
> unset应当在任何情况都调用，因为不掉用unset可能会导致行为错误。
> Spring的ThreadLocal支持该功能，应当始终考虑赞成使用ThreadLocal，没有其他正确的处理代码【待定】。




### 定义新的Advice类型

Spring AOP被设计为可扩展的。
虽然拦截器实现策略目前是在内部使用的，但是它可以支持框架之外任意类型的advice（拦截式环绕增强、前置增强、异常抛出增强和后置增强）。

`org.springframework.aop.framework.adapter`包是一个SPI包，在不改动核心框架的情况下，支持添加新的自定义advice类型。

自定义`Advice`只有一个约束，就是必须实现`org.aopalliance.aop.Advice`标签接口。

请参阅`org.springframework.aop.framework.adapter`包的Javadoc文档，获取更多信息。


### 更多资源

有关Spring AOP的更多示例，请参阅Spring示例应用程序：
* JPetStore的默认配置，展示了将`TransactionProxyFactoryBean`应用于声明式事务管理。
* JPetStore的`/attributes`目录展示了使用属性驱动声明式事务管理。
