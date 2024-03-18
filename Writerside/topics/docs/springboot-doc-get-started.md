# 开始使用 SpringBoot

<show-structure depth="3"/>

本节更详细地介绍了你应该如何使用 Spring Boot，它涵盖了诸如构建系统、自动配置、以及如何运行你的应用程序等主题。

## 1. 构建系统

强烈建议你选择一个支持依赖管理的构建系统，并且它可以从 Maven Central 拉取依赖。建议选择 Maven 或 Gradle，当然选择其他构建系统（例如 Ant）也不是不行，但它们的支持并不特别好。

### 1.1 依赖管理

Spring Boot 的每个版本都提供了一个它所支持的依赖的列表，在实践中你不需要在构建配置中为这些依赖声明版本，因为 Spring Boot 会帮你管理这些。当你升级 Spring Boot 本身时，这些依赖也会一同升级。

> 你仍然可以指定某个依赖的版本，来覆盖 Spring Boot 默认的版本。
> 
{style="note"}

这份依赖清单包含了所有可以在 Spring Boot 中使用的 Spring 模块，以及一份第三方库的列表。该列表以依赖（spring-boot-dependencies）的形式提供，可在 Maven 和 Gradle 中使用。

> Spring Boot 的每个版本都与 Spring 框架的基本版本有关。 强烈建议不要修改 Spring 的版本。
> 
{style="warning"}

### 1.2 Maven

