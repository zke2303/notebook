# Mybatis

## 1、什么是 MyBatis？

MyBatis 是一款优秀半自动的ORM**持久层框架**，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

> ######  **持久层**：指的是就是数据访问层(dao)，是用来操作数据库的。

#### 什么是ORM？

- Object-Relational Mapping，简称 **ORM**，中文译为**对象关系映射**。它是一种编程技术，用于解决面向对象的编程语言（如 Java、Python）和关系型数据库（如 MySQL、PostgreSQL）之间的**阻抗失配**问题。

  简单来说，ORM 就像一个“翻译官”或“桥梁”，将你的代码中的**对象**自动映射到数据库的**表**上。也就是POJO的属性对应数据库表中的Column。



#### Mybatis的优点：

- 简单
- 方便易学
- 简化JDBC操作
- 支持动态 SQL

- SQL 灵活可控

- MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型



## 2、Mybatis的快速入门

1、创建一个Maven项目

2、导入相关的依赖（核心依赖）

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>9.4.0</version>
</dependency>

<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.19</version>
</dependency>
```

3、配置mybatis的核心配置文件，使用xml来配置。注意mybatis-config.xml文件必须位于maven项目中的**resource**目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true"/>
                <property name="username" value="root"/>
                <property name="password" value="123"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

4、创建相应的包和实体类（Pojo）。

在这里我使用了lombok，lombok是一个偷懒的神器，可以通过注解简化Pojo对象的编写。详细请看[Lombok](https://projectlombok.org/)

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {
    private Long id;
    private String name;
    private Long pwd;
}
```

5、编写Dao（Data access Object）接口。

```java
public interface UserMapper {
    Student getById(@Param("id") Long id);
}
```

6、编写xml映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.dao.UserMapper">
	<select id="getById" resultType="com.kuang.pojo.User">
    	select * from user where id = #{id};
    </select>
</mapper>
```

> 注意： 一般情况下，把Dao接口和xml映射文件取同一个前缀名，在这里Dao接口为`UserMapper.class`，xml文件应该为`UserMapper.xml`。且它们要位于同一个包下。

7、在mybatis的全局配置文件mybatis-config.xml文件中，添加<Mappers>标签，让mybatis去扫描我们编写的mapping映射文件.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true"/>
                <property name="username" value="root"/>
                <property name="password" value="123"/>
            </dataSource>
        </environment>
    </environments>
    
    <mappers>
    	<mapper resource="com/kuang/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

8、测试

```java
public class StudentMapperTest{

    @Test
    public void getStudentByIdTest(){
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        
        try (SqlSession session = sqlSessionFactory.openSession()) {
          User user = session.getMapper(UserMapper.class).getById(1L);
        }
    }
}

```

![image-20250903213517641](/home/zhang/.config/Typora/typora-user-images/image-20250903213517641.png)

> 运行到这里其实是有问题的，因为maven项目中，没有把我们编写的xml映射文件加载到classpath中，因此会出现 `Caused by: java.io.IOException: Could not find resource com/kuang/dao/StudentMapper.xml`异常。想要解决这个异常，就必须写改maven的pop.xml配置文件，让maven项目把xml映射文件也加载到`classpath`目录下。
>
> 只需要在pop.xml文件中，添加以下内容就可以把`src/main/java`和`src/main/resources`目录下的所有以.xml和.properties结尾的文件加载到`classpath`下
>
> ```xml
> <build>
>     <resources>
>         <resource>
>             <directory>src/main/java</directory>
>             <includes>
>                 <include>**/*.properties</include>
>                 <include>**/*.xml</include>
>             </includes>
>         </resource>
> 
>         <resource>
>             <directory>src/main/resources</directory>
>             <includes>
>                 <include>**/*.properties</include>
>                 <include>**/*.xml</include>
>             </includes>
>         </resource>
>     </resources>
> </build>
> ```

添加后结果如下, 成功查询

![image-20250903213603914](/home/zhang/.config/Typora/typora-user-images/image-20250903213603914.png)



## 3、Mybatis的配置

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

- configuration（配置）
  - [properties（属性）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#properties)
  - [settings（设置）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#settings)
  - [typeAliases（类型别名）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#typeAliases)
  - [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#typeHandlers)
  - [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#objectFactory)
  - [plugins（插件）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#plugins)
  - [environments（环境配置）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#environments)
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
  - [databaseIdProvider（数据库厂商标识）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#databaseIdProvider)
  - [mappers（映射器）](https://mybatis.org/mybatis-3/zh_CN/configuration.html#mappers)

> 注意，在编写mybatis的核心配置文件时，配置文件的结构非常重要，必须安装上述的顺序进行编写，不然无法运行。



#### 属性（Properties）

**功能**：用于外部化配置，将数据库连接信息等敏感或需要修改的配置项从 XML 文件中分离出来，存放在 `.properties` 文件中。

##### **外部文件**:通过 `<properties resource="..." />` 或 `<properties url="..." />` 引入的属性文件。

1、在`resource`目录下中创建`mysql.properties`文件，用于保存数据连接信息（用户名，密码等敏感或需要修改的配置）

```properties
driverClass=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSH=true;
user=root
password=123
```

2、在mybatis的核心配置文件mybatis-config.xml中，引入properties文件.通过<properties>标签

```xml
<properties resource="mysql.properties"/>
```

因为`resource`是以`classpath`为跟目录，且`mysql.properties`在`resource`目录下，所有可以直接使用文件名进行加载。

3、把数据库连接信息中的敏感信息用`#{}`来替代，`#{}`是xml文件中的占位符，它可以读取properties中加载的内容

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    读取db.properties中的值-->
    <properties resource="mysql.properties"/>
    
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driverClass}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/kuang/dao/UserMapper.xml"/>
    </mappers>

