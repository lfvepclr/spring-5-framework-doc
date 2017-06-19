#<center>27.JMX</center>

## 27 JMX 
## 27.1 引言
Spring对JMX的支持提供了你可以简单、透明的将Spring应用程序集成到JMX的基础架构中。

```
JMX?
        
本章不是介绍JMX的...它不会试图去解释为什么要使用JMX（或JMX实际代表什么含义）的动机。如果你是JMX的新手，请参考本章末尾的[第27.8节，更多资源](jmx.html#jmx-resources)。
```

具体来说，Spring JMX支持提供了四个核心功能：

* 任何Spring bean都会自动注册为JMX MBean
* bean管理接口的灵活控制机制
* 可以通过JSR-160连接器将声明的MBeans暴露给远程
* 远程和本地MBean资源的简单代理

这个功能的设计是应用程序组件在和Spring或JMX接口和类无需耦合的方式工作。事实上，在大多数情况下应用程序为了使用Spring JMX的特性，也不会去关心Spring或者JMX。

##27.2 将Bean暴露给JMX 

MBeanExporter是Spring JMX 框架中的核心类。它负责把Spring bean注册到JMX MBeanServer。例如，下面的例子：

```
package org.springframework.jmx;

public class JmxTestBean implements IJmxTestBean {

        private String name;
        private int age;
        private boolean isSuperman;

        public int getAge() {
                return age;
        }

        public void setAge(int age) {
                this.age = age;
        }

        public void setName(String name) {
                this.name = name;
        }

        public String getName() {
                return name;
        }

        public int add(int x, int y) {
                return x + y;
        }

        public void dontExposeMe() {
                throw new RuntimeException();
        }
}
```

为了将此bean的属性和方法作为一个属性和MBean的操作暴露出来，你只需简单的在配置文件中配置MBeanExporter类的实例并把此bean传递进去，如下：

```
<beans>
    <!-- this bean must not be lazily initialized if the exporting is to happen -->
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter" lazy-init="false">
        <property name="beans">
                <map>
                        <entry key="bean:name=testBean1" value-ref="testBean"/>
                </map>
        </property>
    </bean>
    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>
</beans>
```

上面配置段中相关的bean定义就是导出的bean。beans的属性准确的告诉MBeanExporter，哪个bean必须暴露给JMX MBeanServer。默认配置，在beans Map中的每个key都是相应的value引用的bean的对象名称。可以像27.4中“控制bean的对象名称”描述的那样来修改此行为。
在这个配置下testBean被暴露为一个名字为bean:name=testBean1的MBean。默认，bean的所有公共的属性都会被暴露为属性并且所有的公共方法（那些从Object类继承的）都被会暴露为操作。

>MBeanExporter是一个有生命周期bean（参考“启动和关闭回调”）并且MBeans会在应用程序默认的生命周期中尽可能迟的被暴露。可以通过在暴露的阶段配置或者通过设置自动启动标志位来禁止自动注册。

### 27.2.1 创建MBeanServer 

假设上述的配置是在一个正在运行的应用环境中，并且他有一个(只有一个) MBeanServer已经在运行了。在这种情形下，Spring将会尝试定位正在运行的MBeanServer并把你的bean注册到它上面去。当你的应用是运行在一个诸如Tomcat、IBM WebSphere等有自己MBeanServer的容器内时，这个就非常有用。

然而这种方法在独立的环境或运行在一个没有提供MBeanServer的容器中是没用的。为了解决这个问题，你可以通过添加一个org.springframework.jmx.support.MBeanServerFactoryBean的类实例到你的配置中来创建一个MBeanServer实例。你还可以通过将MBeanServerFactoryBeanMBean返回的MBeanServer值赋给MBeanExporter的server属性使用指定的MBeanServer，例如：

```
<beans>

    <bean id="mbeanServer" class="org.springframework.jmx.support.MBeanServerFactoryBean"/>

    <!--
    this bean needs to be eagerly pre-instantiated in order for the exporting to occur;
    this means that it must not be marked as lazily initialized
    -->
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
            <property name="beans">
                    <map>
                            <entry key="bean:name=testBean1" value-ref="testBean"/>
                    </map>
            </property>
            <property name="server" ref="mbeanServer"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
            <property name="name" value="TEST"/>
            <property name="age" value="100"/>
    </bean>

</beans>
```

这是通过MBeanServerFactoryBean创建的一个MBeanServer实例，并通过server属性把它传递给MBeanExporter。当你自己提供MBeanServer实例时，MBeanExporter不在尝试寻找正在运行的MBeanServer，将直接使用提供的MBeanServer实例。为了让它正常运行，你必须（当然）在你的类路径下有一个JMX的实现。

