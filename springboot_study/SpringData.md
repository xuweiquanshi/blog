# SpringData

## SpringData简介

​	对于数据访问层，无论是 SQL(关系型数据库) 还是 NOSQL(非关系型数据库)，Spring Boot 底层都是采用 Spring Data 的方式进行统一处理。

​	Spring Boot 底层都是采用 Spring Data 的方式进行统一处理各种数据库，Spring Data 也是 Spring 中与 Spring Boot、Spring Cloud 等齐名的知名项目。

Sping Data 官网：https://spring.io/projects/spring-data

数据库相关的启动器 ：可以参考[官方文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/htmlsingle/#using-boot-starter)

## 整合JDBC

### 数据源

创建测试项目测试数据源

1、新建一个项目测试：springboot-data-jdbc ; 引入相应的模块！基础模块

![image-20200505134027060](picture\SpringData\image-20200505134027060.png)

2、项目建好之后，查看pom.xml，发现自动导入了如下的启动器：

```xml
<!--jdbc-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<!--mysql驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

3、编写yaml配置文件连接数据库；

```yml
spring:
  datasource:
    username: root
    password: 123456
    #?serverTimezone=UTC解决 mysql8.0 时区的报错
    url: jdbc:mysql://localhost:3306/company?characterEncoding=UTF-8&serverTimezone=UTC&useUnicode=true
    driver-class-name: com.mysql.cj.jdbc.Driver
```

4、配置完这一些东西后，就可以直接使用了，因为SpringBoot已经默认进行了自动配置；去测试类测试一下。

```java
@SpringBootTest
class SpringbootDataJdbcApplicationTests {
    //DI注入数据源
    @Autowired
    DataSource dataSource;
    @Test
    public void contextLoads() throws SQLException {
        //查看默认数据源
        System.out.println(dataSource.getClass());
        //获得连接
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        //关闭连接
        connection.close();
    }
}
```

结果：可以看到默认给配置的数据源为 : class com.zaxxer.hikari.HikariDataSource。

全局搜索一下，找到数据源的所有自动配置都在 ：DataSourceAutoConfiguration文件：

```java
@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
			DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.Generic.class,
			DataSourceJmxConfiguration.class })
protected static class PooledDataSourceConfiguration {

}
```

这里导入的类都在 DataSourceConfiguration 配置类下，可以看出 Spring Boot 2.2.6 默认使用HikariDataSource 数据源，而以前版本，如 Spring Boot 1.5 **默认**使用 `org.apache.tomcat.jdbc.pool.DataSource `作为数据源。

HikariDataSource 号称 Java WEB 当前**速度最快的数据源**，相比于传统的 C3P0 、DBCP、Tomcat jdbc 等连接池更加优秀；

可以使用` spring.datasource.type` 指定自定义的数据源类型，值为要使用的连接池实现的**完全限定名**。

### JDBCTemplate

1、有了数据源(com.zaxxer.hikari.HikariDataSource)，然后可以拿到数据库连接(java.sql.Connection)，有了连接，就可以使用原生的 JDBC 语句来操作数据库；

2、即使不使用第三方第数据库操作框架，如 MyBatis等，Spring 本身也对原生的JDBC 做了轻量级的封装，即JdbcTemplate。

3、数据库操作的所有 CRUD 方法都在 JdbcTemplate 中。

4、Spring Boot 不仅提供了默认的数据源，同时默认已经配置好了 JdbcTemplate 放在了容器中，程序员只需自己注入即可使用

5、JdbcTemplate 的自动配置是依赖 `org.springframework.boot.autoconfigure.jdbc` 包下的 JdbcTemplateConfiguration 类

**JdbcTemplate主要提供以下几类方法：**

- `execute`方法：可以用于执行任何SQL语句，一般用于执行DDL语句（ddl是数据库模式定义语言，常见的DDL语句例如：创建数据库：CREATE DATABASE；创建表：CREATE TABLE。)；

  **扩展：**SQL语言包括四种主要程序设计语言类别的语句：数据定义语言(DDL)，数据操作语言(DML)，数据控制语言(DCL)和事务控制语言（TCL）。

- `update`及`batchUpdate`方法：update方法用于执行新增、修改、删除等语句；batchUpdate方法用于执行批处理相关语句；

- `query`及`queryForXXX`方法：用于执行查询相关语句；

- `call`方法：用于执行存储过程、函数相关语句。

### 测试类

```java
@RestController
public class JDBCController {
    /**
     * Spring Boot 默认提供了数据源，默认提供了 org.springframework.jdbc.core.JdbcTemplate
     * JdbcTemplate 中会自己注入数据源，用于简化JDBC操作
     * 还能避免一些常见的错误,使用起来也不用再自己来关闭数据库连接
     */
    @Autowired
    private JdbcTemplate jdbcTemplate;

