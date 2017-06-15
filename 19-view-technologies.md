# 19. 视图技术

## 19.1 简介

Spring架构与其他MVC框架所不同的重要一点是视图技术，比如  , 决定使用Groovy Markup Templates 或者Thymeleaf代替JSP仅仅是配置的问题 . 这个章节主要设计主流的视图技术，以及简单提及怎样使用新的技术。这个章节假设你已经熟悉第18.5节[“Resolving views”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/mvc.html#mvc-viewresolver) ，该章节涵盖了视图怎样与MVC框架结合的基本知识。

## 19.2 Thymeleaf

[Thymeleaf](http://www.thymeleaf.org/)是一种与MVC架构结合很好的视图技术. 不仅仅Spring团队而且Thymeleaf 自身也提供了很好的支持。

配置Thymeleaf 对Spring的支持通常需要定义几个bean, 像`ServletContextTemplateResolver`,  `SpringTemplateEngine`和 `ThymeleafViewResolver`。如需要更多详细信息请点击文档 [Thymeleaf+Spring](http://www.thymeleaf.org/documentation.html)。

## 19.3 Groovy Markup Templates

[Groovy Markup Template Engine](http://groovy-lang.org/templating.html#_the_markuptemplateengine)  是另一种被Spring支持的视图技术,此模板引擎是一种主要用于生成类似XML的标记（XML，XHTML，HTML5，…）的模板引擎，但可用于生成任何基于文本的内容。

这需要在classpath上配置Groovy 2.3.1+。

### 19.3.1 配置

配置 Groovy Markup Template Engine相当容易:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

       @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.groovy();
    }

    @Bean
    public GroovyMarkupConfigurer groovyMarkupConfigurer() {
        GroovyMarkupConfigurer configurer = new GroovyMarkupConfigurer();
        configurer.setResourceLoaderPath("/WEB-INF/");
        return configurer;
    }
}
```

使用MVC命名空间的XML文本:

```
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:groovy/>
</mvc:view-resolvers>

<mvc:groovy-configurer resource-loader-path="/WEB-INF/"/>
```

### 19.3.2 例子

和传统模板引擎不同, 这一个依赖于使用构建器语法的DSL。 以下是HTML页面的示例模板:

```
yieldUnescaped '<!DOCTYPE html>'
html(lang:'en') {
    head {
        meta('http-equiv':'"Content-Type" content="text/html; charset=utf-8"')
        title('My page')
    }
    body {
        p('This is an example of HTML contents')
    }
}
```

## 19.4 FreeMarker

FreeMarker是一种模板语言，可以用作Spring MVC应用程序中的视图技术. 更多关于模板语言的信息，请点击站点 [FreeMarker](http://www.freemarker.org/) .

### 19.4.1 依赖

您的Web应用程序需要包含freemarker-2.x.jar才能使用FreeMarker。 通常这包含在WEB-INF / lib文件夹中，其中jar保证由Java EE服务器找到并添加到应用程序的类路径中。 当然，假设你已经有spring-webmvc.jar在你的’WEB-INF / lib’目录了！

### 19.4.2 上下文配置

通过将相关的configurer bean定义添加到您的’\* -servlet.xml’来初始化一个合适的配置，如下所示：

```
<!-- freemarker config -->
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
</bean>

<!--
View resolvers can also be configured with ResourceBundles or XML files. If you need
different view resolving based on Locale, you have to use the resource bundle resolver.
-->
<bean id="viewResolver" class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
    <property name="cache" value="true"/>
    <property name="prefix" value=""/>
    <property name="suffix" value=".ftl"/>
</bean>
```

> 对于非web项目，在你定义的上下文中增加`FreeMarkerConfigurationFactoryBean`

### 19.4.3 创建模板

您的模板需要存储在上面显示的FreeMarkerConfigurer指定的目录中。 如果您使用突出显示的视图解析器，则逻辑视图名称与模板文件名称以类似于JSP的InternalResourceViewResolver的方式相关。 因此，如果您的控制器返回一个包含视图名称为“welcome”的ModelAndView对象，则解析器将查找/WEB-INF/freemarker/welcome.ftl模板。

### 19.4.4 高级FreeMarker配置

FreeMarker的’Settings’和’SharedVariables’可以直接传递给由Spring管理的FreeMarker Configuration对象，方法是在FreeMarkerConfigurer bean上设置相应的bean属性。 freemarkerSettings属性需要一个java.util.Properties对象，而freemarkerVariables属性需要一个java.util.Map。

```
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
    <property name="freemarkerVariables">
        <map>
            <entry key="xml_escape" value-ref="fmXmlEscape"/>
        </map>
    </property>
</bean>

<bean id="fmXmlEscape" class="freemarker.template.utility.XmlEscape"/>
```

> 有关设置和变量的详细信息，请参阅FreeMarker文档，因为它们适用于Configuration对象。

### 19.4.5绑定支持和表单处理

Spring提供了一个用于JSP的标签库，其中包含（其中包含）&lt;spring：bind /&gt;标签。 此标记主要允许表单从表单支持对象显示值，并显示来自网络或业务层的验证程序的失败验证结果。 Spring还支持FreeMarker中的相同功能，还有其他方便的宏用于生成表单输入元素。

#### 绑定宏

在双方语言的spring-webmvc.jar文件中都保留了一组标准的宏，因此它们始终可供适当配置的应用程序使用。

在Spring库中定义的一些宏被认为是内部的（私有的），但在宏定义中不存在这样的范围，使所有的宏都可以被调用代码和用户模板。 以下部分仅出现在您在模板中直接调用的宏。 如果您希望直接查看宏代码，则该文件在包org.springframework.web.servlet.view.freemarker中称为spring.ftl。

#### 简单绑定

在作为Spring MVC控制器的窗体视图的HTML表单（vm / ftl模板）中，您可以使用与以下类似的代码绑定到字段值，并以与JSP等效的类似方式显示每个输入字段的错误消息。 下面列出了以前配置的personForm视图的示例代码：

```
<!-- freemarker macros have to be imported into a namespace. We strongly
recommend sticking to 'spring' -->
<#import "/spring.ftl" as spring/>
<html>
    ...
    <form action="" method="POST">
        Name:
        <@spring.bind "myModelObject.name"/>
        <input type="text"
            name="${spring.status.expression}"
            value="${spring.status.value?html}"/><br>
        <#list spring.status.errorMessages as error> <b>${error}</b> <br> </#list>
        <br>
        ...
        <input type="submit" value="submit"/>
    </form>
    ...
</html>
```

> &lt;@ spring.bind&gt;需要一个“path”参数，它由命令对象的名称组成（除非您在FormController属性中更改它，否则将是“command”），后跟一个句点和该命令上的字段名称 您要绑定到的对象。 也可以使用嵌套字段，如“command.address.street”。 bind macro 假定web.xml中ServletContext参数defaultHtmlEscape指定的默认HTML转义行为。
>
> 称为&lt;@ spring.bindEscaped&gt;的宏的可选格式需要第二个参数，并显式指定是否在状态错误消息或值中使用HTML转义。 根据需要设置为true或false。 附加的表单处理宏简化了HTML转义的使用，并且尽可能使用这些宏。 他们将在下一节中进行说明。

#### 表单输入生成宏

两种语言的其他便利宏简化了绑定和表单生成（包括验证错误显示）。 从来没有必要使用这些宏来生成表单输入字段，并且可以混合使用简单的HTML或直接调用前面突出显示的弹簧绑定宏。

可用宏的下表显示了VTL和FTL定义以及每个参数列表。

| 宏 | FTL 定义 | 消息（根据代码参数从资源束输出字符串） |
| :--- | :--- | :--- |
| &lt;@spring.message code/&gt; | messageText（根据代码参数从资源束输出一个字符串，退回到默认参数的值） | &lt;@spring.messageText code, text/&gt; |
| **url**\(使用应用程序的上下文根前缀相对URL\) | &lt;@spring.url relativeUrl/&gt; | formInput（用于收集用户输入的标准输入字段） |
| &lt;@spring.formInput path, attributes, fieldType/&gt; | **formHiddenInput**\* \(用于提交非用户输入的隐藏输入字段\) | &lt;@spring.formHiddenInput path, attributes/&gt; |
| **formPasswordInput**\* \(用于收集密码的标准输入字段。 请注意，不会在此类型的字段中填充任何值\) | &lt;@spring.formPasswordInput path, attributes/&gt; | **formTextarea**\(大文本字段用于收集长，自由格式的文本输入\) |
| &lt;@spring.formTextarea path, attributes/&gt; | **formSingleSelect**\(d下拉框选项允许选择单个所需的值\) | &lt;@spring.formSingleSelect path, options, attributes/&gt; |
| **formMultiSelect**\(允许用户选择0个或更多值的选项列表框\) | &lt;@spring.formMultiSelect path, options, attributes/&gt; | **formRadioButtons**\(一组单选按钮允许从可用选项中进行单次选择\) |
| &lt;@spring.formRadioButtons path, options separator, attributes/&gt; | **formCheckboxes**\(一组允许选择0个或更多值的复选框\) | &lt;@spring.formCheckboxes path, options, separator, attributes/&gt; |
| **formCheckbox**\(a single checkbox\) | &lt;@spring.formCheckbox path, attributes/&gt; | **showErrors**\(简化显示绑定字段的验证错误\) |

* 在FTL（FreeMarker）中，这两个宏并不是实际需要的，因为您可以使用正常的formInput宏，指定’hidden’或’\`password’作为\`fieldType参数的值。.

任何上述宏的参数具有一致的含义:

* **path： **要绑定的字段的名称（如：“command.name”）
* **选项： **可以在输入字段中选择的所有可用值的映射。地图的键代表将从窗体返回并绑定到命令对象的值。按键显示的映射对象是表单上显示给用户的标签，可能与表单发回的对应值不同。通常这样的地图由控制器作为参考数据提供。可以根据需要的行为使用任何Map实现。对于严格排序的映射，可以使用诸如具有适当比较器的TreeMap的SortedMap，并且可以使用应该以插入顺序返回值的任意地图，使用commons-collections中的LinkedHashMap或LinkedMap
* **分隔符：**多个选项可用作谨慎元素（单选按钮或复选框），用于分隔列表中每个选项的字符序列（即“&lt;br&gt;”）.
  属性：要包含在HTML标签本身中的任意标签或文本的附加字符串。该字符串由字符串回显。例如，在textarea字段中，您可以提供“rows =”5“cols =”60“’的属性，或者您可以传递样式信息，如”style =“border：1px solid silver”
* **classOrStyle：**对于showErrors宏，包含每个错误的span标签将使用的CSS类的名称。如果没有提供信息（或值为空），那么错误将被包裹在&lt;b&gt; &lt;/ b&gt;标签中。

宏中的一些例子在FTL和VTL中的一些中概述。在两种语言之间存在使用差异的情况下，它们将在说明中进行说明。

#### 输入的值

formInput宏在上面的示例中使用path参数（command.name）和一个空的附加属性参数。 宏与所有其他表单生成宏一起在路径参数上执行隐式弹簧绑定。 绑定保持有效，直到发生新的绑定，因此showErrors宏不需要再次传递路径参数 – 它只是在上次创建绑定的字段时操作。

showErrors宏采用一个分隔符参数（用于在给定字段上分隔多个错误的字符），并且还接受第二个参数，此时为类名或样式属性。 请注意，FreeMarker能够为attributes参数指定默认值。

```
<@spring.formInput "command.name"/>
<@spring.showErrors "<br>"/>
```

输出显示在生成名称字段的表单片段中，并在表单提交后在该字段中没有值显示验证错误。 验证通过Spring的验证框架进行。

生成的HTML如下所示：

```
Name:
<input type="text" name="name" value="">
<br>
    <b>required</b>
<br>
<br>
```

formTextarea宏的工作方式与formInput宏相同，并接受相同的参数列表。 通常，第二个参数（属性）将用于传递文本区域的样式信息或行和列属性。

#### 选择字段

四个选择字段宏可用于在HTML表单中生成常见的UI值选择输入。

* formSingleSelect
* formMultiSelect
* formRadioButtons
* formCheckboxes四个宏中的每一个接受一个包含表单字段的值的选项映射，以及与该值对应的标签。 该值和标签可以相同。FTL中的单选按钮示例如下。 表单后备对象为此字段指定了“伦敦”的默认值，因此不需要进行验证。 当表单呈现时，可以在模型中以“cityMap”的名称提供作为参考数据的整个城市列表。

```
...
Town:
<@spring.formRadioButtons "command.address.town", cityMap, ""/><br><br>
```

这将使用一个单选按钮，一个为cityMap中的每个值使用分隔符“”。 不提供其他属性（缺少宏的最后一个参数）。 cityMap对地图中的每个键值对使用相同的String。 地图的键是表单实际提交的POST请求参数，地图值是用户看到的标签。 在上面的例子中，给出了三个众所周知城市的列表和表单后备对象中的默认值，HTML将是

```
Town:
<input type="radio" name="address.town" value="London">London</input>
<input type="radio" name="address.town" value="Paris" checked="checked">Paris</input>
<input type="radio" name="address.town" value="New York">New York</input>
```

如果您的应用程序期望通过内部代码处理城市，则将使用如下面的示例的合适键创建代码映射。

```
protected Map<String, String> referenceData(HttpServletRequest request) throws Exception {
    Map<String, String> cityMap = new LinkedHashMap<>();
    cityMap.put("LDN", "London");
    cityMap.put("PRS", "Paris");
    cityMap.put("NYC", "New York");

    Map<String, String> model = new HashMap<>();
    model.put("cityMap", cityMap);
    return model;
}
```

代码根据单选框的相关代码产生新的输出，但用户仍然会看到更友好的城市名称。

```
Town:
<input type="radio" name="address.town" value="LDN">London</input>
<input type="radio" name="address.town" value="PRS" checked="checked">Paris</input>
<input type="radio" name="address.town" value="NYC">New York</input>
```

#### HTML转义和符合XHTML标准

上述格式宏的默认使用将导致符合HTML 4.01的HTML标记，并且使用Spring绑定支持使用的web.xml中定义的HTML转义的默认值。 为了使标签符合XHTML或覆盖默认的HTML转义值，您可以在模板中指定两个变量（或者在模型中指定模板中可以看到的变量）。 在模板中指定它们的优点是，它们可以在模板处理中稍后更改为不同的值，以便为表单中的不同字段提供不同的行为。

要为您的标记切换到XHTML符合性，请为名为xhtmlCompliant的模型/上下文变量指定值“true”

```
<#-- for FreeMarker -->
<#assign xhtmlCompliant = true in spring>
```

处理此指令后，Spring宏生成的任何标签现在都符合XHTML标准。

以类似的方式，可以为每个字段指定HTML转义：

```
<#-- until this point, default HTML escaping is used -->

<#assign htmlEscape = true in spring>
<#-- next field will use HTML escaping -->
<@spring.formInput "command.name"/>

<#assign htmlEscape = false in spring>
<#-- all future fields will be bound with HTML escaping off -->
```

## 19.5 JSP & JSTL

Spring为JSP和JSTL视图提供了几个开箱即用的解决方案。 使用JSP或JSTL是使用在WebApplicationContext中定义的常规视图解析器完成的。 此外，当然，您需要编写一些实际渲染视图的JSP。

> 设置应用程序使用JSTL是一个常见的错误源，主要是由于混淆了不同的servlet规范JSP和JSTL版本号，它们的含义和如何正确声明taglibs引起的。 文章
>
> [How to Reference and Use JSTL in your Web Application](http://www.mularien.com/blog/2008/04/24/how-to-reference-and-use-jstl-in-your-web-application/)
>
> 提供了常见的陷阱和如何避免它们的有用指南。 请注意，从Spring 3.0开始，最小支持的servlet版本为2.4（JSP 2.0和JSTL 1.1），这有助于减少混淆的范围。

### 19.5.1 视图解析

就像您与Spring集成的任何其他视图技术一样，对于JSP，您将需要一个视图解析器来解析你的视图。 使用JSP进行开发时最常用的视图解析器是InternalResourceViewResolver和ResourceBundleViewResolver。 两者都在WebApplicationContext中声明：

```
<!-- the ResourceBundleViewResolver -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
    <property name="basename" value="views"/>
</bean>

# And a sample properties file is uses (views.properties in WEB-INF/classes):
welcome.(class)=org.springframework.web.servlet.view.JstlView
welcome.url=/WEB-INF/jsp/welcome.jsp

productList.(class)=org.springframework.web.servlet.view.JstlView
productList.url=/WEB-INF/jsp/productlist.jsp
```

如您所见，ResourceBundleViewResolver需要一个属性文件来定义映射到1）一个类和2个URL的视图名称。 使用ResourceBundleViewResolver，您只能使用一个解析器来混合不同类型的视图。

```
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

InternalResourceBundleViewResolver可以配置为如上所述使用JSP。 作为最佳做法，我们强烈建议您将JSP文件放在“WEB-INF”目录下的目录中，以免客户端直接访问。

### 19.5.2 ‘Plain-old’ JSPs 与JSTL

当使用Java标准标签库时，必须使用特殊的视图类JstlView，因为JSTL需要一些准备工作，例如I18N功能才能正常工作。

### 19.5.3 Additional tags facilitating development

Spring提供了请求参数到命令对象的数据绑定，如前几章所述。 为了方便JSP页面的开发与这些数据绑定功能的结合，Spring提供了一些使事情更容易的标签。 所有Spring标签都具有HTML转义功能，以启用或禁用字符转义。

标签库描述符（TLD）包含在spring-webmvc.jar中。 有关各个标签的更多信息，请参见附录“???”。

### 19.5.4 使用Spring的表单标签库

从版本2.0开始，Spring提供了一套全面的数据绑定感知标签，用于在使用JSP和Spring Web MVC时处理表单元素。 每个标签提供对其对应的HTML标签对应的一组属性的支持，使标签熟悉和直观地使用。 标记生成的HTML符合HTML 4.01 / XHTML 1.0标准。

与其他表单/输入标签库不同，Spring的表单标签库与Spring Web MVC集成，使标签能够访问控制器处理的命令对象和引用数据。 如以下示例所示，表单标签使JSP更易于开发，阅读和维护。

我们来看看表单标签，看一下每个标签的使用方式。 我们已经包括生成的HTML片段，其中某些标签需要进一步的讨论。

#### 配置

表单标签库在spring-webmvc.jar中附带。 库描述符称为spring-form.tld。要使用此库中的标记，请将以下指令添加到JSP页面的顶部：

```
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```

其中form是要用于此库的标签的标签名称前缀。

#### 表单标签

此标记呈现HTML“form”标签，并将绑定路径公开到内部标签以进行绑定。 它将命令对象放在PageContext中，以便可以通过内部标记访问命令对象。 该库中的所有其他标签都是表单标签的嵌套标签。

假设我们有一个名为User的域对象。 它是一个具有诸如firstName和lastName等属性的JavaBean。 我们将使用它作为返回form.jsp的表单控制器的表单支持对象。 下面是一个form.jsp的例子：

```
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

firstName和lastName值由页面控制器从PageContext中放置的命令对象中检索。 继续阅读以查看更多关于内部标签如何与表单标签一起使用的复杂示例。

生成的HTML看起来像一个标准格式：

```
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value="Harry"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value="Potter"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

前面的JSP假定表单后备对象的变量名是'command'。 如果您将表单支持对象放在另一个名称下的模型中（绝对是最佳实践），那么可以将表单绑定到命名变量，如下所示：

```
<form:form modelAttribute="user">
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

#### 输入标签

此标记默认使用绑定值和type =’text’呈现HTML“输入”标签。 有关此标记的示例，请参阅[“The form tag”](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/view.html#view-jsp-formtaglib-formtag)一节。 从Spring 3.1开始，您可以使用其他类型的HTML5特定类型，如“email”，“tel”，“date”等。

#### 勾选标签

此标记会显示一个类型为“checkbox”的HTML“input”标签。让我们假设我们的用户有喜好，如通讯订阅和爱好列表。 以下是Preferences类的示例：

```
public class Preferences {

    private boolean receiveNewsletter;
    private String[] interests;
    private String favouriteWord;

    public boolean isReceiveNewsletter() {
        return receiveNewsletter;
    }

    public void setReceiveNewsletter(boolean receiveNewsletter) {
        this.receiveNewsletter = receiveNewsletter;
    }

    public String[] getInterests() {
        return interests;
    }

    public void setInterests(String[] interests) {
        this.interests = interests;
    }

    public String getFavouriteWord() {
        return favouriteWord;
    }

    public void setFavouriteWord(String favouriteWord) {
        this.favouriteWord = favouriteWord;
    }
}
```

The form.jsp would look like:

```
<form:form>
    <table>
        <tr>
            <td>Subscribe to newsletter?:</td>
            <%-- Approach 1: Property is of type java.lang.Boolean --%>
            <td><form:checkbox path="preferences.receiveNewsletter"/></td>
        </tr>

        <tr>
            <td>Interests:</td>
            <%-- Approach 2: Property is of an array or of type java.util.Collection --%>
            <td>
                Quidditch: <form:checkbox path="preferences.interests" value="Quidditch"/>
                Herbology: <form:checkbox path="preferences.interests" value="Herbology"/>
                Defence Against the Dark Arts: <form:checkbox path="preferences.interests" value="Defence Against the Dark Arts"/>
            </td>
        </tr>

        <tr>
            <td>Favourite Word:</td>
            <%-- Approach 3: Property is of type java.lang.Object --%>
            <td>
                Magic: <form:checkbox path="preferences.favouriteWord" value="Magic"/>
            </td>
        </tr>
    </table>
</form:form>
```

复选框标签有3种方法可以满足您所有的复选框需求。

* 方法一 – 当绑定值的类型为java.lang.Boolean时，如果绑定值为true，则输入（复选框）将被标记为“已检查”。 valueattribute对应于setValue（Object）value属性的已解析值。
* 方法二 – 当绑定值为array或java.util.Collection类型时，如果配置的setValue（Object）值存在于绑定集合中，则输入（复选框）将被标记为“checked”.
* 方法三 – 对于任何其他绑定值类型，如果配置的setValue（Object）等于绑定值，则输入（复选框）将被标记为“已检查”。

请注意，无论采用何种方法，都会生成相同的HTML结构。 以下是某些复选框的HTML片段：

```
<tr>
    <td>Interests:</td>
    <td>
        Quidditch: <input name="preferences.interests" type="checkbox" value="Quidditch"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Herbology: <input name="preferences.interests" type="checkbox" value="Herbology"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Defence Against the Dark Arts: <input name="preferences.interests" type="checkbox" value="Defence Against the Dark Arts"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
    </td>
</tr>
```

您可能不会看到的是每个复选框后面的额外的隐藏字段。 当HTML页面中的复选框未被选中时，一旦表单被提交，它的值就不会作为HTTP请求参数的一部分发送到服务器，所以我们需要一个解决方法来为HTML解决这个问题，以便Spring表单数据绑定 工作。 复选框标记遵循现有的Spring约定，为每个复选框添加一个前缀为下划线（“\_”）的隐藏参数。 通过这样做，您正在有效地告诉Spring，“复选框在表单中可见，并且我希望我的对象将窗体数据绑定到其中，以反映该复选框的状态，无论什么”。

#### 复选框标记

此标记会使用“checkbox”类型呈现多个HTML“input”标签。

基于上一个复选框标记部分的示例。 有时您不希望在JSP页面中列出所有可能的选项。 您希望在运行时提供可用选项的列表，并将其传递给标签。 这就是复选框标签的目的。 您传递一个数组，列表或地图，其中包含“items”属性中的可用选项。 通常，bound属性是一个集合，因此它可以保存用户选择的多个值。 以下是使用此标记的JSP的示例：

```
<form:form>
    <table>
        <tr>
            <td>Interests:</td>
            <td>
                <%-- Property is of an array or of type java.util.Collection --%>
                <form:checkboxes path="preferences.interests" items="${interestList}"/>
            </td>
        </tr>
    </table>
</form:form>
```

此示例假定“兴趣列表”是可用作包含要从中选择的值的字符串的模型属性的列表。 在使用地图的情况下，将使用地图条目键作为值，并将地图条目的值用作要显示的标签。 您还可以使用自定义对象，您可以在其中使用“itemValue”提供值的属性名称，并使用“itemLabel”提供标签。

#### 单选标记

此标记呈现类型为“radio”的HTML“input”标签。典型的使用模式将涉及绑定到相同属性但具有不同值的多个标签实例。

```
<tr>
    <td>Sex:</td>
    <td>
        Male: <form:radiobutton path="sex" value="M"/> <br/>
        Female: <form:radiobutton path="sex" value="F"/>
    </td>
</tr>
```

#### 多个单选框标签

此标签使用“radio”类型呈现多个HTML“input”标签。

就像上面的复选框标签一样，您可能希望将可用选项作为运行时变量传递。 对于这种用法，您将使用radiobuttons标签。 您传递一个数组，列表或地图，其中包含“items”属性中的可用选项。 在使用地图的情况下，将使用地图条目键作为值，并将地图条目的值用作要显示的标签。 您还可以使用自定义对象，您可以在其中使用“itemValue”提供值的属性名称，并使用“itemLabel”提供标签。

```
<tr>
    <td>Sex:</td>
    <td><form:radiobuttons path="sex" items="${sexOptions}"/></td>
</tr>
```

#### 密码标签

此标记使用绑定值呈现类型为“password”的HTML“input”标签。

```
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password"/>
    </td>
</tr>
```

请注意，默认情况下，密码值未显示。 如果您想要显示密码值，则将“showPassword”属性的值设置为true，如此。

```
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password" value="^76525bvHGq" showPassword="true"/>
    </td>
</tr>
```

#### 选择标签

此标记呈现HTML“选择”元素。 它支持与选定选项的数据绑定以及嵌套选项和选项标签的使用。

假设用户有一个技能列表。

```
<tr>
    <td>Skills:</td>
    <td><form:select path="skills" items="${skills}"/></td>
</tr>
```

如果用户的技能在Herbology，“技能”行的HTML源代码将如下所示：

```
<tr>
    <td>Skills:</td>
    <td>
        <select name="skills" multiple="true">
            <option value="Potions">Potions</option>
            <option value="Herbology" selected="selected">Herbology</option>
            <option value="Quidditch">Quidditch</option>
        </select>
    </td>
</tr>
```

#### 可选 标签

此标记呈现HTML“选项”。 它根据绑定值适当地设置“选择”。

```
<tr>
    <td>House:</td>
    <td>
        <form:select path="house">
            <form:option value="Gryffindor"/>
            <form:option value="Hufflepuff"/>
            <form:option value="Ravenclaw"/>
            <form:option value="Slytherin"/>
        </form:select>
    </td>
</tr>
```

如果用户的房子在Gryffindor，“房子”行的HTML源代码将如下所示：

```
<tr>
    <td>House:</td>
    <td>
        <select name="house">
            <option value="Gryffindor" selected="selected">Gryffindor</option>
            <option value="Hufflepuff">Hufflepuff</option>
            <option value="Ravenclaw">Ravenclaw</option>
            <option value="Slytherin">Slytherin</option>
        </select>
    </td>
</tr>
```

#### 多个可选标签

此标记呈现HTML“option”标签的列表。 它根据绑定值适当地设置'selected'属性。

```
<tr>
    <td>Country:</td>
    <td>
        <form:select path="country">
            <form:option value="-" label="--Please Select"/>
            <form:options items="${countryList}" itemValue="code" itemLabel="name"/>
        </form:select>
    </td>
</tr>
```

如果用户居住在英国，“Country”行的HTML源代码将如下所示：

```
<tr>
    <td>Country:</td>
    <td>
        <select name="country">
            <option value="-">--Please Select</option>
            <option value="AT">Austria</option>
            <option value="UK" selected="selected">United Kingdom</option>
            <option value="US">United States</option>
        </select>
    </td>
</tr>
```

如示例所示，选项标签与选项标签的组合使用将生成相同的标准HTML，但允许您显式指定JSP中仅用于显示（在其所在的位置）的值，例如 示例：“ - 请选择”。

items属性通常用一组或多个项目对象填充。 itemValue和itemLabel只是指定这些项目对象的bean属性; 否则，项目对象本身将被字符串化。 或者，您可以指定项目地图，在这种情况下，地图键将被解释为选项值，地图值对应于选项标签。 如果同时指定了itemValue和/或itemLabel，则item value属性将应用于map键，item label属性将应用于map值。

#### textarea标签

此标记呈现HTML“textarea”。

```
 <tr>
    <td>Notes:</td>
    <td><form:textarea path="notes" rows="3" cols="20"/></td>
    <td><form:errors path="notes"/></td>
</tr>
```

#### hidden 标签

此标记使用绑定值呈现类型为“hidden”的HTML“input”标签。 要提交未绑定的隐藏值，请使用类型为“hidden”的HTML输入标签。

&lt;form：hidden path =“house”/&gt;  
如果我们选择将“房屋”值提交为隐藏值，则HTML将如下所示：

```
<input name="house" type="hidden" value="Gryffindor"/>
```

#### errors 标签

此标记会在HTML’span’标签中呈现字段错误。 它提供对控制器中创建的错误的访问或由与控制器相关联的任何验证器创建的错误。

假设我们要在提交表单时显示firstName和lastName字段的所有错误消息。 我们有一个名为UserValidator的User类的实例的验证器。

```
public class UserValidator implements Validator {

    public boolean supports(Class candidate) {
        return User.class.isAssignableFrom(candidate);
    }

    public void validate(Object obj, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "required", "Field is required.");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "lastName", "required", "Field is required.");
    }
}
```

form.jsp如下所示：

```
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <%-- Show errors for firstName field --%>
            <td><form:errors path="firstName"/></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <%-- Show errors for lastName field --%>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