### 27.2.2重用存在的MBeanServer 
如果没有指明的server，那么MBeanExporter会尝试去自动探测一个正在运行的MBeanServer。这在大多数情况下只有一个MBeanServer实例的情况下正常运行，但是当有多个实例存在时，exporter可能会选择一个错误的server。在这种情况下，应该要指明你要用的MBeanServer agentId：

```
<beans>
    <bean id="mbeanServer" class="org.springframework.jmx.support.MBeanServerFactoryBean">
        <!-- indicate to first look for a server -->
        <property name="locateExistingServerIfPossible" value="true"/>
        <!-- search for the MBeanServer instance with the given agentId -->
        <property name="agentId" value="MBeanServer_instance_agentId>"/>
    </bean>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="server" ref="mbeanServer"/>
        ...
    </bean>
</beans>
```

对于平台化或一些情形来说，应该使用factory-method来查找动态变化（未知）的MBeanServer的agentId：

```
<beans>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="server">
                <!-- Custom MBeanServerLocator -->
                <bean class="platform.package.MBeanServerLocator" factory-method="locateMBeanServer"/>
        </property>
    </bean>

    <!-- other beans here -->

</beans>
```

### 27.2.3MBeans懒加载
如果你用MBeanExporter配置了一个bean，那么它也配置了延迟初始化，MBeanExporter在不破坏他们关联性的情况下会避免bean的实例化。相反，它会注册一个MBeanServer的代理，直到第一次通过代理向容器获取bean的时候才会初始化。

### 27.2.4MBeans自动注册
任何通过MBeanExporter暴露的bean都是一个有效的MBean，它和MBeanServer注一样，不需要Spring的介入。可以将MBeanExporter的autodetect属性设置为true，MBeans就可以自动被发现：

```
<bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="autodetect" value="true"/>
</bean>

<bean name="spring:mbean=true" class="org.springframework.jmx.export.TestDynamicMBean"/>
```

所有的被称为spring:mbean=true都是有效的JMX MBean，它会被Spring自动注册。默认，bean的名字会被用作ObjectName，JMX注册时自动被发现。这个动作可以被重写，详情参考27.4，“控制bean的ObjectNames”。

### 27.2.5控制注册的动作
考虑Spring MBeanExporter试图使用'bean:name=testBean1'作为ObjectName将MBean注册到MBeanServer的场景。如果一个MBean的实例已经注册了相同的名字，默认情况下这此注册行为将失败(抛出InstanceAlreadyExistsException异常)。
可以严格控制MBean在注册到MBeanServer上何时发生了什么的行为。Spring的JMX支持三种不同的注册行为来控制在注册过程中发现MBean有相同的ObjectName；这些注册方法总结如下：

* 表27.1 注册方法

|注册方法|解释|
|--------|----|
| REGISTRATION_FAIL_ON_EXISTING | 这是默认的注册方法。如果一个MBean实例已经被注册了相同的ObjectName，这个MBean不能注册，并且抛出InstanceAlreadyExistsException异常。已经存在的MBean不受影响。|
| REGISTRATION_IGNORE_EXISTING| 如果一个MBean实例已经被注册了相同的ObjectName，这个MBean不能注册。已经存在的MBean不受影响，也没有异常抛出。这个设置在多个应用之间共享MBeanServer，共享MBean时非常有用。|
| REGISTRATION_REPLACE_EXISTING | 如果一个MBean实例已经被注册了相同的ObjectName，之前存在的MBean将被没有注册的新MBean原地替换(新的MBean替换前一个)。|

上面的值是定义在MBeanRegistrationSupport类中的常量（它是MBeanExporter的父类）。|如果你想改变默认的注册行为，你只需要简单的在MBeanExporter的registrationBehaviorName属性上设置上面定义的值即可。

下面的示例说明了怎样用REGISTRATION_REPLACE_EXISTING改变默认的注册行为：

```
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
                <map>
                        <entry key="bean:name=testBean1" value-ref="testBean"/>
                </map>
        </property>
        <property name="registrationBehaviorName" value="REGISTRATION_REPLACE_EXISTING"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>

```

### 27.3 bean的控制管理接口
在前面的例子中，每个已经被暴露为JMX属性和操作的bean的所有public属性和方法，你都可以通过bean的管理接口来控制。你可以精确的控制你所暴露的bean上的哪个属性和方法作为JMX的属性和操作，Spring JMX提供了全面的、可扩展的机制来控制bean的管理接口。

### 27.3.1 MBeanInfoAssembler接口
底层实现上，MBeanExporter委托了一个org.springframework.jmx.export.assembler.MBeanInfoAssembler接口的实现来负责在每个已经被暴露的bean上定义管理接口。默认实现是org.springframework.jmx.export.assembler.SimpleReflectiveMBeanInfoAssembler，对所有public的属性和方法（如你在前面例子中看到的）都会简单的定义一个管理接口。Spring对MBeanInfoAssembler接口提供了两种额外的实现，它允许使用源码级别的元数据或任意接口来控制管理接口的生成。

