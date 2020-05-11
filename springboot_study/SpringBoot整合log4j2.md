# springboot整合log4j2日志工具

## 一、常用日志框架 

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j……

> 什么是日志门面和日志实现？ 

日志门面：是日志实现的抽象层。

日志实现：具体的日志功能的实现。

SpringBoot默认选用SLF4j（日志门面）和Logback（日志实现）。

> 日志实现

- java.util.logging（JUL）：是JDK在1.4版本中引入的Java原生日志框架
- Log4j：Apache的一个开源项目，可以控制日志信息输送的目的地是控制台、文件、GUI组件等，可以控制每一条日志的输出格式，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。虽然已经停止维护了，但目前绝大部分企业都是用的log4j。
- LogBack：是Log4j的一个改良版本
- Log4j2：Log4j2已经不仅仅是Log4j的一个升级版本了，它从头到尾都被重写了

## 二、日志门面

> 上述介绍的是一些日志框架的实现，这里我们需要用日志门面来解决系统与日志实现框架的耦合性。

* JCL（jakarta commons logging）：Commons Logging 的 目的是为 **"所有的Java日志实现"**提供一个**统一的接口**，它自身也提供一个日志的实现，但是功能非常弱（SimpleLog），所以一般**不会单独使用它**。他允许开发人员使用不同的具体日志实现工具，注意JCL最后一次更新在2014年，**已经过时**。
* SLF4J（Simple Logging Facade for Java）：简单日志门面，它不是一个真正的日志实现，而是一个抽象层（ abstraction layer），它允许你在后台使用任意一个日志实现。
* jboss-logging：是一款类似于slf4j的日志框架，主要用于日志代理，内部采用log4j、log4j2、logback、jdk-logging等框架实现，**很少使用**。

![img](picture\springboot整合log4j2\35274748.jpg)

​	前面介绍的几种日志框架一样，每一种日志框架都有自己单独的API，要使用对应的框架就要使用其对应的API，这就大大的增加应用程序代码对于日志框架的耦合性。

​	使用了slf4j后，对于应用程序来说，无论底层的日志框架如何变，应用程序不需要修改任意一行代码，就可以直接上线了。

## 三、为什么选用log4j2

> 相比与其他的日志系统，log4j2丢数据这种情况少；disruptor技术，在多线程环境下，性能高于logback等10倍以上；利用jdk1.5并发的特性，减少了死锁的发生；

log4j2性能评测：

![img](picture\springboot整合log4j2\45608002.jpg)

![img](picture\springboot整合log4j2\31312446.jpg)

- 可以看到在同步日志模式下, Logback的性能是最糟糕的。
- log4j2的性能无论在同步日志模式还是异步日志模式下都是最佳的。

log4j2优越的性能其原因在于log4j2使用了LMAX,一个无锁的线程间通信库代替了,logback和log4j之前的队列. 并发性能大大提升。

## 四、整合步骤

### 引入jar包

> springboot默认是用logback的日志框架的，所以需要排除logback，不然会出现jar依赖冲突的报错。

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-web</artifactId>  
    <exclusions><!-- 去掉springboot默认配置 -->  
        <exclusion>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-starter-logging</artifactId>  
        </exclusion>  
    </exclusions>  
</dependency> 

<dependency> <!-- 引入log4j2依赖 -->  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-log4j2</artifactId>  
</dependency> 
```

### 配置文件

**配置文件名称**

log4j 2.x版本不再支持像1.x中的.properties后缀的文件配置方式，2.x版本配置文件后缀名只能为".xml",".json"或者".jsn"，默认使用log4j2.xml来命名。

**项目中的存放位置**

系统选择配置文件的优先级(从先到后)如下：

​	(1)classpath下的名为log4j2-test.json 或者log4j2-test.jsn的文件.

​	(2)classpath下的名为log4j2-test.xml的文件.

​	(3)classpath下名为log4j2.json 或者log4j2.jsn的文件.

​	(4)classpath下名为log4j2.xml的文件.

​	我们一般默认使用`log4j2.xml`进行命名。如果本地要测试，可以把log4j2-test.xml放到classpath，而正式环境使用log4j2.xml，则在打包部署的时候不要打包log4j2-test.xml即可。

**如果自定义了文件名或者文件位置，需要在application.yml中配置**

```yml
logging:
  config: xxxx.xml
  level:
    cn.jay.repository: trace