如果我们在firstName和lastName字段中提交一个带有空值的表单，这就是HTML的样子：

```
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <%-- Associated errors to firstName field displayed --%>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <%-- Associated errors to lastName field displayed --%>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

如果我们要显示给定页面的整个错误列表怎么办？ 下面的示例显示了errors标签还支持一些基本的通配符功能。

* `path="*"`– 显示所有的错误
* `path="lastName"`– 显示与`lastName` 相关的错误
* `假如path`省略- 仅显示对象错误

下面的示例将显示页面顶部的错误列表，后跟字段旁边的字段特定错误:

```
<form:form>
    <form:errors path="*" cssClass="errorBox"/>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <td><form:errors path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

HTML是这样的：

```
<form method="POST">
    <span name="*.errors" class="errorBox">Field is required.<br/>Field is required.</span>
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
       </table>
</form>
```

#### HTTP 方法转变

REST的一个关键原则是使用统一接口。这意味着可以使用相同的四种HTTP方法来操作所有资源（URL）：GET，PUT，POST和DELETE。对于每个方法，HTTP规范定义了准确的语义。例如，GET应该永远是一个安全的操作，这意味着没有副作用，PUT或DELETE应该是幂等的，这意味着你可以一遍又一遍地重复这些操作，但最终结果应该是一样的。虽然HTTP定义了这四种方法，但HTML只支持两种方法：GET和POST。幸运的是，有两种可能的解决方法：您可以使用JavaScript来执行您的PUT或DELETE，或者使用“real”方法作为附加参数（在HTML表单中建模为隐藏输入字段）进行POST。后一个技巧是Spring的HiddenHttpMethodFilter所做的。此过滤器是一个简单的Servlet过滤器，因此可以与任何Web框架（不仅仅是Spring MVC）结合使用。只需将此过滤器添加到您的web.xml中，并将带有隐藏\_method参数的POST转换为相应的HTTP方法请求。

