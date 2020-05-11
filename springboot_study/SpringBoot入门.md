# Springboot入门

​	springboot是简化spring应用开发，约定大于配置，去繁从简，just run就能创建一个独立的，产品级别的应用。 

## 一、简介

​	简化spring应用开发的一个框架，整个spring技术栈的一个大整合，J2EE开发的一站式解决方案。

优点：

* 快速创建独立运行的spring项目以及主流框架集成
* 使用嵌入式的servlet容器，应用无需打成war包
* starters自动依赖与版本控制
* 大量的自动配置，简化开发，也可以修改默认值
* 无需配置xml，无代码生成，开箱即用
* 准生产环境的运行时应用监控
* 与云计算的天然集成

## 二、微服务

在2014年，由martin fowler提出.

**微服务：架构风格**

一个应用应该是一组小型服务；可以通过http的方式进行互通；

微服务架构把每个功能元素放进一个独立的服务中，并且通过跨服务器分发这些服务进行扩展，只在需要的时候才复制。

每一个功能元素最终都是一个可独立替换和独立升级的软件单元。

**单体应用：ALL IN ONE**

单体应用程序把它所有的功能放进一个单一进程中，并且通过在多个服务器上复制这个单体进行扩展。

## 三、环境准备

> 环境约束

- jdk1.8；
- Spring Boot 2.2.5 及以上；
- maven3.6以上版本；
- 开发工具：IntellijIDEA 2019.3.3以上；

## 四、Spring Boot HelloWorld

功能要求：浏览器发送hello请求，服务器接收请求并处理，响应Hello World 字符串。

### 一、maven创建

#### 1. 创建一个maven工程

![image-20200418161050419](picture\Springboot入门\image-20200418161050419.png)

**选择maven选项，并选择jdk版本为1.8.**

![image-20200418162148515](picture\Springboot入门\image-20200418162148515.png)

输入项目GroupId为com.xuwei，ArtifactID为spring-boot-HelloWorld，项目名会自动生成。

 ![image-20200418162525903](picture\Springboot入门\image-20200418162525903.png)

选择自动导入。

#### 2. 导入相关依赖

```xml
<!-- 父依赖 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
</parent>
<dependencies>
    <!-- web场景启动器 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
<build>
    <plugins>
        <!-- 打包插件 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

#### 3. 编写一个主程序

**编写HelloWorldApplication.java来启动spring boot应用**

```java
/**
 * @SpringBootApplication 来标注一个程序类说明这是一个spring boot应用
 */
@SpringBootApplication
public class HelloWorldApplication {
    public static void main(String[] args) {
        //让springboot应用启动
        SpringApplication.run(HelloWorldApplication.class, args);
    }
}
```

#### 4. 编写相关的controller

```java
//注意：如果是@Controller注解，请求的方法上必须加上@RequestBody，否则访问的时候会找不到页面（404）
@RestController
public class HelloController {
    //@RequestBody
    @RequestMapping("/hello")
    public String Hello() {
        return "Hello World";
    }
}
```

#### 5. 启动Spring Boot

**启动只需要执行`HelloWorldApplication`**

![image-20200418164211697](picture\Springboot入门\image-20200418164211697.png)

可以看到控制台显示了Tomcat在8080端口启动成功。

接下来访问 http://localhost:8080/hello 。

 ![image-20200418164300177](picture\Springboot入门\image-20200418164300177.png)

页面返回了 Hello World 。

### 二、IDEA快速生成

#### 创建springboot项目

**选择spring initializr**

  ![image-20200419161453122](picture\Springboot入门\image-20200419161453122.png) 

**输入group(分组)、artifact（项目名）、选项类型为maven项目、选择java版本为jdk1.8，其余的idea会自动生成。**

![image-20200419162341605](picture\Springboot入门\image-20200419162341605.png)

 **选择spring web和springboot的版本为2.2.6**![image-20200419162553593](picture\Springboot入门\image-20200419162553593.png)

### 三、 打包部署

**打包springboot应用需要导入插件依赖**

```xml
<build>
    <plugins>
        <!-- 打包插件 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**打开IDEA右侧的maven选项，然后点击package执行打包命令**

 ![image-20200418164749182](picture\Springboot入门\image-20200418164749182.png)

**可以在target目录下找到打包的jar包**

 ![image-20200418164923026](picture\Springboot入门\image-20200418164923026.png)

用命令行窗口进入到jar所在目录，执行`java -jar spring-boot-HelloWorld-1.0-SNAPSHOT.jar`命令。

![image-20200418165135790](picture\Springboot入门\image-20200418165135790.png)

可以看到项目正常运行

 ![image-20200418165211016](picture\Springboot入门\image-20200418165211016.png)

页面也访问成功

## 五、springboot创建彩蛋

修改springboot的启动banner条

 ![image-20200419171022797](picture\Springboot入门\image-20200419171022797.png)

在resource目录下创建banner.txt

```java
////////////////////////////////////////////////////////////////////
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//            佛祖保佑       永不宕机     永无BUG                    //
////////////////////////////////////////////////////////////////////
```

输入该内容，启动springboot就会看到效果了

![image-20200419171138524](picture\Springboot入门\image-20200419171138524.png)

## 六、Hello World 探究

### 1. POM文件

#### 1. 父项目

```xml
<!-- 父依赖 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
</parent>
<!-- 他的父依赖 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.2.6.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

他来真正管理spring boot应用里面的所有依赖版本。

spring boot 的版本管理中心：

以后导入依赖默认是不需要写版本；（没有在dependencies里面管理的依赖自然需要声明版本号）

#### 2. 导入的依赖

```xml
<!-- web场景启动器 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**spring-boot-starter**-==web==

​	spring-boot-starter：springboot场景启动器，帮我们导入了web模块正常运行所依赖的组件；

