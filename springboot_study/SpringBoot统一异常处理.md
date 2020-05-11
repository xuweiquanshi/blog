# SpringBoot异常处理

## 简介  

​	日常开发过程中，难免有的程序会因为某些原因抛出异常，而这些异常一般都是利用try ，catch的方式处理异常或者throw，throws的方式抛出异常不管。这种方法对于程序员来说处理也比较麻烦，对客户来说也不太友好，所以我们希望既能方便程序员编写代码，不用过多的自己去处理各种异常编写重复的代码又能提升用户的体验，这时候全局异常处理就显得很重要也很便捷了，是一种不错的选择。

## 一、 全局异常捕获与处理

Springboot对于异常的处理做了不错的支持，它提供了两个可用的注解。

 **@ControllerAdvice：**用来**开启全局的异常捕获**

 **@ExceptionHandler：**说明**捕获哪些异常**，对哪些异常进行处理。

```java
@ControllerAdvice
public class MyExceptionHandler {
    @ExceptionHandler(value =Exception.class)
	public String exceptionHandler(Exception e){
		System.out.println("发生了一个异常"+e);
       	return e.getMessage();
    }
}
```

上面这段代码的意思是，只要是代码运行过程中有异常就会进行捕获，并输出出这个异常。然后我们随便编写一个会发生异常的代码，测试出来的异常是这样的。

![image-20200510230930785](picture\SpringBoot统一异常处理\image-20200510230930785.png)

​	这对于前后端分离来说并不好，前后端分离之后唯一的交互就是json了，我们也希望将后端的异常变成json返回给前端处理，所以就需要统一结果返回和统一异常处理。

## 二、统一结果返回与统一异常

`Result`类：封装返回结果。

```java
public class Result<T> {
    private Integer code;//状态码
    private String message;//提示消息
    private T data;//数据

    public Result() {
    }

    /**
     * @param code 响应码
     * @param message 响应信息
     */
    public Result(Integer code, String message) {
        this.code = code;
        this.message = message;
    }

    /**
     * @param code 响应码
     * @param message 响应信息
     * @param data 数据
     */
    public Result(Integer code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    /**
     * @param resultEnum 自定义枚举类，包含 code 和 message
     */
    public Result(ResultEnum resultEnum) {
        this.code = resultEnum.getCode();
        this.message = resultEnum.getMessage();
    }

    /**
     * @param resultEnum 自定义枚举类，包含 code 和 message
     * @param data 数据
     */
    public Result(ResultEnum resultEnum, T data) {
        this.code = resultEnum.getCode();
        this.message = resultEnum.getMessage();
        this.data = data;
    }

    /**
     * 自定义异常返回的结果
     * @param definitionException 自定义异常处理类
     * @return 返回自定义异常
     */
    public static Result<Object> defineError(DefinitionException definitionException) {
        return new Result<>(definitionException.getErrorCode(), definitionException.getErrorMessage());
    }

    /**
     * 其他异常处理方法返回的结果
     * @param resultEnum 自定义枚举类，包含 code 和 message
     * @return 返回其他异常
     */
    public static Result<Object> otherError(ResultEnum resultEnum) {
        return new Result<>(resultEnum);
    }

    //这里写get和set方法
}
```

?> 注意：其中**省略**了get，set方法。

`ResultEnum`：自定义枚举类。

```java
public enum ResultEnum {
    // 数据操作定义
    SUCCESS(200, "成功"),
    TIME_OUT(130, "访问超时"),
    NO_PERMISSION(403, "拒绝访问"),
    NO_AUTH(401, "未经授权访问"),
    NOT_FOUND(404, "无法找到资源"),
    METHOD_NOT_ALLOWED(405, "不支持当前请求方法"),
    SERVER_ERROR(500, " 服务器运行异常"),
    NOT_PARAM(10001, "参数不能为空"),
    NOT_EXIST_USER_OR_ERROR_PASSWORD(10002, "该用户不存在或密码错误"),
    NOT_PARAM_USER_OR_ERROR_PASSWORD(10003, "用户名或密码为空");;
    /**
     * 响应码
     */
    private final Integer code;

    /**
     * 响应信息
     */
    private final String message;

    /**
     * 有参构造
     * @param code  响应码
     * @param message 响应信息
     */
    ResultEnum(Integer code, String message) {
        this.code = code;
        this.message = message;
    }
    
    public Integer getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}
```

?>注意：枚举类中定义了常见的错误码以及错误的提示信息。这里我们就定义好了统一的结果返回，其中里面的静态方法是用来当程序异常的时候转换成异常返回规定的格式。

`DefinitionException`：自定义异常处理类。

```java
/*
  自定义异常处理类，继承 RuntimeException
 */
public class DefinitionException extends RuntimeException{
    protected Integer errorCode;//错误码
    protected String errorMessage;//错误消息
    
    //有参，无参，get和set方法
}
```

?> 注意：其中**省略**了有参，无参，get和set方法。

我们可以自定义一个**全局异常处理类**，来处理各种异常,包括自己定义的异常和内部异常。这样可以简化不少代码，不用自己对每个异常都使用try，catch的方式来实现。

`GlobalExceptionHandler`：全局异常处理类。

