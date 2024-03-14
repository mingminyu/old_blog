# 整合 MyBatis

<show-structure depth="3"/>

我们还是先说下，传统方式下如果要将 Mybatis 整合到 Spring 应用中的操作步骤:

1. 在 pom.xml 添加依赖项

    ```xml
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>3.5.6</version>
    </dependency>
    ```
   
2. 定义 `SqlSessionFactoryBean`、`MapperScannerConfigurer`、`DataSource` 这些 Bean 对象

如果使用 SpringBoot 来整合 Mybatis 就变得简单多了，完成下述配置后就算是将 Mybatis 整合成功了，接下来只需要正常编写 Controller、Service 以及 Mapper 即可。

<tabs>
<tab title="依赖项">
<code-block lang="xml">
<![CDATA[
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
]]>
</code-block>
</tab>
<tab title="application.yml">
<code-block lang="yaml">
<![CDATA[
spring:
   datasource:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/mybatis
      username: root
      password: 1234
]]>
</code-block>
</tab>
</tabs>

## 2. 使用 Mybatis 查询数据

接下来，我们在 MySQL 中创建一个数据表，并基于 Mybatis 根据查询条件去查询数据，并将数据在前端上进行展示。

### 2.1 创建示例数据

使用如下 SQL 语句在 MySQL 中创建示例数据:


```SQL
CREATE DATABASE IF NOT EXISTS `examples`;

USE `examples`;

CREATE TABLE IF NOT EXISTS `user`(
    `uid` INT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT "ID",
    `name` VARCHAR(100) COMMENT "姓名",
    `age` TINYINT UNSIGNED COMMENT "年龄",
    `gender` TINYINT UNSIGNED COMMENT "性别: 1-男, 2-女",
    `phone` VARCHAR(11) COMMENT "手机号"
    ) COMMENT "用户表";

INSERT INTO `user`(`name`, `age`, `gender`, `phone`) VALUES ("小明", 18, 1, "13000000000");
INSERT INTO `user`(`name`, `age`, `gender`, `phone`) VALUES ("小莉", 20, 1, "13000000001");
INSERT INTO `user`(`name`, `age`, `gender`, `phone`) VALUES ("小天", 23, 2, "13000000002");
INSERT INTO `user`(`name`, `age`, `gender`, `phone`) VALUES ("张三", 35, 1, "13000000004");
INSERT INTO `user`(`name`, `age`, `gender`, `phone`) VALUES ("小哈", 15, 2, "13000000006");
```

### 2.2 完善应用

按照上述的操作步骤，引入依赖项、在 application.yml 填入配置信息，编写 Controller、Service 以及 Mapper。


<tabs>
<tab title="引入依赖">
<code-block lang="xml">
<![CDATA[
<dependencies>
     <!--MySQL 依赖-->
     <dependency>
         <groupId>com.mysql</groupId>
         <artifactId>mysql-connector</artifactId>
     </dependency>
     <!--Mybatis 依赖-->
     <dependency>
         <groupId>org.mybatis.spring.boot</groupId>
         <artifactId>mybatis-spring-boot-starter</artifactId>
         <version>3.0.0</version>
     </dependency>
</dependencies>
]]>
</code-block>
</tab>
<tab title="application.yml">
<code-block lang="yaml">
<![CDATA[
spring:
    application:
      name: springboot-mybatis

    datasource:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/examples
      username: root
      password: root1234
]]>
</code-block>
</tab>
<tab title="User.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootmybatis.pojo;

public class User {
private Integer uid;
private String name;
private Short age;
private Short gender;
private String phone;

    public User() {}

    public User(Integer uid, String name, Short age, Short gender, String phone) {
        this.uid = uid;
        this.name = name;
        this.age = age;
        this.gender = gender;
        this.phone = phone;
    }

    public Integer getUid() {
        return uid;
    }

    public void setUid(Integer uid) {
        this.uid = uid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Short getAge() {
        return age;
    }

    public void setAge(Short age) {
        this.age = age;
    }

    public Short getGender() {
        return gender;
    }

    public void setGender(Short gender) {
        this.gender = gender;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }
}
]]>
</code-block>
</tab>
<tab title="UserMapper.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootmybatis.mapper;

import com.aifun.springbootmybatis.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

@Mapper
public interface UserMapper {
    @Select("SELECT * FROM user WHERE uid = #{uid}")
    public User findByUid(Integer uid);
}
]]>
</code-block>
</tab>
<tab title="UserService.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootmybatis.service;

import com.aifun.springbootmybatis.pojo.User;

public interface UserService {
    public User findByUid(Integer uid);
}
]]>
</code-block>
</tab>
<tab title="UserServiceImpl.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootmybatis.service;

import com.aifun.springbootmybatis.pojo.User;

public interface UserService {
    public User findByUid(Integer uid);
}
]]>
</code-block>
</tab>
<tab title="UserController.java">
<code-block lang="java">
<![CDATA[
package com.aifun.springbootmybatis.controller;

import com.aifun.springbootmybatis.pojo.User;
import com.aifun.springbootmybatis.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {
    @Autowired
    private UserService userService;
    
    @RequestMapping("/findByUid")
    public User findByUid(Integer uid) {
        return userService.findByUid(uid);
    }
}
]]>
</code-block>
</tab>
</tabs>

> JDBC 连接格式为 jdbc:mysql://localhost:3306/examples，**不要写成** jdbc://mysql://localhost:3306/examples
> 
{style="warning"}


正常启动服务后，我们就可以访问 http://localhost:8080/findByUid?uid=1 看到对应的信息了。

## 3. FAQ

- 如果数据库连接信息不对时，有可能出现错误信息 “java.sql.SQLException: Access denied for user 'root'@'localhost' (using password: YES)”，需要去检查下账号密码是否正确。
- 如果数据库连接配置没有在 application.yml 中提供的话，那么会报错无法创建 `UserMapper` Bean。