为了支持HTTP方法转换，Spring MVC表单标记已更新，以支持设置HTTP方法。例如，从更新的Petclinic示例中获取以下代码段

```
<form:form method="delete">
    <p class="submit"><input type="submit" value="Delete Pet"/></p>
</form:form>
```

这将实际执行HTTP POST，隐藏在请求参数之后的“real”DELETE方法，由HiddenHttpMethodFilter拾取，如web.xml中所定义：

```
<filter>
    <filter-name>httpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpMethodFilter</filter-name>
    <servlet-name>petclinic</servlet-name>
</filter-mapping>
```

相应的@Controller方法如下所示：

```
@RequestMapping(method = RequestMethod.DELETE)
public String deletePet(@PathVariable int ownerId, @PathVariable int petId) {
    this.clinic.deletePet(petId);
    return "redirect:/owners/" + ownerId;
}
```

#### HTML5 标签

从Spring 3开始，Spring表单标签库允许输入动态属性，这意味着您可以输入任何特定于HTML5的属性。

在Spring 3.1中，表单输入标签支持输入“text”以外的类型属性。 这旨在允许渲染新的HTML5特定输入类型，如 ’email’, ‘date’, ‘range’等。 请注意，输入type =’text’不是必需的，因为’text’是默认类型。

## 19.6 Script 模板