</configuration>
```

##### **内部标签**:在xml文件中，自己配置给properties进行赋值

```xml
<properties>
    <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSH=true;"/>
    <property name="username" value="root"/>
    <property name="password" value="123"/>
</properties>
```

##### **方法参数**：通过 `SqlSessionFactoryBuilder.build()` 方法传入的属性。

```java
// 在 Java 代码中定义属性
Properties props = new Properties();
props.setProperty("driver", "com.mysql.cj.jdbc.Driver);
props.setProperty("url", "root");
props.setProperty("username", "jdbc:mysql://localhost:3306/mybatis?useSSH=true;");
props.setProperty("password", "123");
// 使用 build 方法传入
SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream, props);
```

> 注意：在 MyBatis 中，`properties` 属性的配置有多个来源，它们的优先级是有明确顺序的。当同一个属性在多个地方被定义时，优先级高的会覆盖优先级低的。
>
> MyBatis 中 `properties` 的优先级顺序从**高到低**排列如下：
>
> 1. **方法参数**：通过 `SqlSessionFactoryBuilder.build()` 方法传入的属性。
> 2. **外部文件**：通过 `<properties resource="..." />` 或 `<properties url="..." />` 引入的属性文件。
> 3. **内部标签**：在 `<properties>` 标签内部直接定义的属性。



#### 设置（settings）

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项设置的含义、默认值等。

| 设置名                             | 描述                                                         | 有效值                                                       | 默认值                                                |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| cacheEnabled                       | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true                                                  |
| lazyLoadingEnabled                 | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false                                                 |
| aggressiveLazyLoading              | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false                                                | false （在 3.4.1 及之前的版本中默认为 true）          |
| ~~multipleResultSetsEnabled~~      | Deprecated. This option has no effect.                       | true \| false                                                | true                                                  |
| useColumnLabel                     | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。 | true \| false                                                | true                                                  |
| useGeneratedKeys                   | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。 | true \| false                                                | False                                                 |
| autoMappingBehavior                | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                               |
| autoMappingUnknownColumnBehavior   | 指定发现自动映射目标未知列（或未知属性类型）的行为。`NONE`: 不做任何反应`WARNING`: 输出警告日志（`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`）`FAILING`: 映射失败 (抛出 `SqlSessionException`)Note that there could be false-positives when `autoMappingBehavior` is set to `FULL`. | NONE, WARNING, FAILING                                       | NONE                                                  |
| defaultExecutorType                | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                |
| defaultStatementTimeout            | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。         | 任意正整数                                                   | 未设置 (null)                                         |
| defaultFetchSize                   | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。 | 任意正整数                                                   | 未设置 (null)                                         |
| defaultResultSetType               | 指定语句默认的滚动策略。（新增于 3.5.2）                     | FORWARD_ONLY \| SCROLL_SENSITIVE \| SCROLL_INSENSITIVE \| DEFAULT（等同于未设置） | 未设置 (null)                                         |
| safeRowBoundsEnabled               | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true \| false                                                | False                                                 |
| safeResultHandlerEnabled           | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。 | true \| false                                                | True                                                  |
| mapUnderscoreToCamelCase           | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | False                                                 |
| localCacheScope                    | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。 | SESSION \| STATEMENT                                         | SESSION                                               |
| jdbcTypeForNull                    | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。              | OTHER                                                 |
| lazyLoadTriggerMethods             | 指定对象的哪些方法触发一次延迟加载。                         | 用逗号分隔的方法列表。                                       | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage           | 指定动态 SQL 生成使用的默认脚本语言。                        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler             | 指定 Enum 使用的默认 `TypeHandler` 。（新增于 3.4.5）        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls                 | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 | true \| false                                                | false                                                 |
| returnInstanceForEmptyRow          | 当返回行的所有列都是空时，MyBatis默认返回 `null`。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2） | true \| false                                                | false                                                 |
| logPrefix                          | 指定 MyBatis 增加到日志名称的前缀。                          | 任何字符串                                                   | 未设置                                                |
| logImpl                            | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J（3.5.9 起废弃） \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置                                                |
| proxyFactory                       | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。            | CGLIB （3.5.10 起废弃） \| JAVASSIST                         | JAVASSIST （MyBatis 3.3 以上）                        |
| vfsImpl                            | 指定 VFS 的实现                                              | 自定义 VFS 的实现的类全限定名，以逗号分隔。                  | 未设置                                                |
| useActualParamName                 | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 `-parameters` 选项。（新增于 3.4.1） | true \| false                                                | true                                                  |
| configurationFactory               | 指定一个提供 `Configuration` 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为`static Configuration getConfiguration()` 的方法。（新增于 3.2.3） | 一个类型别名或完全限定类名。                                 | 未设置                                                |
| shrinkWhitespacesInSql             | 从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5) | true \| false                                                | false                                                 |
| defaultSqlProviderType             | 指定一个拥有 provider 方法的 sql provider 类 （新增于 3.5.6）. 这个类适用于指定 sql provider 注解上的`type`（或 `value`） 属性（当这些属性在注解中被忽略时）。 (e.g. `@SelectProvider`) | 类型别名或者全限定名                                         | 未设置                                                |
| nullableOnForEach                  | 为 'foreach' 标签的 'nullable' 属性指定默认值。（新增于 3.5.9） | true \| false                                                | false                                                 |
| argNameBasedConstructorAutoMapping | 当应用构造器自动映射时，参数名称被用来搜索要映射的列，而不再依赖列的顺序。（新增于 3.5.10） | true \| false                                                | false                                                 |



> 对于普通程序员来来说，上述的setting很多是不会使用的，我们只需要熟练`cacheEnabled`、`lazyLoadingEnabled`、`logImpl`、`mapUnderscoreToCamelCase`这四个即可。

一个配置完整的 settings 元素的示例如下：

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="aggressiveLazyLoading" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="safeResultHandlerEnabled" value="true"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
  <setting name="defaultScriptingLanguage" value="org.apache.ibatis.scripting.xmltags.XMLLanguageDriver"/>
  <setting name="defaultEnumTypeHandler" value="org.apache.ibatis.type.EnumTypeHandler"/>
  <setting name="callSettersOnNulls" value="false"/>
  <setting name="returnInstanceForEmptyRow" value="false"/>
  <setting name="logPrefix" value="exampleLogPreFix_"/>
  <setting name="logImpl" value="SLF4J | LOG4J | LOG4J2 | JDK_LOGGING | COMMONS_LOGGING | STDOUT_LOGGING | NO_LOGGING"/>
  <setting name="proxyFactory" value="CGLIB | JAVASSIST"/>
  <setting name="vfsImpl" value="org.mybatis.example.YourselfVfsImpl"/>
  <setting name="useActualParamName" value="true"/>
  <setting name="configurationFactory" value="org.mybatis.example.ConfigurationFactory"/>
</settings>
```



