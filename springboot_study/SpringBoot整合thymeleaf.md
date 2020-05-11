

# Thymeleaf模板引擎

## 静态资源 

### 1、webjars静态资源映射规则

SpringBoot中，SpringMVC的web配置都在 WebMvcAutoConfiguration 这个配置类里面；

在这个配置类里面有一个静态类WebMvcAutoConfigurationAdapter，它继承了WebMvcConfigurer，该类中有很多配置方法；

有一个方法：addResourceHandlers 添加资源处理

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
  	if (!this.resourceProperties.isAddMappings()) {
      	// 已禁用默认资源处理
        logger.debug("Default resource handling disabled");
        return;
    }
    // 缓存控制
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    // webjars 配置
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                                             .addResourceLocations("classpath:/META-INF/resources/webjars/")
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
    // 静态资源配置
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                                             .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
}
```

所有的 /webjars/** ， 都需要去 classpath:/META-INF/resources/webjars/ 找对应的资源；

> 什么是webjars

Webjars本质就是以jar包的方式引入我们的静态资源 ， 我们以前要导入一个静态资源文件，直接导入即可。

使用SpringBoot需要使用Webjars，我们可以去搜索一下：

网站：https://www.webjars.org 

要使用jQuery，我们只要要引入jQuery对应版本的pom依赖即可！

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

导入完毕，查看webjars目录结构，并访问Jquery.js文件！

 ![image-20200504173201573](picture\springboot整合thymeleaf\image-20200504173201573.png)

访问：只要是静态资源，SpringBoot就会去对应的路径寻找资源，我们这里访问：http://localhost:8080/webjars/jquery/3.4.1/jquery.js

### 2、resources静态资源映射规则

在`addResourceHandlers()`里面定义了staticPathPattern的第二种映射规则 ：/**

```java
//获取静态资源映射路径
String staticPathPattern = this.mvcProperties.getStaticPathPattern();
```

点进`getStaticPathPattern()`方法，发现

```java
// 进入方法
public String getStaticPathPattern() {
    return this.staticPathPattern;
}
// Path pattern used for static resources.
// 用于静态资源
private String staticPathPattern = "/**";
```

在`this.mvcProperties.getStaticPathPattern()`下面还获取了`this.resourceProperties.getStaticLocations()`

```java
if (!registry.hasMappingForPattern(staticPathPattern)) {
 customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)      .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())) .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