### 27.3.2 源码级别的元数据（Java注解）
使用MetadataMBeanInfoAssembler你可以对bean在源码级别上定义管理接口。org.springframework.jmx.export.metadata.JmxAttributeSource接口封装了元数据的读取。Spring JMX提供了使用Java注解提供了一个名为org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource的默认实现。为了MetadataMBeanInfoAssembler必须配置一个实现了JmxAttributeSource接口的实例才能正常工作（没有默认）。

为了将一个bean标记为JMX的bean，你应该使用ManagedResource类级别的注解。每个你想要暴露为操作的方法都必须用ManagedOperation注解标记，每个想暴露的属性的都必须用ManagedAttribute注解标记。当标记属性的时候你可以忽略getter或setter然后自己创建一个只读或只写的属性。

>注意
>一个被ManagedResource注解的bean，它暴露的操作或属性必须是public的。

下面的例子展示了用注解实现的JmxTestBean类：

```
package org.springframework.jmx;

import org.springframework.jmx.export.annotation.ManagedResource;
import org.springframework.jmx.export.annotation.ManagedOperation;
import org.springframework.jmx.export.annotation.ManagedAttribute;

@ManagedResource(
                objectName="bean:name=testBean4",
                description="My Managed Bean",
                log=true,
                logFile="jmx.log",
                currencyTimeLimit=15,
                persistPolicy="OnUpdate",
                persistPeriod=200,
                persistLocation="foo",
                persistName="bar")
public class AnnotationTestBean implements IJmxTestBean {

        private String name;
        private int age;

        @ManagedAttribute(description="The Age Attribute", currencyTimeLimit=15)
        public int getAge() {
                return age;
        }

        public void setAge(int age) {
                this.age = age;
        }

        @ManagedAttribute(description="The Name Attribute",
                        currencyTimeLimit=20,
                        defaultValue="bar",
                        persistPolicy="OnUpdate")
        public void setName(String name) {
                this.name = name;
        }

        @ManagedAttribute(defaultValue="foo", persistPeriod=300)
        public String getName() {
                return name;
        }

        @ManagedOperation(description="Add two numbers")
        @ManagedOperationParameters({
                @ManagedOperationParameter(name = "x", description = "The first number"),
                @ManagedOperationParameter(name = "y", description = "The second number")})
        public int add(int x, int y) {
                return x + y;
        }

        public void dontExposeMe() {
                throw new RuntimeException();
        }

}
```

可以看到JmxTestBean类被配置了一系列属性的ManagedResource注解标记。这些属性可以通过MBeanExporter产生配置了各种切面的MBean，详情请参考后面的27.3.3节，”源码级别的元数据类型“。

你也注意到了age和name属性都用了ManagedAttribute注解，但是在age属性上，只有getter上使用了。这会使得所有的属性在management接口中都作为属性，只有age属性是只读的。

最终，你会注意到，add(int, int)方法被标记为ManagedOperation属性，而dontExposeMe()这个方法却不是。当使用MetadataMBeanInfoAssembler时，管理接口只包含一个add（int，int）操作。

下面的配置展示了在使用MetadataMBeanInfoAssembler时如何配置MBeanExporter：

```
<beans>
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
            <property name="assembler" ref="assembler"/>
            <property name="namingStrategy" ref="namingStrategy"/>
            <property name="autodetect" value="true"/>
    </bean>

    <bean id="jmxAttributeSource"
                    class="org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource"/>

    <!-- will create management interface using annotation metadata -->
    <bean id="assembler"
                    class="org.springframework.jmx.export.assembler.MetadataMBeanInfoAssembler">
        <property name="attributeSource" ref="jmxAttributeSource"/>
    </bean>

    <!-- will pick up the ObjectName from the annotation -->
    <bean id="namingStrategy"
                    class="org.springframework.jmx.export.naming.MetadataNamingStrategy">
        <property name="attributeSource" ref="jmxAttributeSource"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.AnnotationTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>
</beans>
```

这里你将看到MetadataMBeanInfoAssembler被配置为AnnotationJmxAttributeSource的一个实例，并且通过assembler属性传递给MBeanExporter。这对于利用Spring暴露的MBean元数据驱动的管理接口来说是必须的。

### 27.3.3 源码级别的元数据类型 
使用Spring JMX有以下几种源码级别的元数据类型：

 * 表 27.2. 源码级别的元数据类型

| 目标 | 注解 | 注解类型 | 
| -----| -----| ------- | 
| 把所有类的实例作为JMX管理的资源 |@ManagedResource | 类 |
| 将一个方法作为JMX的操作 | @ManagedOperation | 方法 |
| 将getter或setter标识为部分的JMX属性 | @ManagedAttribute | 方法（只限getter或setter）|
| 对一个操作参数定义描述符 | @ManagedOperationParameter和@ManagedOperationParameters | 方法 |