可以使用Spring将任何在JSR-223脚本引擎上运行的模板库集成到Web应用程序中。以下以广泛的方式描述如何做到这一点。脚本引擎必须实现ScriptEngine和Invocable接口。

已通过以下测试：

* H
  andlebars running on Nashorn
* Mustache running on Nashorn
* React running on Nashorn
* EJS running on Nashorn
* ERB running on JRuby
* String templates running on Jython

### 19.6.1依赖关系

为了能够使用脚本模板集成，您需要在您的类路径中提供脚本引擎：

* Nashorn Javascript引擎内置Java 8+。强烈建议使用最新的更新版本。
* Rhino Javascript引擎内建Java 6和Java 7.请注意，不建议使用Rhino，因为它不支持运行大多数模板引擎。
* 应该添加JRuby依赖以获得Ruby的支持。
* 应该添加Jython依赖关系，以获得Python的支持。

您还需要为基于脚本的模板引擎添加依赖关系。例如，对于Javascript，您可以使用WebJars添加Maven / Gradle依赖关系，以使您的javascript库在类路径中可用。

### 19.6.2如何集成基于脚本的模板

为了能够使用脚本模板，您必须对其进行配置，以便指定各种参数，如要使用的脚本引擎，要加载的脚本文件以及应该调用哪些函数来呈现模板。这是由于ScriptTemplateConfigurer bean和可选的脚本文件。