#### 数据源（dataSource）

dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

- 大多数 MyBatis 应用程序会按示例中的例子来配置数据源。虽然数据源配置是可选的，但如果要启用延迟加载特性，就必须配置数据源。

有三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]"）：

**UNPOOLED**– 这个数据源的实现会每次请求时打开和关闭连接。虽然有点慢，但对那些数据库连接可用性要求不高的简单应用程序来说，是一个很好的选择。 性能表现则依赖于使用的数据库，对某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

- `driver` – 这是 JDBC 驱动的 Java 类全限定名（并不是 JDBC 驱动中可能包含的数据源类）。
- `url` – 这是数据库的 JDBC URL 地址。
- `username` – 登录数据库的用户名。
- `password` – 登录数据库的密码。
- `defaultTransactionIsolationLevel` – 默认的连接事务隔离级别。
- `defaultNetworkTimeout` – 等待数据库操作完成的默认网络超时时间（单位：毫秒）。查看 `java.sql.Connection#setNetworkTimeout()` 的 API 文档以获取更多信息。

作为可选项，你也可以传递属性给数据库驱动。只需在属性名加上“driver.”前缀即可，例如：

- `driver.encoding=UTF8`

这将通过 DriverManager.getConnection(url, driverProperties) 方法传递值为 `UTF8` 的 `encoding` 属性给数据库驱动。

**POOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。

除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：

- `poolMaximumActiveConnections` – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
- `poolMaximumIdleConnections` – 任意时间可能存在的空闲连接数。
- `poolMaximumCheckoutTime` – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
- `poolTimeToWait` – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。
- `poolMaximumLocalBadConnectionTolerance` – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 `poolMaximumIdleConnections` 与 `poolMaximumLocalBadConnectionTolerance` 之和。 默认值：3（新增于 3.4.5）
- `poolPingQuery` – 发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动出错时返回恰当的错误消息。
- `poolPingEnabled` – 是否启用侦测查询。若开启，需要设置 `poolPingQuery` 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句），默认值：false。
- `poolPingConnectionsNotUsedFor` – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。

**JNDI** – 这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。这种数据源配置只需要两个属性：

- `initial_context` – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
- `data_source` – 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。



#### 映射器（mappers）

既然 MyBatis 的行为已经由上述元素配置完了，我们现在就要来定义 SQL 映射语句了。 但首先，我们需要告诉 MyBatis 到哪里去找到这些语句。 在自动查找资源方面，Java 并没有提供一个很好的解决方案，所以最好的办法是直接告诉 MyBatis 到哪里去找映射文件。 你可以使用相对于类路径的资源引用，或完全限定资源定位符（包括 `file:///` 形式的 URL），或类名和包名等。例如：

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

> 注意：在使用class和package是，有一条特别需要注意的点， Mapper接口和Mapper.xml映射文件必须同包同名，不然找不到资源



## 4、XML 映射器

MyBatis 的真正强大在于它的语句映射，这是它的魔力所在。由于它的异常强大，映射器的 XML 文件就显得相对简单。如果拿它跟具有相同功能的 JDBC 代码进行对比，你会立即发现省掉了将近 95% 的代码。MyBatis 致力于减少使用成本，让用户能更专注于 SQL 代码。

SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

- `cache` – 该命名空间的缓存配置。
- `cache-ref` – 引用其它命名空间的缓存配置。
- `resultMap` – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
- ~~`parameterMap` – 老式风格的参数映射。此元素已被废弃，并可能在将来被移除！请使用行内参数映射。文档中不会介绍此元素。~~
- `sql` – 可被其它语句引用的可重用语句块。
- `insert` – 映射插入语句。
- `update` – 映射更新语句。
- `delete` – 映射删除语句。
- `select` – 映射查询语句。



#### 1、select

查询语句是 MyBatis 中最常用的元素之一——光能把数据存到数据库中价值并不大，还要能重新取出来才有用，多数应用也都是查询比修改要频繁。 MyBatis 的基本原则之一是：在每个插入、更新或删除操作之间，通常会执行多个查询操作。

```xml
<select id="getById" parameterType="int" resultType="com.kuang.pojo.User">
  SELECT * FROM USER WHERE ID = #{id}
</select>
```