```java
//@ControllerAdvice+@ResponseBody，开启全局的异常捕获，返回JSON
@RestControllerAdvice 
public class GlobalExceptionHandler {

    /**
     * 处理自定义异常
     * @return Result
     * @ExceptionHandler 说明捕获哪些异常，对那些异常进行处理。
     */
    @ExceptionHandler(value = DefinitionException.class)
    public Result<Object> customExceptionHandler(DefinitionException e) {
        return Result.defineError(e);
    }

    /**
     * 处理其他异常
     * @return Result
     */
    @ExceptionHandler(value = Exception.class)
    public Result<Object> exceptionHandler(Exception e) {
        return Result.otherError(ErrorEnum.INTERNAL_SERVER_ERROR);
    }
}
```

?>说明：将对象解析成json，是为了方便前后端的交互。

## 三、代码测试与结果

### 测试类

`ResultController`：测试的controller类

```java
@RestController
public class ResultController {

    //获取学生信息
    @GetMapping("/student")
    public Result<Student> getStudent() {
        Student student = new Student();
        student.setId(1);
      	student.setAge(18);
        student.setName("XuWwei")
        return new Result<>(ResultEnum.SUCCESS, student);
    }

    //自定义异常处理
    @RequestMapping("/getDeException")
    public Result<Object> DeException() {
        throw new DefinitionException(400, "我出错了");
    }

    //其他异常处理
    @RequestMapping("/getException")
    public Result Exception(){
        Result result = new Result();
        int a=1/0;
        return result;
    }
```

`Student`：学生类

```java
public class Student {
    /**
     * 唯一标识id
     */
    private Integer id;
    /**
     * 姓名
     */
    private String name;
    /**
     * 年龄
     */
    private Integer age;
}
```

?> 注意：其中省略了get，set方法。

### 测试结果

启动项目，一个一个测试

#### 1. 正常测试

 ![image-20200510233927348](picture\SpringBoot统一异常处理\image-20200510233927348.png)

可以看到数据是正常返回json，没有异常。

#### 2. 自定义异常

 ![image-20200510234146514](picture\SpringBoot统一异常处理\image-20200510234146514.png)

可以看到这个自定义的异常被捕获到了，并且返回了一个json。

#### 3. 其他异常

 ![image-20200510234438824](picture\SpringBoot统一异常处理\image-20200510234438824.png)

可以看到这个异常被捕获到了，并且返回了一个json。

!>注意：这种方法是不能处理**404异常**的，捕获不到。

## 四、404异常特殊处理

### 1、修改配置文件

​	默认情况下，SpringBoot是不会抛出404异常的，所以**@ControllerAdvice**也不能捕获到404异常。我们可以通过配置文件来让这个注解能捕获到404异常，在application.properties中添加以下配置：

```properties
#当发现404异常时直接抛出异常
spring.mvc.throw-exception-if-no-handler-found=true
#关闭默认的静态资源路径映射，这样404不会跳转到默认的页面
spring.resources.add-mappings=false
```

但是关闭默认的静态资源路径映射会让**静态资源访问出现问题**，也就是不适合前后端一体的情况。

但是我们可以手动配置静态资源路径映射，就能正常访问静态资源了。

```java
@Configuration
public class ResourceConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //可以访问localhost:8080/static/images/image.jpg
        registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");
    }
}
```

### 2、修改error跳转路径

​	关闭默认的静态资源路径映射显然不太合理，可能会导致其他的错误发生，所以也可以通过修改默认错误页面的跳转路径来达到我们的目的。

在`GlobalExceptionHandler`类中添加`NotFoundExceptionHandler`类，这个类继承了`ErrorController`，可以重写error的跳转路径。

```java
//处理404NotFoundException
@Controller
class NotFoundExceptionHandler implements ErrorController {

    //设置错误页面路径
    @Override
    public String getErrorPath() {
        return "/error";
    }

    //当访问error路径时，返回一个封装的异常的Json
    @RequestMapping("/error")
    @ResponseBody
    public Result<Object> error() {
        return Result.otherError(ResultEnum.NOT_FOUND);
    }
}
```

## 五、拓展异常类

​	`GlobalExceptionHandler`的`exceptionHandler`方法将所有的异常统一返回500系统错误，这不符合我们的设想，所以我们可以通过判断异常的类型，来返回不同的值。

将`exceptionHandler`改成以下代码：

```java
/**
  * 处理其他异常
  * @return Result
  */
@ExceptionHandler(value = Exception.class)
public Result<Object> exceptionHandler(Exception e) {
    if (e instanceof NullPointerException){
        //捕获空指针异常
        return Result.otherError(ResultEnum.NOT_PARAM);
    }else if (e instanceof IllegalAccessException){
        //非法访问异常
        return Result.otherError(ResultEnum.NO_PERMISSION);
    } else{
        return Result.otherError(ResultEnum.SERVER_ERROR);
    }
}
```

?>注意：更多异常可以通过`else if`来细分。

## 六、总结

​	springboot的异常处理，需要通过**@ControllerAdvice**注解以及 **@ExceptionHandler**注解，来拦截所有的异常，并通过一个封装返回值返回。但是，这两个注解无法捕获404NotFound异常，因为SpringBoot默认是不会抛出404异常的，所以要通过继承**ErrorController**来修改404异常的跳转路径，达到捕获404异常的目的。