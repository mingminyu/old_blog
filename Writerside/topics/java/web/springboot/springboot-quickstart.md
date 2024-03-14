# 快速入门

<show-structure depth="3"/>

大名鼎鼎的 Spring 除了提供核心功能的 Spring Framework 外，还提供了以下一些子项目： 
- Spring Data: 用户数据获取
- Spring AMQP: 消息传递
- Spring Security: 认证授权
- Spring Cloud: 服务治理

今天我们开始学习的 Spring Boot，则是 Spring 官方所提供的另外一个子项目，用于快速构建 Spring 应用程序。

## 1. 传统方式构建 Spring 应用

使用传统方式去构建 Spring 程序，通常会存在以下问题:
- **导入依赖繁琐**: 很多依赖需要我们逐个手动导入，容易出现依赖的 JAR 出现冲突的问题，尤其是项目变得越来越大的时候，这种操作就会变得极其复杂

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>5.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>5.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-expression</artifactId>
        <version>5.3.1</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>4.0.1</version>
    </dependency>
</dependencies>
```
{collapsible="true" default-state="expanded" collapsed-title="大量依赖项需要维护"}


- **项目配置繁琐**: 非常多的配置文件（web.xml、applicationContext.xml、springMvc.xml 等）以及配置项

需要在 applicationContext.xml 中声明大量的 Bean 对象，并且这些 Bean 对象需要先声明再使用。例如我们想要声明以一个数据库连接对接对象，我们通常采用如下形式进行定义。

```xml
<beans>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" ref="mybatis-config.xml"/>
    </bean>
    <bean id="mapperScannerConfigure" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.aifun.mapper"/>
    </bean>
</beans>
```
{collapsible="true" default-state="expanded"}

面对如此繁琐的操作，简直让人头大。SpringBoot 的诞生极大地解决这些问题，让创建 Spring 应用变得更加简单。

## 2. Spring 特性

SpringBoot 主要具备起步依赖和自动配置的特性，当然它也有其他特性。

### 2.1 起步依赖

本质上就是一个 Maven 坐标，整合了完成一个功能需要的所有坐标。前面我们介绍过，在使用 Spring 进行 Web 开发时需要引入大量的依赖，而 SpringBoot 则是将这些依赖整合到了 `spring-boot-starter-web` 这个单一依赖中，这样我们在 pom.xml 文件中只需要引入这一个依赖即可。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 2.2 自动配置

遵循约定大于配置的原则，在 Boot 程序启动后，一些 Bean 对象会自动注入到 IOC 容器中，不再需要手动声明，从而简化开发。

前面讲过，如果我们需要将 Mybatis 整合到 Spring 中，需要先引入依赖，再注入对应的 `sqlSessionFactory` 和 `MapperScannerConfigurer` Bean。

<tabs>
<tab title="引入 Mybatis 依赖">
<code-block lang="xml">
<![CDATA[
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>3.5.6</version>
</dependency>
]]>
</code-block>
</tab>
<tab title="注入Bean">
<code-block lang="xml">
<![CDATA[
<beans>
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" ref="mybatis-config.xml"/>
    </bean>
    <bean id="mapperScannerConfigure" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.aifun.mapper"/>
    </bean>
</beans>
]]>
</code-block>
</tab>
</tabs>

有了 SpringBoot 后，我们只需要只引入 `mybatis-spring-boot-starter` 这个依赖即可，简直不要太爽。

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```
{collapsible="true" default-state="expanded"}


### 2.3 其他特性

当然，SpringBoot 也提供了一些其他特性:
- 内嵌的 Tomcat、Jetty（无需部署 war 文件）
- 外部化配置
- 不需要 XML 配置(properties/yml)

## 3. 第一个 SpringBoot 应用

在开始第一个 SpringBoot 应用之前，我们还是说下传统的 Spring 开发流程:

1. 引入依赖 spring-core、spring-context、spring-webmvc 等
2. 在 web-app 中定义 `dispatcherServlet` 配置
3. 编写 Spring 核心配置文件，例如包扫描、注解驱动等
4. 编写 Controller 来处理 /hello 请求
5. 将项目进行打包，部署到 Tomcat 上