使用这些源码级别的元数据类型有以下的配置参数：

| 参数 | 描述 | 应用 |
| -----| -----| ------- | 
| ObjectName | 使用元数据命名策略决定管理资源的ObjectName | ManagedResource |
| description | 设置友好的资源、属性和操作的描述 | ManagedResource, ManagedAttribute, ManagedOperation, ManagedOperationParameter |
| currencyTimeLimit | 设置currencyTimeLimit描述字段值 | ManagedResource, ManagedAttribute | 
| defaultValue | 设置defaultValue描述字段值 | ManagedAttribute | 
| log | 设置log描述字段值 | ManagedResource |
| logFile | 设置logFile描述字段值 | ManagedResource |
| persistPolicy | 置persistPolicy描述字段值 | ManagedResource |
| persistPolicy | 设置persistPolicy描述字段值 | ManagedResource |
| persistPeriod | 设置persistPeriod描述字段值 | ManagedResource |
| persistLocation |设置persistLocation描述字段值 | ManagedResource |
| persistName | 设置persistName描述字段值 | ManagedResource | 
| name | 设置操作参数展示的名字 | ManagedOperationParameter |
| index | 设置操作参数的索引 | ManagedOperationParameter |

### 27.3.4 AutodetectCapableMBeanInfoAssembler接口
为了进一步简化配置，Spring引入了继承了MBeanInfoAssembler接口来支持MBean资源的自动发现的AutodetectCapableMBeanInfoAssembler接口。如果你用AutodetectCapableMBeanInfoAssembler的实例配置了MBeanExporter，那么你可以对暴露给JMX的bean进行“投票”。

AutodetectCapableMBeanInfo的唯一实现是MetadataMBeanInfoAssembler，它将对被标记为ManagedResource的属性进行投票。默认使用bean名字作为ObjectName的配置为：

```
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <!-- notice how no 'beans' are explicitly configured here -->
        <property name="autodetect" value="true"/>
        <property name="assembler" ref="assembler"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

    <bean id="assembler" class="org.springframework.jmx.export.assembler.MetadataMBeanInfoAssembler">
        <property name="attributeSource">
                <bean class="org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource"/>
        </property>
    </bean>

</beans>
```

注意，在这个配置中没有bean被传递给MBeanExporter；然而，JmxTestBean在被标记为ManagedResource属性，而且MetadataMBeanInfoAssembler会发现JmxTestBean并且投票把它包含进来时就被注册了。这种方法唯一问题是JmxTestBean的名字现在有业务含义。你可以通过更改ObjectName创建的默认行为来解决此问题，如27.4，“控制bean的ObjectName”。

### 27.3.5使用Java接口定义管理接口 

除了MetadataMBeanInfoAssembler，Spring也提供了MetadataMBeanInfoAssembler，Spring也包含了InterfaceBasedMBeanInfoAssembler，它允许约束方法和用一些基于集合接口上定义的方法所暴露出来的属性。
虽然标准的暴露机制是使用接口和一个简单命名方案，InterfaceBasedMBeanInfoAssembler通过去除命名约定来继承这个功能，允许你使用多个的接口，并且不需要bean实现MBean接口。
考虑你先前用来对JmxTestBean定义管理接口的接口：

```
public interface IJmxTestBean {

        public int add(int x, int y);

        public long myOperation();

        public int getAge();

        public void setAge(int age);

        public void setName(String name);

        public String getName();

}
```
这个接口定义的方法和属性将会在JMX的MBean上暴露为操作和属性。下面的代码展示了如何用管理接口定义的接口来配置Spring JMX：

```
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                    <entry key="bean:name=testBean5" value-ref="testBean"/>
            </map>
        </property>
        <property name="assembler">
            <bean class="org.springframework.jmx.export.assembler.InterfaceBasedMBeanInfoAssembler">
                <property name="managedInterfaces">
                    <value>org.springframework.jmx.IJmxTestBean</value>
                </property>
            </bean>
        </property>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
```
这里，你可以看到当对任何bean构建管理接口时会使用IJmxTestBean配置InterfaceBasedMBeanInfoAssembler。通过InterfaceBasedMBeanInfoAssembler处理的bean是不需要实现用来产生JMX管理接口的接口，明白这点很重要。
上面的例子中，所有bean的管理接口都是使用IJmxTestBean接口来构建。在许多情况下，不同的bean会使用不同的接口，而不是上面的单一行为。此例中，你可以通过interfaceMappings属性，来传递InterfaceBasedMBeanInfoAssembler的实例，这样key就是bean名字，value就是在这个bean上的用逗号分隔的方法列表。
如果即没有通过managedInterfaces或interfaceMappings属性指明一个管理接口，那么InterfaceBasedMBeanInfoAssembler会通过反射在使用所有实现了该bean的接口来创建一个管理接口。
###27.3.6 使用MethodNameBasedMBeanInfoAssembler
MethodNameBasedMBeanInfoAssembler允许你指定一个方法名列表给要暴露给JMX的属性和操作。配置如下代码：

