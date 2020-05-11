# springboot配置文件

> 配置文件

SpringBoot使用一个全局的配置文件 ， 配置文件名称是固定的 

- application.properties 
  - 语法结构 ：key=value
- application.yml 或者 application.yaml
  - 语法结构 ：key：空格 value

**配置文件的作用 ：**修改SpringBoot自动配置的默认值，因为SpringBoot在底层都给我们自动配置好了；

比如我们可以在配置文件中修改Tomcat 默认启动的端口号！测试一下！

```properties
server.port=8081
```

## yaml语法

### 1. yaml概述

YAML是 "YAML Ain't a Markup Language" （YAML不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）

**这种语言以数据作为中心，而不是以标记语言为重点！**

以前的配置文件，大多数都是使用xml来配置；比如一个简单的端口配置，我们来对比下yaml和xml

传统xml配置：

```xml
<server>    
    <port>8081<port>
</server>
```

yaml配置：

```yml
server：  
 prot: 8080
```

### 2. yaml基础语法

#### 1. 基本语法

1. 空格不能省略;

2. 以**缩进**来控制层级关系;

3. 缩进的空格数不重要，只要相同层级的元素左对齐即可;

4. 属性和值的大小写都是十分敏感的;

5. 缩进不允许使用tab，只允许空格;
6. '#'表示注释，从这个字符一直到行尾，都会被解析器忽略。

#### 2. 数据类型

YAML 支持以下几种数据类型：

- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
- 纯量（scalars）：单个的、不可再分的值

##### 对象

* 对象键值对使用冒号结构表示 **key: value**，冒号后面要加一个空格。
* 也可以使用 **key:{key1: value1, key2: value2, ...}**。
* 还可以使用缩进表示层级关系；

```yaml
key: 
    child-key: value
    child-key2: value2
#行内写法------------------------------------
key: { child-key: value,child-key2: value2 }
```

示例：

```yaml
student:
    name: zhangsan
    age: 3
#行内写法------------------------------------
student: {name: zhangsan,age: 3}
```

##### 数组

以 **-** 开头的行表示构成一个数组：

```yaml
list:
 - A
 - B
 - C
```

行内写法

```yaml
key: [value1, value2, ...]
```

一个相对复杂的例子：

```yaml
companies:
    -
     id: 1
     name: company1
     price: 200W
    -
     id: 2
     name: company2
     price: 500W
```

行内写法

```yaml
companies: [{id: 1,name: company1,price: 200W},{id: 2,name: company2,price: 500W}]
```

##### 复合结构

对象和数组可以结合使用，形成复合结构。

```yaml
languages:
 - Ruby
 - Perl
 - Python 
websites:
 YAML: yaml.org 
 Ruby: ruby-lang.org 
 Python: python.org 
 Perl: use.perl.org 
```

##### 纯量

纯量是**最基本的**，**不可再分**的值。以下数据类型都属于纯量：

- 字符串
- 布尔值
- 整数
- 浮点数
- Null
- 时间
- 日期

> 字符串

字符串是最常见，也是最复杂的一种数据类型。字符串默认不使用引号表示。

```yaml
str: 这是一行字符串
```

如果字符串之中包含空格或特殊字符，需要放在引号之中。

```yaml
str: '内容： 字符串' 
```

单引号和双引号都可以使用，双引号不会对特殊字符转义。

```yaml
s1: '内容\n字符串'	#输出 内容\n字符串
s2: "内容\n字符串"	#输出 内容 换行 字符串
```

单引号之中如果还有单引号，必须连续使用两个单引号转义。

```
str: 'labor''s day' #输出 labor's day
```

字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格。

```yaml
str: 这是一段
  多行
  字符串
```

多行字符串可以使用|保留换行符，也可以使用>折叠换行。

```yaml
this: |
  Foo
  Bar	#输出 Foo\nBar\n
that: >
  Foo
  Bar	#输出 Foo Bar\n
```

+表示保留文字块末尾的换行，-表示删除字符串末尾的换行。

```yaml
s1: |
  Foo
		#输出 Foo\n
s2: |+
  Foo

		#输出 Foo\n\n
s3: |-
  Foo	#输出 Foo
```

