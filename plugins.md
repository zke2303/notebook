# Plugins

## Mybatis

### 1、PageHandler

> `PageHelper` 是一个非常流行且功能强大的 **MyBatis 物理分页插件**。对于任何使用 MyBatis 的 Java 项目来说，它几乎是实现分页功能的首选方案。它的主要目标是让开发者能够以极其简单的方式，对数据库查询进行分页，而无需在 SQL 语句中编写复杂且与具体数据库相关的分页语法（如 `LIMIT`、`ROWNUM` 等）。



#### PageHelper 的核心思想

它的核心思想是 "无侵入式" 设计。你只需要编写正常的、不带分页逻辑的查询语句，`PageHelper` 会通过 MyBatis 的拦截器（Interceptor）机制，自动地在你的原始 SQL 执行之前，将其改写成对应数据库方言的物理分页 SQL。

这意味着，你的 Mapper XML 中的 SQL 语句可以保持非常干净和通用。



#### PageHelper 的主要优点

1. **简单易用**：只需引入依赖并进行简单配置，然后在执行查询语句前调用一行代码即可实现分页。
2. **无侵入性**：你不需要修改任何已有的 SQL 语句。这对于代码的维护和数据库迁移都非常友好。
3. **自动识别数据库方言**：`PageHelper` 能够自动检测你所使用的数据库类型（如 MySQL, Oracle, PostgreSQL, SQL Server 等），并生成相应的分页 SQL。
4. **功能强大**：除了基本的分页功能，它返回的 `PageInfo` 对象包含了非常丰富的分页信息，如总记录数、总页数、当前页码、是否有上一页/下一页等，可以直接用于前端展示。
5. **集成方便**：可以与 Spring、Spring Boot 等主流框架无缝集成。



#### PageHandler的使用

- Maven项目

  1. 导入依赖

     ```xml
     <dependency>
         <groupId>com.github.pagehelper</groupId>
         <artifactId>pagehelper</artifactId>
         <version>5.3.3</version>
     </dependency>
     ```

  2. 在Mybatis的核心配置文件中注册Plug

     ```xml
     <configuration>
         <plugins>
             <!-- 注册分页插件 -->
             <plugin interceptor="com.github.pagehelper.PageInterceptor">
                 <!-- 配置方言 -->
                 <property name="helperDialect" value="mysql"/>
                 <!-- 分页合理化：如果 pageNum<=0，自动查询第一页；如果 pageNum>最大页，自动查询最后一页 -->
                 <property name="reasonable" value="true"/>
                 <!-- 支持通过 Mapper 参数来传递分页参数 -->
                 <property name="supportMethodsArguments" value="true"/>
                 <!-- 是否返回 count 查询 -->
                 <property name="params" value="count=countSql"/>
             </plugin>
         </plugins>
     </configuration>
     ```
  
  3. 在代码中使用
  
     ```java
     public class UserService {
         private UserMapper userMapper; // 通过 SqlSession 获取
     
         public void getUsersByPage(int pageNum, int pageSize) {
             // 设置分页参数
             PageHelper.startPage(pageNum, pageSize);
     
             // 执行查询
             List<User> users = userMapper.selectAll();
     
             // 获取分页信息
             PageInfo<User> pageInfo = new PageInfo<>(users);
     
             System.out.println("总记录数：" + pageInfo.getTotal());
             System.out.println("总页数：" + pageInfo.getPages());
             System.out.println("当前页：" + pageInfo.getPageNum());
             System.out.println("每页条数：" + pageInfo.getPageSize());
             System.out.println("数据：" + users);
         }
     }
     ```

- SpringBoot

  1. 导入依赖

     ```xml
     <dependency>
         <groupId>com.github.pagehelper</groupId>
         <artifactId>pagehelper-spring-boot-starter</artifactId>
         <version>1.4.7</version>
     </dependency>
     
     ```

  2. 配置插件

     ```yml
     pagehelper:
       helperDialect: mysql
       reasonable: true
       supportMethodsArguments: true
       params: count=countSql
     ```

  3. 在代码中使用

     ```java
     // 第1页，每页10条
     PageHelper.startPage(1, 10);
     
     // 调用 mapper 查询
     List<User> users = userMapper.selectAll();
     
     // 获取分页信息
     PageInfo<User> pageInfo = new PageInfo<>(users);
     
     System.out.println("总页数：" + pageInfo.getPages());
     System.out.println("总记录数：" + pageInfo.getTotal());
     
     ```

     