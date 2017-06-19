#17. 使用 O/X(Object/XML)映射器对XML进行编组

##17.1 简介
本章将讨论Spring对于 对象/XML 映射的支持。对象/XML 映射，或 O/X 映射，是指将 XML 文档与 XML 文档对象进行互相转换的操作。这一转换操作也被称作 XML 编组，或 XML 序列化。在本章中，这几个概念都指的是同一个东西。
在 O/X 映射中，将一组对象序列化为 XML 的操作是由一个编组器负责的。与之相对，一个反编组器则被用于将 XML 反序列化为一组对象。而这些操作中的 XML 文件来源可能是一份 DOM 文档，一个输入/输出流，或一个 SAX 管理器。
使用 Spring 提供的支持来实现你的 O/X 映射需求具有如下一些好处：

###17.1.1 便于配置
Spring 的 bean 工厂使得无需构建 JAXB 上下文、JiBX 绑定工厂就可以配置编组器。你可以像配置你的应用中任何一个 Spring bean 一样地配置编组器。另外，相当一部分编组器可以使用基于 XML 架构的配置来进行设置，这让配置工作变得更为容易。

###17.1.2 一致的接口
Spring 的 O/X 映射通过两个全局的接口来执行操作：Marshaller 和 Unmarshaller。这一结构让用户可以在几乎不需要修改编组操作类的前提下，轻易地在不同的 O/X 映射框架之间进行切换。这一结构的另一优势是可以以一种非侵入的方式在代码中混合多种 XML 编组方法（比如有一些编组实现使用 JAXB，而另一些则使用 Castor），从而将各种技术的优势在应用中加以综合利用。

###17.1.3 一致的异常继承
Spring 对来自底层 O/X 映射工具的异常进行了转换，以 XmlMappingException 的形式使之成为 Spring 自身异常继承体系的一部分。这些 Spring 运行时异常将初始异常封装其中，因此所有异常信息都会被完整地保留下来。

##17.2 编组器与反编组器
就如在“简介”中提到的，一个编组器负责将一个对象序列化成 XML，而一个反编组器则将 XML 流反序列化为一个对象。我们将在本节对 Spring 提供的两个相关接口进行描述。

###17.2.1 编组器
Spring 将所有编组操作抽象成了 org.springframework.oxm.Marshaller 中的方法，以下是该接口最主要的一个方法：
```
public interface Marshaller {
  /**
    * 将对象编组并存放在 Result 中.
    */
  void marshal(Object graph, Result result) throws XmlMappingException, IOException;
}
```

Marshaller 接口有一个主方法用于将一个给定对象编组为一个给定的 javax.xml.transform.Result 实现。这里的 Result 是一个用于代表某种 XML 输出格式的标记接口：不同的 Result 实现会封装不同的 XML 表现形式，详见下表：

|Result 的实现|封装的 XML 表现形式|
|--------|----|
| DOMResult | org.w3c.dom.Node |
| SAXResult| org.xml.sax.ContentHandler |
| StreamResult | java.io.File, java.io.OutputStream, or java.io.Writer |

尽管 marshal() 方法的第一个参数只是一个简单对象，但大多数 Marshaller 实现并不真的能处理任意类型的对象。要么这个对象必须在映射文件中定义过映射关系，要么被注解所修饰，要么在编组器中进行过注册，要么与编组器实现拥有共同的基类。参考后面的章节来确定你所选用的 O/X 技术实现具体是怎么做的。

###17.2.2 反编组器
与 Marshaller 接口相对应，还有一个 org.springframework.oxm.Unmarshaller 接口。
```
public interface Unmarshaller {
   /**
     *  将来源 XML 反编组成一个对象
     */
   Object unmarshal(Source source) throws XmlMappingException, IOException;
}
```
此接口同样也有一个方法，从 javax.xml.transform.Source （一个抽象的 XML 输入）读取 XML 数据，并返回一个相对应的 Java 对象。和 Result 接口一样，Source 是一个拥有三个具体实现的标记接口。每一个实现封装了一种 XML 表现形式。详见下表：

|Source 的实现|封装的 XML 表现形式|
|--------|----|
| DOMSource | org.w3c.dom.Node |
| SAXSource| org.xml.sax.InputSource, and org.xml.sax.XMLReader |
| StreamSource | java.io.File, java.io.OutputStream, or java.io.Writer |

