# 文章管理系统

<show-structure depth="3" />

## 1. 项目初始化

### 1.1 创建数据库表

首先创建 Maven 项目，名称为 big-event，接下来我们在 resources/sql/big-event-ddl.sql 中填入以下 SQL 内容并执行:

```SQL
CREATE DATABASE IF NOT EXISTS big_event;

USE big_event;

-- 用户表
CREATE TABLE IF NOT EXISTS `user`
(
    id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT 'ID',
    username VARCHAR(20) NOT NULL UNIQUE COMMENT '用户名',
    password VARCHAR(20) DEFAULT '' COMMENT '密码',
    nickname VARCHAR(20) DEFAULT '' COMMENT '昵称',
    email VARCHAR(128) DEFAULT '' COMMENT '邮箱',
    user_pic VARCHAR(128) DEFAULT '' COMMENT '头像',
    create_time DATETIME NOT NULL COMMENT '创建时间',
    update_time DATETIME NOT NULL COMMENT '更新时间'
) COMMENT '用户表'
;

-- 分类表
CREATE TABLE IF NOT EXISTS `category`
(
    id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT 'ID',
    category_name VARCHAR(32) NOT NULL COMMENT '分类名称',
    category_alias VARCHAR(32) NOT NULL COMMENT '分类别名',
    create_user INT UNSIGNED NOT NULL COMMENT '创建人ID',
    create_time DATETIME NOT NULL COMMENT '创建时间',
    update_time DATETIME NOT NULL COMMENT '更新时间',
    -- 关联外键，实际生产中很少用表关联，因为维护起来比较困难
    CONSTRAINT fk_category_user FOREIGN KEY (`create_user`) REFERENCES `user`(`id`)
);


-- 文章表
CREATE TABLE IF NOT EXISTS `article`
(
    id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT 'ID',
    title VARCHAR(30) NOT NULL COMMENT '文章标题',
    content VARCHAR(1000) NOT NULL COMMENT '文章内容',
    cover_img VARCHAR(128) NOT NULL COMMENT '文章封面',
    state VARCHAR(3) DEFAULT '草稿' COMMENT '文章状态: 已发布/草稿',
    category_id INT UNSIGNED COMMENT '文章分类ID',
    create_user INT UNSIGNED NOT NULL COMMENT '创建人ID',
    create_time DATETIME NOT NULL COMMENT '创建时间',
    update_time DATETIME NOT NULL COMMENT '更新时间',
    CONSTRAINT fk_article_category FOREIGN KEY (`category_id`) REFERENCES `category`(`id`),
    CONSTRAINT fk_article_user FOREIGN KEY (`create_user`) REFERENCES `user`(`id`)
);
```
{collapsible="true" default-state="expanded"}


### 1.2 添加依赖

在 pom.xml 文件中添加依赖项

```xml
<project>
    <!--添加父工程-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.3</version>
    </parent>
    
    <dependencies>
        <!-- Web 依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.3</version>
        </dependency>
        <!-- mybatis 依赖-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>
        <!-- mysql 依赖-->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
        </dependency>
    </dependencies>
</project>
```
{collapsible="true" default-state="expanded"}

### 1.3 添加数据库连接信息

在 application.yml 文件中添加 MySQL 数据库连接信息:

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/big_event
    username: root
    password: root1234
```
{collapsible="true" default-state="expanded"}

### 1.4 完善项目结构

在 com.aifun 目录下创建 controller、mapper、pojo、service、service/impl 文件夹下，并在 pojo 文件夹下创建 User.java、Category.java 以及 Article.java 文件，最后重命名 App.java 为 BigEventApplication.java。

当前项目的文件结构如下:

```Bash
+- com.aifun/
    +- BigEventApplication.java
    +- controller/
    +- mapper/
    +- pojo/
    |   +- User.java
    |   +- Category.java
    |   +- Article.java
    |
    +- service/
    +- utils/
+- resources/
    +- sql/
    |   +- big-event-ddl.sql
    |
    +- application.yml