字符串之中可以插入 HTML 标记。

```yaml
message: |
  <p style="color: red">
    段落
  </p>
```

> 布尔值

```yml
boolean: 
    - TRUE  #true,True都可以
    - FALSE  #false，False都可以
```

> 整数

```yaml
int:
    - 123
    - 0b1010_0111_0100_1010_1110    #二进制表示
```

> 浮点数

```yaml
float:
    - 3.14
    - 6.8523015e+5  #可以使用科学计数法
```

> NULL

使用`~`表示null

```yaml
null:
    parent: ~  
```

> 时间

时间使用ISO 8601格式，时间和日期之间使用T连接，最后使用+代表时区

```yaml
datetime: 
    -  2018-02-17T15:02:31+08:00    
```

> 日期

日期必须使用ISO 8601格式，即yyyy-MM-dd

```yaml
date:
    - 2018-02-17    
```

##### 引用

锚点`&`和别名`*`，可以用来引用。

**&** 用来建立锚点（defaults），**<<** 表示合并到当前数据，***** 用来引用锚点。

```yaml
defaults: &defaults
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  <<: *defaults

test:
  database: myapp_test
  <<: *defaults
```

等同于

```yaml
defaults:
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  adapter:  postgres
  host:     localhost

test:
  database: myapp_test
  adapter:  postgres
  host:     localhost
```

另一个例子:

```yaml
- &showell Steve 
- Clark 
- Brian 
- Oren 
- *showell
#输出	[ 'Steve', 'Clark', 'Brian', 'Oren', 'Steve' ]
```

## 注入配置文件

yaml文件更强大的地方在于，他可以给我们的实体类直接注入匹配值

### 1. beans注入

1、在springboot项目中的resources目录下新建一个文件 application.yml

2、编写一个实体类 Dog；

```java
package com.xuwei.pojo;

import org.springframework.stereotype.Component;

@Component  //注册bean到容器中
@Data 
@AllArgsConstructor 
@NoArgsConstructor  
public class Dog {
    private String name;
    private Integer age;
}

```

以下三个都是lombok的注解：

* **@Data** ：是以下注解的集合：@ToString @EqualsAndHashCode @Getter @Setter @RequiredArgsConstructor
* **@AllArgsConstructor**：生成全参构造器，即有参构造函数
* **@NoArgsConstructor**：生成无参构造函数；

3、我们用@Value给bean注入属性值，给狗狗类测试一下

```java
@Component  //注册bean到容器中
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Dog {
    @Value("大黄")
    private String name;
    @Value("3")
    private Integer age;
}
```

4、在SpringBoot的测试类下注入狗狗输出一下

```java
@SpringBootTest
class SpringBoot02ConfigApplicationTests {
    @Autowired	//将狗的属性值自动注入进来
    Dog dog;

    @Test
    void contextLoads() {
        System.out.println(dog);//打印查看狗的属性值
    }
}
```

结果成功输出，@Value注入成功。

![image-20200421200957256](picture\springboot配置文件\image-20200421200957256.png)

### 2. 加载全局配置文件

1、我们在编写一个复杂一点的实体类：Person 类

```java
@Component	 //注册bean到容器中
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

2、我们来使用yaml配置的方式进行注入，编写一个yaml配置。

```
Person:
  name: xuwei
  age: 18
  happy: false
  birth: 1999/01/01
  maps: {k1:v1,k2:v2}
  lists:
    - code
    - sport
    - music
  dog:
    name: 旺财
    age: 3
```

3、将person这个对象的值，注入到类中。

@ConfigurationProperties作用：

* 将配置文件中配置的每一个属性的值，映射到这个组件中；
* 告诉SpringBoot将本类中的**所有属性**和**配置文件中相关的配置**进行绑定
* 参数 prefix = “person” : 将配置文件中的person下面的所有属性一一对应

```java
@Component
@Data
@AllArgsConstructor
@NoArgsConstructor
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

4、IDEA 提示，springboot配置注解处理器没有找到，让我们看文档，我们可以查看文档，找到一个依赖！

![image-20200421210044977](picture\springboot配置文件\image-20200421210044977.png)