```

点进`getStaticLocations()`

```java
// 进入方法
public String[] getStaticLocations() {
    return this.staticLocations;
}
// 找到CLASSPATH_RESOURCE_LOCATIONS对应的值
private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
// 找到路径
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/","classpath:/resources/", "classpath:/static/", "classpath:/public/" };
```

ResourceProperties 可以设置和我们静态资源有关的参数；这里面指向了它会去寻找资源的文件夹，即上面数组的内容。

所以得出结论，以下四个目录存放的静态资源可以被我们识别：

```java
"classpath:/META-INF/resources/"
"classpath:/resources/"
"classpath:/static/"
"classpath:/public/"
```

### 3、自定义静态资源路径

可以自己通过配置文件来指定，哪些文件夹是放静态资源文件的，在application.properties中配置：

```properties
spring.resources.static-locations=classpath:/coding/,classpath:/xuwei/
```

一旦自定义了静态文件夹的路径，原来的自动配置就都会失效。

## 首页处理

在`EnableWebMvcConfiguration`这个静态类中，定义了`welcomePageHandlerMapping()`

```java
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext, FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(this.getInterceptors(mvcConversionService, mvcResourceUrlProvider));
    return welcomePageHandlerMapping;
}
```

这个欢迎页的映射，就是默认的首页，进入`this.getWelcomePage()`

```java
private Optional<Resource> getWelcomePage() {
    String[] locations = WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations());
    // ::是java8 中新引入的运算符
    // Class::function的时候function是属于Class的，应该是静态方法。
    // this::function的funtion是属于这个对象的。
    return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
}
// 欢迎页就是一个location下的的 index.html 而已
private Resource getIndexHtml(String location) {
    return this.resourceLoader.getResource(location + "index.html");
}
```

欢迎页，静态资源文件夹下的所有 index.html 页面；被 /** 映射。

```java
"classpath:/META-INF/resources/"
"classpath:/resources/"
"classpath:/static/"
"classpath:/public/"
```

访问 index的优先级按照从上到下排列。

## 模板引擎

​	前端交给我们的页面，是html页面。如果是以前开发，需要把他们转成jsp页面，jsp好处就是当查出一些数据转发到JSP页面以后，可以用jsp轻松实现数据的显示及交互等。

​	jsp支持非常强大的功能，包括能写Java代码，但是SpringBoot这个项目首先是以jar的方式，不是war，还有，SpringBoot用的是嵌入式的Tomcat，**默认是不支持jsp的**。

不支持jsp，如果直接用纯静态页面的方式，那给开发会带来非常大的麻烦，所以SpringBoot推荐使用模板引擎。

​	模板引擎，其实听到过很多，其中jsp就是一个模板引擎，还有用的比较多的freemarker，包括SpringBoot推荐的Thymeleaf，模板引擎有非常多，但再多的模板引擎，他们的思想都是一样的，如下图：

![image-20200504191041134](picture\springboot整合thymeleaf\image-20200504191041134.png)

​	模板引擎的作用就是写一个页面模板，比如有些值，是动态的，所以要写**表达式**。而这些值，就是我们在后台**封装**的**数据**。然后把**模板**和**数据**交给**模板引擎**，模板引擎按照**数据**把表达式**解析**并**填充**到指定的**位置**，然后把渲染完的页面输出，这就是模板引擎，不管是jsp还是其他模板引擎，都是这个思想。

​	只不过不同模板引擎之间，语法可能会不一样。主要介绍SpringBoot推荐的Thymeleaf模板引擎，这个模板引擎是一个高级语言的模板引擎，他的语法更简单，功能更强大。

### 引入Thymeleaf

~~Thymeleaf（thyme leaf，百里香叶）~~

Thymeleaf 官网：https://www.thymeleaf.org/

Thymeleaf 在Github 的主页：https://github.com/thymeleaf/thymeleaf

Spring官方文档：2.26对应的版本，[链接](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#using-boot-starter)

找到thymeleaf对应的pom依赖：

```xml
<!--thymeleaf-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

Maven会自动下载jar包

 ![image-20200504192904874](picture\springboot整合thymeleaf\image-20200504192904874.png)

### Thymeleaf分析

​	首先按照SpringBoot的自动配置原理看一下Thymeleaf的自动配置规则，再按照那个规则，来使用。

​	去找Thymeleaf的自动配置类：ThymeleafProperties

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;
	public static final String DEFAULT_PREFIX = "classpath:/templates/";
	public static final String DEFAULT_SUFFIX = ".html";
	private boolean checkTemplate = true;
	private boolean checkTemplateLocation = true;
	private String prefix = DEFAULT_PREFIX;
	private String suffix = DEFAULT_SUFFIX;
	private String mode = "HTML";
	private Charset encoding = DEFAULT_ENCODING;
	private boolean cache = true;
	private Integer templateResolverOrder;
	private String[] viewNames;
	private String[] excludedViewNames;
	private boolean enableSpringElCompiler;
	private boolean renderHiddenMarkersBeforeCheckboxes = false;
	private boolean enabled = true;
}
```

可以在其中看到默认的前缀和后缀，以及默认的资源路径，只需要把html页面放在类路径下的templates下，thymeleaf就可以自动渲染。

### 测试页面

1、编写一个TestController

```java
@Controller
public class TestController {
    @RequestMapping("/t1")
    public String test1(){
        //classpath:/templates/test.html
        return "test";
    }  
}
```

2、编写一个测试页面  test.html 放在 templates 目录下

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>测试页面</h1>
</body>
</html>
```

3、启动项目测试，访问http://localhost:8080/t1

 ![image-20200504195220893](picture\springboot整合thymeleaf\image-20200504195220893.png)

### Thymeleaf 语法

