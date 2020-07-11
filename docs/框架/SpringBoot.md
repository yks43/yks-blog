# 介绍
## Spring
我们知道Spring是重量级企业开发框架 Enterprise JavaBean（EJB） 的替代品，Spring为企业级Java开发提供了一种相对简单的方法，通过 **依赖注入** 和 **面向切面编程** ，用简单的 Java对象（Plain Old Java Object，POJO） 实现了EJB的功能

**虽然Spring的组件代码是轻量级的，但它的配置却是重量级的（需要大量XML配置）**。Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。

尽管如此，我们依旧没能逃脱配置的魔爪。开启某些Spring特性时，比如事务管理和Spring MVC，还是需要用XML或Java进行显式配置。启用第三方库时也需要显式配置，比如基于Thymeleaf的Web视图。配置Servlet和过滤器（比如Spring的DispatcherServlet）同样需要在web.xml或Servlet初始化代码里进行显式配置。组件扫描减少了配置量，Java配置让它看上去简洁不少，但Spring还是需要不少配置。

光配置这些XML文件都够我们头疼的了，占用了我们大部分时间和精力。除此之外，相关库的依赖非常让人头疼，不同库之间的版本冲突也非常常见。

**不过，好消息是：Spring Boot让这一切成为了过去。**

## SpringBoot
> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can “just run”...Most Spring Boot applications need very little Spring configuration.(Spring Boot可以轻松创建独立的生产级基于Spring的应用程序,只要通过 “just run”（可能是run ‘Application’或java -jar 或 tomcat 或 maven插件run 或 shell脚本）便可以运行项目。大部分Spring Boot项目只需要少量的配置即可)

**简而言之，从本质上来说，Spring Boot就是Spring，它做了那些没有它你自己也会去做的Spring Bean配置。**

## 为什么需要SpringBoot
> Spring Framework旨在简化J2EE企业应用程序开发。Spring Boot Framework旨在简化Spring开发。
### 引申概念
#### EJB（Enterprise JavaBean）
> 企业级JavaBean（Enterprise JavaBean, EJB）是一个用来构筑企业级应用的服务器端可被管理组件。 Java企业版API（Java Enterprise Edition）中提供了对EJB的规范。 EJB是一个封装有某个应用程序之业务逻辑服务器端组件。

#### J2EE
> J2EE的全称是Java 2 Platform Enterprise Edition，它是由SUN公司领导、各厂家共同制定并得到广泛认可的工业标准，或者说，它是在SUN公司领导下，多家公司参与共同制定的企业级分布式应用程序开发规范。目前，J2EE是市场上主流的企业级分布式应用平台的解决方案

### SpringBoot的主要优点
1.  开发基于 Spring 的应用程序很容易。
2.  Spring Boot 项目所需的开发或工程时间明显减少，通常会提高整体生产力。
3.  Spring Boot不需要编写大量样板代码、XML配置和注释。
4.  Spring引导应用程序可以很容易地与Spring生态系统集成，如Spring JDBC、Spring ORM、Spring Data、Spring Security等。
5.  Spring Boot遵循“固执己见的默认配置”，以减少开发工作（默认配置可以修改）。
6.  Spring Boot 应用程序提供嵌入式HTTP服务器，如Tomcat和Jetty，可以轻松地开发和测试web应用程序。（这点很赞！普通运行Java程序的方式就能运行基于Spring Boot web 项目，省事很多）
7.  Spring Boot提供命令行接口(CLI)工具，用于开发和测试Spring Boot应用程序，如Java或Groovy。
8.  Spring Boot提供了多种插件，可以使用内置工具(如Maven和Gradle)开发和测试Spring Boot应用程序。 

#  开发环境要求!

## JDK

推荐 JDK 1.8 版本

## 构建工具

Maven

Gradle

## 开发工具

推荐IDEA

## Web服务器

### Spring Boot支持以下嵌入式servlet容器:

*   Tomcat

*   Jetty

*   Undertow

#  快速入门

## 新建 Spring Boot 项目常用的两种方式

1.  可以通过 https://start.spring.io/ 这个网站来生成一个 Spring Boot 的项目。

注意勾选上 Spring Web 这个模块，这是我们所必需的一个依赖。当所有选项都勾选完毕之后，点击下方的按钮 Generate 下载这个 Spring Boot 的项目。下载完成并解压之后，我们直接使用 IDEA 打开即可。