由于文档只到2.1.9，所以只能改成2.1.9的

![image-20200421210209307](picture\springboot配置文件\image-20200421210209307.png)

```xml
<!-- 导入配置文件处理器，配置文件进行绑定就会有提示，需要重启 -->
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
      <optional>true</optional>
</dependency>
```

5、确认以上配置都OK之后，我们去测试类中测试一下：

```java
@SpringBootTest
class SpringBoot02ConfigApplicationTests {
    @Autowired	//自动注入person的属性值
    Person person;

    @Test
    void contextLoads() {
        System.out.println(person);
    }
}
```

结果：所有值全部注入成功。

![image-20200421210934438](picture\springboot配置文件\image-20200421210934438.png)

注意：配置文件的key 值 和 属性的值必须一致，否则结果输出该属性为null。

### 3. 加载指定的配置文件

**@PropertySource ：**加载指定的配置文件。

**@configurationProperties**：默认从全局配置文件中获取值；

1、在resources目录下新建一个**person.properties**文件

 ```properties
name=xuwei
 ```

2、然后在代码中指定加载person.properties文件

```java
@Component
@Data
@AllArgsConstructor
@NoArgsConstructor
@PropertySource("classpath:person.properties")
public class Person {
    @Value("${name}")
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

3、再次输出测试一下：指定配置文件绑定成功！

![image-20200421213458123](picture\springboot配置文件\image-20200421213458123.png)

> 配置文件占位符

配置文件还可以编写占位符生成随机数

```yaml
person:
  name: xuwei${random.uuid} #随机 uuid
  age: ${random.int} #随机整数
  happy: false
  birth: 1999/01/01
  maps: {k1:v1,k2:v2}
  lists:
    - code
    - sport
    - music
  dog:
    name: ${person.hello:hello}_旺财 #占位符，如果person.hello存在则输出，不存在则输出hello
    age: 3
```

结果：因为person.hello的值不存在，所以输出了hello

![image-20200421222354054](picture\springboot配置文件\image-20200421222354054.png)

### 4. properties配置

配置文件除了yml还有我们之前常用的properties 。

【注意】properties配置文件在写中文的时候，会有乱码 ， 我们需要去IDEA中设置编码格式为UTF-8；

settings-->FileEncodings 中配置；

![image-20200421222642197](picture\springboot配置文件\image-20200421222642197.png)

1、新建一个实体类User

```java
@Component //注册bean
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private String name;
    private Integer age;
    private String gender;
}
```

2、编辑配置文件 user.properties

```properties
user.name=xuwei
user.age=18
user.gender=男
```

3、在User类上使用@Value来进行注入

```java
@Component //注册bean
@Data
@AllArgsConstructor
@NoArgsConstructor
@PropertySource("classpath:user.properties")//指定位置加载配置文件
public class User {
    @Value("${user.name}")  //从配置文件中取值
    private String name;
    @Value("#{2*9}")    // #{SPEL} Spring的EL表达式
    private Integer age;
    @Value("男")     // 字面量
    private String gender;
}
```

4、Springboot测试

```java
@SpringBootTest
class SpringBoot02ConfigApplicationTests {
    @Autowired	//自动注入user的属性值
    User user;
    @Test
    void contextLoads() {
        System.out.println(user);//打印user的属性值
    }
}
```

结果

![image-20200421223850165](picture\springboot配置文件\image-20200421223850165.png)

### 5. 对比小结

@Value这个使用起来并不友好！我们需要为每个属性单独注解赋值，比较麻烦；我们来看个功能对比图

|                | @ConfigurationProperties | @Value     |
| -------------- | ------------------------ | ---------- |
| 功能           | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定       | 支持                     | 不支持     |
| SPEL           | 不支持                   | 支持       |
| JSR303数据校验 | 支持                     | 不支持     |
| 复杂类型封装   | 支持                     | 不支持     |

1. @ConfigurationProperties只需要写一次即可 ， @Value则需要每个字段都添加。
2. 松散绑定：比如我的yaml中写的last-name，这个和lastName是一样的， - 后面跟着的字母默认是大写的。这就是松散绑定。
3. JSR303数据校验 ， 这个就是我们可以在字段是增加一层过滤器验证 ， 可以保证数据的合法性。
4. 复杂类型封装，yaml中可以封装对象 ， 使用value就不支持。

结论：

* 配置yaml和配置properties都可以获取到值 ，但推荐使用yaml；
* 如果我们在某个业务中，只需要获取配置文件中的某个值，可以使用 @value；
* 如果编写了一个JavaBean来和配置文件进行一一映射，就使用@configurationProperties。

### 6. JSR303数据校验

JSR：**Java Specification Requests**，意思是Java 规范提案

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {
    @Email(regexp = "@*.*")
    private String name;
    private Integer age;
    private Boolean happy;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

使用JSR303校验需要使用@Validated注解。

约束注解都在`javax.validation.constraints`包下，以下列举几个常用的

| 约束      | 详细信息                           |
| --------- | ---------------------------------- |
| @Email    | 被注释的元素必须是电子邮箱地址     |
| @Length   | 被注释的字符串大小必须在指定范围内 |
| @NotEmpty | 被注释的字符串不能为null也不能为空 |
| @Range    | 被注释的元素必须在合适的范围内     |
| @NotNull  | 被注释的元素不能为null             |

## bean注入方式

### xml配置文件注入

在resource文件夹下新建一个bean.xml，并注入HelloService

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="com.xuwei.service.HelloService"/>
</beans>
```