例如，为了渲染Mustache模板，感谢Java 8+提供的Nashorn Javascript引擎，您应该声明以下配置：

```
@Configuration
@EnableWebMvc
public class MustacheConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("mustache.js");
        configurer.setRenderObject("Mustache");
        configurer.setRenderFunction("render");
        return configurer;
    }
}
```

XML 表示如下:

```
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:script-template/>
</mvc:view-resolvers>

<mvc:script-template-configurer engine-name="nashorn" render-object="Mustache" render-function="render">
    <mvc:script location="mustache.js"/>
</mvc:script-template-configurer>
```

你期望的controller：

```
@Controller
public class SampleController {

    @RequestMapping
    public ModelAndView test() {
        ModelAndView mav  = new ModelAndView();
        mav.addObject("title", "Sample title").addObject("body", "Sample body");
        mav.setViewName("template.html");
        return mav;
    }
}
```

而Mustache模板是：

```
<html>
    <head>
        <title>{{title}}</title>
    </head>
    <body>
        <p>{{body}}</p>
    </body>
</html>
```

使用以下参数调用render函数：

* 字符串模板：模板内容
* 地图模型：视图模型
* String url：模板url（自4.2.2起）

Mustache.render（）与此签名本身兼容，因此您可以直接调用它。

如果您的模板技术需要一些自定义，您可以提供一个实现自定义渲染功能的脚本。例如，Handlerbars需要在使用它们之前编译模板，并且需要一个polyfill才能模拟服务器端脚本引擎中不可用的一些浏览器设施。