```
{collapsible="true" default-state="expanded"}

此外，我们需要给 BigEventApplication.java 添加 `@SpringBootApplicatin` 启动注解。

```Java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BigEventApplication
{
    public static void main( String[] args )
    {
        SpringApplication.run(BigEventApplication.class, args);
    }
}
```

## 2. 用户接口

首先我们先来完善下实体类，这里介绍一个新工具 lombok，它的用处是在编译阶段，为实体类自动生成 `setter`、`getter`、`toString` 方法。

首先我们需要先在 pom.xml 文件中引入依赖:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

接下来我们在 User.java、Article.java 和 Category.java 中添加 `@Data` 注解:

<tabs>
<tab title="User.java">
<code-block lang="java">
<![CDATA[
import lombok.Data;
import java.time.LocalDateTime;

@Data
public class User {
    private Integer id;  // 主键 ID
    private String username;  // 用户名
    private String password;  // 密码
    private String nickname; // 昵称
    private String email; // 邮箱
    private String userPic;  // 用户头像地址
    // 注意: `createTime` 和 `updateTime` 对应到 MySQL 数据库表的字段为 `create_time` 和 `update_time`，两者命名方式不一样
    private LocalDateTime createTime;  // 创建时间
    private LocalDateTime updateTime;  // 更新时间
}
]]>
</code-block>
</tab>
<tab title="Article.java">
<code-block lang="java">
<![CDATA[
import lombok.Data;
import java.time.LocalDateTime;

@Data
public class Article {
private Integer id;  // 主键 ID
private String title;  // 文章标题
private String content;  // 文章内容
private String coverImg;  // 封面图像
private String state;  // 发布状态: 已发布/草稿
private Integer categoryId;  // 文章分类 ID
private Integer createUser;  // 创建人 ID
private LocalDateTime createTime;  // 创建时间
private LocalDateTime updateTime;  // 更新时间
}
]]>
</code-block>
</tab>
<tab title="Category.java">
<code-block lang="java">
<![CDATA[
import lombok.Data;
import java.time.LocalDateTime;

@Data
public class Category {
    private Integer id;  // 主键 ID
    private String categoryName;  // 分类名称
    private String categoryAlias;  // 分类别名
    private Integer createUser;  // 创建人 ID
    private LocalDateTime createTime;  // 创建时间
    private LocalDateTime updateTime;  // 更新时间
}
]]>
</code-block>
</tab>
<tab title="Result.java">
<code-block lang="java">
<![CDATA[
package com.aifun.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@NoArgsConstructor
@AllArgsConstructor
@Data
public class Result<T> {
    private Integer code;  // 业务状态码，0 表示成功，1 表示失败
    private String message;  // 提示信息
    private T data;

    // 快速返回操作成功的响应结果(带响应数据)
    public static <E> Result<E> success(E data) {
        return new Result<>(0, "操作成功", data);
    }

    // 快速返回操作成功响应结果
    public static Result success() {
        return new Result(0,  "操作成功", null);
    }

    public static Result error(String message) {
        return new Result(1, message, null);
    }
}
]]>
</code-block>
</tab>
</tabs>

### 2.1 用户注册

#### 2.1.1 接口文档

> <format color="Cyan" style="bold">POST</format> /user/register
>
> Content-Type: application/x-www-form-urlencoded


<format color="DarkOrange" style="bold">请求参数</format>

| 名称       | 说明  | 类型     | 是否必须 | 备注         |
|----------|-----|--------|------|------------|
| username | 用户名 | String | 是    | 5-16 位非空字符 |
| password | 用户名 | String | 是    | 5-16 位非空字符 |


<format color="Green" style="bold">响应参数</format>

| 名称      | 类型     | 是否必须 | 备注               |
|---------|--------|------|------------------|
| code    | number | 必须   | 响应码，0 表示成功，1表示失败 |
| message | string |      | 提示信息             |
| data    | object |      |                  |

<tabs>
<tab title="请求数据">
<code-block lang="bash">
<![CDATA[
username=zhangsan&password=1234
]]>
</code-block>
</tab>
<tab title="响应样例数据">
<code-block lang="json">
<![CDATA[
{
  "code": 0,
  "message": "操作成功",
  "data": "abcasa....zax"
}
]]>
</code-block>
</tab>
</tabs>

#### 2.1.2 注册接口开发

接下来我们参考 API 文档，完成对应的实体类的代码开发。

<tabs>
<tab title="UserController">
<code-block lang="java">
<![CDATA[
package com.aifun.controller;

import com.aifun.pojo.Result;
import com.aifun.pojo.User;
import com.aifun.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {
@Autowired
private UserService userService;
    @PostMapping("/register")
    public Result result(String username, String password) {
        // 查询用户
        User user = userService.findByUserName(username);
        if (user == null) {
            // 没有被占用则注册
            userService.register(username, password);
            return Result.success();
        } else {
            // 占用则返回占用信息
            return Result.error("用户名已被占用");
        }
    }
}
]]>
</code-block>
</tab>
<tab title="UserService">
<code-block lang="java">
<![CDATA[
package com.aifun.service;

import com.aifun.pojo.User;

public interface UserService {
    // 根据用户名查询
    User findByUserName(String username);
    // 根据用户名注册
    void register(String username, String password);
}
]]>
</code-block>
</tab>
<tab title="UserServiceImpl">
<code-block lang="java">
<![CDATA[
package com.aifun.service.impl;

import com.aifun.mapper.UserMapper;
import com.aifun.pojo.User;
import com.aifun.service.UserService;
import com.aifun.utils.Md5Util;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.DigestUtils;

@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;

    @Override
    public User findByUserName(String username) {
        return userMapper.findByUserName(username);
    }

    @Override
    public void register(String username, String password) {
        // 给密码进行加密，这里使用 Spring 封装的 MD5 加密功能
        String md5str = DigestUtils.md5DigestAsHex(password.getBytes());
        userMapper.add(username, md5str);
    }
}

]]>
</code-block>
</tab>
<tab title="UserMapper">
<code-block lang="java">
<![CDATA[
package com.aifun.mapper;

import com.aifun.pojo.User;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

@Mapper
public interface UserMapper {
    // 根据用户名查询用户
    @Select("SELECT * FROM user WHERE username=#{username}")
    User findByUserName(String username);
    // 添加
    @Insert("INSERT INTO user(username, password, create_time, update_time) " +
            "VALUES (#{username}, #{password}, NOW(), NOW())")
    void add(String username, String password);
}
]]>
</code-block>
</tab>
</tabs>

接下来我们使用 Postman 来测试下接口的效果，注意选择 x-www-form-urlencoded，在 Body 中填入 `username->zhangsan` 以及 `password=1234`。

> 需要注意的是，使用 IDEA 自带的 HTTP Client 发送表单数据，似乎不起作用，无法将请求发送到正确的 endpoint 上。
> 
{style="warning"}


#### 2.1.3 参数校验


接口文档中，我们看到对于 `username` 和 `password` 参数是有一定验证要求的。

我们可以使用原生 Java 代码进行验证，代码逻辑如下:

```Java
if (
    username != null && username.length() >= 5 && username.length() <= 16 &&
    password != null && password.length() >= 5 && password.length() <= 16
    ):