###17.2.3 XmlMappingException
Spring 对来自底层 O/X 映射工具的异常进行了转换，以 XmlMappingException 的形式使之成为 Spring 自身异常继承体系的一部分。 这些 Spring 运行时异常将初始异常封装其中，因此所有异常信息都会被完整地保留下来。
额外地，虽然底层的 O/X 映射工具并未提供支持，但 MarshallingFailureException 和 UnmarshallingFailureException 让编组与反编组操作中产生的异常得以能够被区分开来。
The O/X Mapping exception hierarchy is shown in the following figure:

以下是 O/X 映射异常的继承层次：

![](https://i2.wp.com/docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/images/oxm-exceptions.png.pagespeed.ce.mS7MxnqbH6.png)


##17.3 Marshaller 与 Unmarshaller 的使用
Spring 的 OXM 可被用于十分广泛的场景。在以下的例子中，我们将使用这一功能将一个由 Spring 管理的应用程序的配置编组为一个 XML 文件。我们用了一个简单的 JavaBean 来表示这些配置：
```
public class Settings {
	private boolean fooEnabled;
	public boolean isFooEnabled() {
		return fooEnabled;
	}
	public void setFooEnabled(boolean fooEnabled) {
		this.fooEnabled = fooEnabled;
	}
}
```

应用程序的主类使用这个 bean 来存放应用的配置信息。除了主要方法外，主类还包含下面两个方法：saveSettings() 将配置 bean 保存成一个名为 settings.xml 的文件， loadSettings() 则将配置信息从 XML 文件中读取出来。另有一个 main() 方法负责构建 Spring 应用上下文，并调用前述两个方法。
```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import javax.xml.transform.stream.StreamResult;
import javax.xml.transform.stream.StreamSource;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.oxm.Marshaller;
import org.springframework.oxm.Unmarshaller;
public class Application {
	private static final String FILE_NAME = "settings.xml";
	private Settings settings = new Settings();
	private Marshaller marshaller;
	private Unmarshaller unmarshaller;
	
	public void setMarshaller(Marshaller marshaller) {
		this.marshaller = marshaller;
	}
	
	public void setUnmarshaller(Unmarshaller unmarshaller) {
		this.unmarshaller = unmarshaller;
	}
	
	public void saveSettings() throws IOException {
		FileOutputStream os = null;
		try {
			os = new FileOutputStream(FILE_NAME);
			this.marshaller.marshal(settings, new StreamResult(os));
		} finally {
			if (os != null) {
				os.close();
			}
		}
	}
	
	public void loadSettings() throws IOException {
		FileInputStream is = null;
		try {
			is = new FileInputStream(FILE_NAME);
			this.settings = (Settings) this.unmarshaller.unmarshal(new StreamSource(is));
		} finally {
			if (is != null) {
				is.close();
			}
		}
	}
	
	public static void main(String[] args) throws IOException {
		ApplicationContext appContext =
				new ClassPathXmlApplicationContext("applicationContext.xml");
		Application application = (Application) appContext.getBean("application");
		application.saveSettings();
		application.loadSettings();
	}
}
```

需要将 marshaller 和 unmarshaller 这两个属性赋值才能使 Application 正确运行。我们可以使用以下 applicationContext.xml 的内容来实现这一目的：
```
<beans>
	<bean id="application" class="Application">
		<property name="marshaller" ref="castorMarshaller" />
		<property name="unmarshaller" ref="castorMarshaller" />
	</bean>
	<bean id="castorMarshaller" class="org.springframework.oxm.castor.CastorMarshaller"/>
</beans>
```

该应用中使用了 Castor 这一编组器实例，但我们可以使用在本章稍后描述的任何一个编组器实例来替换 Castor。Castor 默认并不需要任何进一步的配置，所以 bean 定义十分简洁。另外由于 CastorMarshaller 同时实现了 Marshaller 与 Unmarshaller 接口，所以我们可以同时把 castorMarshaller bean 赋值给应用的 marshaller 与 unmarshaller 属性。

此范例应用将会产生如下 settings.xml 文件：
```
<?xml version="1.0" encoding="UTF-8"?>
<settings foo-enabled="false"/>
```

##17.4 基于 XML 架构的配置
可以使用来自 OXM 命名空间的 XML 标签是对编组器的配置变得更简洁。要使用这些标签，请在 XML 文件开头引用恰当的 XML 架构。以下是一个引用 oxm 的示例，请注意粗体字部分：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:oxm="http://www.springframework.org/schema/oxm" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd 
	http://www.springframework.org/schema/oxm 
	http://www.springframework.org/schema/oxm/spring-oxm.xsd">
```

目前可用的标签有以下这些：<br>
• [jaxb2-marshaller](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#oxm-jaxb2-xsd)<br>
• [jibx-marshaller](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#oxm-jibx-xsd)<br>
• [castor-marshaller](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#oxm-castor-xsd)<br>

每一个标签将会在接下来的章节中逐一介绍。下面是一个 JAXB2 的配置示例：
```
<oxm:jaxb2-marshaller id="marshaller" contextPath="org.springframework.ws.samples.airline.schema"/>
```

##17.5 JAXB
JAXB 绑定编译器将 W3C XML 架构实现为一到数个 Java 类，一个 jaxb.properties 文件，可能还会有数个资源文件。JAXB 同时还支持从被注解的 Java 类生成 XML 架构。

Spring 支持基于 17.2 Marshaller 和 Unmarshaller 所提到的 Marshaller 和 Unmarshaller 结构实现的 JAXB 2.0 API 编组策略。相应的类文件都定义在   org.springframework.oxm.jaxb 包下面。

###17.5.1 Jaxb2Marshaller
Jaxb2Marshaller 类同时实现了 Marshaller 和 Unmarshaller 接口。这个类需要上下文路径以正常运作，你可以通过 contextPath 属性来设置。上下文路径是一组由冒号（：）分隔的 Java 包名。这些包下面包含了由 XML 架构所生成的对应 Java 类。另外你可以通过设置一个叫 classesToBeBound 的属性来配置一组可以被编组器支持的类。架构的验证则通过向 bean 中配置一到多个 XML 架构的 xsd 文件资源来实现。下面是一个 bean 的配置示例：
```
<beans>
	<bean id="jaxb2Marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
		<property name="classesToBeBound">
			<list>
				<value>org.springframework.oxm.jaxb.Flight</value>
				<value>org.springframework.oxm.jaxb.Flights</value>
			</list>
		</property>
		<property name="schema" value="classpath:org/springframework/oxm/schema.xsd"/>
	</bean>
...
</beans>
```

###17.5.2 基于 XML 架构的配置
Jaxb2-marshaller 标签 配置了一个 org.springframework.oxm.jaxb.Jaxb2Marshaller 实例。以下是一个配置实例：
```
<oxm:jaxb2-marshaller id="marshaller" contextPath="org.springframework.ws.samples.airline.schema"/>
```

如果要配置需要被绑定的类，则可以使用 class-to-be-bound 子标签：
```
<oxm:jaxb2-marshaller id="marshaller">
	<oxm:class-to-be-bound name="org.springframework.ws.samples.airline.schema.Airport"/>
	<oxm:class-to-be-bound name="org.springframework.ws.samples.airline.schema.Flight"/>
	...
</oxm:jaxb2-marshaller>
```

可用的标签属性如下表：

|属性 |描述 |是否必需 |
|----|----|----|
|id|编组器的id|no|
|contextPath|JAXB上下文路径|no|

##17.6 Castor
Castor XML 映射是一个开源的 XML 绑定框架。它允许使用者将包含在一个 Java 对象模型中的数据转换成 XML 文档，或者反之。Castor 默认并不需要额外的配置就可以使用。但如果使用者希望能够对框架行为拥有更多的控制，则可以通过在配置中引入一个映射文件来达到这一目的。

读者可以从 Castor 的项目主页上了解更多相关内容。Spring 对这一框架的集成代码都在 org.springframework.oxm.castor 包底下。

###17.6.1 CastorMarshaller
与 JAXB 类似，CastorMarshaller 类同时实现了 Marshaller 和 Unmarshaller 接口。它可以通过以下配置来被引用：
```
<beans>
	<bean id="castorMarshaller" class="org.springframework.oxm.castor.CastorMarshaller" />
	...
</beans>
```

###17.6.2 映射
尽管 Castor 的默认编组行为基本能够适应大部分应用场景，有时候还是需要一些自定义的能力。这一需求可以通过使用一个 Castor 的映射文件来实现。更多信息请参考 Castor XML Mapping。

映射文件可以通过 mapppingLocation 属性进行设置，如下是一个配置了 classpath 资源的示例：
```
<beans>
	<bean id="castorMarshaller" class="org.springframework.oxm.castor.CastorMarshaller" >
		<property name="mappingLocation" value="classpath:mapping.xml" />
	</bean>
</beans>
```

###17.6.3 基于 XML 架构的配置
一个 castor-marshaller 代表了一个 org.springframework.oxm.castor.CastorMarshaller 实例。示例如下：
<oxm:castor-marshaller id="marshaller" mapping-location="classpath:org/springframework/oxm/castor/mapping.xml"/>

编组器实例可以用两种方式进行配置，一种是（通过 mapping-location 属性）指定映射文件的位置
，另一种则是（通过 target-class 或 target-package 属性）指定包含有相应 XML 描述信息的 Java POJO。后一种方式一般会与通过 XML 架构自动生成 Java 代码的功能结合使用。

以下是可用的标签属性：

|属性 |描述 |是否必需 |
|----|----|----|
|id|编组器的id|no|
|encoding|反编组 XML 文件使用的编码 |no|
|target-class|一个 Java POJO 类名，此类中包含一个（由代码自动生成器生成的） XML 类的描述信息 |no|
|target-package|一个包含了（由代码自动生成器生成的）POJO 和相应 Castor XML 描述类的 Java 包名 |no|
|mapping-location|Castor XML 映射文件的位置|no|

##17.7 JiBX
JiBX 框架提供的解决方案思路与 Hibernate 对于 ORM 的解决方案思路类似：通过一个绑定定义指定了你的 Java 对象与 XML 文件之间互相转换的规则。在准备好绑定并编译了类文件后，一个 JiBX 编译器将会对编译好的类文件进行增强，在其中加入一些辅助代码，并自动添加用于处理在类实例与 XML 文档之间相互转换的操作代码。

请参考 [JiBX官方网站](http://jibx.sourceforge.net/) 来了解更多信息。Spring 对于框架的集成代码则都在 org.springframework.oxm.jibx 包下面。

###17.7.1 JibxMarshaller
JiBXMarshaller 类同时实现了 Marshaller 和 Unmarshaller 接口。它需要使用者设置编组的目的类的类名才能正确工作。设置类名的属性是 targetClass。另外还有一个可选属性是 bingdingName，用户可以通过这个属性配置绑定名。接下来的示例中，我们将绑定 Flight 类：
```
<beans>
	<bean id="jibxFlightsMarshaller" class="org.springframework.oxm.jibx.JibxMarshaller">
		<property name="targetClass">org.springframework.oxm.jibx.Flights</property>
	</bean>
	...
</beans>
```

一个 JibxMarshaller 只能处理一个目的类与 XML 的相互转换，如果你需要编组多个类，你必需配置相应数量的 JibxMarshallers bean，然后在 targetClass 里面指定相应各个类的类名。


###17.7.2 基于 XML 的配置
jibx-marshaller 标签配置了  org.springframework.oxm.jibx.JibxMarshaller 的实例。以下是一个示例：
```
<oxm:jibx-marshaller id="marshaller" target-class="org.springframework.ws.samples.airline.schema.Flight"/>
```

标签的可用属性如下：

|属性 |描述 |是否必需 |
|----|----|----|
|id|编组器的id|no|
|target-class|此编组器所对应的目标类 |yes|
|bindingName|此编组器使用的绑定名 |no|

##17.8 XStream
Xstream 是一个用于将对象与 XML 文档进行序列化与反序列化的简单类库。它不需要任何映射关系，并且会生成整齐的 XML 文档。

请参考 [XStream](https://x-stream.github.io/) 的项目主页以获取更多信息。Spring 对此框架的集成代码都在 org.springframework.oxm.xstream 包下面。

XstreamMarshaller 类不需进行任何配置便可直接在 applicationContext.xml 文件中直接配置成 bean 进行使用。不过你可以配置一个包含了字符串别名与类之间对应关系的 别名映射表  来实现对 XML 解析结果的自定义：
```
<beans>
	<bean id="xstreamMarshaller" class="org.springframework.oxm.xstream.XStreamMarshaller">
		<property name="aliases">
			<props>
				<prop key="Flight">org.springframework.oxm.xstream.Flight</prop>
			</props>
		</property>
	</bean>
	...
</beans>
```

Xstream 默认允许对任何类进行反编组操作，但这可能会导致安全隐患。因此不建议使用 XStreamMarshaller 对来源于外部（比如公网）的 XML 文档进行反编组。如果你坚持使用 XStreamMarshaller 反编组来自外部的 XML 文档，请如下例演示的一样设置 supportedClasses 属性：
```
<bean id="xstreamMarshaller" class="org.springframework.oxm.xstream.XStreamMarshaller">
	<property name="supportedClasses" value="org.springframework.oxm.xstream.Flight"/>
	...
</bean>
```

设置这一属性能够确保只有指定的类才能够被用于反编组。

更进一步，你可以通过注册 自定义转换器 来确保只有你指定的类才能够被反编组。建议在明确指定了所有转换器后，在列表的最后加上 CatchAllConverter ，这样一来便可确保具有低优先级以及安全风险的 Xstream 默认转换器不会被调用。

这里需要注意的是 XStream 是一个 XML 序列化类库，而非一个数据绑定类库，所以它对命名空间的支持有限。这就使得 XStream 并不适合于用在网络服务中。