```
@Configuration
@EnableWebMvc
public class MustacheConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("polyfill.js", "handlebars.js", "render.js");
        configurer.setRenderFunction("render");
        configurer.setSharedEngine(false);
        return configurer;
    }
}
```

> \[注意\]
>
> 如果使用非线程安全的脚本引擎，而不使用非并行设计的模板库（例如，在Nashorn上运行的Handlebars或React），则需要将sharedEngine属性设置为false。在这种情况下，由于这个错误，需要Java 8u60或更高版本。

polyfill.js仅定义Handlebars需要正确运行的窗口对象：

```
var window = {};
```

这个基本的render.js实现在使用之前编译模板。生产就绪实现还应该存储和重新使用缓存的模板/预编译模板。这可以在脚本端完成，以及您需要的任何定制（例如，管理模板引擎配置）。

```
function render(template, model) {
    var compiledTemplate = Handlebars.compile(template);
    return compiledTemplate(model);
}
```

## 19.7 XML编组视图

MarshallingView使用org.springframework.oxm包中定义的XML Marshaller将响应内容呈现为XML。 要编组的对象可以使用MarhsallingView的\`modelKey bean属性显式设置。 或者，视图将遍历所有模型属性并编组Marshaller支持的第一个类型。 有关org.springframework.oxm软件包中的功能的更多信息，请参阅使用O / X Mappers编组XML的章节。

## 19.8 Tiles

在使用Spring的Web应用程序中，可以将Tiles 与任何其他视图技术集成在一起。 以下以广泛的方式描述如何做到这一点。

### 19.8.1 依赖

为了可以使用Tiles，您必须添加对Tiles 3.0.1版或更高版本的依赖关系，以及其对您的项目的传递依赖性。

### 19.8.2 怎样集成 Tiles

为了使用Tiles，你需要使用文件定义 配置 \(对于基本定义和其他Tiles内容，请查看[http://tiles.apache.org](https://tiles.apache.org/)\). 在Spring 是使用 `TilesConfigurer`. 请看下面的例子:

```
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/general.xml</value>
            <value>/WEB-INF/defs/widgets.xml</value>
            <value>/WEB-INF/defs/administrator.xml</value>
            <value>/WEB-INF/defs/customer.xml</value>
            <value>/WEB-INF/defs/templates.xml</value>
        </list>
    </property>