```
<bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
    <property name="beans">
        <map>
            <entry key="bean:name=testBean5" value-ref="testBean"/>
        </map>
    </property>
    <property name="assembler">
        <bean class="org.springframework.jmx.export.assembler.MethodNameBasedMBeanInfoAssembler">
            <property name="managedMethods">
                <value>add,myOperation,getName,setName,getAge</value>
            </property>
        </bean>
    </property>
</bean>
```

上面实例你可以看到，增加了暴露给JMX操作的方法myOperation和getName、setName(String) 和 getAge()等JMX的属性。在上面的代码中，暴露给JMX的方法都和bean有映射的关系。为了控制一个bean一个bean的暴露，使用MethodNameMBeanInfoAssembler的methodMappings属性来映射bean的名字到方法名字列表上。
## 27.4控制bean的ObjectNames
在底层，MBeanExporter委托了一个ObjectNamingStrategy的实现来获取每个bean在注册时的ObjectName。默认实现为KeyNamingStrategy，ObjectName作为为bean Map的key。而且，KeyNamingStrategy可以将bean Map的key映射到一个属性文件来解析ObjectName。除此KeyNamingStrategy之外，Spring还提供了两个额外的ObjectNamingStrategy实现：一个基于bean的JVM标识来构建ObjectName的IdentityNamingStrategy和使用代码级元数据来获取ObjectName的MetadataNamingStrategy。

### 27.4.4 从属性中读取ObjectName
你可以配置自己的KeyNamingStrategy实例，并从一个配置的属性实例中读取ObjectName，而不是bean 的key。KeyNamingStrategy将尝试使用bean对应的key来找出它在Properties中的入口。如果没有找到入口或Properties实例为空，那么bean将使用它自己的key。
下面的代码展示了KeyNamingStrategy的一个简单配置：

```
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="testBean" value-ref="testBean"/>
            </map>
        </property>
        <property name="namingStrategy" ref="namingStrategy"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

    <bean id="namingStrategy" class="org.springframework.jmx.export.naming.KeyNamingStrategy">
        <property name="mappings">
            <props>
                <prop key="testBean">bean:name=testBean1</prop>
            </props>
        </property>
        <property name="mappingLocations">
            <value>names1.properties,names2.properties</value>
        </property>
    </bean>

</beans>
```
这里KeyNamingStrategy实例是用一个Properties实例来配置的，Properties是由mapping属性定义的Properties实例和位于mapping属性定义的目录下的属性文件合并而成。在这个配置中，testBean的ObjectName由bean:name=testBean1定义，因此它是bean的key对应的属性实例的入口。
如果Properties实例中找不到入口，那么bean的key将是它的ObjectName。

### 27.4.2 使用MetadataNamingStrategy
MetadataNamingStrategy使用每个bean上ManagedResource属性的objectName属性来创建ObjectName。下面代码展示了MetadataNamingStrategy的配置：

```
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="testBean" value-ref="testBean"/>
            </map>
        </property>
        <property name="namingStrategy" ref="namingStrategy"/>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

    <bean id="namingStrategy" class="org.springframework.jmx.export.naming.MetadataNamingStrategy">
        <property name="attributeSource" ref="attributeSource"/>
    </bean>

    <bean id="attributeSource"
                        class="org.springframework.jmx.export.annotation.AnnotationJmxAttributeSource"/>

</beans>
```

如果ManagedResource属性没有提供objectName，那么将会用下面的格式创建ObjectName：[fully-qualified-package-name]:type=[short-classname],name=[bean-name]。例如，下面的bean产生的ObjectName为：com.foo:type=MyClass,name=myBean。

```
<bean id="myBean" class="com.foo.MyClass"/>
```
### 27.4.3基于MBean导出的注解配置
如果你更喜欢使用基于注解的方式来定义management接口，则可以很方便的使用MBeanExporter的一个子类：AnnotationMBeanExporter。当定义一个子类的实例时，将不需要再配置namingStrategy，assembler和attributeSource，因为它会使用标注的基于Java 元数据的注解（自动检测一直开启）。实际上，不是定义一个MBeanExporter，@EnableMBeanExport和@Configuration注解支持更简单的语法。

```
@Configuration@EnableMBeanExport
public class AppConfig {

}
```
如果你更喜欢XML的配置方式，'context:mbean-export'可以达到相同的目的。

```
<context:mbean-export/>
```
如果需要，你可以对特定的MBean 服务提供一个引用，并且defaultDomain（AnnotationMBeanExporter的属性）属性接受生成“ObjectNames”的备用值。如上节的MetadataNamingStrategy所述，它将用于代替完全限定的包名称。

