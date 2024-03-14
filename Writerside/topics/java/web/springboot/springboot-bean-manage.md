# Bean 管理

<show-structure depth="3"/>

## 1. Bean 扫描

在 Spring 应用开发中，实现 Bean 扫描的方式有 xml 配置和注解 2 种方式。

<tabs>
<tab title="xml配置">
<code-block lang="xml">
<![CDATA[
<context:component-scan-base-package="com.aifun"/>
]]>
</code-block>
</tab>
<tab title="注解">
<code-block lang="java">
<![CDATA[
@ComponentScan(basePackages="com.aifun")
]]>
</code-block>
</tab>
</tabs>


但是我们在 SpringBoot 应用开发时，这两种方式都没有用，程序却正确扫描到了，这是为何呢？

```Java
@SpringBootApplication
public class SpringbootQuickstartApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootQuickstartApplication.class, args);
    }
}
```

实际上 `@SpringBootApplication` 注解是很多注解的整合，看到 `@ComponentScan` 注解也被整合在里面了。此外，值得一提的是，对于 `@ComponentScan` 注解而言，如果不添加路径的话，默认扫描的就是添加注解下的包以及子包。

```Java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public @interface SpringBootApplication {}
```

如果当前的包位于主程序入口文件的上级目录中，那我们可以手动添加需要扫描包的路径:

```Java
@ComponentScan(basePackages="com.aifun")
@SpringBootApplication
public class SpringbootQuickstartApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringbootQuickstartApplication.class, args);
    }
}
```

## 2. Bean 注册

我们可以在类上添加 `@Component`、`@Controller`、`@Service`、`@Repository` 这 4 个注解，从而将 Bean 对象添加到 IoC 容器中。

| 注解          | 说明               | 位置                              |
|-------------|------------------|---------------------------------|
| @Component  | 声明 bean 的基础注解    | 不属于以下三类时，用此注解                   |
| @Controller | @Component 的衍生注解 | 标注在控制器上                         |
| @Service    | @Component 的衍生注解 | 标注在业务类上                         |
| @Repository | @Component 的衍生注解 | 标注在数据访问类上(因为与 Mybatis 整合，用得比较少) |


如果要注册的 bean 对象来自第三方，是无法使用 `@Component` 及其衍生注解声明的，这个时候就需要使用 `@Bean` 或者 `@Import` 来解决。    


现在我们有一个 common-pojo-1.0-SNAPSHOT.jar 包（提供了 `Country` 和 `Province` 类），我们可以先使用 `mvn install` 将 Jar 安装到本地仓库，然后再在 SpringBoot 应用中引入。

```Bash
mvn install:install-file \
    -Dfile=<Jar包在本地磁盘的路径> \
    -DgroupId=<组织名称> \
    -DartifactId=<项目名称> \
    -Dpackaging=<打包方式> \
    -Dversion=<版本号>
```

完成将包安装到本地仓库后，接下来我们就需要在 SpringBoot 应用 pom.xml 文件中依赖依赖项:

```xml
<dependency>
    <groupId>cn.aifun</groupId>
    <artifactId>common-pojo</artifactId>
    <version>1.0</version>
</dependency>
```

### 2.1 @Bean

当在方法上添加了 `@Bean` 注解时，SpringBoot 会将方法的返回值交给 IoC 容器管理，自动成为 IoC 容器的 bean 对象。

```Java
@SpringBootApplication
public class SpringBootRegisterApplication {
    @Bean
    public Resolver resolver() {
        return new Resolver();
    }
}
```

我们还是基于前面本地安装的 common-pojo-1.0-SNAPSHOT.jar 做一个代码示例，我们注入该 Jar 包中的 `Country` 对象:

```Java
@SpringBootApplication
public class SpringBootRegisterApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootRegisterApplication.class, args);
        Country country = context.getBean(Country.class);
        System.out.println(country);
    }
    
    // 注入 Country 对象
    @Bean
    public Country country() {
        return new Country();
    }
}
```
{collapsible="true" default-state="expanded"}


这种方式并<format color="red">不推荐使用</format>，因为启动类最好就负责启动，不要做过多额外事情。更好的使用方式是，我们单独用一个配置类，来引入外部 Bean。

