---
title: Mybatis笔记
date: 2021-06-09 21:22:15
tags: SSM
---
## Mybatis.xml
总的配置文件
Mybatis的配置文件，数据库的连接
```xml
<environment id="development">
        <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
</environment>
```
其中数据库的url配置包含时区，否则会报错
## UserDao.xml
对应接口的配置文件
具体的操作的配置
- namespace其中的包名要和接口的包名一致
- id：就是对应的namespace中的方法名；
- resultType:Sql语句执行的返回值！
- parameterType:参数的类型！
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.Dao.UserDao">   <!--这个包名要和接口的包名一致-->
    <select id="getUser" resultType="com.pojo.User">  <!--这里的id即为接口中的方法名称-->
        select * from user
    </select>

    <insert id="addUser" parameterType="com.pojo.User">
        insert into user value (#{id},#{username});
    </insert>
</mapper>
```
将下面代码配置到Mybatis.xml中
```xml
<mappers>
    <mapper resource="UserDao.xml"/>
</mappers>
```
## 工具类的编写
编写工具类来产生SqlSession对象来调用方法
```java
public class MybatisUtils {
    static SqlSessionFactory sqlSessionFactory;
    static {

        try {
            String resource = "Mybatis.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static SqlSession getsqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```
## CRUD增删改查
1.select
选择查询语句UserDao.xml配置
```xml
    <select id="getUserById" parameterType="int" resultType="com.pojo.User">
        select * from user where id = #{id}
    </select>
```
2.insert
插入语句UserDao.xml配置
```xml
    <insert id="addUser" parameterType="com.pojo.User">
        insert into user value (#{id},#{username});
    </insert>
```
3.update
更新语句UserDao.xml配
```xml
    <update id="upDate" parameterType="com.pojo.User">
        update user set username = #{username} where id = #{id}
    </update>
```
4.delete
删除语句UserDao.xml配置
```xml
    <delete id="DeleteUser" parameterType="int">
        delete from user where id = #{id}
    </delete>
```
**每一个配置都对应接口中的一个方法**
5.测试类
```java
@Test
    public void test05(){
        SqlSession sqlSession = MybatisUtils.getsqlSession();
        UserDao mapper = sqlSession.getMapper(UserDao.class);
        int add = mapper.DeleteUser(4);
        System.out.println(add);
        sqlSession.commit();
        sqlSession.close();
    }
```
通过工具类生成SqlSession对象来获得接口，调用相应的方法   
**增删改必须要提交事务**
模糊查询
 1.java代码执行的时候，传递通配符
 2.在sql拼接中使用通配符！
## 配置解析
- 1.核心配置文件
Mybatis-config.xml
```xml
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
    environment（环境变量）
        transactionManager（事务管理器）
        dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```
- 2.环境配置
MyBatis 可以配置成适应多种环境，
**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**
默认事务管理器JDBC，有连接池
- 3.属性（properties）需要配置在最前面
我们可以通过properties属性来实现引用配置文件
这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。
```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```
- 可以直接引入外部文件
- 可以在其中增加属性配置
- 如果两个文件有相同字段，优先使用外部配置文件
- 4.类型别名（typeAliases）
- 类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。
```xml
方式一：
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
</typeAliases>
```
```xml
方式二：指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
这种方式的别名就为类名，首字母小写
但是当在类上增加注解是，别名为注解
@Alias("author")
public class Author {
    ...
}
```
实体类少用一，多用二
- 5.设置（settings）
无
- 6.映射器（mappers）
MapperRegistry:注册绑定Mapper文件
方式一：【推荐使用】
```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
</mappers>
```
方式二：使用class文件绑定
```xml
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
</mappers>
注意：
1.接口和配置文件必须同名
2.接口和配置文件必须在同一个包下
```
- 7.结果集映射resultMap
```xml
<resultMap id="UserMap" type="User">
    <result column="id" property="id">
    <result column="name" property="name">
    <result column="pwd" property="password">
</resultMap>
<select id="getUser" resultMap="UserMap">
    select * from user where id = #{id}
</select>
其中column对应数据库中的字段名，property对应实体类的属性名，可以解决数据库字段名与实体类属性名不一致导致查找是数据为null的异常
```
## 日志
1.日志工厂
- SLF4J
- LOG4J
- STDOUT_LOGGING  Mybatis中的默认的日志
**在Mybatis中具体使用哪个日志实现，在设置中设定**
**STDOUT_LOGGING效果**
```java
Logging initialized using 'class org.apache.ibatis.logging.stdout.StdOutImpl' adapter.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
PooledDataSource forcefully closed/removed all connections.
Opening JDBC Connection
Created connection 1484673893.
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@587e5365]
==>  Preparing: select * from user
==> Parameters: 
<==    Columns: id, username
<==        Row: 1, wang
<==        Row: 2, li
<==        Row: 3, wu
<==      Total: 3
User{id=1, username='wang'}
User{id=2, username='li'}
User{id=3, username='wu'}
Resetting autocommit to true on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@587e5365]
Closing JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@587e5365]
Returned connection 1484673893 to pool.
```
2.log4j
- 1.先导入包
```xml
<!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>

```
简单使用
1.在要使用log4j的类中，导入包：apache的包
2.日志对象，参数为当前类的class
3.日志级别
- info
- debug
- error
## 分页
1.使用limit分页
```sql
select * from user limit startIndex,pageSize;
```
2.使用Mybatis实现分页
- 1.接口
- 2.Mapper.xml
- 3.测试
## 使用注解开发
- 注解直接在接口上实现
```java
@Select("select * from user")
List<User> getUser();
```
- 绑定注解支持
```xml
<mappers>
    <mapper class="com.Dao.UserDao"/>
</mappers>
```
- 本质为反射机制
  底层动态代理
在工具类中实现自动提交事务
```java
public static SqlSession getsqlSession(){
    return sqlSessionFactory.openSession(true);
}
将参数传递为true，设置autocommit
```
**有多个参数时必须加@Param注解**
Lombok
- 安装插件
- 导入jar包
- 加注解
```java
@Getter and @Setter
@FieldNameConstants
@ToString
@EqualsAndHashCode
@AllArgsConstructor //带参构造
@RequiredArgsConstructor 
@NoArgsConstructor  //无参构造
@Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
@Data //无参，get,set,tostring,hashcode,equals
@Builder
@SuperBuilder
@Singular
@Delegate
@Value
@Accessors
@Wither
@With
@SneakyThrows
```
```java
@Data //get,set,tostring,hashcode,equals
@AllArgsConstructor //带参构造
@NoArgsConstructor//无参构造
public class User {
    private int id;
    private String username;
}
```