```

为了更好地对请求参数进行校验，SpringBoot 我们提供了 Spring Validation 参数验证框架，通过使用预定义的注解完成参数的校验。接下来我们介绍下如何使用 Spring Validation 对注册接口参数进行合法性校验。
1. 引入 Spring Validation 起步依赖

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    ```

2. 在参数面前添加 `@Pattern` 注解
3. 在 Controller 类添加 `@Validated` 注解

    ```Java
    import com.aifun.pojo.Result;
    import com.aifun.pojo.User;
    import com.aifun.service.UserService;
    import jakarta.validation.constraints.Pattern;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.validation.annotation.Validated;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    @RequestMapping("/user")
    @Validated
    public class UserController {
        @Autowired
        private UserService userService;
    
        @PostMapping("/register")
        public Result result(@Pattern(regexp="^\\S{5,16}$") String username, @Pattern(regexp="^\\S{5,16}$") String password) {
            // 查询用户
            User user = userService.findByUserName(username);
            if (user == null) {
                // 没有被占用则注册
                userService.register(username, password);
                return Result.success();
            } else {
                // 占用则返回占用信息
                return Result.error("用户名已被占用");
            }
        }
    }
    ```
   

重启服务，本次请求我们将 username 字段设置为 `z`，会发现 Postman 给我们显示一个 500 请求，观测 IDE 终端我们可以看到 “result.username: must match "^\S{5,16}$", result.password: must match "^\S{5,16}$"” 错误信息。

对于 Web 应用来说，参数验证失败造成的异常，后端返回一个 500 的消息这种方式肯定很不友好，我们还是需要按照接口文档中规定输出指定格式，这个时候我们就可以使用**全局异常处理器**帮助我们去处理从异常。

```Java
package com.aifun.exception;

import com.aifun.pojo.Result;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        e.printStackTrace();
        return Result.error(StringUtils.hasLength(e.getMessage())? e.getMessage(): "操作失败");
    }
}
```

### 2.2 用户登录

#### 2.2.1 接口文档

> <format color="Cyan" style="bold">POST</format> /user/login
> 
> Content-Type: application/x-www-form-urlencoded


<format color="DarkOrange" style="bold">请求参数</format>

| 名称       | 说明  | 类型     | 是否必须 | 备注         |
|----------|-----|--------|------|------------|
| username | 用户名 | String | 是    | 5-16 位非空字符 |
| password | 用户名 | String | 是    | 5-16 位非空字符 |


<format color="Green" style="bold">响应参数</format>

| 名称      | 类型     | 是否必须 | 备注               |
|---------|--------|------|------------------|
| code    | number | 必须   | 响应码，0 表示成功，1表示失败 |
| message | string |      | 提示信息             |
| data    | string |      | 返回 jwt 字符串       |


<tabs>
<tab title="请求数据">
<code-block lang="bash">
<![CDATA[
username=zhangsan&password=1234
]]>
</code-block>
</tab>
<tab title="响应样例数据">
<code-block lang="json">
<![CDATA[
{
  "code": 0,
  "message": "操作成功",
  "data": "abcasa....zax"
}
]]>
</code-block>
</tab>
</tabs>








### 2.3 获取用户详细信息

### 2.4 更新用户基本信息


### 2.5 更新用户头像


### 2.6 更新用户密码




<seealso>
<category ref="ref_docs">
    <a href="https://postman.org.cn">Postman 中文文档</a>
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