Thymeleaf 官网：https://www.thymeleaf.org/ ，去下载Thymeleaf的官方文档。

#### 简单测试 

**需要查出一些数据，在页面中展示**

1、修改测试请求，增加数据传输；

```java
@RequestMapping("/t1")
public String test1(Model model){
    //存入数据
    model.addAttribute("msg","Hello Thymeleaf");
    //classpath:/templates/test.html
    return "test";
}
```

2、使用thymeleaf，需要在html文件中导入命名空间的约束，方便提示。

```properties
xmlns:th="http://www.thymeleaf.org"
```

3、编写前端页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>狂神说</title>
</head>
<body>
<h1>测试页面</h1>
<!--th:text就是将div中的内容设置为它指定的值，和Vue一样-->
<div th:text="${msg}"></div>
</body>
</html>
```

4、启动测试，访问http://localhost:8080/t1

 ![image-20200504195802170](picture\springboot整合thymeleaf\image-20200504195802170.png)

#### Thymeleaf的使用语法

**1、可以使用任意的 th:attr 来替换Html中原生属性的值！**

![image-20200504200250801](picture\springboot整合thymeleaf\image-20200504200250801.png)



**2、表达式**

```yaml
Simple expressions:（表达式语法）
Variable Expressions: ${...}：获取变量值；OGNL；
    1）、获取对象的属性、调用方法
    2）、使用内置的基本对象：#18
         #ctx : the context object.
         #vars: the context variables.
         #locale : the context locale.
         #request : (only in Web Contexts) the HttpServletRequest object.
         #response : (only in Web Contexts) the HttpServletResponse object.
         #session : (only in Web Contexts) the HttpSession object.
         #servletContext : (only in Web Contexts) the ServletContext object.

    3）、内置的一些工具对象：
　　　　　　#execInfo : information about the template being processed.
　　　　　　#uris : methods for escaping parts of URLs/URIs
　　　　　　#conversions : methods for executing the configured conversion service (if any).
　　　　　　#dates : methods for java.util.Date objects: formatting, component extraction, etc.
　　　　　　#calendars : analogous to #dates , but for java.util.Calendar objects.
　　　　　　#numbers : methods for formatting numeric objects.
　　　　　　#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
　　　　　　#objects : methods for objects in general.
　　　　　　#bools : methods for boolean evaluation.
　　　　　　#arrays : methods for arrays.
　　　　　　#lists : methods for lists.
　　　　　　#sets : methods for sets.
　　　　　　#maps : methods for maps.
　　　　　　#aggregates : methods for creating aggregates on arrays or collections.
==================================================================================

  Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
  Message Expressions: #{...}：获取国际化内容
  Link URL Expressions: @{...}：定义URL；
  Fragment Expressions: ~{...}：片段引用表达式

Literals（字面量）
      Text literals: 'one text' , 'Another one!' ,…
      Number literals: 0 , 34 , 3.0 , 12.3 ,…
      Boolean literals: true , false
      Null literal: null
      Literal tokens: one , sometext , main ,…
      
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
    
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
    
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
    
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
    
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
    
Special tokens:
    No-Operation: _
```

**测试：**

1、 编写一个Controller，放一些数据

```java
@RequestMapping("/t2")
    public String test2(Map<String,Object> map){
        //存入数据
        map.put("msg","<h1>Hello</h1>");
        map.put("users", Arrays.asList("xuwei","虚伪"));
        //classpath:/templates/test.html
        return "test";
    }
```

2、测试页面取出数据


```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>测试页面</h1>
<!--th:text就是将div中的内容设置为它指定的值，和Vue一样-->
<div th:text="${msg}"></div>
<div th:utext="${msg}"></div><!--不转义-->
<div>[[${msg}]]</div><!--行内写法-->