```

### 配置文件模版

> log4j是通过一个.properties的文件作为主配置文件的，而现在的log4j2则已经弃用了这种方式，采用的是.xml，.json或者.jsn这种方式来做。

日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Configuration后面的status，这个用于设置log4j2自身内部的信息输出，可以不设置，如果不设置的话，默认值就为ERROR，当设置成trace时，你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置文件和重新配置本身，设置间隔秒数-->
<!--<configuration monitorInterval="5" status="ERROR">-->
<configuration monitorInterval="5">
  <!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->

  <!--变量配置-->
  <Properties>
    <!-- 格式化输出：%d表示日期格式化；%t表示线程名；%-5level：级别从左显示5个字符宽度；-->
    <!-- %logger{36} 表示 Logger 的类名名字最长36个字符 -->
    <!-- %L：表示行号；%M：表示方法名；%msg：日志消息；%n是换行符 -->
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS}  %-5level [%t]  %class{36}(%L) %M : %msg%n" />
    <!-- 定义日志存储的路径 -->
    <property name="FILE_PATH" value="更换为你的日志路径" />
    <property name="FILE_NAME" value="更换为你的项目名" />
  </Properties>

  <appenders>

    <console name="Console" target="SYSTEM_OUT">
      <!--输出日志的格式-->
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <!--控制台只输出level及其以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
    </console>

    <!--文件会打印出所有信息，这个log每次运行程序会自动清空，由append属性决定，适合临时测试用-->
    <File name="Filelog" fileName="${FILE_PATH}/test.log" append="false">
      <PatternLayout pattern="${LOG_PATTERN}"/>
    </File>

    <!-- 这个会打印出所有的info及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
    <RollingFile name="RollingFileInfo" fileName="${FILE_PATH}/info.log" filePattern="${FILE_PATH}/${FILE_NAME}-INFO-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

    <!-- 这个会打印出所有的warn及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
    <RollingFile name="RollingFileWarn" fileName="${FILE_PATH}/warn.log" filePattern="${FILE_PATH}/${FILE_NAME}-WARN-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

    <!-- 这个会打印出所有的error及以下级别的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
    <RollingFile name="RollingFileError" fileName="${FILE_PATH}/error.log" filePattern="${FILE_PATH}/${FILE_NAME}-ERROR-%d{yyyy-MM-dd}_%i.log.gz">
      <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
      <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
      <PatternLayout pattern="${LOG_PATTERN}"/>
      <Policies>
        <!--interval属性用来指定多久滚动一次，默认是1 hour-->
        <TimeBasedTriggeringPolicy interval="1"/>
        <SizeBasedTriggeringPolicy size="10MB"/>
      </Policies>
      <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件开始覆盖-->
      <DefaultRolloverStrategy max="15"/>
    </RollingFile>

  </appenders>

  <!--Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。-->
  <!--然后定义loggers，只有定义了logger并引入的appender，appender才会生效-->
  <loggers>
	<!--若是additivity设为false，则子Logger只会在自己的appender里输出，而不会在父Logger的appender里输出。-->
    <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
    <logger name="org.mybatis" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </logger>
    <Logger name="org.springframework" level="info" additivity="false">
      <AppenderRef ref="Console"/>
    </Logger>

    <root level="info"> <!--其他logger输出的日志信息-->
      <appender-ref ref="Console"/>
      <appender-ref ref="Filelog"/>
      <appender-ref ref="RollingFileInfo"/>
      <appender-ref ref="RollingFileWarn"/>
      <appender-ref ref="RollingFileError"/>
    </root>
  </loggers>

</configuration>
```

**用户自定义日志级别**

```xml
<CustomLevels>
    <CustomLevel name="USER" intLevel="350" />
</CustomLevels>
```

内置于Log4J的标准日志级别

| 标准等级 |     intLevel      |
| :------: | :---------------: |
|   OFF    |         0         |
|  FATAL   |        100        |
|  ERROR   |        200        |
|   WARN   |        300        |
|   INFO   |        400        |
|  DEBUG   |        500        |
|  TRACE   |        600        |
|   ALL    | Integer.MAX_VALUE |

## 配置参数简介

### 1、日志级别

> 机制：如果一条日志信息的级别大于等于配置文件的级别，就记录。

- trace：追踪，就是程序推进一下，可以写个trace输出
- debug：调试，一般作为最低级别，trace基本不用。
- info：输出重要的信息，使用较多
- warn：警告，有些信息不是错误信息，但也要给程序员一些提示。
- error：错误信息。用的也很多。
- fatal：致命错误。

### 2、输出源

- CONSOLE（输出到控制台）
- FILE（输出到文件）

### 3、格式

- SimpleLayout：以简单的形式显示
- HTMLLayout：以HTML表格显示
- PatternLayout：自定义形式显示

### 4、 PatternLayout自定义日志布局

```yml
%d{yyyy-MM-dd HH:mm:ss, SSS} : 日志生产时间,输出到毫秒的时间
%-5level : 输出日志级别，-5表示左对齐并且固定输出5个字符，如果不足在右边补0
%c : logger的名称(%logger)
%t,%thread : 输出当前线程名称
%p : 日志输出格式
%m,%msg : 日志内容，即 logger.info("message")
%n : 换行符
%C,%class : Java类名(%F)
%L,%line : 行号
%M,%method : 方法名
%l : 输出语句所在的行数, 包括类名、方法名、文件名、行数
hostName : 本地机器名
hostAddress : 本地ip地址
```

## Log4j2配置详解

### 1. Configuration根节点

有两个属性:

* status	
* monitorinterval