</bean>
```

您可以看到，有五个文件包含定义，它们都位于“WEB-INF / defs”目录中。 在WebApplicationContext初始化时，将加载文件，并初始化定义工厂。 之后，这个定义文件中的Tile可以在Spring Web应用程序中用作视图。 为了能够使用视图，您必须拥有一个ViewResolver，就像Spring使用的任何其他视图技术一样。 下面你可以找到两种可能性：UrlBasedViewResolver和ResourceBundleViewResolver。

您可以通过添加下划线，然后添加区域设置来指定特定于区域设置的瓷砖定义。 例如：

```
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/tiles.xml</value>
            <value>/WEB-INF/defs/tiles_fr_FR.xml</value>
        </list>
    </property>
</bean>
```

使用此配置，tiles\_fr\_FR.xml将用于具有fr\_FR语言环境的请求，默认情况下将使用tiles.xml。

#### UrlBasedViewResolver

UrlBasedViewResolver为给定的viewClass实例化其必须解析的每个视图。

```
<bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.tiles3.TilesView"/>
</bean>
```

#### ResourceBundleViewResolver

ResourceBundleViewResolver必须提供一个属性文件，其中包含解析器可以使用的viewnames和viewclasses：

```
<bean id="viewResolver" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
    <property name="basename" value="views"/>