> 在xml文件中，有两种占位符，`#{}`和`${}`,它们的含义是不同的。
>
> `#{}`: **处理方式**：MyBatis 会将 `#{}` 替换为**问号 `?`**，并使用 **`PreparedStatement`** 来设置参数值。
>
> **优点**：
>
> - **防止 SQL 注入**：MyBatis 会对传入的参数值进行预编译和类型转换，有效地防止了 SQL 注入攻击。
> - **数据安全**：它会将参数值作为**数据**处理，而不是 SQL 代码的一部分，从而保证了数据库的安全性。
> - **性能更好**：由于使用了 `PreparedStatement`，数据库会缓存预编译后的 SQL 语句，后续执行时可以复用，提高了性能。
>
> 
>
> `${}`: **处理方式**：MyBatis 会直接将 `${}` 替换为**变量的字符串值**。
>
> **优点**：
>
> - **动态 SQL 拼接**：可以用来动态地拼接 SQL 语句中的**表名**、**列名**、**排序字段**等，而这些部分是无法作为参数传递的。
>
> **缺点**：
>
> - **存在 SQL 注入风险**：它直接将字符串值拼接到 SQL 语句中，如果传入的参数值包含恶意代码，可能导致 SQL 注入攻击。

因为`${}`存在SQL注入的风险，因此一般情况下都是使用`#{}`



select标签的属性：

