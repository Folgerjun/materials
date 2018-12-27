---
title: SpringBoot2 整合 Sharding JDBC 实现 Mysql 读写分离
date: 2018-12-27 15:20:34
categories: [开发,总结]
tags: [SpingBoot2,Java,ShardingJDBC]
---

> 想直接要源码的，[点这里](https://github.com/Folgerjun/various-simple-examples/tree/master/spring-boot-sharding-jdbc)。

---

## 简介
Sharding-JDBC 定位为轻量级 Java 框架，在 Java 的 JDBC 层提供的额外服务。 它使用客户端直连数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。

- 适用于任何基于 Java 的 ORM 框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template 或直接使用 JDBC
- 基于任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP 等
- 支持任意实现JDBC规范的数据库。目前支持 MySQL，Oracle，SQLServer 和 PostgreSQL

## 前言
本例只是简单实现了 Sharding-JDBC 中的读写分离功能，请注意。

**所用到的技术栈及版本：**

- SpringBoot 2.0.4
    + Spring Data JPA
    + HikariDataSource
    + Gson 2.8.5
    + lombok 1.16.22
    + mysql-connector-java 5.1.46
- sharding-jdbc-core 2.0.3

## 主要部分

### 配置文件：application.yml
```
# JPA
spring:
  jpa:
    show-sql: true
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    hibernate:
      ddl-auto: create

# Server
server:
  port: 8888

# Sharding JDBC
sharding:
  jdbc:
    data-sources:
      ds_master:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/master?characterEncoding=utf8&useSSL=false
        username: root
        password: root
      ds_slave:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/slave?characterEncoding=utf8&useSSL=false
        username: root
        password: root
    master-slave-rule:
      name: ds_ms
      master-data-source-name: ds_master
      slave-data-source-names: ds_slave
      load-balance-algorithm-type: round-robin

```
这里用的是 springboot2.0 默认的数据库连接池 HikariDataSource

- `load-balance-algorithm-type`：
查询时的负载均衡算法，目前有2种算法，round_robin（轮询）和random（随机）
- `master-data-source-name`： 主数据源名称
- `slave-data-source-names`： 从数据源名称 多个用逗号隔开

### 存放数据源数据：ShardingMasterSlaveConfig.java
```
package com.example.shardingjdbc.config;

import java.util.HashMap;
import java.util.Map;

import org.springframework.boot.context.properties.ConfigurationProperties;

import com.zaxxer.hikari.HikariDataSource;

import io.shardingjdbc.core.api.config.MasterSlaveRuleConfiguration;
import lombok.Data;

/**
 * 存放数据源
 * 
 * @author ffj
 *
 */
@Data
@ConfigurationProperties(prefix = "sharding.jdbc")
public class ShardingMasterSlaveConfig {

    private Map<String, HikariDataSource> dataSources = new HashMap<>();

    private MasterSlaveRuleConfiguration masterSlaveRule;
}
```
用了 Lombok 显得简便了些

### 配置数据源：ShardingDataSourceConfig.java
```
package com.example.shardingjdbc.config;

import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;

import javax.sql.DataSource;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.zaxxer.hikari.HikariDataSource;

import io.shardingjdbc.core.api.MasterSlaveDataSourceFactory;

/**
 * 配置数据源详细信息
 * 
 * @author ffj
 *
 */
@Configuration
@EnableConfigurationProperties(ShardingMasterSlaveConfig.class)
@ConditionalOnProperty({ "sharding.jdbc.data-sources.ds_master.jdbc-url",
        "sharding.jdbc.master-slave-rule.master-data-source-name" })
public class ShardingDataSourceConfig {

    private static final Logger log = LoggerFactory.getLogger(ShardingDataSourceConfig.class);

    @Autowired(required = false)
    private ShardingMasterSlaveConfig shardingMasterSlaveConfig;

    /**
     * 配置数据源
     * 
     * @return
     * @throws SQLException
     */
    @Bean("dataSource")
    public DataSource masterSlaveDataSource() throws SQLException {
        shardingMasterSlaveConfig.getDataSources().forEach((k, v) -> configDataSource(v));
        Map<String, DataSource> dataSourceMap = new HashMap<>();
        dataSourceMap.putAll(shardingMasterSlaveConfig.getDataSources());
        DataSource dataSource = MasterSlaveDataSourceFactory.createDataSource(dataSourceMap,
                shardingMasterSlaveConfig.getMasterSlaveRule(), new HashMap<>());
        log.info("masterSlaveDataSource config complete！！");
        return dataSource;
    }

    /**
     * 可添加数据源一些配置信息
     * 
     * @param dataSource
     */
    private void configDataSource(HikariDataSource dataSource) {
        dataSource.setMaximumPoolSize(20);
        dataSource.setMinimumIdle(5);
    }
}

```
主要的配置内容就是这些了，接下来我们编写几个方法来测试。

## 测试
- 先创建一个实体类

大众测试实体类，我选 `UserEntity`:
```
package com.example.shardingjdbc.entity;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * 测试用户类
 * 
 * @author ffj
 *
 */
@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity(name = "user")
public class UserEntity implements Serializable {

    /**
     * 
     */
    private static final long serialVersionUID = -6171110531081112401L;
    @Id
    private int id;
    @Column(length = 32)
    private String name;
    @Column(length = 16)
    private int age;

}
```
同样，Lombok 不可少。由于之前 `application.yml` 中 `ddl-auto` 设置的是 `create`，所以每次重启程序都会重新生成空表。

- 我选择 `JPA` 的原因就是它作为简单测试最适合不过了
```
package com.example.shardingjdbc.repository;

import org.springframework.data.jpa.repository.JpaRepository;

import com.example.shardingjdbc.entity.UserEntity;

public interface UserRepository extends JpaRepository<UserEntity, Integer> {

}

```
只要继承 `JpaRepository` 就可以了，我们只需要使用它的基本方法即可。

- 写个 `Controller` 类
```
package com.example.shardingjdbc.controller;

import java.util.List;

import javax.annotation.Resource;

import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

import com.example.shardingjdbc.entity.UserEntity;
import com.example.shardingjdbc.service.UserService;
import com.google.gson.Gson;

/**
 * 用户测试类
 * 
 * @author ffj
 *
 */
@RestController
public class UserController {

    @Resource
    private UserService userService;

    @PostMapping("/save")
    public String saveUser() {
        UserEntity user = new UserEntity(1, "张三", 22);
        userService.saveUser(user);
        return "success";
    }

    @PostMapping("/getUser")
    public String getUsers() {
        List<UserEntity> users = userService.getUsers();
        return new Gson().toJson(users);
    }

}
```
Service 就不贴了，就是简单调用。

方便查看测试结果，这里用 `Gson` 来转化为 `Json` 输出。

- 启动程序

从上到下看启动日志：
```
2018-12-27 15:16:12.907  INFO 2940 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2018-12-27 15:16:13.132  INFO 2940 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2018-12-27 15:16:13.141  INFO 2940 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Starting...
2018-12-27 15:16:13.147  INFO 2940 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Start completed.
2018-12-27 15:16:13.148  INFO 2940 --- [           main] c.e.s.config.ShardingDataSourceConfig    : masterSlaveDataSource config complete
```
可以看出有两个数据源，没毛病。
```
Hibernate: drop table if exists user
Hibernate: create table user (id integer not null, age integer, name varchar(32), primary key (id)) engine=InnoDB
```
程序启动 `user` 表重建，没毛病。
```
2018-12-27 15:16:15.198  INFO 2940 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-12-27 15:16:15.550  INFO 2940 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2018-12-27 15:16:15.587  INFO 2940 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8888 (http) with context path ''
2018-12-27 15:16:15.591  INFO 2940 --- [           main] c.e.s.ShardingJdbcApplication            : Started ShardingJdbcApplication in 5.36 seconds (JVM running for 5.725)
2018-12-27 15:16:15.592  INFO 2940 --- [           main] c.e.s.ShardingJdbcApplication            : ----------启动成功----------
```
端口为配置文件中指定的 `8888`，启动成功日志打印，也没毛病，成功。

**注意：启动程序前别忘了先自行创建数据库！**

现在我们在 `slave` 库中执行以下提供的 `sql` 文件，或者自行创建对应表（表结构必须一致，可以先建库然后从主库中复制已生成的表），并在其中添加数据。
```
/*
Navicat MySQL Data Transfer

Source Server         : localhost
Source Server Version : 50720
Source Host           : localhost:3306
Source Database       : slave

Target Server Type    : MYSQL
Target Server Version : 50720
File Encoding         : 65001

Date: 2018-12-27 16:23:00
*/

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL,
  `age` int(11) DEFAULT NULL,
  `name` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('2', '23', '李四');
```
好了，从库中表和数据都有了，进入正题。

- 测试主库插入数据
我用的是 `Postman` ：选择 `POST` 方式，url：localhost:8888/save，点击 `Send`。
```
Hibernate: insert into user (age, name, id) values (?, ?, ?) // 打印出这条日志
```
返回 `success`，成功执行。查看主数据库中 `user` 表数据，确实插入，成功！

- 测试从库查询数据
还是 `Postman` ：选择 `POST` 方式，url：localhost:8888/getUser，点击 `Send`。
```
Hibernate: select userentity0_.id as id1_0_, userentity0_.age as age2_0_, userentity0_.name as name3_0_ from user userentity0_ // 打印出这条日志
```
返回： `[{"id":2,"name":"李四","age":23}]`，数据正确，成功！

以上就是 SpringBoot2.0 + ShardingJDBC 实现数据库读写分离的全部内容了。

## 参考博文
- [springboot2.0中用sharding-jdbc实现读写分离，集成Druid](https://blog.csdn.net/vipbupafeng/article/details/80256958)
- [Spring Boot中整合Sharding-JDBC读写分离示例](https://juejin.im/post/5b88979bf265da435944018c)