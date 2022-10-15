# Spring Boot

Spring Boot是Spring开源组织下的子项目，是Spring组件一站式解决方案，主要是简化了使用Spring的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手。



## 概述

### SpringBoot特点

1. 容易上手，提升开发效率，为 Spring 开发提供一个更快、更广泛的入门体验。
2. 开箱即用，远离繁琐的配置。
3. 提供了一系列大型项目通用的非业务性功能，例如：内嵌服务器、安全管理、运行数据监控、运行状况检查和外部化配置等。
4. 没有代码生成，也不需要XML配置。
5. 避免大量的 Maven 导入和各种版本冲突。



- 为了解决java开发中的，繁多的配置、底下的开发效率，复杂的部署流程，和第三方技术集成难度大的问题，产生了spring boot。
- springboot 使用 “习惯优于配置”的理念让项目快速运行起来，使用springboot很容易创建一个独立运行的jar，内嵌servlet容器
- springboot的核心功能一：独立运行spring项目，springboot可以以jar包的形式独立运行，运行一个springboot项目只需要 java -jar xxx.jar 来运行
- springboot的核心功能二：内嵌servlet容器，可以内嵌tomcat，接天jetty，或者undertow，这样我们就可以不用war包形式部署项目
- springboot的核心功能三，提供starter简化maven配置，spring提供了一系列starter pom 来简化maven的依赖加载， 当使用了 spring-boot-starter-web时，会自动加载所需要的依赖包
- springboot的核心功能三：自动配置spring sprintboot 会根据在类路径的jar包，类，为jar包中的类自动配置bean，这样会极大的减少使用的配置，会根据启动类所在的目录，自动配置bean



### 运行方式

1）打包用命令或者放到容器中运行

2）用 Maven/ Gradle 插件运行

3）直接执行 main 方法运行



## 相关注解

启动类上面的注解是@SpringBootApplication，它也是 Spring Boot 的核心注解，主要组合包含了以下 3 个注解：

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。

@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能： @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

@ComponentScan：Spring组件扫描。



### @SpringBootApplication

`@SpringBootApplication`注解是一个快捷的配置注解，在被它标注的类中，可以定义一个或多个Bean，并自动触发自动配置Bean和自动扫描组件。此注解相当于`@Configuration`、`@EnableAutoConfiguration`和`@ComponentScan`的组合。



### @EnableAutoConfiguration

@EnableAutoConfiguration注解用于通知Spring，根据当前类路径下引入的依赖包，自动配置与这些依赖包相关的配置项。



## 常用注解

### @ConditionalOnBean

### @ConditionalOnMissingBean

这两个注解属于对象条件注解，根据是否存在某个对象作为依据来决定是否要执行某些配置方法。



### @ConditionalOnClass

### @ConditionalOnMissingClass

这两个注解属于类条件注解，它们根据是否存在某个类作为判断依据来决定是否要执行某些配置。



### @ConditionalExpression

此注解可以让我们控制更细粒度的基于表达式的配置条件限制。当表达式满足某个条件或者表达式为真的时候，将会执行被此注解标注的方法。



### @ConditionalOnProperty

@ConditionalOnProperty 注解会根据Spring配置文件中的配置项是否满足配置要求，从而决定是否要执行被其标注的方法。



```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional({OnPropertyCondition.class})
public @interface ConditionalOnProperty {
    //数组，获取对应property名称的值，与name不可同时使用
    String[] value() default {}; 
    //property名称的前缀，可有可无
    String prefix() default "";
 	//数组，property完整名称或部分名称（可与prefix组合使用，组成完整的property名称），与value不可同时使用
    String[] name() default {};
 	//可与name组合使用，比较获取到的属性值与havingValue给定的值是否相同，相同才加载配置
    String havingValue() default "";
 	//缺少该property时是否可以加载。如果为true，没有该property也会正常加载；反之报错
    boolean matchIfMissing() default false;
 	//是否可以松散匹配，至今不知道怎么使用的
    boolean relaxedNames() default true;
}
```



***example***

```java
@Bean
@ConditionalOnProperty(value = "sample.zipkin.enabled", havingValue = "false")
public Reporter<Span> spanReporter() {
    return Reporter.CONSOLE;
}
```



### @ConditionalOnResource

此注解用于检测当某个配置文件存在使，则触发被其标注的方法。



### @ConditionalOnWebApplication

### @ConditionalOnNotWebApplication

这两个注解用于判断当前的应用程序是否是Web应用程序。如果当前应用是Web应用程序，则使用Spring WebApplicationContext,并定义其会话的生命周期。



## 配置

### 自动配置

注解 @EnableAutoConfiguration, @Configuration, @ConditionalOnClass 就是自动配置的核心，

@EnableAutoConfiguration 给容器导入META-INF/spring.factories 里定义的自动配置类。筛选有效的自动配置类。每一个自动配置类结合对应的 xxxProperties.java 读取配置文件进行自动配置功能。



### 核心配置文件

spring boot 核心的两个配置文件：

- bootstrap (. yml 或者 . properties)：boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，配置在应用程序上下文的引导阶段生效。一般来说我们在 Spring Cloud Config 或者 Nacos 中会用到它。且 boostrap 里面的属性不能被覆盖；
- application (. yml 或者 . properties)： 由ApplicatonContext 加载，用于 spring boot 项目的自动化配置。



### 配置加载类型

在 Spring Boot 里面，可以使用以下几种方式来加载配置。

- properties文件；
- YAML文件；
- 系统环境变量；
- 命令行参数；
- XML配置；



可以在 Spring Beans 里面直接使用这些配置文件中加载的值，如：

1、使用 @Value 注解直接注入对应的值，这能获取到 Spring 中 Environment 的值；