| 属性             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `id`             | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| `parameterType`  | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以根据语句中实际传入的参数计算出应该使用的类型处理器（TypeHandler），默认值为未设置（unset）。 |
| ~~parameterMap~~ | ~~用于引用外部 parameterMap 的属性，目前已被废弃。请使用行内参数映射和 parameterType 属性。~~ |
| `resultType`     | 期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。 |
| `resultMap`      | 对外部 resultMap 的命名引用。结果映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂的映射问题都能迎刃而解。 resultType 和 resultMap 之间只能同时使用一个。 |
| `flushCache`     | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。 |
| `useCache`       | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。 |
| `timeout`        | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
| `fetchSize`      | 这是一个给驱动的建议值，尝试让驱动程序每次批量返回的结果行数等于这个设置值。 默认值为未设置（unset）（依赖驱动）。 |
| `statementType`  | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `resultSetType`  | FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。 |
| `databaseId`     | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。 |
| `resultOrdered`  | 这个设置仅针对嵌套结果 select 语句：如果为 true，则假设结果集以正确顺序（排序后）执行映射，当返回新的主结果行时，将不再发生对以前结果行的引用。 这样可以减少内存消耗。默认值：`false`。 |
| `resultSets`     | 这个设置仅适用于多结果集的情况。它将列出语句执行后返回的结果集并赋予每个结果集一个名称，多个名称之间以逗号分隔。 |
| `affectData`     | Set this to true when writing a INSERT, UPDATE or DELETE statement that returns data so that the transaction is controlled properly. Also see [Transaction Control Method](https://mybatis.org/mybatis-3/zh_CN/java-api.html#transaction-control-methods). Default: `false` (since 3.5.12) |



#### 2、insert, update 和 delete

数据变更语句 insert，update 和 delete 的实现非常接近：

```xml
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>

<update id="updateAuthor">
  update Author set
    username = #{username},
    password = #{password},
    email = #{email},
    bio = #{bio}
  where id = #{id}
</update>

<delete id="deleteAuthor">
  delete from Author where id = #{id}
</delete>

```



数据变更语句的属性：

| 属性               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `id`               | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| `parameterType`    | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以根据语句中实际传入的参数计算出应该使用的类型处理器（TypeHandler），默认值为未设置（unset）。 |
| ~~`parameterMap`~~ | ~~用于引用外部 parameterMap 的属性，目前已被废弃。请使用行内参数映射和 parameterType 属性。~~ |
| `flushCache`       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。 |
| `timeout`          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
| `statementType`    | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `useGeneratedKeys` | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。 |
| `keyProperty`      | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| `keyColumn`        | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| `databaseId`       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。 |



> 注意： 在SQL语句传参时，如果不使用@Param注解来声明参数名称时，在mapper接口中写多个参数时，就不可以直接使用形参了。MyBatis 会自动为每个参数创建一个以 `param` 开头的键，例如 `param1`、`param2`。
>
> 例：
>
> ```java
> Integer login(String name, String pwd);
> ```
>
> ```xml
> <select id="login" resultType="integer">
>     select COUNT(*) from user where name=#{name} and pwd = #{pwd}
> </select>
> ```
>
> ![image-20250903230111475](/home/zhang/.config/Typora/typora-user-images/image-20250903230111475.png)
>
> 此时只能使用`param1`、`param2`
>
> ```xml
> <select id="login" resultType="integer">
>     select COUNT(*) from user where name=#{param1} and pwd = #{param2}
> </select>
> ```
>
> ![image-20250903230219485](/home/zhang/.config/Typora/typora-user-images/image-20250903230219485.png)







#### 3、结果映射

`resultMap` 元素是 MyBatis 中最重要最强大的元素。它可以让你从 90% 的 JDBC `ResultSets` 数据提取代码中解放出来，并在一些情形下允许你进行一些 JDBC 不支持的操作。实际上，在为一些比如连接的复杂语句编写映射代码的时候，一份 `resultMap` 能够代替实现同等功能的数千行代码。ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。

##### 1、 什么是结果映射

在 MyBatis 中，结果映射是指将 SQL 查询结果集中的列值映射到 Java 对象的属性中。通过配置结果映射，可以轻松地将复杂的查询结果转换为便于操作的 Java 对象。

```xml
<select id="getById" resultType="com.kuang.pojo.User">
    select * from user where id = #{id};
</select>
```

上述代码就是一个简单的结果映射，它通过resultType属性，指定了数据库查询field映射到Pojo对象中的方式，resultType是采用相同字段名进行映射的，也就是说 `select * from user where id = #{id};`查询的结果是 `id`, `name`, `pwd`字段，MyBatis 会通过反射机制，查找 JavaBean 对象中与这些列名**同名**（不区分大小写）的**属性**。然后通过setting方法进行赋值，因此这种`resultType`方式，数据库的field名必须和pojo对象的属性名同名



##### 2、 高级结果映射

高级结果映射，通常指的是 MyBatis 中的 **`<resultMap>` 标签**。

当 Mybatis 的 `resultType` 无法满足你的需求时，`<resultMap>` 就是你实现复杂映射的强大工具。它让你能够**手动、精确地控制**数据库查询结果与 Java 对象之间的映射关系。

`resultMap` 正是为了解决这些问题而存在的：

1. **列名和属性名不匹配**：数据库列是 `user_name`，而 Java 属性是 `userName`。
2. **复杂的数据类型映射**：需要自定义类型处理器（Type Handler）来转换数据。
3. **一对一关联**：一个用户对象包含一个地址对象。
4. **一对多关联**：一个用户对象包含一个订单列表。
5. **多对多关联**：一个学生拥有多门课程，一门课程有多个学生。



`<resultMap>` 的核心标签

`<resultMap>` 标签提供了一系列子标签来定义映射规则：

- **`<id>`**：用于指定**主键列**。MyBatis 会使用主键来识别结果集的唯一性，这对于处理复杂关联关系非常重要。
- **`<result>`**：用于指定**普通列**。
- **`<association>`**：用于处理**一对一**的关联关系。
- **`<collection>`**：用于处理**一对多**的关联关系。
- **`<discriminator>`**：用于处理**鉴别器**，根据某一列的值来选择不同的 `resultMap`。



###### 1、一对一关联 (`<association>`)

假设一个用户（`User`）拥有一个地址（`Address`）。

1. 查询所有需要的字段，然后赋值

   ```xml
   <resultMap id="userAddressMap" type="com.example.User">
     <id property="id" column="user_id"/>
     <result property="name" column="user_name"/>
     
     <association property="address" javaType="com.example.Address">
       <id property="id" column="address_id"/>
       <result property="street" column="address_street"/>
       <result property="city" column="address_city"/>
     </association>
   </resultMap>
   
   <select id="getUserWithAddress" resultMap="userAddressMap">
     SELECT
       u.id AS user_id,
       u.name AS user_name,
       a.id AS address_id,
       a.street AS address_street,
       a.city AS address_city
     FROM
       user u
     LEFT JOIN
       address a ON u.address_id = a.id
   </select>
   ```

2. 使用多表查询

   ```xml
   <resultMap id="userAddressMap" type="user">
   	<id properties="id" column="user_id"/>
       <result property="name" column="user_name"/>
       
       <association properties="address" select="getAddress" column="address_id" javaType="address">
           <id property="id" column="address_id"/>
           <result property="street" column="address_street"/>
           <result property="city" column="address_city"/>
       </association>
       
   </resultMap>
   
   <select id="getUser" resultMap="userAddressMap">
   	select * from user where id = #{id}
   </select>
   
   <select id="getAddress" resultMap="address">
   	select * from address where id = #{address_id}
   </select>
   ```

   > ```xml
   > <association properties="address" select="getAddress" column="address_id" javaType="address">
   >     <id property="id" column="address_id"/>
   >     <result property="street" column="address_street"/>
   >     <result property="city" column="address_city"/>
   > </association>
   > ```
   >
   > 对于这个语句，我个人的理解是， `properties="address"`对应`User`类中的`address`属性， `select="getAddress"`代表这个`association`需要去调用id为getAddress的sql语句，传递的参数是在`getUser`中查询到的`address_id`属性值，也就是说`column="address_id"`是指定传参的值， `javaType="address"`代表的是这个`select="getAddress"`方法查询的结果是Address对象。然后就可以通过`id`和`result`标签对Address类对象进行赋值了

###### 2、一对多关联 (`<collection>`)

假设一个用户（`User`）拥有多个订单（`Order`）。

```xml
<resultMap id="userOrderMap" type="com.example.User">
  <id property="id" column="user_id"/>
  <result property="name" column="user_name"/>
  
  <collection property="orders" ofType="com.example.Order">
    <id property="id" column="order_id"/>
    <result property="amount" column="order_amount"/>
  </collection>
</resultMap>

<select id="getUserWithOrders" resultMap="userOrderMap">
  SELECT
    u.id AS user_id,
    u.name AS user_name,
    o.id AS order_id,
    o.amount AS order_amount
  FROM
    user u
  LEFT JOIN
    orders o ON u.id = o.user_id
</select>
```



## 5、缓存

#### 概述

缓存的作⽤：通过减少IO的⽅式，来提⾼程序的执⾏效率。

缓存就是指存在内存中的临时数据，使用缓存能够减少和数据库交互的次数，提高效率。将相同查询条件的sql语句执行一遍后得到的结果存在内存或者某种缓存介质中，当下次遇到一模一样的查询sql时候不在执行sql与数据库交互，而是直接从缓存中获取结果，减少服务器的压力。

mybatis缓存包括：

● ⼀级缓存：将查询到的数据存储到SqlSession中。范围比较小，正对于一次sql会话

● ⼆级缓存：将查询到的数据存储到SqlSessionFactory中。范围比较大，针对于整个数据库级别的。

● 或者集成其它第三⽅的缓存：⽐如EhCache【Java语⾔开发的】、Memcache【C语⾔开发的】等。
缓存只针对于DQL语句，也就是缓存机制只对应select语句。

> 在mybatis中，缓存是以Map集合实现的。MyBatis的缓存key是一个复杂的对象，包含了执行SQL所需的所有关键信息。这个对象被称为`CacheKey`，它是由多个部分组合而成的，主要包括：
>
> - **SQL语句的ID (Mapped Statement ID)**：这是XML映射文件中定义的唯一标识符，例如`com.example.mapper.UserMapper.selectUserById`。
> - **SQL语句本身**：经过处理后的、不包含参数占位符的原始SQL字符串。
> - **传入的参数对象**：如果查询有参数，这个参数对象（或参数列表）会被用来生成key的一部分。
> - **分页信息 (RowBounds)**：如果使用了分页，分页的偏移量和限制数量也会被纳入key中。
> - **执行环境 (Environment ID)**：如果你配置了多个数据库环境，环境ID也会被考虑。
> - **MyBatis配置 (Configuration)**：某些配置信息也会影响key的生成。
>
> ![image-20250904092053389](/home/zhang/.config/Typora/typora-user-images/image-20250904092053389.png)





#### MyBatis 一级缓存

##### 一级缓存原理

⼀级缓存默认是开启的。不需要做任何配置。也无法关闭，只能通过某种方式使一级缓存失效。

在一次 SqlSession 中（数据库会话），程序执行多次查询，且查询条件完全相同，多次查询之间程序没有其他增删改操作，则第二次及后面的查询可以从缓存中获取数据，不会走数据库，会走缓存。简单来说，一级缓存的生命周期就是打开SqlSeesion到关闭SqlSeesion之间。



##### 什么时候一级缓存失效?

第一次DQL和第二次DQL之间你做了以下两件事中的任意一件，都会让一级缓存清空:
1．执行了sqlSession的clearCache()方法，这是手动清空缓存。
2．执行了INSERT或DELETE或UPDATE语句。不管你是操作哪张表的，都会清空一级缓存。



#### Mybatis二级缓存

二级缓存的范围是SqlSessionFactory。

使用二级缓存需要具备以下几个条件:

<setting name="cacheEnabled" value="true"> 这个配置表示启用 MyBatis 的二级缓存功能。默认就是true，虽然是默认开启的，但是我们最好还是去显示开启它，显示开启的易读性更高，其他接手我们的程序后也能快速判断出是否开二级缓存

在需要使⽤⼆级缓存的StudentMapper.xml⽂件中添加配置：<cache/>

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.kuang.dao.StudentMapper">
	
    	<cache/>	
    
        <select id="getStudentList" resultMap="StudentTeacher">
            select * from student;
        </select>
</mapper>
```

> 注意： 使⽤⼆级缓存的实体类对象**必须是可序列化**的，也就是必须实现java.io.Serializable接⼝，否则会出现以下异常。
>
> ![image-20250904094009876](/home/zhang/.config/Typora/typora-user-images/image-20250904094009876.png)

二级缓存的原理是**在`SqlSession`提交（`commit`）或关闭（`close`）时**，MyBatis才会将该`SqlSession`一级缓存中的数据同步到二级缓存中。

这意味着，如果在同一个`SqlSession`中进行多次查询，即使你开启了二级缓存，第二次查询也只会命中一级缓存。只有当第一个`SqlSession`结束其生命周期（通过`commit()`或`close()`），第二次查询才会从二级缓存中获取数据。





- 映射语句文件中的所有 select 语句的结果将会被缓存。
- 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
- 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
- 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
- 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
- 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

**提示** 缓存只作用于 cache 标签所在的映射文件中的语句。如果你混合使用 Java API 和 XML 映射文件，在共用接口中的语句将不会被默认缓存。你需要使用 @CacheNamespaceRef 注解指定缓存作用域。

这些属性可以通过 cache 元素的属性来修改。比如：

```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。

可用的清除策略有：

- `LRU` – 最近最少使用：移除最长时间不被使用的对象。
- `FIFO` – 先进先出：按对象进入缓存的顺序来移除它们。
- `SOFT` – 软引用：基于垃圾回收器状态和软引用规则移除对象。
- `WEAK` – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

默认的清除策略是 LRU。

flushInterval（刷新间隔）属性可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。 默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。

size（引用数目）属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。当size设置为0时，其实是没有进行限制，也就是无限大小。

readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。

**提示** 二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。



## 6、动态 SQL

#### 1、概述

动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。



#### 2、动态 SQL 的作用

动态SQL是根据不同条件和需求，动态生成SQL语句的一种技术。它的作用主要有以下几点：

- 条件灵活：使用动态SQL可以根据不同的条件生成不同的SQL语句，使得查询、更新或删除数据时能够根据具体情况进行灵活的处理。

- 查询优化：有时候在编写静态SQL语句时难以预料到查询条件的变化，而使用动态SQL可以根据运行时的条件动态调整查询语句，从而更好地适应实际情况，提高查询性能。

- 动态表名和字段名：有时候需要根据不同的场景来操作不同的表或字段，这时候就可以利用动态SQL来动态构建表名和字段名，实现灵活性和扩展性。

- 防止SQL注入：通过使用参数化查询或者绑定变量的方式来构建动态SQL，可以有效防止SQL注入攻击，提升系统的安全性。




#### 3、动态SQL的常用标签

| 常用标签 | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| if       | 根据指定的条件判断是否包含某部分SQL代码，使得SQL语句在运行时更具灵活性。 |
| where    | 生成动态的WHERE子句，只有满足条件时才包含WHERE子句，避免不必要的WHERE关键字。 |
| choose   | 根据不同的条件选择执行不同的SQL片段，实现类似于switch-case语句的功能。 |
| foreach  | 对集合进行循环，并在SQL语句中使用循环的结果，可以用于动态构建IN或VALUES子句。 |
| set      | 生成动态的SET子句，只有满足条件时才包含SET子句，用于动态更新表中的字段。 |
| trim     | 对SQL语句进行修剪和重组，去掉多余的AND或OR等，以便根据不同的条件动态生成合适的SQL语句。 |