    //查询所有用户
    @RequestMapping("/userList")
    public List<Map<String, Object>> userList() {
        String sql = "select * from employee";
        return jdbcTemplate.queryForList(sql);
    }

    //添加一个用户
    @RequestMapping("/user/add")
    public Boolean userAdd() {
        String sql = "insert into employee (lastName,email,birth,gender,d_id) values ('张三','12345@qq.com','" + new Date().toLocaleString() + "',1,2)";
        jdbcTemplate.execute(sql);
        return true;
    }

    //根据用户id查询用户信息
    @RequestMapping("/user/{id}")
    public Object user(@PathVariable("id") Integer id) {
        String sql = "select emp.*,dep.* from employee emp,department dep where d_id=dep.id and emp.id = ?";
        return jdbcTemplate.queryForMap(sql, id);
    }

    //修改指定用户信息
    @RequestMapping("/user/update/{id}")
    public Boolean userUpdate(@PathVariable("id") Integer id) {
        String sql = "update employee set lastName=?,email=? where id = ?";
        Object[] objects = new Object[3];
        objects[0] = "虚伪";
        objects[1] = "123456@qq.com";
        objects[2] = id;
        jdbcTemplate.update(sql, objects);
        return true;
    }

    //删除指定用户
    @RequestMapping("/user/delete/{id}")
    public Boolean userDelete(@PathVariable("id") Integer id) {
        String sql = "delete from employee where id = ?";
        jdbcTemplate.update(sql, id);
        return true;
    }
}
```

> 数据库

部门表

```sql
create table department
(
    id int auto_increment primary key,
    departmentName varchar(255) null
);
```

员工表

```sql
create table employee
(
    id int auto_increment primary key,
    lastName varchar(255) null,
    email    varchar(255) null,
    gender   int          null,
    birth    datetime     null,
    d_id     int          null
);
```

测试请求，结果正常；

## 整合Druid

### Druid简介

Java程序很大一部分要操作数据库，为了提高性能操作数据库的时候，又不得不使用数据库连接池。

Druid 是阿里巴巴开源平台上一个数据库连接池实现，结合了 C3P0、DBCP 等 DB 池的优点，同时加入了**日志监控**。

Druid 可以很好的监控 DB 池连接和 SQL 的执行情况，天生就是针对监控而生的 DB 连接池。

Druid已经在阿里巴巴部署了超过600个应用，经过一年多生产环境大规模部署的严苛考验。

Spring Boot 2.0 以上默认使用 Hikari 数据源，可以说 Hikari 与 Driud 都是当前 Java Web 上最优秀的数据源。

Github地址：https://github.com/alibaba/druid/

### 配置参数

**com.alibaba.druid.pool.DruidDataSource 基本配置参数如下：**

| 配置                         | 缺省值             | 说明                                                         |
| ---------------------------- | ------------------ | ------------------------------------------------------------ |
| name                         |                    | 配置这个属性的意义在于没如果存在多个数据源，监控的时候可以通过名字来区分开来。如果没有配置，将会生成一个名字，格式是"DataSource-"+System.identityHashCode(this) |
| jdbcUrl                      |                    | 连接数据库的url，不同数据库不一样                            |
| username                     |                    | 连接数据库的用户名                                           |
| password                     |                    | 连接数据库的密码。如果你不希望密码直接写在配置文件中，可以使用ConfigFilter |
| driverClassName              | 根据url自动识别    | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName(建议配置) |
| initialSize                  | 0                  | 初始化时建立物理连接的个数，初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive                    | 8                  | 最大连接池数量                                               |
| maxIdle                      | 8                  | 已经不再使用，配置了也没效果。maxIdle是Druid为了方便DBCP用户迁移而增加的，maxIdle是一个混乱的概念。连接池只应该有maxPoolSize和minPoolSize，druid只保留了maxActive和minIdle，分别相当于maxPoolSize和minPoolSize。 |
| minIdle                      |                    | 最小连接池数量                                               |
| maxWait                      |                    | 获取连接时最大等待时间，单位毫秒，配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁 |
| poolPreparedStatements       | false              | 是否缓存preparedStatement，也就是PsCache。PSCache对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭 |
| maxOpenPreparedStatements    | -1                 | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置打一下，比如说100 |
| validationQuery              |                    | 用来检测连接是否有效的sql，要求是一个查询语句。如果validationQuery为null，testOnBorrow、testOnReturn 、testWhileIdle都不会起作用 |
| validationQueryTimeout       |                    | 单位：秒，检测连接是否有效的超时时间。底层调用jdbc Statement对象的void setQueryTimeout(int seconds)方法 |
| testOnBorrow                 | true               | 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能 |
| testOnReturn                 | false              | 归还连接时执行它validationQuery检测连接是否有效，做了这个配置会降低性能 |
| testWhileIdle                | false              | 建议配置为true，不影响性能，并且保证安全性。申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunMills，执行validationQuery检测连接是否有效 |
| timeBetweenEvictionRunMillis | 1分钟（1.0.14）    | 有两个含义:Destory线程会检测连接的间隔时间testWhileIdle的判断依据，详细看testWhileIdele属性的说明 |
| numTestsPerEvictionRun       |                    | 不再使用，一个DruidDataSource只支持一个EvicationRun          |
| minEvictableIdleTimeMillis   | 30分钟（1.0.14）   | 连接保持空闲而不被驱逐的最长时间                             |
| connectionInitSqls           |                    | 物理连接初始化的时候执行sql                                  |
| exceptionSorter              | 根据dbType自动识别 | 当数据库抛出一些不可恢复的异常时，抛弃连接                   |
| filters                      |                    | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有:监控统计用的filter：stat日志用的filter;log4j防御注入的filter:wall |
| proxyFilters                 |                    | 类型是List<com.alibaba.druid,filter.Filter>，如果同时配置filter和proxyFilters，是组合关系，并非替换关系 |

### 支持的数据库

| 数据库    | 支持状态         |
| --------- | ---------------- |
| mysql     | 支持，大规模使用 |
| oracle    | 支持，大规模使用 |
| sqlserver | 支持             |
| postgres  | 支持             |
| db2       | 支持             |
| h2        | 支持             |
| derby     | 支持             |
| sqlite    | 支持             |
| sybase    | 支持             |

### 配置数据源

1、添加上Druid 数据源依赖。

Druid从0.1.18之后版本都会发布到[maven中央仓库](https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter)中，所以只需要在项目的pom.xml中引入。

```xml
<!-- https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter -->
<!--  druid数据源 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.22</version>
</dependency>
```

2、切换数据源。

​	因为 Spring Boot 2.0 以上默认使用`com.zaxxer.hikari.HikariDataSource`数据源，但可以通过 `spring.datasource.type` 指定数据源。

```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/company?characterEncoding=UTF-8&serverTimezone=UTC&useUnicode=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource # 自定义druid数据源
```

3、数据源切换之后，在测试类中注入 DataSource，然后获取到它，测试输出；

```java
@SpringBootTest
class SpringbootDataJdbcApplicationTests {
    //DI注入数据源
    @Autowired
    DataSource dataSource;
    @Test
    public void contextLoads() throws SQLException {
        //查看默认数据源
        System.out.println(dataSource.getClass());
        //获得连接
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        //关闭连接
        connection.close();
    }
}
```

![image-20200505142837401](picture\SpringData\image-20200505142837401.png)

4、切换成功！既然切换成功，就可以设置数据源连接初始化大小、最大连接数、等待时间等设置项。

```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/company?characterEncoding=UTF-8&serverTimezone=UTC&useUnicode=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      #Spring Boot 默认是不注入这些属性值的，需要自己绑定
      #druid 数据源专有配置
      initialSize: 5	#初始化时建立物理连接的个数
      minIdle: 5	#最小连接池数量
      maxActive: 20	#最大连接池数量
      maxWait: 60000
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true

      #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
      #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
      #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
      filters: stat,wall,log4j
      maxPoolPreparedStatementPerConnectionSize: 20
      useGlobalDataSourceStat: true
      connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
      filter:
        stat:
          enabled: true
      stat-view-servlet:
        enabled: true
