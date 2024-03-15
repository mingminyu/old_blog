# 自动装配

<show-structure depth="3"/>

实际开发场景中，我们会将常用的组件封装到 Stater 中以便大家共同使用，要想实现自定义的 Starter，就需要我们对自动装配的原理有一个深刻的了解，才有可能写出比较好的 Starter 组件出来。


自动装配遵循约定大于配置的原则，在 SpringBoot 程序启动后，起步依赖中的一些 Bean 对象会自动注入到 IoC 容器中。

## 1. 源码分析 {collapsible="true" default-state="expanded"}

之前提到过，SpringBoot 程序如果引入 `spring-boot-stater-web` 依赖了，那么在程序启动后会自动向 IoC 容器中注入 `dispatcherServlet` 对象。

<tabs>
<tab title="pom.xml">
<code-block lang="xml">
<![CDATA[
<dependencies>
    <!-- 注意这里不是 spring-boot-starter-web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
</dependencies>
]]>
</code-block>
</tab>
<tab title="SpringbootAutoconfigApplication.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootautoconfig;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class SpringbootAutoConfigApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringbootAutoConfigApplication.class, args);
        System.out.println(context.getBean("dispatcherServlet"));
    }

}

]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="java">
No bean named 'dispatcherServlet' available
</code-block>
</tab>
</tabs>

可以看到，IoC 容器中并没有 `DispatcherServlet` 对象，那么接下来我们引入 `spring-boot-starter-web` 依赖后，看看程序运行结果有啥变化:


<tabs>
<tab title="pom.xml">
<code-block lang="xml">
<![CDATA[
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!-- 引入 spring-boot-starter-web 依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
]]>
</code-block>
</tab>
<tab title="SpringbootAutoconfigApplication.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootautoconfig;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
public class SpringbootAutoConfigApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringbootAutoConfigApplication.class, args);
        System.out.println(context.getBean("dispatcherServlet"));
    }

}

]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="java">
org.springframework.web.servlet.DispatcherServlet@6137cf6e
</code-block>
</tab>
</tabs>

接下来，我们分析下源码，看 SpringBoot 实现此功能。

<tabs>
<tab title="SpringBootApplication.java">
<code-block lang="java">
<![CDATA[
// 其他注解我们先不关注，单看这两个注解
@SpringBootConfiguration
@EnableAutoConfiguration
public @interface SpringBootApplication {}
]]>
</code-block>
</tab>
<tab title="SpringBootConfiguration.java">
<code-block lang="java">
<![CDATA[
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@Indexed
public @interface SpringBootConfiguration {}
]]>
</code-block>
</tab>

<tab title="EnableAutoConfiguration.java">
<code-block lang="java">
<![CDATA[
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {}
]]>
</code-block>
</tab>
<tab title="AutoConfigurationImportSelector.java">
<code-block lang="java">
<![CDATA[
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
     public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
            AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }

    protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.getConfigurationClassFilter().filter(configurations);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationEntry(configurations, exclusions);
        }
    }
    
    protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
        List<String> configurations = ImportCandidates.load(AutoConfiguration.class, this.getBeanClassLoader()).getCandidates();
        Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. If you are using a custom packaging, make sure that file is correct.");
        return configurations;
    }
}
]]>
</code-block>
</tab>
<tab title="DeferredImportSelector.java">
<code-block lang="java">
<![CDATA[
public interface DeferredImportSelector extends ImportSelector {}
]]>
</code-block>
</tab>
</tabs>


> 这里需要注意下，对于 `AutoConfigurationImportSelector` 中的 `selectImports` 方法被不会被 SpringBoot 自动调用，实际调用的是更为复杂的流程，但原理上是差不多的。
> 
{style="warning"}