<!--遍历数据-->
<!--th:each每次遍历都会生成当前这个标签：官网#9-->
<div th:each="user : ${users}" th:text="${user}"></div>
<!--行内写法：官网#12-->
<div th:each="user : ${users}">[[${user}]]</div>
</body>
</html>
```

3、测试结果

 ![image-20200504201838938](picture\springboot整合thymeleaf\image-20200504201838938.png)

## 页面国际化

有的时候，网站会涉及中英文甚至多语言的切换，这时候就需要页面国际化。

> 准备工作

先在IDEA中统一设置properties的编码。

![image-20200504210146320](picture\springboot整合thymeleaf\image-20200504210146320.png)

> 配置文件编写

1、在resources资源文件下新建一个i18n目录，存放国际化配置文件。

2、建立一个login.properties文件，还有一个login_zh_CN.properties，发现IDEA自动识别了国际化操作，文件夹里的文件合并了。

 ![image-20200504210312247](picture\springboot整合thymeleaf\image-20200504210312247.png)

3、可以在这上面去新建文件。

![image-20200504210633410](picture\springboot整合thymeleaf\image-20200504210633410.png)

4、接下来编写配置，可以看到idea下面有另外一个视图。

 ![image-20200504210827288](picture\springboot整合thymeleaf\image-20200504210827288.png)

这个视图点击 + 号可以直接添加属性；我们新建一个login.tip，可以看到边上有三个文件框可以输入。

 ![image-20200504210905036](picture\springboot整合thymeleaf\image-20200504210905036.png)

![image-20200504211003402](picture\springboot整合thymeleaf\image-20200504211003402.png)

然后依次添加其他页面内容即可。

 ![image-20200504211030458](picture\springboot整合thymeleaf\image-20200504211030458.png)

login.properties ：

```properties
#默认
login.page=登录
login.tip=请登录
login.username=用户名
login.password=密码
login.remember=记住我
login.btn=登录

#英文：
login.page=Sign in
login.tip=Please sign in
login.username=username
login.password=password
login.remember=remember me
login.btn=sign in

#中文
login.page=登录
login.tip=请登录
login.username=用户名
login.password=密码
login.remember=记住我
login.btn=登录
```

> 配置文件生效探究

​	去看SpringBoot对国际化的自动配置！这里又涉及到一个类：MessageSourceAutoConfiguration，里面有一个方法，这里发现SpringBoot已经自动配置好了管理我们国际化资源文件的组件 ResourceBundleMessageSource；

```java
// 获取 properties 传递过来的值进行判断
@Bean
public MessageSource messageSource(MessageSourceProperties properties) {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    if (StringUtils.hasText(properties.getBasename())) {
     //设置国际化文件的基础名(将国际化配置文件中的路径以"，"分割为数组，表明可以配置多个路径)
     messageSource.setBasenames(StringUtils
.commaDelimitedListToStringArray(StringUtils.trimAllWhitespace(properties.getBasename())));
    }
    if (properties.getEncoding() != null) {
        messageSource.setDefaultEncoding(properties.getEncoding().name());
    }
    messageSource.setFallbackToSystemLocale(properties.isFallbackToSystemLocale());
    Duration cacheDuration = properties.getCacheDuration();
    if (cacheDuration != null) {
        messageSource.setCacheMillis(cacheDuration.toMillis());
    }
    messageSource.setAlwaysUseMessageFormat(properties.isAlwaysUseMessageFormat());
    messageSource.setUseCodeAsDefaultMessage(properties.isUseCodeAsDefaultMessage());
    return messageSource;
}
```

将国际化文件放在i18n目录下，所以要去配置messages的路径；

```properties
spring.messages.basename=i18n/login
```

>配置页面国际化值

去页面获取国际化的值，查看Thymeleaf的文档，找到message取值操作为：#{...}。

![image-20200504212608641](picture\springboot整合thymeleaf\image-20200504212608641.png)

启动项目，访问一下，发现已经自动识别为中文。

 ![image-20200504212728000](picture\springboot整合thymeleaf\image-20200504212728000.png)

**让页面可以根据按钮自动切换中文英文！**

>配置国际化解析

​	在Spring中有一个国际化的Locale （区域信息对象），里面有一个叫做LocaleResolver （获取区域信息对象）的解析器。去WebMvcAutoConfiguration，寻找locale，看到SpringBoot默认配置：

 ```java
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(
    prefix = "spring.mvc",
    name = {"locale"}
)
public LocaleResolver localeResolver() {
    // 容器中没有就使用系统的，有的话就用用户配置的
    if (this.mvcProperties.getLocaleResolver() == org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties.LocaleResolver.FIXED) {
        return new FixedLocaleResolver(this.mvcProperties.getLocale());
    } else {
        // 接收头国际化分解
        AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
        localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
        return localeResolver;
    }
}
 ```

AcceptHeaderLocaleResolver 这个类中有一个方法

```java
@Override
public Locale resolveLocale(HttpServletRequest request) {
    Locale defaultLocale = getDefaultLocale();
     // 默认的就是根据请求头带来的区域信息获取Locale进行国际化
    if (defaultLocale != null && request.getHeader("Accept-Language") == null) {
        return defaultLocale;
    }
    Locale requestLocale = request.getLocale();
    List<Locale> supportedLocales = getSupportedLocales();
    if (supportedLocales.isEmpty() || supportedLocales.contains(requestLocale)) {
        return requestLocale;
    }
    Locale supportedLocale = findSupportedLocale(request, supportedLocales);
    if (supportedLocale != null) {
        return supportedLocale;
    }
    return (defaultLocale != null ? defaultLocale : requestLocale);
}
```

​	假如想点击链接让自定义的国际化资源生效，就需要让自定义的Locale生效，写一个自定义的LocaleResolver，在链接上携带区域信息。

修改一下前端页面的跳转连接：

```html
<!-- 这里传入参数不需要使用"?"使用(key=value)-->
<a class="btn btn-sm" th:href="@{/index.html(lang='zh_CN')}">中文</a>
<a class="btn btn-sm" th:href="@{/index.html(lang='en_US')}">English</a>
```

去写一个处理的组件类

```java
package com.xuwei.config;