spring boot将所有的功能场景都**抽取**出来，做成一个个的starters（启动器），只需要在项目里引入这些starter，相关场景的**所有依赖都会导入**进来，要用什么功能就导入什么场景的启动器。

### 2. 主程序类，主入口

```java
/**
 * @SpringBootApplication 来标注一个程序类说明这是一个spring boot应用
 */
@SpringBootApplication
public class HelloWorldApplication {
    public static void main(String[] args) {
        //让springboot应用启动
        SpringApplication.run(HelloWorldApplication.class, args);
    }
}
```

**@SpringBootApplication：**Spring Boot 应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

> @SpringBootConfiguration

作用：SpringBoot的配置类 ，标注在某个类上 ， 表示这是一个SpringBoot的配置类；

点进去得到下面的 @Component@Configurationpublic

```java
@Configuration
public @interface SpringBootConfiguration {}

@Component
public @interface Configuration {}
```

**@Configurationpublic：**说明这是一个配置类 ，配置类就是对应Spring的xml 配置文件。

**@Component：**说明启动类本身也是Spring中的一个组件而已，负责启动应用。

>@EnableAutoConfiguration

作用：告诉SpringBoot开启自动配置功能，这样自动配置才能生效；

---

**@AutoConfigurationPackage ：**自动配置包。

```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}
```

**@Import：**Spring底层注解@import ， 给容器中导入一个组件。

**Registrar.class** 作用：将主启动类的所在包及包下面所有子包里面的所有组件扫描到Spring容器 。

---

**@Import({AutoConfigurationImportSelector.class}) ：给容器导入组件 ；**

**AutoConfigurationImportSelector** ：自动配置导入选择器。

点进这个类看源码：

```java
// 获得候选的配置
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {   
//这里的getSpringFactoriesLoaderFactoryClass（）方法   
//返回的就是我们最开始看的启动自动导入配置文件的注解类；EnableAutoConfiguration    
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());   
Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");    
    return configurations;
}
```

**META-INF/spring.factories**：这是自动配置的核心文件。

这个方法又调用了  SpringFactoriesLoader 类的静态方法！我们进入SpringFactoriesLoader类loadFactoryNames() 方法

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {    
    String factoryClassName = factoryClass.getName();
    //这里它又调用了loadSpringFactories方法
    return (List)loadSpringFactories(classLoader)
        .getOrDefault(factoryClassName, Collections.emptyList());
}
```

进入loadSpringFactories方法

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    //获得classLoader ， 我们返回可以看到这里得到的就是EnableAutoConfiguration标注的类本身
    MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
    if (result != null) {
        return result;
    } else {
        try {
            //去获取一个资源 "META-INF/spring.factories"
            Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
            LinkedMultiValueMap result = new LinkedMultiValueMap();
            //将读取到的资源遍历，封装成为一个Properties
            while(urls.hasMoreElements()) {
                URL url = (URL)urls.nextElement();
                UrlResource resource = new UrlResource(url);
                Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                Iterator var6 = properties.entrySet().iterator();

                while(var6.hasNext()) {
                    Entry<?, ?> entry = (Entry)var6.next();
                    String factoryClassName = ((String)entry.getKey()).trim();
                    String[] var9 = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                    int var10 = var9.length;

                    for(int var11 = 0; var11 < var10; ++var11) {
                        String factoryName = var9[var11];
                        result.add(factoryClassName, factoryName.trim());
                    }
                }
            }
            cache.put(classLoader, result);
            return result;
        } catch (IOException var13) {
            throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var13);
        }
    }
}
```

发现一个多次出现的文件：spring.factories，全局搜索它

![image-20200419215729872](picture\Springboot入门\image-20200419215729872.png)

**META-INF/spring.factories：是自动配置的核心文件**

![image-20200419220451407](picture\Springboot入门\image-20200419220451407.png)

我们在上面的自动配置类随便找一个打开看看，比如 ：WebMvcAutoConfiguration

![image-20200419220531993](picture\Springboot入门\image-20200419220531993.png)

可以看到这些一个个的都是JavaConfig配置类，而且都注入了一些Bean。

**自动配置**真正实现是从classpath中搜寻所有的**META-INF/spring.factories**配置文件 ，并将其中对应的 org.springframework.boot.autoconfigure. 包下的配置项，通过反射实例化为对应标注了 @Configuration的JavaConfig形式的IOC容器配置类 ， 然后将这些都汇总成为一个实例并加载到IOC容器中。

以前我们需要自己配置的，springboot都帮我们配置了。

**结论：**

1. SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值
2. 将这些值作为自动配置类导入容器 ， 自动配置类就生效 ， 帮我们进行自动配置工作；
3. 整个J2EE的整体解决方案和自动配置都在springboot-autoconfigure的jar包中；
4. 它会给容器中导入非常多的自动配置类 （xxxAutoConfiguration）, 就是给容器中导入这个场景需要的所有组件 ， 并配置好这些组件 ；
5. 有了自动配置类 ， 免去了我们手动编写配置注入功能组件等的工作；

### 3.SpringApplication.run分析

分析该方法主要分两部分，一部分是SpringApplication的实例化，二是run方法的执行；

> SpringApplication

**这个类主要做了以下四件事情：**

1、推断应用的类型是普通的项目还是Web项目

2、查找并加载所有可用初始化器 ， 设置到initializers属性中

3、找出所有的应用程序监听器，设置到listeners属性中

4、推断并设置main方法的定义类，找到运行的主类

查看构造器：

```java
public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
    // ......
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    this.setInitializers(this.getSpringFactoriesInstances();
    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = this.deduceMainApplicationClass();
}
```

>run方法流程分析

![img](picture\Springboot入门\1418974-20200309184347408-1065424525.png)