```
@EnableMBeanExport(server="myMBeanServer", defaultDomain="myDomain")
@Configuration
ContextConfiguration {

}
```
```
<context:mbean-export server="myMBeanServer" default-domain="myDomain"/>
```

>不要使用基于接口的AOP代理和JMX注解自动发现相结合的方式。基于接口的代理对隐藏目标类，它也会隐藏JMX管理的resource注解。因此，在这种情况下使用目标类代理：通过在<aop:config/>, <tx:annotation-driven/>等上设置'proxy-target-class'标志，否则，JMX也可能在启动时被忽略。

## 27.5 JSR-160连接器

对于远程访问，Spring JMX模块在org.springframework.jmx.support package包中提供了两种FactoryBean实现，用于创建服务端和客户端的连接器。

### 27.5.1 服务端连接器

为了使用Spring JMX 来创建，需要使用以下配置启动并暴露JSR-160 JMXConnectorServer：

```
<bean id="serverConnector" class="org.springframework.jmx.support.ConnectorServerFactoryBean"/>
```
ConnectorServerFactoryBean创建的JMXConnectorServer默认会绑定到"service:jmx:jmxmp://localhost:9875".serverConnector通过JMXMP协议在本地的9875端口上将本地的MBeanServer暴露给客户端。注意，JMXMP协议是JSR 160标记为可选协议：当前 ，JMX主要的开源实现MX4J，它只提供了基于JDK的协议，而不支持JMXMP。

分别使用serviceUrl和ObjectName属性来指明另一个URL并把JMXConnectorServer自身注册为一个MBeanServer：

```
<bean id="serverConnector"
        class="org.springframework.jmx.support.ConnectorServerFactoryBean">
    <property name="objectName" value="connector:name=rmi"/>
    <property name="serviceUrl"
            value="service:jmx:rmi://localhost/jndi/rmi://localhost:1099/myconnector"/>
</bean>
```

如果设置了ObjectName属性，Spring将自动使用在ObjectName底下使用MBeanServer注册连接器。下面的例子展示了当你创建一个JMXConnector时，你可以传递完整的参数给ConnectorServerFactoryBean。

```
<bean id="serverConnector"
        class="org.springframework.jmx.support.ConnectorServerFactoryBean">
    <property name="objectName" value="connector:name=iiop"/>
    <property name="serviceUrl"
        value="service:jmx:iiop://localhost/jndi/iiop://localhost:900/myconnector"/>
    <property name="threaded" value="true"/>
    <property name="daemon" value="true"/>
    <property name="environment">
        <map>
            <entry key="someKey" value="someValue"/>
        </map>
    </property>
</bean>
```

注意，当使用基于RMI连接器时，需要启动查找服务（tnameserv or rmiregistry）来完成名称注册。如果你使用Spring 通过RMI来导出远程服务，那么Spring已经构建了一个RMI注册。如果没有，你可以通过下面的配置简单的启动一个注册：

```
<bean id="registry" class="org.springframework.remoting.rmi.RmiRegistryFactoryBean">
    <property name="port" value="1099"/>
</bean>
```

### 27.5.2 客户端连接器

下面展示了使用MBeanServerConnectionFactoryBean创建远程JSR-160 MBeanServer 的MBeanServerConnection：

```
<bean id="clientConnector" class="org.springframework.jmx.support.MBeanServerConnectionFactoryBean">
    <property name="serviceUrl" value="service:jmx:rmi://localhost/jndi/rmi://localhost:1099/jmxrmi"/>
</bean>
```

### 27.5.3 通过Hessian 或 SOAP 的JMX

JSR-160允许客户端和服务端之间进行通讯的方式进行扩展。上面的例子使用JSR-160规范（IIOP和JRMP）和（可选）JMXMP所需的强制基于RMI的实现。通过使用其他的提供者或JMX的实现（例如MX4J）你可以通过简单的HTTP或SSL或其他方式利用诸如SOAP或Hessian之类的协议：

```
<bean id="serverConnector" class="org.springframework.jmx.support.ConnectorServerFactoryBean">
    <property name="objectName" value="connector:name=burlap"/>
    <property name="serviceUrl" value="service:jmx:burlap://localhost:9874"/>
</bean>
```

上面的例子使用了 MX4J 3.0.0，关于MX4J请参考官方文档.

## 27.6 通过代理访问MBeans

Spring JMX 允许你创建代理，它将重新路由到本地或者远程MBeanServer中注册的MBean。这些代理提供了标准的Java接口来和MBean进行交互。下面的代码展示了如何在本地允许的MBeanServer中配置代理：

```
<bean id="proxy" class="org.springframework.jmx.access.MBeanProxyFactoryBean">
    <property name="objectName" value="bean:name=testBean"/>
    <property name="proxyInterface" value="org.springframework.jmx.IJmxTestBean"/>
</bean>
```