import org.springframework.util.StringUtils;
import org.springframework.web.servlet.LocaleResolver;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;
//继承了LocaleResolver(语言解析器)接口
public class MyLocaleResolver implements LocaleResolver {
    //解析请求
    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        //获取请求中的语言参数
        String language = httpServletRequest.getParameter("lang");
        Locale locale = Locale.getDefault();//如果没有就使用默认值
        //如果请求的链接携带了国际化的参数
        if (!StringUtils.isEmpty(language)){
            //zh_CN
            String[] s = language.split("_");
            //国家，地区
            locale = new Locale(s[0],s[1]);
        }
        return locale;
    }
    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {
    }
}
```

为了让自定义的区域化信息能够生效，需要在配置类中配置这个组件！在自定义的mvcConfig下添加bean：

```java
@Bean
public LocaleResolver localeResolver() {
    return new MyLocaleResolver();
}
```

**重启项目，访问测试，发现点击按钮可以实现成功切换**

![image-20200504214956866](picture\springboot整合thymeleaf\image-20200504214956866.png)

## 错误页面设置

### 默认的错误页面

在template文件夹下，建一个error文件夹，将404.html放入

 ![image-20200504215427386](picture\springboot整合thymeleaf\image-20200504215427386.png)

重启项目，输入不存在的网址，就会发现跳转到自定义的404页面

### 配置文件配置error文件夹

有一个ErrorMvcAutoConfiguration配置类

```java
public class ErrorMvcAutoConfiguration {
    private final ServerProperties serverProperties;

    public ErrorMvcAutoConfiguration(ServerProperties serverProperties) {
        this.serverProperties = serverProperties;
	}
}
```

进入ServerProperties，找到ErrorProperties

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {
    @NestedConfigurationProperty
    private final ErrorProperties error = new ErrorProperties();
}
```

再进入ErrorProperties，找到错误页面的配置路径，可以看到@Value，说明配置属性是error.path，但是是通过ServerProperties去加载，所以要加上server前缀。

```java
@Value("${error.path:/error}")
private String path = "/error";
```

完整属性配置为：

```properties
server.error.path=/error
```

### jar包中自定义错误页面

创建Spring Boot项目，默认打包方式是jar，内部使用内嵌tomcat等servlet容器。