要学习如何在 Maven 中使用 Spring Boot，请参阅 Spring Boot 的 [Maven 插件文档](https://docs.spring.io/spring-boot/docs/3.2.0-SNAPSHOT/maven-plugin/reference/htmlsingle)。

### 1.3 Gradle

要了解如何在 Gradle 中使用 Spring Boot，请看 Spring Boot 的 [Gradle 插件文档](https://docs.spring.io/spring-boot/docs/3.2.0-SNAPSHOT/gradle-plugin/reference/htmlsingle)。


### 1.4 Ant


### 1.5 Starter

Starter 是一系列开箱即用的依赖，你可以在你的应用程序中导入它们。通过 Starter 可以获得所有你需要的 Spring 和相关技术的一站式服务，免去了需要到处大量复制粘贴依赖的烦恼。例如，如果你想开始使用 Spring 和 JPA 进行数据库访问，那么可以直接在你的项目中导入 `spring-boot-starter-data-jpa` 依赖。

Starter 含了很多你需要的依赖，以使项目快速启动和运行，并拥有一套一致的、受支持的可管理的过渡性依赖。

Starter的命名
: 所有官方的 Starter 都遵循一个类似的命名模式；`spring-boot-starter-*`，其中 `*` 是一个特定类型的应用程序，这种命名结构是为了在你需要寻找 Starter时提供帮助。许多 IDE 中集成的 Maven 可以让你按名称搜索依赖。例如，如果安装了相应的 Eclipse 或 Spring Tools 插件，你可以在 POM 编辑器中按下 ctrl-space ，然后输入 `spring-boot-starter` 来获取完整的列表。
: 正如 “创建你自己的 Starter” 一节所解释的，第三方启动器不应该以 `spring-boot` 开头，因为它是留给 Spring Boot 官方组件的。相反，第三方启动器通常以项目的名称开始。例如，一个名为 thirdpartyproject 的第三方启动器项目通常被命名为 `thirdpartyproject-spring-boot-starter`。

以下是 Spring Boot 在 `org.springframework.boot` 这个 groupId 下提供的 `starter` 组件。

| 名称                                                | 描述                                                                                      |
|:--------------------------------------------------|:----------------------------------------------------------------------------------------|
| `spring-boot-starter`                             | 核心 Starter，包含自动配置、日志以及 YAML                                                             |
| `spring-boot-starter-activemq`                    | 使用 Apache ActiveMQ 处理 JMS 消息                                                            |
| `spring-boot-starter-amqp`                        | 使用 Spring AMQP 和 RabbitMQ                                                               |
| `spring-boot-starter-aop`                         | 使用 Spring AOP 和 AspectJ 进行 aspect-oriented 编程                                           |
| `spring-boot-starter-artemis`                     | 使用 Apache Artemis 处理 JMS 消息                                                             |
| `spring-boot-starter-batch`                       | 使用 Spring Batch                                                                         |
| `spring-boot-starter-cache`                       | 使用 Spring 缓存                                                                            |
| `spring-boot-starter-data-cassandra`              | 使用 Cassandra 分布式数据库和 Spring Data Cassandra                                              |
| `spring-boot-starter-data-cassandra-reactive`     | 使用 Cassandra 分布式数据库和 Spring Data Cassandra Reactive                                     |
| `spring-boot-starter-data-couchbase`              | 使用 Couchbase 文档数据库和 Spring Data Couchbase                                               |
| `spring-boot-starter-data-couchbase-reactive`     | 使用 Couchbase 文档数据库和 Spring Data Couchbase Reactive                                      |
| `spring-boot-starter-data-elasticsearch`          | 使用 Elasticsearch 搜索分析引擎和 Spring Data Elasticsearch                                      |
| `spring-boot-starter-data-jdbc`                   | 使用 Spring Data JDBC                                                                     |
| `spring-boot-starter-data-jpa`                    | 通过 Hibernate 使用 Spring Data JPA                                                         |
| `spring-boot-starter-data-ldap`                   | 使用 Spring Data LDAP                                                                     |
| `spring-boot-starter-data-mongodb`                | 使用 MongoDB 文档数据库和 Spring Data MongoDB                                                   |
| `spring-boot-starter-data-mongodb-reactive`       | 使用 MongoDB 文档数据库和 Spring Data MongoDB Reactive                                          |
| `spring-boot-starter-data-neo4j`                  | 使用 Neo4j 图数据库和 Spring Data Neo4j                                                        |
| `spring-boot-starter-data-r2dbc`                  | 使用 Spring Data R2DBC                                                                    |
| `spring-boot-starter-data-redis`                  | 通过 Spring Data Redis 和 Lettuce 客户端使用 Redis 键值对数据存储                                      |
| `spring-boot-starter-data-redis-reactive`         | 通过 Spring Data Redis reactive 和 Lettuce 客户端使用 Redis 键值对数据存储                             |
| `spring-boot-starter-data-rest`                   | 使用 Spring Data REST 和 Spring MVC 来曝光 Spring Data 仓库                                     |
| `spring-boot-starter-freemarker`                  | 使用 FreeMarker 视图搭建 MVC Web 应用                                                           |
| `spring-boot-starter-graphql`                     | 使用 Spring GraphQL 建立 GraphQL 应用                                                         |
| `spring-boot-starter-groovy-templates`            | 使用 Groovy Templates 视图建立 MVC Web 应用                                                     |
| `spring-boot-starter-hateoas`                     | 通过 Spring MVC 和 Spring HATEOAS 建立超媒体 RESTful Web 应用                                     |
| `spring-boot-starter-integration`                 | 使用 Spring Integration                                                                   |
| `spring-boot-starter-jdbc`                        | 通过 HikariCP 连接池使用 JDBC                                                                  |
| `spring-boot-starter-jersey`                      | 使用 JAX-RS 和 Jersey 建立 RESTful Web 应用，另一个平替是 `spring-boot-starter-web`                   |
| `spring-boot-starter-jooq`                        | 通过 JDBC 使用 jOOQ 连接数据库，其他可选 `spring-boot-starter-data-jpa` 或者 `spring-boot-starter-jdbc` |
| `spring-boot-starter-json`                        | 读写 json                                                                                 |
| `spring-boot-starter-mail`                        | 使用 Java Mail 和 Spring 邮件发送                                                              |
| `spring-boot-starter-mustache`                    | 使用 Mustache 视图建立 Web 应用                                                                 |
| `spring-boot-starter-oauth2-authorization-server` | 使用 Spring Authorization Server 特性                                                       |
| `spring-boot-starter-oauth2-client`               | 使用 Spring Security 的 OAuth2/OpenID Connect 客户端特性                                        |
| `spring-boot-starter-oauth2-resource-server`      | 使用 Spring Security 的 OAuth2 资源服务器特性                                                     |
| `spring-boot-starter-quartz`                      | 使用 Quartz 调度器                                                                           |
| `spring-boot-starter-rsocket`                     | 建立 RSocket 客户端和服务端                                                                      |
| `spring-boot-starter-security`                    | 使用 Spring Security                                                                      |
| `spring-boot-starter-test`                        | 通过 JUnit Jupiter、Hamcrest 、Mockito 库测试 Spring Boot 应用                                   |
| `spring-boot-starter-thymeleaf`                   | 使用 Thymeleaf 视图建立 MVC 应用                                                                |
| `spring-boot-starter-validation`                  | 通过 Hibernate Validator 使用 Java Bean Validation                                          |
| `spring-boot-starter-web`                         | 使用 Spring MVC 建立 Web 应用（包含 Restful），使用 Tomcat 作为嵌入容器                                    |
| `spring-boot-starter-web-services`                | 使用 Spring Web Services                                                                  |
| `spring-boot-starter-webflux`                     | 使用 Spring Reactive Web 建立 WebFlux 应用                                                    |
| `spring-boot-starter-websocket`                   | 使用 Spring MVC WebSocket 建立 WebSocket 应用                                                 |


除了应用 Starter 外，以下启动器可用于添加生产部署功能。

| 命名                             | 描述                                           |
|:-------------------------------|:---------------------------------------------|
| `spring-boot-starter-actuator` | 使用 Spring Boot Actuator，它可以为生产部署应用提供监控和管理的特性 |


最后，Spring Boot 还包括以下 Starter，如果你想排除或更换特定的技术实现，那么可以使用它们。

| 名称                                  | 描述                                                           |
|:------------------------------------|:-------------------------------------------------------------|
| `spring-boot-starter-jetty`         | 使用 Jetty 作为默认的 servlet 容器，其他可选 `spring-boot-starter-tomcat   |
| `spring-boot-starter-log4j2`        | 使用 Log4j2 记录日志，其他可选 `spring-boot-starter-logging             |
| `spring-boot-starter-logging`       | 使用 Logback 记录日志，默认的 logging starter                          |
| `spring-boot-starter-reactor-netty` | 使用 Reactor Netty 作为默认的 reactive HTTP 服务器                     |
| `spring-boot-starter-tomcat`        | 使用 Tomcat 作为默认的 servlet 容器(被 `spring-boot-starter-web` 所使用)  |
| `spring-boot-starter-undertow`      | 使用 Undertow 默认的 servlet 容器，其他可选 `spring-boot-starter-tomcat` |


> 要学习如何更换技术实现，请看[更换WEB服务器](https://springdoc.cn/spring-boot/howto.html#howto.webserver.use-another)和[日志系统](https://springdoc.cn/spring-boot/howto.html#howto.logging.log4j)的操作文档。
> 
{style="note"}

## 2. 代码结构

实际上，Spring Boot 对代码的布局，也没有特别的要求。但是，一个比较好的代码结构能像项目的维护变得更加简单。

### 2.1. default 包

当一个类不包括 package 的声明时，它被认为是在 `default` 包中，通常应避免使 `default` 包。对于使用了 `@ComponentScan`、`@ConfigurationPropertiesScan`、`@EntityScan` 或者 `@SpringBootApplication` 注解的 SpringBoot 应用程序来说，它可能会造成一个问题：项目中的所有 jar 包里面的所有 class 都会被读取（扫描）。

> 我们建议你遵循 Java 推荐的包的命名惯例，使用域名反转作为包名（例如 com.example.project）。
> 
{style="note"}

### 2.2 启动类的位置

我们通常建议你将你启动类放在一个根包中，高于其他的类，`@SpringBootApplication` 注解一般都是添加在启动类上的，它默认会扫描当前类下的所有子包。例如，如果你正在编写一个 JPA 应用程序，你的 `@Entity` 类只有定义在启动类的子包下才能被扫描加载到。这样的好处也显而易见，`@SpringBootApplication` 默认只会扫描加载你项目工程中的组件。

> 如果你不想使用 `@SpringBootApplication`，它所导入的 `@EnableAutoConfiguration` 和 `@ComponentScan` 注解定义了该行为，所以你也可以使用这些来代替。
> 
{style="note"}


<tabs>
<tab title="典型代码结构">
<code-block lang="java">
<![CDATA[
com
 +- example
     +- myapplication
         +- MyApplication.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
]]>
</code-block>
</tab>
<tab title="更常用的代码结构">
<code-block lang="java">
<![CDATA[
com/
 +- example/
     +- myapplication/
         +- MyApplication.java
         |
         +- controller/
         |  +- CustomerController.java
         |  +- OrderController.java
         | 
         +- service/
         |  +- CustomerService.java
         |  +- OrderService.java
         |  +- impl/
         |  |   +- CustomerServiceImpl.java
         |  |   +- OrderServiceImpl.java
         |  
         +- pojo/
         |  +- Customer.java
         |  +- Order.java
         |
         +- mapper/
             +- CustomerMapper.java
             +- OrderMapper.java
             +- OrderRepository.java
]]>
</code-block>
</tab>
</tabs>

MyApplication.java 文件声明了 `main` 方法，以及标识了基本的 `@SpringBootApplication` 注解，如下所示。


<tabs>
<tab title="Java">
<code-block lang="java">
<![CDATA[
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
]]>
</code-block>
</tab>
<tab title="Kotlin">
<code-block lang="kotlin">
<![CDATA[
@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
    runApplication<MyApplication>(*args)
}
]]>
</code-block>
</tab>
</tabs>

## 3. Configuration 类

SpringBoot 倾向于通过 Java 代码来进行配置的定义。虽然也可以使用 XML 来配置 `SpringApplication`，但还是建议你通过 `@Configuration` 类来进行配置。通常，可以把启动类是作为主要的 `@Configuration` 类。

> 网上有很多关于 “通过 XML 配置 Spring” 的示例，如果可以的话，还是尽量使用 Java 代码的方式来实现相同的配置。你可以尝试着搜索 `Enable*` 注解。
> 
{style="note"}

### 3.1 导入额外的 Configuration 类

你不需要把所有的 `@Configuration` 放在一个类中，通常做法是 `@Import` 注解可以用来导入额外的配置类。另外，你可以使用 `@ComponentScan` 来自动扫描加载所有 Spring 组件，包括 `@Configuration` 类。

### 3.2 导入 XML Configuration

如果你确实需要使用基于 XML 的配置，我们建议你仍然从 `@Configuration` 类开始，然后通过 `@ImportResource` 注解来加载 XML 配置文件。


## 4. 自动装配（配置）

SpringBoot 的自动装配机制会试图根据所添加的依赖自动配置 Spring 应用程序，例如，如果你添加了 HSQLDB 依赖，而且你没有手动配置任何 DataSource Bean，那么 SpringBoot 就会自动配置内存数据库。

通过将 `@EnableAutoConfiguration` 或 `@SpringBootApplication` 注解添加到你的 `@Configuration` 类中，从而开启自动配置功能。

> 应该只添加一个 `@SpringBootApplication` 或 `@EnableAutoConfiguration` 注解，建议添加到主要的 `@Configuration` 类上。
> 
{style="note"}

### 4.1 逐步取代自动配置

自动配置是非侵入性的，在任何时候都可以开始定义你自己的配置来取代自动配置的特定部分。例如，如果你添加了你自己的 DataSource bean，默认的嵌入式数据库支持就会“退步”从而让你的自定义配置生效。

如果你想知道在应用中使用了哪些自动配置，你可以在启动命令后添加 `--debug` 参数，这个参数会为核心的 logger 开启 debug 级别的日志，会在控制台输出自动装配项目以及触发自动装配的条件。

### 4.2 禁用指定的自动装配类

如果你想禁用掉项目中某些自动装配类，你可以在 `@SpringBootApplication` 注解的 `exclude` 属性中指定，如下例所示。

<tabs>
<tab title="Java">
<code-block lang="java">
<![CDATA[
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;

@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class MyApplication {

}
]]>
</code-block>
</tab>
<tab title="Kotlin">
<code-block lang="kotlin">
<![CDATA[
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration

@SpringBootApplication(exclude = [DataSourceAutoConfiguration::class])
class MyApplication
]]>
</code-block>
</tab>
</tabs>


如果要禁用的自动装配类不在 classpath 上（没有导入），那么你可以在注解的 `excludeName` 属性中指定类的全路径名称。`exclude` 和 `excludeName` 属性在 `@EnableAutoConfiguration` 中也可以使用。最后，你也可以在配置文件中通过 `spring.autoconfigure.exclude[]` 配置来定义要禁用的自动配置类列表。

> 你可以同时使用**注解 + 配置**的方式来禁用自动装配类。

> 自动配置类一般都是 `public` 的，除了这个类的名称以外（用来禁用它）的任何东西，例如它的方法和属性，包括嵌套的配置类，都不建议你去使用。
> 
{style="note"}


## 5. Spring Bean 和 依赖注入

你可以使用任何标准的 Spring 技术来定义你的 Bean 以及依赖注入关系，推荐使用构造函数注入，并使用 @ComponentScan 注解来扫描Bean。

如果你按照上面的建议构造你的代码（将你的启动类定位在顶级包中），你可以在启动类添加 `@ComponentScan` 注解，也不需要定义它任何参数，你的所有应用组件（`@Component`、`@Service`、`@Repository`、`@Controller` 和其他）都会自动注册为 Spring Bean。


也可以直接使用 `@SpringBootApplication` 注解（该注解已经包含了 `@ComponentScan`）。下面的例子展示了一个 `@Service` Bean，它使用构造器注入的方式注入了 RiskAssessor Bean。

<tabs>
<tab title="Java">
<code-block lang="java">
<![CDATA[
@Service
public class MyAccountService implements AccountService {
    private final RiskAssessor riskAssessor;

    public MyAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }
    // ...
}
]]>
</code-block>
</tab>
<tab title="Kotlin">
<code-block lang="kotlin">
<![CDATA[
import org.springframework.stereotype.Service

@Service
class MyAccountService(private val riskAssessor: RiskAssessor) : AccountService
]]>
</code-block>
</tab>
</tabs>

如果一个 Bean 有多个构造函数，你需要用 `@Autowired` 注解来告诉 Spring 该用哪个构造函数进行注入。

<tabs>
<tab title="Java">
<code-block lang="java">
<![CDATA[
import java.io.PrintStream;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class MyAccountService implements AccountService {
    private final RiskAssessor riskAssessor;
    private final PrintStream out;

    @Autowired
    public MyAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
        this.out = System.out;
    }

    public MyAccountService(RiskAssessor riskAssessor, PrintStream out) {
        this.riskAssessor = riskAssessor;
        this.out = out;
    }
    // ...
}
]]>
</code-block>
</tab>
<tab title="Kotlin">
<code-block lang="kotlin">
<![CDATA[
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.stereotype.Service
import java.io.PrintStream

@Service
class MyAccountService : AccountService {
    private val riskAssessor: RiskAssessor
    private val out: PrintStream

    @Autowired
    constructor(riskAssessor: RiskAssessor) {
        this.riskAssessor = riskAssessor
        out = System.out
    }

    constructor(riskAssessor: RiskAssessor, out: PrintStream) {
        this.riskAssessor = riskAssessor
        this.out = out
    }
    // ...
}
]]>
</code-block>
</tab>
</tabs>

