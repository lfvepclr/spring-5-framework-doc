---
title: Spring 5 验证、数据绑定和类型转换[译]
date: 2017-05-13 21:15:12
tags: Spring
---

## 5.1 介绍

> JSR-303/JSR-349 Bean Validation
>
> 在设置支持方面，Spring Framework 4.0支持Bean Validation 1.0(JSR-303)和Bean Validation 1.1(JSR-349)，也将其改写成了Spring的`Validator`接口。
>
> 正如[5.8 Spring验证](#5.8 Spring Validation)所述，应用程序可以选择一次性全局启用Bean验证，并使其专门用于所有的验证需求。
>
> 正如[5.8.3 配置DataBinder](#5.8.3 Configuring a DataBinder)所述，应用程序也可以为每个`DataBinder`实例注册额外的Spring `Validator`实例，这可能有助于不通过使用注解而插入验证逻辑。

考虑将验证作为业务逻辑是有利有弊的，Spring提供了一种不排除利弊的用于验证(和数据绑定)的设计。具体的验证不应该捆绑在web层，应该容易本地化并且它应该能够插入任何可用的验证器。考虑到以上这些，Spring想出了一个`Validator`接口，它在应用程序的每一层基本都是可用的。数据绑定对于将用户输入动态绑定到应用程序的领域模型上(或者任何你用于处理用户输入的对象)是非常有用的。Spring提供了所谓的`DataBinder`来处理这个。`Validator`和`DataBinder`组成了`validation`包，其主要用于但并不局限于MVC框架。

`BeanWrapper`是Spring框架中的一个基本概念且在很多地方使用。然而，你可能并不需要直接使用`BeanWrapper`。尽管这是参考文档，我们仍然觉得有一些说明需要一步步来。我们将会在本章中解释`BeanWrapper`，因为你极有可能会在尝试将数据绑定到对象的时候使用它。

Spring的DataBinder和底层的BeanWrapper都使用PropertyEditor来解析和格式化属性值。`PropertyEditor`概念是JavaBeans规范的一部分，并会在本章进行说明。Spring 3不仅引入了"core.convert"包来提供一套通用类型转换工具，还有一个高层次的"format"包用于格式化UI字段值。可以将这些新包视作更简单的PropertyEditor替代方式来使用，本章还会对此进行讨论。

## 5.2 使用Spring的验证器接口进行验证

Spring具有一个`Validator`接口可以让你用于验证对象。`Validator`接口在工作时需要使用一个`Errors`对象，以便于在验证过程中，验证器可以将验证失败的信息报告给这个`Errors`对象。

让我们考虑一个小的数据对象：

```java
public class Person {

    private String name;
    private int age;
    
    // the usual getters and setters...

}
```

通过实现`org.springframework.validation.Validator`的下列两个接口，我们打算为`Person`类提供验证行为：

- `support(Class)` - 这个`Validator`是否可以验证给定`Class`的实例
- `validate(Object,org.springframework.validation.Errors)` - 验证给定的对象并且万一验证错误，可以将这些错误注册到给定的`Errors`对象

实现一个`Validator`是相当简单的，特别是当你知道Spring框架还提供了`ValidationUtils`辅助类：

```java
public class PersonValidator implements Validator {

	/**
	 * This Validator validates *just* Person instances
	 */
	public boolean supports(Class clazz) {
		return Person.class.equals(clazz);
	}

	public void validate(Object obj, Errors e) {
		ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
		Person p = (Person) obj;
		if (p.getAge() < 0) {
			e.rejectValue("age", "negativevalue");
		} else if (p.getAge() > 110) {
			e.rejectValue("age", "too.darn.old");
		}
	}
}
```

正如你看到的，`ValidationUtils`类的`static`  `rejectIfEmpty(..)`方法被用于拒绝那些值为`null`或者空字符串的`'name'`属性。除了上面展示的例子之外，去看一看`ValidationUtils`的java文档有助于了解它提供的功能。

通过实现单个的`Validator`类来逐个验证富对象中的嵌套对象当然是有可能的，然而将验证逻辑封装在每个嵌套类对象自身的`Validator`实现中可能是一种更好的选择。`Customer`就是一个*'富'*对象的简单示例，它由两个字符串属性(姓和名)以及一个复杂对象`Address`组成。`Address`对象可能独立于`Customer`对象使用，因此已经实现了一个独特的`AddressValidator`。如果你想要你的`CustomerValidator`不借助于复制粘贴而重用包含在`AddressValidator`中的逻辑，那么你可以通过依赖注入或者实例化你的`CustomerValidator`中的`AddressValidator`，然后像这样使用它：

```java
public class CustomerValidator implements Validator {

	private final Validator addressValidator;

	public CustomerValidator(Validator addressValidator) {
		if (addressValidator == null) {
			throw new IllegalArgumentException("The supplied [Validator] is " +
				"required and must not be null.");
		}
		if (!addressValidator.supports(Address.class)) {
			throw new IllegalArgumentException("The supplied [Validator] must " +
				"support the validation of [Address] instances.");
		}
		this.addressValidator = addressValidator;
	}

	/**
	 * This Validator validates Customer instances, and any subclasses of Customer too
	 */
	public boolean supports(Class clazz) {
		return Customer.class.isAssignableFrom(clazz);
	}

	public void validate(Object target, Errors errors) {
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
		ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
		Customer customer = (Customer) target;
		try {
			errors.pushNestedPath("address");
			ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
		} finally {
			errors.popNestedPath();
		}
	}
}
```

验证错误被报告给传递到验证器的`Errors`对象。在使用Spring Web MVC的情况下，你可以使用`<spring:bind/>`标签来检查错误信息，不过当然你也可以自己检查错误对象。有关它提供的方法的更多信息可以在java文档中找到。

## 5.3 将代码解析成错误消息

在之前我们已经谈论了数据绑定和验证，最后一件值得讨论的事情是输出对应于验证错误的消息。在我们上面展示的例子里，我们拒绝了`name`和`age`字段。如果我们要使用`MessageSource`来输出错误消息，我们将会使用我们在拒绝该字段(这个情况下是'姓名'和'年龄')时给出的错误代码。当你调用(不管是直接调用还是间接通过使用`ValidationUtils`类调用)来自`Errors`接口的`rejectValue`或者其他`reject`方法时，其底层实现不仅会注册你传入的代码，还会注册一些额外的错误代码。注册怎样的错误代码取决于它所使用的`MessageCodesResolver`，默认情况下，会使用`DefaultMessageCodesResolver`，其不仅会使用你提供的代码注册消息，还会注册包含你传递给拒绝方法的字段名称的消息。所以如果你使用`rejectValue("age", "too.darn.old")`来拒绝一个字段，除了`too.darn.old`代码，Spring还会注册`too.darn.old.age`和`too.darn.old.age.int`(第一个会包含字段名称且第二个会包含字段类型)。这样做是为了方便开发人员来定位错误消息等。

有关`MessageCodesResolver`和其默认策略的更多信息可以分别在`MessageCodesResolver`以及`DefaultMessageCodesResolver`的在线java文档中找到。

## 5.4 Bean操作和BeanWrapper

`org.springframework.beans`包遵循Oracle提供的JavaBeans标准。一个JavaBean只是一个包含默认无参构造器的类，它遵循一个命名约定(通过一个例子)：一个名为`bingoMadness`属性将有一个设置方法`setBingoMadness(..)`和一个获取方法`getBingoMadness(..)`。有关JavaBeans和其规范的更多信息，请参考Oracle的网站([javabeans](https://docs.oracle.com/javase/6/docs/api/java/beans/package-summary.html))。

beans包里一个非常重要的类是`BeanWrapper`接口和它的相应实现(`BeanWrapperImpl`)。引用自java文档，`BeanWrapper`提供了设置和获取属性值(单独或批量)、获取属性描述符以及查询属性以确定它们是可读还是可写的功能。`BeanWrapper`还提供对嵌套属性的支持，能够不受嵌套深度的限制启用子属性的属性设置。然后，`BeanWrapper`提供了无需目标类代码的支持就能够添加标准JavaBeans的`PropertyChangeListeners`和`VetoableChangeListeners`的能力。最后然而并非最不重要的是，`BeanWrapper`提供了对索引属性设置的支持。`BeanWrapper`通常不会被应用程序的代码直接使用，而是由`DataBinder`和`BeanFactory`使用。

`BeanWrapper`的名字已经部分暗示了它的工作方式：它包装一个bean以对其执行操作，比如设置和获取属性。

### 5.4.1 设置并获取基本和嵌套属性

使用`setPropertyValue(s)`和`getPropertyValue(s)`可以设置并获取属性，两者都带有几个重载方法。在Spring自带的java文档中对它们有更详细的描述。重要的是要知道对象属性指示的几个约定。几个例子：

**表 5.1. 属性示例**

| 表达式                    | 说明                                       |
| ---------------------- | ---------------------------------------- |
| `name`                 | 表示属性`name`与方法`getName()`或`isName()`和`setName()`相对应 |
| `account.name`         | 表示属性`account`的嵌套属性`name`与方法`getAccount().setName()`或`getAccount().getName()`相对应 |
| `account[2]`           | 表示索引属性`account`的第三个元素。索引属性可以是`array`、`list`或其他自然排序的集合 |
| `account[COMPANYNAME]` | 表示映射属性`account`被键*COMPANYNAME*索引到的映射项的值  |

下面你会发现一些使用`BeanWrapper`来获取和设置属性的例子。

*(如果你不打算直接使用`BeanWrapper`，那么下一部分对你来说并不重要。如果你仅使用`DataBinder`和`BeanFactory`以及它们开箱即用的实现，你应该跳到关于`PropertyEditor`部分的开头)。*

考虑下面两个类：

```java
public class Company {

	private String name;
	private Employee managingDirector;

	public String getName() {
		return this.name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public Employee getManagingDirector() {
		return this.managingDirector;
	}

	public void setManagingDirector(Employee managingDirector) {
		this.managingDirector = managingDirector;
	}
}
```

```java
public class Employee {

	private String name;

	private float salary;

	public String getName() {
		return this.name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public float getSalary() {
		return salary;
	}

	public void setSalary(float salary) {
		this.salary = salary;
	}
}
```

以下的代码片段展示了如何检索和操纵实例化的`Companies`和`Employees`的某些属性：

```java
BeanWrapper company = new BeanWrapperImpl(new Company());
// setting the company name..
company.setPropertyValue("name", "Some Company Inc.");
// ... can also be done like this:
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// ok, let's create the director and tie it to the company:
BeanWrapper jim = new BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// retrieving the salary of the managingDirector through the company
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```

### 5.4.2 内置PropertyEditor实现

Spring使用`PropertyEditor`的概念来实现`Object`和`String`之间的转换。如果你考虑到它，有时候换另一种方式表示属性可能比对象本身更方便。举个例子，一个`Date`可以以人类可读的方式表示(如`String` `'2007-14-09'`)，同时我们依然能把人类可读的形式转换回原始的时间(甚至可能更好：将任何以人类可读形式输入的时间转换回`Date`对象)。这种行为可以通过注册类型为`PropertyEditor`的自定义编辑器来实现。在`BeanWrapper`或上一章提到的特定IoC容器中注册自定义编辑器，可以使其了解如何将属性转换为期望的类型。请阅读Oracle为`java.beans`包提供的java文档来获取更多关于`PropertyEditor`的信息。

这是Spring使用属性编辑的两个例子：

- 使用`PropertyEditor`来完成*bean的属性设置*。当提到将`java.lang.String`作为你在XML文件中声明的某些bean的属性值时，Spring将会(如果相应的属性的设置方法具有一个`Class`参数)使用`ClassEditor`尝试将参数解析成`Class`对象。
- 在Spring的MVC框架中*解析HTTP请求的参数*是由各种`PropertyEditor`完成的，你可以把它们手动绑定到`CommandController`的所有子类。

Spring有一些内置的`PropertyEditor`使生活变得轻松。它们中的每一个都已列在下面，并且它们都被放在`org.springframework.beans.propertyeditors`包中。大部分但并不是全部(如下所示)，默认情况下会由`BeanWrapperImpl`注册。在某种方式下属性编辑器是可配置的，那么理所当然，你可以注册你自己的变种来覆盖默认编辑器：

<span id="5.4.2-Built-in PropertyEditor implementations">**Table 5.2. 内置PropertyEditor**</span>

| 类                         | 说明                                       |
| ------------------------- | ---------------------------------------- |
| `ByteArrayPropertyEditor` | 针对字节数组的编辑器。字符串会简单地转换成相应的字节表示。默认情况下由`BeanWrapperImpl`注册。 |
| `ClassEditor`             | 将类的字符串表示形式解析成实际的类形式并且也能返回实际类的字符串表示形式。如果找不到类，会抛出一个`IllegalArgumentException`。默认情况下由`BeanWrapperImpl`注册。 |
| `CustomBooleanEditor`     | 针对`Boolean`属性的可定制的属性编辑器。默认情况下由`BeanWrapperImpl`注册，但是可以作为一种自定义编辑器通过注册其自定义实例来进行覆盖。 |
| `CustomCollectionEditor`  | 针对集合的属性编辑器，可以将原始的`Collection`转换成给定的目标`Collection`类型。 |
| `CustomDateEditor`        | 针对java.util.Date的可定制的属性编辑器，支持自定义的时间格式。不会被默认注册，用户必须使用适当格式进行注册。 |
| `CustomNumberEditor`      | 针对任何Number子类(比如`Integer`、`Long`、`Float`、`Double`)的可定制的属性编辑器。默认情况下由`BeanWrapperImpl`注册，但是可以作为一种自定义编辑器通过注册其自定义实例来进行覆盖。 |
| `FileEditor`              | 能够将字符串解析成`java.io.File`对象。默认情况下由`BeanWrapperImpl`注册。 |
| `InputStreamEditor`       | 一次性的属性编辑器，能够读取文本字符串并生成(通过中间的`ResourceEditor`以及`Resource`)一个`InputStream`对象，因此*`InputStream`类型的属性可以直接以字符串设置。请注意默认的使用方式不会为你关闭`InputStream`！默认情况下由`BeanWrapperImpl`注册。 |
| `LocaleEditor`            | 能够将字符串解析成`Locale`对象，反之亦然(字符串格式是*[country]*[variant]，这与Locale提供的toString()方法是一样的)。默认情况下由`BeanWrapperImpl`注册。 |
| `PatternEditor`           | 能够将字符串解析成`java.util.regex.Pattern`对象，反之亦然。 |
| `PropertiesEditor`        | 能够将字符串(按照`java.util.Properties`类的java文档定义的格式进行格式化)解析成`Properties`对象。默认情况下由`BeanWrapperImpl`注册。 |
| `StringTrimmerEditor`     | 用于缩减字符串的属性编辑器。有选择性允许将一个空字符串转变成`null`值。不会进行默认注册，需要在用户有需要的时候注册。 |
| `URLEditor`               | 能够将一个URL的字符串表示解析成实际的`URL`对象。默认情况下由`BeanWrapperImpl`注册。 |

Spring使用`java.beans.PropertyEditorManager`来设置可能需要的属性编辑器的搜索路径。搜索路径中还包括了`sun.bean.editors`，这个包里面包含如`Font`、`Color`类型以及其他大部分基本类型的`PropertyEditor`实现。还要注意的是，如果`PropertyEditor`类与它们所处理的类位于同一个包并且除了'Editor'后缀之外拥有相同的名字，那么标准的JavaBeans基础设施会自动发现这些它们(不需要你显式的注册它们)。例如，有人可能会有以下的类和包结构，这已经足够识别出`FooEditor`类并将其作为**`Foo`**类型属性的`PropertyEditor`。

```
com
  chank
    pop
      Foo
      FooEditor // the PropertyEditor for the Foo class
```

要注意的是在这里你也可以使用标准JavaBeans机制的`BeanInfo`(在[in not-amazing-detail here](https://docs.oracle.com/javase/tutorial/javabeans/advanced/customization.html)有描述)。在下面的示例中，你可以看到使用`BeanInfo`机制为一个关联类的属性显式注册一个或多个`PropertyEditor`实例。

```
com
  chank
    pop
      Foo
      FooBeanInfo // the BeanInfo for the Foo class
```

这是被引用到的`FooBeanInfo`类的Java源代码。它会将一个`CustomNumberEditor`同`Foo`类的`age`属性关联。

```java
public class FooBeanInfo extends SimpleBeanInfo {

	public PropertyDescriptor[] getPropertyDescriptors() {
		try {
			final PropertyEditor numberPE = new CustomNumberEditor(Integer.class, true);
			PropertyDescriptor ageDescriptor = new PropertyDescriptor("age", Foo.class) {
				public PropertyEditor createPropertyEditor(Object bean) {
					return numberPE;
				};
			};
			return new PropertyDescriptor[] { ageDescriptor };
		}
		catch (IntrospectionException ex) {
			throw new Error(ex.toString());
		}
	}
}
```

<span id="Registering additional custom PropertyEditors" />

#### 注册额外的自定义PropertyEditor

当bean属性设置成一个字符串值时，Spring IoC容器最终会使用标准JavaBeans的`PropertyEditor`将这些字符串转换成复杂类型的属性。Spring预先注册了一些自定义`PropertyEditor`(例如将一个以字符串表示的类名转换成真正的`Class`对象)。此外，Java的标准JavaBeans `PropertyEditor`查找机制允许一个`PropertyEditor`只需要恰当的命名并同它支持的类位于相同的包，就能够自动发现它。

如果需要注册其他自定义的`PropertyEditor`，还有几种可用机制。假设你有一个`BeanFactory`引用，最人工化的方式(但通常并不方便或者推荐)是直接使用`ConfigurableBeanFactory`接口的`registerCustomEditor()`方法。另一种略为方便的机制是使用一个被称为`CustomEditorConfigurer`的特殊的bean factory后置处理器(*post-processor*)。虽然bean factory后置处理器可以与`BeanFactory`实现一起使用，但是因为`CustomEditorConfigurer`有一个嵌套属性设置过程，所以强烈推荐它与`ApplicationContext`一起使用，这样就可以采用与其他bean类似的方式来部署它，并自动检测和应用。

请注意所有的bean工厂和应用上下文都会自动地使用一些内置属性编辑器，这些编辑器通过一个被称为`BeanWrapper`的接口来处理属性转换。`BeanWrapper`注册的那些标准属性编辑器已经列在[上一部分](#5.4.2-Built-in PropertyEditor implementations)。 此外，针对特定的应用程序上下文类型，`ApplicationContext`会用适当的方法覆盖或添加一些额外的编辑器来处理资源查找。

标准的JavaBeans `PropertyEditor`实例用于将字符串表示的属性值转换成实际的复杂类型属性。`CustomEditorConfigurer`，一个bean factory后处理器，可以为添加额外的`PropertyEditor`到`ApplicationContext`提供便利支持。

考虑一个用户类`ExoticType`和另外一个需要将`ExoticType`设为属性的类`DependsOnExoticType`： 

```java
package example;

public class ExoticType {

	private String name;

	public ExoticType(String name) {
		this.name = name;
	}
}

public class DependsOnExoticType {

	private ExoticType type;

	public void setType(ExoticType type) {
		this.type = type;
	}
}
```

当东西都被正确设置时，我们希望能够分配字符串给type属性，而`PropertyEditor`会在背后将其转换成实际的`ExoticType`实例：

```xml
<bean id="sample" class="example.DependsOnExoticType">
	<property name="type" value="aNameForExoticType"/>
</bean>
```

`PropertyEditor`实现可能与此类似：

```java
// converts string representation to ExoticType object
package example;

public class ExoticTypeEditor extends PropertyEditorSupport {

	public void setAsText(String text) {
		setValue(new ExoticType(text.toUpperCase()));
	}
}
```

最后，我们使用`CustomEditorConfigurer`将一个新的`PropertyEditor`注册到`ApplicationContext`，那么在需要的时候就能够使用它：

```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
	<property name="customEditors">
		<map>
			<entry key="example.ExoticType" value="example.ExoticTypeEditor"/>
		</map>
	</property>
</bean>
```

#### 使用PropertyEditorRegistrar

另一种将属性编辑器注册到Spring容器的机制是创建和使用一个`PropertyEditorRegistrar`。当你需要在几个不同场景里使用同一组属性编辑器，这个接口会特别有用：编写一个相应的registrar并在每个用例里重用。`PropertyEditorRegistrar`与一个被称为`PropertyEditorRegistry`的接口配合工作，后者被Spring的`BeanWrapper`(以及`DataBinder`)实现。当与`CustomEditorConfigurer`配合使用的时候，`PropertyEditorRegistrar`特别方便([这里](#Registering additional custom PropertyEditors)有介绍)，因为前者暴露了一个方法`setPropertyEditorRegistrars(..)`：以这种方式添加到`CustomEditorConfigurerd`的`PropertyEditorRegistrar`可以很容易地在`DataBinder`和Spring MVC `Controllers`之间共享。另外，它避免了在自定义编辑器上的同步需求：一个`PropertyEditorRegistrar`可以为每一次bean创建尝试创建新的`PropertyEditor`实例。

使用`PropertyEditorRegistrar`可能最好还是以一个例子来说明。首先，你需要创建你自己的`PropertyEditorRegistrar`实现：

```java
package com.foo.editors.spring;

public final class CustomPropertyEditorRegistrar implements PropertyEditorRegistrar {

	public void registerCustomEditors(PropertyEditorRegistry registry) {

		// it is expected that new PropertyEditor instances are created
		registry.registerCustomEditor(ExoticType.class, new ExoticTypeEditor());

		// you could register as many custom property editors as are required here...
	}
}
```

也可以查看`org.springframework.beans.support.ResourceEditorRegistrar`当作一个`PropertyEditorRegistrar`实现的示例。注意在它的`registerCustomEditors(..)`方法实现里是如何为每个属性编辑器创建新的实例的。

接着我们配置了一个`CustomEditorConfigurerd`并将我们的`CustomPropertyEditorRegistrar`注入其中：

```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
	<property name="propertyEditorRegistrars">
		<list>
			<ref bean="customPropertyEditorRegistrar"/>
		</list>
	</property>
</bean>

<bean id="customPropertyEditorRegistrar"
	class="com.foo.editors.spring.CustomPropertyEditorRegistrar"/>
```

最后，有点偏离本章的重点，针对你们之中使用[Spring's MVC web framework](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html)的那些人，使用`PropertyEditorRegistrar`与数据绑定的`Controller`(比如`SimpleFormController`)配合使用会非常方便。下面是一个在`initBinder(..)`方法的实现里使用`PropertyEditorRegistrar`的例子：

```java
public final class RegisterUserController extends SimpleFormController {

	private final PropertyEditorRegistrar customPropertyEditorRegistrar;

	public RegisterUserController(PropertyEditorRegistrar propertyEditorRegistrar) {
		this.customPropertyEditorRegistrar = propertyEditorRegistrar;
	}

	protected void initBinder(HttpServletRequest request,
			ServletRequestDataBinder binder) throws Exception {
		this.customPropertyEditorRegistrar.registerCustomEditors(binder);
	}

	// other methods to do with registering a User
}
```

这种`PropertyEditor`注册的风格可以导致简洁的代码(`initBinder(..)`的实现仅仅只有一行！)，同时也允许将通用的`PropertyEditor`注册代码封装到一个类里然后根据需要在尽可能多的`Controller`之间共享。

## 5.5 Spring类型转换

Spring 3引入了`core.convert`包来提供一个一般类型的转换系统。这个系统定义了实现类型转换逻辑的服务提供接口(SPI)以及在运行时执行类型转换的API。在Spring容器内，这个系统可以当作是PropertyEditor的替代选择，用于将外部bean的属性值字符串转换成所需的属性类型。这个公共的API也可以在你的应用程序中任何需要类型转换的地方使用。

### 5.5.1 Converter SPI

实现类型转换逻辑的SPI是简单并且强类型的：

```java
package org.springframework.core.convert.converter;

public interface Converter<S, T> {

	T convert(S source);

}
```

要创建属于你自己的转换器，只需要简单的实现以上接口即可。泛型参数S表示你想要进行转换的源类型，而泛型参数T表示你想要转换的目标类型。如果一个包含S类型元素的集合或数组需要转换为一个包含T类型的数组或集合，那么这个转换器也可以被透明地应用，前提是已经注册了一个委托数组或集合的转换器(默认情况下会是`DefaultConversionService`处理)。

对每次方法`convert(S)`的调用，source参数值必须确保不为空。如果转换失败，你的转换器可以抛出任何非受检异常(*unchecked exception*)；具体来说，为了报告一个非法的source参数值，应该抛出一个`IllegalArgumentException`。还有要注意确保你的`Converter`实现必须是线程安全的。

为方便起见，`core.convert.support`包已经提供了一些转换器实现，这些实现包括了从字符串到数字以及其他常见类型的转换。考虑将`StringToInteger`作为一个典型的`Converter`实现示例：

```java
package org.springframework.core.convert.support;

final class StringToInteger implements Converter<String, Integer> {

	public Integer convert(String source) {
		return Integer.valueOf(source);
	}

}
```

### 5.5.2 ConverterFactory

当你需要集中整个类层次结构的转换逻辑时，例如，碰到将String转换到java.lang.Enum对象的时候，请实现`ConverterFactory`：

```java
package org.springframework.core.convert.converter;

public interface ConverterFactory<S, R> {

	<T extends R> Converter<S, T> getConverter(Class<T> targetType);

}
```

泛型参数S表示你想要转换的源类型，泛型参数R表示你可以转换的那些范围内的类型的基类。然后实现getConverter(Class<T>)，其中T就是R的一个子类。

考虑将`StringToEnum`作为ConverterFactory的一个示例：

```java
package org.springframework.core.convert.support;

final class StringToEnumConverterFactory implements ConverterFactory<String, Enum> {

	public <T extends Enum> Converter<String, T> getConverter(Class<T> targetType) {
		return new StringToEnumConverter(targetType);
	}

	private final class StringToEnumConverter<T extends Enum> implements Converter<String, T> {

		private Class<T> enumType;

		public StringToEnumConverter(Class<T> enumType) {
			this.enumType = enumType;
		}

		public T convert(String source) {
			return (T) Enum.valueOf(this.enumType, source.trim());
		}
	}
}
```

### 5.5.3 GenericConverter

当你需要一个复杂的转换器实现时，请考虑GenericConverter接口。GenericConverter具备更加灵活但是不太强的类型签名，以支持在多种源类型和目标类型之间的转换。此外，当实现你的转换逻辑时，GenericConverter还可以使源字段和目标字段的上下文对你可用，这样的上下文允许类型转换由字段上的注解或者字段声明中的泛型信息来驱动。

```java
package org.springframework.core.convert.converter;

public interface GenericConverter {

	public Set<ConvertiblePair> getConvertibleTypes();

	Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

要实现一个GenericConverter，getConvertibleTypes()方法要返回支持的源-目标类型对，然后实现convert(Object,TypeDescriptor,TypeDescriptor)方法来实现你的转换逻辑。源TypeDescriptor提供了对持有被转换值的源字段的访问，目标TypeDescriptor提供了对设置转换值的目标字段的访问。

一个很好的GenericConverter的示例是一个在Java数组和集合之间进行转换的转换器。这样一个ArrayToCollectionConverter可以通过内省声明了目标集合类型的字段以解析集合元素的类型，这将允许原数组中每个元素可以在集合被设置到目标字段之前转换成集合元素的类型。

> 由于GenericConverter是一个更复杂的SPI接口，所以对基本类型的转换需求优先使用Converter或者ConverterFactory。 

#### ConditionalGenericConverter

有时候你只想要在特定条件成立的情况下`Converter`才执行，例如，你可能只想要在目标字段存在特定注解的情况下才执行`Converter`，或者你可能只想要在目标类中定义了特定方法，比如`static` `valueOf`方法，才执行`Converter`。`ConditionalGenericConverter`是`GenericConverter`和`ConditionalConveter`接口的联合，允许你定义这样的自定义匹配条件：

```java
public interface ConditionalGenericConverter
        extends GenericConverter, ConditionalConverter {

	boolean matches(TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

`ConditionalGenericConverter`的一个很好的例子是一个在持久化实体标识和实体引用之间进行转换的实体转换器。这个实体转换器可能只匹配这样的条件--目标实体类声明了一个静态的查找方法，例如`findAccount(Long)`，你将在`matches(TypeDescriptor,TypeDescriptor)`方法实现里执行这样的查找方法的检测。

### 5.5.4 ConversionService API

ConversionService接口定义了运行时执行类型转换的统一API，转换器往往是在这个门面(*facade*)接口背后执行：

```java
package org.springframework.core.convert;

public interface ConversionService {

	boolean canConvert(Class<?> sourceType, Class<?> targetType);

	<T> T convert(Object source, Class<T> targetType);

	boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);

	Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);

}
```

大多数ConversionService实现也会实现`ConverterRegistry`接口，这个接口提供一个用于注册转换器的服务提供接口(SPI)。在内部，一个ConversionService实现会以委托给注册其中的转换器的方式来执行类型转换逻辑。

`core.convert.support`包已经提供了一个强大的ConversionService实现，`GenericConversionService`是适用于大多数环境的通用实现，`ConversionServiceFactory`以工厂的方式为创建常见的ConversionService配置提供了便利。

### 5.5.5 配置ConversionService

ConversionService是一个被设计成在应用程序启动时会进行实例化的无状态对象，随后可以在多个线程之间共享。在一个Spring应用程序中，你通常会为每一个Spring容器(或者应用程序上下文ApplicationContext)配置一个ConversionService实例，它会被Spring接收并在框架需要执行一个类型转换时使用。你也可以将这个ConversionService直接注入到你任何的Bean中并直接调用。

> 如果Spring没有注册ConversionService，则会使用原始的基于PropertyEditor的系统。

要向Spring注册默认的ConversionService，可以用`conversionService`作为id来添加如下的bean定义：

```xml
<bean id="conversionService"
	class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```

默认的ConversionService可以在字符串、数字、枚举、映射和其他常见类型之间进行转换。为了使用你自己的自定义转换器来补充或者覆盖默认的转换器，可以设置`converters`属性，该属性值可以是Converter、ConverterFactory或者GenericConverter之中任何一个的接口实现。

```xml
<bean id="conversionService"
		class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
		<set>
			<bean class="example.MyCustomConverter"/>
		</set>
	</property>
</bean>
```

在一个Spring MVC应用程序中使用ConversionService也是比较常见的，可以去看Spring MVC章节的[Section 18.16.3 “Conversion and Formatting”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-conversion)。

在某些情况下，你可能希望在转换期间应用格式化，可以看[5.6.3 "FormatterRegistry SPI"](#5.6.3 FormatterRegistry SPI)获取使用`FormattingConversionServiceFactoryBean`的细节。

### 5.5.6 编程方式使用ConversionService

要以编程方式使用ConversionService，你只需要像处理其他bean一样注入一个引用即可：

```java
@Service
public class MyService {

	@Autowired
	public MyService(ConversionService conversionService) {
		this.conversionService = conversionService;
	}

	public void doIt() {
		this.conversionService.convert(...)
	}
}
```

对大多数用例来说，`convert`方法指定了可以使用的目标类型，但是它不适用于更复杂的类型比如参数化元素的集合。例如，如果你想要以编程方式将一个`Integer`的`List`转换成一个`String`的`List`，就需要为原类型和目标类型提供一个正式的定义。

幸运的是，`TypeDescriptor`提供了多种选项使事情变得简单：

```java
DefaultConversionService cs = new DefaultConversionService();

List<Integer> input = ....
cs.convert(input,
	TypeDescriptor.forObject(input), // List<Integer> type descriptor
	TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));
```

注意`DefaultConversionService`会自动注册对大部分环境都适用的转换器，这其中包括了集合转换器、标量转换器还有基本的`Object`到`String`的转换器。可以通过调用`DefaultConversionService`类上的静态方法`addDefaultConverters`来向任意的`ConverterRegistry`注册相同的转换器。

因为值类型的转换器可以被数组和集合重用，所以假设标准集合处理是恰当的，就没有必要创建将一个`S`的`Collection`转换成一个`T`的`Collection`的特定转换器。

## 5.6 Spring字段格式化

如上一节所述，`core.convert`包是一个通用类型转换系统，它提供了统一的ConversionService API以及强类型的Converter SPI用于实现将一种类型转换成另外一种的转换逻辑。Spring容器使用这个系统来绑定bean属性值，此外，Spring表达式语言(SpEL)和DataBinder也都使用这个系统来绑定字段值。举个例子，当SpEL需要将`Short`强制转换成`Long`来完成一次`expression.setValue(Object bean, Object value)`尝试时，core.convert系统就会执行这个强制转换。

现在让我们考虑一个典型的客户端环境如web或桌面应用程序的类型转换要求，在这样的环境里，你通常会经历将字符串进行转换以支持客户端回传的过程以及转换回字符串以支持视图渲染的过程。此外，你经常需要对字符串值进行本地化。更通用的*core.convert*包中的Converter SPI不直接解决这种格式化要求。Spring 3为此引入了一个方便的Formatter SPI来直接解决这些问题，这个接口为客户端环境提供一种简单强大并且替代PropertyEditor的方案。

一般来说，当你需要实现通用的类型转换逻辑时请使用Converter SPI，例如，在java.util.Date和java.lang.Long之间进行转换。当你在一个客户端环境(比如web应用程序)工作并且需要解析和打印本地化的字段值时，请使用Formatter SPI。ConversionService接口为这两者提供了一套统一的类型转换API。

### 5.6.1 Formatter SPI

Formatter SPI实现字段格式化逻辑是简单并且强类型的：

```java
package org.springframework.format;

public interface Formatter<T> extends Printer<T>, Parser<T> {
}
```

Formatter接口扩展了Printer和Parser这两个基础接口：

```java
public interface Printer<T> {
	String print(T fieldValue, Locale locale);
}
```

```java
import java.text.ParseException;

public interface Parser<T> {
	T parse(String clientValue, Locale locale) throws ParseException;
}
```

要创建你自己的格式化器，只需要实现上面的Formatter接口。泛型参数T代表你想要格式化的对象的类型，例如，`java.util.Date`。实现`print()`操作可以将类型T的实例按客户端区域设置的显示方式打印出来。实现`parse()`操作可以从依据客户端区域设置返回的格式化表示中解析出类型T的实例。如果解析尝试失败，你的格式化器应该抛出一个ParseException或者IllegalArgumentException。请注意确保你的格式化器实现是线程安全的。

为方便起见，`format`子包中已经提供了一些格式化器实现。`number`包提供了`NumberFormatter`、`CurrencyFormatter`和`PercentFormatter`，它们通过使用`java.text.NumberFormat`来格式化`java.lang.Number`对象 。`datetime`包提供了`DateFormatter`，其通过使用`java.text.DateFormat`来格式化`java.util.Date`。`datetime.joda`包基于[Joda Time library](http://www.joda.org/joda-time/)提供了全面的日期时间格式化支持。

考虑将`DateFormatter`作为`Formatter`实现的一个例子：

```java
package org.springframework.format.datetime;

public final class DateFormatter implements Formatter<Date> {

	private String pattern;

	public DateFormatter(String pattern) {
		this.pattern = pattern;
	}

	public String print(Date date, Locale locale) {
		if (date == null) {
			return "";
		}
		return getDateFormat(locale).format(date);
	}

	public Date parse(String formatted, Locale locale) throws ParseException {
		if (formatted.length() == 0) {
			return null;
		}
		return getDateFormat(locale).parse(formatted);
	}

	protected DateFormat getDateFormat(Locale locale) {
		DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);
		dateFormat.setLenient(false);
		return dateFormat;
	}

}
```

Spring团队欢迎社区驱动的`Formatter`贡献，可以登陆网站[jira.spring.io](https://jira.spring.io/browse/SPR)了解如何参与贡献。 

### 5.6.2 注解驱动的格式化

如你所见，字段格式化可以通过字段类型或者注解进行配置，要将一个注解绑定到一个格式化器，可以实现AnnotationFormatterFactory：

```java
package org.springframework.format;

public interface AnnotationFormatterFactory<A extends Annotation> {

	Set<Class<?>> getFieldTypes();

	Printer<?> getPrinter(A annotation, Class<?> fieldType);

	Parser<?> getParser(A annotation, Class<?> fieldType);

}
```

泛型参数A代表你想要关联格式化逻辑的字段注解类型，例如`org.springframework.format.annotation.DateTimeFormat`。让`getFieldTypes()`方法返回可能使用注解的字段类型，让`getPrinter()`方法返回一个可以打印被注解字段的值的打印机(Printer)，让`getParser()`方法返回一个可以解析被注解字段的客户端值的解析器(Parser)。

下面这个AnnotationFormatterFactory实现的示例把@NumberFormat注解绑定到一个格式化器，此注解允许指定数字样式或模式：

```java
public final class NumberFormatAnnotationFormatterFactory
		implements AnnotationFormatterFactory<NumberFormat> {

	public Set<Class<?>> getFieldTypes() {
		return new HashSet<Class<?>>(asList(new Class<?>[] {
			Short.class, Integer.class, Long.class, Float.class,
			Double.class, BigDecimal.class, BigInteger.class }));
	}

	public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) {
		return configureFormatterFrom(annotation, fieldType);
	}

	public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) {
		return configureFormatterFrom(annotation, fieldType);
	}

	private Formatter<Number> configureFormatterFrom(NumberFormat annotation,
			Class<?> fieldType) {
		if (!annotation.pattern().isEmpty()) {
			return new NumberFormatter(annotation.pattern());
		} else {
			Style style = annotation.style();
			if (style == Style.PERCENT) {
				return new PercentFormatter();
			} else if (style == Style.CURRENCY) {
				return new CurrencyFormatter();
			} else {
				return new NumberFormatter();
			}
		}
	}
}
```

要触发格式化，只需要使用@NumberFormat对字段进行注解：

```java
public class MyModel {

	@NumberFormat(style=Style.CURRENCY)
	private BigDecimal decimal;

}
```

#### Format Annotation API

`org.springframework.format.annotation`包中存在一套可移植(portable)的格式化注解API。请使用@NumberFormat格式化java.lang.Number字段，使用@DateTimeFormat格式化java.util.Date、java.util.Calendar、java.util.Long(注：此处可能是原文错误，应为java.lang.Long)或者Joda Time字段。

下面这个例子使用@DateTimeFormat将java.util.Date格式化为ISO时间(yyyy-MM-dd)

```java
public class MyModel {

	@DateTimeFormat(iso=ISO.DATE)
	private Date date;

}
```

<span id="5.6.3 FormatterRegistry SPI" />

### 5.6.3 FormatterRegistry SPI

FormatterRegistry是一个用于注册格式化器和转换器的服务提供接口(SPI)。`FormattingConversionService`是一个适用于大多数环境的FormatterRegistry实现，可以以编程方式或利用`FormattingConversionServiceFactoryBean`声明成Spring bean的方式来进行配置。由于它也实现了`ConversionService`，所以可以直接配置它与Spring的DataBinder以及Spring表达式语言(SpEL)一起使用。

请查看下面的FormatterRegistry SPI：

```java
package org.springframework.format;

public interface FormatterRegistry extends ConverterRegistry {

	void addFormatterForFieldType(Class<?> fieldType, Printer<?> printer, Parser<?> parser);

	void addFormatterForFieldType(Class<?> fieldType, Formatter<?> formatter);

	void addFormatterForFieldType(Formatter<?> formatter);

	void addFormatterForAnnotation(AnnotationFormatterFactory<?, ?> factory);

}
```

如上所示，格式化器可以通过字段类型或者注解进行注册。

FormatterRegistry SPI允许你集中地配置格式化规则，而不是在你的控制器之间重复这样的配置。例如，你可能要强制所有的时间字段以某种方式被格式化，或者是带有特定注解的字段以某种方式被格式化。通过一个共享的FormatterRegistry，你可以只定义这些规则一次，而在需要格式化的时候应用它们。

### 5.6.4 FormatterRegistrar SPI

FormatterRegistrar是一个通过FormatterRegistry注册格式化器和转换器的服务提供接口(SPI)：

```java
package org.springframework.format;

public interface FormatterRegistrar {

	void registerFormatters(FormatterRegistry registry);

}
```

当要为一个给定的格式化类别(比如时间格式化)注册多个关联的转换器和格式化器时，FormatterRegistrar会非常有用。

下一部分提供了更多关于转换器和格式化器注册的信息。

### 5.6.5 在Spring MVC中配置格式化

请查看Spring MVC章节的[Section 18.16.3 "Conversion and Formatting"](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-conversion)。 

## 5.7 配置一个全局的日期&时间格式

默认情况下，未被`@DateTimeFormat`注解的日期和时间字段会使用`DateFormat.SHORT`风格从字符串转换。如果你愿意，你可以定义你自己的全局格式来改变这种默认行为。

你将需要确保Spring不会注册默认的格式化器，取而代之的是你应该手动注册所有的格式化器。请根据你是否依赖Joda Time库来确定是使用`org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar`类还是`org.springframework.format.datetime.DateFormatterRegistrar`类。

例如，下面的Java配置会注册一个全局的'yyyyMMdd'格式，这个例子不依赖于Joda Time库：

```java
@Configuration
public class AppConfig {

	@Bean
	public FormattingConversionService conversionService() {

		// Use the DefaultFormattingConversionService but do not register defaults
		DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService(false);

		// Ensure @NumberFormat is still supported
		conversionService.addFormatterForFieldAnnotation(new NumberFormatAnnotationFormatterFactory());

		// Register date conversion with a specific global format
		DateFormatterRegistrar registrar = new DateFormatterRegistrar();
		registrar.setFormatter(new DateFormatter("yyyyMMdd"));
		registrar.registerFormatters(conversionService);

		return conversionService;
	}
}
```

如果你更喜欢基于XML的配置，你可以使用一个`FormattingConversionServiceFactoryBean`，这是同一个例子，但这次使用了Joda Time：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
		http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd>

	<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="registerDefaultFormatters" value="false" />
		<property name="formatters">
			<set>
				<bean class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
			</set>
		</property>
		<property name="formatterRegistrars">
			<set>
				<bean class="org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar">
					<property name="dateFormatter">
						<bean class="org.springframework.format.datetime.joda.DateTimeFormatterFactoryBean">
							<property name="pattern" value="yyyyMMdd"/>
						</bean>
					</property>
				</bean>
			</set>
		</property>
	</bean>
</beans>
```

> Joda Time提供了不同的类型来表示`date`、`time`和`date-time`的值，`JodaTimeFormatterRegistrar`中的`dateFormatter`、`timeFormatter`和`dateTimeFormatter`属性应该为每种类型配置不同的格式。`DateTimeFormatterFactoryBean`提供了一种方便的方式来创建格式化器。

如果你在使用Spring MVC，请记住要明确配置所使用的转换服务。针对基于`@Configuration`的Java配置方式这意味着要继承`WebMvcConfigurationSupport`并且覆盖`mvcConversionService()`方法。针对XML的方式，你应该使用`mvc:annotation-drive`元素的`'conversion-service'`属性。更多细节请看[Section 18.16.3 "Conversion and Formatting"](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-conversion)。 

<span id="5.8 Spring Validation" />

## 5.8 Spring验证

Spring  3对验证支持引入了几个增强功能。首先，现在全面支持JSR-303 Bean Validation API；其次，当采用编程方式时，Spring的DataBinder现在不仅可以绑定对象还能够验证它们；最后，Spring MVC现在已经支持声明式地验证`@Controller`的输入。

### 5.8.1 JSR-303 Bean Validation API概述

JSR-303对Java平台的验证约束声明和元数据进行了标准化定义。使用此API，你可以用声明性的验证约束对领域模型的属性进行注解，并在运行时强制执行它们。现在已经有一些内置的约束供你使用，当然你也可以定义你自己的自定义约束。

为了说明这一点，考虑一个拥有两个属性的简单的PersonForm模型：

```java
public class PersonForm {
	private String name;
	private int age;
}
```

JSR-303允许你针对这些属性定义声明性的验证约束：

```java
public class PersonForm {

	@NotNull
	@Size(max=64)
	private String name;

	@Min(0)
	private int age;

}
```

当此类的一个实例被实现JSR-303规范的验证器进行校验的时候，这些约束就会被强制执行。

有关JSR-303/JSR-349的一般信息，可以访问网站[Bean Validation website](http://beanvalidation.org/)去查看。有关默认参考实现的具体功能的信息，可以参考网站[Hibernate Validator](http://hibernate.org/validator/)的文档。想要了解如何将Bean验证器提供程序设置为Spring bean，请继续保持阅读。

### 5.8.2 配置Bean验证器提供程序

Spring提供了对Bean Validation API的全面支持，这包括将实现JSR-303/JSR-349规范的Bean验证提供程序引导为Spring Bean的方便支持。这样就允许在应用程序任何需要验证的地方注入`javax.validation.ValidatorFactory`或者`javax.validation.Validator`。

把`LocalValidatorFactoryBean`当作Spring bean来配置成默认的验证器：

```xml
<bean id="validator"
	class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
```

以上的基本配置会触发Bean Validation使用它默认的引导机制来进行初始化。作为实现JSR-303/JSR-349规范的提供程序，如Hibernate Validator，可以存在于类路径以使它能被自动检测到。

#### 注入验证器

`LocalValidatorFactoryBean`实现了`javax.validation.ValidatorFactory`和`javax.validation.Validator`这两个接口，以及Spring的`org.springframework.validation.Validator`接口，你可以将这些接口当中的任意一个注入到需要调用验证逻辑的Bean里。

如果你喜欢直接使用Bean Validtion API，那么就注入`javax.validation.Validator`的引用：

```java
import javax.validation.Validator;

@Service
public class MyService {

	@Autowired
	private Validator validator;
```

如果你的Bean需要Spring Validation API，那么就注入`org.springframework.validation.Validator`的引用：

```java
import org.springframework.validation.Validator;

@Service
public class MyService {

	@Autowired
	private Validator validator;

}
```

#### 配置自定义约束

每一个Bean验证约束由两部分组成，第一部分是声明了约束和其可配置属性的`@Constraint`注解，第二部分是实现约束行为的`javax.validation.ConstraintValidator`接口实现。为了将声明与实现关联起来，每个`@Constraint`注解会引用一个相应的验证约束的实现类。在运行期间，`ConstraintValidatorFactory`会在你的领域模型遇到约束注解的情况下实例化被引用到的实现。

默认情况下，`LocalValidatorFactoryBean`会配置一个`SpringConstraintValidatorFactory`，其使用Spring来创建约束验证器实例。这允许你的自定义约束验证器可以像其他Spring bean一样从依赖注入中受益。

下面显示了一个自定义的`@Constraint`声明的例子，紧跟着是一个关联的`ConstraintValidator`实现，其使用Spring进行依赖注入：

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class)
public @interface MyConstraint {
}
```

```java
import javax.validation.ConstraintValidator;

public class MyConstraintValidator implements ConstraintValidator {

	@Autowired;
	private Foo aDependency;

	...
}
```

如你所见，一个约束验证器实现可以像其他Spring bean一样使用@Autowired注解来自动装配它的依赖。

#### Spring驱动的方法验证

被Bean Validation 1.1以及作为Hibernate Validator 4.3中的自定义扩展所支持的方法验证功能可以通过配置`MethodValidationPostProcessor`的bean定义集成到Spring的上下文中：

```xml
<bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>
```

为了符合Spring驱动的方法验证，需要对所有目标类用Spring的`@Validated`注解进行注解，且有选择地对其声明验证组，这样才可以使用。请查阅`MethodValidationPostProcessor`的java文档来了解针对Hibernate Validator和Bean Validation 1.1提供程序的设置细节。

#### 附加配置选项

对于大多数情况，默认的`LocalValidatorFactoryBean`配置应该足够。有许多配置选项来处理从消息插补到遍历解析的各种Bean验证结构。请查看`LocalValidatorFactoryBean`的java文档来获取关于这些选项的更多信息。

<span id="5.8.3 Configuring a DataBinder" />

### 5.8.3 配置DataBinder

从Spring 3开始，DataBinder的实例可以配置一个验证器。一旦配置完成，那么可以通过调用`binder.validate()`来调用验证器，任何的验证错误都会自动添加到DataBinder的绑定结果(BindingResult)。

当以编程方式处理DataBinder时，可以在绑定目标对象之后调用验证逻辑：

```java
Foo target = new Foo();
DataBinder binder = new DataBinder(target);
binder.setValidator(new FooValidator());

// bind to the target object
binder.bind(propertyValues);

// validate the target object
binder.validate();

// get BindingResult that includes any validation errors
BindingResult results = binder.getBindingResult();
```

通过`dataBinder.addValidators`和`dataBinder.replaceValidators`，一个DataBinder也可以配置多个`Validator`实例。当需要将全局配置的Bean验证与一个DataBinder实例上局部配置的Spring `Validator`结合时，这一点是非常有用的。

### 5.8.4 Spring MVC 3 验证

请查看Spring MVC章节的[Section 18.16.4 "Validation"](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-config-validation)。 