```

5、导入Log4j 的依赖

```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

配置Log4j.properties

```properties
#只显示INFO以及INFO以上级别
log4j.rootLogger=INFO, stdout 
# Console output，输出到命令行
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

6、现在需要自己为 DruidDataSource 绑定全局配置文件中的参数，再添加到容器中，而不再使用 Spring Boot 的自动生成，需要添加自定义DruidDataSource组件到容器中，并绑定属性；

```java
@Configuration
public class DruidConfig {
	
    /*
       将自定义的 Druid数据源添加到容器中，不再让 Spring Boot 自动创建
       绑定全局配置文件中的 druid 数据源属性到 com.alibaba.druid.pool.DruidDataSource从而让它们生效
       @ConfigurationProperties(prefix = "spring.datasource.druid")：作用就是将 全局配置文件中
       前缀为 spring.datasource.druid的属性值注入到 com.alibaba.druid.pool.DruidDataSource 的同名参数中
     */
    @ConfigurationProperties(prefix = "spring.datasource.druid")
    @Bean
    public DataSource druidDataSource() {
        return DruidDataSourceBuilder.create().build();
    }
}
```

​	原本引入Druid 配置 druidDataSourc() 可以直接new出来，而换成druid starter之后，就变成了`DruidDataSourceBuilder.create().build()`，去构建 DruidDataSourceWrapper 这个类，因为该类不允许被new出来。 

7、去测试类中测试；

```java
@SpringBootTest
class SpringbootDataJdbcApplicationTests {
    //DI注入数据源
    @Autowired
    DataSource dataSource;
    
