# 4. 资源

## 4.1 介绍

仅仅使用 JAVA 的 java.net.URL 和针对不同 URL 前缀的标准处理器，并不能满足我们对各种底层资源的访问，比如：我们就不能通过 URL 的标准实现来访问相对类路径或者相对 ServletContext 的各种资源。虽然我们可以针对特定的 url 前缀来注册一个新的 URLStreamHandler（和现有的针对各种特定前缀的处理器类似，比如 http：），然而这往往会是一件比较麻烦的事情\(要求了解 url 的实现机制等），而且 url 接口也缺少了部分基本的方法，如检查当前资源是否存在的方法。



## 4.2 Resource 接口

相对标准 url 访问机制，spring 的 Resource 接口对抽象底层资源的访问提供了一套更好的机制。

```
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isOpen();

    URL getURL() throws IOException;

    File getFile() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();

}

```

```
public interface InputStreamSource {

    InputStream getInputStream() throws IOException;

}

```

Resource 接口里的最重要的几个方法：

* getInputStream\(\): 定位并且打开当前资源，返回当前资源的 InputStream。预计每一次调用都会返回一个新的 – InputStream，因此关闭当前输出流就成为了调用者的责任。
* exists\(\): 返回一个 boolean，表示当前资源是否真的存在。
* isOpen\(\): 返回一个 boolean，表示当前资源是否一个已打开的输入流。如果结果为 true，返回的 InputStream 不能多次读取，只能是一次性读取之后，就关闭 InputStream，以防止内存泄漏。除了 InputStreamResource，其他常用 Resource 实现都会返回 false。
* getDescription\(\): 返回当前资源的描述，当处理资源出错时，资源的描述会用于错误信息的输出。一般来说，资源的描述是一个完全限定的文件名称，或者是当前资源的真实 url。

Resource 接口里的其他方法可以让你获得代表当前资源的 URL 或 File 对象（前提是底层实现可兼容的，也支持该功能）。

Resource抽象在Spring本身被广泛使用，作为需要资源的许多方法签名中的参数类型。 某些Spring API中的其他方法（例如各种ApplicationContext实现的构造函数）采用一个String，它以未安装或简单的形式用于创建适用于该上下文实现的资源，或者通过String路径上的特殊前缀，允许调用者 以指定必须创建和使用特定的资源实现。

Resource 接口（实现）不仅可以被 spring 大量的应用，其也非常适合作为你编程中访问资源的辅助工具类。当你仅需要使用到 Resource 接口实现时，可以直接忽略 spring 的其余部分。单独使用 Rsourece 实现，会造成代码与 spring 的部分耦合，可也仅耦合了其中一小部分辅助类，而且你可以将 Reource 实现作为 URL 的一种访问底层更为有效的替代，与你引入其他库来达到这种目的是一样的。

需要注意的是 Resource 实现并没有去重新发明轮子，而是尽可能地采用封装。举个例子，UrlResource 里就封装了一个 URL 对象，在其内的逻辑就是通过封装的 URL 对象来完成的。

## 4.3 内置的 Resource 实现

spring 直接提供了多种开箱即用的 Resource 实现。

### 4.3.1 UrlResource

UrlResource 封装了一个 java.net.URL 对象，用来访问 URL 可以正常访问的任意对象，比如文件、an HTTP target, an FTP target, 等等。所有的 URL 都可以用一个标准化的字符串来表示。如通过正确的标准化前缀，可以用来表示当前 URL 的类型，当中就包括用于访问文件系统路径的 file:,通过 http 协议访问资源的 http:,通过 ftp 协议访问资源的 ftp:，还有很多……

可以显式化地使用 UrlResource 构造函数来创建一个 UrlResource，不过通常我们可以在调用一个 api 方法是，使用一个代表路径的 String 参数来隐式创建一个 UrlResource。对于后一种情况，会由一个 javabean PropertyEditor 来决定创建哪一种 Resource。如果路径里包含某一个通用的前缀（如 classpath:\),PropertyEditor 会根据这个通用的前缀来创建恰当的 Resource；反之，如果 PropertyEditor 无法识别这个前缀，会把这个路径作为一个标准的 URL 来创建一个 UrlResource。

### 4.3.2 ClassPathResource

ClassPathResource 可以从类路径上加载资源，其可以使用线程上下文加载器、指定加载器或指定的 class 类型中的任意一个来加载资源。