你可以看到在ObjectName: bean:name=testBean下注册的MBean创建代理。通过proxyInterfaces属性来控制代理实现的一系列接口，InterfaceBasedMBeanInfoAssembler使用和MBean上属性相同规则来操作这些接口上的方法映射规则和属性。

MBeanProxyFactoryBean可以通过MBeanServerConnection为任何可以访问的MBean创建一个代理。默认，使用本地的MBeanServer，但是你可以重写它，并提供一个指向远程MBeanServer的MBeanServerConnection来满足远程MBean的代理。

```
<bean id="clientConnector"
        class="org.springframework.jmx.support.MBeanServerConnectionFactoryBean">
    <property name="serviceUrl" value="service:jmx:rmi://remotehost:9875"/>
</bean>

<bean id="proxy" class="org.springframework.jmx.access.MBeanProxyFactoryBean">
    <property name="objectName" value="bean:name=testBean"/>
    <property name="proxyInterface" value="org.springframework.jmx.IJmxTestBean"/>
    <property name="server" ref="clientConnector"/>
</bean>
```
你可以看到，我们使用MBeanServerConnectionFactoryBean创建了一个指向远程机器的MBeanServerConnection。MBeanServerConnection通过server属性传递给MBeanProxyFactoryBean。创建的代理通过MBeanServerConnection将所有的调用都转发到MBeanServer。

## 27.7 通知

Spring JMX提供了对JMX通知的全面支持。

### 27.7.1 注册通知监听器

Spring JMX的支持使得将对任意数量的NotificationListeners注册到任意数量的MBean（包括通过Spring MBeanExporter 导出的MBean和通过其他机制注册的MBean）。示例，考虑这么一个场景，当目标MBean的每个属性每次改变的时候都会通知（通过通知）。

```
package com.example;

import javax.management.AttributeChangeNotification;
import javax.management.Notification;
import javax.management.NotificationFilter;
import javax.management.NotificationListener;

public class ConsoleLoggingNotificationListener
        implements NotificationListener, NotificationFilter {

    public void handleNotification(Notification notification, Object handback) {
        System.out.println(notification);
        System.out.println(handback);
    }

    public boolean isNotificationEnabled(Notification notification) {
        return AttributeChangeNotification.class.isAssignableFrom(notification.getClass());
    }

}
```

```
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
        <property name="notificationListenerMappings">
            <map>
                <entry key="bean:name=testBean1">
                    <bean class="com.example.ConsoleLoggingNotificationListener"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
```

通过上面的配置，目标MBean（bean:name=testBean1）每次以广播形式发送JMX通知，通过notificationListenerMappings属性注册为ConsoleLoggingNotificationListener的监听器将被通知。 ConsoleLoggingNotificationListener可以采取任何它认为合适的动作来响应通知。
 
你可以直接使用bean的名称作为导出bena的监听器之间的连接：
 
 ```
 <beans>
 
    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
        <property name="notificationListenerMappings">
            <map>
                <entry key="testBean">
                    <bean class="com.example.ConsoleLoggingNotificationListener"/>
                </entry>
            </map>
        </property>
    </bean>
 
    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>
 
 </beans>
 ```
 
如果你想对封闭的MBeanExporter导出的所有bean注册一个NotificationListener实例，可以使用特殊通配符‘*’（无引号）作为notificationListenerMappings属性map中的key；例如：

```
<property name="notificationListenerMappings">
    <map>
        <entry key="*">
            <bean class="com.example.ConsoleLoggingNotificationListener"/>
        </entry>
    </map>
</property>
```

如果需要执行反转（针对MBean注册一些不同的监听器），那么必须使用notificationListeners的属性列表（不是notificationListenerMappings属性）。这次，不是简单配置单个MBean的NotificationListener，而是配置NotificationListenerBean实例，NotificationListenerBean在MBeanServer中封装了NotificationListener和ObjectName（或ObjectNames）。NotificationListenerBean也封装了其他属性，例如NotificationFilter和用于高级JMX通知场景的任意handback对象。

使用NotificationListenerBean实例时和前面的不同配置：

```
<beans>

    <bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
        <property name="beans">
            <map>
                <entry key="bean:name=testBean1" value-ref="testBean"/>
            </map>
        </property>
        <property name="notificationListeners">
            <list>
                <bean class="org.springframework.jmx.export.NotificationListenerBean">
                    <constructor-arg>
                        <bean class="com.example.ConsoleLoggingNotificationListener"/>
                    </constructor-arg>
                    <property name="mappedObjectNames">
                        <list>
                            <value>bean:name=testBean1</value>
                        </list>
                    </property>
                </bean>
            </list>
        </property>
    </bean>

    <bean id="testBean" class="org.springframework.jmx.JmxTestBean">
        <property name="name" value="TEST"/>
        <property name="age" value="100"/>
    </bean>

</beans>
```