我们去 `org.springframework.boot.autoconfigure-3.2.3.jar` 下 `META-INF/spring` 目录下找到 `org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件，搜索下 `DispatcherServlet` 关键词，可以看到确实已经将 `DispatcherServletAutoConfiguration` 包含在文件目录中:

<tabs>
<tab title="AutoConfiguration.imports">
<code-block lang="java">
<![CDATA[
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration
]]>
</code-block>
</tab>
<tab title="DispatcherServletAutoConfiguration.java">
<code-block lang="java">
<![CDATA[
@AutoConfigureOrder(Integer.MIN_VALUE)
@AutoConfiguration(
    after = {ServletWebServerFactoryAutoConfiguration.class}
)
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@ConditionalOnClass({DispatcherServlet.class})
public class DispatcherServletAutoConfiguration {
    @Configuration(
        proxyBeanMethods = false
    )
    @Conditional({DefaultDispatcherServletCondition.class})
    @ConditionalOnClass({ServletRegistration.class})
    @EnableConfigurationProperties({WebMvcProperties.class})
    protected static class DispatcherServletConfiguration {
        @Bean(
            name = {"dispatcherServlet"}
        )
        public DispatcherServlet dispatcherServlet(WebMvcProperties webMvcProperties) {
            DispatcherServlet dispatcherServlet = new DispatcherServlet();
            dispatcherServlet.setDispatchOptionsRequest(webMvcProperties.isDispatchOptionsRequest());
            dispatcherServlet.setDispatchTraceRequest(webMvcProperties.isDispatchTraceRequest());
            this.configureThrowExceptionIfNoHandlerFound(webMvcProperties, dispatcherServlet);
            dispatcherServlet.setPublishEvents(webMvcProperties.isPublishRequestHandledEvents());
            dispatcherServlet.setEnableLoggingRequestDetails(webMvcProperties.isLogRequestDetails());
            return dispatcherServlet;
        }
    }
}
]]>
</code-block>
</tab>
<tab title="AutoConfiguration.java">
<code-block lang="java">
<![CDATA[
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration(
    proxyBeanMethods = false
)
@AutoConfigureBefore
@AutoConfigureAfter
public @interface AutoConfiguration {
}
]]>
</code-block>
</tab>
</tabs>

那么针对之前 Common-pojo-1.0-SNAPSHOT.jar 引入，如何实现自动装配呢，步骤如下:
1. 定义 CommonConfig，注入 `Country` 和 `Province` Bean 对象
2. 定义 CommonAutoConfig，通过 `@Import` 注入 CommonConfig
3. 找到 org.springframework.boot.autoconfigure.AutoConfiguration.imports，添加 `com.aifun.config.CommonAutoConfig`
4. 在项目 pom.xml 文件中引入 common-pojo 依赖，便完成了起步自动装配

---------

总结 SpringBoot 的自动装配原理
: 1. 在主启动类上添加 `SpringBootApplication` 注解，该注解组合了 `EnableAutoConfiguration` 注解
: 2. `EnableAutoConfiguration` 注解又组合了 Import 注解，导入了 `AutoConfigurationImportSelector` 类
: 3. 实现 `selectImports` 方法，这个方法经过层层调用，最终会读取 META-INF 目录下的后缀名为 `.imports` 的文件，读取的是 spring.factories 文件
: 4.读取到全类名之后，SpringBoot 会解析注册条件，也就是 `@Conditional` 及其衍生注解，把满足注册条件的 Bean 对象自动注入到 IoC 容器中

## 2. 自定义 Starter

在实际开发场景中，经常会定义一些公共组件，提供给各个项目团队使用。在 SpringBoot 项目中，一般会将这些公共组件封装为 SpringBoot 的 Stater。

一般来说，起步依赖会有两个工程组成，这里以 Mybatis 为例:
- **自动配置**: Maven: org.mybatis.spring.boot:mybatis-spring-boot-autoconfigure:3.0.0
- **依赖管理**: Maven: org.mybatis.spring.boot:mybatis-spring-boot-starter:3.0.0

接下来，我们的目标是自定义一个 Mybatis 的 starter，这里说明一些操作步骤:
- 创建 imybatis-spring-boot-autoconfigure 模块，提供自动配置功能，并自定义配置文件 META-INF/spring/xxx.imports
- 创建 imybatis-spring-boot-starter 模块，在 starter 中引入自动配置模块