    @Test
    public void test() throws SQLException {
        //看一下默认数据源
        System.out.println(dataSource.getClass());
        //获得连接
        Connection connection =   dataSource.getConnection();
        System.out.println(connection);

        DruidDataSource druidDataSource = (DruidDataSource) dataSource;
        System.out.println("druidDataSource 数据源最大连接数：" + druidDataSource.getMaxActive());
        System.out.println("druidDataSource 数据源初始化连接数：" + druidDataSource.getInitialSize());

        //关闭连接
        connection.close();
    }
}
```

![image-20200505150043739](picture\SpringData\image-20200505150043739.png)

### 配置Druid数据源监控

Druid 数据源具有监控的功能，并提供了一个 web 界面方便用户查看，类似安装 路由器 时，提供的默认web页面。

所以需要设置 Druid 的后台管理页面，比如登录账号、密码等；配置后台管理；

```java
//在 DruidConfig 类中配置 Druid 监控管理后台的Servlet；
//后台监控
@Bean
public ServletRegistrationBean<StatViewServlet> statViewServlet() {
    ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean<>(new StatViewServlet(), "/druid/*");

    // 这些参数可以在 com.alibaba.druid.support.http.StatViewServlet的父类 com.alibaba.druid.support.http.ResourceServlet 中找到
    Map<String, String> initParameters = new HashMap<>();
    initParameters.put("loginUsername", "admin"); //后台管理界面的登录账号
    initParameters.put("loginPassword", "123456"); //后台管理界面的登录密码
    
    //后台允许谁可以访问
    //initParameters.put("allow", "localhost")：表示只有本机可以访问
    //initParameters.put("allow", "")：为空或者为null时，表示允许所有访问
    initParameters.put("allow", "");
    //deny：Druid 后台拒绝谁访问
    //initParameters.put("deny", "192.168.1.20");表示禁止此ip访问
    
    //设置初始化参数
    bean.setInitParameters(initParameters);
    return bean;
}
```

配置完毕后，启动项目访问 ：http://localhost:8080/druid/login.html

![image-20200505150700267](picture\SpringData\image-20200505150700267.png)

输入设置的用户名密码，进入

![image-20200505150800867](picture\SpringData\image-20200505150800867.png)

### 配置filter过滤器

```java
//在 DruidConfig 类中配置Druid web监控的filter过滤器
//WebStatFilter：用于配置Web和Druid数据源之间的管理关联监控统计
@Bean
public FilterRegistrationBean<WebStatFilter> webStatFilter() {
    FilterRegistrationBean<WebStatFilter> bean = new FilterRegistrationBean<>();
    bean.setFilter(new WebStatFilter());

    //exclusions：设置哪些请求进行过滤排除掉，从而不进行统计
    Map<String, String> initParameters = new HashMap<>();
    initParameters.put("exclusions", "*.js,*.css,/druid/*,/jdbc/*");
    bean.setInitParameters(initParameters);

    //"/*" 表示过滤所有请求
    bean.setUrlPatterns(Collections.singletonList("/*"));
    return bean;
}
```

## 整合MyBatis

官方文档：http://mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/

Maven仓库地址：https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter

### 整合测试

1、导入 MyBatis 所需要的依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.2</version>
</dependency>
```