有两个子节点:

* Appenders
* Loggers(表明可以定义多个Appender和Logger).

status：用来指定log4j本身的打印日志的级别。
monitorinterval：用于指定log4j自动重新配置的监测间隔时间，单位是s,最小是5s。 

### 2. Appenders节点

常见的有四种子节点:Console、File、RollingFile、Socket.

> Console节点

**用来定义输出到控制台的Appender。**

- name：指定Appender的名字。
- target：SYSTEM_OUT 或 SYSTEM_ERR,一般只设置默认:SYSTEM_OUT。
- PatternLayout：输出格式，不设置默认为:%m%n。

> File节点

**用来定义输出到指定位置的文件的Appender。**

- name：指定Appender的名字。
- fileName：指定输出日志的目的文件带全路径的文件名。
- PatternLayout：输出格式，不设置默认为:%m%n。

> RollingFile节点

**用来定义超过指定条件自动删除旧的创建新的Appender。**

- name：指定Appender的名字。
- fileName：指定输出日志的目的文件带全路径的文件名。
- PatternLayout：输出格式，不设置默认为:%m%n。
- filePattern ：指定当发生Rolling时，文件的转移和重命名规则。
- Policies:指定滚动日志的策略，就是什么时候进行新建日志文件输出日志。
- TimeBasedTriggeringPolicy：Policies子节点，基于时间的滚动策略，interval属性用来指定多久滚动一次，默认是1 hour。modulate=true用来调整时间：比如现在是早上3am，interval是4，那么第一次滚动是在4am，接着是8am，12am...而不是7am。
- SizeBasedTriggeringPolicy：Policies子节点，基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小。
- DefaultRolloverStrategy：用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的，创建新的(通过max属性)。

>  Socket节点

**用来定义将日志输出至别的服务器上。**

```xml
<Socket name="server" host="10.140.254.54" port="12345" protocol="UDP">
       <PatternLayout pattern="%d{MMM dd HH:mm:ss.SSS} %-5p %X{HostName} PRISM %X{AccountID}/%X{ApplicationID}/%X{SessionGUID}/%X{SessionNumber}/1/[%X{CallID}] [%t] %m%n" />
</Socket>
```

### 3. Loggers节点

**常见的有两种:Root和Logger。**

Root节点用来指定项目的根日志，如果没有单独指定Logger，那么就会默认使用该Root日志输出。

- level:日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < AppenderRef：Root的子节点，用来指定该日志输出到哪个Appender。
- Logger节点：用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。
- level：日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF。
- name:用来指定该Logger所适用的类或者类所在的包全路径,继承自Root节点.
- AppenderRef：Logger的子节点，用来指定该日志输出到哪个Appender,如果没有指定，就会默认继承自Root.如果指定了，那么会在指定的这个Appender和Root的Appender中都会输出，此时我们可以设置Logger的additivity="false"只在自定义的Appender中进行输出。

## 简单使用

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogExampleOther {
  private static final Logger log =  LoggerFactory.getLogger(LogExampleOther.class);
  
  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}
```

## 使用lombok工具简化创建Logger类

> lombok就是一个注解工具jar包，能帮助我们省略一繁杂的代码。具体介绍可以看[这篇教程](https://www.cnblogs.com/keeya/p/9929617.html)。

```java
@Slf4j
public class LogExampleOther {
  
  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}
```

## 设置控制台打印彩色日志

在Log4j 2.10以前的版本，pattern中配置`%highlight`属性是可以正常打印彩色日志的

例如：

```
pattern: "%d{yyyy-MM-dd HH:mm:ss.SSS} %highlight{%-5level} [%t] %highlight{%c{1.}.%M(%L)}: %msg%n"
```

但是更新到2.10版本以后，控制台中就无法显示彩色日志了，各种级别的日志混杂在一起，难以阅读

通过查阅[官方文档](http://logging.apache.org/log4j/2.x/manual/layouts.html#enable-jansi)，发现在2.10版本以后，Log4j2默认关闭了Jansi（一个支持输出ANSI颜色的类库）

```html
ANSI Styling on Windows
ANSI escape sequences are supported natively on many platforms but are not by default on Windows. To enable ANSI support add the Jansi jar to your application and set property log4j.skipJansi to false. This allows Log4j to use Jansi to add ANSI escape codes when writing to the console.

NOTE: Prior to Log4j 2.10, Jansi was enabled by default. The fact that Jansi requires native code means that Jansi can only be loaded by a single class loader. For web applications this means the Jansi jar has to be in the web container's classpath. To avoid causing problems for web applications, Log4j will no longer automatically try to load Jansi without explicit configuration from Log4j 2.10 onward.
```

可见，配置 log4j.skipJansi 这个全局属性即可。

IDEA中，点击右上角->Edit Configurations，在VM options中添加

```properties
-Dlog4j.skipJansi=false
```

启动应用，显示效果如下：

![image-20200503145218880](picture\springboot整合log4j2\image-20200503145218880.png)