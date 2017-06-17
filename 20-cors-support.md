## 20. CORS 支持

## 20.1 简介

出于安全考虑，浏览器禁止AJAX调用驻留在当前来源之外的资源。 例如，当您在一个标签中检查您的银行帐户时，您可以在另一个标签中打开evil.com网站。 evil.com的脚本不能使用您的凭据向您的银行API发出AJAX请求（例如，从您的帐户中提款）！

[Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)\(CORS\) 是 大多数浏览器实现的[W3C](https://www.w3.org/TR/cors/)规范，允许您以灵活的方式指定什么样的跨域请求被授权，而不是使用一些较不安全和不太强大的黑客工具（如IFRAME或JSONP）。

从Spring Framework 4.2开始，CORS支持开箱即用。. CORS 请求\([including preflight ones with an`OPTIONS`method](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/FrameworkServlet.java#L906)\) 被自动调度到各种已注册的HandlerMappings.  由于CorsProcessor的实现（默认为DefaultCorsProcessor），为了根据您提供的CORS配置添加相关的CORS响应头（如Access-Control-Allow-Origin），它们处理CORS预检请求并拦截CORS简单实际的请求。

## 20.2 Controller 方法CORS 配置

你如果想使用CORS ，可以在`@RequestMapping上增加`[`@CrossOrigin`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html)注解. 默认情况下@CrossOrigin允许@RequestMapping注释中指定的所有起始点和HTTP方法：

```
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin
	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```

对于整个 controller的CORS 的支持:

```
@CrossOrigin(origins = "http://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```

在上面的例子中，`retrieve()` 和 `remove()对实现了对CORS 的支持`,因此你可以使用`@CrossOrigin` 属性对自已的程序进行自定义的CORS 配置。

您甚至可以同时使用控制器级和方法级的CORS配置; 然后，Spring将组合两个注释的属性以创建合并的CORS配置。

```
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin("http://domain2.com")
	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, path = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```

## 20.3 全局CORS 配置

除了细粒度，基于注释的配置，您也可能想要定义一些全局CORS配置。 这与使用过滤器类似，但可以在Spring MVC中声明，并结合细粒度的@CrossOrigin配置。 默认情况下，所有起始点和GET，HEAD和POST方法都是允许的。

### 20.3.1 JavaConfig

配置全局CORS 如下所示:

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/**");
	}
}
```

你也可以轻易的改变任意属性，以及仅将此CORS配置应用于特定路径模式：

```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/api/**")
			.allowedOrigins("http://domain2.com")
			.allowedMethods("PUT", "DELETE")
			.allowedHeaders("header1", "header2", "header3")
			.exposedHeaders("header1", "header2")
			.allowCredentials(false).maxAge(3600);
	}
}
```

### 20.3.2 XML 命名空间

以下最小的XML配置为/ \*\*路径模式启用CORS，具有与上述JavaConfig示例相同的默认属性：

```
<mvc:cors>
	<mvc:mapping path="/**" />
</mvc:cors>
```

也可以使用自定义属性声明几个CORS映射：

```
<mvc:cors>

	<mvc:mapping path="/api/**"
		allowed-origins="http://domain1.com, http://domain2.com"
		allowed-methods="GET, PUT"
		allowed-headers="header1, header2, header3"
		exposed-headers="header1, header2" allow-credentials="false"
		max-age="123" />

	<mvc:mapping path="/resources/**"
		allowed-origins="http://domain1.com" />

</mvc:cors>
```

## 20.4 高级自定义

[CorsConfiguration](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/cors/CorsConfiguration.html) 允许你定义CORS 请求怎样被处理: 允许origins, headers, methods, 如.：他可以使用以下几种方式处理:

* [`AbstractHandlerMapping#setCorsConfiguration()`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/servlet/handler/AbstractHandlerMapping.html#setCorsConfiguration-java.util.Map-)允许指定映射到路径模式（如/ api / \*\*）的几个 [CorsConfiguration](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/cors/CorsConfiguration.html)实例的Map
* 子类通过重载`AbstractHandlerMapping#getCorsConfiguration(Object, HttpServletRequest)`方法提供自已的`CorsConfiguration`.
* Handlers 通过实现 [`CorsConfigurationSource`](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/cors/CorsConfigurationSource.html)接口\(像[`ResourceHttpRequestHandler`](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/resource/ResourceHttpRequestHandler.java)现在做的一样\) 对每个请求提供一个[CorsConfiguration](http://docs.spring.io/spring-framework/docs/5.0.0.M5/javadoc-api/org/springframework/web/cors/CorsConfiguration.html) 实例.

## 20.5 基于 CORS 支持的过滤

为了支持基于过滤器的安全框架（如[Spring Security](http://projects.spring.io/spring-security/)）的CORS，或者与其他不支持本地CORS的库一起支持CORS，Spring Framework还提供了一个 [`CorsFilter`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/filter/CorsFilter.html). 而不是使用@CrossOrigin或WebMvcConfigurer＃addCorsMappings（CorsRegistry），您需要注册一个自定义过滤器，如下所示:

```
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

public class MyCorsFilter extends CorsFilter {

	public MyCorsFilter() {
		super(configurationSource());
	}

	private static UrlBasedCorsConfigurationSource configurationSource() {
		CorsConfiguration config = new CorsConfiguration();
		config.setAllowCredentials(true);
		config.addAllowedOrigin("http://domain1.com");
		config.addAllowedHeader("*");
		config.addAllowedMethod("*");
		UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
		source.registerCorsConfiguration("/**", config);
		return source;
	}
}
```

您需要确保CorsFilter在其他过滤器之前过滤, 请参考[this blog post](https://spring.io/blog/2015/06/08/cors-support-in-spring-framework#filter-based-cors-support) ，了解如何配置Spring Boot。