2.  直接通过 IDEA 来生成一个 Spring Boot 的项目：`File->New->Project->Spring Initializr`。

## 项目结构

``` 
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- controller
      |  +- CustomerController.java
      |  
      | +- config
      |  +- swagerConfig.java
      |
```

1.  `Application.java`是项目的启动类
2.  domain目录主要用于实体（Entity）与数据访问层（Repository）
3.  service 层主要是业务类代码
4.  controller 负责页面访问控制
5.  config 目录主要放一些配置类

## 注解分析

大概可以把 `@SpringBootApplication `看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan `注解的集合。根据 SpringBoot官网，这三个注解的作用分别是：

-   `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
-   `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的bean，注解默认会扫描该类所在的包下所有的类。
-   `@Configuration`：允许在上下文中注册额外的bean或导入其他配置类。

所以说 `@SpringBootApplication `就是几个重要的注解的组合，为什么要有它？当然是为了省事，避免了我们每次开发 Spring Boot 项目都要写一些必备的注解。

## HelloWorld开发步骤

1.  创建SpringBoot工程
2.  引入依赖
3.  编写配置文件(application.yml)
4.  编写启动类
5.  编写controller层代码

>   `@RestController`是Spring 4 之后新加的注解，如果在Spring4之前开发 RESTful Web服务的话，你需要使用`@Controller` 并结合`@ResponseBody`注解，也就是说`@Controller` +`@ResponseBody`= `@RestController`。

6.  运行启动类

# 基础

## 读取配置文件

### 通过 `@value` 读取比较简单的配置信息

使用 `@Value("${property}")` 读取比较简单的配置信息

``` java
@Value("${yks.name}")
String name;
```



### 通过`@ConfigurationProperties`读取并与 bean 绑定

在配置类上加上@ConfigurationProperties(prefix = "yks"),@Component

### 通过`@ConfigurationProperties`读取并校验

**`ProfileProperties` 类没有加 `@Component` 注解。我们在我们要使用`ProfileProperties` 的地方使用`@EnableConfigurationProperties`注册我们的配置bean**

### `@PropertySource`读取指定 properties 文件

```java
@Component
@PropertySource("classpath:website.properties")
```

使用：

```
@Autowired
private WebSite webSite;

System.out.println(webSite.getUrl());
```

## RESTful Web服务

RESTful Web 服务与传统的 MVC 开发一个关键区别是返回给客户端的内容的创建方式：**传统的 MVC 模式开发会直接返回给客户端一个视图，但是 RESTful Web 服务一般会将返回的数据以 JSON 的形式返回，这也就是现在所推崇的前后端分离开发。**

### 常用注解

1.  `@RestController` **将返回的对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中。**绝大部分情况下都是直接以 JSON 形式返回给客户端，很少的情况下才会以 XML 形式返回。转换成 XML 形式还需要额为的工作，上面代码中演示的直接就是将对象数据直接以 JSON 形式写入 HTTP 响应(Response)中。
2.  `@RequestMapping` :上面的示例中没有指定 GET 与 PUT、POST 等，因为**`@RequestMapping`默认映射所有HTTP Action**，你可以使用`@RequestMapping(method=ActionType)`来缩小这个映射。
3.  `@PostMapping`实际上就等价于 `@RequestMapping(method = RequestMethod.POST)`，同样的 `@DeleteMapping` ,`@GetMapping`也都一样，常用的 HTTP Action 都有一个这种形式的注解所对应。
4.  `@PathVariable` :取url地址中的参数。`@RequestParam `url的查询参数值。
5.  `@RequestBody`:可以**将 HttpRequest body 中的 JSON 类型数据反序列化为合适的 Java 类型。**
6.  `ResponseEntity`: **表示整个HTTP Response：状态码，标头和正文内容**。我们可以使用它来自定义HTTP Response 的内容。

## 异常处理

### 1. 使用 **@ControllerAdvice和**@ExceptionHandler处理全局异常

### 2. `@ExceptionHandler` 处理 Controller 级别的异常

### 3. ResponseStatusException

## 过滤器

### Filter

>   Filter 过滤器主要是用来过滤用户请求的，它允许我们对用户请求进行前置处理和后置处理，比如实现 URL 级别的权限控制、过滤非法请求等等。Filter 过滤器是面向切面编程——AOP 的具体实现（AOP切面编程只是一种编程思想而已）。Filter 是依赖于 Servlet 容器，`Filter`接口就在 Servlet 包下面，属于 Servlet 规范的一部分。所以，很多时候我们也称其为“增强版 Servlet”。

如果我们需要自定义 Filter 的话非常简单，只需要实现 `javax.Servlet.Filter` 接口，然后重写里面的 3 个方法即可！

### Filter如何实现过滤

`Filter`接口中有一个叫做 `doFilter` 的方法，这个方法实现了对用户请求的过滤。具体流程大体是这样的：

1.  用户发送请求到 web 服务器,请求会先到过滤器；
2.  过滤器会对请求进行一些处理比如过滤请求的参数、修改返回给客户端的 response 的内容、判断是否让用户访问该接口等等。
3.  用户请求响应完毕。
4.  进行一些自己想要的其他操作。

### 自定义Filter

#### 自己手动注册配置实现	

1.  **自定义的 Filter 需要实现`javax.Servlet.Filter`接口，并重写接口中定义的3个方法。**

```java
@Component
public class MyFilter implements Filter {
    private static final Logger logger = LoggerFactory.getLogger(MyFilter.class);