2、使用 @ConfigurationProperties 注解把对应的值绑定到一个对象；

3、直接获取注入 Environment 进行获取；

配置属性的方式很多，Spring boot使用了一种独有的 PropertySource 可以很方便的覆盖属性的值。



#### YAML配置

YAML 是一种人类可读的数据序列化语言。它通常用于配置文件。与属性文件相比，如果我们想要在配置文件中添加复杂的属性，YAML 文件就更加结构化，而且更少混淆。可以看出 YAML 具有分层配置数据。

YAML 现在可以算是非常流行的一种配置文件格式了，无论是前端还是后端，都可以见到 YAML 配置。那么 YAML 配置和传统的 properties 配置相比到底有哪些优势呢？

1. 配置有序，在一些特殊的场景下，配置有序很关键
2. 支持数组，数组中的元素可以是基本数据类型也可以是对象
3. 简洁

相比 properties 配置文件，YAML 还有一个缺点，就是不支持 @PropertySource 注解导入自定义的 YAML 配置。



#### XML配置

Spring Boot 推荐使用 Java 配置而非 XML 配置，但是 Spring Boot 中也可以使用 XML 配置，通过 @ImportResource 注解可以引入一个 XML 配置。

```java
@ImportResource({"classpath:some-context.xml","classpath:another-context.xml"})
```



### 配置加载顺序

1、开发者工具 `Devtools` 全局配置参数；

2、单元测试上的 [`@TestPropertySource](mailto:`@TestPropertySource)` 注解指定的参数；

3、单元测试上的 [`@SpringBootTest](mailto:`@SpringBootTest)` 注解指定的参数；

4、命令行指定的参数，如 `java -jar springboot.jar --name="Java技术栈"`；

5、命令行中的 `SPRING_APPLICATION_JSONJSON` 指定参数, 如 `java -Dspring.application.json='{"name":"Java技术栈"}' -jar springboot.jar`

6、`ServletConfig` 初始化参数；

7、`ServletContext` 初始化参数；

8、JNDI参数（如 `java:comp/env/spring.application.json`）；

9、Java系统参数（来源：`System.getProperties()`）；

10、操作系统环境变量参数；

11、`RandomValuePropertySource` 随机数，仅匹配：`ramdom.*`；

12、JAR包外面的配置文件参数（`application-{profile}.properties（YAML）`）

13、JAR包里面的配置文件参数（`application-{profile}.properties（YAML）`）

14、JAR包外面的配置文件参数（`application.properties（YAML）`）

15、JAR包里面的配置文件参数（`application.properties（YAML）`）

16、[`@Configuration](mailto:`@Configuration)`配置文件上 [`@PropertySource](mailto:`@PropertySource)` 注解加载的参数；

17、默认参数（通过 `SpringApplication.setDefaultProperties` 指定）；



数字小的优先级越高，即数字小的会覆盖数字大的参数值，我们来实践下，验证以上配置参数的加载顺序。



## 实践场景

### 跨域问题

跨域可以在前端通过 JSONP 来解决，但是 JSONP 只可以发送 GET 请求，无法发送其他类型的请求，在 RESTful 风格的应用中，就显得非常鸡肋，因此我们推荐在后端通过 （CORS，Cross-origin resource sharing） 来解决跨域问题。这种解决方案并非 Spring Boot 特有的，在传统的 SSM 框架中，就可以通过 CORS 来解决跨域问题，只不过之前我们是在 XML 文件中配置 CORS ，现在可以通过实现WebMvcConfigurer接口然后重写addCorsMappings方法解决跨域问题。



#### 配置类方式

##### SpringMVC方式

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowCredentials(true)
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                .maxAge(3600);
    }

}
```



##### 过滤器方式

一个http请求，先走filter，到达servlet后才进行拦截器的处理，如果我们把cors放在filter里，就可以优先于权限拦截器执行。

```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource urlBasedCorsConfigurationSource = new UrlBasedCorsConfigurationSource();
        urlBasedCorsConfigurationSource.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsFilter(urlBasedCorsConfigurationSource);
    }

}
```



### 全局异常拦截

Spring 提供了一种使用 ControllerAdvice 处理异常的非常有用的方法。 我们通过实现一个 ControlerAdvice 类，来处理控制器类抛出的所有异常。



## 参考资料

- [SpringBoot中 Jackson 日期的时区和日期格式问题](https://blog.csdn.net/jianxia801/article/details/89741073)



# Spring Boot Starter

启动器是一套方便的依赖没描述符，它可以放在自己的程序中。你可以一站式的获取你所需要的 Spring 和相关技术，而不需要依赖描述符的通过示例代码搜索和复制黏贴的负载。

Spring Boot 也提供了其它的启动器项目包括，包括用于开发特定类型应用程序的典型依赖项。

- spring-boot-starter-web-services - SOAP Web Services

- spring-boot-starter-web - Web 和 RESTful 应用程序

- spring-boot-starter-test - 单元测试和集成测试

- spring-boot-starter-jdbc - 传统的 JDBC

- spring-boot-starter-hateoas - 为服务添加 HATEOAS 功能

- spring-boot-starter-security - 使用 SpringSecurity 进行身份验证和授权

- spring-boot-starter-data-jpa - 带有 Hibeernate 的 Spring Data JPA

- spring-boot-starter-data-rest - 使用 Spring Data REST 公布简单的 REST 服务



## spring-boot-starter-security

为了实现 Spring Boot 的安全性，我们使用 spring-boot-starter-security 依赖项，并且必须添加安全配置。它只需要很少的代码。配置类将必须扩展WebSecurityConfigurerAdapter 并覆盖其方法。



# 组件整合

## Mybatis

## Mybatis-plus