> 上面示例中通过构造器注入的 `riskAssessor` 字段被标识为了 `final`，表示一旦 Bean 创建这个字段就不被改变了，这也是我们推荐的做法。
> 
{style="note"}

## 6. 使用 @SpringBootApplication 注解

许多 SpringBoot 开发者希望他们的应用程序能够使用自动配置、组件扫描，并且能够在他们的 `application class` 上定义额外的配置。一个 `@SpringBootApplication` 注解就可以用来启用这三个功能，如下:
- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@ComponentScan`: 对应用程序所在的包启用 `@Component` 扫描（见最佳实践）
- `@SpringBootConfiguration`: 允许在 `Context` 中注册额外的 Bean 或导入额外的配置类。这是 Spring 标准的 `@Configuration` 的替代方案，有助于在你的集成测试中检测配置

<tabs>
<tab title="Java">
<code-block lang="java">
<![CDATA[
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// 等同于 @SpringBootConfiguration @EnableAutoConfiguration @ComponentScan
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
]]>
</code-block>
</tab>
<tab title="Kotlin">
<code-block lang="kotlin">
<![CDATA[
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

// 等同于 @SpringBootConfiguration @EnableAutoConfiguration @ComponentScan
@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
    runApplication<MyApplication>(*args)
}
]]>
</code-block>
</tab>
</tabs>

`@SpringBootApplication` 也提供了属性别名来定制 `@EnableAutoConfiguration` 和 `@ComponentScan` 中的属性。

这些功能都不是强制必须的，你可以随时只使用其中任意功能的注解。例如，你不需要在你的应用程序中使用组件扫描或配置属性扫描。

<tabs>
<tab title="Java">
<code-block lang="java">
<![CDATA[
import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.Import;

@SpringBootConfiguration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import({ SomeConfiguration.class, AnotherConfiguration.class })
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
]]>
</code-block>
</tab>
<tab title="Kotlin">
<code-block lang="kotlin">
<![CDATA[
import org.springframework.boot.SpringBootConfiguration
import org.springframework.boot.autoconfigure.EnableAutoConfiguration
import org.springframework.boot.docs.using.structuringyourcode.locatingthemainclass.MyApplication
import org.springframework.boot.runApplication
import org.springframework.context.annotation.Import

@SpringBootConfiguration(proxyBeanMethods = false)
@EnableAutoConfiguration
@Import(SomeConfiguration::class, AnotherConfiguration::class)
class MyApplication

fun main(args: Array<String>) {
    runApplication<MyApplication>(*args)
}
]]>
</code-block>
</tab>
</tabs>

在这个例子中，MyApplication 和其他 SpringBoot 应用程序一样，只是不能自动检测到 `@Component` 和 `@ConfigurationProperties` 注解的类，而是明确导入用户定义的 Bean（见 `@Import`）。

## 7. 运行你的应用

把应用打包成可执行 jar，并且使用嵌入式 HTTP 服务器的最大优势，就是可以像其他程序一样运行应用。该样本适用于调试 SpringBoot 应用程序。 你不需要任何特殊的 IDE 插件或扩展。


> 本节只涉及基本的 jar 打包，如果你选择将你的应用程序打包成 war 文件，请参阅你所使用的 Server 和 IDE 的文档。

### 7.1 在 IDE 中运行应用

你可以在 IDE中 像运行普通 Java 程序一样运行你的 SpringBoot 应用。不过，你首先需要导入你的项目。不同的 IDE 导入方式可能不同，大多数 IDE 可以直接导入 Maven 项目。 

如果你不能直接将项目导入IDE，你可以通过构建插件来生成IDE的元数据。包括用于 Eclipse 和 IDEA 的 Maven 插件。Gradle为各种 IDE 都提供了插件。

> 如果你不小心把一个 Web 应用程序运行了两次，你会看到 “Port already in use” 这个异常提示。Spring Tools 用户可以使用 Relaunch 按钮而不是 Run 按钮来实现“重启”（如果程序已经在运行，那么会先关闭它再启动）。
> 
{style="warning"}

### 7.2 运行打包后的应用

如果你使用 SpringBoot 的 Maven 或 Gradle 插件来创建可执行 jar，你可以使用 `java -jar` 来运行你的打包后应用程序，如下例所示:

```Bash
java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