```Java
@Configuration
public class CommonConfig {
    @Bean
    public Resolver resolver() {
        return new Resolver();
    }
}
```
{collapsible="true" default-state="expanded"}

接下来，我们将这两个 `Country` 和 `Province` 类定义在 `CommonConfig` 类中:


<tabs>
<tab title="CommonConfig.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootregister.config;

@Configuration
public class CommonConfig {
    // 对象默认的名字是方法名
    @Bean
    public Country country() {
        return new Country();
    }

    @Bean
    public Province province() {
        return new Province();
    }
}
]]>
</code-block>
</tab>
<tab title="SpringBootRegisterApplication.java">
<code-block lang="java">
<![CDATA[
@SpringBootApplication
public class SpringBootRegisterApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootRegisterApplication.class, args);
        Country country = context.getBean(Country.class);
        System.out.println(country);
        System.out.println(context.getBean("province"));
    }
}
]]>
</code-block>
</tab>
</tabs>


我们也可以在 `@Bean` 中指定对象的名称，需要注意的是，一旦手动指定名称后，我们就不再可以使用方法名获取对象了，必须使用指定的来进行获取。

<tabs>
<tab title="CommonConfig.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootregister.config;

@Configuration
public class CommonConfig {
    // 对象默认的名字是方法名
    @Bean
    public Country country() {
        return new Country();
    }

    @Bean("pv")
    public Province province() {
        return new Province();
    }
}
]]>
</code-block>
</tab>
<tab title="SpringBootRegisterApplication.java">
<code-block lang="java">
<![CDATA[
@SpringBootApplication
public class SpringBootRegisterApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootRegisterApplication.class, args);
        Country country = context.getBean(Country.class);
        System.out.println(country);
        // 这里使用 province 获取对象是会报错的
        System.out.println(context.getBean("pv"));
    }
}
]]>
</code-block>
</tab>
</tabs>

如果方法的内部需要使用到 IoC 中已经存在的 Bean 对象，那么只需要在方法上声明即可，Spring 会自动进行注入。


<tabs>
<tab title="依赖其他Bean">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootregister.config;

@Configuration
public class CommonConfig {
    // 对象默认的名字是方法名
    @Bean
    public Country country() {
        return new Country();
    }

    @Bean
    public Province province(Country country) {
        System.out.println("Province inner: " + country);
        return new Province();
    }
}
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="java">
<![CDATA[
Province inner: Country{name='null', system='null'}
]]>
</code-block>
</tab>
</tabs>



### 2.2 @Import

在启动类上添加 `@Import` 注解，导入一个其他类 `Xxx.class`，这时 Spring 就会将该类的 Bean 对象导入到 IoC 容器中。通常我们使用 `@Import` 导入配置类以及 `ImportSelector` 接口的实现类。


```Java
@Import(Xxx.class)
@SpringBootApplication
@Import(ComponentConfig.class)
public class SpringbootRegisterApplication {}
```
{collapsible="true" default-state="expanded"}


这里假设 `config` 包并不在启动类目录下，那么我们在启动类中获取不到 `Country` 和 `Province` 了，这个时候我们需要在启动类上添加配置类:

```Java
@SpringBootApplication
@Import(CommonConfig.class)
public class SpringbootRegisterApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootRegisterApplication.class, args);
        Country country = context.getBean(Country.class);
        System.out.println(country);
        // 这里使用 province 获取对象是会报错的
        System.out.println(context.getBean("province"));
    }
}
```
{collapsible="true" default-state="expanded"}

如果配置类比较多的情况下，可以在 `@Import` 注解中引入多个配置类:

````Java
@SpringBootApplication
@Import(CommonConfig.class, MySQLConfig.class, RedisConfig.class)
public class SpringbootRegisterApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootRegisterApplication.class, args);
        Country country = context.getBean(Country.class);
        System.out.println(country);
        // 这里使用 province 获取对象是会报错的
        System.out.println(context.getBean("province"));
    }
}
````
{collapsible="true" default-state="expanded"}

但是如果配置类有 100 个，这样也不太合适，那我们可以