上面的例子和第一个例子是等价的。现在假设我们想每发出一个通知就给出一个handback对象，除此之外我们还想通过一个NotificationFilter过滤外来的通知。（关于什么是handback对象，什么是NotificationFilter，请参考JMX规范（1.2）的“JMX通知模型”）。

```
<beans>

	<bean id="exporter" class="org.springframework.jmx.export.MBeanExporter">
		<property name="beans">
			<map>
				<entry key="bean:name=testBean1" value-ref="testBean1"/>
				<entry key="bean:name=testBean2" value-ref="testBean2"/>
			</map>
		</property>
		<property name="notificationListeners">
			<list>
				<bean class="org.springframework.jmx.export.NotificationListenerBean">
					<constructor-arg ref="customerNotificationListener"/>
					<property name="mappedObjectNames">
						<list>
							<!-- handles notifications from two distinct MBeans -->
							<value>bean:name=testBean1</value>
							<value>bean:name=testBean2</value>
						</list>
					</property>
					<property name="handback">
						<bean class="java.lang.String">
							<constructor-arg value="This could be anything..."/>
						</bean>
					</property>
					<property name="notificationFilter" ref="customerNotificationListener"/>
				</bean>
			</list>
		</property>
	</bean>

	<!-- implements both the NotificationListener and NotificationFilter interfaces -->
	<bean id="customerNotificationListener" class="com.example.ConsoleLoggingNotificationListener"/>

	<bean id="testBean1" class="org.springframework.jmx.JmxTestBean">
		<property name="name" value="TEST"/>
		<property name="age" value="100"/>
	</bean>

	<bean id="testBean2" class="org.springframework.jmx.JmxTestBean">
		<property name="name" value="ANOTHER TEST"/>
		<property name="age" value="200"/>
	</bean>

</beans>

```

### 27.7.2 发布通知
Spring不仅提供了对注册接受通知的支持，而且还用于发布通知。

>请注意，本节仅与通过MBeanExporter暴露的Spring管理的MBean相关。任何现有的用户定义的MBean都应该使用标准的JMX API来发布通知。

Spring JMX支持的通知发布的关键接口为NotificationPublisher（定义在org.springframework.jmx.export.notification包下面）。任何通过MBeanExporter实例导出为MBean的bean都可以实现NotificationPublisherAware的相关接口来获取NotificationPublisher实例。NotificationPublisherAware接口通过一个简单的setter方法将NotificationPublisher的实例提供给实现bean，这个bean就可以用来发布通知。

如javadoc中的NotificationPublisher类所述，通过NotificationPublisher机制来发布事件被管理的bean是对任何通知监听器状态管理的不负责。Spring JMX支持将处理所有JMX基础问题。所有人需要做的就是和应用开发人员一样实现NotificationPublisherAware接口并通过NotificationPublisher实例开始发布事件。注意，NotificationPublisher将在管理bean被注册到MBeanServer之后被设置。

使用NotificationPublisher实例非常简单，创建一个简单的JMX通知实例（或一个适当的Notification子类实例），通知中包含发布事件相关的数据 ，然后在NotificationPublisher实例上调用sendNotification（Notification），传递Notification。

下面是一个简单的例子，在这种场景下，导出的JmxTestBean实例在每次调用add(int, int)时会发布一个NotificationEvent。

```
package org.springframework.jmx;

import org.springframework.jmx.export.notification.NotificationPublisherAware;
import org.springframework.jmx.export.notification.NotificationPublisher;
import javax.management.Notification;

public class JmxTestBean implements IJmxTestBean, NotificationPublisherAware {

	private String name;
	private int age;
	private boolean isSuperman;
	private NotificationPublisher publisher;

	// other getters and setters omitted for clarity

	public int add(int x, int y) {
		int answer = x + y;
		this.publisher.sendNotification(new Notification("add", this, 0));
		return answer;
	}

	public void dontExposeMe() {
		throw new RuntimeException();
	}

	public void setNotificationPublisher(NotificationPublisher notificationPublisher) {
		this.publisher = notificationPublisher;
	}

}
```

NotificationPublisher接口和使其全部运行的机制是Spring JMX支持的良好的功能之一。然而它带来的代价是你的类和Spring，JMX耦合在一起；与以往一样，我们给出实用的建议，如果你需要NotificationPublisher提供的功能，那么你需要接受Spring和JMX的耦合。

## 27.8 更多资源

这章节包含了关于JMX更多的资源链接：

* Oracle的[JMX主页](http://www.oracle.com/technetwork/java/javase/tech/javamanagement-140525.html)
* [JMX规范](https://jcp.org/aboutJava/communityprocess/final/jsr003/index3.html)(JSR-000003)
* [JMX远程API规范](https://jcp.org/aboutJava/communityprocess/final/jsr160/index.html)（JSR-000160）
* [MX4J主页](http://mx4j.sourceforge.net)（JMX规范的各种开源实现）