打包后的 jar 程序，也支持通过命令行参数开启远程调试服务，如下:

```Bash
java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

### 7.3 使用 Maven 插件

SpringBoot Maven 插件包括一个 `run` 目标（goal），可用于快速编译和运行你的应用程序。应用程序以 `exploded` 的形式运行，就像在 IDE 中一样，运行 SpringBoot 应用的 Maven 命令如下:

```Bash
mvn spring-boot:run
```

如果项目的构建需要消耗很大的内存，你可以考虑使用 `MAVEN_OPTS` 环境变量来修改最大内存，参考如下:

```Bash
export MAVEN_OPTS=-Xmx1024m
```
### 7.4 使用 Gradle 插件

SpringBoot Gradle 插件还包括一个 `bootRun` 任务，可以用来以 `exploded` 的形式运行你的应用程序。只要你应用了 `org.springframework.boot` 和 `java` 插件，就会添加 `bootRun` 任务，如下例所示。

```Bash
gradle bootRun
```

同上，如果 Gradle 构建项目出现内存不足，可以通过 `JAVA_OPTS` 环境变量来增加 JVM 的内存。

```Bash
export JAVA_OPTS=-Xmx1024m
```

### 7.5 热部署(Hot Swapping)

由于 SpringBoot 应用程序是普通的Java应用程序，JVM的热替换功能可以直接使用。但是，JVM的热替换能替换的字节码有限。要想获得更完整的解决方案，可以使用 [JRebel](https://www.jrebel.com/products/jrebel) 。

`spring-boot-devtools` 模块提供了快速重启应用程序的支持。详情请见[热部署 “How-to”](https://springdoc.cn/spring-boot/howto.html#howto.hotswapping)

## 8. 开发者工具(Developer Tools)

SpringBoot 提供了一套额外的工具，可以让你更加愉快的开发应用。`spring-boot-devtools` 模块可以包含在任何项目中，以在开发期间提供一些有用的特性。要使用 devtools，请添加以下依赖到项目中。


<tabs>
<tab title="Maven">
<code-block lang="xml">
<![CDATA[
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
]]>
</code-block>
</tab>
<tab title="Gradle">
<code-block lang="gradle">
<![CDATA[
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
]]>
</code-block>
</tab>
</tabs>


> Devtools 可能会导致类加载相关的一些问题，特别是在多模块项目中。“诊断类加载问题” 解释了如何诊断和解决这些问题。


如果你的应用程序是通过 `java -jar` 启动的，或者是从一个特殊的 `classloader` 启动的，那么它就被认为是一个 "生产级别的应用程序"，开发者工具会被自动禁用。你可以通过 `spring.devtools.restart.enabled` 配置属性来控制这一行为。要启用 devtools，无论用于启动应用程序的类加载器是什么，请设置启动参数 `-Dspring.devtools.restart.enabled=true`。在生产环境中不能这样做，因为运行 devtools 会有安全风险。要禁用 devtools，请删除该依赖或者设置启动参数 `-Dspring.devtools.restart.enabled=false`。


应该在 Maven 中把这个依赖的 scope 标记为 `optional`，或在 Gradle 中使用 `developmentOnly` 配置（如上所示）。以防止使用你的项目的其他模块，传递地依赖了 devtools。


重新打包的压缩文件默认不包含 devtools，如果你想使用某个[远程 devtool 功能](https://springdoc.cn/spring-boot/using.html#using.devtools.remote-applications)，你需要包含它。使用 Maven 插件时，将 `excludeDevtools` 属性设为 `false`。使用 Gradle 插件时，[配置任务的 classpath，使其包括 `developmentOnly` 配置](https://docs.spring.io/spring-boot/docs/3.2.0-SNAPSHOT/gradle-plugin/reference/htmlsingle/#packaging-executable-configuring-including-development-only-dependencies)。

### 8.1 诊断类加载问题

如[重启 Vs 重载](https://springdoc.cn/spring-boot/using.html#using.devtools.restart.restart-vs-reload) 一节所述，重启功能是通过使用两个 `classloader` 实现的。对于大多数应用程序来说，这种方法效果很好。然而，它有时会导致类加载问题，特别是在多模块项目中。

为了诊断类加载问题是否确实是由 devtools 和它的两个类加载器引起的，[请试着禁用 restart](https://springdoc.cn/spring-boot/using.html#using.devtools.restart.disable)。如果这能解决你的问题，[请定制 restart 类加载器](https://springdoc.cn/spring-boot/using.html#using.devtools.restart.customizing-the-classload) 以包括你的整个项目。

### 8.2. 属性的默认值

在 SpringBoot 支持的一些库中，会使用缓存来提高性能。例如[模板引擎](https://springdoc.cn/spring-boot/web.html#web.servlet.spring-mvc.template-engines)会缓存已编译的模板，以避免重复解析模板文件。另外，SpringMVC 可以在响应静态资源时往响应中添加 HTTP 缓存头。

虽然缓存在生产中是非常有益的，但在开发过程中可能会产生反作用，使你无法看到你在应用程序中刚做的改动。由于这个原因，`spring-boot-devtools`  默认禁用了缓存选项。

缓存的选项通常是通过 `application.properties` 文件中的属性来配置的，例如 Thymeleaf 提供了 `spring.thymeleaf.cache[]` 属性。与其需要手动设置这些属性，`spring-boot-devtools` 模块会在开发场景下合理的设置这些属性。

| 名称                                               | 默认值               |
|:-------------------------------------------------|:------------------|
| `server.error.include-binding-errors`            | `always`          |
| `server.error.include-message`                   | `always`          |
| `server.error.include-stacktrace`                | `always`          |
| `server.servlet.jsp.init-parameters.development` | `true`            |
| `server.servlet.session.persistent`              | `true`            |
| `spring.docker.compose.readiness.wait`           | `only-if-started` |
| `spring.freemarker.cache`                        | `false`           |
| `spring.graphql.graphiql.enabled`                | `true`            |
| `spring.groovy.template.cache`                   | `false`           |
| `spring.h2.console.enabled`                      | `true`            |
| `spring.mustache.servlet.cache`                  | `false`           |
| `spring.mvc.log-resolved-exception`              | `true`            |
| `spring.reactor.netty.shutdown-quiet-period`     | `0s`              |
| `spring.template.provider.cache`                 | `false`           |
| `spring.thymeleaf.cache`                         | `false`           |
| `spring.web.resources.cache.period`              | `0`               |
| `spring.web.resources.chain.cache`               | `false`           |

> 如果你不希望设置属性的默认值，你可以在你的 application.properties 中把 `spring.devtools.add-properties[]` 设置为 `false`。

在开发 SpringMVC 和 Spring WebFlux 应用程序时，你可能需要更多关于 Web 请求的信息，开发者工具建议你为 Web 日志组启用 DEBUG 日志。这将给你提供关于客户端的请求信息，哪个 handler 正在处理它、对应的响应结果以及其他细节。如果你希望记录所有的请求细节（包括潜在的敏感信息），你可以打开 `spring.mvc.log-request-details[]` 或 `spring.codec.log-request-details[]` 配置。

###  8.3. 自动重启

使用 spring-boot-devtools 的应用程序会在 classpath 上的文件发生变化时自动重启。当在 IDE 中工作时，这可能是一个有用的功能，因为它为代码变化提供了一个非常快速的反馈。默认情况下，classpath 上任何指向目录的条目都会被监测到变化。注意，某些资源，如静态资源和视图模板发生变化时，[不需要重启应用程序](https://springdoc.cn/spring-boot/using.html#using.devtools.restart.excluding-resources)。


触发重启
: 由于 DevTools 监控 classpath 资源，触发重启的唯一方法是更新 classpath。无论你使用的是 IDE 还是构建插件，被修改的文件都必须被重新编译以触发重启，导致更新 classpath 的方式取决于你所使用的工具。
: - 在 Eclipse 中，保存一个修改过的文件会导致 classpath 被更新并触发重启。
: - 在 IntelliJ IDEA中，构建项目（ Build +→+ Build Project ）有同样的效果。
: - 如果使用构建插件，运行 Maven 的 mvn compile 或 Gradle的 gradle build 会触发重启。


> 如果你用 Maven 或 Gradle 的构建插件重启，你必须将 `forking` 设置为 `enabled`。如果你禁用 `forking`，devtools 使用的“应用隔离类加载器（isolated application classloader）”将不会被创建，重启功能将无法正常运行。


与 LiveReload 一起使用时，自动重启的效果非常好，详见 [LiveReload 部分](https://springdoc.cn/spring-boot/using.html#using.devtools.livereload)。如果你使用 JRebel，自动重启将被禁用，而采用动态类重载，其他 devtools 特性（如LiveReload和属性覆盖）仍然可以使用。

DevTools 依靠应用程序上下文的关机 hook 来在重启期间关闭它，如果你禁用了关机钩子（`SpringApplication.setRegisterShutdownHook(false)`），它就不能正确工作。

DevTools 需要定制 `ApplicationContext` 所使用的 `ResourceLoader`。如果你的应用程序已经提供了一个，它将被“包装”（装饰者设计模式）起来。不支持直接覆盖 `ApplicationContext` 上的 `getResource` 方法

> 在使用AspectJ 切面时，不支持自动重启。
> 
{style="warning"}


重启 vs 重载
: Spring Boot 通过两个类加载器实现了重启，不变的类（例如，来自第三方jar的类）被加载到一个 *base* classloader。你正在开发的类被加载到 *restart* classloader中。当应用程序被重新启动时，*restart* classloader 被丢弃，并被创建一个新的。这种方法意味着应用程序的重启通常比 “冷启动” 快得多，因为 *base* classloader 已经可用并被填充。
: 如果你发现重启对你的应用程序来说不够快，或者你遇到了类加载相关的问题，你可以考虑重载技术，如 `ZeroTurnaround` 的 [JRebel](https://jrebel.com/software/jrebel)。这些技术的工作原理是在类被加载时对其进行重写，使其更容易被重载。


#### I) 记录条件评估的变化

默认情况下，每次你的应用程序重新启动时，都会记录一份显示条件评估 delta 的报告。该报告显示了你的应用程序的自动配置的变化，因为你做了一些改变，如添加或删除Bean和设置配置属性。

要禁用报告的记录，请设置以下属性:

<tabs>
<tab title="Properties">
<code-block lang="python">
<![CDATA[
spring.devtools.restart.log-condition-evaluation-delta=false
]]>
</code-block>
</tab>
<tab title="Yaml">
<code-block lang="yaml">
<![CDATA[
spring:
  devtools:
    restart:
      log-condition-evaluation-delta: false
]]>
</code-block>
</tab>
</tabs>

#### II) 排除资源

某些资源在被改变时不一定需要触发重启，例如 Thymeleaf 模板可以就地编辑。默认情况下，改变 `/META-INF/maven`、`/META-INF/resources`、`/resources`、`/static`、`/public`、`/templates` 中的资源不会触发重启，但会触发[实时重载](https://springdoc.cn/spring-boot/using.html#using.devtools.livereload)。如果你想自定义这些排除项，可以使用 `spring.devtools.restart.exclude` 属性。例如，要只排除 `/static` 和 `/public` ，你可以设置以下属性。

<tabs>
<tab title="Properties">
<code-block lang="python">
<![CDATA[
spring.devtools.restart.exclude=static/**,public/**

]]>
</code-block>
</tab>
<tab title="Yaml">
<code-block lang="yaml">
<![CDATA[
spring:
  devtools:
    restart:
      exclude: "static/**,public/**"
]]>
</code-block>
</tab>
</tabs>

#### III) 监控额外的路径

当你对不在 classpath 上的文件进行修改时，你可能希望你的应用程序被重新启动或重新加载。为此，使用 `spring.devtools.restart.extra-paths` 属性来配置监控变化的额外路径。你可以使用[前面说过的](https://springdoc.cn/spring-boot/using.html#using.devtools.restart.excluding-resources)的 `spring.devtools.restart.exclude` 属性来控制额外路径下的变化是触发完全重启还是[实时重载](https://springdoc.cn/spring-boot/using.html#using.devtools.livereload)。


#### IV) 禁止重启

如果你不想使用重启功能，你可以通过使用 `spring.devtools.restart.enabled` 属性来禁用它。在大多数情况下，你可以在你的 application.properties 中设置这个属性（这样做仍然会初始化 restart 类加载器，但它不会监控文件变化）。

如果你需要完全禁用重启支持（例如，因为它不能与特定的库一起工作），你需要在调用 `SpringApplication.run(…)` 之前将 `spring.devtools.restart.enabled` 属性设置为 `false` ，如下面的例子中所示。

<tabs>
<tab title="Java">
<code-block lang="java">
<![CDATA[
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        System.setProperty("spring.devtools.restart.enabled", "false");
        SpringApplication.run(MyApplication.class, args);
    }
}
]]>
</code-block>
</tab>
<tab title="Kotlin">
<code-block lang="kotlin">
<![CDATA[
import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication

@SpringBootApplication
object MyApplication {
    @JvmStatic
    fun main(args: Array<String>) {
        System.setProperty("spring.devtools.restart.enabled", "false")
        SpringApplication.run(MyApplication::class.java, *args)
    }
}
]]>
</code-block>
</tab>
</tabs>

#### V) 使用 trigger file

如果你只想在在特定时间触发重启，你可以使用 “trigger file”，这是一个特殊的文件，当你想实际触发重启检查时，那你就对这个文件进行修改。

> 对文件的任何更新都会触发检查，但只有当 Devtools 检测到它有改动发生时，才会实际重新启动。


通过属性 `spring.devtools.restart.trigger-file` 设置trigger file 文件的名称（不包括任何路径）。trigger file 必须在 classpath 路径上。

例如，如果你有一个结构如下的项目，那么你的 “trigger-file” 属性应该设置为如下。

<tabs>
<tab title="文件结构">
<code-block lang="python">
<![CDATA[
src
+- main
   +- resources
      +- .reloadtrigger
]]>
</code-block>
</tab>
<tab title="Properties">
<code-block lang="java">
<![CDATA[
spring.devtools.restart.trigger-file=.reloadtrigger
]]>
</code-block>
</tab>
<tab title="Yaml">
<code-block lang="yaml">
<![CDATA[
spring:
  devtools:
    restart:
      trigger-file: ".reloadtrigger"
]]>
</code-block>
</tab>
</tabs>

现在只有当 src/main/resources/.reloadtrigger 被更新时才会重新启动。

> 你可能想把 `spring.devtools.restart.trigger-file` 设置为[全局设置](https://springdoc.cn/spring-boot/using.html)，这样你的所有项目都会以同样的方式行事。

有些 IDE 有一些功能，使你不需要手动更新你的触发器文件。[Spring Tools for Eclipse](https://spring.io/tools) 和 [IntelliJ IDEA (Ultimate Edition)](https://www.jetbrains.com/idea/)）都有这种支持，对于Spring Tools，你可以使用控制台视图中的 “reload” 按钮（需要把你的 `trigger-file` 命名为 `.reloadtrigger`）。对于 IntelliJ IDEA，你可以按照其文档中的说明进行操作。

#### VI) 自定义重启类加载器(Restart Classloader)

正如前面在["Restart vs Reload"](https://springdoc.cn/spring-boot/using.html)所描述的，重启功能是通过使用两个 classloader 实现的。如果这导致了其他的问题，你可能需要自定义类加载器。

默认情况下，你的 IDE 中任何打开的项目都是用 “restart” 类加载器加载的，而任何常规的 `.jar` 文件都是用 “base” 类加载器加载。如果你使用 `mvn spring-boot:run` 或 `gradle bootRun` 也是一样：包含了 `@SpringBootApplication` 的项目用 “restart” 类加载器加载，其他都用 “base” 类加载器。

你可以通过创建 `META-INF/spring-devtools.properties` 文件来指定 SpringBoot 用不同的类加载器来加载你的项目的不同部分。`spring-devtools.properties` 文件可以包含以 `restart.exclude` 和 `restart.include` 开头的属性，该属性的值是一个基于 classpath 的正则表达式，如以下例子所示。
- `include` 元素是应该被“拉”到 “restart” 类加载器中的项目，
- `exclude` 元素是应该被“推”到 “base” 类加载器中的项目。

<tabs>
<tab title="Properties">
<code-block lang="java">
<![CDATA[
restart.exclude.companycommonlibs=/mycorp-common-[\\w\\d-\\.]+\\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w\\d-\\.]+\\.jar
]]>
</code-block>
</tab>
<tab title="Yaml">
<code-block lang="yaml">
<![CDATA[
restart:
  exclude:
    companycommonlibs: "/mycorp-common-[\\w\\d-\\.]+\\.jar"
  include:
    projectcommon: "/mycorp-myproj-[\\w\\d-\\.]+\\.jar"
]]>
</code-block>
</tab>
</tabs>


> 所有的 key 必须是唯一的，只要一个属性以 `restart.include.` 或 `restart.exclude.` 开头，就会被考虑进去。
> 
{style="note"}

####  VII) 已知的限制

如果你使用标准的 `ObjectInputStream` 来反序列化对象，那么重启功能的效果可能并不好。如果你需要反序列化数据，你可能需要使用到 Spring 的 `ConfigurableObjectInputStream` 与 `Thread.currentThread().getContextClassLoader()`。

不幸的是，一些第三方库在反序列化时没有考虑到上下文的类加载器。如果你发现这样的问题，你需要向原作者请求修复。


### 8.4 LiveReload

`spring-boot-devtools` 模块包括一个内嵌的 LiveReload 服务器，可以用来在资源发生变化时触发浏览器刷新。LiveReload 浏览器扩展可以从 [livereload.com](http://livereload.com/extensions/)免费获得，支持Chrome、Firefox和Safari。

如果你不想在应用程序运行时启动LiveReload服务器，你可以将 `spring.devtools.livereload.enabled` 属性设置为 `false`。

> 你一次只能运行一个LiveReload服务器，在启动你的应用程序之前，确保没有其他 LiveReload 服务器正在运行。如果你从你的 IDE 启动多个应用程序，只有第一个有 LiveReload 支持。


> 为了在文件变化时触发LiveReload，必须启用 [“自动重启”](https://springdoc.cn/spring-boot/using.html#using.devtools.restart)。
>
{style="warning"}

### 8.5 全局设置

你可以通过在 `$HOME/.config/spring-boot` 目录下添加以下任何文件来配置全局的 devtools 设置。

1. `spring-boot-devtools.properties`
2. `spring-boot-devtools.yaml`
3. `spring-boot-devtools.yml`

添加到这些文件中的任何属性都适用于你机器上使用 devtools 的 *所有* SpringBoot 应用程序。例如，如果要将重启配置为总是使用[trigger file](https://springdoc.cn/spring-boot/using.html#using.devtools.restart.triggerfile)，你可以在 `spring-boot-devtools` 文件中添加以下属性。

<tabs>
<tab title="Properties">
<code-block lang="python">
<![CDATA[
spring.devtools.restart.trigger-file=.reloadtrigger
]]>
</code-block>
</tab>
<tab title="Yaml">
<code-block lang="yaml">
<![CDATA[
spring:
  devtools:
    restart:
      trigger-file: ".reloadtrigger"
]]>
</code-block>
</tab>
</tabs>

默认情况下，`$HOME` 是用户的主目录。要自定义这个位置，请设置 `SPRING_DEVTOOLS_HOME` 环境变量或 `spring.devtools.home` 系统属性。

> 如果在 `$HOME/.config/spring-boot` 中找不到 devtools 配置文件，则会在 `$HOME` 目录的根部搜索是否有 `.spring-boot-devtools.properties` 文件。这允许你与那些不支持 `$HOME/.config/spring-boot` 位置的旧版 SpringBoot 的应用程序共享 devtools 全局配置。





 
<seealso>
<category ref="ref_docs">
    <a href="https://springdoc.cn/spring-boot/using.html">使用SpringBoot进行开发</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>