然后在启动类上通过`@ImportResource`扫描该bean配置文件

```java
@ImportResource({"classpath:bean.xml"})
@SpringBootApplication
public class SpringBoot02ConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBoot02ConfigApplication.class, args);
    }
}
```

测试是否注入成功

```java
import org.springframework.context.ApplicationContext;	
@Autowired
ApplicationContext ioc;
@Test
void testHelloService() {
    //测试容器中是否包含helloService这个bean
    boolean helloService = ioc.containsBean("helloService");
    System.out.println(helloService);
}
```

测试成功

 ![image-20200503174537447](picture\springboot配置文件\image-20200503174537447.png)

### 注解方式注入

@Configuration：指明当前类是一个配置类，就是用来替代之前的spring配置文件

@Bean：将方法的返回值添加到容器中，容器的id默认就是方法名

新建一个HelloConfig，加上@Configuration，在通过@Bean注解将new HelloService()注入容器中

```java
@Configuration
public class HelloConfig {
    @Bean //将方法的返回值添加到容器中，容器的id默认就是方法名
    public HelloService helloService(){
        System.out.println("配置类@bean给容器中注入组件");
        return new HelloService();
    }
}
```

测试是否注入成功

 ![image-20200503192143371](picture\springboot配置文件\image-20200503192143371.png)

## Profile

### 1、多Profile文件

我们在主配置文件编写的时候，配置文件名可以是application-{profile}.properties/yml

默认使用application.properties的方式

### 2、yml支持多文档块方式

用`---`来分割不同模块的yaml配置

```yml
server:
  port: 8081
spring:
  profiles:
  #激活指定的文档块
    active: dev 
---
#dev文档块
server:
  port: 8082
spring:
  profiles: dev
---
#prod文档块
server:
  port: 8084
spring:
  profiles: prod
```

### 3、激活指定profile

1. 在主配置文件中指定`spring.profiles.active=dev`

2. 命令行：`--spring.profiles.active=dev`

   **idea配置**

![image-20200503212053514](picture\springboot配置文件\image-20200503212053514.png)

​	**打成jar包**

​	用`java -jar -xxx -Dspring.profiles.active=dev`启动

3. jvm参数：`-Dspring.profiles.active=dev`，

![image-20200503213042344](picture\springboot配置文件\image-20200503213042344.png)

## 配置文件加载位置

spring boot 启动会扫描以下位置的application.properties或者application.yml文件作为springboot的默认配置文件

- file：./config/
- file：./
- classpath：/config/
- classpath：/

1. 以上是按照优先级从高到低的配置，所有位置的文件都会被加载，高优先级配置内容会覆盖低优先级配置内容。

2. 也可以通过spring.config.location来改变默认配置