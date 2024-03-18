# 文章管理系统

<show-structure depth="3"/>

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


```xml
<project>
    <!--添加父工程-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.3</version>
    </parent>
    
</project>
```