2、配置数据库连接信息（不变）

```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://localhost:3306/company?characterEncoding=UTF-8&serverTimezone=UTC&useUnicode=true
    driver-class-name: com.mysql.cj.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      #Spring Boot 默认是不注入这些属性值的，需要自己绑定
      #druid 数据源专有配置
      initialSize: 5	#初始化时建立物理连接的个数
      minIdle: 5	#最小连接池数量
      maxActive: 20	#最大连接池数量
      maxWait: 60000
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true

      #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
      #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
      #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
      filters: stat,wall,log4j
      maxPoolPreparedStatementPerConnectionSize: 20
      useGlobalDataSourceStat: true
      connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
      filter:
        stat:
          enabled: true
      stat-view-servlet:
        enabled: true
```

3、创建实体类，导入 Lombok。（注意：IDEA使用Lombok需要先安装Lombok插件）

Department.java

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Component
public class Department {
    private Integer id;	//部门ID
    private String departmentName; //部门名
}
```

4、创建mapper目录以及对应的 Mapper 接口

DepartmentMapper.java

```java
//@Mapper : 表示本类是一个 MyBatis 的 Mapper
@Mapper
@Repository
public interface DepartmentMapper {

    // 获取所有部门信息
    List<Department> getDepartments();

    // 通过id获得部门
    Department getDepartmentById(Integer id);
}
```

5、对应的Mapper映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xuwei.mapper.DepartmentMapper">
    
    <select id="getDepartments" resultType="Department">
        select * from department;
    </select>

    <select id="getDepartment" resultType="Department" parameterType="int">
        select * from department where id = #{id};
    </select>
</mapper>
```

6、配置文件中mapper路径

```yml
mybatis:
  type-aliases-package: com.xuwei.pojo #别名，表示resultType，parameterType的对象到该包下寻找
  mapper-locations: "classpath:mapper/*.xml"
```

**pom文件配置资源过滤问题**（如果没有问题就不需要配置）

```xml
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.xml</include>
        </includes>
        <filtering>true</filtering>
    </resource>
</resources>
```

7、编写部门的 DepartmentController 进行测试。

```java
@RestController
public class DepartmentController {
    @Autowired
    DepartmentMapper departmentMapper;

    // 查询所有部门信息
    @GetMapping("/Departments")
    public List<Department> getDepartments(){
        return departmentMapper.getDepartments();
    }

    // 根据部门ID查询部门信息
    @GetMapping("/department/{id}")
    public Department getDepartment(@PathVariable("id") Integer id){
        return departmentMapper.getDepartmentById(id);
    }
}
```

8、启动项目访问进行测试

![image-20200505152944878](picture\SpringData\image-20200505152944878.png)