</bean>
```

```
...
welcomeView.(class)=org.springframework.web.servlet.view.tiles3.TilesView
welcomeView.url=welcome (this is the name of a Tiles definition)

vetsView.(class)=org.springframework.web.servlet.view.tiles3.TilesView
vetsView.url=vetsView (again, this is the name of a Tiles definition)

findOwnersForm.(class)=org.springframework.web.servlet.view.JstlView
findOwnersForm.url=/WEB-INF/jsp/findOwners.jsp
...
```

您可以看到，当使用ResourceBundleViewResolver时，您可以轻松地混合不同的视图技术。

请注意，TilesView类支持JSTL（JSP标准标记库）。

#### SimpleSpringPreparerFactory and SpringBeanPreparerFactory

作为高级功能，Spring还支持两种特殊的Tiles PreparerFactory实现。有关如何在您的Tiles定义文件中使用ViewPreparer引用的详细信息，请查看Tiles文档。

指定SimpleSpringPreparerFactory根据指定的preparer类自动导航ViewPreparer实例，应用Spring的容器回调以及应用配置的Spring BeanPostProcessors。如果Spring的上下文范围注释配置已激活，ViewPreparer类中的注释将被自动检测并应用。请注意，这需要在Tiles定义文件中的preparer类，就像默认的PreparerFactory一样。

指定SpringBeanPreparerFactory对指定的prepareer名称而不是类进行操作，从DispatcherServlet的应用程序上下文中获取相应的Spring bean。在这种情况下，完整的bean创建过程将控制Spring应用程序上下文，允许使用显式依赖注入配置，范围bean等。请注意，您需要为每个preparer名称定义一个Spring bean定义（如你的Tiles定义）。

```
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/general.xml</value>
            <value>/WEB-INF/defs/widgets.xml</value>
            <value>/WEB-INF/defs/administrator.xml</value>
            <value>/WEB-INF/defs/customer.xml</value>
            <value>/WEB-INF/defs/templates.xml</value>
        </list>
    </property>

    <!-- resolving preparer names as Spring bean definition names -->
    <property name="preparerFactoryClass"
            value="org.springframework.web.servlet.view.tiles3.SpringBeanPreparerFactory"/>

</bean>
```