    @Override
    public void init(FilterConfig filterConfig) {
        logger.info("初始化过滤器：", filterConfig.getFilterName());
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //对请求进行预处理
        logger.info("过滤器开始对请求进行预处理：");
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        String requestUri = request.getRequestURI();
        System.out.println("请求的接口为：" + requestUri);
        long startTime = System.currentTimeMillis();
        //通过 doFilter 方法实现过滤功能
        filterChain.doFilter(servletRequest, servletResponse);
        // 上面的 doFilter 方法执行结束后用户的请求已经返回
        long endTime = System.currentTimeMillis();
        System.out.println("该用户的请求已经处理完毕，请求花费的时间为：" + (endTime - startTime));
    }

    @Override
    public void destroy() {
        logger.info("销毁过滤器");
    }
}
```

2. **在配置中注册自定义的过滤器。**

```java
@Configuration
public class MyFilterConfig {
    @Autowired
    MyFilter myFilter;
    @Bean
    public FilterRegistrationBean<MyFilter> thirdFilter() {
        FilterRegistrationBean<MyFilter> filterRegistrationBean = new FilterRegistrationBean<>();

        filterRegistrationBean.setFilter(myFilter);

        filterRegistrationBean.setUrlPatterns(new ArrayList<>(Arrays.asList("/api/*")));

        return filterRegistrationBean;
    }
}
```

#### 通过提供好的一些注解实现

**在自己的过滤器的类上加上`@WebFilter` 然后在这个注解中通过它提供好的一些参数进行配置。**

```java
@WebFilter(filterName = "MyFilterWithAnnotation", urlPatterns = "/api/*")
public class MyFilterWithAnnotation implements Filter {

   ......
}
```

另外，为了能让 Spring 找到它，你需要在启动类上加上 `@ServletComponentScan` 注解。

#### 定义多个拦截器，并决定它们的执行顺序

**在配置中注册自定义的过滤器，通过`FilterRegistrationBean` 的`setOrder` 方法可以决定 Filter 的执行顺序。**

## 拦截器

### Interceptor

>   **拦截器(Interceptor)同** Filter 过滤器一样，它俩都是面向切面编程——AOP 的具体实现（AOP切面编程只是一种编程思想而已）。
>
>   你可以使用 Interceptor 来执行某些任务，例如在 **Controller** 处理请求之前编写日志，添加或更新配置......
>
>   在 **Spring中**，当请求发送到 **Controller** 时，在被**Controller**处理之前，它必须经过 **Interceptors**（0或更多）。
>
>   **Spring Interceptor**是一个非常类似于**Servlet Filter** 的概念 。

### 过滤器和拦截器的区别

-   过滤器（Filter）：当你有一堆东西的时候，你只希望选择符合你要求的某一些东西。定义这些要求的工具，就是过滤器。
-   拦截器（Interceptor）：在一个流程正在进行的时候，你希望干预它的进展，甚至终止它进行，这是拦截器做的事情。

### 自定义 Interceptor

自定义 **Interceptor** 的话必须实现 **org.springframework.web.servlet.HandlerInterceptor**接口或继承 **org.springframework.web.servlet.handler.HandlerInterceptorAdapter**类，并且需要重写下面下面3个方法：

```java
public boolean preHandle(HttpServletRequest request,
                         HttpServletResponse response,
                         Object handler)
 
 