当类路径上资源存于文件系统中，ClassPathResource 支持以 java.io.File 的形式访问，可当类路径上的资源存于尚未解压\(没有 被Servlet 引擎或其他可解压的环境解压）的 jar 包中，ClassPathResource 就不再支持以 java.io.File 的形式访问。鉴于上面所说这个问题，spring 中各式 Resource 实现都支持以 jave.net.URL 的形式访问。

可以显式使用 ClassPathResource 构造函数来创建一个 ClassPathResource ，不过通常我们可以在调用一个 api 方法时，使用一个代表路径的 String 参数来隐式创建一个 ClassPathResource。对于后一种情况，会由一个 javabean PropertyEditor 来识别路径中 classpath: 前缀，从而创建一个 ClassPathResource。

### 4.3.3 FileSystemResource

这是针对 java.io.File 提供的 Resource 实现。显然，我们可以使用 FileSystemResource 的 getFile\(\) 函数获取 File 对象，使用 getURL\(\) 获取 URL 对象。

### 4.3.4 ServletContextResource

这是为了获取 web 根路径的 ServletContext 资源而提供的 Resource 实现。

ServletContextResource 完全支持以流和 URL 的方式访问，可只有当 web 项目是已解压的\(不是以 war 等压缩包形式存在\)且该 ServletContext 资源存于文件系统里，ServletContextResource 才支持以 java.io.File 的方式访问。至于说到，我们的 web 项目是否已解压和相关的 ServletContext 资源是否会存于文件系统里，这个取决于我们所使用的 Servlet 容器。若 Servlet 容器没有解压 web 项目，我们可以直接以 JAR 的形式的访问，或者其他可以想到的方式（如访问数据库）等。

### 4.3.5 InputStreamResource

这是针对 InputStream 提供的 Resource 实现。建议，在确实没有找到其他合适的 Resource 实现时，才使用 InputSteamResource。如果可以，尽量选择 ByteArrayResource 或其他基于文件的 Resource 实现来代替。

与其他 Resource 实现已比较，InputStreamRsource 倒像一个已打开资源的描述符,因此，调用 isOpen\(\) 方法会返回 true。除了在需要获取资源的描述符或需要从输入流多次读取时，都不要使用 InputStreamResource 来读取资源。

### 4.3.6 ByteArrayResource

这是针对字节数组提供的 Resource 实现。可以通过一个字节数组来创建 ByteArrayResource。

当需要从字节数组加载内容时，ByteArrayResource 是一个不错的选择，使用 ByteArrayResource 可以不用求助于 InputStreamResource。

## 4.4 ResourceLoader 接口

ResourceLoader 接口是用来加载 Resource 对象的，换句话说，就是当一个对象需要获取 Resource 实例时，可以选择实现 ResourceLoader 接口。

```
public interface ResourceLoader {

    Resource getResource(String location);

}
```

spring 里所有的应用上下文都是实现了 ResourceLoader 接口，因此，所有应用上下文都可以通过 getResource\(\) 方法获取 Resource 实例。

当你在指定应用上下文调用 getResource\(\) 方法时，而指定的位置路径又没有包含特定的前缀，spring 会根据当前应用上下文来决定返回哪一种类型 Resource。举个例子，假设下面的代码片段是通过 ClassPathXmlApplicationContext 实例来调用的，

```
Resource template = ctx.getResource("some/resource/path/myTemplate.txt");
```

那 spring 会返回一个 ClassPathResource 对象；类似的，如果是通过实例 FileSystemXmlApplicationContext 实例调用的，返回的是一个 FileSystemResource 对象；如果是通过 WebApplicationContext 实例的，返回的是一个 ServletContextResource 对象…… 如上所说，你就可以在指定的应用上下中使用 Resource 实例来加载当前应用上下文的资源。

还有另外一种场景里，如在其他应用上下文里，你可能会强制需要获取一个 ClassPathResource 对象，这个时候，你可以通过加上指定的前缀来实现这一需求，如：

```
Resource template = ctx.getResource("classpath:some/resource/path/myTemplate.txt");
```

类似的，你可以通过其他任意的 url 前缀来强制获取 UrlResource 对象：

```
Resource template = ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

```
Resource template = ctx.getResource("http://myhost.com/resource/path/myTemplate.txt");
```

下面，给出一个表格来总结一下 spring 根据各种位置路径加载资源的策略：

Table 4.1. Resource strings

| 前缀 | 例子 | 解释 |
| :--- | :--- | :--- |
| classpath: | classpath:com/myapp/config.xml | 从类路径加载 |
| file: | [file:///data/config.xml](http://data/config.xml) | 以URL形式从文件系统加载 |
| http: | [http://myserver/logo.png](http://myserver/logo.png) | 以URL形式加载 |
| \(none\) | /data/config.xml | 由底层的ApplicationContext实现决定 |

## 4.5 ResourceLoaderAware 接口

ResourceLoaderAware 是一个特殊的标记接口，用来标记提供 ResourceLoader 引用的对象。

```
public interface ResourceLoaderAware {

    void setResourceLoader(ResourceLoader resourceLoader);
}
```

当将一个 ResourceLoaderAware 接口的实现类部署到应用上下文时\(此类会作为一个 spring 管理的 bean）, 应用上下文会识别出此为一个 ResourceLoaderAware 对象，并将自身作为一个参数来调用 setResourceLoader\(\) 函数，如此，该实现类便可使用 ResourceLoader 获取 Resource 实例来加载你所需要的资源。（附：为什么能将应用上下文作为一个参数来调用 setResourceLoader\(\) 函数呢？不要忘了，在前文有谈过，spring 的所有上下文都实现了 ResourceLoader 接口）。

当然了，一个 bean 若想加载指定路径下的资源，除了刚才提到的实现 ResourcesLoaderAware 接口之外（将 ApplicationContext 作为一个 ResourceLoader 对象注入），bean 也可以实现 ApplicationContextAware 接口，这样可以直接使用应用上下文来加载资源。但总的来说，在需求满足都满足的情况下，最好是使用的专用 ResourceLoader 接口，因为这样代码只会与接口耦合，而不会与整个 spring ApplicationContext 耦合。与 ResourceLoader 接口耦合，抛开 spring 来看，就是提供了一个加载资源的工具类接口。

从 spring 2.5 开始，除了实现 ResourceLoaderAware 接口，也可采取另外一种替代方案——依赖于 ResourceLoader 的自动装配。”传统”的 constructor 和 bytype 自动装配模式都支持 ResourceLoader 的装配（可参阅 Section 5.4.5, “自动装配协作者” ）——前者以构造参数的形式装配，后者以 setter 方法中参数装配。若为了获得更大的灵活性\(包括属性注入的能力和多参方法\)，可以考虑使用基于注解的新注入方式。使用注解 @Autowiring 标记 ResourceLoader 变量，便可将其注入到成员属性、构造参数或方法参数中\( @autowiring 详细的使用方法可参考Section 3.9.2, “@Autowired”.\)。

## 4.6 资源依赖

如果bean本身将通过某种动态过程来确定和提供资源路径，那么bean可以使用ResourceLoader接口来加载资源。 j假设以某种方式加载一个模板，其中需要的特定资源取决于用户的角色。 如果资源是静态的，那么完全消除ResourceLoader接口的使用是有意义的，只需让bean公开它需要的Resource属性，那么它们就会以你所期望的方式被注入。

什么使得它们轻松注入这些属性，是所有应用程序上下文注册和使用一个特殊的JavaBeans PropertyEditor，它可以将String路径转换为Resource对象。 因此，如果myBean具有“资源”类型的模板属性，则可以使用该资源的简单字符串进行配置，如下所示：

```
<bean id="myBean" class="...">
<property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```

请注意，资源路径没有前缀，因为应用程序上下文本身将用作ResourceLoader，资源本身将通过ClassPathResource，FileSystemResource或ServletContextResource（根据需要）加载，具体取决于上下文的确切类型。

如果需要强制使用特定的资源类型，则可以使用前缀。 以下两个示例显示如何强制使用ClassPathResource和UrlResource（后者用于访问文件系统文件）。

```
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```

## 4.7 应用上下文和资源路径

### 4.7.1 构造应用上下文

（某一特定）应用上下文的构造器通常可以使用字符串或字符串数组所指代的\(多个\)资源\(如 xml 文件\)来构造当前上下文。

当指定的位置路径没有带前缀时，那从指定位置路径创建的 Resource 类型\(用于后续加载 bean 定义\),取决于所使用应用上下文。举个列子，如下所创建的 ClassPathXmlApplicationContext ：

```
ApplicationContext ctx = new ClassPathXmlApplicationContext("conf/appContext.xml");
```

会从类路径加载 bean 的定义，因为所创建的 Resource 实例是 ClassPathResource.但所创建的是 FileSystemXmlApplicationContext 时，

```
ApplicationContext ctx = new FileSystemXmlApplicationContext("conf/appContext.xml");
```

则会从文件系统加载 bean 的定义，这种情况下，资源路径是相对工作目录而言的。

注意：若位置路径带有 classpath 前缀或 URL 前缀，会覆盖默认创建的用于加载 bean 定义的 Resource 类型，比如这种情况下的 FileSystemXmlApplicationContext

```
ApplicationContext ctx = new FileSystemXmlApplicationContext("classpath:conf/appContext.xml");
```

，实际是从类路径下加载了 bean 的定义。可是，这个上下文仍然是 FileSystemXmlApplicationContext，而不是 ClassPathXmlApplicationContext，在后续作为 ResourceLoader 来使用时，不带前缀的路径仍然会从文件系统中加载。

**构造 ClassPathXmlApplicationContext 实例 – 快捷方式**

ClassPathXmlApplicationContext 提供了多个构造函数，以利于快捷创建 ClassPathXmlApplicationContext 的实例。最好莫不过使用只包含多个 xml 文件名（不带路径信息）的字符串数组和一个 Class 参数的构造器，所省略路径信息 ClassPathXmlApplicationContext 会从 Class 参数 获取：

下面的这个例子，可以让你对个构造器有比较清晰的认识。试想一个如下类似的目录结构：

```
com/
  foo/
	services.xml
	daos.xml
    MessengerService.class
```

由 services.xml 和 daos.xml 中 bean 所组成的 ClassPathXmlApplicationContext，可以这样来初始化：

```
ApplicationContext ctx = new ClassPathXmlApplicationContext(
                new String[] {"services.xml", "daos.xml"}, MessengerService.class);
```

欲要知道 ClassPathXmlApplicationContext 更多不同类型的构造器，请查阅 Javadocs 文档。

### 4.7.2 使用通配符构造应用上下文

从前文可知，应用上下文构造器的中的资源路径可以是单一的路径（即一对一地映射到目标资源）；另外资源路径也可以使用高效的通配符——可包含 classpath\*：前缀 或 ant 风格的正则表达式（使用 spring 的 PathMatcher 来匹配）。

通配符机制的其中一种应用可以用来组装组件式的应用程序。应用程序里所有组件都可以在一个共知的位置路径发布自定义的上下文片段，则最终应用上下文可使用 classpath\*: 在同一路径前缀\(前面的共知路径）下创建，这时所有组件上下文的片段都会被自动组装。

谨记，路径中的通配符特定用于应用上下文的构造器，只会在应用构造时有效，与其 Resource 自身类型没有任何关系。不可以使用 classpth\*：来构造任一真实的 Resource，因为一个资源点一次只可以指向一个资源。（如果直接使用 PathMatcher 的工具类，也可以在路径中使用通配符）

#### Ant 风格模式

以下是一些使用了 Ant 风格的位置路径：

```
/WEB-INF/*-context.xml
  com/mycompany/**/applicationContext.xml
  file:C:/some/path/*-context.xml
  classpath:com/mycompany/**/applicationContext.xml
```

当位置路径使用了 ant 风格，解释器会遵循一套复杂且预定义的逻辑来解释这些位置路径。解释器会先从位置路径里获取最靠前的不带通配符的路径片段，使用这个路径片段来创建一个 Resource ，并从 Resource 里获取其 URL，若所获取到 URL 前缀并不是 “jar:”,或其他特殊容器产生的特殊前缀（如 WebLogic 的 zip:,WebSphere wsjar\),则从 Resource 里获取 java.io.File 对象，并通过其遍历文件系统。进而解决位置路径里通配符;若获取的是 “jar:”的 URL ，解析器会从其获取一个 java.net.JarURLConnection 或手动解析此 URL，并遍历 jar 文件的内容进而解决位置路径的通配符。

#### 对可移植性的影响

如果指定的路径已经是文件URL（显式地或隐含地，因为基本的ResourceLoader是一个文件系统的，那么通配符将保证以完全可移植的方式工作。

如果指定的路径是类路径位置，则解析器必须通过Classloader.getResource（）调用获取最后一个非通配符路径段URL。 由于这只是路径的一个节点（而不是最后的文件），在这种情况下，它实际上是未定义的（在ClassLoader javadocs中）返回的是什么样的URL。 实际上，它始终是一个java.io.File，它表示类路径资源解析为文件系统位置的目录或某种类型的jar URL，其中类路径资源解析为一个jar位置。 尽管如此，这种操作仍然存在可移植性问题。

如果为最后一个非通配符段获取了一个jar URL，解析器必须能够从中获取java.net.JarURLConnection，或者手动解析jar URL，以便能够遍历该jar的内容，然后解析 通配符。 这将在大多数环境中正常工作，但在其他环境中将会失败，并且强烈建议您在依赖它之前，彻底地在您的特定环境中彻底测试来自jar的资源的通配符解析。

#### classpath\*: 的可移植性

当构造基于 xml 文件的应用上下文时，位置路径可以使用 classpath\*：前缀：

```
ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath*:conf/appContext.xml");
```

classpath\*：的使用表示类路径下所有匹配文件名称的资源都会被获取\(本质上就是调用了 ClassLoader.getResources\(…\) 方法），接着将获取到的资源组装成最终的应用上下文。

> 通配符路径依赖了底层 classloader 的 getResource 方法。可是现在大多数应用服务器提供了自身的 classloader 实现，其处理 jar 文件的形式可能各有不同。要在指定服务器测试 classpath\*: 是否有效，简单点可以使用 getClass\(\).getClassLoader\(\).getResources\(“”\) 去加载类路径 jar包里的一个文件。尝试在两个不同的路径加载名称相同的文件，如果返回的结果不一致，就需要查看一下此服务器中与 classloader 行为设置相关的文档。

在位置路径的其余部分，classpath\*: 前缀可以与 PathMatcher 结合使用，如：” classpath\*:META-INF/\*-beans.xml”。这种情况的解析策略非常简单：取位置路径最靠前的无通配符片段，调用 ClassLoader.getResources\(\) 获取所有匹配的类层次加载器可加载的的资源，随后将 PathMacher 的策略应用于每一个获得的资源（起过滤作用）。

#### 通配符的补充说明

除非所有目标资源都存于文件系统，否则classpath\*：和 ant 风格模式的结合使用，都只能在至少有一个确定根包路径的情况下，才能达到预期的效果。换句话说，就是像 classpath\*:\*.xml 这样的 pattern 不能从根目录的 jar 文件中获取资源，只能从根目录的扩展目录获取资源。此问题的造成源于 jdk ClassLoader.getResources\(\) 方法的局限性——当向 ClassLoader.getResources\(\) 传入空串时\(表示搜索潜在的根目录\)，只能获取的文件系统的文件位置路径，即获取不了 jar 中文件的位置路径。

如果在多个类路径上存在所搜索的根包，那使用 classpath: 和 ant 风格模式一起指定的资源不保证找到匹配的资源。因为使用如下的 pattern classpath:com/mycompany/\*\*/service-context.xml  
去搜索只在某一个路径存在的指定资源com/mycompany/package1/service-context.xml  
时,解析器只会对 getResource\(“com/mycompany”\) 返回的\(第一个\) URL 进行遍历和解释，则当在多个类路径存在基础包节点 “com/mycompany” 时\(如在多个 jar 存在这个基础节点\),解析器就不一定会找到指定资源。因此，这种情况下建议结合使用 classpath\*: 和 ant 风格模式，classpath\*：会让解析器去搜索所有包含基础包节点的类路径。

### 4.7.3 FileSystemResource 注意事项

FileSystemResource 没有依附 FileSystemApplicationContext，因为 FileSystemApplicationContext 并不是一个真正的 \`ResourceLoader。FileSystemResource 并没有按约定规则来处理绝对和相对路径。相对路径是相对与当前工作而言，而绝对路径则是相对文件系统的根目录而言。

然而为了向后兼容，当 FileSystemApplicationContext 是一个 ResourceLoader 实例时，我们做了一些改变 —— 不管 FileSystemResource\` 实例的位置路径是否以 / 开头， FileSystemApplicationContext 都强制将其作为相对路径来处理。事实上，这意味着以下例子等效：

```
ApplicationContext ctx = new FileSystemXmlApplicationContext("conf/context.xml");
```

```
ApplicationContext ctx = new FileSystemXmlApplicationContext("/conf/context.xml");
```

还有：（即使它们的意义不一样 —— 一个是相对路径，另一个是绝对路径。）

```
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("some/resource/path/myTemplate.txt");
```

```
FileSystemXmlApplicationContext ctx = ...;
ctx.getResource("/some/resource/path/myTemplate.txt");
```

实践中，如果确实需要使用绝对路径，建议放弃 FileSystemResource / FileSystemXmlApplicationContext 在绝对路劲的使用，而强制使用 file: 的 UrlResource。

```
// Resource 只会是 UrlResource，与上下文的真实类型无关
ctx.getResource("file:///some/resource/path/myTemplate.txt");
```

```
// 强制 FileSystemXmlApplicationContext 通过 UrlResource 加载资源
ApplicationContext ctx = new FileSystemXmlApplicationContext("file:///conf/context.xml");
```