实现ErrorPageRegistrar接口，定义具体异常的URL路径：

```java
import org.springframework.boot.web.servlet.ErrorPage;
import org.springframework.boot.web.servlet.ErrorPageRegistrar;
import org.springframework.boot.web.servlet.ErrorPageRegistry;
import org.springframework.http.HttpStatus;

public class MyErrorPageRegistrar implements ErrorPageRegistrar {
    @Override
    public void registerErrorPages(ErrorPageRegistry errorPageRegistry) {
        ErrorPage page404 = new ErrorPage(HttpStatus.NOT_FOUND, "/404");
        ErrorPage page500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500");

        errorPageRegistry.addErrorPages(page404, page500);
    }
}
```

然后在一个配置类中声明该Bean:

```java
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
    return new MyErrorPageRegistrar();
}
```

在Controller中添加404和500：

```java
/**
  * 404 error
  * @return
  */
@RequestMapping("/404")
public String error404() {
    return "commons/404";
}

/**
  * 500 error
  * @return
  */
@RequestMapping("/500")
public String error500() {
    return "commons/500";
}
```

在resources/templaes/commons目录下创建404.html和500.html下即可。

### war包中自定义错误页面

Spring Boot打成war包发布到tomcat中，首先需要配置如下：

1. 将Spring Boot入口类继承 org.springframework.boot.web.support.SpringBootServletInitializer 类

   ```java
   @SpringBootApplication
   @EnableTransactionManagement
   public class HelloApplication extends SpringBootServletInitializer {
   
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(HelloApplication.class);
    }
   
    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
    }
   }
   ```

2. pom.xml中的packaging设置成war

   ```xml
   <packaging>war</packaging>
   ```

3. pom.xml中添加属性start-class，设置为Spring Boot入口类

   ```xml
   <properties>
       <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
       <java.version>1.8</java.version>
       <thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
       <thymeleaf-layout-dialect.version>2.1.1</thymeleaf-layout-dialect.version>
       <start-class>com.feng.hello.HelloApplication</start-class>
   </properties>
   ```

4. pom.xml中将tomcat的依赖的scope设置成provided

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-tomcat</artifactId>
       <scope>provided</scope>
   </dependency>
   ```

此时打包后生成的war包可以发布到tomcat中了，但是404等错误画面默认是 *Whitelable Error page* ，这种方式明显不够友好，通过如下步骤自定义错误画面：

1. 首先禁用Whitelabel error page, 在application.properties中添加

   ```properties
   server.error.whitelabel.enabled=false
   ```

2. 定义error.html, 404.html, 500.html等错误画面

   ```html
   <!DOCTYPE HTML>
   <html xmlns:th="http://www.thymeleaf.org">
   <head>
       <head th:include="fragments/header :: header"/>
       <script type="text/javascript">
           function toIndex(){
               window.location.href = ctx + "/index";
           }
       </script>
   </head>
   <body class="body-bg w100">
   <div class="Center" style="height: 100%;">
       <div class="Container">
           <div class="Container-1" style="height: 100%;">
               <div class="Content bg" style="text-align: center;">
                   <h1>Sorry, page not found</h1>
                   <a onclick="toIndex()">Go Home</a>
               </div>
           </div>
       </div>
   </div>
   </body>
   </html>
   ```

3. 创建Controller实现 org.springframework.boot.autoconfigure.web.ErrorController 接口

   ```java
   @Controller
   public class AppErrorController implements ErrorController {
   
       @Override
       public String getErrorPath() {
           return "/error";
       }
   
       @RequestMapping("/error")
       public String handleError(HttpServletRequest request) {
           Object status = request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
   
           if (status != null) {
               Integer statusCode = Integer.valueOf(status.toString());
   
               if (statusCode == HttpStatus.NOT_FOUND.value()) {
                   return "commons/404";
               } else if (statusCode == HttpStatus.INTERNAL_SERVER_ERROR.value()) {
                   return "commons/500";
               }
           }
           return "commons/error";
       }
   }
   ```

此时如果访问不存在的画面就会显示自定义的画面。