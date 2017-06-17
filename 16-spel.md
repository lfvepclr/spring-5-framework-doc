## **6.1 介绍**

Spring Expression Language（简称SpEL）是一种功能强大的表达式语言、用于在运行时查询和操作对象图；语法上类似于Unified EL，但提供了更多的特性，特别是方法调用和基本字符串模板函数。

虽然目前已经有许多其他的Java表达式语言，例如OGNL，MVEL和Jboss EL，SpEL的诞生是为了给Spring社区提供一种能够与Spring生态系统所有产品无缝对接，能提供一站式支持的表达式语言。它的语言特性由Spring生态系统的实际项目需求驱动而来，比如基于eclipse的Spring Tool Suite（Spring开发工具集）中的代码补全工具需求。尽管如此、SpEL本身基于一套与具体实现技术无关的API，在需要的时候允许其他的表达式语言实现集成进来。

尽管SpEL在Spring产品中是作为表达式求值的核心基础模块，本身可以脱离Spring独立使用。为了体现它的独立性，本章节中的许多例子都将SpEL作为独立的表达式语言来使用。不过这样就需要每次都先创建一些基础框架类如解析器，而对于大多数Spring用户来说并不需要去关注这些基础框架类，仅仅只需要写相应的字符串求值表达式即可。一个典型的例子就是把SpEL集成进XML bean配置或者基于注解的Bean定义声明中（详见章节：[Expression support for defining bean definitions](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/expressions.html#expressions-beandef)）

本章节包含SpEL的语言特性，它的API及语法。很多地方用到了Inventor类及相关的Society类作为表达式求值的操作例子对象，这几个类的定义及操作它们的数据都列在本章的末尾.

## **6.2 功能特性**

SpEL支持以下的一些特性：

* 字符表达式
* 布尔和关系操作符
* 正则表达式
* 类表达式
* 访问properties，arrays，lists，maps等集合
* 方法调用
* 关系操作符
* 赋值
* 调用构造器
* Bean对象引用
* 创建数组
* 内联lists
* 内联maps
* 三元操作符
* 变量
* 用户自定义函数
* 集合投影
* 集合选择
* 模板表达式

## **6.3 使用SpEL的接口进行表达式求值**

本节介绍SpEL接口及其表达式语言的简单使用方法。完整的语言文档见：[Language Reference](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/html/expressions.html#expressions-language-ref)

下面代码介绍了使用SpEL API来解析字符串表达式’Hello World’的示例

```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");
String message = (String) exp.getValue();
```

最常用的SpEL类和接口都放在包org.springframework.expression及其子包和spel.support下

接口ExpressionParser用来解析一个字符串表达式。在这个例子中字符串表达式即用单引号括起来的字符串.接口Expression用于对上面定义的字符串表达式求值。 调用parser.parseExpression和exp.getValue分别可能抛出ParseException和EvaluationException。

SpEL支持一系列广泛的特性，例如方法调用，访问属性，调用构造函数等。

下面举一个方法调用的例子，在String文本后面调用concat方法。

```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')");
String message = (String) exp.getValue();
```

message的值现在就是 ‘Hello World!’.

接下去是一个访问JavaBean属性的例子，String类的Bytes属性通过以下的方法调用

```
// 调用'getBytes()'
Expression exp = parser.parseExpression("'Hello World'.bytes");
byte[] bytes = (byte[]) exp.getValue();
```

SpEL同时也支持级联属性调用、和标准的prop1.prop2.prop3方式是一样的；同样属性值设置也是类似的方式.

公共方法也可以被访问到：

```
ExpressionParser parser = new SpelExpressionParser();

// 调用 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length");
int length = (Integer) exp.getValue();
```

除了使用字符串表达式、也可以调用String的构造函数：

```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()");
String message = exp.getValue(String.class);
```

针对泛型方法的使用，例如：public &lt;T&gt; T getValue\(Class&lt;T&gt; desiredResultType\) 使用这样的方法不需要将表达式的值转换成具体的结果类型。如果具体的值类型或者使用类型转换器都无法转成对应的类型、会抛EvaluationException的异常

SpEL中更常见的用途是提供一个针对特定对象实例（叫做根对象）求值的表达式字符串。使用方法有两种，  
具体用哪一种要看每次调用表达式求值时相应的对象实例是否每次都会变化。接下来我们分别举两个例子说明，  
第一个例子我们要做的是从Inventor类的实例中解析name属性：

```
// 创建并设置一个calendar实例
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// 构造器参数有： name, birthday, and nationality.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name");

EvaluationContext context = new StandardEvaluationContext(tesla);
String name = (String) exp.getValue(context);
```

最后一行，字符串变量name将会被设置成”Nikola Tesla”. 通过StandardEvaluationContext类你能指定哪个对象的”name”会被求值这种机制用于当根对象不会被改变的场景，在求值上下文中只会被设置一次。相反，如果根对象会经常改变，则需要在每次调用getValue的时候被设置，就像如下的示例代码：

```
// 创建并设置一个calendar实例
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// 构造器参数有： name, birthday, and nationality.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("name");
String name = (String) exp.getValue(tesla);
```

在上面这个例子中inventor类实例tesla直接在getValue中设置，表达式解析器底层框架会自动创建和管理一个默认的求值上下文-不需要被显式声明

StandardEvaluationContext创建相对比较耗资源，在重复多次使用的场景下内部会缓存部分中间状态加快后续的表达式求值效率。因此建议在使用过程中尽可能被缓存和重用，而不是每次在表达式求值时都重新创建一个对象。

在很多使用场景下理想的方式是事先配置好求值上下文，但是在实际调用中仍然可以给getValue设置一个不同的根对象。getValue允许在同一个调用中同时指定两者，在这种场景下运行时传入的根对象会覆盖在求值上下文中事先指定的根对象。

> Note：在单独使用SpEL时需要创建解析器、解析表达式、以及求值上下文和对应的根对象。但是在实际使用过程中、更常用的使用方式是只需要在配置文件里面配置SpEL字符串表达式即可，例如针对Spring Bean或者Spring Web Flow的定义。在这种场景下解析器，求值上下文，根对象和任何事先定义的变量都会被容器默认创建好，用户除了写表达式不需要做任何其他事情。

作为最后一个介绍性的例子，我们沿用上面的例子写一个使用布尔操作符的求值例子

```
Expression exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(context, Boolean.class); // 求值结果是True
```

### **6.3.1 EvaluationContext接口**

EvaluationContext接口在求值表达式中需要解析属性，方法，字段的值以及类型转换中会用到。其默认实现类StandardEvaluationContext使用反射机制来操作对象。为获得更好的性能缓存了`java.lang.reflect.Method`,`java.lang.reflect.Field`, 和`java.lang.reflect.Constructor实例`。

StandardEvaluationContext中你可以使用setRootObject\(\)方法显式设置根对象，或通过构造器直接传入根对象。你还可以通过调用setVariable\(\)和registerFunction\(\)方法指定在表达式中用到的变量和函数。变量和函数的使用在语言参考的 Variables和Functions两章节有详细说明。使用StandardEvaluationContext你还可以注册自定义的构造器解析器（ConstructorResolvers）,方法解析器（MethodResolvers）,和属性存取器（PropertyAccessor）来扩展SpEL如何计算表达式，详见具体类的JavaDoc文档。

**类型转换**

SpEL默认使用Spring核心代码中的conversion service来做类型转换（org.springframework.core.convert.ConversionService）。这个类本身内置了很多常用的转换器，同时也可以扩展使用自定义的类型转换器。另外一个核心功能是它可以识别泛型。这意味着当在表达式中使用泛型类型时、SpEL会确保任何处理的对象的类型正确性。

实际应用中这意味着什么？这里举一个例子、拿赋值来说，比如使用setValue来设置List属性。属性的类型实际上是List&lt;Boolean&gt;，SpEL可以识别List中的元素类型并转换成Boolean类型。下面是示例代码：

```
class Simple {
    public List<Boolean> booleanList = new ArrayList<Boolean>();
}

Simple simple = new Simple();

simple.booleanList.add(true);

StandardEvaluationContext simpleContext = new StandardEvaluationContext(simple);

// false is passed in here as a string. SpEL and the conversion service will
// correctly recognize that it needs to be a Boolean and convert it
parser.parseExpression("booleanList[0]").setValue(simpleContext, "false");

// b will be false
Boolean b = simple.booleanList.get(0);
```

### **6.3.2 解析器配置**

可以通过使用一个解析器配置对象 \(org.springframework.expression.spel.SpelParserConfiguration\)来配置SpEL表达式解析器。这个配置对象可以控制一些表达式组件的行为。例如：数组或者集合元素查找的时候如果当前位置对应的对象是Null，可以通过事先配置来自动创建元素。这个在表达式是由多次属性链式引用的时候比较重要。如果设置的数组或者List位置越界时可以自动增加数组或者List长度来兼容.

```
class Demo {
    public List<String> list;
}

// Turn on:
// - auto null reference initialization
// - auto collection growing
SpelParserConfiguration config = new SpelParserConfiguration(true,true);

ExpressionParser parser = new SpelExpressionParser(config);

Expression expression = parser.parseExpression("list[3]");

Demo demo = new Demo();

Object o = expression.getValue(demo);

// demo.list will now be a real collection of 4 entries
// Each entry is a new empty String
```

SpEL表达式编译器的行为也可以通过配置来实现

### **6.3.3 SpEL编译**

Spring4.1 包括了一个基本的表达式编译器。表达式求值过程中往往可以利用动态解释功能来提供很多灵活性，但是却达不到理想的性能。当表达式不是被经常使用时是可以接受的，但是当被其他组件像Spring Integration使用时，性能往往更重要，这时动态性没有太多的需求。

新的SpEL编译器针对这样的需求进行了定制。编译器会在求值时同时创建一个真实的Java类包含表达式的行为，利用这种方式来达到更快的表达式求值速度。在编译过程中、编译器刚开始不知道表达式具体的类型、但它会在表达式求值过程中收集相应的信息来确定最终的类型。例如，编译器无法仅仅从表达式自身来分析出某个属性引用的类型，但是在首次进行解释求值时就可以确定了。当然，如果不同的表达式元素类型后续发生变化、基于这种信息来编译结果就不准确了。因此编译最适合的场景是表达式在多次求值过程中类型信息不会变化。

例如一个简单的表达式如下：

```
someArray[0].someProperty.someOtherProperty < 0.1
```

这个表达式包含数组访问，属性引用和数值运算，其性能开销不可忽视。  
在一次50000次循环的性能基准评测中，如果只用解析器需要耗时75毫秒，但如果使用已编译的版本则只需要3毫秒

#### **编译器配置**

编译器模式不是默认打开的，有两种方法可以打开。一种是前面已经提到过的解析器配置的时候，另一种是当SpEL集成到其他组件时通过设置系统属性的方式打开。本节会同时介绍两种方式.

首先比较重要的是编译器本身有几种操作模式，都定义在枚举类\(org.springframework.expression.spel.SpelCompilerMode\)中。所有的模式如下：

**OFF**-编译器关闭；默认是关闭的  
**IMMEDIATE**-即时生效模式，表达式会尽快的被编译。基本是在第一次求值后马上就会执行。如果编译表达式出错（往往都是因为上面提到的类型发生改变的情况）则表达式求值的调用点会抛出异常。  
**MIXED**-混合模式，在混合模式中表达式会自动在解释器模式和编译器模式之间切换。在发生了几次解释处理后会切换到编译模式，如果编译模式哪里出错了（像上面提到的类型发生变化）则表达式会自动切换回解释器模式。过一段时间如果运用正常又会切换回编译模式。基本上像在IMMEDIATE模式下会抛出的那些异常都会被内部处理掉。

即时生效模式之所以会存在是因为混合模式会带来副作用。如果一个已编译的表达式在部分执行成功后发生错误的话，有可能已经导致系统的状态发生变化。这种场景下调用方并不希望表达式在解释模式下重新执行一遍、因为这意味着部分表达式会被执行两遍。

在选择了一个模式后，使用SpelParserConfiguration 来配置解释器：

```
SpelParserConfiguration config = new SpelParserConfiguration(SpelCompilerMode.IMMEDIATE,
    this.getClass().getClassLoader());

SpelExpressionParser parser = new SpelExpressionParser(config);

Expression expr = parser.parseExpression("payload");

MyMessage message = new MyMessage();

Object payload = expr.getValue(message);
```

指定编译器类型时也可以同时指定类加载器\(不指定传入Null也是允许的\)。已编译的表达式将会被定义在所有被创建的子类加载器中。比较重要的一点是一旦指定了一个类加载器、需要确保表达式求值过程中所有涉及的类型对它都是可见的。如果没有明确指定则会使用缺省的类加载器（一般是当前正在执行表达式求值的线程所关联的上下文类加载器）

第二种方法是当SpEL内嵌到其他组件时仅通过配置对象不太容易配置实现的时候使用。在这种场景下往往使用系统属性来设置。属性spring.expression.compiler.mode可以被设置成SpelCompilerMode枚举值的其中一种（off, immediate, 或者 mixed）。

#### **编译器的局限性**

在Srping4.1中基础的编译框架就已经有了。但框架还不支持编译所有类型的表达式。最初的设计初衷主要在于先集中优化比较耗性能且又经常使用的表达式。以下类型的表达式目前还不能被编译：

涉及到赋值的表达式  
依赖于转换服务的表达式  
使用到自定义解析器或者存取器的表达式  
使用到选择器或者投影的表达式

更多类型的表达式将来都会被支持编译

## **6.4 Bean定义时使用表达式**

无论XML还是注解类型的Bean定义都可以使用SpEL表达式。在两种方式下定义的表达式语法都是一样的，即：\#{ }

### **6.4.1 XML类型的配置**

Bean属性或者构造函数使用表达式的方式如下：

```
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>
```

在下面的例子中systemProperties 事先已被定义好，因此表达式中可以直接使用。注意：在已定义的变量前无需加\#

```
<bean id="taxCalculator" class="org.spring.samples.TaxCalculator">
    <property name="defaultLocale" value="#{ systemProperties['user.region'] }"/>

    <!-- other properties -->
</bean>
```

你还可以通过Name注入的方式使用其他Bean的属性，例如：

```
<bean id="numberGuess" class="org.spring.samples.NumberGuess">
    <property name="randomNumber" value="#{ T(java.lang.Math).random() * 100.0 }"/>

    <!-- other properties -->
</bean>

<bean id="shapeGuess" class="org.spring.samples.ShapeGuess">
    <property name="initialShapeSeed" value="#{ numberGuess.randomNumber }"/>

    <!-- other properties -->
</bean>
```

**6.4.2 基于注解的配置**  
@Value可以在属性字段，方法和构造器变量中使用，指定一个默认值。

下面的例子中给属性字段设置默认值：

```
public static class FieldValueTestBean

    @Value("#{ systemProperties['user.region'] }")
    private String defaultLocale;

    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public String getDefaultLocale() {
        return this.defaultLocale;
    }

}
```

通过Set方法设置默认值：

```
public static class PropertyValueTestBean

    private String defaultLocale;

    @Value("#{ systemProperties['user.region'] }")
    public void setDefaultLocale(String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }

    public String getDefaultLocale() {
        return this.defaultLocale;
    }

}
```

使用Autowired注解的方法和构造器也可以使用@Value注解.

```
public class SimpleMovieLister {

    private MovieFinder movieFinder;
    private String defaultLocale;

    @Autowired
    public void configure(MovieFinder movieFinder,
            @Value("#{ systemProperties['user.region'] }") String defaultLocale) {
        this.movieFinder = movieFinder;
        this.defaultLocale = defaultLocale;
    }

    // ...
}
```

```
public class MovieRecommender {

    private String defaultLocale;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao,
            @Value("#{systemProperties['user.country']}") String defaultLocale) {
        this.customerPreferenceDao = customerPreferenceDao;
        this.defaultLocale = defaultLocale;
    }

    // ...
}
```

## **6.5 语言参考**

### **6.5.1 字符表达式**

字符串表达式类型支持strings,数值类型 \(int, real, hex\),布尔和null.strings值通过单引号引用。如果字符串里面又包含字符串，通过双引号引用。

下面列出了字符串表达式的常用例子。通常它们不会被单独使用，而是结合一个更复杂的表达式一起使用，例如，在逻辑比较运算符中使用表达式：

```
ExpressionParser parser = new SpelExpressionParser();

// evals to "Hello World"
String helloWorld = (String) parser.parseExpression("'Hello World'").getValue();

double avogadrosNumber = (Double) parser.parseExpression("6.0221415E+23").getValue();

// evals to 2147483647
int maxValue = (Integer) parser.parseExpression("0x7FFFFFFF").getValue();

boolean trueValue = (Boolean) parser.parseExpression("true").getValue();

Object nullValue = parser.parseExpression("null").getValue();
```

数字类型支持负数，指数和小数点。默认情况下实数会使用Double.parseDouble\(\)解析。

### **6.5.2 属性, 数组, 列表, Maps, 索引**

属性引用比较简单：只需要用点号\(.\)标识级联的各个属性值。Inventor类的实例：pupin,和tesla，所用到的数据在Classes used in the examples一节有列出。  
下面的表达式示例用来解析Tesla的出生年及Pupin的出生城市。

```
// evals to 1856
int year = (Integer) parser.parseExpression("Birthdate.Year + 1900").getValue(context);

String city = (String) parser.parseExpression("placeOfBirth.City").getValue(context);
```

属性名的第一个字母可以是大小写敏感的。数组和列表的内容可以使用方括号来标记

```
ExpressionParser parser = new SpelExpressionParser();

// Inventions Array
StandardEvaluationContext teslaContext = new StandardEvaluationContext(tesla);

// evaluates to "Induction motor"
String invention = parser.parseExpression("inventions[3]").getValue(
        teslaContext, String.class);

// Members List
StandardEvaluationContext societyContext = new StandardEvaluationContext(ieee);

// evaluates to "Nikola Tesla"
String name = parser.parseExpression("Members[0].Name").getValue(
        societyContext, String.class);

// List and Array navigation
// evaluates to "Wireless communication"
String invention = parser.parseExpression("Members[0].Inventions[6]").getValue(
        societyContext, String.class);
```

Maps的值由方括号内指定字符串的Key来标识引用。在下面这个例子中，因为Officers map的Key是string类型，我们可以用过字符串常量指定。

```
// Officer's Dictionary

Inventor pupin = parser.parseExpression("Officers['president']").getValue(
        societyContext, Inventor.class);

// evaluates to "Idvor"
String city = parser.parseExpression("Officers['president'].PlaceOfBirth.City").getValue(
        societyContext, String.class);

// setting values
parser.parseExpression("Officers['advisors'][0].PlaceOfBirth.Country").setValue(
        societyContext, "Croatia");
```

### **6.5.3 内联列表**

列表（Lists）可以用过大括号{}直接引用

```
// evaluates to a Java list containing the four numbers
List numbers = (List) parser.parseExpression("{1,2,3,4}").getValue(context);

List listOfLists = (List) parser.parseExpression("{{'a','b'},{'x','y'}}").getValue(context);
```

{}本身代表一个空list.因为性能的关系，如果列表本身完全由固定的常量值组成，这个时候会创建一个常量列表来代替表达式，而不是每次在求值的时候创建一个新的列表。

### **6.5.4 内联Maps**

Maps也可以直接通过{key:value}标记的方式在表达式中使用

```
// evaluates to a Java map containing the two entries
Map inventorInfo = (Map) parser.parseExpression("{name:'Nikola',dob:'10-July-1856'}").getValue(context);

Map mapOfMaps = (Map) parser.parseExpression("{name:{first:'Nikola',last:'Tesla'},dob:{day:10,month:'July',year:1856}}").getValue(context);
```

{:} 本身代表一个空的Map。因为性能的原因：如果Map本身包含固定的常量或者其他级联的常量结构（lists或者maps）则一个常量Map会创建来代表表达式，而不是每次求值的时候都创建一个新的Map.Map的Key并不一定需要引号引用、比如上面的例子就没有引用。

### **6.5.5 创建数组**

数组可以使用类似于Java的语法创建，创建时可以事先指定数组的容量大小、这个是可选的。

```
int[] numbers1 = (int[]) parser.parseExpression("new int[4]").getValue(context);

// Array with initializer
int[] numbers2 = (int[]) parser.parseExpression("new int[]{1,2,3}").getValue(context);

// Multi dimensional array
int[][] numbers3 = (int[][]) parser.parseExpression("new int[4][5]").getValue(context);
```

在创建多维数组时还不支持事先指定初始化的值。

### **6.5.6 方法**

方法可以使用典型的Java编程语法来调用。方法可以直接在字符串常量上调用。可变参数也是支持的。

```
// string literal, evaluates to "bc"
String c = parser.parseExpression("'abc'.substring(2, 3)").getValue(String.class);

// evaluates to true
boolean isMember = parser.parseExpression("isMember('Mihajlo Pupin')").getValue(
        societyContext, Boolean.class);
```

### **6.5.7 运算符**

#### **关系运算符**

关系运算符；包括==,&lt;&gt;,&lt;,&lt;=,&gt;,&gt;=等标准运算符都是可以直接支持的

```
// evaluates to true
boolean trueValue = parser.parseExpression("2 == 2").getValue(Boolean.class);

// evaluates to false
boolean falseValue = parser.parseExpression("2 &amp;lt; -5.0").getValue(Boolean.class);

// evaluates to true
boolean trueValue = parser.parseExpression("'black' &amp;lt; 'block'").getValue(Boolean.class);
```

> Note：&gt;/&lt;运算符和null做比较时遵循一个简单的规则:null代表什么都没有（不代表0）.因此，所有的值总是大于null\(X&gt;null总是true\)  
> 也就是没有一个值会小于什么都没有\(x&lt;null总是返回false\).尽量不要在数值比较中使用null,而是和0做比较（例如X&gt;0或者X&lt;0）.

除了标准的关系运算符，SpEL还支持instanceof关键字和基于matches操作符的正则表达式。

> 备注：需要注意元数据类型会自动装箱成包装类型，因此1 instanceof T\(int\)结果是false，1 instanceof T\(Integer\)的结果是true.

每一个符号操作符也可以通过首字母简写的方式标识。这样可以避免表达式所用的符号在当前表达式所在的文档中存在特殊含义而带来的冲突（比如XML文档的&lt;）.简写的符号有：lt \(&lt;\), gt \(&gt;\), le \(?\), ge \(&gt;=\), eq \(==\), ne \(!=\), div \(/\), mod \(%\), not \(!\). 这里不区分大小写。

#### **逻辑运算符**

支持的逻辑运算符有：and,or,和not.它们的使用方法如下：

```
// -- AND --

// evaluates to false
boolean falseValue = parser.parseExpression("true and false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') and isMember('Mihajlo Pupin')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- OR --

// evaluates to true
boolean trueValue = parser.parseExpression("true or false").getValue(Boolean.class);

// evaluates to true
String expression = "isMember('Nikola Tesla') or isMember('Albert Einstein')";
boolean trueValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);

// -- NOT --

// evaluates to false
boolean falseValue = parser.parseExpression("!true").getValue(Boolean.class);

// -- AND and NOT --
String expression = "isMember('Nikola Tesla') and !isMember('Mihajlo Pupin')";
boolean falseValue = parser.parseExpression(expression).getValue(societyContext, Boolean.class);
```

#### **算术运算符**

加号运算符可以同时用于数字和字符串。减号，乘法和除法只能用于数字。其他支持的算术运算符有取模\(%\)和指数\(^\).遵循标准运算符优先级。下面是一些例子：

```
// Addition
int two = parser.parseExpression("1 + 1").getValue(Integer.class); // 2

String testString = parser.parseExpression(
        "'test' + ' ' + 'string'").getValue(String.class); // 'test string'

// Subtraction
int four = parser.parseExpression("1 - -3").getValue(Integer.class); // 4

double d = parser.parseExpression("1000.00 - 1e4").getValue(Double.class); // -9000

// Multiplication
int six = parser.parseExpression("-2 * -3").getValue(Integer.class); // 6

double twentyFour = parser.parseExpression("2.0 * 3e0 * 4").getValue(Double.class); // 24.0

// Division
int minusTwo = parser.parseExpression("6 / -3").getValue(Integer.class); // -2

double one = parser.parseExpression("8.0 / 4e0 / 2").getValue(Double.class); // 1.0

// Modulus
int three = parser.parseExpression("7 % 4").getValue(Integer.class); // 3

int one = parser.parseExpression("8 / 5 % 2").getValue(Integer.class); // 1

// Operator precedence
int minusTwentyOne = parser.parseExpression("1+2-3*8").getValue(Integer.class); // -21
```

### **6.5.8 赋值**

属性可以使用赋值运算符设置。可以通过调用setValue设置。但是也可以在getValue方法中设置

```
Inventor inventor = new Inventor();
StandardEvaluationContext inventorContext = new StandardEvaluationContext(inventor);

parser.parseExpression("Name").setValue(inventorContext, "Alexander Seovic2");

// alternatively

String aleks = parser.parseExpression(
        "Name = 'Alexandar Seovic'").getValue(inventorContext, String.class);
```

### **6.5.9 类型**

T操作符是一个特殊的操作符、可以同于指定java.lang.Class的实例（类型）。静态方法也可以通过这个操作符调用。

StandardEvaluationContext使用TypeLocator来查找类型，其中StandardTypeLocator（这个可以被替换使用其他类）默认对java.lang包里的类型可见。也就是说 T\(\)引用java.lang包里面的类型不需要限定包全名，但是其他类型的引用必须要。

```
Class dateClass = parser.parseExpression("T(java.util.Date)").getValue(Class.class);

Class stringClass = parser.parseExpression("T(String)").getValue(Class.class);

boolean trueValue = parser.parseExpression(
        "T(java.math.RoundingMode).CEILING &amp;lt; T(java.math.RoundingMode).FLOOR")
        .getValue(Boolean.class);
```

### **6.5.10 构造器**

构造器可以使用new操作符来调用。除了元数据类型和String（比如int,float等可以直接使用）都需要限定类的全名。

```
nventor einstein = p.parseExpression(
		"new org.spring.samples.spel.inventor.Inventor('Albert Einstein', 'German')")
		.getValue(Inventor.class);

//create new inventor instance within add method of List
p.parseExpression(
		"Members.add(new org.spring.samples.spel.inventor.Inventor(
			'Albert Einstein', 'German'))").getValue(societyContext);
```

### **6.5.11 变量**

表达式中的变量可以通过语法\#变量名使用。变量可以在StandardEvaluationContext中通过方法setVariable设置。

```
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
StandardEvaluationContext context = new StandardEvaluationContext(tesla);
context.setVariable("newName", "Mike Tesla");

parser.parseExpression("Name = #newName").getValue(context);

System.out.println(tesla.getName()) // "Mike Tesla"
```

#### **\#this和\#root变量**

\#this变量永远指向当前表达式正在求值的对象（这时不需要限定全名）。变量\#root总是指向根上下文对象。\#this在表达式不同部分解析过程中可能会改变，但是\#root总是指向根

```
// create an array of integers
List<Integer> primes = new ArrayList<Integer>();
primes.addAll(Arrays.asList(2,3,5,7,11,13,17));

// create parser and set variable 'primes' as the array of integers
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setVariable("primes",primes);

// all prime numbers > 10 from the list (using selection ?{...})
// evaluates to [11, 13, 17]
List<Integer> primesGreaterThanTen = (List<Integer>) parser.parseExpression(
		"#primes.?[#this>10]").getValue(context);
```

### **6.5.12 函数**

你可以扩展SpEL，在表达式字符串中使用自定义函数。这些自定义函数是通过StandardEvaluationContext的registerFunction来注册的

```
public void registerFunction(String name, Method m)
```

首先定义一个Java方法作为函数的实现、例如下面是一个将字符串反转的方法。

```
public abstract class StringUtils {

	public static String reverseString(String input) {
		StringBuilder backwards = new StringBuilder();
		for (int i = 0; i &amp;lt; input.length(); i++)
			backwards.append(input.charAt(input.length() - 1 - i));
		}
		return backwards.toString();
	}
}
```

然后将这个方法注册到求值上下文中就可以应用到表达式字符串中。

```
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();

context.registerFunction("reverseString",
	StringUtils.class.getDeclaredMethod("reverseString", new Class[] { String.class }));

String helloWorldReversed = parser.parseExpression(
	"#reverseString('hello')").getValue(context, String.class);
```

### **6.5.13 Bean引用**

如果求值上下文已设置bean解析器，可以在表达式中使用（@）符合来查找Bean

```
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("@foo").getValue(context);
```

如果是访问工厂Bean，bean名字前需要添加前缀\(&\)符号

```
ExpressionParser parser = new SpelExpressionParser();
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new MyBeanResolver());

// This will end up calling resolve(context,"&amp;amp;foo") on MyBeanResolver during evaluation
Object bean = parser.parseExpression("&amp;amp;foo").getValue(context);
```

### **6.5.14 三元操作符 \(If-Then-Else\)**

你可以在表达式中使用三元操作符来实现if-then-else的条件逻辑。下面是一个小例子：

```
String falseString = parser.parseExpression(
		"false ? 'trueExp' : 'falseExp'").getValue(String.class);
```

在这个例子中，因为布尔值false返回的结果一定是’falseExp’。下面是一个更实际的例子。

```
parser.parseExpression("Name").setValue(societyContext, "IEEE");
societyContext.setVariable("queryName", "Nikola Tesla");

expression = "isMember(#queryName)? #queryName + ' is a member of the ' " +
		"+ Name + ' Society' : #queryName + ' is not a member of the ' + Name + ' Society'";
```

### **6.5.15 Elvis运算符**

Elvis运算符可以简化Java的三元操作符，是Groovy中使用的一种操作符。如果使用三元操作符语法你通常需要重复写两次变量名，例如：

```
String name = "Elvis Presley";
String displayName = name != null ? name : "Unknown";
```

使用Elvis运算符可以简化写法，这个符号的名字由来是它很像Elvis的发型（译者注：Elvis=Elvis Presley，猫王，著名摇滚歌手）

```
ExpressionParser parser = new SpelExpressionParser();

String name = parser.parseExpression("name?:'Unknown'").getValue(String.class);

System.out.println(name); // 'Unknown'
```

下面是一个复杂一点的例子:

```
ExpressionParser parser = new SpelExpressionParser();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
StandardEvaluationContext context = new StandardEvaluationContext(tesla);

String name = parser.parseExpression("Name?:'Elvis Presley'").getValue(context, String.class);

System.out.println(name); // Nikola Tesla

tesla.setName(null);

name = parser.parseExpression("Name?:'Elvis Presley'").getValue(context, String.class);

System.out.println(name); // Elvis Presley
```

### **6.5.16 安全引用运算符**

安全引用运算符主要为了避免空指针，源于Groovy语言。很多时候你引用一个对象的方法或者属性时都需要做非空校验。为了避免此类问题、使用安全引用运算符只会返回null而不是抛出一个异常。

```
ExpressionParser parser = new SpelExpressionParser();

Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
tesla.setPlaceOfBirth(new PlaceOfBirth("Smiljan"));

StandardEvaluationContext context = new StandardEvaluationContext(tesla);

String city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, String.class);
System.out.println(city); // Smiljan

tesla.setPlaceOfBirth(null);

city = parser.parseExpression("PlaceOfBirth?.City").getValue(context, String.class);

System.out.println(city); // null - does not throw NullPointerException!!!
```

> 备注：Elvis操作符可以在表达式中赋默认值，例如。在一个@Value表达式中：@Value\(“\#{systemProperties\[‘pop3.port’\] ?: 25}”\)  
> 上面的例子如果系统属性pop3.port已定义会直接注入，如果未定义，则返回默认值25.

### 6.5.17 集合筛选

该功能是SpEL中一项强大的语言特性，允许你将源集合选择其中的某几项生成另外一个集合。选择器使用语法.?\[selectionExpression\].通过该表达式可以过滤集合并返回原集合中的子集。例如，下面的例子我们返回inventors对象中的国籍为塞尔维亚的子集：

```
List<Inventor> list = (List<Inventor>) parser.parseExpression(
		"Members.?[Nationality == 'Serbian']").getValue(societyContext);
```

筛选可以同时在list和maps上面使用。对于list来说是选择的标准是针对单个列表的每一项来比对求值，对于map来说选择的标准是针对Map的每一项（类型为Java的Map.Entry）。Map项的Key和alue都可以作为筛选的比较选项

下面的例子中表达式会返回一个新的map,包含原map中值小于27的所有子项。

```
Map newMap = parser.parseExpression("map.?[value<27]").getValue();
```

除了可以返回所有被选择的元素，也可以只返回第一或者最后一项。返回第一项的选择语法是：

^\[…​\]，返回最后一项的选择语法是 $\[…​\].

### **6.5.18 集合投影**

投影使得一个集合通过子表达式求值，并返回一个新的结果。投影的语法是 !\[projectionExpression\]. 举一个通俗易懂的例子，假设我们有一个inventors 对象列表，但是我们想返回每一个inventor出生的城市列表。我们需要遍历inventor的每一项，通过 ‘placeOfBirth.city’来求值。下面是具体的代码例子：

```
// returns ['Smiljan', 'Idvor' ]
List placesOfBirth = (List)parser.parseExpression("Members.![placeOfBirth.city]");
```

也可以在Map上使用投影、在这种场景下投影表达式会作用于Map的每一项（类型为Java的Map.Entry）。Map项的Key和alue都可以作为选择器的比较选项Map投影的结果是一个list，包含map每一项被投影表达式求值后的结果。

### **6.5.19 表达式模板**

表达式模板运行在一段文本中混合包含一个或多个求值表达式模块。各个求值块都通过可被自定义的前后缀字符分隔，一个通用的选择是使用\#{ }作为分隔符。例如：

```
String randomPhrase = parser.parseExpression(
		"random number is #{T(java.lang.Math).random()}",
		new TemplateParserContext()).getValue(String.class);

// evaluates to "random number is 0.7038186818312008"
```

求值的字符串是通过字符文本’random number is’以及\#{}分隔符中的表达式求值结果拼接起来的，在这个例子中就是调用random\(\)的结果。方法parseExpression\(\)的第二个传入参数类型是ParserContext。ParserContext接口用来确定表达式该如何被解析、从而支持表达式的模板功能。其实现类TemplateParserContext的定义如下：

```
public class TemplateParserContext implements ParserContext {

	public String getExpressionPrefix() {
		return "#{";
	}

	public String getExpressionSuffix() {
		return "}";
	}

	public boolean isTemplate() {
		return true;
	}
}
```

## **6.6 本章节例子中使用的类**

Inventor.java

```
package org.spring.samples.spel.inventor;

import java.util.Date;
import java.util.GregorianCalendar;

public class Inventor {

    private String name;
    private String nationality;
    private String[] inventions;
    private Date birthdate;
    private PlaceOfBirth placeOfBirth;

    public Inventor(String name, String nationality) {
        GregorianCalendar c= new GregorianCalendar();
        this.name = name;
        this.nationality = nationality;
        this.birthdate = c.getTime();
    }

    public Inventor(String name, Date birthdate, String nationality) {
        this.name = name;
        this.nationality = nationality;
        this.birthdate = birthdate;
    }

    public Inventor() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getNationality() {
        return nationality;
    }

    public void setNationality(String nationality) {
        this.nationality = nationality;
    }

    public Date getBirthdate() {
        return birthdate;
    }

    public void setBirthdate(Date birthdate) {
        this.birthdate = birthdate;
    }

    public PlaceOfBirth getPlaceOfBirth() {
        return placeOfBirth;
    }

    public void setPlaceOfBirth(PlaceOfBirth placeOfBirth) {
        this.placeOfBirth = placeOfBirth;
    }

    public void setInventions(String[] inventions) {
        this.inventions = inventions;
    }

    public String[] getInventions() {
        return inventions;
    }
}
```

PlaceOfBirth.java

```
package org.spring.samples.spel.inventor;

public class PlaceOfBirth {

    private String city;
    private String country;

    public PlaceOfBirth(String city) {
        this.city=city;
    }

    public PlaceOfBirth(String city, String country) {
        this(city);
        this.country = country;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String s) {
        this.city = s;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

}
```

Society.java

```
package org.spring.samples.spel.inventor;

import java.util.*;

public class Society {

    private String name;

    public static String Advisors = "advisors";
    public static String President = "president";

    private List&amp;lt;Inventor&amp;gt; members = new ArrayList&amp;lt;Inventor&amp;gt;();
    private Map officers = new HashMap();

    public List getMembers() {
        return members;
    }

    public Map getOfficers() {
        return officers;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public boolean isMember(String name) {
        for (Inventor inventor : members) {
            if (inventor.getName().equals(name)) {
                return true;
            }
        }
        return false;
    }

}
```