public void postHandle(HttpServletRequest request,
                       HttpServletResponse response,
                       Object handler,
                       ModelAndView modelAndView)
 
 
public void afterCompletion(HttpServletRequest request,
                            HttpServletResponse response,
                            Object handler,
                            Exception ex)
```

注意： ***preHandle***方法返回 **true**或 **false**。如果返回 **true**，则意味着请求将继续到达 **Controller** 被处理。

**`OldLoginInterceptor`**是一个拦截器，如果用户输入已经被废弃的链接 **“ / admin / oldLogin”**，它将重定向到新的 **“ / admin / login”。**

#### 配置拦截器

```java
import github.javaguide.springbootfilter.interceptor.AdminInterceptor;
import github.javaguide.springbootfilter.interceptor.LogInterceptor;
import github.javaguide.springbootfilter.interceptor.OldLoginInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // LogInterceptor apply to all URLs.
        registry.addInterceptor(new LogInterceptor());

        // Old Login url, no longer use.
        // Use OldURLInterceptor to redirect to a new URL.
        registry.addInterceptor(new OldLoginInterceptor())//
                .addPathPatterns("/admin/oldLogin");

        // This interceptor apply to URL like /admin/*
        // Exclude /admin/oldLogin
        registry.addInterceptor(new AdminInterceptor())//
                .addPathPatterns("/admin/*")//
                .excludePathPatterns("/admin/oldLogin");
    }

}
```

## 参数校验

>   **数据的校验的重要性就不用说了，即使在前端对数据进行校验的情况下，我们还是要对传入后端的数据再进行一遍校验，避免用户绕过浏览器直接通过一些 HTTP 工具直接向后端请求一些违法数据。**

### 环境搭建

相关依赖

```xml
   <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.0.9.Final</version>
   </dependency>
   <dependency>
             <groupId>javax.el</groupId>
             <artifactId>javax.el-api</artifactId>
             <version>3.0.0</version>
     </dependency>
     <dependency>
            <groupId>org.glassfish.web</groupId>
            <artifactId>javax.el</artifactId>
            <version>2.2.6</version>
     </dependency>
```

实体类字段上加注解

**类上加上 `Validated` 注解了，这个参数可以告诉 Spring 去校验方法参数。**

需要验证的参数前加@Vaild，如果验证失败会抛出MethodArgumentNotValidException，默认情况下，Spring会将此异常转换为HTTP Status 400（错误请求）。

## 定时任务

>   很多时候我们都需要为系统建立一个定时任务来帮我们做一些事情，SpringBoot 已经帮我们实现好了一个，我们只需要直接使用即可，当然你也可以不用 SpringBoot 自带的定时任务，整合 Quartz 很多时候也是一个不错的选择。

### 创建一个 scheduled task

启动类上加上@`EnableScheduling`注解，方法上使用 `@Scheduled` 注解就能很方便地创建一个定时任务

Cron 表达式: 主要用于定时作业(定时任务)系统定义执行时间或执行频率的表达式，非常厉害，你可以通过 Cron 表达式进行设置定时任务每天或者每个月什么时候执行等等操作。

推荐一个在线Cron表达式生成器：http://cron.qqe2.com/

### 自定义线程池执行 scheduled task

### @EnableAsync 和 @Async 使定时任务并行执行

# 面试题

## @RestController vs @Controller

*   Controller 返回一个页面

单独使用 `@Controller` 不加 `@ResponseBody`的话一般使用在要返回一个视图的情况，这种情况属于比较传统的Spring MVC 的应用，对应于前后端不分离的情况。

*   @RestController 返回JSON 或 XML 形式数据

但`@RestController`只返回对象，对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中，这种情况属于 RESTful Web服务，这也是目前日常开发所接触的最常用的情况（前后端分离）。

*   @Controller +@ResponseBody 返回JSON 或 XML 形式数据

在Spring4之前开发 RESTful Web服务的话，需要使用`@Controller` 并结合`@ResponseBody`注解，也就是说`@Controller` +`@ResponseBody`= `@RestController`（Spring 4 之后新加的注解）。