对于 SpringBoot 而言，开发步骤就非常简单：
1. 创建 Maven 工程: 
   - 在 IDEA 新建项目配置中选择 **Spring Initializr**，类型选择 **Maven**，打包直接默认 **Jar** 包即可
   - 选择 **SpringBoot** 的版本，以及 **Web** 选项下 **Spring Web**
   
2. 引入依赖，这里只需要引入一个起步依赖就可以了

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    ```
   {collapsible="true" default-state="expanded"}

3. 无需更改配置，在 springbootquickstart 包创建一个 controller 子包，并创建 HelloController.java 文件，直接编写 Controller

    ```Java
    @RestController
    public class HelloController {
        @RequestMapping("/hello")
        public String hello() {
            System.out.println("Hello, Spring Boot!");
            return "Hello, Spring Boot!"
        }
    }
    ```
   {collapsible="true" default-state="expanded"}
   
4. 提供启动类

    ```Java
    @SpringBootApplication
    public class SpringBootStartApplication {
        public static void main(String[] args) {
            SpringApplication.run(SpringBootStartApplication.class, args);
        }
    }
    ```
   {collapsible="true" default-state="expanded"}

5. 打成 JAR 包后可直接运行


<tabs>
<tab title="pom.xml">
<code-block lang="xml">
<![CDATA[
<!--  SpringBoot 工程的父工程，用于管理起步依赖的版本  -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.3</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
]]>
</code-block>
</tab>
<tab title="SpringbootQuickstartApplication.java">
<code-block lang="java">
<![CDATA[
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// 启动类
@SpringBootApplication
public class SpringbootQuickstartApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootQuickstartApplication.class, args);
    }
}
]]>
</code-block>
</tab>
<tab title="HelloController.java">
<code-block lang="java">
<![CDATA[
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}
]]>
</code-block>
</tab>
</tabs>

成功启动服务后，我们就可以通过访问 http://localhost:8080/hello 看到浏览器返回的信息了。

## 4. 手动创建 SpringBoot 工程

先前我们是通过 IDEA 的 Spring Initializr 插件创建的 SpringBoot 应用，下面我们也演示下不借助这个插件，我们应该如何创建 SpringBoot 项目:
1. 创建 Maven 工程: 在 IDEA 新建项目配置中选择 **Maven Archetype**，在 Archetype 选项中选择 `org.apache.maven.archetypes:maven-archetype-quickstart`

2. 引入依赖: 在 pom.xml 文件中引入新增 SpringBoot 启动项依赖

    ```xml
    <project>
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>3.2.3</version>
        </parent>
        <dependencies>
            <dependency>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
        </dependencies>
    </project>
    ```
   {collapsible="true" default-state="expanded"}

3. 提供启动类: 在 MyApplication.java 文件中添加注解

    ```Java
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @SpringBootApplication
    public class SpringBootCreateManualApplication {
        public static void main(String[] args) {
            System.out.println("Hello, Spring Boot!");
            SpringApplication.run(SpringBootCreateManualApplication.class, args);
        }
    }
    ```
   {collapsible="true" default-state="expanded"}

4. 新建 resources 文件夹（与 java 文件夹同级），新建 application.properties 文件。

5. 新建 controller 包，并新增 HelloController.java 文件

    ```Java
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    public class HelloController {
        @RequestMapping("/hello")
        public String hello() {
            return "Hello, Spring Boot!";
        }
    }
    ```
    {collapsible="true" default-state="expanded"}

接下来我们就可以像之前那样启动我们的 SpringBoot 应用了。

## 5. 打 Jar 包独立运行

如果程序正常运行没有问题，那么可以借助 `maven package` 命令来将 SpringBoot 应用打成 Jar 包（最便捷的方式借助 IDEA 的 Maven 插件进行打包），然后在终端中运行:

```Bash
java -Dfile.encoding=UTF-8 \
    -Dsun.stdout.encoding=UTF-8 \
    -Dsun.stderr.encoding=UTF-8 \
    -jar ./target/springboot-quickstart-0.0.1-SNAPSHOT.jar
```
{collapsible="true" default-state="expanded"}

我们就可以看到打包后的应用正常启动了，访问的方式还是和之前的一样。