<tabs>
<tab title="CommonImportSelector.java">
<code-block lang="java">
<![CDATA[
// ImportSelector 实现类
public class CommonImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 通常情况下，我们会将需要扫描的库路径写在配置文件中
        return new String[]("com.aifun.config.CommonConfig");
    }
}
]]>
</code-block>
</tab>
<tab title="SpringbootRegisterApplication.java">
<code-block lang="java">
<![CDATA[
@SpringBootApplication
@Import(CommonImportSelector.class)
public class SpringbootRegisterApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootRegisterApplication.class, args);
        Country country = context.getBean(Country.class);
        System.out.println(country);
        // 这里使用 province 获取对象是会报错的
        System.out.println(context.getBean("province"));
    }
}
]]>
</code-block>
</tab>
</tabs>

通常情况下，我们会将需要扫描的包路径写在配置文件中，然后程序读取该配置文件即可。

<tabs>
<tab title="common.imports">
<code-block lang="java">
<![CDATA[
com.aifun.config.CommonConfig
]]>
</code-block>
</tab>
<tab title="CommonImportSelector.java">
<code-block lang="java">
<![CDATA[
public class CommonImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 读取 common.imports 文件内容
        List<String> imports = new ArrayList<>();
        InputStream inputStream = CommonImportSelector.class.getClassLoader().getResourceAsStream("common.imports");
        BufferedReader br = new BufferedReader(new InputStreamReader(inputStream));
        
        String line = null;
        try {
            while ((line = br.readline()) != null) {
                imports.add(line);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }

        return imports.toArray(new String[0]);
    }
}
]]>
</code-block>
</tab>
</tabs>


我们还可以让代码更简洁一点，即将启动类的 `@Import(CommonImportSelector.class)` 去掉，从而添加一个组合注解，就像 `@SpringbootApplication` 注解一样。

<tabs>
<tab title="EnableCommonConfig.java">
<code-block lang="java">
<![CDATA[
import com.aifun.config.CommonConfig;
import org.springframework.context.annotation.Import;

import java.lang.annotation.Target;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;


@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(CommonImportSelector.class)
public @interface EnableCommonConfig {
    
}
]]>
</code-block>
</tab>
<tab title="启动类">
<code-block lang="java">
<![CDATA[
@SpringBootApplication
@EnableCommonConfig
public class SpringbootRegisterApplication {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootRegisterApplication.class, args);
        Country country = context.getBean(Country.class);
        System.out.println(country);
        // 这里使用 province 获取对象是会报错的
        System.out.println(context.getBean("province"));
    }
}
]]>
</code-block>
</tab>
</tabs>

至此，我们的代码就变得非常优秀了！

## 3. 注册条件


前面介绍的比较简单，显然我们使用 `Country` 对象时一般会类属性进行赋值的，比如我们需要将 `Country` 实例的`name` 属性赋值为 China。


<tabs>
<tab title="CommonConfig.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootregister.config;

@Configuration
public class CommonConfig {
    @Bean
    public Country country() {
        Country country  = new Country();
        country.setName("China");
        country.setSystem("Socialism");
        return country;
    }

    @Bean
    public Province province(Country country) {
        System.out.println("Province inner: " + country);
        return new Province();
    }
}
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="java">
<![CDATA[
Province inner: Country{name='China', system='Socialism'}
]]>
</code-block>
</tab>
</tabs>


接下来，我们将这个信息写在配置文件中，而不是写死在 Java 代码中，这就用到前面提到了 `@Vlaue` 注解了。

<tabs>
<tab title="application.yml">
<code-block lang="yaml">
<![CDATA[
country:
    name: China
    system: Socialism
]]>
</code-block>
</tab>
<tab title="tab2">
<code-block lang="python">
<![CDATA[
package com.aifun.springbootregister.config;

@Configuration
public class CommonConfig {
    @Bean
    public Country country(@Value("${country.name}") String name, @Value("${country.system}")  String system) {
        Country country  = new Country();
        country.setName(name);
        country.setSystem(system);
        return country;
    }

    @Bean
    public Province province(Country country) {
        System.out.println("Province inner: " + country);
        return new Province();
    }
}
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="java">
<![CDATA[
Province inner: Country{name='China', system='Socialism'}
]]>
</code-block>
</tab>
</tabs>



